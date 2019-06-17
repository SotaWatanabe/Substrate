# Polkadotへの接続

## Polkadot Development Kit

PolkadotのParachainにつなぐメリットは以下の2点です。

* セキュリティのシェア
* インターチェーンコミュニケーション

独自チェーンを作成した場合に、一番の問題点はセキュリティだと思いますが、PolkadotにつなぐことによってPolkadotチェーンのセキュリティをParachain（独自チェーン）に流入することが可能になります。また、Polkadotにつながったチェーン間でdata、Valueの受け渡しが可能になります。

Polkadot Development KitはParachainを簡単につくるためのキットであり以下の2つの要素で構成されています。

* _State transition function_ 
* コレイターノード 

1つの状態から次の状態に移行するために必要なファンクション。またコレイターノードは以下の役割を持ちます。

{% hint style="info" %}
トランザクションコレイター（略してコレイター）は、バリデーターが正当なparacahinブロックを生成するのをサポートするグループである。彼らは、特定のparachainの“full-node”を持つ。これは、現在のPoWブロックチェーンにてマイナーがしているのと同じように、新しいブロックを監視しトランザクションを実行する為に必要な情報を保持している。普通の状態では、まだ承認されていないブロックを生成するために、トランザクションを照合し実行する。そして、ゼロ知識証明と共にブロックをparachainのブロックを提案する責任をもっているバリデーターに伝播する。
{% endhint %}

![Polkadot &#x30DB;&#x30EF;&#x30A4;&#x30C8;&#x30DA;&#x30FC;&#x30D1;&#x30FC;](.gitbook/assets/screen-shot-2019-06-02-at-22.34.38.png)

厳密に言うと、Polkadot Development KitはSubstrateとCumulusです。Substrateで作成したチェーンにCumulusをインポートする（書き足す）ことによりPolkadotでParachainとなるチェーンを作成することができます。

## Cumulus Runtime

SubstrateベースのチェーンをPolkadotのバリデーターが検証することができるようにするもの。CumulusをSubstrateにインポートするのは簡単で、以下の１行をマクロに追加します。

```text
runtime::register_validate_block!(Block, BlockExecutor);
```

Coming Soon.....

