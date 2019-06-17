# ink! 概要

ink!とは、Substrate上でスマートコントラクトをRustで記述するためのeDSLです。

## ink!の言語構造

ink! は3層構造であると言えます。

* **Core**: スマートコントラクトを書く際の最も低レイヤーな記述で、データストレージなどを行います。
* **Model**: [Fleetwood](https://github.com/paritytech/fleetwood)に影響を受けた中間レイヤーの記述です。
* **Language \(Lang\)**: CoreとModelをベースに、簡潔ににスマートコントラクトを記述します。

ink!では、コントラクトは必ず2Stepで実行されます。

1. コードをブロックチェーン上に置く。
2. コードのインスタンスを生成する。

Ethureumでは初期値だけが違うようなコントラクトのたびにFull sizeのコードがブロック上に置かれていますが、Substrateでは共通部分のみのコードが置かれれば良いため、データサイズの効率が良いです。

注意点としては、コントラクトの値は必ず初期化されていなければなりません。さもなければ、コントラクトによる内部変数の状態の遷移が起きないままgasだけが消費されてしまいます。

コントラクトはDeployトレイトが実装されていなければならず、Deployトレイトにはdeploy関数が必須です。基本的に初期化はdeploy関数内で行います。

## コードの例

次の章でデプロイするink!コードの例です。これは、"true"と"false"をとるbool値を反転させることができるスマートコントラクトです。順に見ていきます。

{% code-tabs %}
{% code-tabs-item title="flipper/src/lib.rs" %}
```rust
#![cfg_attr(not(any(test, feature = "std")), no_std)]

use ink_core::{
    env::println,
    memory::format,
    storage,
};
use ink_lang::contract;

contract! {
    /// "true"と"false"をとるbool値を"flip"メッセージで反転できるスマートコントラクト。
    /// "get" メッセージで現在の状態を取得できる。
    struct Flipper {
        /// 現在の真偽値の状態変数
        value: storage::Value<bool>,
    }

    impl Deploy for Flipper {
        /// スマートコントラクトデプロイ時に真偽値をfalseに初期化
        fn deploy(&mut self) {
            self.value.set(false)
        }
    }

    impl Flipper {
        /// 真偽値を反転させるスマートコントラクトの関数
        pub(external) fn flip(&mut self) {
            *self.value = !*self.value;
        }

        /// 現在の真偽値の状態を返す関数
        pub(external) fn get(&self) -> bool {
            println(&format!("Storage Value: {:?}", *self.value));
            *self.value
        }
    }
}

#[cfg(all(test, feature = "test-env"))]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        let mut contract = Flipper::deploy_mock();
        assert_eq!(contract.get(), false);
        contract.flip();
        assert_eq!(contract.get(), true);
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}



### 変数の定義

`struct Flipper`中で、内部変数を定義します。

{% code-tabs %}
{% code-tabs-item title="flipper/src/lib.rs" %}
```rust
    struct Flipper {
        /// 現在の真偽値の状態変数
        value: storage::Value<bool>,
    }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

`storage::Value`は`u8`以外の様々なプリミティブ型：`bool`, `u{16,32,64,128}`, `i{8,16,32,64,128}`, `String`, タプル, 配列をサポートしています。

また、AccountId,Balance,HashなどのSubstrate固有の型を利用するためには、ink\_core::envからインポートすれば良いです。

```rust
use ink_core::env::{AccountId, Balance};

contract! {
    struct MyContract {
        // Store some AccountId
        my_account: storage::Value<AccountId>,
        // Store some Balance
        my_balance: storage::Value<Balance>,
        // Store some Hash
        my_hash: storage::Value<Hash>,
    }
    ...
}
```

 その他にも、以下のように変数としてMapを持つこともできます。これはAccountIdからu32へ対応させるマップです。

```rust
contract! {
    struct MyContract {
        my_number_map: storage::HashMap<AccountId, u32>,
    }
    impl Deploy for MyContract {
        fn deploy(&mut self) {}
    }
    impl MyContract {
        fn my_number_or_zero(&self, of: &AccountId) -> u32 {
            let balance = self.my_number_map.get(of).unwrap_or(&0);
            *balance
        }
    }
}
```

{% hint style="info" %}
HashMapのAPIはink! の [core/storage/collections/hash\_map](https://github.com/paritytech/ink/blob/master/core/src/storage/collections/hash_map/impls.rs) から確認することが出来ます。
{% endhint %}

{% code-tabs %}
{% code-tabs-item title="core/storage/collections/hash\_map/impls.rs" %}
```rust
...
    pub fn insert(&mut self, key: K, val: V) -> Option<V> {...}
    pub fn remove<Q>(&mut self, key: &Q) -> Option<V> {...}
    pub fn get<Q>(&self, key: &Q) -> Option<&V> {...}
    pub fn get_mut<Q>(&mut self, key: &Q) -> Option<&mut V> {...}
...
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### deploy時の初期化処理

FlipperはDeployトレイトを実装し、Deployトレイトはdeploy関数を含みます。deploy関数の中に、スマートコントラクトをデプロイする際の初期化処理として内部変数への値の設定を行います。

{% code-tabs %}
{% code-tabs-item title="flipper/src/lib.rs" %}
```rust
    impl Deploy for Flipper {
        /// スマートコントラクトデプロイ時に真偽値をfalseに初期化
        fn deploy(&mut self) {
            self.value.set(false)
        }
    }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
deploy関数が呼び出されるタイミングは、正確には、コードがブロックチェーン上に配置された時ではなく、その後にコードからインスタンスが作成されるタイミングです。\(前述の通り、ink!でのスマートコントラクト作成は2段階です\)

そのため、一度コードをチェーンにセットした後は、初期値のみが異なるスマートコントラクトであれば簡単に作成出来ます。
{% endhint %}

### スマートコントラクトメッセージ

`impl Flipper` 中に、コントラクトのインスタンス作成後にメッセージとして実行できる関数flipとgetを書きます。公開関数にするために、通常の`pub`ではなく`pub(external)` 記述する必要があります。

{% code-tabs %}
{% code-tabs-item title="flipper/src/lib.rs" %}
```rust

    impl Flipper {
        /// 真偽値を反転させるスマートコントラクトの関数
        pub(external) fn flip(&mut self) {
            *self.value = !*self.value;
        }

        /// 現在の真偽値の状態を返す関数
        pub(external) fn get(&self) -> bool {
            println(&format!("Storage Value: {:?}", *self.value));
            *self.value
        }
    }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

内部変数を反転させるflipは、内部変数の変更を伴うため`&mut self` をとる必要があります。

get関数中では、println とformat! を組み合わせて現在の真偽値を表示しています。ターミナルでは以下のように表示されています。

![](.gitbook/assets/sukurnshotto-2019-06-07-64213.png)

クライアントからのコントラクト呼び出しgetでは、関数の返り値としてのbool値は直接取得できず、このようにprintlnで表示された情報を通して内部状態を確認しています。

{% hint style="info" %}
そもそもスマートコントラクトは状態を遷移させるためのもので、**Substrateのランタイム呼び出し自体は値を返しません**。実際既存のコントラクト\(Ethereum\)などもコントラクト呼び出しでは値をreturnしない仕様になっております。
{% endhint %}

println とformat! の利用には、以下のインポートが必要です。

```rust
use ink_core::env::println;
use ink_core::memory::format;
```

ただしこれらはあくまでデバッグ用の関数なので、substrate コマンド実行の際に `--dev` ****オプションがなければrejectされます。



