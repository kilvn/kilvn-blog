---
title: "以太坊 Geth 的安装和使用，开启RPC接口"
slug: "eth-geth-install-and-use"
description: ""
date: "2018-05-01T15:24:59Z"
thumbnail: ""
categories:
  - "区块链"
tags:
  - "eth"
  - "geth"
  - "rpc"
  - "以太坊"
---
#### 钱包的安装

windows2008安装geth

[https://geth.ethereum.org/downloads/](https://geth.ethereum.org/downloads/) 下载window的.exe安装包，运行安装即可。

Ubuntu 安装geth

	sudo add-apt-repository -y ppa:ethereum/ethereum
	sudo apt-get update
	sudo apt-get install ethereum


#### 启动钱包同步并开启geth rpc接口

windows2008

	geth -fast -datadir "C:\Ethereum" –maxpeers 100 console 2>>geth.log –rpc –rpcapi "eth,personal" –rpcport 8545 –rpccorsdomain '"*"'

Ubuntu下

	geth –fast –datadir "/root/Gopath/eth_geth" –maxpeers 100 console 2>>"/root/Gopath/eth_geth/geth.log" –rpc –rpcapi "eth,personal,web3"

RPC接口说明

	–rpc –rpcapi "eth,personal,web3"  表示开启RPC接口，引号里面的表示开启的模块

连接到已经在运行的geth节点

	geth attach http://localhost:8545
 

#### geth钱包同步状态

启动geth的时候加了console命令，表示进入命令行模式，geth的运行日志保存在geth.log文件内

查看geth同步状态，通过syncing命令

	> eth.syncing
	false

返回false表示初次同步已完成，可以使用。

	> eth.syncing
	{
	currentBlock: 5530418,
	highestBlock: 5530484,
	knownStates: 130373961,
	pulledStates: 130373960,
	startingBlock: 5512117
	}

``currentBlock`` 表示当前已同步到的高度，可以通过在 [etherscan.io](https://etherscan.io/) 首页查看``LAST BLOCK``项最新高度，预估剩余同步需要的时间。

返回上面这样的对象，说明还没有同步完成，不能进行任何操作，包括查询钱包余额，转账等操作。

需要等待钱包同步完成，返回false才能使用，仅限于初次安装geth客户端。

同步完成后，以后只是递增同步。

geth 客户端需要长时间启动，否则节点得不到同步，交易平台也无法调用接口。


#### geth命令可以参考以下：

中文翻译版：https://blog.csdn.net/qq_28114645/article/details/78802041

英文官方版：https://github.com/ethereum/wiki/wiki/JSON-RPC

