---
title: MySQL 中的 MVCC
date: 2025-08-25
description: 
tags:
  - MySQL
  - MVCC
categories:
  - MySQL
---
# 什么是 MVCC？

**MVCC：Multi-Version Concurrency Control，多版本并发控制**。MVCC 是一种在数据库管理系统中用来提高并发性能的机制。它主要用于在不加锁或减少加锁的情况下，实现事务的隔离性，从而允许多个事务并发读写数据而不会相互阻塞。

# MVCC 的核心机制

## 场景引入

一张数据库表里面存了很多数据，现在有很多并发事务来访问或者修改数据。在并发事务情况下会出现脏读、不可重复读、幻读问题。

想要解决脏读，就需要读已提交的隔离级别，也就是说要想办法保证一个事务只能读到其他事务已提交的数据；其他事务没提交的数据，不应该被读取到。

想要解决不可重复读，就需要可重复读隔离级别，也就是说一个事务第一次读取之后，后面每次读到的数据都应该和第一次一样。即便后面有其他事务新提交了数据也不应该读取到。 

## MVCC 的工作机制

要想实现上述的需求，就需要用到 **MVCC** 了。

MVCC 维护了一份数据的多个版本，每个事务修改一次，就生成一个对应的版本，让不同的事务去读不同的版本。

在读已提交的隔离级别下，让事务去读取已经提交的数据版本，这样就能避免脏读；在不可重复读的隔离级别下，让事务每次都读取同一个版本的数据，这样每次读到的就都一样了，就能避免可重复读的问题。 

### 具体操作

把修改的数据版本记录到 undo log 日志中，然后给表增加一个隐藏字段：回滚指针。

让回滚指针指向 undo log 日志，使用回滚指针将历史版本串联为一个链表。这样的话，想读哪个历史版本，沿着链表 一直找就可以了。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825211816620.png" alt="image-20250825211816445" style="zoom:67%;" />

此时就引出了一个问题：**一个事务来查数据，怎么知道我要查哪个版本的数据呢?**

解决方案就是给每个事务分配一个事务 id，事务 id **自增分配**。通过对比事务 id 的大小，就能知道哪个事务创建的早，哪个事务创建的晚。创建比较晚的事务修改的数据不让创建早的事务看到就行了。

所以我们需要给表再增加一个隐藏字段：修改数据的事务 id。谁修改了它，就把对应的事务 id 记录下来。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825153432795.png" alt="image-20250825153432631" style="zoom:67%;" />

### readview

除了 undo log 日志外，还需要一个东西：readview。readview 中存在 4 个重要的字段：

1. `creator_trx_id` ：创建当前 readview 的事务 id。
2. `m_ids` ：创建 readview 时，当前数据库中存在但未提交的所有事务 id 列表。
3. `min_trx_id` ：事务 id 列表中最小的事务 id。
4. `max_trx_id` ：创建 readview 时，应该分配的下一个事务的 id。

这些字段的作用都体现在下图中：

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825213803769.png" alt="image-20250825213803676" style="zoom:75%;" />

readview 的本质就是描绘了一个创建当前事务时的事务 id 数轴。通过对比数据版本的 `trx_id` 在数轴的哪个位置就能知道这个数据版本是否对当前事务是可见的。

主要有以下的几种情况：

**情况一：当前数据的 `trx_id` 值小于 readview 中的 `min_trx_id` 值**

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825214529618.png" alt="image-20250825214529523" style="zoom:75%;" />







<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825214911438.png" alt="image-20250825214911351" style="zoom:75%;" />







<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825214831577.png" alt="image-20250825214831507" style="zoom:75%;" />



























































































