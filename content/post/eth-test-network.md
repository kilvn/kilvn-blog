---
title: "以太坊创建测试区块和挖矿"
slug: "eth-test-network"
description: ""
date: "2018-05-01T15:38:24Z"
thumbnail: ""
categories:
  - "blockchain"
tags:
  - "eth"
  - "以太坊"
  - "挖矿"
---
新建genesis.json文件放到datadir目录下

	{
		"config": {
			"chainId": 15,
			"homesteadBlock": 0,
			"eip155Block": 0,
			"eip158Block": 0,
			"ByzantiumBlock": 0
		},
		"coinbase" : "0x0000000000000000000000000000000000000000",
		"difficulty" : "0x40000",
		"extraData" : "",
		"gasLimit" : "0xffffffff",
		"nonce" : "0x0000000000000042",
		"mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
		"parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
		"timestamp" : "0x00",
		"alloc": {}
	}

制造创世区块

	geth –datadir "./" init genesis.json

创建自己的私有链条

	geth –identity "kilvn-test" –datadir "./" –nodiscover console 2>>geth.log –rpc –rpcapi "db,eth,net,web3,personal"

查看帐户列表

	eth.accounts

新建一个帐户

	personal.newAccount("123456")

查看第一个帐户

	eth.coinbase

设置挖矿收益用户为第一个用户

	miner.setEtherbase(eth.coinbase)

启动矿机

	miner.start()

查看第一个帐户的余额

	eth.getBalance(eth.coinbase)
	eth.getBalance(eth.accounts[0])

查看区块高度

	eth.blockNumber

停止矿机

	miner.stop()

