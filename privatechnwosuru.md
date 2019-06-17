# Privateチェーンを構築する

この章では、Substrateを用いてPrivateチェーンを構築する方法を解説します。

まず始めにSubstrateのv1をインストールします。

```text
$ cd desktop
$ git clone -b v1.0 https://github.com/paritytech/substrate
$ cd substrate
```

keypairsを作成するためのsubkeyを最初にインストールします。

```text
$ cargo install --force --path subkey subkey
```

![](.gitbook/assets/screen-shot-2019-06-04-at-11.47.18.png)

次にブロックチェーンノードを起動するためにコンパイルします。SubstrateはRuntimeを独自に定義することによってチェーンをカスタマイズすることができます。Substrateのディレクトリーにはnodeとnode-templateというディレクトリーが用意されていますが、ここではよりシンプルなnode-templateの方を使います。

```text
$ cd node-template
# コンパイル
$ ./scripts/init.sh
$ ./scripts/build.sh
$ cargo build
```

Substrateをアップデートしたい場合には、

```text
# v1.0 のbranchでgit pullを行う
$ git checkout v1.0
$ git pull
```

を行い、アップデートした上でビルドを行います。

### AliceとBobがブロックチェーンを作成する

#### Aliceの処理

今回はAliceとBobのみがアクセスすることができるプライベートネットワークを作成します。まずAlice側の処理から開始します。

```text
$ pwd
/Users/watanabesouta/desktop/substrate

$ ./target/debug/node-template \
  --base-path /tmp/alice \ #データをどこに管理するか
  --chain=local \ #どのチェーンの仕様を使うか
  --key //Alice \ #どのキーをつかうか
  --port 30333 \ #ポート番号
  --telemetry-url ws://telemetry.polkadot.io:1024 \　#telemetry dataをどのサーバーにおくるか
  --validator \ #バリデートする権限
  --name AlicesNode #ノードの名前
```

上記のコマンドを実行すると、以下のように処理が実行されます。

![](.gitbook/assets/screen-shot-2019-06-04-at-13.15.16.png)

* `Local node identity is: QmUkc9rPukftYmSGjM7J7PvxGEJTcGQhRfSA83SRv8K1Y1` 
  *  BobがAliceのノードと接続を行うときに必要なID。
* `Listening for new connections on 127.0.0.1:9944.`
  * RPCコネクションポート番号
* `./target/debug/node-template --help` 
  * を行うことで可能な処理を見ることができる

UIは、[Polkadot JS](https://polkadot.js.org/apps/#/explorer)が用意されています。仮にUIを変更したい場合には、

```text
$ git clone https://github.com/polkadot-js/apps
$ cd apps

$ yarn
$ yarn run start
```

を行い、localhost3000にアクセスします。

![](.gitbook/assets/screen-shot-2019-06-04-at-13.34.40.png)

ここで見られるように、last blockが0のままです。これはつまりブロックが生成されていません。Substrateのノードはバリデーターが2人以上になった時からブロック生成を開始します。現在Aliceしかいないので0のままとなっています。

#### Bob側の処理

Aliceのノードを動かした状態で、Bobは以下のコードを実行します。

```text
$ cd desktop/substrate
$ ./target/debug/node-template \
  --base-path /tmp/bob \
  --chain=local \
  --key //Bob \
  --port 30333 \
  --telemetry-url ws://telemetry.polkadot.io:1024 \
  --validator \
  --name BobsNode \
  --bootnodes /ip4/<Alices IP Address>/tcp/<Alices Port>/p2p/<Alices Node ID>
```

* AliceのIPアドレスには192.168.1.1を
* Aliceのポート番号は30333を
* Aliceのノード番号はQmNTx7qYd5JiasUTzjaAbTCUeH5ot9xMjRePRjkSEdD7MYを入力します。（Aliceのノードを立ち上げたときに確認することができます。）

![](.gitbook/assets/screen-shot-2019-06-04-at-13.48.28.png)

![Peer1&#x3068;&#x306A;&#x308A;&#x3064;&#x306A;&#x304C;&#x3063;&#x3066;&#x3044;&#x308B;&#x3053;&#x3068;&#x304C;&#x308F;&#x304B;&#x308A;&#x307E;&#x3059;](.gitbook/assets/screen-shot-2019-06-04-at-13.48.57.png)

参考：[Start a private network with Substrate](https://docs.substrate.dev/docs/deploying-a-substrate-node-chain)

