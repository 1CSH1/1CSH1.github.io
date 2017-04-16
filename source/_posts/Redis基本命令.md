---
title: Redis基本命令
date: 2016-03-24 21:48:06
categories: [Redis]
tags: [Redis, 技术人生]
---
**set/get命令（如果已经存在该key，则替换）**

```
> set key "abc"
OK
> get key
"abc"
```
如果健存在，则执行失败；如果健不存在，则执行成功：

```
> set key newval nx
(nil)
> set key newval xx
OK
```
设置Integer值，原子性增减

```
> set count 100
OK
> incr count
(integer)101
> incr count
(integer)102
> incrby count 50
(integer)152
> decr count 2
(integer)150
> decrby count 50
(integer)100
```

**mset/mget命令**

```
> mset a 10 b 20 c 30
OK
> mget a b c
1) "10"
2) "20"
3) "30"
```

**exists命令【返回1表示数据库中存在，返回0表示数据库中不存在】**

```
> set mykey hello
OK
> exists mykey
(integer) 1
> del mykey
(integer) 1
> exists mykey
(integer) 0
```

**type命令【返回某个键的值的类型】**

```
> set key x
OK
> type key
string
> del key
(integer) 1
> type key
none
```

**expire命令**
过期时间可以设置为秒或者毫秒精度。
过期时间分辨率总是 1 毫秒。
过期信息被复制和持久化到磁盘，当 Redis 停止时时间仍然在计算 (也就是说 Redis 保存了过期时间)

```
> set key "expire"
OK
> expire key 5
(integer) 1
> get key 
"expire"
> get key //5秒过后
(nil)
//设置过期时间 10 秒
> set key 100 ex 10
OK
//查看剩余时间
> ttl key
(integer) 6
> ttl key
(integer) 3
//设置毫秒级，用pexpire/pttl命令
>set key "expire"
OK
> pexpire key 10000
(integer) 1
> pttl key
(integer) 4107
> pttl key
(integer) 1108
```

**lpush / rpush / lrange / lpop / rpop 命令**

```
//rpush 从右边添加一个元素
> rpush list A
(integer) 1
> rpush list B
(integer) 2
//lpush 从左边添加一个元素
> lpush list C
(integer) 3
//lrange x[开始元素] y[结束元素] 可以是负数，-1表示最后一个元素，-2表示倒数第二个元素
> lrange list 0 -1
1) "C"
2) "A"
3) "B"
> lrange list 0 1
1) "C"
2) "A"
> lrange list 0 2
1) "C"
2) "A"
3) "B"
//可以同时添加多个元素
> rpush mlist 1 2 3 4 5 6 "abc"
(integer) 7
> lrange mlist 0 -1
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "abc"
//lpop 从左边弹出元素
> lpop mlist
"1"
//rpop 从右边弹出元素
> rpop mlist
"abc"
```

**ltrim 命令**

```
> rpush nlist 1 2 3 4 5
(integer) 5
//保存0 - 2 的元素，其他元素丢弃掉
> ltrim nlist 0 2
OK
> lrange nlist 0 -1
1) "1"
2) "2"
3) "3"
```

列表阻塞操作
**BRPOP 和 BLPOP 两个命令**
它们是当列表为空时 RPOP 和 LPOP 的会阻塞版本：仅当
一个新元素被添加到列表时，或者到达了用户的指定超时时间，才返回给调用者

```
//先将tasks填入元素
> rpush tasks 1 2 3
(integer) 3
//如果tasks有元素，则弹出右边的一个元素，如果tasks无元素，则等待5秒，如果5秒内没有元素添加到tasks，则返回null
> brpop tasks 5
1) "tasks"
2) "3"
> blpop tasks 5
1) "tasks"
2) "1"
> blpop tasks 5
1) "tasks"
2) "2"
> blpop tasks 5
(nil)
(5.08s)
```

集合、有序集合、哈希三条规则：
1. 当我们向聚合(aggregate)数据类型添加一个元素，如果目标键不存在，添加元素前将创建一个空的聚合数
据类型。
2. 当我们从聚合数据类型删除一个元素，如果值为空，则键也会被销毁。
3. 调用一个像 LLEN 的只读命令(返回列表的长度)，或者一个写命令从空键删除元素，总是产生和操作一个持
有空聚合类型值的键一样的结果。

规则1：

```
> del mylist
(integer) 1
> lpush mylist 1 2 3
(integer) 3
```

不能执行一个错误键类型的操作

```
> set foo bar
OK
> lpush foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```

规则2：
```
> lpush mylist 1 2 3
(integer) 3
> exists mylist
(integer) 1
> lpop mylist
"3"
> lpop mylist
"2"
> lpop mylist
"1"
> exists mylist
(integer) 0
```

当所有元素弹出后，键就不存在了

规则3：

```
> del mylist
(integer) 0
> llen mylist
(integer) 0
> lpop mylist
(nil)
```

**哈希/散列**

```
> hmset user:1000 username csh birthday 1993 address GZ
OK
> hget user:1000 username
"csh"
> hget user:1000 birthday
"1993"
> hget user:1000 address
"GZ"
//获取所有键值对
> hgetall user:1000
1) "username"
2) "csh"
3) "birthday"
4) "1993"
5) "address"
6) "GZ"
//根据传入的多个键获取相应的值，如果没有该键值对，则返回(nil)
> hmget user:1000 username birthday no-such-field
1) "csh"
2) "1993"
3) (nil)
//自增键值对的值
> hincrby user:1000 birthday 10
(integer) 2003
> hincrby user:1000 birthday 10
(integer) 2013
```

Set操作

**SADD 命令【添加元素到集合】**

```
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 3
2. 1
3. 2
```

**sismember 命令【判断元素是否在set中】**

```
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```

假设，我们想标记新闻。如果我们的 ID 为 1000 的新闻，被标签 1，2，5 和 77 标记，我们可以有一个这篇新闻
被关联标记 ID 的集合：

```
> sadd news:1000:tags 1 2 5 77
(integer) 4
```

然而有时候我们也想要一些反向的关系：被某个标签标记的所有文章：
```
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1

```

**smembers 命令【获取指定对象的标签】**

```
> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```

**sinter 命令【获取同时被1,2,10,27标记的新闻】**
```
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```

扑克例子：
&emsp;&emsp;抽取一个元素的命令是 SPOP，就方便为很多问题建模。
&emsp;&emsp;例如，为了实现一个基于 web 的扑克游戏，你可以将你的一副牌表示为集合。假设我们使用一个字符前缀表示(C)lubs 梅花， (D)iamonds 方块，(H)earts 红心，(S)pades 黑桃。

```
> sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
S7 S8 S9 S10 SJ SQ SK
(integer) 52
```

&emsp;&emsp;现在我们为每位选手提供 5 张牌。SPOP 命令删除一个随机元素，返回给客户端，是这个场景下的最佳操作。
&emsp;&emsp;然而，如果我们直接对这副牌调用，下一局我们需要再填充一副牌，这个可能不太理想。所以我们一开始要复制一下 deck 键的集合到 game:1:deck 键。
&emsp;&emsp;这是通过使用 SUNIONSTORE 命令完成的，这个命令通常对多个集合执行交集，然后把结果存储在另一个集合中。而对单个集合求交集就是其自身，于是我可以这样拷贝我的这副牌：

```
> sunionstore game:1:deck deck
(integer) 52
```
现在我们准备好为第一个选手提供 5 张牌：
```
> spop game:1:deck
"C6"
> spop game:1:deck
"CQ"
> spop game:1:deck
"D1"
> spop game:1:deck
"CJ"
> spop game:1:deck
"SJ"
```

&emsp;&emsp;只有一对 jack，不太理想……
&emsp;&emsp;现在是时候介绍提供集合中元素数量的命令。这个在集合理论中称为集合的基数(cardinality，也称集合的势)，所以相应的 Redis 命令称为 SCARD。

```
> scard game:1:deck
(integer) 47
```

数学计算式为：52 - 5 = 47。
&emsp;&emsp;当你只需要获得随机元素而不需要从集合中删除，SRANDMEMBER 命令则适合你完成任务。它具有返回重复的和非重复的元素的能力。

有序集合（Sorted Sets）
按照如下规则排序：
• 如果 A 和 B 是拥有不同分数的元素，A.score > B.score，则 A > B。
• 如果 A 和 B 是有相同的分数的元素，如果按字典顺序 A 大于 B，则 A > B。A 和 B 不能相同，因为排序集合只能有唯一元素。

**zadd命令【添加有序集合元素】**

```
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer 1)
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```

**zrange命令【获取有序集合的一个范围】**

```
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```

**zrevrange命令【反顺序】**

```
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
//添加withscores来显示分数
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```

**zrangebyscore / zremrangebyscore / zrank 命令**

```
//返回分数在负无穷到 1950 之间的所有元素(包括两个极端)
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
//删除出生于1940到1960的人
> zremrangebyscore hackers 1940 1960
(integer) 4
//返回排行
> zrank hackers "Anita Borg"
(integer) 4
```

字典分数
如果集合中有相同的分数，允许按字典顺序获取范围

```
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
"Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
0 "Linus Torvalds" 0 "Alan Turing"
//分数相同的按照字母字典顺序排
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
//查询从B到P的元素
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds
```

位图（Bitmaps）

```
> setbit key 10 1
(integer) 1
> getbit key 10
(integer) 1
> getbit key 11
(integer) 0
> setbit key 0 1
(integer) 0
> setbit key 100 1
(integer) 0
//获取个数
> bitcount key
(integer) 2
```


**回收策略(Eviction policies)**
&emsp;&emsp;当 maxmemory 限制到达的时候，Redis 将采取的准确行为是由 maxmemory-policy 配置指令配置的。
以下策略可用：
• noeviction：当到达内存限制时返回错误。当客户端尝试执行命令时会导致更多内存占用(大多数写命令，除
了 DEL 和一些例外)。
• allkeys-lru：回收最近最少使用(LRU)的键，为新数据腾出空间。
• volatile-lru：回收最近最少使用(LRU)的键，但是只回收有设置过期的键，为新数据腾出空间。
• allkeys-random：回收随机的键，为新数据腾出空间。
• volatile-random：回收随机的键，但是只回收有设置过期的键，为新数据腾出空间。
• volatile-ttl：回收有设置过期的键，尝试先回收离 TTL 最短时间的键，为新数据腾出空间。
&emsp;&emsp;当没有满足前提条件的话，volatile-lru，volatile-random 和 volatile-ttl 策略就表现得和 noeviction 一样了。
&emsp;&emsp;选择正确的回收策略是很重要的，取决于你的应用程序的访问模式，但是，你可以在程序运行时重新配置策略，使用 INFO 输出来监控缓存命中和错过的次数，以调优你的设置。
一般经验规则：
&emsp;&emsp;• 如果你期待你的用户请求呈现幂律分布(power-law distribution)，也就是，你期待一部分子集元素被访问得远比其他元素多，可以使用 allkeys-lru 策略。在你不确定时这是一个好的选择。
&emsp;&emsp;• 如果你是循环周期的访问，所有的键被连续扫描，或者你期待请求正常分布(每个元素以相同的概率被访问)，可以使用 allkeys-random 策略。
&emsp;&emsp;• 如果你想能给 Redis 提供建议，通过使用你创建缓存对象的时候设置的 TTL 值，确定哪些对象应该被过期，你可以使用 volatile-ttl 策略。
&emsp;&emsp;当你想使用单个实例来实现缓存和持久化一些键，allkeys-lru 和 volatile-random 策略会很有用。但是，通常最好是运行两个 Redis 实例来解决这个问题。
&emsp;&emsp;另外值得注意的是，为键设置过期时间需要消耗内存，所以使用像 allkeys-lru 这样的策略会更高效，因为在内存压力下没有必要为键的回收设置过期时间。

**回收过程 (Eviction process)**
&emsp;&emsp;理解回收的过程是这么运作的非常的重要：
&emsp;&emsp;&emsp;• 一个客户端运行一个新命令，添加了新数据。
&emsp;&emsp;&emsp;• Redis 检查内存使用情况，如果大于 maxmemory 限制，根据策略来回收键。
&emsp;&emsp;&emsp;• 一个新的命令被执行，如此等等。
&emsp;&emsp;我们通过检查，然后回收键以返回到限制以下，来连续不断的穿越内存限制的边界。
&emsp;&emsp;如果一个命令导致大量的内存被占用 (像一个很大的集合交集保存到一个新的键)，一会功夫内存限制就会被这个明显的内存量所超越。


参考文章：
[Redis 3.0 中文版](http://wiki.jikexueyuan.com/project/redis-guide/)