---
title: 一致性Hash原理
date: 2020-09-18 19:56:24
tags: 
    - 分布式
---

## 前言
互联网公司中，绝大部分都没有马爸爸系列的公司那样财大气粗，他们即没有强劲的服务器、也没有钱去购买昂贵的海量数据库。那他们是怎么应对大数据量高并发的业务场景的呢？  
这个和当前的开源技术、海量数据架构都有着不可分割的关系。比如通过mysql、nginx等开源软件，通过架构和低成本的服务器搭建千万级别的用户访问系统。  
怎么样搭建一个好的系统架构，这个话题我们能聊上个七天七夜。这里我主要结合Redis集群来讲一下一致性Hash的相关问题。

## Redis集群的使用

我们在使用Redis的过程中，为了保证Redis的高可用，我们一般会对Redis做主从复制，组成Master-Master或者Master-Slave的形式，进行数据的读写分离

![](https://upload-images.jianshu.io/upload_images/6555006-d7be3c1271b96a7e.png?imageMogr2/auto-orient/strip|imageView2/2/w/701/format/webp)

**当缓存数据量超过一定的数量时，我们就要对Redis集群做分库分表的操作。**

来个栗子，我们有一个电商平台，需要使用Redis存储商品的图片资源，存储的格式为键值对，key值为图片名称，Value为该图片所在的文件服务器的路径，我们需要根据文件名，查找到文件所在的文件服务器上的路径，我们的图片数量大概在3000w左右，按照我们的规则进行分库，规则就是随机分配的，我们以每台服务器存500w的数量，部署12台缓存服务器，并且进行主从复制，架构图如下图1-2所示：

![](https://upload-images.jianshu.io/upload_images/6555006-a6505383af3d36d1.png?imageMogr2/auto-orient/strip|imageView2/2/w/688/format/webp)

由于我们定义的规则是随机的，所以我们的数据有可能存储在任何一组Redis中，比如我们需要查询"product.png"的图片，由于规则的随机性，我们需要遍历所有Redis服务器，才能查询得到。这样的结果显然不是我们所需要的。

所以我们会想到按某一个字段值进行Hash值、取模。所以我们就看看使用Hash的方式是怎么进行的。

## 使用Hash的Redis集群

如果我们使用Hash的方式，每一张图片在进行分库的时候都可以定位到特定的服务器，示意图如图1-3所示：

![](https://upload-images.jianshu.io/upload_images/6555006-7899a430dc3142a8.png?imageMogr2/auto-orient/strip|imageView2/2/w/708/format/webp)

从上图中，我们需要查询的是图product.png，由于我们有6台主服务器，所以计算的公式为：hash(product.png) % 6 = 5, 我们就可以定位到是5号主从，这们就省去了遍历所有服务器的时间，从而大大提升了性能。

## 使用Hash时遇到的问题

在上述hash取模的过程中，我们虽然不需要对所有Redis服务器进行遍历而提升了性能。但是，使用Hash算法缓存时会出现一些问题:

**Redis服务器变动时，所有缓存的位置都会发生改变。**

比如，现在我们的Redis缓存服务器增加到了8台，我们计算的公式从hash(product.png) % 6 = 5变成了hash(product.png) % 8 = ? 结果肯定不是原来的5了。  
再者，6台的服务器集群中，当某个主从群出现故障时，无法进行缓存，那我们需要把故障机器移除，所以取模数又会从6变成了5。我们计算的公式也会变化。

由于上面hash算法是使用取模来进行缓存的，为了规避上述情况，Hash一致性算法就诞生了

## 一致性Hash算法原理

一致性Hash算法也是使用取模的方法，不过，上述的取模方法是对服务器的数量进行取模，而一致性的Hash算法是对**2的32方取模**。

**即，一致性Hash算法将整个Hash空间组织成一个虚拟的圆环，Hash函数的值空间为0 ~ 2^32 - 1(一个32位无符号整型)** 整个哈希环如下：

![](https://upload-images.jianshu.io/upload_images/6555006-1f81e81466729c6b.png?imageMogr2/auto-orient/strip|imageView2/2/w/830/format/webp)

整个圆环以**顺时针方向组织**，圆环正上方的点代表0，0点右侧的第一个点代表1，以此类推。  
第二步，我们将各个服务器使用Hash进行一个哈希，具体可以选择**服务器的IP或主机名作为关键字**进行哈希，这样每台服务器就确定在了哈希环的一个位置上，比如我们有三台机器，使用IP地址哈希后在环空间的位置如图1-4所示：

![](https://upload-images.jianshu.io/upload_images/6555006-1f100c1012b06b40.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

现在，我们使用以下算法定位数据访问到相应的服务器：

>将数据Key使用相同的函数Hash计算出哈希值，并确定此数据在环上的位置，从此位置沿环顺时针查找，遇到的服务器就是其应该定位到的服务器。

例如，现在有ObjectA，ObjectB，ObjectC三个数据对象，经过哈希计算后，在环空间上的位置如下：

![](https://upload-images.jianshu.io/upload_images/6555006-defb48ae9714580d.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

根据一致性算法，Object -> NodeA，ObjectB -> NodeB, ObjectC -> NodeC

## 一致性Hash算法的容错性和可扩展性

现在，假设我们的Node C宕机了，我们从图中可以看到，A、B不会受到影响，只有Object C对象被重新定位到Node A。所以我们发现，在一致性Hash算法中，如果一台服务器不可用，受影响的数据仅仅是此服务器到其环空间前一台服务器之间的数据（这里为Node C到Node B之间的数据），其他不会受到影响。如图1-6所示：


![](https://upload-images.jianshu.io/upload_images/6555006-cd54d5c30e9cad6f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

另外一种情况，现在我们系统增加了一台服务器Node X，如图1-7所示：

![](https://upload-images.jianshu.io/upload_images/6555006-8f61754de37eb380.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

此时对象ObjectA、ObjectB没有受到影响，只有Object C重新定位到了新的节点X上。

如上所述：**一致性Hash算法对于节点的增减都只需重定位环空间中的一小部分数据，有很好的容错性和可扩展性。**

## 数据倾斜问题

在一致性Hash算法服务节点太少的情况下，容易因为节点分布不均匀面造成数据倾斜（**被缓存的对象大部分缓存在某一台服务器上**）问题，如图1-8特例：

![](https://upload-images.jianshu.io/upload_images/6555006-c504a13cbe34e617.png?imageMogr2/auto-orient/strip|imageView2/2/w/806/format/webp)

这时我们发现有大量数据集中在节点A上，而节点B只有少量数据。为了解决数据倾斜问题，一致性Hash算法引入了虚拟节点机制，即对每一个服务器节点计算多个哈希，每个计算结果位置都放置一个此服务节点，称为虚拟节点。

具体操作可以为服务器IP或主机名后加入编号来实现

![](https://upload-images.jianshu.io/upload_images/6555006-f15ec4f10a433beb.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

数据定位算法不变，只需要增加一步：虚拟节点到实际点的映射。所以加入虚拟节点之后，即使在服务节点很少的情况下，也能做到数据的均匀分布。





