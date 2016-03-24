---
layout: post
title:  "Ethereum in Practice - Quick Start Guide"
date:   2016-03-24
categories: ethereum
---

I've found [Ethereum documentation](http://ethdocs.org/en/latest/index.html) to be very thorough. And yet,
I've found it hard to understand how to approach Ethereum in practice - connect to the network, develop
and debug of a contract, deploy it and actually use it. This is what this post / guide / tutorial, and few coming posts, are about.
Straight and to the point.

This post certainly doesn't cover the theory which is already well explained in the official documentation, but
rather focuses on *how to get one up and running in, really, few minutes; because it is really that simple* when you know how.

And also when you know some theory as well. So, if you didn't read the official documentation, please, do so. This
post assumes that at the very least one knows
[what Ethereum is and how it works](http://ethdocs.org/en/latest/introduction/what-is-ethereum.html) and
[what an account, an EOA, a contract, gas and transactions are](http://ethdocs.org/en/latest/contracts-and-transactions/account-types-gas-and-transactions.html),
and finally would like to put it all to practice. There is a bare minimum of basic theory below otherwise.

Here, in this post, following subject are covered:

 - connecting to the network
 - creating account and getting Ether
 - transferring some Ether, just <s>for the fun of it</s> to burn some gas

And next post (coming soon!) will cover:

 - using private network to debug contracts
 - implementing a simple contract, deploy it to the network and use it from an external application
 - deployment of a contract to Homestad ("main" Ethereum network)

Enough words, let's do stuff!


*Contents*

* This line is a placeholder to generate the table of contents
{:toc}


# Install a client

> &#8505;&nbsp; *Ethereum network is decentralized. It means that there is no central server and what is called "ethereum
> client" (like `eth` or `geth` for example) is really a part of the network when it is online. The network is all
> clients altogether. You know BitTorrent, right? So this is kind of the same, but better.*

So, step #1 - install the client. Again, there is no single client, there are multiple to chose from. Ethereum is
described in a specification, and multiple clients exist all implementing it. But really, as of the time of this
writing, the most popular is `geth`, which is my choice as well, and which is the one used throughout this post in the
examples. Install instructions depend on the OS, so I'm just
[pointing to `geth` GitHub repository](https://github.com/ethereum/go-ethereum).

Once installed, make sure it works:

{% highlight bash %}
$ geth help
{% endhighlight %}

If you see the `geth` help - you are all set and ready to go!

Now, let's connect! Where though? `geth` can connect to any Ethereum network and there are really many: "main"
Ethereum network - currently Homestad, test network - Morden, old test network - Olympic (don't use that), or any
private network (you can also create your very own private network, yes).


# Connect to the testnet

I know, I know, you are anxious to connect to something real, not just a lonely one-node private network on your
computer. Well, on the other hand, you probably don't want to pay real money for those simple tests we are going to do
(read - do not buy [and spend] Ether). So let's connect to the Morden testnet. It is also quite lightweight (will take
less time... more on it below), and you can mine with CPU on it to earn some Ether (unlike Homestad, where you'd need
to mine with GPU and wait for quite lot before you earn some Ether). Don't worry, we will try other options later on
as well, and everything we will do in testnet applies exactly the same way to the Homestad. So, to connect to Morden:

{% highlight bash %}
$ geth --testnet --datadir ~/.ethereum-testnet
{% endhighlight %}

This starts a `geth` client - that is, it will attempt to connect to the test network (`--testnet`) and download the
blockchain from the network. Yes, all of it, entire blockchain. By the time of this writing it is about 2.5 GiB.
`--datadir` parameter is here just to put the local data storage (your copy of the blockchain, keystore [accounts]
and other local client data) into a dedicated directory, in order to separate it from other networks when we will
connect to them later on. You can actually avoid that last option, because `geth` will create a `testnet` subdirectory
anyway, but I just prefer to keep things explicitly separated.

> &#8505;&nbsp; *You can run `geth` with `--mine` option and `console` command at once, and redirect the log to the log
> file, if you are familiar with all this. Or, use the command exactly as above if you are not sure what other options will do, and
> follow the post to find out about them.*

Now, wait for the client to download the blockchain. With my computer and my network connection it took me about 30 or 40
minutes. You can track the progress in the log output, see the `imported` messages and block numbers to the end. Like this:

    I0324 18:13:21.624335    5650 blockchain.go:1251] imported 256 block(s) (0 queued 0 ignored) including 1023 txs in 3.460755062s. #1002380 [80d47304 / 75592e91]

You can find out the latest block number in one of the Ethereum online browsers, for example, like
[Etherscan for testnet](http://testnet.etherscan.io/). Note that there are many Ethereum browsers online, but not all
of them exist for testnet. This is why I prefer Etherscan - it exists for both Homestad and Morden. Anyway, by the time
of this writing, last block on testnet is #627190.

Once your client downloads the blockchain - you can use the network to the fullest! Like, create transactions! But wait,
to create transactions, one needs an account.

# Create an account

This is [well documented](http://ethdocs.org/en/latest/account-management.html#creating-an-account). Stop the client if
it is running and run:

{% highlight bash %}
$ geth --testnet --datadir ~/.ethereum-testnet account new
{% endhighlight %}

You know all the options here already, and `account new` is self-explanatory. Enter password when prompted; I don't
need to tell that this password and the keyfile generated into `~/.ethereum-testnet/testnet/keystore` directory are
strictly personal and are *the only way* to access your account, do I?

OK, all set! But wait a minute - if we want to create a transaction, we need some Ether - some Ether to transfer in a
transaction, and also some Ether to pay for gas.

How do you get Ether? It depends.

# Mine blocks and earn Ether on testnet

You can buy some Ether (but only for Homestad, you can't buy Ether on testnet), or mine blocks and get Ether as a mining reward
(which is relatively easy on testnet and much much harder on Homestad unless you have some real good GPU and a lot of
time). Now, as we continue with testnet, let's mine some. Let's run `geth` little differently this time:

{% highlight bash %}
$ geth --testnet --datadir "~/.ethereum-testnet" --mine console 2>> ~/.ethereum-testnet.log
{% endhighlight %}

OK, `--mine` option enables mining, `console` command launches an interactive console where you could issue commands
to the client, and finally we redirect the client log to a file or the console will be bloated with the log otherwise.

Let's see first, if we mined some blocks:

{% highlight bash %}
$ tail -f ~/.ethereum-testnet.log
{% endhighlight %}

Before your client can mine, it needs to generate [DAG](http://ethdocs.org/en/latest/mining.html#what-is-mining), so
you need to wait for it to be generated first. See those `Generating DAG: XX%` messages? And after it is generated ...

When you see a message like `🔨  Mined block (#654321 / ffeedd)` - it means you mined a block successfully. Although, it
doesn't mean it counts. Network can chose another, more "effective" blockchain branch, starting from another block, so
that's why you should `Wait 5 blocks for confirmation`. 5 blocks later, typically, you will see a message like
`🔨 🔗  Mined 5 blocks back: block #654321`. Congratulations! You mined a block and you just earned yourself 5 Ether on
the testnet!

# Check your account balance

Yes, Ether you just earned should be now on the account created above. (As a side note, know that if you have multiple
accounts, you can chose which one to use as the coinbase one - the one to transfer mining rewards to). Remember we ran
`geth` with `console` command? Go to the console and run:


{% highlight javascript %}
> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
{% endhighlight %}

This should now be at least 5 (or actually little more because you are also given Ether for the price of the gas used to
mine the block), or more if you mined more than 1 block by this time. Hey, you are virtually rich now!


# Transfer some Ether

OK, now that you are reach on Ether (in testnet), you can transfer some of it to another account. Which one? Hey, why
not create another account for yourself and transfer some funds between the two accounts? In the console:

{% highlight javascript %}
> personal.newAccount()
{% endhighlight %}

This will create a new account. The result is the same as if we'd do that with `geth account new` command.

Now, to transfer funds, let's first define some transaction elements:

{% highlight javascript %}
> var from = eth.accounts[0];
> var to   = eth.accounts[1];
> var amount = web3.toWei(1, "ether");
{% endhighlight %}

You can check balances before the transaction:

{% highlight javascript %}
> web3.fromWei(eth.getBalance(from))
> web3.fromWei(eth.getBalance(to))
{% endhighlight %}

And now make a transaction:

{% highlight javascript %}
> eth.sendTransaction({from: from, to: to, value: amount})
{% endhighlight %}

Oh, does it ask for a password now? Sure. You can check any account balance without the password, but if you want to make
a transaction - you need to *unlock* an account. Only unlocked accounts can create transactions. And, only the owner
can unlock it - you need both a key and a password to it. That's the point - otherwise you could have been able to send
a transaction from any account. For fun, you can find any account mentioned in some transaction on Etherscan, and try
checking its balance. Success! You can do that. Send a transaction from it? Certainly not.

There are other ways to unlock accounts, but for now - go on and enter your password. Now, sending account (`from`) is
unlocked for the entire session in this console. Note the output - it is the transaction number. Note it. Now, we need
to wait some time for the transaction to get into next mined block. Let's wait. Meanwhile...

You can monitor the blockchain in the console, but hey, remember you can also monitor the testnet on
[Etherscan](http://testnet.etherscan.io)? Why not give it a try now? We will do all the same in the console in a
moment, but let's go to Etherscan now and find your transaction in the
[list of transactions](http://testnet.etherscan.io/txs). You might need to wait some time for a network to mine a block
with your transaction. Okay... Now, that you saw your transaction, let's check try the same in the console. First,
let's see account balances:

{% highlight javascript %}
> web3.fromWei(eth.getBalance(from))
> web3.fromWei(eth.getBalance(to))
{% endhighlight %}

Did you note something special? Did you note that although `to` account now contains 1 Ether exactly, your `from`
account is now slightly more than 1 Ether less. You know why? Yes, gas! You also paid for a gas used in the transaction.
Let's see some transaction details:

{% highlight javascript %}
> eth.getTransaction("0x1786ffd39975833bf1ed2f09657d1585cb3f2cc73f60ac04277eef1")
{% endhighlight %}

Substitute the transaction number above by yours. What you will see are the transaction details. Among all, you will
also see the gas consumed and gas price applied by the network by the time the transaction was effective. Something like:

{% highlight javascript %}
{
  blockHash: "0x7bf06c68e9a9d3b1c8057fe44f18b4096b8f2908136e46e8f3cfc862",
  blockNumber: 654321,
  from: "0xd416324aa49f3a6db3e999ec823a7c38",
  gas: 90000,
  gasPrice: 20000000000,
  hash: "0x1786ffd39975833bf1ed2f09657d1585cb3f2cc73f60ac04277eef1",
  input: "0x",
  nonce: 48578,
  to: "0xd7e675831dd752e6e7e404124fc8004f",
  transactionIndex: 0,
  value: 1000000000000000000
}
{% endhighlight %}

> &#8505;&nbsp; *Note, that for this transaction you didn't define gas and gas price limitts, so "network defaults" were used for these,
> but you can define them explicitly for you transaction, in order to limit the cost in the case it runs too
> high and abondon a transaction. Which is typically a good idea actually.*

OK, that was fun! Now what?

# Run a private network, create a contract and deploy it to Homestad

After all, we are developers, we are here to develop stuff, not just use a command line to burn gas. So, we want to
develop and deploy a contract next. And, that's real easy.

So, these are the exact topics I will cover in the coming posts. This post grows little too big, so I prefer to take a
break and see you soon!

*Please*, let me know what do you think in the comments! Thank you for reading!
