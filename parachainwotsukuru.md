# Parachainをつくる

Paracahinを作成するには最低でも5 Dotsが必要になります。DotはRiotの[Polkadot公式チャネル](https://riot.im/app/#/room/#polkadot-watercooler:matrix.org)で頼む、もしくはFaucetでもらうことができます。

ここではPolkadot Wikiにかかれている"Deploy Parachain"を参考に簡単なParachainを作成します。

まず始めにGitHubをクローンします。

```text
$ git clone https://github.com/paritytech/polkadot.git
```

Rustがインストールされているかを確認します。

```text
$ curl https://sh.rustup.rs -sSf | sh
$ sudo apt install make clang pkg-config libssl-dev
$ rustup update
```

ディレクトリーを移動し、ビルドします。

```text
$ cd polkadot/test-parachains
$ ./build.sh
```

ビルドを行ったことにより、WASMで実行可能なファイルが生成されます。これはadderフォルダーの中で確認できます。

**補足：執筆時の最新版はv.0.4.4です。最新版にしないとbuildが通らない可能性があります。**

```text
MacBook-Air-7:polkadot watanabesouta$ git tag
v0.1.0
v0.1.1
v0.1.69
v0.2.0
v0.2.1
v0.2.2
v0.2.3
v0.2.4
v0.2.5
v0.3.0
v0.3.1
v0.3.10
v0.3.11
v0.3.12
v0.3.13
v0.3.14
v0.3.15
v0.3.16
v0.3.17
v0.3.18
v0.3.19
v0.3.2
v0.3.20
v0.3.21
v0.3.3
v0.3.4
v0.3.5
v0.3.6
v0.3.6-1
v0.3.7
v0.3.8
v0.3.9
v0.4.0
v0.4.1
v0.4.2
v0.4.3
v0.4.4
MacBook-Air-7:polkadot watanabesouta$ git checkout v0.4.4
Note: checking out 'v0.4.4'.
```

![](.gitbook/assets/screen-shot-2019-06-04-at-1.00.36.png)

今作ろうとしているチェーンはadderチェーンといい、送られたメッセージを足すだけのシンプルなチェーンです。なので、ファイル名がadderとなっています。

Parachainを起動するにあたり、コレイターを動かす必要があります。コレイターの役割に関しては、[Polkadot Whitepaper](https://github.com/stakedtechnologies/PolkadotWP)を見ることをおすすめします。

```text
$ cd test-parachains/adder/collator
$ cargo build
$ cargo run
```

**補足：生成されたWASMファイルが見当違いのところに生成されているのでPathを指定し直す必要があります。**

![](.gitbook/assets/screen-shot-2019-06-04-at-18.26.51.png)

ここで重要になるのは、Hexコード。今回の場合は以下です。

```text
Hex: 0x00000000000000000000000000000000000000000000000000000000000000000000000000000000011b4d03dd8c01f1049143cf9c4c817e4b167f1d1b83e5c6f0f10d89ba1e7bce
```

これはgenesisブロックの内容として必要となります。

Cargo runをした状態で[Polkadot UI](https://polkadot.js.org/apps/#/extrinsics)にアクセスします。

![](.gitbook/assets/screen-shot-2019-06-04-at-22.00.11.png)

次に必要となる情報を埋めていきます。

* democracy
* Propose\(proposal, value\)
* parachains
* registerParachain\(id, code, initial\_head\_date\)
* initial\_head\_dateには上記で得たHexコードを入力します
* valueにはParachainをつなぐのに最低限必要な5と入力します
* codeにはadder.wasmをアップロードします

![](.gitbook/assets/screen-shot-2019-06-04-at-22.05.51.png)

これでsubmitを行うとdemocracyのreferenda（投票）で承認が得られると、ParachainとしてPolkadotに接続できます。

参考：[Deploy Parachains](https://wiki.polkadot.network/en/latest/polkadot/build/deploy-parachains/)

