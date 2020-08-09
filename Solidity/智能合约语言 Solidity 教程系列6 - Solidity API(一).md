---
title: 智能合约语言 Solidity 教程系列6 - Solidity API(一)
date: 2019-07-27 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 区块和交易的属性

用来提供一些区块链当前的信息。

```js
blockhash(uint blockNumber) returns (bytes32)：返回给定区块号的哈希值，只支持最近 256 个区块，且不包含当前区块。
block.coinbase (address): 当前块矿工的地址。
block.difficulty (uint):当前块的难度。
block.gaslimit (uint):当前块的 gaslimit。
block.number (uint):当前区块的块号。
block.timestamp (uint): 当前块的 Unix 时间戳（从 1970/1/1 00:00:00 UTC 开始所经过的秒数）
gasleft() (uint256): 获取剩余 gas。
msg.data (bytes): 完整的调用数据（calldata）。
msg.gas (uint): 当前还剩的 gas（弃用）。
msg.sender (address): 当前调用发起人的地址。
msg.sig (bytes4):调用数据(calldata)的前四个字节（例如为：函数标识符）。
msg.value (uint): 这个消息所附带的以太币，单位为 wei。
now (uint): 当前块的时间戳(block.timestamp 的别名)
tx.gasprice (uint) : 交易的 gas 价格。
tx.origin (address): 交易的发送者（全调用链）
```

注意：
msg 的所有成员值，如 msg.sender,msg.value 的值可以因为每一次外部函数调用，或库函数调用发生变化（因为 msg 就是和调用相关的全局变量）。

不应该依据 block.timestamp, now 和 block.blockhash 来产生一个随机数（除非你确实需要这样做），这几个值在一定程度上被矿工影响（比如在赌博合约里，不诚实的矿工可能会重试去选择一个对自己有利的 hash）。

对于同一个链上连续的区块来说，当前区块的时间戳（timestamp）总是会大于上一个区块的时间戳。

为了可扩展性的原因，你只能查最近 256 个块，所有其它的将返回 0.

## ABI 编码函数

solidity 提供了一下函数，用来直接得到 ABI 编码信息，这些函数有：
```js
* abi.encode(...) returns (bytes)：计算参数的 ABI 编码。
* abi.encodePacked(...) returns (bytes)：计算参数的紧密打包编码
* abi. encodeWithSelector(bytes4 selector, ...) returns (bytes)： 计算函数选择器和参数的 ABI 编码
* abi.encodeWithSignature(string signature, ...) returns (bytes): 等价于* 
* abi.encodeWithSelector(bytes4(keccak256(signature), ...)
```

通过 ABI 编码函数可以在不用调用函数的情况下，获得 ABI 编码值，下面通过一段代码来看看这些方式的使用：

```js
pragma solidity ^0.4.24;

contract testABI {
    function abiEncode() public constant returns (bytes) {
        abi.encode(1);  // 计算 1 的ABI编码
        return abi.encodeWithSignature("set(uint256)", 1); //计算函数set(uint256) 及参数1 的ABI 编码
    }
}
```

## 错误处理

```js
assert(bool condition)
用于判断内部错误，条件不满足时抛出异常
require(bool condition):
用于判断输入或外部组件错误，条件不满足时抛出异常
require(bool condition, string message)
同上，多了一个错误信息。
revert():
终止执行并还原改变的状态
revert(string reason)
同上，提供一个错误信息。
```

之前老的错误处理方式用 throw 及 if ... throw，这种方式会消耗掉所有剩余的 gas。目前 throw 的方式已经被弃用。

