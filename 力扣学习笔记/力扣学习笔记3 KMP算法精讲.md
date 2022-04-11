---
title: 学习笔记三 KMP算法介绍
date: 2022-02-18
updated:
tags: 
- 力扣
- KMP
- 算法
categories: 力扣学习笔记
excerpt: KMP算法是处理字符串查找的关键算法之一，本篇文章将KMP算法的实现过程抽丝剥茧，一点一点整理出KMP问题解题框架。
---



# 学习笔记三 KMP算法介绍

本题来自：[力扣28.实现strStr()](https://leetcode-cn.com/problems/implement-strstr/)

## 题目描述
简单来说，题目要求是从一个字符串 haystack 中找到另一个字符串 needle 出现的第一个位置。如果按暴力解法，题目解答很容易想到是 O(m * n)  的时间复杂度。那么，有没有一种解法，它的时间复杂度可以降到 **O(m + n)** 呢？

有！这就是我们接下来要介绍的 **KMP 算法**

## KMP的主要思想
- **KMP的主要思想是：**当出现字符串不匹配时，可以知道一部分之前已经匹配的文本内容，避免从头再去做匹配了。这也是我们能将时间复杂度降低的本质。

## KMP实现的关键步骤
### 一、构建 next 数组
先来解释几个名词：前缀，后缀，最长公共前后缀，前缀表
> 前缀：不包含最后一个字符的，并以第一个字符为开头的所有子字符串
> 后缀：不包含第一个字符的，并以最后一个字符为结尾的所有子字符串

> 最长公共前后缀：对于一个字符而言，最长的相同的前缀后缀
> 前缀表：前缀表是记录下标 i 之前（包括 i）的字符串中，有多大长度的最长公共前后缀

而我们要构建的 next 数组，就是所谓的前缀表。

KMP 算法实现的第一个难点便是 next 数组的构建求解，即 **对于一个子串，如何去求它的最长公共前后缀？**

**构造 next 数组其实就是计算短串的前缀表的过程**，其关键步骤主要有如下三步：

- 初始化
- 处理前后缀不相同的情况
- 处理前后缀相同的情况

#### 1，初始化
首先我们定义一个函数 getNext 去构建 next 数组，函数参数为指向 next 数组的指针，和一个字符串。
```C++
void getNext(int* next, string needle)   //利用函数封装，使主函数逻辑更加简洁
```
由于我们要比较一个字符串的前缀后缀，那就必须有两个指针分别在字符串的开头和结尾。而对于 next 而言，第一位只有一个字母的情况前缀后缀都是 0 ，可以直接初始化。
```C++
int left = 0;
next[0] = left;
for (int right = 1; right < needle.size(); right++) {
        
}
```
#### 2，后缀不相同
如果在中间某个位置发现 right 与 left 的值不相同，就需要不断进行回溯。

由于 next 数组保留着前后缀相同的位置，那么回溯的位置便是 next 数组中 left 的前一个位置 left - 1 。如果回溯后的 left 仍然不等于 right 位置的字符，那就继续回溯，直到 left 到达字符串左边界。但是为了防止 left - 1 溢出数组边界，left = 1 时便就是边界了，因此判断条件为 left > 0。
```C++
while (left > 0 && needle[left] != needle[right]) {
	left = next[left - 1];
}
```
#### 3，后缀相同
如果 left 和 right 的后缀相同，那么说明找到了相同的前后缀。left，right 递增，同时把 left 的值赋给 next 数组。
```C++
if (needle[left] == needle[right]) {
	left++;
}
next[right] = left;
```

#### 4，完整代码
因此，构建 next 数组完整代码如下：（时间复杂度O(n)）
```C++
    void getNext(int* next, string needle) {
        int left = 0;
        next[0] = left;
        for (int right = 1; right < needle.size(); right++) {
            while (left > 0 && needle[left] != needle[right]) {
                left = next[left - 1];
            }
            if (needle[left] == needle[right]) {
                left++;
            }
            next[right] = left;
        }
    }
```

### 二、利用 next 数组进行匹配
那么有同学就要问了，得到这个前缀表，到底对字符串匹配过程有什么帮助呢？
别急，我们来看一个小例子。

- 例：如果我们要从 aabaabaafa 中寻找 aabaaf 第一次出现的位置。

**1，暴力解法：**两个字符串均从第一个位置开始比较，比较到第6位发现 b 不等于 f ，比较失败。长串走到第二个位置；短串回到第一个位置，继续比较直到长串走完。（ 时间复杂度O(m * n））
**2，KMP算法：**两个字符串均从第一个位置开始比较，比较到第6位发现 b 不等于 f，比较失败。长串位置不变；短串，调用 next 数组，找到 next 数组第5位的数字，并作为位置传递给短串，这时候短串到第3位 b 字符，和长串匹配，比较继续。（时间复杂度O(m)）

#### 匹配错误时
只是再解释一下匹配错误时的情况，**第6位 f 匹配失败，这时候第5位 next 数组的值为2，这说明此时第4、5位和第1、2位相同，又因为第4、5位已经和长串匹配成功，这说明第1、2位也可以匹配成功，因此直接从第三位，也就是数组下标为2的地方开始短串匹配即可。**

#### 匹配成功时
长短串位置均正常递增即可，只是需要增加结束条件，即短串位置到达短串右端点，这说明长串已经有位置可以将短串完全匹配，循环结束。

我们可以发现，匹配过程和 next 数组生成过程代码几乎完全一样，因此代码的思路模版可以进行记忆背诵。
- 完整代码：
```C++
        int n = 0;
        for (int h = 0; h < haystack.size(); h++) {
            while (n > 0 && haystack[h] != needle[n]) {
                n = next[n - 1];
            }
            if (haystack[h] == needle[n]) {
                ++n;
            }
            if (n == needle.size()) {
                return h - needle.size() + 1;
            }
        }
        return -1;
```

## KMP完整代码
```C++
class Solution {
public:
    void getNext(int* next, string needle) {
        int left = 0;
        next[0] = left;
        for (int right = 1; right < needle.size(); right++) {
            while (left > 0 && needle[left] != needle[right]) {
                left = next[left - 1];
            }
            if (needle[left] == needle[right]) {
                left++;
            }
            next[right] = left;
        }
    }

    int strStr(string haystack, string needle) {
        if (needle.size() == 0) {
            return 0;
        }
        int next[needle.size()];
        getNext(next, needle);
        int n = 0;
        for (int h = 0; h < haystack.size(); h++) {
            while (n > 0 && haystack[h] != needle[n]) {
                n = next[n - 1];
            }
            if (haystack[h] == needle[n]) {
                ++n;
            }
            if (n == needle.size()) {
                return h - needle.size() + 1;
            }
        }
        return -1;
    }
};
```