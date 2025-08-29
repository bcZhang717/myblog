---
title: MySQL 三大日志与两阶段提交
date: 2025-08-26
tags:
  - MySQL
  - 日志
categories:
  - MySQL
description: >-
  这篇文章详细介绍了MySQL的三大日志：undo log、redo log和binlog的作用与机制。undo
  log用于保证事务的原子性，记录数据的版本变化以支持回滚和MVCC；redo
  log确保事务的持久性，通过记录物理页的修改并在崩溃后恢复数据；binlog则是逻辑日志，用于备份恢复和主从复制。文章还深入讲解了两阶段提交的过程，通过prepare和commit阶段协调redo
  log和binlog的一致性，确保事务在异常情况下能够正确恢复，从而保障数据的一致性和可靠性。
summary: >-
  这里是爱谦AI，这篇文章详细介绍了MySQL的三大日志：undo log、redo log和binlog的作用与机制。undo
  log用于保证事务的原子性，记录数据的版本变化以支持回滚和MVCC；redo
  log确保事务的持久性，通过记录物理页的修改并在崩溃后恢复数据；binlog则是逻辑日志，用于备份恢复和主从复制。文章还深入讲解了两阶段提交的过程，通过prepare和commit阶段协调redo
  log和binlog的一致性，确保事务在异常情况下能够正确恢复，从而保障数据的一致性和可靠性。
---

MySQL 的三大日志：undo log、redo log、binlog
# undo log

## undo log 保证事务的原子性

执行一组 SQL 语句，要么全部成功，要么全部失败，失败了需要把前面执行成功的 SQL 语句进行回滚，这就是 MySQL 事务的原子性。此时就出现了一个问题：数据都进行修改了，是怎么进行回滚的？

这就需要 undo log 了。undo log 中记录了数据的不同版本，每修改一次就记录一次对应的版本，随后把数据不同的版本通过一个回滚指针串联起来，这就是 undo log 版本链。

<img src="https://picgo-blog-1335849645.cos.ap-guangzhou.myqcloud.com/images/20250825153432795.png" alt="image-20250825153432631" style="zoom:67%;" />

也就是说，只要事务执行过程中出现了数据的修改，MySQL 就会把旧数据记录到 undo log 日志当中。如果事务正常提交，那就什么事没有；如果事务执行出现了异常，那就会读取 undo log 日志找到旧数据的版本进行回滚，**这就是 undo log 保证事务原子性的原理**。

## undo log 与 MVCC

**undo log 的第二个作用就是配合 readview 与表中的隐藏字段实现 MVCC。**

简单来说，MVCC 执行普通的 select 语句时，会去对比事务的 id 和 readview 来判断数据版本是否对当前的事务可见。如果不可见，就继续沿着 undo log 版本链去查找当前事务可见的数据版本。

# redo log

## buffer pool

MySQL 中的数据是存储在磁盘当中的，但是每次都从磁盘读取数据就会很慢，因此 MySQL 设计了一个 buffer pool 缓冲池。MySQL 存数据是以页为单位的，一页的大小是 16 KB，每查询一次数据就会从磁盘把这一页的数据都加载出来，然后放到 buffer pool 当中。

## 程序的空间局部性

为什么每次查询一条数据都会把一整页加载到 buffer pool 当中，而不是单独的这一条数据？

这就涉及到了程序的空间局部性原理：一个内存地址被访问，这就意味着它附近的内存地址也有可能会被访问，这就好比你刷抖音，当前的视频被访问，那么下一个视频会有极其大的概率也会被访问，这就是一种翻页。

而 MySQL 一次性把一整页的数据都加载到 buffer pool 当中，那么对于连续存储的数据来说，命中率就会非常高，比如数组。后续再次查找数据的时候就会优先从 buffer pool 中去读取，不存在了再去磁盘当中读取。相应地，写入数据也是先写入 buffer pool，但是不会直接刷入数据到磁盘，而是先将修改的页标记为脏页，再由后台线程在某个时间把脏页刷入到磁盘。

## redo log 的必要性

如果在 buffer pool 还没有刷入脏页到磁盘中的时候，MySQL 宕机了，不就出现了数据丢失，该怎么避免？

此时 redo log 的作用就体现出来了。MySQL 会把某个磁盘页修改的数据记录在 redo log 中，事务提交之后就刷盘 redo log。如果 buffer pool 中的数据未刷盘就宕机了，那么就可以读取 redo log 来恢复数据，**这也就是 redo log 能够保证事务持久性的原因**，能够让 MySQL 做到崩溃后的数据恢复。

## 问题一

**为什么不直接刷盘 buffer pool，而是刷盘 redo log？**

因为 buffer pool 刷盘是随机 IO，刷盘的时候需要找到某个磁盘页，然后修改，然后再去找另外一个磁盘页，再修改。本来磁盘操作就慢，随机 IO 的话就更慢了。

但是 redo log 只记录在哪个磁盘位置做了怎样的修改，刷盘时只需要往 redo log 日志文件后追加就行了，这就是顺序 IO，不需要找这个磁盘页找那个磁盘页，磁盘的磁头只需要沿着一个方向移动就行了。

这也就是为什么 redo log 里面要记录数据页的物理修改，而不是直接记录数据。所以只要 redo log 一刷盘，即便 MySQL 崩溃了也能够恢复数据，事务的持久性也能够得到保证。

## 问题二

**如果 redo log 还没来得及刷盘，MySQL 就宕机了，会丢失数据吗？**

我们需要先了解一下 redo log 的刷盘机制，再来解答这个问题。

redo log 其实也不是直接写入磁盘的，redo log 有自己的缓存，叫 redo log buffer。redo log 会先写入到 redo log buffer，然后统一把 redo log buffer 中的数据刷入到磁盘中。

MySQL 中有一个参数 `innodb_flush_log_at_trx_commit` ，可以控制 redo log 的刷盘时间。

如果值为 0，事务提交时会把 redo log 的数据放入 redo log buffer，并不会刷入磁盘。

如果值为 1，事务提交时会把 redo log 的数据刷入磁盘，刷盘完成后才会告诉客户端事务执行成功了。这也就保证了事务只要完成，即便 MySQL 崩溃了数据也不会丢失。所以一般情况下，把这个参数设置为 1，那事务提交 redo log 就能刷盘，事务的持久性也能够得到保证，1 也是这个参数的默认值。

如果值为 2，事务提交时会把 redo log 的数据写入操作系统的文件缓存中，也就是 page cache。操作系统本身对文件也是有缓存的，数据写入 page cache        后，操作系统就会在某个时间把数据写入磁盘，只要操作系统不崩溃，那就能保证持久性。

## 问题三

**现在需要做数据库的备份用于备份恢复，或者需要搭建主从架构，完成主从复制，那么 redo log 能胜任这些工作吗？**

这是不行的。redo log 是一种环形日志，其空间大小是固定的。全部写满了就会从头再开始写，边写边覆盖前面的数据。因此它只能记录事务提交后没有被刷盘的数据，已经刷盘的会从 redo log 中擦除掉。

redo log 是事务级的数据记录，不是数据库级的。它只能做一些因为断电或者数据库故障导致 buffer pool 中的数据未刷盘的数据恢复，不能做数据库全量数据恢复。

如果需要进行数据库的数据恢复，则需要使用到 binlog 日志文件。

# binlog

只要对 MySQL 做了变更，无论是数据还是表结构的增删改，都会记录一个二进制日志 binlog。

redo log 是物理日志，记录的内容是 "在某个数据页上做了什么修改"。而 binlog 是逻辑日志，记录的内容就类似于 SQL 语句本身，所以 binlog 非常适合做备份恢复和主从同步。

> 删库跑路时也最好把 binlog 一起顺手删了，这是非常可刑的。

相比于 redo log，binlog 也有对应的 binlog cache。事务执行过程中，会先把 binlog 日志写到 binlog cache；事务提交的时候，就会把 binlog cache 刷到磁盘中。

# 两阶段提交

**事务提交后，redo log 和 binlog 都要刷盘，但是如果一个刷盘成功了，一个失败了，两份日志就不一致了，这种情况该怎么办？**

为了解决两份日志之间的一致性问题，MySQL 将 redo log 的写入拆成了两个步骤：`prepare` 和 `commit` ，这就是两阶段提交。

整个执行流程如下：

- 开启事务
- 更新数据 
- 写入 redo log，此时 redo log 是 `prepare` 阶段
- 提交事务，写入 binlog，并将 redolog 设置为 `commit` 阶段

**1、如果写入 redo log 时出现了异常，该怎么解决？**

此时 redo 处于 `prepare` 阶段，binlog 中没有数据，直接回滚事务即可。

**2、 如果写入 binlog 时出现了异常，该怎么解决？**

此时 binlog 已经写入，但 redo 还没有进入 `commit` 阶段，此时就需要对比 redo log `prepare` 阶段的数据与 binlog 是否一致，一致就提交事务，不一致就回滚事务。

**3、怎么比较 redo log 与 binlog 的数据是否一致？**

redo log 与 binlog 中存在一个共同的字段 `XID` 。因此崩溃恢复的时候，扫描 redo log，如果发现有 `prepare` 的 redolog，则利用它的 `XID` 去 binlog 查询，如果找到对应的数据，则说明数据都保存了，事务可以提交，反之事务回滚。

由此看来，两阶段提交最终还是要看 binlog 日志。