---
title: 学习笔记四 KMP算法应用
date: 2022-02-18
updated:
tags: 
- 力扣
- KMP
- 算法
categories: 力扣学习笔记
excerpt: KMP算法是处理字符串查找的关键算法之一，本篇文章延续上一篇文章对KMP算法的讲解，再分析了一道经典例题。
---



# 学习笔记四 KMP算法应用

本题来自：[力扣459.重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern/)

## 题目描述
- 给定一个非空的字符串 s ，判断 s 是否能通过它的一个子串重复多次获得。

这时候肯定有同学会问了，连要比较的短串是什么都不知道，**怎么用KMP呢？**

## 编程思路
还是和 KMP算法实现过程一样，我们需要一个 next 数组来记录目前的最长前后缀。
- 以一个简单例子入手：abcabcabc
- 可以发现，该字符串的最长前后缀是 abcabc，而**该字符串减去**这个**最长前后缀**正好就是循环的子字符串。
- 这究竟是不是巧合？
- 我们发现，如果字符串长度减去最长前后缀的长度正好可以被该字符串整除，那么这个字符串必然是可以被子字符串循环生成的！

因此思路便是：

**1，建立字符串 s 的 next 数组。**

**2，通过上述条件判断 true or false。**

### 构造 next 数组
构造 next 数组的方法和 KMP算法完全一致，这里不再赘述
```C++
    void getNext(int* next, int n,const string s) {
        if (n > 0) next[0] = 0;
        int left = 0;
        for (int right = 1; right < n; right++) {
            while (left != 0 && s[left] != s[right]) {
                left = next[left - 1];
            }
            if (s[left] == s[right]) {
                left++;
            }
            next[right] = left;
        } 
    }
```
### 借助 next 数组进行判断
由于需要判断的就只有字符串 s 末尾的最长前后缀，因此也很容易写出。

**需要注意的一点是：**在进行 true 和 false 的判断时，条件除了
```C++
n % (n - next[n - 1]) == 0  //条件一
```
以外，还应该有
```C++
next[n - 1] != 0            //条件二
```
这是因为，**如果 next[n - 1] 值为 0，那么第一式就变成 n % n = 0 的恒等式了**。
> 例如，字符串 abac 便只满足条件一，但它应该 return false。



判断部分代码如下：
```C++
    bool repeatedSubstringPattern(string s) {
        if (s.size() == 0) return false;
        int n = s.size();
        int next[n];
        getNext(next, n, s);
        if (next[n - 1] != 0 && n % (n - next[n - 1]) == 0) return true;
        return false;
    }
```
## 完整代码
```C++
class Solution {
public:
    void getNext(int* next, int n,const string s) {
        if (n > 0) next[0] = 0;
        int left = 0;
        for (int right = 1; right < n; right++) {
            while (left != 0 && s[left] != s[right]) {
                left = next[left - 1];
            }
            if (s[left] == s[right]) {
                left++;
            }
            next[right] = left;
        } 
    }
    bool repeatedSubstringPattern(string s) {
        if (s.size() == 0) return false;
        int n = s.size();
        int next[n];
        getNext(next, n, s);
        if (next[n - 1] != 0 && n % (n - next[n - 1]) == 0) return true;
        return false;
    }
};
```