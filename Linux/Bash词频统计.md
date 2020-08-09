---
title: Bash词频统计
date: 2020-02-21 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 问题背景
写一个 bash 脚本以统计一个文本文件 words.txt 中每个单词出现的频率。

为了简单起见，你可以假设：

- words.txt只包括小写字母和 ' ' 。
- 每个单词只由小写字母组成。
- 单词间由一个或多个空格字符分隔。

示例:

假设 words.txt 内容如下：
```yaml
the day is sunny the the
the sunny is is
```
你的脚本应当输出（以词频降序排列）：
```yaml
the 4
is 3
sunny 2
day 1
```

说明:

- 不要担心词频相同的单词的排序问题，每个单词出现的频率都是唯一的。
- 你可以使用一行 Unix pipes 实现吗？

## 解决方法

```shell script
cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -r | awk '{print $2" "$1}'
```

```yaml
cat ——浏览文件
tr -s ——替换字符串（空格换为换行）保证了一行一个单词
sort ——默认ASCII值排序，排序号后还会有重复
uniq —— 去重，-c再输出重复次数。结果就是 ”4 abc“ abc出现了4次
sort -r —— 反向排序，也就是从大到小。得到按频率高低的结果
awk ——格式化输出，规定输出是先字符串再重复次数，所以先$2再$1，中间空格分隔
```
```yaml
注意：当重复的行并不相邻时，uniq 命令是不起作用的，即若文件内容为以下时，uniq 命令不起作用，所以此处在执行uniq命令之前，首先先进行了一次sort，保证相同的词相邻
```