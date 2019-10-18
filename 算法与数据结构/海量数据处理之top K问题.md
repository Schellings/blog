---
title: 海量数据处理之top K问题
date: 2019-01-15 16:56:24
tags: 算法与数据结构
---

## 前言

topK问题是一个经典的海量数据处理问题，比如热搜上每天都会更新出排行前10的热门搜索信息，再或者通过大数据找出陕西省人最爱吃的水果等，都可以使用topK问题来解决，其核心思想就是最小堆的引入。

## topK问题分析
在海量数据中找出出现频率最高的前k个数，或者从海量数据中找出最大的前k个数，这类问题通常被称为top K问题。

下面我们通过一个简单的示例来说明：假如面试官给你100W个数据，请找出其中最大的前K个数，并且现在仅仅有1M的空间？
在32位操作系统中，默认一个字为4字节，则有以下运算：

```yaml
NeedSize = 100w * 4 / 1024 / 1024 = 4 M
```
计算结果约等于4M，很显然1M的空间根本不够。也就是说，即使用最复杂的方法排序你也无法找到一个合适的空间来存储，因此引入了**最小堆**的数据结构。

当然假如这道题不再限制空间的大小，你会如何解决？可能不少人会说排序啊，下面我给大家证明一下时间复杂度：

```yaml
设在n个数中找出最大的前k个数(n远大于k)
 
1>排序的时间复杂度：O（n^2）
 
2>最小堆的时间复杂度：
 
         ①建堆：klogk （logk是以2为底）
 
         ②比较+调整：（n-k）* logk 
 
若：klogk + (n-k)*logk > n^2,则排序的时间复杂度低.
 
即：k > 2^n.
因为：k远小于n
所以：此情况不存在,最小堆的时间复杂度最优.
```

## 解决思路

- ① 定义两个数组，arr用于存储海量数据，top用于存储最小堆。
- ② 将海量数据的前k个元素先填满top堆。
- ③ 调整top堆为最小堆结构。
- ④ 通过遍历将新数据与堆顶元素（此时堆顶元素最小）比较，大于堆顶元素就入堆，并下调堆结构。
- ⑤ 遍历结束，则堆中的元素即n个数中最大的前k个数。

## 编码实现

### 最小堆实现

```java
public class MinHeap  
{  
    // 堆的存储结构 - 数组  
    private int[] data;  
      
    // 将一个数组传入构造方法，并转换成一个小根堆  
    public MinHeap(int[] data)  
    {  
        this.data = data;  
        buildHeap();  
    }  
      
    // 将数组转换成最小堆  
    private void buildHeap()  
    {  
        // 完全二叉树只有数组下标小于或等于 (data.length) / 2 - 1 的元素有孩子结点，遍历这些结点。  
        // *比如上面的图中，数组有10个元素， (data.length) / 2 - 1的值为4，a[4]有孩子结点，但a[5]没有*  
        for (int i = (data.length) / 2 - 1; i >= 0; i--)   
        {  
            // 对有孩子结点的元素heapify  
            heapify(i);  
        }  
    }  
      
    private void heapify(int i)  
    {  
        // 获取左右结点的数组下标  
        int l = left(i);    
        int r = right(i);  
          
        // 这是一个临时变量，表示 跟结点、左结点、右结点中最小的值的结点的下标  
        int smallest = i;  
          
        // 存在左结点，且左结点的值小于根结点的值  
        if (l < data.length && data[l] < data[i])    
            smallest = l;    
          
        // 存在右结点，且右结点的值小于以上比较的较小值  
        if (r < data.length && data[r] < data[smallest])    
            smallest = r;    
          
        // 左右结点的值都大于根节点，直接return，不做任何操作  
        if (i == smallest)    
            return;    
          
        // 交换根节点和左右结点中最小的那个值，把根节点的值替换下去  
        swap(i, smallest);  
          
        // 由于替换后左右子树会被影响，所以要对受影响的子树再进行heapify  
        heapify(smallest);  
    }  
      
    // 获取右结点的数组下标  
    private int right(int i)  
    {    
        return (i + 1) << 1;    
    }     
  
    // 获取左结点的数组下标  
    private int left(int i)   
    {    
        return ((i + 1) << 1) - 1;    
    }  
      
    // 交换元素位置  
    private void swap(int i, int j)   
    {    
        int tmp = data[i];    
        data[i] = data[j];    
        data[j] = tmp;    
    }  
      
    // 获取对中的最小的元素，根元素  
    public int getRoot()  
    {  
            return data[0];  
    }  
  
    // 替换根元素，并重新heapify  
    public void setRoot(int root)  
    {  
        data[0] = root;  
        heapify(0);  
    }  
}  
```

### 利用最小堆获取TopK

```java
public class TopK  
{  
    public static void main(String[] args)  
    {  
        // 源数据  
        int[] data = {56,275,12,6,45,478,41,1236,456,12,546,45};  
          
// 获取Top5  
        int[] top5 = topK(data, 5);  
          
        for(int i=0;i<5;i++)  
        {  
            System.out.println(top5[i]);  
        }  
    }  
      
    // 从data数组中获取最大的k个数  
    private static int[] topK(int[] data,int k)  
    {  
        // 先取K个元素放入一个数组topk中  
        int[] topk = new int[k];   
        for(int i = 0;i< k;i++)  
        {  
            topk[i] = data[i];  
        }  
          
        // 转换成最小堆  
        MinHeap heap = new MinHeap(topk);  
          
        // 从k开始，遍历data  
        for(int i= k;i<data.length;i++)  
        {  
            int root = heap.getRoot();  
              
            // 当数据大于堆中最小的数（根节点）时，替换堆中的根节点，再转换成堆  
            if(data[i] > root)  
            {  
                heap.setRoot(data[i]);  
            }  
        }  
        return topk;  
}  
}  
```

