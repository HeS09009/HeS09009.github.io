---
title: 锁机制注入 bypass 雷池 WAF！
date: 2025-06-27 00:44:00 +0800
categories: [漏洞分析, Web安全]
tags: [SQL注入, Bypass WAF, Web安全]
media_subpath: /
---

# 锁机制注入 bypass 雷池 WAF！

在 SQL 注入的延时注入中，常见的函数有 `sleep()` 直接延时、`BENCHMARK()` 通过让数据库进行大量的计算而达到延时的效果、笛卡尔积、正则匹配等，但还有一个常常被忽略的函数，也就是 MySQL 中的锁机制。虽然早些年就已经出现过相关技术文章，但它的应用却几乎见不到，也没有文章对其机制和运用进行深入讲解，而且该函数也常常被 WAF 忽略导致延时。:contentReference[oaicite:1]{index=1}

## Mysql 锁机制介绍

### 函数介绍

`GET_LOCK()` 是 MySQL 提供的一个用户级锁函数，用于在应用层实现跨会话的锁机制：

```sql
GET_LOCK(str lock_name, int timeout)
```
**参数：**

lock_name（字符串）：锁名称（最大64字符，区分大小写）

timeout（整数）：等待超时时间（秒），0表示立即返回，负数表示无限等待（MySQL 5.7.5+）

**返回值：**

1：成功获取锁

0：获取锁超时（其他会话持有锁且未在指定时间内释放）

NULL：发生错误（如参数无效、内存不足等）

