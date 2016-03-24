---
layout: post
title:  "Ethereum in Practice - Quick Start Guide - Part 2"
date:   2016-03-24
categories: ethereum
hidden: 1
---

> THIS POST IS A WORK IN PROGRESS

# Run a private network

TODO


# Homestad

Well, you know everything now about connecting to the network. Connecting to the Homestad is really not that different
form how we did that for testnet.

*I actually suggest that you skip this section for the moment and continue with the sample transactions and contracts on
testnet or your own private network, until you think you are ready and you actually want to use the Homestad. I still
give these instructions here for the complete overview though.*

One trick, if you don't plan to mine blocks in Homestad, and you normally won't if you only plan to develop
contracts and use it, you can avoid downloading the entire blockchain - you will use less diskspace, and it will be
much faster. So:

    $ geth --fast console 2>> ~/.ethereum.log

You know what this does, except that this `--fast` option, as you guessed, is what is mentioned above - it avoids
downloading entire blockchain but only the data necessary for a client to use the network, but not to mine.

Then, again, create an account. Then, get some Ether on it. Wait, this is little different now. Again, you can mine, or
buy Ether. See the list of Ether
[marketplaces](http://ethdocs.org/en/latest/ether.html#list-of-centralised-exchange-marketplaces) or
[fixed rate exchanges](http://ethdocs.org/en/latest/ether.html#centralised-fixed-rate-exchanges) to buy Ether or
exchange Bitcoins for it, for example.

TODO: note about buying for the new account
