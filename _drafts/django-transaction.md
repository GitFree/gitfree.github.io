---
layout: post
title: 浅谈Django的事务管理
categories: [Django]
description: some word here
keywords: Django, Transaction, 事务
---
*说明：本文基于Django 1.10版本编写，不保证对于早期的Django版本适用，尤其是Djnago 1.7之前的版本。*

事务是在开发互联网应用中非常常见的保持操作原子性的技术手段，Django对事务的支持非常强大，对于常见的应用开发场景都能提供有力支持。
本文对Django的事务进行了简单介绍，从Django事务默认行为、Django事务常用功能、Django事务锁支持、Django事务使用中的常见问题几个方面对Django的事务进行了
全方位解读。

### 一、Django事务默认行为
使用Django事务，第一个件事就是要搞懂Django这个Web框架对于事务的处理方式。Django事务[官方文档](https://docs.djangoproject.com/en/1.10/topics/db/transactions/#django-s-default-transaction-behavior)第一段就对此做出了明确说明，原文如下：

> #### Django’s default transaction behavior¶
>
> Django’s default behavior is to run in autocommit mode. Each query is immediately committed to the database, unless a transaction is active. See below for details.
>
> Django uses transactions or savepoints automatically to guarantee the integrity of ORM operations that require multiple queries, especially delete() and update() queries.
>
> Django’s TestCase class also wraps each test in a transaction for performance reasons.

翻译成中文就是：
> Django事务默认运行在自动提交(autocommit)模式。只要不在事务中，每次请求都会被立即提交到数据库。
>
> Django对某些需要数据完整性（原子性）的ORM操作会自动使用事务或保存点，比如需要多次数据库请求的ORM操作，当然还有delete和update的操作。
>
> 出于性能考虑（译注：方便不同用例的数据隔离和跑完用例后的数据清理，Django跑完一个用例后会调用rollback回滚当前用例产生的数据），Django的TestCase类也会把每个测试用例跑在事务中。

以上这段官网说明对Django事务默认行为表达的已经很清楚，但是关于Djnago为什么默认运行在autocommit模式还有一些特别的需要说明。
因为在SQL标准中，每一条SQL语句都会开启一个事务（除非当前已经在一个事务中了），然后这个事务必须被显式的提交或者回滚，
而且[PEP249](https://www.python.org/dev/peps/pep-0249)也与SQL一样，要求Python实现的数据库API必须默认关闭autocommit模式。  
但是默认关闭autocommit显然对于开发者很不方便，因为要自己维护事务，所以代码中需要有大量的commit和rollback语句，所以作为Web框架的Django本着方便Web开发的目的默认开启了autocommit
（注意Django属于Web框架，并不属于PEP249所规定的数据库API，因为Django并不对数据库直接操作，比如操作MySQL，还需要安装MySQL-python包）。在开启autocommit的情况下，每条SQL查询都被
包在独立的事务中（除非当前已经在一个事务中了），这条语句执行完后，对应的事务会立刻被提交或回滚，这样看这条语句是这行成功了还是失败了。
当然Django也提供了关闭默认autocommit的选项，只要将settings中的AUTOCOMMIT设置为False就行。但是除非你有非常特殊的应用场景并且非常确信一定要自己去管理事务，否则千万不要这么做。

### 二、Django事务常用功能
### 三、Django事务锁支持
### 四、Django事务使用中的常见问题
