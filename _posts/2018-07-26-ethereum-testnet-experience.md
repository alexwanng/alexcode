---
layout: post
title:  " 以太坊测试网络试验 "
date:   2018-07-26 23:55:18 +0800
categories: 以太坊 100days
typora-copy-images-to: ../assets/images/2018-07
typora-root-url: ../../blog_alexcode
---
内容来自《深入浅出ETH原理与智能合约》


* TOC
{:toc}




## 创建目录

mkdir -p ~/testnet/chaindata 



## 创建账户

```bash
geth account new --datadir ~/testnet/chaindata 
```

```bash
Address: {aaae4eae489903c2806f7f3e19e91bab424dc697}
```





## 清理db

```bash
geth removedb -datadir ~/testnet/chaindata 
```



## customgensis.json:

```json
{
"config": {
    "chainId": 201807286666, 
    "homesteadBlock": 0,
    "eip155Block": 0,
    "eip158Block": 0
},
"difficulty": "0x2000", 
"gasLimit": "0x2100000", 
"alloc": {
  "aaae4eae489903c2806f7f3e19e91bab424dc697": 
  { "balance": "0x8000000000000000000" }
}
}
```



## 初始化测试网络

```bash
geth --identity "AlexTestNode" --datadir ~/testnet/chaindata init ~/testnet/customgensis.json
```



## 运行测试网络：

```bash
geth --rpc --rpcaddr 0.0.0.0 --rpccorsdomain "*" --ws --wsaddr 0.0.0.0 --wsorigins "*" --datadir ~/testnet/chaindata --networkid 201807286666 --nodiscover
```



![](https://alextencentcos-1256436283.file.myqcloud.com/2018-07-26-025034.jpg)



## 登陆javascript console

```bash
geth attach ~/testnet/chaindata/geth.ipc
```



## 查询账户

```bash
> eth.accounts
["0xaaae4eae489903c2806f7f3e19e91bab424dc697", "0xc9268330593542cbc4e9bad3cb2214bbcd932f5a", "0x7e60765e5754b0e5da1b9693582c79d62c84dd1b"]
```



其中部分账户创建使用 personal.newAccount



## 使用变量绑定账户地址：

```js
addr1 = eth.accounts[0]
addr2 = eth.accounts[1]
addr3 = eth.accounts[2]
```



## 在开始之前先检查下各个账户有多少钱

```bash
> web3.fromWei(eth.getBalance(addr1), "ether")
37778.931862957161709568
> web3.fromWei(eth.getBalance(addr2), "ether")
0
> web3.fromWei(eth.getBalance(addr3), "ether")
0
>
```



## 将挖矿受益人设置为addr3

```bash
> miner.setEtherbase(addr3)
true
> eth.coinbase
"0x7e60765e5754b0e5da1b9693582c79d62c84dd1b"
> eth.blockNumber
0
>
```



## 交易之前， 解锁发起帐号

```bash
> personal.unlockAccount(addr1, "123456")
true
>

> txpool.status
{
  pending: 0,
  queued: 0
}
>
```



## 发起交易：

```bash
> eth.sendTransaction({from:addr1, to:addr2, value: web3.toWei(10.0, "ether")})
"0x6cd3bc9adadf42a8d3c090041a2cecaa328412b397c71dcc90383082e3ffaf9d"
>


> txpool.status
{
  pending: 1,
  queued: 0
}
>
```



## 开始挖矿

```bash
miner.start()
```





![](https://alextencentcos-1256436283.file.myqcloud.com/2018-07-26-141145.jpg)



![](https://alextencentcos-1256436283.file.myqcloud.com/2018-07-26-141314.jpg)



## 停止挖矿

```bash
miner.stop()


> eth.blockNumber
7

> txpool.status
{
  pending: 0,
  queued: 0
}
>


```



## 挖矿后查看接收人的余额

```bash
> web3.fromWei(eth.getBalance(addr2), "ether")
10
>
```



## 查看发起人的余额: (还有部分gas费用)

```bash
> web3.fromWei(eth.getBalance(addr1), "ether")
37768.931484957161709568
>
```



## 查看旷工的收益：

```bash
> web3.fromWei(eth.getBalance(addr3), "ether")
35.000378
>

```



这里35的收益是挖出了7个区块， 每个区块奖励5eth， 小数点后面的奖励来自交易费用

