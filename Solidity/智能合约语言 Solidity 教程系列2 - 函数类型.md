---
title: 智能合约语言 Solidity 教程系列2 - 函数类型
date: 2019-07-16 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 函数类型（Function Types）

函数也是一种类型，且属于值类型。
可以将一个函数赋值给一个函数类型的变量。还可以将一个函数作为参数进行传递。也可以在函数调用中返回一个函数。

函数类型有两类：内部(internal)和外部(external)函数

- 内部(internal)函数只能在当前合约内被调用（在当前的代码块内，包括内部库函数，和继承的函数中）。
- 外部(external)函数由地址和函数方法签名两部分组成，可作为外部函数调用的参数，或返回值。

函数类型定义如下：

```js
function (<parameter types>) {internal|external} [pure|constant|view|payable] [returns (<return types>)]
```

- 如果函数不需要返回，则省去 returns ()
- 函数类型默认是 internal， 因此 internal 可以省去。
- 合约中函数本身默认是 public 的， 仅仅是当作类型名使用时默认是 internal 的。
- 有两个方式访问函数，一种是直接用函数名f, 一种是this.f， 前者用于内部函数，后者用于外部函数。
- 如果一个函数变量没有初始化，直接调用它将会产生异常。如果 delete 了一个函数后调用，也会发生同样的异常。

## 函数可见性

```js
public - 任意访问
private - 仅当前合约内
internal - 仅当前合约及所继承的合约
external - 仅外部访问（在内部也只能用外部访问方式访问）
```
## 最佳实践

先上一个例子看 public 与 external 不同，代码如下：

```js
pragma solidity^0.4.18;

contract Test {
    uint[10] x = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    
    function test(uint[10] a) public returns (uint){
         return a[9]*2;
    }

    function test2(uint[10] a) external returns (uint){
         return a[9]*2;
    }
    
    function calltest() {
        test(x);
    }
  
    function calltest2() {
        this.test2(x);
        //test2(x);  //不能在内部调用一个外部函数，会报编译错误。
    }  
    
}

```

我们分别调用 test 及 test2 ，对比执行花费的 gas。

![](https://img.learnblockchain.cn/2017/test_func.jpg!wl)
![](https://img.learnblockchain.cn/2017/test_func2.jpg!wl)

可以看到调用 pubic 函数花销更大，这是为什么呢？

当使用 public 函数时，Solidity会立即复制数组参数数据到内存， 而 external 函数则是从 calldata 读取，而分配内存开销比直接从 calldata 读取要大的多。

那为什么 public 函数要复制数组参数数据到内存呢？是因为 public 函数可能会被内部调用，而内部调用数组的参数是当做指向一块内存的指针。

对于 external 函数不允许内部调用，它直接从calldata读取数据，省去了复制的过程。

```js
如果确认一个函数仅仅在外部访问，请用external。
当需要内部调用的时候，请用public
```
