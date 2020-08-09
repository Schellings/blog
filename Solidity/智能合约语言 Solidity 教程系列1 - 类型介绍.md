---
title: 智能合约语言 Solidity 教程系列1 - 类型介绍
date: 2019-07-15 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 写在前面

Solidity 是以太坊智能合约编程语言，阅读本文前，你应该对以太坊、智能合约有所了解。
如果你还不了解，建议你先看以太坊是什么、智能合约以及环境搭建

## 类型

Solidity是一种静态类型语言，意味着每个变量（本地或状态变量）需要在编译时指定变量的类型（或至少可以推倒出类型）。

Solidity 提供了一些基本类型可以用来组合成复杂类型。

Solidity 类型分为两类：

- 值类型(Value Type) - 变量在赋值或传参时，总是进行值拷贝。
- 引用类型(Reference Types)

### 值类型(Value Type)

值类型包含：

- 布尔类型(Booleans) 关键字 bool
- 整型(Integers) 关键字 int/uint 分为8-256字节（8为步长），例如16字节的int 标记为int16，其中int/uint默认为256
```js
整数常量除法，在早期的版本中是被截断的，但现在可以被转为有理数了，如 5/2 的值为 2.5
```
- 定长浮点型(Fixed Point Numbers)
- 定长字节数组(Fixed-size byte arrays) bytes1、bytes2...bytes32以1为步长
- 有理数和整型常量(Rational and Integer Literals) 
```js
整型常量是有一系列 0-9 的数字组成，10 进制表示
10 进制小数常量（Decimal fraction literals）带了一个. ,且在小数点左右两侧至少要存在一位数字，即1. .1 1.0都是合法的小数常量
科学符号也支持，基数可以是小数，指数必须是整数， 有效的表示如： 2e10, -2e10, 2e-10, 2.5e1。
数字常量表达式本身支持任意精度，也就是可以不会运算溢出，或除法截断。但当它被转换成对应的非常量类型，或者将他们与非常量进行运算，则不能保证精度了。
```
- 字符串常量（String literals）
```js
字符串常量是指由单引号，或双引号引起来的字符串 ("foo" or 'bar')。字符串并不像 C 语言，包含结束符，"foo"这个字符串大小仅为三个字节。和整数常量一样，字符串的长度类型可以是变长的。
```
- 十六进制常量（Hexadecimal literals）
```js
十六进制常量，以关键字 hex 打头，后面紧跟用单或双引号包裹的字符串，内容是十六进制字符串，如 hex"001122ff"
```
- 枚举(Enums)

在 Solidity 中，枚举可以用来自定义类型。它可以显示的转换与整数进行转换，但不能进行隐式转换。显示的转换会在运行时检查数值范围，如果不匹配，将会引起异常。枚举类型应至少有一名成员。

```js
pragma solidity ^0.4.0;

contract test {
    enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
    ActionChoices choice;
    ActionChoices constant defaultChoice = ActionChoices.GoStraight;

    function setGoStraight() {
        choice = ActionChoices.GoRight;
    }

    function getChoice() returns (ActionChoices) {
        return choice;
    }

    function getDefaultChoice() returns (uint) {
        return uint(defaultChoice);
    }
}
```

- 函数类型(Function Types)

- 地址类型(Address)

地址类型address是一个值类型

**balance** 属性及 **transfer()** 函数   
balance 用来查询账户余额，transfer()用来发送以太币（以 wei 为单位）。
```js
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

```js
注：如果 x 是合约地址，合约的回退函数（fallback 函数）会随transfer调用一起执行（这个是 EVM 特性），如果因gas耗光或其他原因失败，转移交易会还原并且合约会抛异常停止。
```

```js
关于回退函数（fallback 函数），简单来说它是合约中无函数名函数，每一个合约里面只能包含一个回退函数，当合约执行异常时，回退函数被执行，因此因该尽量保证逻辑简单，防止过多消耗gas
```

**send()** 函数  
send 与 transfer 对应，但更底层。如果执行失败，transfer 不会因异常停止，而 send 会返回 false。

关于函数的部分，这里不过多赘述，后面会针对一些常用的函数做梳理，包括构造函数，回退函数，纯函数，视图函数，函数访问性等

- 地址常量(Address Literals)

一个能通过地址合法性检查（address checksum test）十六进制常量就会被认为是地址，如 0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF。  
而不能通过地址合法性检查的 39 到 41 位长的十六进制常量，会提示一个警告，被视为普通的有理数常量。


地址合法性检查定义在[EIP-55](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)

### 引用类型

- 变长字节数组（Dynamically-sized byte array） 

bytes:动态分配大小字节数组， 参见[Arrays](http://solidity.readthedocs.io/en/latest/types.html#arrays),不是值类型！
string:动态分配大小 UTF8 编码的字符类型，参看[Arrays](http://solidity.readthedocs.io/en/latest/types.html#arrays)。不是值类型！

```js
bytes 用来存储任意长度的字节数据，string 用来存储任意长度的(UTF-8 编码)的字符串数据。
如果长度可以确定，尽量使用定长的如 byte1 到 byte32 中的一个，因为这样更省空间。
```

- 结构体(Structs)
```js
  // 定义一个包含两个成员的新类型
    struct Funder {
        address addr;
        uint amount;
    }
```
通过结构体组合现有的一些类型，来完成自定义类型的创建。

Solidity的类型详情查看官方文档：[Solidity类型](https://solidity.readthedocs.io/en/latest/types.html)
