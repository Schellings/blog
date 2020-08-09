---
title: 智能合约语言 Solidity 教程系列3 - 数据存储位置
date: 2019-07-18 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 数据位置(Data location)

在系列第一篇，我们提到 Solidity 类型分为两类：值类型(Value Type) 及 引用类型(Reference Types)。  
前面我们已经介绍完了值类型，接下来会介绍引用类型。

引用类型是一个复杂类型，占用的空间通常超过 256 位， 拷贝时开销很大，因此我们需要考虑将它们存储在什么位置，是memory（内存中，数据不是永久存在）还是storage（永久存储在区块链中）

所有的复杂类型如数组(arrays)和结构体(struct)有一个额外的属性：数据的存储位置（data location）。可为memory和storage。

根据上下文的不同，大多数时候数据位置有默认值，也通过指定关键字 storage 和 memory 修改它。

- 函数参数（包含返回的参数）默认是memory。
- 局部复杂类型变量（local variables）和 状态变量（state variables） 默认是storage。

```js
局部变量：局部作用域（越过作用域即不可被访问，等待被回收）的变量，如函数内的变量。状态变量：合约内声明的公有变量
```
还有一个存储位置是：calldata，用来存储函数参数，是只读的，不会永久存储的一个数据位置。外部函数的参数（不包括返回参数）被强制指定为 calldata。效果与 memory 差不多。

## 赋值行为
```js
数据位置指定非常重要，因为他们影响着赋值行为。
```

- 在 memory 和 storage 之间或与状态变量之间相互赋值，总是会创建一个完全独立的拷贝。
- 而将一个 storage 的状态变量，赋值给一个 storage 的局部变量，是通过引用传递。所以对于局部变量的修改，同时修改关联的状态变量。
- 另一方面，将一个 memory 的引用类型赋值给另一个 memory 的引用，不会创建拷贝（即：memory 之间是引用传递）。

```js
注意：不能将 memory 赋值给局部变量。
对于值类型，总是会进行拷贝。
```

下面看一段代码：

```js
pragma solidity ^0.4.0;

contract C {
    uint[] x; //  x的存储位置是storage

    // memoryArray的存储位置是 memory
    function f(uint[] memoryArray) public {
        x = memoryArray;    // 从 memory 复制到 storage
        var y = x;          // storage 引用传递局部变量y（y 是一个 storage 引用）
        y[7];               // 返回第8个元素
        y.length = 2;       // x同样会被修改
        delete x;           // y同样会被修改

        // 错误， 不能将memory赋值给局部变量
        // y = memoryArray;  

        // 错误，不能通过引用销毁storage
        // delete y;        

        g(x);               // 引用传递， g可以改变x的内容
        h(x);               // 拷贝到memory， h无法改变x的内容
    }

    function g(uint[] storage storageArray) internal {}
    function h(uint[] memoryArray) public {}
}
```

## 总结

### 强制的数据位置(Forced data location)
- 外部函数(External function)的参数(不包括返回参数)强制为：calldata
- 状态变量(State variables)强制为： storage
### 默认数据位置（Default data location）
- 函数参数及返回参数：memory
- 复杂类型的局部变量：storage

## 关于栈（stack）

EVM 是一个基于栈的语言，栈实际是在内存(memory)的一个数据结构，每个栈元素占为 256 位，栈最大长度为 1024。
值类型的局部变量是存储在栈上。

## 不同存储的消耗（gas 消耗）

```js
storage 会永久保存合约状态变量，开销最大
memory 仅保存临时变量，函数调用之后释放，开销很小
stack 保存很小的局部变量，几乎免费使用，但有数量限制。
```