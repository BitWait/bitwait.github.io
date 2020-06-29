---
layout: post
title:  "longestPalindrome"
description: ARTS for week 4.
date:   2020-06-29 21:03:36 +0530
---
- Algorithms longestPalindrome
- Review Teach Yourself Programming in Ten Years
- Tip 以PING讲解二层转发原理
- Share 为什么多线程读写 shared_ptr 要加锁？


## Algorithms --- 最长回文子串
### 回文串特点：
- 从左到右与从右到左是一样的
- 从回文串的中间向两边是对称的

### 分析：
找到最长的如何保存？

### 源码：

```
string palindrome(std::string& s, int left, int right)
{
    // left = right 的时候，此时回文中心是一个空隙，向两边扩散得到的回文子串的长度是奇数
    // right = left + 1 的时候，此时回文中心是一个字符，向两边扩散得到的回文子串的长度是偶数

	while (left >= 0 && right < s.size() && s[left] == s[right])
	{
		left--;
		right++;
	}
	return s.substr(left + 1, right - left - 1);
}

string longestPalindrome(string s)
{
	string res;
	for (int i = 0; i < s.size(); i++)
	{
		string s1 = palindrome(s, i, i);
		string s2 = palindrome(s, i, i+1);
		res = res.size() > s1.size() ? res : s1;
		res = res.size() > s2.size() ? res : s2;
	}
	return res;
}
```
## Review
### 3/5 http://www.kegel.com/c10k.html
#### 备注单词
- underlying  底层的
- compliance  合规
- regardless 不管
- stuck 卡住
#### 关键语句
- AIO is normally used with edge-triggered completion notification, i.e. a signal is queued when the operation is complete. (It can also be used with level triggered completion notification by calling aio_suspend(), though I suspect few people do this.)

## Tip --- 以PING讲解二层转发原理
- pc1根据pc2的ip与自己的掩码计算是否属于同一子网===》属于同一子网走二层，不属于走三层
- 属于：pc1查询本机arp表是否含有pc2对应的mac ==>无对应mac发送arp；有则直接发送icmp
- 无对应mac: 发送arp请求（目的mac为全F,请求ip为pc2）
- pc2收到arp广播：回复arp应答给pc1
- pc1收到应答则更新arp表，并发送icmp请求报文给pc2，pc2收到请求报文则回复给pc1(ping 成功)

## Share
### 为什么多线程读写 shared_ptr 要加锁？
```
版权声明：本文为CSDN博主「陈硕」的原创文章，遵循CC 4.0 
BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/Solstice/article/details/8547547
```
- shared_ptr）的引用计数本身是安全且无锁的，但对象的读写则不是，因为 shared_ptr 有两个数据成员，读写操作不能原子化。根据文档（http://www.boost.org/doc/libs/release/libs/smart_ptr/shared_ptr.htm#ThreadSafety）， shared_ptr 的线程安全级别和内建类型、标准库容器、std::string 一样，即：
- 一个 shared_ptr 对象实体可被多个线程同时读取（文档例1）；
- 两个 shared_ptr 对象实体可以被两个线程同时写入（例2），“析构”算写操作；
- 如果要从多个线程读写同一个 shared_ptr 对象，那么需要加锁（例3~5）。
- 请注意，以上是 shared_ptr 对象本身的线程安全级别，不是它管理的对象的线程安全级别。
