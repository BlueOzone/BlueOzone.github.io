---
title: Redis初识(基本理论)
categories: Redis
tags: Redis技术
toc: true
---

本篇会介绍关于Redis的基本知识，关于Redis的介绍和特点等，后面一篇将会讲解Redis的稍微高级的内容，比如集群和实际工作中的运维命令。Redis是比较热门的数据库，所以我觉得学好和用好在工作上的帮助还是挺大的。
<!--more-->

### Redis是什么
来看一下官网的介绍：Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。
Redis全称：REmote DIctionary Server(远程字典服务器)。是完全开源免费的，用C语言编写的，遵守BSD协议，是一个高性能的(key/value)分布式内存数据库，基于内存运行，并支持持久化的NoSQL数据库，是当前最热门的NoSQL数据库之一，也被人们称为数据结构服务器。

### Redis的特点与功能
Redis的特点可能比较明显，大概分为三点，可以围绕数据库大都支持的持久化、数据备份和本身的数据结构来展开。
特点一：Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载到内存中继续使用，Redis的持久化有两种方式，后面会介绍到。
特点二：Redis支持五大数据类型的数据，具体是key-value、list、set、hash、zset，Redis都支持这几类数据结构的存储。
特点三：Redis支持数据的备份，即master-slave模式的数据备份。

Redis的功能：
内存存储和持久化：Redis支持异步将内存中的数据写到硬盘上，同时不影响继续服务。
功能和特点在Redis上是一致的，Redis的特点，也是他的功能，面试是问到功能可以回答以上三个特点。
Redis在实际生产过程中，可以用来取最新N个数据的操作，比如将最新10条评论的ID放在Redis的List集合里面；也可以模拟类似HttpSession这种需要设定过期时间的功能；也可做定时器、计数器。

### Redis的五大数据类型
前面介绍Redis的特点有说到，Redis支持五大数据类型的数据，具体是key-value、list、set、hash、zset；下面我们就这五大数据类型，还有Redis的key(键)来分别介绍下：

#### key(键)
Redis 键命令用于管理 redis 的键。

关于键的常用操作：
`keys *`：查看所有key，key的数量较大的时候会造成redis阻塞，谨慎使用；\*号符合其实是正则里面的\*，所以keys命令后面可以写正则匹配key。
`exists key`：判断某个key是否存在，返回1代表存在，0代表不存在。
`move key db`：把该key从当前库移除。
`expire key 秒钟`：给这个key设置过期时间。
`ttl key`：查看这个key的生存时间，-1代表用不过期，-2代表已过期。
`type key`：查看这个key的类型。

#### String(字符串)
String是Redis的最基本的类型，一个key对应一个value；他是二进制安全的，意思就是Redis的String可以包含任何数据，例如jpg图片或者序列化的对象；Redis中一个字符串的value最多可以是512M。

关于字符串的常用操作：
`SET key value`：设置指定 key 的值。
`GET key`：获取指定 key 的值。
`GETRANGE key start end`：返回 key 中字符串值的子字符
`GETSET key value`：将给定 key 的值设为 value ，并返回 key 的旧值(old value)。
`GETBIT key offset`：对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
`MGET key1 [key2..]`：获取所有(一个或多个)给定 key 的值。
`SETBIT key offset value`：对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
`SETEX key seconds value`：将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
`SETNX key value`：只有在 key 不存在时设置 key 的值。
`SETRANGE key offset value`：用 value 参数覆写给定 key 所储存的字符串值，从偏移量 offset 开始。
`STRLEN key`：返回 key 所储存的字符串值的长度。
`MSET key value [key value ...]`：同时设置一个或多个 key-value 对。
`MSETNX key value [key value ...]`：同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。
`PSETEX key milliseconds value`：这个命令和 SETEX 命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像 SETEX 命令那样，以秒为单位。
`INCR key`：将 key 中储存的数字值增一。
`INCRBY key increment`：将 key 所储存的值加上给定的增量值（increment） 。
`INCRBYFLOAT key increment`：将 key 所储存的值加上给定的浮点增量值（increment） 。
`DECR key`：将 key 中储存的数字值减一。
`DECRBY key decrement`：key 所储存的值减去给定的减量值（decrement） 。
`APPEND key value`：如果 key 已经存在并且是一个字符串， APPEND 命令将指定的 value 追加到该 key 原来值（value）的末尾。


#### List(列表)
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部(左边)或者尾部(右边)。它的底层实际是个链表。一个列表最多可以包含 2的32次方 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

关于List的常用操作：
`BLPOP key1 [key2 ] timeout`：移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
`BRPOP key1 [key2 ] timeout`：移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
`BRPOPLPUSH source destination timeout`：从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。
`LINDEX key index`：通过索引获取列表中的元素。
`LINSERT key BEFORE|AFTER pivot value`：在列表的元素前或者后插入元素。
`LLEN key`：获取列表长度。
`LPOP key`：移出并获取列表的第一个元素。
`LPUSH key value1 [value2]`：将一个或多个值插入到列表头部。
`LPUSHX key value`：将一个值插入到已存在的列表头部。
`LRANGE key start stop`：获取列表指定范围内的元素。
`LREM key count value`：移除列表元素。
`LSET key index value`：通过索引设置列表元素的值。
`LTRIM key start stop`：对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。
`RPOP key`：移除列表的最后一个元素，返回值为移除的元素。
`RPOPLPUSH source destination`：移除列表的最后一个元素，并将该元素添加到另一个列表并返回。
`RPUSH key value1 [value2]`：在列表中添加一个或多个值。
`RPUSHX key value`：为已存在的列表添加值。


#### set(集合)
Redis 的 Set 是 String 类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

关于set的常用操作：
`SADD key member1 [member2]`
向集合添加一个或多个成员
`SCARD key`
获取集合的成员数
`SDIFF key1 [key2]`
返回第一个集合与其他集合之间的差异。
`SDIFFSTORE destination key1 [key2]`
返回给定所有集合的差集并存储在 destination 中
`SINTER key1 [key2]`
返回给定所有集合的交集
`SINTERSTORE destination key1 [key2]`
返回给定所有集合的交集并存储在 destination 中
`SISMEMBER key member`
判断 member 元素是否是集合 key 的成员
`SMEMBERS key`
返回集合中的所有成员
`SMOVE source destination member`
将 member 元素从 source 集合移动到 destination 集合
`SPOP key`
移除并返回集合中的一个随机元素
`SRANDMEMBER key [count]`
返回集合中一个或多个随机数
`SREM key member1 [member2]`
移除集合中一个或多个成员
`SUNION key1 [key2]`
返回所有给定集合的并集
`SUNIONSTORE destination key1 [key2]`
所有给定集合的并集存储在 destination 集合中
`SSCAN key cursor [MATCH pattern] [COUNT count]`
迭代集合中的元素


#### zset(有序集合)
Redis 有序集合和集合一样也是 string 类型元素的集合,且不允许重复的成员。
**不同的是每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。**
有序集合的成员是唯一的,但分数(score)却可以重复。
集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

关于zset的常用操作：
ZADD key score1 member1 [score2 member2]
向有序集合添加一个或多个成员，或者更新已存在成员的分数
ZCARD key
获取有序集合的成员数
ZCOUNT key min max
计算在有序集合中指定区间分数的成员数
ZINCRBY key increment member
有序集合中对指定成员的分数加上增量 increment
ZINTERSTORE destination numkeys key [key ...]
计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 destination 中
ZLEXCOUNT key min max
在有序集合中计算指定字典区间内成员数量
ZRANGE key start stop [WITHSCORES]
通过索引区间返回有序集合指定区间内的成员
ZRANGEBYLEX key min max [LIMIT offset count]
通过字典区间返回有序集合的成员
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT]
通过分数返回有序集合指定区间内的成员
ZRANK key member
返回有序集合中指定成员的索引
ZREM key member [member ...]
移除有序集合中的一个或多个成员
ZREMRANGEBYLEX key min max
移除有序集合中给定的字典区间的所有成员
ZREMRANGEBYRANK key start stop
移除有序集合中给定的排名区间的所有成员
ZREMRANGEBYSCORE key min max
移除有序集合中给定的分数区间的所有成员
ZREVRANGE key start stop [WITHSCORES]
返回有序集中指定区间内的成员，通过索引，分数从高到低
ZREVRANGEBYSCORE key max min [WITHSCORES]
返回有序集中指定分数区间内的成员，分数从高到低排序
ZREVRANK key member
返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序
ZSCORE key member
返回有序集中，成员的分数值
ZUNIONSTORE destination numkeys key [key ...]
计算给定的一个或多个有序集的并集，并存储在新的 key 中
ZSCAN key cursor [MATCH pattern] [COUNT count]
迭代有序集合中的元素（包括元素成员和元素分值）



#### hash(哈希)
Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。
Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

关于hash的常用操作：
`HDEL key field1 [field2]`：删除一个或多个哈希表字段。
`HEXISTS key field`：查看哈希表 key 中，指定的字段是否存在。
`HGET key field`：获取存储在哈希表中指定字段的值。
`HGETALL key`：获取在哈希表中指定 key 的所有字段和值。
`HINCRBY key field increment`：为哈希表 key 中的指定字段的整数值加上增量 increment 。
`HINCRBYFLOAT key field increment`：为哈希表 key 中的指定字段的浮点数值加上增量 increment。
`HKEYS key`：获取所有哈希表中的字段。
`HLEN key`：获取哈希表中字段的数量。
`HMGET key field1 [field2]`：获取所有给定字段的值。
`HMSET key field1 value1 [field2 value2 ]`：同时将多个 field-value (域-值)对设置到哈希表 key 中。
`HSET key field value`：将哈希表 key 中的字段 field 的值设为 value 。
`HSETNX key field value`：只有在字段 field 不存在时，设置哈希表字段的值。
`HVALS key`：获取哈希表中所有值。
`HSCAN key cursor [MATCH pattern] [COUNT count]`：迭代哈希表中的键值对。

### CAP原则
**CAP**
C：**Consistency（一致性）**
在分布式环境中，一致性是指数据在多个副本之间是否能够保持一致的特性（这点跟ACID中的一致性含义不同）。
对于一个将数据副本分布在不同节点上的分布式系统来说，如果对第一个节点的数据进行了更新操作并且更新成功后，却没有使得第二个节点上的数据得到相应的更新，于是在对第二个节点的数据进行读取操作时，获取的依然是更新前的数据（称为脏数据），这就是典型的分布式数据不一致情况。在分布式系统中，如果能够做到针对一个数据项的更新操作执行成功后，所有的用户都能读取到最新的值，那么这样的系统就被认为具有强一致性（或严格的一致性）。

---

A：**Availability（可用性）**
可用性是指系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果，如果超过了这个时间范围，那么系统就被认为是不可用的。
『有限的时间内』是一个在系统设计之初就设定好的运行指标，不同的系统会有很大的差别。比如对于一个在线搜索引擎来说，通常在0.5秒内需要给出用户搜索关键词对应的检索结果。而对应Hive来说，一次正常的查询时间可能在20秒到30秒之间。
『返回结果』是可用性的另一个非常重要的指标，它要求系统在完成对用户请求的处理后，返回一个正常的响应结果。正常的响应结果通常能够明确地反映出对请求的处理结果，及成功或失败，而不是一个让用户感到困惑的返回结果。
让我们再来看看上面提到的在线搜索引擎的例子，如果用户输入指定的搜索关键词后，返回的结果是一个系统错误，比如"OutOfMemoryErroe"或"System Has Crashed"等提示语，那么我们认为此时系统是不可用的。

---

P：**Partition tolerance**（分区容错性）

分区容错性要求一个分布式系统需要具备如下特性：分布式系统在遇到任何网络分区故障的时候，仍然能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。
网络分区是指在分布式系统中，不同的节点分布在不同的子网络（机房或异地网络等）中，由于一些特殊的原因导致这些子网络之间出现网络不连通的状况，但各个子网络的内部网络是正常的，从而导致整个系统的网络环境被切分成了若干个孤立的区域。

---

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。
因此，根据CAP原理将NoSQL数据库分成了满足CA原则、满足CP原则和满足AP原则三大类：
CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
CP - 满足一致性，分区容错性的系统，通常性能不是特别高。
AP - 满足可用性，分区容错性的系统，通常可能对一致性的要求低一些，满足最终一致性即可。

---

**Consistency 和 Availability 的矛盾**
一致性和可用性，为什么不可能同时成立？答案很简单，因为可能通信失败（即出现分区容错）。
假设数据库一和数据库二同时为客户端服务，这两个库的数据一致。

如果保证 数据库二 的一致性，那么数据库一必须在写操作时，锁定 数据库二 的读操作和写操作。只有数据同步后，才能重新开放读写。锁定期间，数据库二 不能读写，没有可用性不。
如果保证 数据库二 的可用性，那么势必不能锁定 数据库二，所以一致性不成立。
综上所述，数据库二 无法同时做到一致性和可用性。系统设计时只能选择一个目标。如果追求一致性，那么无法保证所有节点的可用性；如果追求所有节点的可用性，那就没法做到一致性。


---

综上所述，由于网络的原因，肯定会出现延迟和丢包等问题，所以：
**分区容错性是我们必须要实现的。**

所以只能在一致性和可用性之间进行权衡，没有NoSQL系统能同时保证这三点。
CA 传统的Oracle数据库
AP 大多数网站架构的选择
CP Redis、MongoDB


### BASE原则
**BASE**
BASE是Basically Available(基本可用）、Soft state(软状态）和Eventually consistent(最终一致性）三个短语的简写。BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方法来使系统达到最终一致性。接下来，我们着重对BASE中的三要素进行讲解。

**基本可用**
基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性——但请注意，这绝不等价于系统不可用。一下就是两个"基本可用"的例子。

响应时间上的损失：正常情况下，一个在线搜索引擎需要在0.5秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1~2秒。

功能上的损失：正常情况下，在一个电子商务网站（比如淘宝）上购物，消费者几乎能够顺利地完成每一笔订单。但在一些节日大促购物高峰的时候（比如双十一、双十二），由于消费者的购物行为激增，为了保护系统的稳定性（或者保证一致性），部分消费者可能会被引导到一个降级页面，如下：


**软状态**
软状态是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同的数据副本之间进行数据同步的过程存在延时。

**最终一致性**
最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性。

最终一致性是一种特殊的弱一致性：系统能够保证在没有其他新的更新操作的情况下，数据最终一定能够达到一致的状态，因此所有客户端对系统的数据访问都能够获取到最新的值。同时，在没有发生故障的前提下，数据到达一致状态的时间延迟，取决于网络延迟、系统负载和数据复制方案设计等因素。

在实际工程实践中，最终一致性存在一下五类主要变种。

因果一致性(Causal consistency)

读己之所写(Read your writes)

会话一致性(Session consistency)

单调读一致性(Monotonic read consistency)

单调写一致性(Monotonic write consistency)

以上就是最终一致性的五种常见的变种，在实际系统实践中，可以将其中的若干个变种互相结合起来，以构建一个具有最终一致性特性的分布式系统。事实上，最终一致性并不是只有那些大型分布式系统才涉及的特性，许多现代的关系型数据库都采用了最终一致性模型。在现代关系型数据库中（比如MySQL和PostgreSQL），大多都会采用同步或异步方式来实现主备数据复制技术。在同步方式中，数据的复制过程通常是更新事务的一部分，因此在事务完成后，主备数据库的数据就会达到一致。而在异步方式中，备库的更新往往会存在延时，这取决于事务日志在主备数据库之间传输的时间长短。如果传输时间过长或者甚至在日志传输过程中出现异常导致无法及时将事务应用到备库上，那么很显然，从备库中读取的数据将是旧的，因此就出现了数据不一致的情况。当然，无论是采用多次重试还是人为数据订正，关系型数据库还是能够保证最终数据达到一致，这就是系统提供最终一致性保证的经典案例。

### redis数据持久化
Redis 提供了不同级别的持久化方式:

RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。
AOF持久化方式记录每次对服务器写的操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾。Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式。
你也可以同时开启两种持久化方式，在这种情况下, 当redis重启的时候会**优先载入AOF文件来恢复原始的数据**，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
最重要的事情是了解RDB和AOF持久化方式的不同，让我们以RDB持久化方式开始：

**RDB(Redis DataBase)**
RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。
Redis会单独创建(fork)一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

redis.conf的配置文件中，关于SNAPSHOTTING模块的全是RDB的相关配置。官方给出的配置文件中有这几种：
`save 900 1`：若900S(15min)内数据库改动了一次，则触发快照条件。
`save 300 10`：若300S(5min)内数据库改动了十次，则触发快照条件。
`save 60 10000`：若60S(1min)内数据库改动了一万次，则触发快照条件。
若不想使用持久化方式，可以配置成`save ""`。
**注意**：
手动执行`save`或`bgsave`命令也会触发快照条件生成dump.rdb文件。
`save`：执行save命令时redis只管保存，其他不管，全部进入阻塞状态。
`bgsave`：Redis会在后台异步进行快照操作，快照的同时还可以响应客户端请求。可以通过`lastsave`命令获取最后一次成功执行快照的时间。
执行`flushall`命令，也会产生dump.rdb文件，但里面是空的，毫无意义。

**RDB持久化方式恢复数据：**
将RDB产生的备份文件(dum.rdb)移动到redis安装目录并启动redis服务即可。

**RDB的优点**
RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份。
RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复。
RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以RDB持久化方式可以最大化redis的性能。
与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些。

**RDB的缺点**
如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你。
RDB 需要经常fork子进程来保存数据集到硬盘上,当数据集比较大的时候,fork的过程是非常耗时的,可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度。

---

**AOF(Appendonly File)**
AOF持久化方式**用日志的形式记录每次对服务器写的操作**，当服务重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以redis协议追加保存每次写的操作到文件末尾。Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。

redis.conf的配置文件中，关于APPEND ONLY MODE模块的全是AOF的相关配置。官方给出的配置文件中默认是不开启的：
`appendonly no`：想要开启AOF持久化方式，修改成yes即可。

`appendfsync always`：同步持久化，每次发生数据变更会立即记录到磁盘，性能较差但数据完整性比较好。
`appendfsync everysec`：出厂默认配置，异步操作，每秒记录；如果发生宕机最多丢失一秒的数据。
`appendfsync no`：从不主动同步。


**AOF持久化方式恢复数据：**
AOF会产生文件(appendonly.aof)，redis启动时会重新执行这些命令来恢复原始的数据。

**如何选择使用哪种持久化方式？**
一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。
如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化。
有很多用户都只使用 AOF 持久化， 但我们并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快， 除此之外， 使用 RDB 还可以避免之前提到的 AOF 程序的 bug 。

### redis事务
Redis的事务可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，**按顺序地串行化执行而不会被其他命令插入，不许加塞**。
以下摘自官网的解释：

>事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

>事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。(注意，这里只是说执行的这个动作，即使事务中有某条/某些命令执行时失败了， 事务队列中的其他命令仍然会继续执行 —— Redis 不会停止执行事务中的命令。)

事务的开启：
`MULTI`：输入该命令开启事务。

命令的入队：
将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面

事务的执行：
按顺序执行事务队列里面的命令，并返回执行结果。

关于事务的常用操作：
`DISCARD`：取消事务，放弃执行事务块内的所有命令。
`EXEC`：执行所有事务块内的命令。
`MULTI`：标记一个事务块的开始。
`UNWATCH`：取消 WATCH 命令对所有 key 的监视。
`WATCH key [key ...]`：监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

通常事务要配合watch和unwatch命令使用。这里的事务都是基于单机模式下的事务说明，关于集群的话事务的结果和操作有所不同。

### Redis复制
Redis复制，也就是常说的主从复制，主机数据更新后根据配置和策略，自动会以异步复制的方式，同步数据到备机的机制；master以写为主，slave以读为主。

Redis复制能用你做读写分离、容灾恢复。

Redis复制操作：
`slaveof ip port`：输入该命令后，ip所在机器的Redis将会成为主机，本机成为从机。
`slaveof no one`：使本机成为主数据库，也就结束了与其他数据库的同步。

以下摘自官网：
系统的运行依靠三个主要的机制：

- 当一个 master 实例和一个 slave 实例连接正常时， master 会发送一连串的命令流来保持对 slave 的更新，以便于将自身数据集的改变复制给 slave ， ：包括客户端的写入、key 的过期或被逐出等等。

- 当 master 和 slave 之间的连接断开之后，因为网络问题、或者是主从意识到连接超时， slave 重新连接上 master 并会尝试进行部分重同步：这意味着它会尝试只获取在断开连接期间内丢失的命令流。

- 当无法进行部分重同步时， slave 会请求进行全量重同步。这会涉及到一个更复杂的过程，例如 master 需要创建所有数据的快照，将之发送给 slave ，之后在数据集更改时持续发送命令流到 slave 。

---

接下来的是一些关于 Redis 复制的非常重要的事实：

- Redis 使用异步复制，slave 和 master 之间异步地确认处理的数据量

- 一个 master 可以拥有多个 slave

- slave 可以接受其他 slave 的连接。除了多个 slave 可以连接到同一个 master 之外， slave 之间也可以像层叠状的结构（cascading-like structure）连接到其他 slave 。自 Redis 4.0 起，所有的 sub-slave 将会从 master 收到完全一样的复制流。

- Redis 复制在 master 侧是非阻塞的。这意味着 master 在一个或多个 slave 进行初次同步或者是部分重同步时，可以继续处理查询请求。

- 复制在 slave 侧大部分也是非阻塞的。当 slave 进行初次同步时，它可以使用旧数据集处理查询请求，假设你在 redis.conf 中配置了让 Redis 这样做的话。否则，你可以配置如果复制流断开， Redis slave 会返回一个 error 给客户端。但是，在初次同步之后，旧数据集必须被删除，同时加载新的数据集。 slave 在这个短暂的时间窗口内（如果数据集很大，会持续较长时间），会阻塞到来的连接请求。自 Redis 4.0 开始，可以配置 Redis 使删除旧数据集的操作在另一个不同的线程中进行，但是，加载新数据集的操作依然需要在主线程中进行并且会阻塞 slave 。

- 复制既可以被用在可伸缩性，以便只读查询可以有多个 slave 进行（例如 O(N) 复杂度的慢操作可以被下放到 slave ），或者仅用于数据安全。

- 可以使用复制来避免 master 将全部数据集写入磁盘造成的开销：一种典型的技术是配置你的 master Redis.conf 以避免对磁盘进行持久化，然后连接一个 slave ，其配置为不定期保存或是启用 AOF。但是，这个设置必须小心处理，因为重新启动的 master 程序将从一个空数据集开始：如果一个 slave 试图与它同步，那么这个 slave 也会被清空。

### Redis哨兵模式

Sentinel(哨兵)是用于监控redis集群中Master状态的工具，是Redis 的高可用性解决方案，sentinel哨兵模式已经被集成在redis2.4之后的版本中。
sentinel是redis高可用的解决方案，sentinel系统可以监视一个或者多个redis master服务，以及这些master服务的所有从服务；当某个master服务下线时，自动将该master下的某个从服务升级为master服务替代已下线的master服务继续处理请求。

sentinel可以让redis实现主从复制，当一个集群中的master失效之后，sentinel可以选举出一个新的master用于自动接替master的工作，集群中的其他redis服务器自动指向新的master同步数据。一般建议sentinel采取奇数台，防止某一台sentinel无法连接到master导致误切换。

哨兵的启动和监控：
在Redis的目录下有sentinel.conf这个配置文件，可以配置监控的具体信息。
`sentinel monitor 被监控数据库名字(随便填) 127.0.0.1(被监控数据库ip) 6379(端口) 2(投票数，多于这个值即可成为主机)`
