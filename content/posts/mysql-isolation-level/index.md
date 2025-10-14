---
title: "MySQL 中的四种隔离等级"
slug: "mysql-isolation-level"
date: "2020-08-30T02:01:00+0000"
lastmod: "2025-01-16T09:42:31+0000"
draft: false
tags:
  - "MySQL"
  - "Database"
visibility: "public"
---
# 0X00 What's this

我们知道 MySQL 中存在“事务”这么个事物（我是故意拗口的，哈哈哈哈哈哈哈）；我们也知道事务“一荣俱荣，一损俱损”（要么事物内所有查询均生效，要么均不生效）。那么现在问题来了，银行数据库中有两个事务在同时进行，我们来看一下这两个事务

> 一条 SQL 就是一个查询，不一定是 `SELECT xxx` 才叫查询。

```sql
    -- 事务 A    shawn 转账给 bluce 100 块钱
    BEGIN;
    UPDATE account SET money = money - 100 WHERE username = 'shawn';    -- 这行代码标记为  A-1
    UPDATE account SET money = money + 100 WHERE username = 'bluce';     -- 这行代码标记为  A-2
    COMMIT;

    -- 事务 B
    BEGIN;
    SELECT money FROM account WHERE username = 'shawn';    -- 这行代码标记为 B-1
    SELECT money FROM account WHERE username = 'bluce';     -- 这行代码标记为 B-2
    COMMIT;
```

现在有这么一个情况：shawn 和 bluce 两人账上都有 100 块钱，此时事务 A 和 B 同时开始执行，那么事务 B 所查询到的两人的账户余额究竟是多少呢？这个问题并没有一个确切的结果，因为 B 在查询的时候，转账正在进行中。不过要回答这个问题也不是做不到，就需要引入标题中提到的“隔离等级”这个概念了。

我们在 MySQL 中使用这四种隔离等级：READ UNCOMMITTED/READ COMMITTED/REPEATABLE READ/SERIALIZABLE。隔离等级的不同就意味着在同一个时间节点下查询到的内容可能不同，数据的可靠性也会变得不同。接下来我们来简单了解一下这四种隔离等级，并且通过一个测试数据库来验证一下。

# 0X01 How to try it

首先要建一个数据库来做测试，表结构和两条基础数据长这样，可以通过随便什么方法把表先建好。

```sql
    CREATE TABLE `account` (
      `id` int(11) NOT NULL,
      `username` varchar(10),
      `money` int(11) DEFAULT 0,
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

    INSERT INTO `account` VALUES (1, 'shawn', 100),(2, 'bluce', 100);
```

然后我们打开两个 MySQL 的终端连接（一定是两个），其中一个用来执行事务 A，一个用来修改隔离等级然后观察结果。

修改隔离等级的时候，可以选择“全局”或者“会话”两种级别，我们在测试阶段就只需要修改“会话”级就可以了。用的是`SET session transaction isolation level read committed;`这样的方法，其中最后的`read committed`就是其中一个隔离等级。

> 注意在会话级别修改隔离等级应该是在你 select 去验证的会话里执行，而非执行事务的会话。换个说法就是“隔离等级”是针对事务外的，而非针对事务。

# 0X02 READ UNCOMMITTED

首先是 `READ UNCOMMITTED` 这个最简单的隔离机制：“读未查询”。其实这个翻译是准确的，就是有点别扭，换个说法叫“可以读到未提交的数据”。可以直接理解成“没有隔离”。也就是说所有改动都是实时的，别人事务里改动也是实时的，即使事务还没`commit`我也能看到。

```
    +----+----------+-------+
    | id | username | money |
    +----+----------+-------+
    | 1  | shawn    | 100   |
    | 2  | bluce    | 100   |
    +----+----------+-------+
```

拿上面的数据库来举个例子，现在数据库里数据是这个样子的。首先我们打开数据库终端两个，分别叫做“甲/乙”好了，为了好区分。甲负责执行事务，乙负责检验效果。我们在甲终端里开始一个事务，但是不提交

```sql
    mysql root@127.0.0.1:test> BEGIN;
    Query OK, 0 rows affected
    Time: 0.002s
    mysql root@127.0.0.1:test> UPDATE account SET money = money - 100 WHERE username = 'shawn';
    Query OK, 1 row affected
    Time: 0.003s;
    mysql root@127.0.0.1:test> UPDATE account SET money = money + 100 WHERE username = 'bluce';
    Query OK, 1 row affected
    Time: 0.002s
```sh

然后在乙终端里将会话的事务隔离等级改为 `READ UNCOMMITTED`，用这条命令`SET session transaction isolation level read uncommitted`就可以了。接下来我们再从终端乙查询一下整张表的数据

```
    +----+----------+-------+
    | id | username | money |
    +----+----------+-------+
    | 1  | shawn    | 0     |
    | 2  | bluce    | 200   |
    +----+----------+-------+
``

发现转账已经成功了。问题也就出在这儿了，假设我们终端甲里执行的事物还有好多事情要做，而且后面执行过程中出错回滚了，那当前的数据其实就是错误的。这种读到脏数据的行为我们称之为 **脏读** 。就光是这两条语句，都有可能出问题的。如果我们在事物给 shawn 的钱 100 之后读，就能读到 shawn 没有钱且 bluce 有 100 块，那此时的钱就凭空消失了 100。再或者说，shawn 在系统里给别人转账，点击确定后自己钱没了，然后系统在把钱加到对方账户上的时候发现对方账户有问题，事务就回滚了，shawn 发现自己的钱从 100 到 0 再到 100，就很奇怪。命名是转账成功变 0，不成功就保留 100 的事情，结果居然在两个数字减反复横跳，就很奇怪。

> 记得最后在终端甲上 commit 一下，要不然一会儿数据对不上了

如果要解决脏读的问题，其实也比较简单，可以引入下面提到的 READ COMMITTED隔离级别。

# 0X03 READ COMMITTED

想要让数据更靠谱就只有牺牲资源，比如加锁。在这种隔离等级下不会出现上面的脏读现象，只有当事务`commit`之后事物外才能读到改动。“那岂不是这样就完美了？” too young too simple. 正如前面说的，这种操作是要加锁的，既然加锁就有可能出问题。首先是并发，有 100 个人都要给 bluce 转账，触发了 100 个类似于上面的查询，第一个开始的会取得 bluce 的排他锁，后面的就直接卡住了。等第一个释放掉之后，后面 99 个事务再抢。。。这样还算好的，如果其中一个事物不止做了这些，还有更复杂的操作，导致事务耗时比较久，那么后面的所有事务都会被卡住，并且有可能会导致事务超时。

还有一个问题，涉及到索引。`UPDATE account SET money = money - 100 WHERE username = 'shawn' `这条 SQL 对应的 username 字段如果在 account 表里没有索引，会发生什么？MySQL 会不知道哪条才是`username = 'shawn'`，那咋办？锁表，准确的来说不是加表锁，是给表里所有行加行锁，等找到`username = 'shawn'`的时候再释放其他的。如果表里有 100w 数据，那就要给 999999 条数据做无意义的加锁解锁操作。

# 0X04 REPEATABLE READ

这个是 MySQL 默认采用的隔离级别，称为“可重复读”。既然有可重复读就一定会有一个对应的“不可重复读”，这里简单介绍一下怎么叫可重复读和不可重复读，还是以两个事务为例子。前面提到的两个隔离等级都是“不可重复读”的，这个很好理解，只是中文叫法有点奇怪，我们看一个“不可重复读”的例子（shawn/bluce 各有 100 块存款）

```sql
    -- 事务 A
    BEGIN;
    UPDATE account SET money = money - 100 WHERE username = 'shawn';    -- 这行代码标记为  A-1
    UPDATE account SET money = money + 100 WHERE username = 'bluce';     -- 这行代码标记为  A-2
    COMMIT;

    -- 事务 B
    BEGIN;
    SELECT money FROM account WHERE username = 'shawn';     -- 第一波查询
    SELECT money FROM account WHERE username = 'bluce';

    SELECT money FROM account WHERE username = 'shawn';     -- 第二波查询
    SELECT money FROM account WHERE username = 'bluce';
    COMMIT;
```

当两个事务同时开始后，事务 B 的两波查询可能会得到不同的结果，相同的 SQL 却得到了不同的数据，是会导致问题的。以 READ COMMITTED 为例，两个事物同时开始，B 完成了第一波查询发现俩人都有 100 块；这时候 A 才执行两条修改数据的查询，并且提交了结果；事务 B 继续执行查询，发现 shawn 没钱了，而 bluce 多了 100。这种情况在稍微复杂一点的事务中就会引发数据一致性的问题。所以**MySQL选择了使用可重复读来做默认的隔离参数** 。

还有一个跟“不可重复读”类似的问题叫做“幻读”（跟幻术-月读没啥关系），因为只是防止数据更新造成“不可重复读”的话，我们可以通过加锁来解决，加什么锁呢，表锁？太费性能了，那就行锁吧。可是行锁有个问题，就是锁不到 INSERT ，毕竟数据还没有呢，你怎么锁。所以幻读简单来说是这样的：

  1. 事务 A 实现一个“当用户名为 xxx 的用户不存在时就创建这条数据”
  2. 当事务 A `SELECT`了一趟发现不存在后，事务 B 创建了这个用户
  3. 事务 A 因为自己刚刚看了数据不存在，就创建这个用户， _然后就报错了_

> 所以说“不可重复读”和“幻读”是两回事，但是本质上都会导致一个事务内对统一条件的 query 出现偏差。

那么 MySQL 是怎么解决“不可重复读”和“幻读”这两个问题的呢？可以简单想象成，每当你开始一个事务的时候，就相当于给 MySQL 做了一个 git 分支，当你事务执行完了再 merge 回去（或者想象成快照）。

  1. **当然，MySQL当然不是这么实现的，只是用这种方法可以简单快速的理解到效果，而非原理**
  2. **当然，MySQL当然不是这么实现的，只是用这种方法可以简单快速的理解到效果，而非原理**
  3. **当然，MySQL当然不是这么实现的，只是用这种方法可以简单快速的理解到效果，而非原理**

我这里因为只是介绍四种隔离级别就不再扩展了，有兴趣的话可以看一看 MVCC 相关的内容。

# 0X05 SERIALIZABLE

最后一个隔离级别相当于“全隔离”，读数据加共享锁、写数据加排他锁、读写还互斥，**并发性能“无敌”** 。当然了，这也不都是缺点，如果你恰巧需要一个没什么并发但是又对数据及时性和可靠性要求极高，那还真可以试试。

 _而且在这种隔离级别下，SELECT 都是要加锁的，你要是来个全表扫描，别人数据都写不进去了~_

# 0X06 Done.

参考内容：《高性能 MySQL》和[美团技术团队博客](<https://tech.meituan.com/2014/08/20/innodb-lock.html>)
