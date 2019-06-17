# ink! をデプロイする

## セットアップ

### Substrateをインストール

基本的には以下の記事に従ってください。

{% page-ref page="substrateinsutru.md" %}

ですが最新版をインストールしても何かしらの不具合が発生する方法がありますので、その場合はバージョンを指定してインストールする方法をとります。

Homebrewから必要パッケージをインストールします。llvmは非常に時間がかかる可能性があります。

```bash
brew install openssl cmake llvm
```

次に、Substrateをcloneし、特定のcommitまで巻き戻してからインストールします。

```bash
g=`mktemp -d`
git clone https://github.com/paritytech/substrate $g
pushd $g
git checkout db9b69c158f234c0a9df706ebee0bbe6645ddeac
./scripts/init.sh
./scripts/build.sh
cargo install --force --path subkey subkey
cargo install --force --path . substrate
popd
```

### wasm ユーティリティ

SubstrateのコントラクトはWebAssembly\(wasm\)にコンパイルされるため、いくつかのwasmユーティリティのインストールが必要です。

まずはwasmのためのRustの環境を整えます。

```bash
rustup update nightly
rustup update stable

rustup target add wasm32-unknown-unknown --toolchain nightly
```

ただしink!はRustの特定のバージョンと結び付けられているため、現時点では

```bash
rustup install nightly-2019-05-21
rustup target add wasm32-unknown-unknown --toolchain nightly-2019-05-21
```

を実行してください。続いて、

* [Binaryen](https://github.com/WebAssembly/binaryen)
* [Wabt](https://github.com/WebAssembly/wabt)
* [Parity wasm-utils](https://github.com/paritytech/wasm-utils)

をインストールします。

```bash
brew install binaryen
brew install wabt
cargo install pwasm-utils-cli --bin wasm-prune
```

後に `wasm2wat` \(wabt\), `wat2wasm` \(wabt\), `wasm-opt` \(binaryen\),  `wasm-prune` \(wasm-utils\)を用います。

### ink! CLI

ink!の新しいプロジェクトを作成するためのプログラムをインストールします。

```bash
cargo install --force --git https://github.com/paritytech/ink cargo-contract
```

## ink! プロジェクトのビルド

### プロジェクトの作成

以上のセットアップの後、以下のコマンドでink!のプロジェクトを作成します。

```bash
cargo contract new flipper
```

ここではflipperのサンプルプログラムがlib.rsに実装されたものが作成されます。ディレクトリの構造は以下のようになっています。

```text
flipper
|
+-- .cargo
|   |
|   +-- config      <-- コンパイル設定 (Safe Math Flag)
|
+-- src
|   |
|   +-- lib.rs      <-- コンパイルソース
|
+-- build.sh        <-- Wasmビルドスクリプト
|
+-- rust-toolchain
|
+-- Cargo.toml
|
+-- .gitignore
```

### テスト

オフチェーンでのテストを実行するスクリプトです。テストがパスしていれば成功です。

```text
cargo test --features test-env
```

### wasmビルド

ビルドスクリプトに実行権限を与えた後、実行します。これでtargetフォルダ以下にwasmファイル等が作成されます。

```bash
chmod +x build.sh
./build.sh
```

他にも、targetフォルダにFlipper.jsonが作成されます。

{% code-tabs %}
{% code-tabs-item title="Flipper.json" %}
```text
{
  "name": "Flipper",
  "deploy": {
    "args": []
  },
  "messages": [{
    "name": "flip",
    "selector": 970692492,
    "mutates": true,
    "args": [],
    "return_type": null
  }, {
    "name": "get",
    "selector": 4266279973,
    "mutates": false,
    "args": [],
    "return_type": "bool"
  }]
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

deployの際に必要な引数、呼び出すことができる関数の情報などを見ることができます。

## ink! をデプロイ

ビルドしたwasmファイルをいよいよSubstrateチェーンにデプロイします。

それではまず、Substrateを起動してみましょう。

```text
substrate --dev
```

以下のようにブロックが生成されているのが見えるはずです。

これで、

![](.gitbook/assets/sukurnshotto-2019-06-06-214038.png)

もし今までにsubstrateを実行したことがありチェーンを作り直したい場合は

```text
substrate　purge-chain --dev
```

で既存のチェーンを削除してから実行してください。

起動できたら、[http://polkadot.js.org/apps](http://polkadot.js.org/apps/) からチェーンの状態を確認できます。

![](.gitbook/assets/sukurnshotto-2019-06-07-00324.png)

### コードをブロックに配置

左側のバーから**Contracts**に移動し、**Code**タブからWasmバイナリをブロックチェーン上に配置します。_deployment account_でAliceを選択し、_compiled contract WASM_ で先ほどビルドしたtarget以下の`flipper-pruned.wasm`を選択します。_contract ABI_には`Flipper.json`\(関数を呼び出す際に必要です\)を選択し、_maximum gas allowed_に 500000を設定します。ここまでできたらDeployボタンを押しましょう。コードがブロックに書き込まれ、`Contract.CodeStored`イベントが発生するはずです。　

![](.gitbook/assets/sukurnshotto-2019-06-07-35030.png)

{% hint style="info" %}
もし、以下のようにsystem.ExtrinsicFailedエラーが出た場合、maximum gas allowedが足りていない場合があります。増やしてみてください。
{% endhint %}

![](.gitbook/assets/sukurnshotto-2019-06-07-40540.png)

### コントラクトのインスタンスの作成

上でコードをチェーン上にデプロイしましたが、まだコントラクトは動いていません\(関数も呼び出せません\)。コードからインスタンスを作成して、初めてコントラクトが動き始めます。

コードのデプロイが成功すると**Contracts** に **Instance**タブが作成されており、_code for this contract_から先ほどのwasmコントラクトが選択できるはずです、_endowment_に1000を、_maximum gas allowed_ に500000を設定して**Instance**ボタンを押してください。

![](.gitbook/assets/sukurnshotto-2019-06-07-42629.png)

上手く行くと沢山のイベントが発生し、**Call**タブが作成されます（ついに関数が呼び出せるようになりました）。

{% hint style="info" %}
イベントメッセージを読むと分かりますが、実はコントラクトのインスタンス作成の際にアカウントが新規作成されています。このアカウント作成ために1000のendowmentが必要となっています。
{% endhint %}

![](.gitbook/assets/sukurnshotto-2019-06-07-43108.png)

## ink!のコントラクト呼び出し

以上でコントラクトが完全にデプロイされたので、Flipperの二つの関数を呼び出していきます。

### get\(\)

`deploy()` 関数ではFlipper コントラクトに初期値として `false` を設定していました。

**Call** タブで _message to send_  に`get(): bool`.  送る_value_ に 0 を、 _maximum gas allowed_ に `100,000` を設定します。

![](.gitbook/assets/sukurnshotto-2019-06-07-63837.png)

そうすると、ターミナルからFlipperの状態が確認できました。確かにdeploy\(\)で設定した初期値のfalseになっています。

![](.gitbook/assets/sukurnshotto-2019-06-07-63925.png)

### flip\(\)

続いて、Flipperの内部変数をtrueに反転させましょう。**Call** タブで _message to send_  に`flip()`.  送る_value_ に 0 を、 _maximum gas allowed_ に `100,000` を設定します。

![](.gitbook/assets/sukurnshotto-2019-06-07-64057.png)

flip\(\)で内部変数の反転ができたら、再びget\(\)を呼び出してターミナルを確認してみましょう。確かに初期値のfalseからtrueに変わっています。

![](.gitbook/assets/sukurnshotto-2019-06-07-64213.png)

これで、2つの関数get\(\)とflip\(\)のコントラクト呼び出しが確認できました。

