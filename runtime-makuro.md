# Runtime マクロ

## Runtime の構造

Substrate Runtimeの構成は上の図のようになっており、Substrate Runtime Module Libraryを組み合わせてRuntimeを作成することになります。\(construct\_runtime!マクロで行います\)。

ですが、SRML以外にもモジュールを自作することができるので、その際に必要となるマクロについて説明します。

![Architecture of a Runtime](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/214076/7cd925bd-49c3-44e3-7911-cf56f781c13d.png)

## カスタムモジュールの定義

自作のモジュールを定義する際、Rustのマクロを用います。マクロなので、通常のRustとは違う構文になることに注意してください。

### decl\_storage!

ブロックチェーンに保存するデータを定義します。

基本的な構文は以下のようになります。例えば値を定義することも、mapを定義することもできます。\(それぞれuse support::StoreValue,use support::StoreMapが必要です。\)

{% code-tabs %}
{% code-tabs-item title="runtime/src/mysrml.rs" %}
```rust
decl_storage! {
    trait Store for Module<T: Trait> as MyStore {
        MyValue : u32;
        MyMap: map u32 => u32;
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

wasmコンパイルされると、javascriptからは

```javascript
runtime.mysrml.myValue
runtime.mysrml.myMap(1)
```

のように呼び出せます。先頭が小文字になるので注意が必要です。

### decl\_module!

ユーザーが最終的に呼び出すことになるpublicな関数を定義します。\(Polkadot UIで言うExtrinct\)

{% hint style="info" %}
この関数は、panicしてはいけません。状態遷移の途中でエラーが発生すると、状態が巻き戻されず中途半端なまま終了します。

例えば、データの所有権を移す状態遷移と引き換えにbalanceを送る状態遷移を行おうとしたが、balanceが足りずに実行時エラーが発生すると、データの所有権のみが移り終了してしまします。

そのためエラーが発生する可能性を予め関数内でチェックしなければなりません。
{% endhint %}

decl\_module!の基本的な構文は以下のようになります。これはdecl\_storage!で定義したMyValueとMyMapに値を設定する関数です。

{% code-tabs %}
{% code-tabs-item title="runtime/src/mysrml.rs" %}
```rust
decl_module! {
  pub struct Module<T: Trait> for enum Call where origin: T::Origin {
    fn set_my_value(origin, my_value: u32) -> Result {
      let _sender = ensure_signed(origin)?;
      <MyValue<T>>::put(my_value);
      Ok(())
    }

    fn set_my_map(origin, key: u32, val: u32) -> Result {
      let _sender = ensure_signed(origin)?;
      <MyMap<T>>::insert(key,Val);
      Ok(())
    }
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* pub struct〜の定義はおまじないのようにほぼそのまま書くことになると思われます。
* originは関数を実行しているアカウントを指すので、関数を定義する際に必要となります。関数定義にoriginの型指定がありませんが、\(Rustでは関数定義の際に必要\)これはマクロなのでなんか上手いことやってくれています。

というのも、マクロが展開されると内部的にはCallというenumが作られ、Runtimeではそれを呼び出すことにります。

{% code-tabs %}
{% code-tabs-item title="runtime/src/mysrml.rs" %}
```rust
pub enum Call<T: Trait> {
    set_my_value(u32),
    set_my_map(u32,u32),
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

wasmコンパイルされると、javascriptからは

```javascript
call.mysrml.setMyValue(1)
call.mysrml.setMyMap(1,2)
```

のように呼び出せるようになります。\(命名規則に注意が必要です。\)

### construct\_runtime!

こちらはlib.rsに書くマクロです。

SRMLのモジュールや自作のカスタムモジュールから、最終的にRuntimeで実行できるもの全てをincludeしています。

​substrate-node-templateでのサンプルは以下のようになっており、{ }の中で一行ずつ、SRMLからモジュールの型を指定してをインポートしています。

{% code-tabs %}
{% code-tabs-item title="runtime/src/lib.rs" %}
```rust
construct_runtime!(
    pub enum Runtime with Log(InternalLog: DigestItem<Hash, Ed25519AuthorityId>) where
        Block = Block,
        NodeBlock = opaque::Block,
        UncheckedExtrinsic = UncheckedExtrinsic
    {
        System: system::{default, Log(ChangesTrieRoot)},
        Timestamp: timestamp::{Module, Call, Storage, Config<T>, Inherent},
        Consensus: consensus::{Module, Call, Storage, Config<T>, Log(AuthoritiesChange), Inherent},
        Aura: aura::{Module},
        Indices: indices,
        Balances: balances,
        Sudo: sudo,
    }
);
```
{% endcode-tabs-item %}
{% endcode-tabs %}

例えば先ほどdecl\_storage!とdecl\_module!で定義したモジュールを含める場合は、以下の行を追加します。

```text
MySrml: mysrml::{Module ,Call, Storage},
```

他にも、インポートするモジュールの型にはEvent, Origin, Config, Log, Inherentなどがあります。

