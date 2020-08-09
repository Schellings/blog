---
title: 智能合约语言 Solidity 教程系列4 - 数组
date: 2019-07-20 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 数组（Arrays）

数组可以声明时指定长度，也可以是动态变长。

一个元素类型为 T，固定长度为 k 的数组，可以声明为T[k]，而一个动态大小（变长）的数组则声明为T[]。

还可以声明一个多维数组，如声明一个类型为 uint 的数组长度为 5 的变长数组（5 个元素都是变长数组），可以声明为 uint[][5]。（注意，相比非区块链语言，多维数组的长度声明是反的。）

要访问第三个动态数组的第二个元素，使用 x[2][1]。数组的序号是从 0 开始的，序号顺序与定义相反。

bytes和string是一种特殊的数组。bytes类似byte[]，但在外部函数作为参数调用中，bytes会进行压缩打包。string类似bytes，但不提供长度和按序号的访问方式（目前）。

所以应该尽量使用bytes而不是byte[]。

```js
可以将字符串 s 通过 bytes(s)转为一个 bytes，可以通过bytes(s).length获取长度，bytes(s)[n]获取对应的 UTF-8 编码。通过下标访问获取到的不是对应字符，而是 UTF-8 编码，比如中文编码是多字节，变长的，所以下标访问到的只是其中的一个编码。
类型为数组的状态变量，可以标记为public，从而让Solidity创建一个访问器，如果要访问数组的某个元素，指定数字下标就好了。（稍后代码事例）
```
## 创建内存数组

可使用 new 关键字创建一个 memory 的数组。与 stroage 数组不同的是，你不能通过.length 的长度来修改数组大小属性。我们来看看下面的例子：

```js
pragma solidity ^0.4.16;

contract C {
    function f(uint len) public pure {
        uint[] memory a = new uint[](7);
                
        //a.length = 100;  // 错误
        bytes memory b = new bytes(len);
        // Here we have a.length == 7 and b.length == len
        a[6] = 8;
    }
}
```

## 数组常量及内联数组

数组常量，是一个数组表达式（还没有赋值到变量）。下面是一个简单的例子：

```js
pragma solidity ^0.4.16;

contract C {
    function f() public pure {
        g([uint(1), 2, 3]);
    }
   // pure纯函数关键字
    function g(uint[3] _data) public pure {
        // ...
    }
}
```

通过数组常量，创建的数组是 memory 的，同时还是定长的。元素类型则是使用刚好能存储的元素的能用类型，比如[1, 2, 3]，只需要 uint8 即可存储，它的类型是uint8[3] memory。

由于 g()方法的参数需要的是 uint（默认的 uint 表示的其实是 uint256），所以需要对第一个元素进行类型转换，使用 uint(1)来进行这个转换。

## 成员

### length 属性

数组有一个.length 属性，表示当前的数组长度。storage 的变长数组，可以通过给。length 赋值调整数组长度。memory 的变长数组不支持。

不能通过访问超出当前数组的长度的方式，来自动实现改变数组长度。memory 数组虽然可以通过参数，灵活指定大小，但一旦创建，大小不可调整。

### push 方法

storage 的变长数组和 bytes 都有一个push方法（string 没有），用于附加新元素到数据末端，返回值为新的长度。