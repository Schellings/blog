---
title: 智能合约语言 Solidity 教程系列5 - 结构体与映射
date: 2019-07-20 19:56:24
tags: 
    - 智能合约
    - Solidity
---
<meta name="referrer" content="no-referrer" />

## 结构体(Structs)

Solidity提供struct来定义自定义类型，自定义的类型是引用类型。

```js
pragma solidity ^0.4.11;

contract CrowdFunding {
    // 定义一个包含两个成员的新类型
    struct Funder {
        address addr;
        uint amount;
    }

    struct Campaign {
        address beneficiary;
        uint fundingGoal;
        uint numFunders;
        uint amount;
        mapping (uint => Funder) funders;
    }

    uint numCampaigns;
    mapping (uint => Campaign) campaigns;

    function newCampaign(address beneficiary, uint goal) public returns (uint campaignID) {
        campaignID = numCampaigns++; // campaignID 作为一个变量返回
        // 创建一个结构体实例，存储在storage ，放入mapping里
        campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
    }

    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        // 用mapping对应项创建一个结构体引用
        // 也可以用 Funder(msg.sender, msg.value) 来初始化.
        c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
        c.amount += msg.value;
    }

    function checkGoalReached(uint campaignID) public returns (bool reached) {
        Campaign storage c = campaigns[campaignID];
        if (c.amount < c.fundingGoal)
            return false;
        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
```

上面是一个简化版的众筹合约，但它可以让我们理解structs的基础概念，struct可以用于映射和数组中作为元素。其本身也可以包含映射和数组等类型。

```js
注意：
不能声明一个 struct 同时将自身 struct 作为成员，这个限制是基于结构体的大小必须是有限的。
但struct可以作为mapping的值类型成员。
注意在函数中，将一个struct赋值给一个局部变量（默认是 storage 类型），实际是拷贝的引用，所以修改局部变量值的同时，会影响到原变量。
```

## 映射(Mappings)

映射类型，一种键值对的映射关系存储结构。定义方式为 mapping(_KeyType => _KeyValue)。
- 键类型允许除映射、变长数组、合约、枚举、结构体外的几乎所有类型。
- 值类型没有任何限制，可以为任何类型包括映射类型。

```js
映射可以被视作为一个哈希表，所有可能的键会被虚拟化的创建，映射到一个类型的默认值（二进制的全零表示）。在映射表中，并不存储键的数据，仅仅存储它的 keccak256 哈希值，这个哈希值在查找值时需要用到。
正因为此，映射是没有长度的，也没有键集合或值集合的概念。
```
映射的值类型也可以是映射，使用访问器访问时，要提供这个映射值所对应的键，不断重复这个过程。

```js
pragma solidity ^0.4.0;

contract MappingExample {
    mapping(address => uint) public balances;

    function update(uint newBalance) public {
        balances[msg.sender] = newBalance;
    }
}

contract MappingUser {
    function f() public returns (uint) {
        MappingExample m = new MappingExample();
        m.update(100);
        return m.balances(this);
    }
}
```

注意：映射并未提供迭代输出的方法，可以自行实现一个这样的数据结构。参考[iterable mapping](https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol)