---
description: Plasmバージョン0.2.0
---

# PlasmでPlasmaチェーンをつくる

![](.gitbook/assets/screen-shot-2019-06-01-at-17.43.16.png)

We have been making Plasm for several months. Last week, we [announced](https://medium.com/@satelliteye/release-of-plasm-ver0-2-0-plasma-on-substrate-6dfcf306752f) the official release of Plasm ver0.2.0 which adds Plasma functions to your Substrate chain. From now on, not only we but also **YOU** can make a plasma chain with Plasm and Substrate. I don’t intend to explain what Substrate is and what Plasma is. If you are not familiar with them, you can check [here](https://medium.com/staked-technologies/plasm-plasma-on-substrate-16f017fc41e), here, [here](https://plasma.io/), [Plasm GitHub](https://github.com/stakedtechnologies/Plasm) and [Plasm Client GitHub](https://github.com/stakedtechnologies/plasm-client)

In this article, we would like to show you how to make Plasma chains with Plasm and Substrate. If you want to know the basic functions and philosophy of Plasm, you can check [our previous article](https://medium.com/staked-technologies/plasm-plasma-on-substrate-16f017fc41e).

![](.gitbook/assets/logo.svg)

Plasm ver0.2.0 is a prototype which has the following functions.

### **①** [**Plasma MVP implementation**](https://github.com/stakedtechnologies/Plasm) <a id="0f23"></a>

1. [**Plasm-Parent**](https://github.com/stakedtechnologies/Plasm/tree/master/core/parent) provides the parent chain’s specification. Mainly, Plasm-Parent has the logic of each [exit game](https://github.com/cryptoeconomicslab/plasma-chamber/wiki/Exit-Game).
2. [**Plasm-Child**](https://github.com/stakedtechnologies/Plasm/tree/master/core/child) provides the child chain’s specification.
3. **Plasm** [**UTXO**](https://github.com/stakedtechnologies/Plasm/tree/master/core/utxo)**/**[**Merkle**](https://github.com/stakedtechnologies/Plasm/tree/master/core/merkle) provides the data structure which manages transactions.

### ② [Plasma Client implementation](https://github.com/stakedtechnologies/plasm-client) <a id="73be"></a>

1. **Plasm Util** is a wrapper function to call the endpoint of blockchains.
2. **Plasm Operator** is monitoring a parent chain and a child chain to make the deposit/exit successful.
3. **Plasm CLI** is a CLI tool to call the endpoint.
4. **Plasm Wallet** is an application to send, withdraw and receive tokens.

As a next step, let’s make a wallet demo application on your laptop and see what’s happening.

## Step1 <a id="927a"></a>

Clone [**Plasm**](https://github.com/stakedtechnologies/Plasm.git) from our GitHub.

```text
> git clone https://github.com/stakedtechnologies/Plasm.git
> cd Plasm
> git checkout v0.2.0
```

## Step2 <a id="cb97"></a>

Build Plasm Node. After a successful build, you can run Plasma nodes.

```text
> cargo build
> ./target/debug/plasm-node --dev
```

## Step3 <a id="e492"></a>

Open another terminal and clone Plasm-Client from our GitHub.

```text
> git clone https://github.com/stakedtechnologies/plasm-client.git
> cd plasm-client
> git checkout v0.2.0
```

## Step4 <a id="f428"></a>

Start **Plasm Operator.** The operator is monitoring both the parent chain and the child chain. When the parent chain deposits tokens to the child chain or the child chain exits tokens to the parent chain, the operator writes the root hash of the child chain on the parent chain.

```text
> cd packages/operator
> cp ../../env.sample .env
> yarn install
> yarn start
```

![](https://cdn-images-1.medium.com/max/1600/1*hW9OvDD3Pw-OpuXm5QkR7g.png)

If you can see the output below, this project has been successful.

![](.gitbook/assets/screen-shot-2019-06-01-at-17.43.27.png)

Yeah, you made it, but what actually happened? Let me clarify! \(You can skip if you want. This is complicated, so we will publish another article focused on this topic.\)

> Plasm-Operator gets an event which the plasma child/parent chain issues and processes the following steps.

> **Parent chain’s events  
> -** When an operator receives a Submit event, she finalizes the status of the child chain.  
> - When an operator receives a Deposit event, she sends tokens to the issuer.  
> - When an operator receives an ExitStart event, she deletes the UTXO which was used for exiting on the child chain.

> **Child chain’s events.**  
> When an operator receives a Submit event, she submits the root hash to the parent chain. The Submit event on a child chain is issued regularly. （You can decide this logic. For this time, 1 time in 5 blocks.\)

## **Step5** <a id="c4df"></a>

Open another terminal and move to **plasm-client** root directory. Then, start Plasm Wallet UI Demo Application.

```text
> cd packages/wallet
> yarn install
> yarn dev
```

![](https://cdn-images-1.medium.com/max/1600/1*LDgQNGlKJLU4L7LnPcdhzQ.png)

After that, let’s go to [localhost:8000](http://localhost:8000/) on your browser.

We will create 2 different accounts and send/receive tokens by using the wallet application I mentioned above.

## Account Registration <a id="9027"></a>

First, you need to register your demo account. Since a default operator is Alice, you should add **//Alice** ①. Then, create an account ②. You can check Alice’s balance ③.

![](https://cdn-images-1.medium.com/max/2400/1*nD1MPjjGs_6liE6ZAAKfGA.png)

To send tokens from Alice to Bob, Alice to Tom and Bob to Tom, generate Bob’s and Tom’s account as well.

![](https://cdn-images-1.medium.com/max/2400/1*DqnL6OFlSLYEKYTGsvPt7w.png)

## Token Transfer on Parent Chain <a id="c123"></a>

As a next step, we will send tokens from Alice to Bob and Alice to Tom on the parent chain.

![](https://cdn-images-1.medium.com/max/1600/1*0P8F4R8uv7p6BZkAvcgi0Q.png)

![](https://cdn-images-1.medium.com/max/1200/1*p_i7_Kcq3mNQyJRJzIBiGA.png)

![](https://cdn-images-1.medium.com/max/1200/1*juq-JPTAV-wsGbKIUADGHg.png)Send tokens from Alice to Bob. You can send from Alice to Tom as well.

Enter the account name and decide the amount of token. Then, click the “Send” button. Keep your eye on the “ParentBalance” next to the account name. After a successful transaction, you will notice that Bob’s amount is increasing. Currently, we collect the exchange fee from the sender on the parent chain. So, Bob and Tom need to have some tokens on the parent chain.

## Deposit \(Deposit tokens from Parent Wallet to Child Wallet.\) <a id="040e"></a>

![](https://cdn-images-1.medium.com/max/1600/1*6PE3Qf8xRaRtB8Mwy_v0ZA.png)

Third, we will send tokens from the parent chain to the child chain. For this time, Bob deposits 5,000,000 tokens to the child chain. Just keep in mind, it takes time to increase ChildBalance because the operator checks the event and executes a transaction.

![](https://cdn-images-1.medium.com/max/1200/1*82M5Bua_oyvG-AzHf8Refw.png)

![](https://cdn-images-1.medium.com/max/1200/1*rdNutUwSA6EyvTZESgCt8A.png)Deposit \(Parent to Child\)

## Token Transfer on Child Chain <a id="b4a3"></a>

![](https://cdn-images-1.medium.com/max/1600/1*Q5iSW0VMv7oUEctVV-ojlA.png)

Fourth, let’s send some tokens from Bob to Tom on the **child chain**.

![](https://cdn-images-1.medium.com/max/1200/1*URodLDlvlalBNy3Yc02SOQ.png)

![](https://cdn-images-1.medium.com/max/1200/1*bYHmXubuQvc1qVgyTrpLEQ.png)1,000,000 units from Bon to Tom.

Bob has 5,000,000 tokens. He sent 1,000,000 tokens out of 5,000,000 to Tom.

## Exit Part1（Exit tokens from ChildWallet to ParentWallet.） <a id="9e58"></a>

![](https://cdn-images-1.medium.com/max/1600/1*JUYBXgDRCszWpoGfJtBC_Q.png)

Exit tokens from Tom’s account on the child chain to his account on the parent chain. If you type your account name, you can find UTXO lists you have. A child chain has all transaction histories you made and tokens are exited based on UTXO.

![](https://cdn-images-1.medium.com/max/2400/1*EM3sH1quRow-cDoXbrOjAA.png)

Press the ExitStart button so that you can exit your tokens to the parent chain.

![](https://cdn-images-1.medium.com/max/2400/1*N1Fqy4Gn_DcxC4hiBCSVcw.png)

**BUT**, you have to wait about 60 seconds. It is a Plasma challenge period which we decided. Full node holders can challenge the legitimacy of exits in it.

## Exit Part2（ExitFinalize ChildWallet to ParentWallet.） <a id="57bc"></a>

Click ExitFinalize.

![](https://cdn-images-1.medium.com/max/2400/1*QiId4iV9snnigqkhZJj-iw.png)

Then,

![](https://cdn-images-1.medium.com/max/1600/1*KRG-BHrLdbtJeaEzx5ACAA.png)Exit successful!! Awesome!!

Finally, the exit is successful. Well done!! This is a simple demo, but it’s one giant leap for the Polkadot/Substrate community!!

## Future Works <a id="3e6d"></a>

### ver0.2.0rc1 <a id="c8e9"></a>

Actually, we just have one node in this tutorial because we used balances SRML, the default setting. We will divide this node into a parent node and a child node using PlasmUtxo SRML.

### ver0.5.0 <a id="a68c"></a>

Connect our root chain to Polkadot Testnet.

### v0.7.0 <a id="d1e0"></a>

Plasma Cash implementation

### v1.0.0 <a id="3f94"></a>

Plasma Chamber implementation

### Another **Important** Task <a id="c62e"></a>

Improve ExitGame implementation

## Contact <a id="7da1"></a>

If you have a question, feel free to ping me on Twitter or on Riot \( @sate:matrix.org, @sota:matrix.org\)

