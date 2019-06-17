# Substrate UIをつくる

## 事前作業

こちらは当たり前ですが、npmとnodeの最新版はインストールしておきましょう。Substrateを動かすにはRustといくつかのディペンデンシーをインストールしておく必要がありますが、こちらは以下のコードでインストールできます。

 `curl https://getsubstrate.io -sSf | bash`

 時間が結構かかります。

次に、Substrate UIをインストールしてきます。

 `substrate-ui-new substrate`

Substrate UIと同じディレクトリでSubstrateのnode templateもインストールしておきます。

 `substrate-node-new substrate-node-template sota`

 sotaのところには自分の名前を入れてください。

ノードを起動するために、`substrate-node-template`内で

 `./target/release/substrate-node-template --dev`

 を実行します。

実行にあたり、ブロック掘削が始まったのがわかります。

 [![Screen Shot 2019-02-12 at 13.23.28.png](https://camo.qiitausercontent.com/44fcbcce7e82fc26f27edbca454a60e0aa8eb3ed/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f61396333623761662d306438312d343431312d363261392d3035353264663464373065612e706e67)](https://camo.qiitausercontent.com/44fcbcce7e82fc26f27edbca454a60e0aa8eb3ed/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f61396333623761662d306438312d343431312d363261392d3035353264663464373065612e706e67)

走っているコードを見てみると、versionやblockの数などがわかります。Local Node Addressは`/ip4/0.0.0.0/tcp/30333/p2p/QmWH6NCm78tbgGuuH6MmzGvuwhxCRoMT1agrpNzumFshEi`となっていますが、これ多分、libp2pですね。

> プロトコルの拡張性が高いのか。  
> “Why libp2p?” by [@tomaka17](https://twitter.com/tomaka17?ref_src=twsrc%5Etfw) [https://t.co/PIezUBGARJ](https://t.co/PIezUBGARJ)— Sota Watanabe \(渡辺創太\) \(@WatanabeSota\) [February 11, 2019](https://twitter.com/WatanabeSota/status/1094792539649929216?ref_src=twsrc%5Etfw)

[![20180915133806.png](https://camo.qiitausercontent.com/67fc64d1816ee0cc4cbf2ae0d1c7ed191a1fd627/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f65346465323836382d303131322d316466622d643739362d6164323563373866623830312e706e67)](https://camo.qiitausercontent.com/67fc64d1816ee0cc4cbf2ae0d1c7ed191a1fd627/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f65346465323836382d303131322d316466622d643739362d6164323563373866623830312e706e67)

Stackはこういう仕組みになっています。

ここまでできたら、先程インストールした`substrate-ui`のディレクトリーに移動し`npm run dev`コマンドを実行します。この際に、別のタブでブロックの掘削を同時に行っておく必要があります。こんな感じになるはずです。

 [![Screen Shot 2019-02-12 at 13.32.39.png](https://camo.qiitausercontent.com/646c8c7e16a3227bf424370327b503c5fe46c6b3/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f66323366323531642d613538652d373033612d323734352d3263643131303036363863312e706e67)](https://camo.qiitausercontent.com/646c8c7e16a3227bf424370327b503c5fe46c6b3/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f66323366323531642d613538652d373033612d323734352d3263643131303036363863312e706e67)

その後、[http://localhost:8000/](http://localhost:8000/)にアクセスします。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.49.49.png)

ここで自分で作成したチェーンとやりとりをすることができます。

## ネットワークに他のプレーヤーを追加する

独自環境を構築したので、現時点では参加者は自分1人です。ここに別の参加者を加えたいと思います。

`git clone https://github.com/paritytech/substrate`で、Substrateのインストールを行います。

 [![Screen Shot 2019-02-12 at 15.02.14.png](https://camo.qiitausercontent.com/1222d5ad5f719541fc9b9afe7ffeaf7125e52fcc/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f61666639303734362d636134392d353965382d316131312d3030353465396336653066342e706e67)](https://camo.qiitausercontent.com/1222d5ad5f719541fc9b9afe7ffeaf7125e52fcc/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f61666639303734362d636134392d353965382d316131312d3030353465396336653066342e706e67)

現在ファイルは3つです。インストールしたsubstrateのファイル内に`subkey`というファイルがあるので、その中で、

 `$ subkey restore Alice`

 を実行するとAliceのKey情報が入手できます。

[![Screen Shot 2019-02-12 at 15.00.54.png](https://camo.qiitausercontent.com/e0880c325f440c3dea4ec1589789b52d1a6399ee/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f38613034623137372d666163342d363031362d643531642d3933633863336131343037352e706e67)](https://camo.qiitausercontent.com/e0880c325f440c3dea4ec1589789b52d1a6399ee/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3332373930342f38613034623137372d666163342d363031362d643531642d3933633863336131343037352e706e67)

これを`localhost:8000`のインターフェース上で入力することでAliceを加えることができます。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.50.55.png)

追加しました。

これでAliceのWalletからDefaultのWalletにunitsを送信することができます。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.51.33.png)

## Runtimeモジュールを作成する

RuntimeとはSubstrateにおけるブロック処理ロジックなどを決める機能です。State Transaction Functiuonとも言われるときもあり、WebAssemblyバイナリーでオンチェーンに記載さてています。詳しくは[Scraobox](https://scrapbox.io/StakedTechnologies/Substrate_%E2%91%A0Basic)に記載しています。

前回、`substrate-node-template`をインストールしましたが、`./runtime/src/demo.rs`のフォルダに記載していきます。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.53.02.png)

作成した`demo.rs`にいくつかのライブラリーをインストールします。

```text
use parity_codec::Encode;
use support::{StorageValue, dispatch::Result, decl_module, decl_storage};
use runtime_primitives::traits::Hash;
use {balances, system::{self, ensure_signed}};

pub trait Trait: balance::Trait {}
```

Web3 Summitのプレゼンでは、簡単な賭けゲームを作成していました。ゲームに勝てば、Potに溜まっていた金額を得ることができ、負ければPotに没収されるという簡単な仕組みです。

ライブラリーをインストールしたので、次に、モジュールを宣言する必要があります。\(module declaration）

```text
decl_module! {
    pub struct Module<T: Trait> for enum Call where origin: T::Origin {
        fn play(origin) -> Result{
            //ゲームをプレイするロジック
        }

        fn set_payment(_origin, value: T::Balance) -> Result{
            //ゲームのpaymentの仕様
        }
    }
}
```

```text
decl_storage! {
  trait Store for Module<T: Trait> as Demo {
    Payment get(payment): Option<T::Balance>;
    Pot get(pot): T::Balance;
  }
}
```

strageモジュールはオンチェーンに載せる情報を定義するために使用します。

次に、モジュール内のロジックを書いていきます。

#### ゲームをプレイするロジック

```text
fn play(origin) -> Result {

  let sender = ensure_signed(origin)?;
  //decl_strage!で宣言しているからpaymentが使える
  let payment = Self::payment().ok_or("Must have payment amount set")?;
　//senderの残高を減少させる
  <balances::Module<T>>::decrease_free_balance(&sender, payment)?;

  //ハッシュ関数を通してハッシュ値の最初のbyteが128以下であれば勝ち。potにあった金額がSenderに払われる
  if (<system::Module<T>>::random_seed(), &sender)
  .using_encoded(<T as system::Trait>::Hashing::hash)
  .using_encoded(|e| e[0] < 128)
  {
    <balances::Module<T>>::increase_free_balance_creating(&sender, <Pot<T>>::take());
  }
　
  //結果どうあれ、senderが賭けに参加した金額がデポジットされる
  <Pot<T>>::mutate(|pot| *pot += payment);

  Ok(())
}
```

#### ゲーム内のPaymentのロジック

```text
fn set_payment(_origin, value: T::Balance) -> Result {
　//イニシャルpaymentがセットされていない場合の処理
  if Self::payment().is_none() {

    <Payment<T>>::put(value);
    <Pot<T>>::put(value);
  }

  Ok(())
}
```

## 変更をアップデートする

上記でModuleを設定しました。変更を実行するために`./runtime/src/lib.rs`を編集します。

`mod demo；`の追記。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.54.13.png)

`impl demo::Trait for Runtime {}`の追記

![](.gitbook/assets/screen-shot-2019-06-01-at-21.54.49.png)

`Demo: demo::{Module, Call, Storage},`の追記

![](.gitbook/assets/screen-shot-2019-06-01-at-21.55.23.png)

上記で新しいRuntimeモジュールを作成したので、次にブロックチェーンのアップデートをします。

Substrate-node-templateのディレクトリでビルド  
`substrate-node-template $ ./build.sh`  


![](.gitbook/assets/screen-shot-2019-06-01-at-21.56.02.png)

ビルド後、substrate-uiの`npm run dev`で接続したlocalhost8000にて一番下にRuntime Upgradeができるので、Select Runtimeを押し、

`./runtime/wasm/target/wasm32-unknown-unknown/release/node_runtime.compact.wasm`

を選択する。これでRuntimeのアップデートができました。Developer Consoleでみると動いていることがわかります。

![](.gitbook/assets/screen-shot-2019-06-01-at-21.56.35.png)

## UIを整える

UIを整えるにはsubstrate-uiレポジトリーを編集する必要があります。  
`./substrate-ui/src/app.jsx`を編集します。

新しく以下のセクションを追加します。

```text
<Divider hidden />
<Segment style={{margin: '1em'}} padded>
  <Header as='h2'>
    <Icon name='game' />
    <Header.Content>
      Play the game
      <Header.Subheader>Play the game here!</Header.Subheader>
    </Header.Content>
  </Header>
  <div style={{paddingBottom: '1em'}}>
    <div style={{fontSize: 'small'}}>player</div>
    <SignerBond bond={this.player}/>
    <If condition={this.player.ready()} then={<span>
      <Label>Balance
        <Label.Detail>
          <Pretty value={runtime.balances.balance(this.player)}/>
        </Label.Detail>
      </Label>
    </span>}/>
  </div>
  <TransactButton
    content="Play"
    icon='game'
    tx={{
      sender: this.player,
      call: calls.demo.play()
    }}
  />
  <Label>Pot Balance
    <Label.Detail>
      <Pretty value={runtime.demo.pot}/>
    </Label.Detail>
  </Label>
</Segment>

```

同時に、exportの部分に`this.player = new Bond;`を追記します。

```text
................
this.seedAccount.use()
this.runtime = new Bond;
//これ
this.player = new Bond;
}
```

これでUIを構築することができ、ゲームをPlayすることができます。  


![](.gitbook/assets/screen-shot-2019-06-01-at-21.57.11.png)

Source:大本のビデオは[こちら](https://www.youtube.com/watch?v=0IoUZdDi5Is&feature=youtu.be)を参考にしてください。

