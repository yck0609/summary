# 1 Redis（Remote Dictionary Server，远程字典服务）

> Redis是 C 语言编写的基于内存亦可持久化的高性能<mark>非关系型</mark>键值对数据库

## 1.2 Redis优缺点

### 1.2.1 优点

> 1. 基于内存操作，内存读写速度快
> 
> 2. 支持多种数据类型，包括String、Hash、List、Set、ZSet等
> 
> 3. 支持持久化。Redis支持RDB和AOF两种持久化机制，有效地避免数据丢失问题
> 
> 4. 支持事务。Redis的所有操作都是原子性的，支持对几个操作合并后的原子性执行
> 
> 5. 支持主从复制。主节点会自动将数据同步到从节点，可以进行读写分离
> 
> 6. 单线程处理。Redis6.0引入了多线程，多线程用于处理网络数据的读写和协议解析，Redis命令执行还是单线程的

### 1.2.2 缺点

> 1. 对结构化查询的支持比较差
> 
> 2. 数据库容量受到物理内存的限制，不适合用作海量数据的高性能读写，局限在较小数据量的操作
> 
> 3. 难以在线扩容，在集群容量达到上限时在线扩容会变得很复杂

## 1.3 Redis应用场景

> 1. 缓存：减轻 MySQL 的查询压力，提升系统性能
> 
> 2. 排行榜：利用 Redis 的 SortSet （有序集合）实现
> 
> 3. 计数器：利用 Redis 中原子性的自增操作，我们可以统计类似用户点赞数、用 户访问数等。这类操作如果用 MySQL，频繁的读写会带来相当大的压力
> 
> 4. 限速器：限制用户访问某个 API 的频率，常用于抢购时，防止用户疯狂点击带来不必要的压力
> 
> 5. 好友关系：利用集合的一些命令，比如求交集、并集、差集等。可以方便解决一些共同好友、共同爱好之类的功能
> 
> 6. 消息队列：可以使用Redis自身的发布/订阅模式或者List来实现简单的消息队列，实现异步操作，比如：到货通知、邮件发送之类的需求
> 
> 7. Session 共享：Session 是保存在服务器的文件中，如果是集群服务，同一个用户过来 可能落在不同机器上，这就会导致用户频繁登陆；采用 Redis 保存 Session 后，无论用户落在那台机器上都能够获取到对应的 Session 信息
> 
> 8. 分布式锁。在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现

# 2 数据库基本结构

## 2.1 数据库的切换

> Redis服务器将所有数据库都保存在服务器状态`redis.h/redisServer`结构的db数组中，db数组的每个项都是一个`redis.h/redisDb`结构，每个redisDb结构代表一个数据库

```C
struct redisServer 
{ 
    // ... 
    redisDb *db; 
    int dbnum;
    // ...
};
```

初始化时，程序会根据当前服务器的`dbnum`属性来决定建立数据库的个数，默认创建16个。每个Redis客户端都有自己的目标数据库，当客户端执行读写命令时，就需要切换数据库。默认情况下，Redis客户端的目标数据库为0号数据库，但客户端可以通过执行SELECT命令来切换目标数据库

```shell
redis> SELECT 2
OK
```

在服务器内部，客户端状态redisClient结构的db属性记录了客户端当前的目标数据库，这个属性是一个指向redisDb结构的指针：

```C
typedef struct redisClient 
{
    // ...
    redisDb *db; //记录客户端当前正在使用的数据库
    // ...
} redisClient;
```

## 2.2 键空间（key space）

### 2.2.1 键空间结构

Redis是一个键值对数据库服务器，每个数据库都是一个redis.h/redisDb结构。其中`dict`字典保存了数据库中所有的键值对，我们将这个字典称为键空间

```C
typedef struct redisDb 
{ // ... 
    dict *dict; // 数据库键空间，保存着数据库中的所有键值对 
    // ...
} redisDb;
```

键空间的键就是数据库的键，每个键是一个**字符串对象**。键空间的值就是数据库的值，可以是字符串对象、列表对象、哈希表对象、集合对象和有序集合对象中的一种

```C
redis> SE message "hello world"
OK

redis> RPUSH alphabet "a" "b" "c"
(integer)3

redis> HSET book name "Redis in Action"
(integer) 1

redis> HSET book author "Josiah L. Carlson"
(integer) 1

redis> HSET book publisher "Manning"
(integer) 1
```

数据库的键空间结构如下：

<img src="https://bucket-1259555870.cos.ap-chengdu.myqcloud.com/20200104105103.png"  style="zoom:75%;display: block; margin: 0px auto; vertical-align: middle;"> 

### 2.2.2 设置过期时间

通过**EXPIRE命令**或者**PEXPIRE命令**，**客户端**可以以**秒**或者**毫秒**精度某个键设置**生存时间（Time To Live，TTL）**，在经过指定的秒数或者毫秒数之后，服务器就会**自动删除生存时间为0的键**：

```C
redis> SET key value
OK

redis> EXPIRE key 5
(integer) 1

redis> GET key // 5秒之内
"value"
redis> GET key // 5秒之后
(nil)
```

与前面相似，客户端可以通过**EXPIREAT命令**或**PEXPIREAT命令**，以秒或者毫秒精度给数据库中的某个键设置**过期时间（expire time）**。过期时间由UNIX时间戳表示。

而**TTL命令**和**PTTL命令**则返回**一个键的剩余生存时间。**

所有的命令在Redis中**最终都会转化为PEXPIREAT**执行。

在RedisDb结构中，**在键空间之外**，有一个expires字典专门保存所有键的过期时间，我们称之为**过期字典**。过期字典保存的值是long long 类型整数，**保存一个毫秒精度的UNIX时间戳**。

```C
typedef struct redisDb 
{ 
    // ...  
    dict *expires; // 过期字典，保存着键的过期时间
    // ...
} redisDb;
```

虽然键空间和过期时间都有相同的键，但他们以指针形式指向同一个键，不会造成空间浪费。

### 2.2.3 过期键的删除策略

**（1）定时删除**

在设置键的过期时间的同时，创建一个定时器（timer），定时结束后删除。

定时删除**有利于内存管理**，但**对CPU不友好**。如果过期键太多，删除会占用相当一部分CPU。

所以策略应该是：当有大量命令请求服务器处理时，并且服务器内存充足，就应该优先将CPU资源安排在处理客户端请求上，而不是删除过期键。

创建一个定时器需要用到Redis服务器中的**时间事件**，而当前时间事件的实现方式——无序链表，查找一个事件的时间复杂度为$O(N)$，并**不能高效地处理大量时间事件**。

**（2）惰性删除**

放着不管，每次从键空间获取时检查是否过期，过期就删除。

**对CPU最友好**，**但浪费内存**。如果数据库中有很多过期键，而这些过期键永远也不会被访问的话，他们就会永远占据空间，可视为**内存泄漏**。

一些和时间有关的数据，比如日志，在某个时间点后，他们的访问就会很少。如果这类过期数据大量积压，会造成严重的内存浪费。

存在于`db.c/expireIfNeeded`函数。**所有读写数据库的Redis命令在执行之前都会调用`expireIfNeeded`函数对输入键进行检查**：

- 过期，函数将输入键删除
- 不过期，函数不动作

**（3）定期删除**

每隔一段时间，程序检查一次数据库，删除过期键。

定期删除是一种折中，通过选择较为空闲的时间点来处理过期键，减少CPU压力。同时也能及时释放内存，避免内存泄漏。

过期键的定期删除策略由`redis.c/activeExpireCycle`函数实现，**每当Redis的服务器周期性操作`redis.c/serverCron`函数执行时**，`activeExpireCycle`函数就会被调用，它在规定的时间内，**分多次遍历服务器中的各个数据库**，从数据库的expires字典中**随机检查一部分键的过期时间**，并删除其中的过期键。

全局变量`current_db`会**记录当前`activeExpireCycle`函数检查的进度**，并在**下一次检查时接着上一次的进度进行处理**。比如说，如果当前`activeExpireCycle`函数在遍历10号数据库时返回了，那么下次就会从11号数据库开始工作。

如果所有数据库都被检查了一遍，则`current_db`将会被置0，然后开始新一轮检查。

## 2.3 数据库通知

通知可以让客户端通过订阅给定的频道或者模式，来获知数据库中键的变化，以及数据库中命令的执行情况。

### 2.4.1 订阅通知

订阅有两种模式：

- 订阅某一个键，返回键的所有操作
- 订阅某一个操作，返回执行这个操作的键

情况1，从0号数据库订阅了键message的消息。如果此时有其他客户端操作了message，则会将消息通知到此处。

```C
127.0.0.1:6379> SUBSCRIBE _ _keyspace@0_ _:message
Reading messages... (press Ctrl-C to quit)

1) "subscribe" // 订阅信息
2) "__keyspace@0__:message"
3) (integer) 1

1) "message" //执行SET命令
2) "_ _keyspace@0_ _:message"
3) "set"

1) "message" //执行EXPIRE命令
2) "_ _keyspace@0_ _:message"
3) "expire"
```

情况2，客户端订阅了0号数据库中的DEL命令。

```C
127.0.0.1:6379> SUBSCRIBE _ _keyevent@0_ _:del
Reading messages... (press Ctrl-C to quit)
1) "subscribe" // 订阅信息
2) "_ _keyevent@0_ _:del"
3) (integer) 1

1) "message" //键key执行了DEL命令
2) "_ _keyevent@0_ _:del"
3) "key"

1) "message" //键number执行了DEL命令
2) "_ _keyevent@0_ _:del"
3) "number"
```

### 2.4.2 发送通知

发送数据库通知的功能是由`notify.c/notifyKeyspaceEvent`函数实现，函数声明如下：

```C
void notifyKeyspaceEvent(int type,char *event,robj *key,int dbid);
```

**type参数是发送的通知的类型，event、keys和dbid分别是事件的名称、产生事件的键，以及产生事件的数据库编号**，函数会根据type参数以及这三个参数来构建事件通知的内容，以及接收通知的频道名。

比如SADD命令的实现函数中，通知的发送方式是

```C
void saddCommand(redisClient* c)
{
    //...
    if(added)
    {
        //...添加成功，发送通知
           notifyKeyspaceEvent(REDIS_NOTIFY_SET,"add",c->argv[1],c->db->id);
        //...
    }
}
```

当SADD命令成功地向集合添加了一个集合元素之后，命令就会发送通知，**该通知的类型为REDIS_NOTIFY_SET（表示这是一个集合键通知）**，名称为sadd（表示这是执行SADD命令所产生的通知）。

发布时调用的`notifyKeyspaceEvent`函数逻辑是：

1. 检查服务器是否允许发送此类通知，如果不允许就返回
2. 是否允许发送**键空间通知**（4.1提到的情况1），允许就发送
3. 是否允许发送**键事件通知**（4.2提到的情况2），允许就发送 

# 3 Redis 数据类型

| 数据类型        | 可以存储的值      | 操作                                                 |
|:-----------:| ----------- | -------------------------------------------------- |
| STRING      | 字符串、整数或者浮点数 | 对整个字符串或者字符串的其中一部分执行操作对整数和浮点数执行自增或者自减操作             |
| LIST        | 列表          | 从两端压入或者弹出元素对单个或者多个元素进行修剪，只保留一个范围内的元素               |
| SET         | 无序集合        | 添加、获取、移除单个元素，检查一个元素是否存在于集合中，计算交集、并集、差集，从集合里面随机获取元素 |
| HASH        | 包含键值对的无序散列表 | 添加、获取、移除单个键值对，获取所有键值对，检查某个键是否存在                    |
| ZSET        | 有序集合        | 添加、获取、删除元素，根据分值范围或者成员来获取元素，计算一个键的排名                |
| GEO         |             |                                                    |
| HyperLogLog |             |                                                    |
| BitMap      |             |                                                    |
| BitField    |             |                                                    |
| Stream      |             |                                                    |

## 3.1 字符串

### 3.1.1 简单动态字符串(simple dynamic string, SDS)

> 简单动态字符串是Redis的默认字符串表示，定义在`sds.h/sdshdr`中

```c
struct sdshdr{
   int len; // len 保存了SDS保存字符串的长度，记录buf数组中已使用字节的数量
   int free; // 记录 buf 数组中未使用的字节数量
   char buf[]; // buf[] 数组用来保存字符串的每个元素
}
```

### 3.1.2 C字符串的缺点

> C字符串获取长度的能力有限 ：C字符串需要依靠遍历获取长度，时间复杂度$O(N)$
> 
> 杜绝缓冲区溢出 ：由于C字符串不记录长度，当我们拼接两个字符串的时候，容器可能因为空间不足发生溢出
> 
> 减少修改字符串时带来的内存重分配次数 ：C字符串类似于数组，每次修改大小都会重新分配以此内存
> 
> 二进制安全 ：C字符串以空字符`\0`结尾，使得 C 字符串只能保存文本数据， 而不能保存像图片、音频、视频、压缩文件这样的二进制数据

### 3.1.3 SDS优势

> 常数复杂度获取字符串长度 ：由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。通过 strlen key 命令可以获取 key 的字符串长度。
> 
> 杜绝缓冲区溢出 ：我们知道在 C 语言中使用 strcat 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，会首先根据记录的 len 属性检查内存空间是否满足需求，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。
> 
> 减少修改字符串的内存重新分配次数 ：C语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。而对于SDS，由于len属性和free属性的存在，对于修改字符串SDS实现了空间预分配和惰性空间释放两种策略：
> 
> 1、空间预分配：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
> 
> 2、惰性空间释放：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 free 属性将这些字节的数量记录下来，等待后续使用。（当然SDS也提供了相应的API，当我们有需要时，也可以手动释放这些未使用的空间。）
> 
> 二进制安全 ：因为C字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此C字符串无法正确存取；而所有 SDS 的API 都是以处理二进制的方式来处理 buf 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束。
> 
> 兼容部分 C 字符串函数 ：虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以重用 C 语言库<string.h> 中的一部分函数。

## 3.2 链表（list）

> Redis自定义链表数据结构，定义`adlist.h/listNode`以及`adlist.h/list` 来持有链表

### 3.2.1 listNode

```c
typedef struct listNode {
    struct listNode *prev; // 前置节点
    struct listNode *next; // 后置节点
    void *value; // 节点的值
} listNode;
```

### 3.2.2 list

```c
typedef struct list {
    listNode *head; // 表头节点
    listNode *tail; // 表尾节点
    unsigned long len; // 链表所包含的节点数量

    void *(*dup)(void *ptr); // 节点值复制函数
    void (*free)(void *ptr); // 节点值释放函数
    int (*match)(void *ptr, void *key); // 节点值对比函数
} list;
```

### 3.2.3 list的特点

> 双端：链表具有前置节点和后置节点的引用，获取这两个节点时间复杂度都为O(1)。
> 无环：表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL,对链表的访问都是以 NULL 结束。　　
> 带链表长度计数器：通过 len 属性获取链表长度的时间复杂度为 O(1)。
> 多态：链表节点使用 void* 指针来保存节点值，可以保存各种不同类型的值。

## 3.3 字典（dict）

> Redis自定义哈希表结构，字典又称为符号表或者关联数组、或映射（map），是一种用于保存键值对的抽象数据结构。字典中的每一个键 key 都是唯一的，通过 key 可以对值来进行查找或修改。哈希表是由数组 table 组成，table 中每个元素都是指向 dict.h/dictEntry 结构，dictEntry 结构定义如下：

### 3.3.1 哈希表节点（dictEntry）

>  每个 `dictEntry` 结构都保存着一个键值对，`v` 属性则保存着键值对中的值， 值可以是一个指针， 或者是一个 `uint64_t` 整数， 又或者是一个 `int64_t` 整数。 `next` 属性是指向另一个哈希表节点的指针， 这个指针可以将多个哈希值相同的键值对连接在一次， 以此来**解决键冲突（collision）的问题**。key 用来保存键，val 属性用来保存值，值可以是一个指针，也可以是uint64_t整数，也可以是int64_t整数。注意这里还有一个指向下一个哈希表节点的指针，我们知道哈希表最大的问题是存在哈希冲突，如何解决哈希冲突，有开放地址法和链地址法。这里采用的便是链地址法，通过next这个指针可以将多个哈希值相同的键值对连接在一起，用来解决哈希冲突。

```c
typedef struct dictEntry{
   void *key; //键
   union{ //值
     void *val;
     uint64_tu64;
     int64_ts64;
   }v;
   struct dictEntry *next; //指向下一个哈希表节点，形成链表
}dictEntry
```

### 3.3.2 dictht

```c
typedef struct dictht{
   dictEntry **table; //哈希表数组
   unsigned long size; //哈希表大小
   unsigned long sizemask; //哈希表大小掩码，用于计算索引值，总是等于 size-1
   unsigned long used; //该哈希表已有节点的数量
}dictht
```

`table` 是一个数组， 数组中的每个元素都是一个指向 `dict.h/dictEntry` 结构的指针。

### 3.3.3 dictType

 `dictType` 结构保存了一簇用于操作特定类型键值对的函数。

```C
typedef struct dictType {
    unsigned int (*hashFunction)(const void *key); // 计算哈希值的函数
    void *(*keyDup)(void *privdata, const void *key); // 复制键的函数
    void *(*valDup)(void *privdata, const void *obj); // 复制值的函数
    int (*keyCompare)(void *privdata, const void *key1, const void *key2); // 对比键的函数
    void (*keyDestructor)(void *privdata, void *key); // 销毁键的函数
    void (*valDestructor)(void *privdata, void *obj); // 销毁值的函数
} dictType;
```

### 3.3.4 dict

上面提到的是哈希表和哈希表节点的实现，现在来说真正的**字典结构**。Redis 中的字典由 `dict.h/dict` 结构表示：

```C
typedef struct dict {
    dictType *type; // 类型特定函数
    void *privdata; // 私有数据
    dictht ht[2]; // 哈希表
    int rehashidx; // rehash 索引，当 rehash 不在进行时，值为 -1
} dict;
```

其中`type` 属性和 `privdata` 属性是**针对不同类型的键值对， 为创建多态字典而设置的**。- `privdata` 属性则保存了需要传给那些类型特定函数的可选参数。哈希表数组ht包含了两个元素， 一般情况下， 字典只使用 `ht[0]` 哈希表， `ht[1]` 哈希表只会在对 `ht[0]` 哈希表进行 rehash 时使用。除了 `ht[1]` 之外， 另一个和 rehash 有关的属性就是 `rehashidx` ： 它记录了 rehash 目前的进度， 如果目前没有在进行 rehash ， 那么它的值为 `-1` 。下图展示了一个普通状态下（没有rehash）的字典

### 3.3.5 哈希算法

①、哈希算法：Redis计算哈希值和索引值方法如下：

```java
#1、使用字典设置的哈希函数，计算键 key 的哈希值

hash = dict->type->hashFunction(key);

#2、使用哈希表的sizemask属性和第一步得到的哈希值，计算索引值

index = hash & dict->ht[x].sizemask;
```

　②、解决哈希冲突：这个问题上面我们介绍了，方法是链地址法。通过字典里面的 *next 指针指向下一个具有相同索引值的哈希表节点。

　③、扩容和收缩：当哈希表保存的键值对太多或者太少时，就要通过 rerehash(重新散列）来对哈希表进行相应的扩展或者收缩。具体步骤：

如果执行扩展操作，会基于原哈希表创建一个大小等于 ht[0].used*2n 的哈希表（也就是每次扩展都是根据原哈希表已使用的空间扩大一倍创建另一个哈希表）。相反如果执行的是收缩操作，每次收缩是根据已使用空间缩小一倍创建一个新的哈希表。

重新利用上面的哈希算法，计算索引值，然后将键值对放到新的哈希表位置上。

所有键值对都迁徙完毕后，释放原哈希表的内存空间。

　④、触发扩容的条件：

　　　　　　1、服务器目前没有执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于1。

　　　　　　2、服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令，并且负载因子大于等于5。

　　　　ps：负载因子 = 哈希表已保存节点数量 / 哈希表大小。

程序需要先根据键值对的键计算出**哈希值和索引值**， 然后再根据索引值， 将包含新键值对的哈希表节点放到哈希表数组的指定索引上面。

Redis 计算哈希值和索引值的方法如下：

```C
// 使用字典设置的哈希函数，计算键 key 的哈希值
hash = dict->type->hashFunction(key);

// 使用哈希表的 sizemask 属性和哈希值，计算出索引值
// 根据情况不同， ht[x] 可以是 ht[0] 或者 ht[1]
index = hash & dict->ht[x].sizemask;
```

至于Redis的哈希值计算方法，使用的是 MurmurHash2。这种算法的优点在于， 即使输入的键是有规律的， 算法仍能给出一个很好的随机分布性， 并且算法的计算速度也非常快。

### 3.3.6 Rehash

随着操作的不断执行， 哈希表保存的键值对会逐渐地增多或者减少， 为了让哈希表的**负载因子（load factor）**维持在一个合理的范围之内， **当哈希表保存的键值对数量太多或者太少时， 程序需要对哈希表的大小进行相应的扩展或者收缩。**

再哈希的关键在于**重新分配哈希表的大小**，分配的原则如下：

- 如果执行拓展操作`ht[1]` 的大小为第一个大于等于 `ht[0].used * 2` 的 $2^n$，比如原表大小为4，则 `ht[0].used * 2`结果为8，而8刚好是$2^3$，所以新的大小是8。
- 如果执行的是收缩操作， 那么 `ht[1]` 的大小为第一个大于等于 `ht[0].used` 的$2^ n$

完成分配后，将保存在 `ht[0]` 中的所有键值对 **rehash** 到 `ht[1]` 上面，然后 将 `ht[1]` 设置为 `ht[0]` ， 并在 `ht[1]` 新创建一个空白哈希表， 为下一次 rehash 做准备。

决定是否再Hash的要素来自于负载因子，计算方法如下：

```C
//负载因子 = 哈希表已保存节点数量 / 哈希表大小
load_factor = ht[0].used / ht[0].size
```

### 3.3.7 渐进式Rehash

也就是说扩容和收缩操作不是一次性、集中式完成的，而是分多次、渐进式完成的。如果保存在Redis中的键值对只有几个几十个，那么 rehash 操作可以瞬间完成，但是如果键值对有几百万，几千万甚至几亿，那么要一次性的进行 rehash，势必会造成Redis一段时间内不能进行别的操作。所以Redis采用渐进式 rehash,这样在进行渐进式rehash期间，字典的删除查找更新等操作可能会在两个哈希表上进行，第一个哈希表没有找到，就会去第二个哈希表上进行查找。但是进行 增加操作，一定是在新的哈希表上进行的。

如果键值对很多，则将`ht[0]`重新hash到`ht[1]`上，则会导致服务器在一段时间内停止服务。为了避免这种问题，需要分多次渐进式的慢慢映射。

关键点在于维持一个**索引计数器变量** `rehashidx` ， 并将它的值设置为 `0` ， 表示 rehash 工作正式开始。

在 rehash 进行期间， 每次对字典执行增删改查， 程序除了执行指定的操作以外， **还会顺带将 `ht[0]` 哈希表在 `rehashidx` 索引上的所有键值对 rehash 到 `ht[1]`** ， 当 rehash 工作完成之后， 程序将 `rehashidx` **属性的值增一**。

完成后程序将 `rehashidx` 属性的值设为 `-1` ， 表示 rehash 操作已完成。

渐进式 rehash 的好处在于它采取分而治之的方式， 将 rehash 键值对所需的计算工作均滩到对字典的每个增删改查上， 从而避免了集中式 rehash 而带来的庞大计算量。

## 3.4 跳跃表（skiplist）

> 链表随机读写能力差，当增删改查的时候，如果要找到目标元素就需要遍历链表。假设某个数据结构是有序的，我们就会想到用二分法来快速查找，但**链表是没有索引的**，所以我们需要添加。但是我们的链表不是静态的，增加和删除会破坏二分结构，所以我们就不强制要求 `1:2` 了，一个节点要不要被索引，建几层的索引，都在节点插入时由随机决定。跳跃表是一种有序数据结构，它通过在每个节点中维持多个指向其它节点的指针，从而达到快速访问节点的目的。具有如下性质：
> 
> 1、由很多层结构组成；
> 
> 　　2、每一层都是一个有序的链表，排列顺序为由高层到底层，都至少包含两个链表节点，分别是前面的head节点和后面的nil节点；
> 
> 　　3、最底层的链表包含了所有的元素；
> 
> 　　4、如果一个元素出现在某一层的链表中，那么在该层之下的链表也全都会出现（上一层的元素是当前层的元素的子集）；
> 
> 　　5、链表中的每个节点都包含两个指针，一个指向同一层的下一个链表节点，另一个指向下一层的同一个链表节点；

### 3.4.1 zskiplistNode

Redis中跳跃表节点定义如下：

```c
typedef struct zskiplistNode {
   struct zskiplistLevel{ //层
      struct zskiplistNode *forward; //前进指针
      unsigned ***\*int\**** span; //跨度
   }level[];
   struct zskiplistNode *backward; //后退指针
   double score; //分值
   robj *obj; //成员对象
} zskiplistNode
```

Redis 的跳跃表由 `redis.h/zskiplistNode` 和 `redis.h/zskiplist` 两个结构定义， 其中 `zskiplistNode` 结构用于表示跳跃表**节点**， 而 `zskiplist`结构则用于保存跳跃表节点的相关信息， 比如**节点的数量， 以及指向表头节点和表尾节点的指针**， 等等。

在`zskiplist`中`level` 记录目前跳跃表内最大层数（表头不算），`length`记录包含的节点数量（表头不算）。

`zskiplistNode` 结构包含以下属性：

- 层：每一层有两个属性
  - 前进指针用于访问位于表尾方向的其他节点
  - 跨度则记录了前进指针所指向节点和当前节点的距离。
- 后退指针(bw)：指向位于当前节点的前一个节点。后退指针在程序从表尾向表头遍历时使用。
- 分值(score)：各个节点中的 `1.0` 、 `2.0` 和 `3.0` 是节点所保存的分值。用于从小到大排列。**如果分值相同，则成员对象小的排在前面。**
- 成员对象（obj）：各个节点中的 `o1` 、 `o2` 和 `o3` 是节点所保存的成员对象。

**（1）层**

每次创建一个新跳跃表节点的时候， 程序都根据幂次定律 （[power law](http://en.wikipedia.org/wiki/Power_law)，**越大的数出现的概率越小**） **随机**生成一个介于 `1` 和 `32` 之间的值作为 `level` 数组的大小， 这个大小就是层的“高度”。

**（2）前进指针**

前进指针分属于不同的层，`level[i].forward`，用于从表头向表尾方向访问节点。

**（3）跨度**

跨度也分属不同的层，指向 `NULL` 的所有前进指针的跨度都为 `0`， 因为它们没有连向任何节点。

跨度实际上是用来计算**位次**（rank）的： **将沿途访问过的所有层的跨度累计起来， 得到的结果就是目标节点在跳跃表中的排位。**

### 3.4.2 zskiplist

多个跳跃表节点构成一个跳跃表：

```c
typedef struct zskiplist{
   structz skiplistNode *header, *tail; //表头节点和表尾节点
   unsigned long length; //表中节点的数量
   int level; //表中层数最大的节点的层数
}zskiplist;
```

搜索：从最高层的链表节点开始，如果比当前节点要大和比当前层的下一个节点要小，那么则往下找，也就是和当前层的下一层的节点的下一个节点进行比较，以此类推，一直找到最底层的最后一个节点，如果找到则返回，反之则返回空。

插入：首先确定插入的层数，有一种方法是假设抛一枚硬币，如果是正面就累加，直到遇见反面为止，最后记录正面的次数作为插入的层数。当确定插入的层数k后，则需要将新元素插入到从底层到k层。

删除：在各个层中找到包含指定值的节点，然后将节点从链表中删除即可，如果删除以后只剩下头尾两个节点，则删除这一层。

## 3.5 整数集合

整数集合（intset）是Redis用于保存整数值的集合抽象数据类型，它可以保存类型为int16_t、int32_t 或者int64_t 的整数值，并且保证集合中不会出现重复元素。

整数集合（intset）是 Redis 用于保存整数值的集合抽象数据结构， 它可以保存类型为 `int16_t` 、 `int32_t` 或者 `int64_t` 的整数值， 并且保证集合中不会出现重复元素。

每个 `intset.h/intset` 结构表示一个整数集合：

`contents` 数组是整数集合的底层实现： 整数集合的每个元素都是 `contents` 数组的一个数组项（item）， 从小到大有序地排列，不包含任何重复项。

虽然 `intset` 结构将 `contents` 属性声明为 `int8_t` 类型的数组， 但实际上 `contents` 数组并不保存任何 `int8_t` 类型的值 —— **`contents` 数组的真正类型取决于 `encoding` 属性的值**：

- `encoding` 为 `INTSET_ENC_INT16`，`int16_t` 类型的数组，范围$[-2^{16},2^{16}-1]$
- `encoding`  `INTSET_ENC_INT32` ， 是一个 `int32_t` 类型的数组。
- `encoding` 为 `INTSET_ENC_INT64` ， 是一个 `int64_t` 类型的数组

```c
typedef struct intset{
   uint32_t encoding; //编码方式
   uint32_t length; //集合包含的元素数量
   int8_t contents[]; //保存元素的数组
}intset;
```

整数集合的每个元素都是 contents 数组的一个数据项，它们按照从小到大的顺序排列，并且不包含任何重复项。
length 属性记录了 contents 数组的大小。
需要注意的是虽然 contents 数组声明为 int8_t 类型，但是实际上contents 数组并不保存任何 int8_t 类型的值，其真正类型有 encoding 来决定。

### 3.5.2 升级

当我们新增的元素类型比原集合元素类型的长度要大时，需要对整数集合进行升级，才能将新元素放入整数集合中。具体步骤：

　　1、根据新元素类型，扩展整数集合底层数组的大小，并为新元素分配空间。

　　2、将底层数组现有的所有元素都转成与新元素相同类型的元素，并将转换后的元素放到正确的位置，放置过程中，维持整个元素顺序都是有序的。

　　3、将新元素添加到整数集合中（保证有序）。

　　升级能极大地节省内存。

每当我们要将一个新元素添加到整数集合里面， 并且**新元素的类型比整数集合元素的类型长时**， 整数集合需要先进行**升级（upgrade）**， 然后才能将新元素添加到整数集合里面。

过程如下：

1. 根据新类型，扩展整数集合底层数组的空间大小， 并为新元素分配空间
2. 将底层数组现有的所有元素都转换成与新元素相同的类型， 并将类型转换后的元素有序放置。
3. 将新元素添加到底层数组里面。

### 3.5.3 降级

整数集合不支持降级操作，一旦对数组进行了升级，编码就会一直保持升级后的状态。当一个集合中**只包含整数**，并且**元素的个数不是很多**的话，redis 会用**整数集合**作为底层存储，它可以节省很多内存整数集合不支持降级操作， 一旦对数组进行了升级， 编码就会一直保持升级后的状态。即使我们将集合里唯一一个真正需要使用 `int64_t` 类型来保存的元素 `4294967295` 删除了， 整数集合的编码仍然会维持 `INTSET_ENC_INT64`。

## 3.6 压缩列表（ziplist）

压缩列表是Redis为了节省内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型数据结构，一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。压缩列表并不是对数据利用某种算法进行压缩，而是将数据按照一定规则编码在一块连续的内存区域，目的是节省内存。

压缩列表的每个节点构成如下：
　　①、previous_entry_ength：记录压缩列表前一个字节的长度。previous_entry_ength的长度可能是1个字节或者是5个字节，如果上一个节点的长度小于254，则该节点只需要一个字节就可以表示前一个节点的长度了，如果前一个节点的长度大于等于254，则previous length的第一个字节为254，后面用四个字节表示当前节点前一个节点的长度。利用此原理即当前节点位置减去上一个节点的长度即得到上一个节点的起始位置，压缩列表可以从尾部向头部遍历。这么做很有效地减少了内存的浪费。

　　②、encoding：节点的encoding保存的是节点的content的内容类型以及长度，encoding类型一共有两种，一种字节数组一种是整数，encoding区域长度为1字节、2字节或者5字节长。

　　③、content：content区域用于保存节点的内容，节点内容类型和长度由encoding决定。

压缩列表（ziplist）**是列表键和哈希键的底层实现之一。**当一个列表键只包含少量列表项， 并且每个列表项要么就是**小整数值或长度比较短的字符串**， 那么 Redis 就会使用压缩列表来做列表键的底层实现。

### 3.6.1 压缩列表的构成

压缩列表是 Redis 为了节约内存而开发的， 由一系列特殊编码的**连续内存块组成的顺序型（sequential）数据结构。**

| 属性      | 类型       | 长度  | 作用                  |
|:-------:|:--------:|:---:|:-------------------:|
| zlbytes | uint32_t | 4字节 | 整个压缩列表占用内存字节数       |
| zltail  | uint32_t | 4字节 | 记录表尾节点距离表起始地址有多少个字节 |
| zllen   | uint16_t | 2字节 | 记录节点数量              |
| entryX  |          | 不定  | 节点                  |
| zlend   | uint8_t  | 1字节 | 用于标记末端              |

- `zlbytes` 属性的值为 `0x50` （十进制 `80`）， 表示压缩列表的总长为 `80` 字节。
- `zltail` 属性的值为 `0x3c` （十进制 `60`），如果一个指向压缩列表起始地址的指针 `p` ， 那么只要用指针 `p` 加上偏移量 `60` ， 就可以计算出表尾节点 `entry3` 的地址。
- `zllen` 属性的值为 `0x3` （十进制 `3`）， 表示压缩列表包含三个节点。

### 3.6.2 压缩列表的节点构成

每个压缩列表节点可以保存一个字节数组或者一个整数值， 其中， 字节数组可以是以下三种长度的其中一种：

1. 长度小于等于`63`($2^6-1$)字节的字节数组；
2. 长度小于等于 `16383` ($2^{14}-1$)字节的字节数组；
3. 长度小于等于 `4294967295` ($2^{32}-1$)字节的字节数组；

而整数值则可以是以下六种长度的其中一种：

1. `4` 位长，介于 `0` 至 `12` 之间的无符号整数；
2. `1` 字节长的有符号整数；
3. `3` 字节长的有符号整数；
4. `int16_t` 类型整数；
5. `int32_t` 类型整数；
6. `int64_t` 类型整数。

每个压缩列表节点都由 `previous_entry_length` 、 `encoding` 、 `content` 三个部分组成。

**（1）previous_entry_length**

以字节为单位， 记录了压缩列表中**前一个节点的长度。**这个属性的长度可以是1字节或5字节，如果前一个小于254则使用1字节，反之使用5字节（ 其中属性的**第一字节会被设置为 `0xFE`（十进制值 `254`）**， 而之后的四个字节则用于保存前一节点的长度）

程序可以通过指针运算， 根据当前节点的起始地址来**计算出前一个节点的起始地址**。进而可以回溯到表头。

**（2）encoding**

节点的 `encoding` 属性记录了节点的 `content` 属性所保存数据的类型以及长度。编码由8位组成。

**如果是字符类型**，则开头两位`00`,`01`,`10`分别表示1字节，2字节，5字节，后6位表示字符串长度。

如果是整数类型，则开头必是11，然后从第6位开始往低位开始计数：

### 3.6.3 连锁更新

每个节点的 `previous_entry_length` 属性都记录了前一个节点的长度：

- 如果前一节点的长度小于 `254` 字节， 那么 `previous_entry_length` 属性需要用 `1` 字节长的空间来保存这个长度值。
- 如果前一节点的长度大于等于 `254` 字节， 那么 `previous_entry_length` 属性需要用 `5` 字节长的空间来保存这个长度值。

假设现在有一些长度为252字节的节点，他们在`previous_entry_length`中保存为1字节。现在插入了一个260字节的新节点，`new` 将成为 `e1` 的前置节点。

因为 `e1` 的 `previous_entry_length` 属性仅长 `1` 字节， 它没办法保存新节点 `new` 的长度， 所以程序将对压缩列表执行空间重分配操作， **并将`e1` 节点的 `previous_entry_length` 属性从原来的 `1` 字节长扩展为 `5` 字节长。**

由于`previous_entry_length` 的变化，导致`e1`的长度也发生了变化$252+4=256>254$，所以导致`e2`也必须更新它的`previous_entry_length` 。这就是连锁更新。

除了添加节点外，删除节点也会导致连锁更新，若删除一个260字节的节点，则后一个节点长度也会变化。如果很不凑巧，小于254，则又会引起后序效应。

连锁更新在最坏情况下需要对压缩列表执行 `N` 次空间重分配操作， 而每次空间重分配的最坏复杂度为$O(N)$ ， 所以连锁更新的最坏复杂度为 $O(N^2)$ 。 

# 4 常用命令

## 4.1 字符串

### SET

> `SET`将字符串 `value` 关联到 `key` ，若 `key` 存在则覆写旧值，操作成功返回 `OK`

```shell
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```




- `EX seconds` ： 将键的过期时间设置为 `seconds` 秒，清除该键原有的生存时间（TTL） 
- `PX milliseconds` ： 将键的过期时间设置为 `milliseconds` 毫秒
- `NX` ： 键不存在时对键进行设置操作，否则返回空批量回复nil（NULL Bulk Reply）
- `XX` ： 键存在时对键进行设置操作，否则返回空批量回复nil

```bash
对不存在的键进行设置
redis> SET key "value"
OK

使用 `EX` 选项
redis> SET key-with-expire-time "value" EX 10086
OK

redis> SET key-with-pexpire-time "value" PX 123321 使用 `PX` 选项
OK

redis> SET key "new-value" NX
(nil)   # 键已经存在，设置失败

redis> SET exists-key "value" XX 使用 `XX` 选项
(nil)   # 因为键不存在，设置失败
```

### GET

> 返回与键 `key` 相关联的字符串值，若键 `key` 不存在， 返回 `nil` ，若键 `key` 的值并非字符串类型， 返回错误

```shell
GET key
```

```bash
redis> GET key
"value"

redis> GET key1 # 对不存在的键 `key` 执行 `GET` 命令
(nil)

redis> LPUSH db redis mongodb mysql
(integer) 3

redis> GET db # 对不是字符串类型的键 `key` 执行 `GET` 命令
(error) ERR Operation against a key holding the wrong kind of value
```

### GETSET

> 将键 `key` 的值设为 `value` ， 返回键 `key` 在被设置之前的旧值。若键 `key` 不存在， 返回 `nil` ，当键 `key` 存在但不是字符串类型时， 返回错误

```shell
GETSET key value
```

```bash
redis> GETSET key1 value1      # 没有旧值，返回 nil
(nil)

redis> GETSET key value1      # 返回旧值 mongodb
"value" 
```

### STRLEN

> 返回键 `key` 储存的字符串值的长度。当键 `key` 不存在， 返回 `0` 。当 `key` 储存的不是字符串值时， 返回错误

```shell
STRLEN key
```

```bash
redis> STRLEN key
(integer) 5
```

### APPEND

> 如果键 `key` 已经存在并且它的值是一个字符串， `APPEND` 命令将把 `value` 追加到键 `key` 现有值的末尾。如果 `key` 不存在， `APPEND` 将键 `key` 的值设为 `value` ， 就像执行 `SET key value` 一样。返回追加 `value` 之后， 键 `key` 的值的长度。

```shell
APPEND key value
```

```bash
redis> APPEND key "hhh"     # 长度从 5 个字符增加到 8 个字符
(integer) 8
```

### SETRANGE

> 从偏移量 `offset` 开始， 用 `value` 参数覆写键 `key` 储存的字符串值，返回被修改之后， 字符串值的长度。如果键 `key` 原来储存的字符串长度比偏移量小，那么原字符和偏移量之间的空白将用零字节(zerobytes, `"\x00"` )填充。不存在的键 `key` 当作空白字符串处理。

```shell
SETRANGE key offset value
```

```bash
redis> SET greeting "hello world"
OK

redis> SETRANGE greeting 6 "Redis" # 对非空字符串执行 `SETRANGE` 命令：
(integer) 11

redis> GET greeting
"hello Redis"

redis> EXISTS empty_string
(integer) 0

redis> SETRANGE empty_string 5 "Redis!"   # 对不存在的 key 使用 SETRANGE
(integer) 11

redis> GET empty_string                   # 空白处被"\x00"填充
"\x00\x00\x00\x00\x00Redis!"
```

### GETRANGE

GETRANGE key start end

返回键 `key` 储存的字符串值的指定部分， 字符串的截取范围由 `start` 和 `end` 两个偏移量决定 (包括 `start` 和 `end` 在内)。负数偏移量表示从字符串的末尾开始计数， `-1` 表示最后一个字符， `-2` 表示倒数第二个字符， 以此类推。
`GETRANGE` 通过保证子字符串的值域(range)不超过实际字符串的值域来处理超出范围的值域请求。

Note

`GETRANGE` 命令在 Redis 2.0 之前的版本里面被称为 `SUBSTR` 命令。

`GETRANGE` 命令会返回字符串值的指定部分。

```bash
redis> SET greeting "hello, my friend"
OK

redis> GETRANGE greeting 0 4          # 返回索引0-4的字符，包括4。
"hello"

redis> GETRANGE greeting -1 -5        # 不支持回绕操作
""

redis> GETRANGE greeting -3 -1        # 负数索引
"end"

redis> GETRANGE greeting 0 -1         # 从第一个到最后一个
"hello, my friend"

redis> GETRANGE greeting 0 1008611    # 值域范围不超过实际字符串，超过部分自动被符略
"hello, my friend"
```

### INCR

INCR key

为键 `key` 储存的数字值加上一。

如果键 `key` 不存在， 那么它的值会先被初始化为 `0` ， 然后再执行 `INCR` 命令。

如果键 `key` 储存的值不能被解释为数字， 那么 `INCR` 命令将返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

`INCR` 命令是一个针对字符串的操作。 因为 Redis 并没有专用的整数类型， 所以键 `key` 储存的值在执行 `INCR` 命令时会被解释为十进制 64 位有符号整数。

`INCR` 命令会返回键 `key` 在执行加一操作之后的值。

```bash
redis> SET page_view 20
OK

redis> INCR page_view
(integer) 21

redis> GET page_view    # 数字值在 Redis 中以字符串的形式保存
"21"
```

### INCRBY

**INCRBY key increment**

为键 `key` 储存的数字值加上增量 `increment` 。

如果键 `key` 不存在， 那么键 `key` 的值会先被初始化为 `0` ， 然后再执行 `INCRBY` 命令。

如果键 `key` 储存的值不能被解释为数字， 那么 `INCRBY` 命令将返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于递增(increment) / 递减(decrement)操作的更多信息

在加上增量 `increment` 之后， 键 `key` 当前的值。

键存在，并且值为数字：

```bash
redis> SET rank 50
OK

redis> INCRBY rank 20
(integer) 70

redis> GET rank
"70"
```

键不存在：

```bash
redis> EXISTS counter
(integer) 0

redis> INCRBY counter 30
(integer) 30

redis> GET counter
"30"
```

键存在，但值无法被解释为数字：

```bash
redis> SET book "long long ago..."
OK

redis> INCRBY book 200
(error) ERR value is not an integer or out of range
```

### INCRBYFLOAT

**INCRBYFLOAT key increment**

为键 `key` 储存的值加上浮点数增量 `increment` 。

如果键 `key` 不存在， 那么 `INCRBYFLOAT` 会先将键 `key` 的值设为 `0` ， 然后再执行加法操作。

如果命令执行成功， 那么键 `key` 的值会被更新为执行加法计算之后的新值， 并且新值会以字符串的形式返回给调用者。

无论是键 `key` 的值还是增量 `increment` ， 都可以使用像 `2.0e7` 、 `3e5` 、 `90e-2` 那样的指数符号(exponential notation)来表示， 但是， **执行 INCRBYFLOAT 命令之后的值**总是以同样的形式储存， 也即是， 它们总是由一个数字， 一个（可选的）小数点和一个任意长度的小数部分组成（比如 `3.14` 、 `69.768` ，诸如此类)， 小数部分尾随的 `0` 会被移除， 如果可能的话， 命令还会将浮点数转换为整数（比如 `3.0` 会被保存成 `3` ）。

此外， 无论加法计算所得的浮点数的实际精度有多长， `INCRBYFLOAT` 命令的计算结果最多只保留小数点的后十七位。

当以下任意一个条件发生时， 命令返回一个错误：

- 键 `key` 的值不是字符串类型(因为 Redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）；
- 键 `key` 当前的值或者给定的增量 `increment` 不能被解释(parse)为双精度浮点数。

在加上增量 `increment` 之后， 键 `key` 的值。

```bash
redis> GET decimal
"3.0"

redis> INCRBYFLOAT decimal 2.56
"5.56"

redis> GET decimal
"5.56"
```

### DECR

**DECR key**

为键 `key` 储存的数字值减去一。

如果键 `key` 不存在， 那么键 `key` 的值会先被初始化为 `0` ， 然后再执行 `DECR` 操作。

如果键 `key` 储存的值不能被解释为数字， 那么 `DECR` 命令将返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于递增(increment) / 递减(decrement)操作的更多信息， 请参见 `INCR` 命令的文档。

`DECR` 命令会返回键 `key` 在执行减一操作之后的值。

对储存数字值的键 `key` 执行 `DECR` 命令：

```bash
redis> SET failure_times 10
OK

redis> DECR failure_times
(integer) 9
```

对不存在的键执行 `DECR` 命令：

```bash
redis> EXISTS count
(integer) 0

redis> DECR count
(integer) -1
```

## DECRBY

**DECRBY key decrement**

将键 `key` 储存的整数值减去减量 `decrement` 。

如果键 `key` 不存在， 那么键 `key` 的值会先被初始化为 `0` ， 然后再执行 `DECRBY` 命令。

如果键 `key` 储存的值不能被解释为数字， 那么 `DECRBY` 命令将返回一个错误。

本操作的值限制在 64 位(bit)有符号数字表示之内。

关于更多递增(increment) / 递减(decrement)操作的更多信息， 请参见 `INCR` 命令的文档。

### 返回值

`DECRBY` 命令会返回键在执行减法操作之后的值。

### 代码示例

对已经存在的键执行 `DECRBY` 命令：

```bash
redis> SET count 100
OK

redis> DECRBY count 20
(integer) 80
```

对不存在的键执行 `DECRBY` 命令：

```bash
redis> EXISTS pages
(integer) 0

redis> DECRBY pages 10
(integer) -10
```

## MSET

**MSET key value [key value …]**

同时为多个键设置值。

如果某个给定键已经存在， 那么 `MSET` 将使用新值去覆盖旧值， 如果这不是你所希望的效果， 请考虑使用 `MSETNX` 命令， 这个命令只会在所有给定键都不存在的情况下进行设置。

`MSET` 是一个原子性(atomic)操作， 所有给定键都会在同一时间内被设置， 不会出现某些键被设置了但是另一些键没有被设置的情况。

### 返回值

`MSET` 命令总是返回 `OK` 。

### 代码示例

同时对多个键进行设置：

```bash
redis> MSET date "2012.3.30" time "11:00 a.m." weather "sunny"
OK

redis> MGET date time weather
1) "2012.3.30"
2) "11:00 a.m."
3) "sunny"
```

覆盖已有的值：

```bash
redis> MGET k1 k2
1) "hello"
2) "world"

redis> MSET k1 "good" k2 "bye"
OK

redis> MGET k1 k2
1) "good"
2) "bye"
```

## MSETNX

**MSETNX key value [key value …]**

当且仅当所有给定键都不存在时， 为所有给定键设置值。

即使只有一个给定键已经存在， `MSETNX` 命令也会拒绝执行对所有键的设置操作。

`MSETNX` 是一个原子性(atomic)操作， 所有给定键要么就全部都被设置， 要么就全部都不设置， 不可能出现第三种状态。

### 返回值

当所有给定键都设置成功时， 命令返回 `1` ； 如果因为某个给定键已经存在而导致设置未能成功执行， 那么命令返回 `0` 。

### 代码示例

对不存在的键执行 `MSETNX` 命令：

```bash
redis> MSETNX rmdbs "MySQL" nosql "MongoDB" key-value-store "redis"
(integer) 1

redis> MGET rmdbs nosql key-value-store
1) "MySQL"
2) "MongoDB"
3) "redis"
```

对某个已经存在的键进行设置：

```bash
redis> MSETNX rmdbs "Sqlite" language "python"  # rmdbs 键已经存在，操作失败
(integer) 0

redis> EXISTS language                          # 因为 MSETNX 命令没有成功执行
(integer) 0                                     # 所以 language 键没有被设置

redis> GET rmdbs                                # rmdbs 键也没有被修改
"MySQL"
```

## MGET

**MGET key [key …]**

返回给定的一个或多个字符串键的值。

如果给定的字符串键里面， 有某个键不存在， 那么这个键的值将以特殊值 `nil` 表示。

### 返回值

`MGET` 命令将返回一个列表， 列表中包含了所有给定键的值。

### 代码示例

```bash
redis> SET redis redis.com
OK

redis> SET mongodb mongodb.org
OK

redis> MGET redis mongodb
1) "redis.com"
2) "mongodb.org"

redis> MGET redis mongodb mysql     # 不存在的 mysql 返回 nil
1) "redis.com"
2) "mongodb.org"
3) (nil)
```

## 哈希表

散列(Hashes)

hset hash-key sub-key value - 在散列中设置给定的键值对
hget hash-key sub-key - 在散列中获取指定键的值
hgetall hash-key - 获取散列中所有的键值对
hdel hash-key sub-key - 移除散列中的给定键（存在返回1，不存在返回0）

集合(Sets)

sadd set-key item - 将给定元素item添加到集合（返回0表示元素已存在于集合中，1表示添加成功）
smembers set-key - 返回集合中的所有元素
sismember set-key item - 检查给定元素item是否存在于集合中
srem set-key item - 如果item存在于集合中，移除该元素（返回移除元素的数量）
SCARD set-key - 获取集合的成员数

有序集合(Sorted Sets)

zadd zset-key score member - 将一个带有给定分值的成员添加到有序集合中
zrange zset-key start stop [withscores] - 根据元素在有序集合中所处的位置，从有序集合里面获取多个元素
zrangebyscore zset-key start stop [withscores] - 获取有序集合在给定分值范围内的所有元素
zrem zset-key member - 在有序集合中移除给定成员（存在返回1，不存在返回0）
zrevrank zset-key member - 返回有序集合成员 member的排名，成员按照分值从大到小排列
zrevrange zset-key start stop [withscores] - 返回有序集合给定排名范围内的成员，成员按照分值从大到小排列

### HSET

**HSET hash field value**

> 

将哈希表 `hash` 中域 `field` 的值设置为 `value` 。

如果给定的哈希表并不存在， 那么一个新的哈希表将被创建并执行 `HSET` 操作。

如果域 `field` 已经存在于哈希表中， 那么它的旧值将被新值 `value` 覆盖。

### 返回值

当 `HSET` 命令在哈希表中新创建 `field` 域并成功为它设置值时， 命令返回 `1` ； 如果域 `field` 已经存在于哈希表， 并且 `HSET` 命令成功使用新值覆盖了它的旧值， 那么命令返回 `0` 。

### 代码示例

设置一个新域：

```bash
redis> HSET website google "www.g.cn"
(integer) 1

redis> HGET website google
"www.g.cn"
```

对一个已存在的域进行更新：

```bash
redis> HSET website google "www.google.com"
(integer) 0

redis> HGET website google
"www.google.com"
```

## HSETNX

**HSETNX hash field value**

当且仅当域 `field` 尚未存在于哈希表的情况下， 将它的值设置为 `value` 。

如果给定域已经存在于哈希表当中， 那么命令将放弃执行设置操作。

如果哈希表 `hash` 不存在， 那么一个新的哈希表将被创建并执行 `HSETNX` 命令。

### 返回值

`HSETNX` 命令在设置成功时返回 `1` ， 在给定域已经存在而放弃执行设置操作时返回 `0` 。

### 代码示例

域尚未存在， 设置成功：

```bash
redis> HSETNX database key-value-store Redis
(integer) 1

redis> HGET database key-value-store
"Redis"
```

域已经存在， 设置未成功， 域原有的值未被改变：

```bash
redis> HSETNX database key-value-store Riak
(integer) 0

redis> HGET database key-value-store
"Redis"
```

## HGET

**HGET hash field**

返回哈希表中给定域的值。

### 返回值

`HGET` 命令在默认情况下返回给定域的值。

如果给定域不存在于哈希表中， 又或者给定的哈希表并不存在， 那么命令返回 `nil` 。

### 代码示例

域存在的情况：

```bash
redis> HSET homepage redis redis.com
(integer) 1

redis> HGET homepage redis
"redis.com"
```

域不存在的情况：

```bash
redis> HGET site mysql
(nil)
```

## HEXISTS

**HEXISTS hash field**

检查给定域 `field` 是否存在于哈希表 `hash` 当中。

### 返回值

`HEXISTS` 命令在给定域存在时返回 `1` ， 在给定域不存在时返回 `0` 。

### 代码示例

给定域不存在：

```bash
redis> HEXISTS phone myphone
(integer) 0
```

给定域存在：

```bash
redis> HSET phone myphone nokia-1110
(integer) 1

redis> HEXISTS phone myphone
(integer) 1
```

## HDEL

**HDEL key field [field …]**

删除哈希表 `key` 中的一个或多个指定域，不存在的域将被忽略。

### 返回值

被成功移除的域的数量，不包括被忽略的域。

### 代码示例

```bash
# 测试数据

redis> HGETALL abbr
1) "a"
2) "apple"
3) "b"
4) "banana"
5) "c"
6) "cat"
7) "d"
8) "dog"


# 删除单个域

redis> HDEL abbr a
(integer) 1


# 删除不存在的域

redis> HDEL abbr not-exists-field
(integer) 0


# 删除多个域

redis> HDEL abbr b c
(integer) 2

redis> HGETALL abbr
1) "d"
2) "dog"
```

## HLEN

**HLEN key**

返回哈希表 `key` 中域的数量。

- **时间复杂度：**
  
  O(1)

- **返回值：**
  
  哈希表中域的数量。当 `key` 不存在时，返回 `0` 。

```bash
redis> HSET db redis redis.com
(integer) 1

redis> HSET db mysql mysql.com
(integer) 1

redis> HLEN db
(integer) 2

redis> HSET db mongodb mongodb.org
(integer) 1

redis> HLEN db
(integer) 3
```

## HSTRLEN

**HSTRLEN key field**

返回哈希表 `key` 中， 与给定域 `field` 相关联的值的字符串长度（string length）。

如果给定的键或者域不存在， 那么命令返回 `0` 。

- **返回值：**
  
  一个整数。

```bash
redis> HMSET myhash f1 "HelloWorld" f2 "99" f3 "-256"
OK

redis> HSTRLEN myhash f1
(integer) 10

redis> HSTRLEN myhash f2
(integer) 2

redis> HSTRLEN myhash f3
(integer) 4
```

## HINCRBY

**HINCRBY key field increment**

为哈希表 `key` 中的域 `field` 的值加上增量 `increment` 。

增量也可以为负数，相当于对给定域进行减法操作。

如果 `key` 不存在，一个新的哈希表被创建并执行 [HINCRBY](#hincrby) 命令。

如果域 `field` 不存在，那么在执行命令前，域的值被初始化为 `0` 。

对一个储存字符串值的域 `field` 执行 [HINCRBY](#hincrby) 命令将造成一个错误。

本操作的值被限制在 64 位(bit)有符号数字表示之内。

- **返回值：**
  
  执行 [HINCRBY](#hincrby) 命令之后，哈希表 `key` 中域 `field` 的值。

```bash
# increment 为正数

redis> HEXISTS counter page_view    # 对空域进行设置
(integer) 0

redis> HINCRBY counter page_view 200
(integer) 200

redis> HGET counter page_view
"200"


# increment 为负数

redis> HGET counter page_view
"200"

redis> HINCRBY counter page_view -50
(integer) 150

redis> HGET counter page_view
"150"


# 尝试对字符串值的域执行HINCRBY命令

redis> HSET myhash string hello,world       # 设定一个字符串值
(integer) 1

redis> HGET myhash string
"hello,world"

redis> HINCRBY myhash string 1              # 命令执行失败，错误。
(error) ERR hash value is not an integer

redis> HGET myhash string                   # 原值不变
"hello,world"
```

## HINCRBYFLOAT

**HINCRBYFLOAT key field increment**

为哈希表 `key` 中的域 `field` 加上浮点数增量 `increment` 。

如果哈希表中没有域 `field` ，那么 [HINCRBYFLOAT](#hincrbyfloat) 会先将域 `field` 的值设为 `0` ，然后再执行加法操作。

如果键 `key` 不存在，那么 [HINCRBYFLOAT](#hincrbyfloat) 会先创建一个哈希表，再创建域 `field` ，最后再执行加法操作。

当以下任意一个条件发生时，返回一个错误：

- 域 `field` 的值不是字符串类型(因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
- 域 `field` 当前的值或给定的增量 `increment` 不能解释(parse)为双精度浮点数(double precision floating point number)

[HINCRBYFLOAT](#hincrbyfloat) 命令的详细功能和 INCRBYFLOAT key increment 命令类似，请查看 INCRBYFLOAT key increment 命令获取更多相关信息。

- **返回值：**
  
  执行加法操作之后 `field` 域的值。

```bash
# 值和增量都是普通小数

redis> HSET mykey field 10.50
(integer) 1
redis> HINCRBYFLOAT mykey field 0.1
"10.6"


# 值和增量都是指数符号

redis> HSET mykey field 5.0e3
(integer) 0
redis> HINCRBYFLOAT mykey field 2.0e2
"5200"


# 对不存在的键执行 HINCRBYFLOAT

redis> EXISTS price
(integer) 0
redis> HINCRBYFLOAT price milk 3.5
"3.5"
redis> HGETALL price
1) "milk"
2) "3.5"


# 对不存在的域进行 HINCRBYFLOAT

redis> HGETALL price
1) "milk"
2) "3.5"
redis> HINCRBYFLOAT price coffee 4.5   # 新增 coffee 域
"4.5"
redis> HGETALL price
1) "milk"
2) "3.5"
3) "coffee"
4) "4.5"
```

## HMSET

**HMSET key field value [field value …]**

同时将多个 `field-value` (域-值)对设置到哈希表 `key` 中。

此命令会覆盖哈希表中已存在的域。

如果 `key` 不存在，一个空哈希表被创建并执行 [HMSET](#hmset) 操作。

- **返回值：**
  
  如果命令执行成功，返回 `OK` 。当 `key` 不是哈希表(hash)类型时，返回一个错误。

```bash
redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HGET website google
"www.google.com"

redis> HGET website yahoo
"www.yahoo.com"
```

## HMGET

**HMGET key field [field …]**

返回哈希表 `key` 中，一个或多个给定域的值。

如果给定的域不存在于哈希表，那么返回一个 `nil` 值。

因为不存在的 `key` 被当作一个空哈希表来处理，所以对一个不存在的 `key` 进行 [HMGET](#hmget) 操作将返回一个只带有 `nil` 值的表。

- **返回值：**
  
  一个包含多个给定域的关联值的表，表值的排列顺序和给定域参数的请求顺序一样。

```bash
redis> HMSET pet dog "doudou" cat "nounou"    # 一次设置多个域
OK

redis> HMGET pet dog cat fake_pet             # 返回值的顺序和传入参数的顺序一样
1) "doudou"
2) "nounou"
3) (nil)                                      # 不存在的域返回nil值
```

## HKEYS

**HKEYS key**

返回哈希表 `key` 中的所有域。

- **返回值：**
  
  一个包含哈希表中所有域的表。当 `key` 不存在时，返回一个空表。

```bash
# 哈希表非空

redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HKEYS website
1) "google"
2) "yahoo"


# 空哈希表/key不存在

redis> EXISTS fake_key
(integer) 0

redis> HKEYS fake_key
(empty list or set)
```

## HVALS

**HVALS key**

返回哈希表 `key` 中所有域的值。

- **返回值：**
  
  一个包含哈希表中所有值的表。当 `key` 不存在时，返回一个空表。

```bash
# 非空哈希表

redis> HMSET website google www.google.com yahoo www.yahoo.com
OK

redis> HVALS website
1) "www.google.com"
2) "www.yahoo.com"


# 空哈希表/不存在的key

redis> EXISTS not_exists
(integer) 0

redis> HVALS not_exists
(empty list or set)
```

## HGETALL

**HGETALL key**

返回哈希表 `key` 中，所有的域和值。

在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

- **返回值：**
  
  以列表形式返回哈希表的域和域的值。若 `key` 不存在，返回空列表。

```bash
redis> HSET people jack "Jack Sparrow"
(integer) 1

redis> HSET people gump "Forrest Gump"
(integer) 1

redis> HGETALL people
1) "jack"          # 域
2) "Jack Sparrow"  # 值
3) "gump"
4) "Forrest Gump"
```

## HSCAN

**HSCAN key cursor [MATCH pattern] [COUNT count]**

具体信息请参考 SCAN cursor [MATCH pattern] [COUNT count] 命令。

# 列表

RPush list-key item - 将给定值推入列表的右端
LRange list-key start stop - 获取列表在给定范围上的所有值
LIndex list-key index - 获取列表在给定位置上的单个元素
LPop list-key - 从列表的左端pop出一个值，并返回该值

## LPUSH

LPUSH key value [value …]

将一个或多个值 `value` 插入到列表 `key` 的表头

如果有多个 `value` 值，那么各个 `value` 值按从左到右的顺序依次插入到表头： 比如说，对空列表 `mylist` 执行命令 `LPUSH mylist a b c` ，列表的值将是 `c b a` ，这等同于原子性地执行 `LPUSH mylist a` 、 `LPUSH mylist b` 和 `LPUSH mylist c` 三个命令。

如果 `key` 不存在，一个空列表会被创建并执行 [LPUSH](#lpush) 操作。

当 `key` 存在但不是列表类型时，返回一个错误。

Note

在Redis 2.4版本以前的 [LPUSH](#lpush) 命令，都只接受单个 `value` 值。

### 返回值

执行 [LPUSH](#lpush) 命令后，列表的长度。

### 代码示例

## LPUSHX

LPUSHX key value

将值 `value` 插入到列表 `key` 的表头，当且仅当 `key` 存在并且是一个列表。

和 [LPUSH key value value …\] 命令相反，当 `key` 不存在时， [LPUSHX](#lpushx) 命令什么也不做。

### 返回值

[LPUSHX](#lpushx) 命令执行之后，表的长度。

### 代码示例

## RPUSH

RPUSH key value [value …]

将一个或多个值 `value` 插入到列表 `key` 的表尾(最右边)。

如果有多个 `value` 值，那么各个 `value` 值按从左到右的顺序依次插入到表尾：比如对一个空列表 `mylist` 执行 `RPUSH mylist a b c` ，得出的结果列表为 `a b c` ，等同于执行命令 `RPUSH mylist a` 、 `RPUSH mylist b` 、 `RPUSH mylist c` 。

如果 `key` 不存在，一个空列表会被创建并执行 [RPUSH](#rpush) 操作。

当 `key` 存在但不是列表类型时，返回一个错误。

Note

在 Redis 2.4 版本以前的 [RPUSH](#rpush) 命令，都只接受单个 `value` 值。

### 返回值

执行 [RPUSH](#rpush) 操作后，表的长度。

### 代码示例

## RPUSHX

RPUSHX key value

将值 `value` 插入到列表 `key` 的表尾，当且仅当 `key` 存在并且是一个列表。

和 [RPUSH key value value …\] 命令相反，当 `key` 不存在时， [RPUSHX](#rpushx) 命令什么也不做。

### 返回值

[RPUSHX](#rpushx) 命令执行之后，表的长度。

### 代码示例

## LPOP

LPOP key

移除并返回列表 `key` 的头元素。

### 返回值

列表的头元素。 当 `key` 不存在时，返回 `nil` 。

### 代码示例

## RPOP

RPOP key

移除并返回列表 `key` 的尾元素。

### 返回值

列表的尾元素。 当 `key` 不存在时，返回 `nil` 。

### 代码示例

## RPOPLPUSH

RPOPLPUSH source destination

命令 RPOPLPUSH 在一个原子时间内，执行以下两个动作：

- 将列表 `source` 中的最后一个元素(尾元素)弹出，并返回给客户端。
- 将 `source` 弹出的元素插入到列表 `destination` ，作为 `destination` 列表的的头元素。

举个例子，你有两个列表 `source` 和 `destination` ， `source` 列表有元素 `a, b, c` ， `destination` 列表有元素 `x, y, z` ，执行 `RPOPLPUSH source destination` 之后， `source` 列表包含元素 `a, b` ， `destination` 列表包含元素 `c, x, y, z` ，并且元素 `c` 会被返回给客户端。

如果 `source` 不存在，值 `nil` 被返回，并且不执行其他动作。

如果 `source` 和 `destination` 相同，则列表中的表尾元素被移动到表头，并返回该元素，可以把这种特殊情况视作列表的旋转(rotation)操作。

### 返回值

被弹出的元素。

### 代码示例

### 模式： 安全的队列

Redis的列表经常被用作队列(queue)，用于在不同程序之间有序地交换消息(message)。一个客户端通过 LPUSH key value [value …\]命令将消息放入队列中，而另一个客户端通过 RPOP key或者 BRPOP key [key …\] timeout 命令取出队列中等待时间最长的消息。

不幸的是，上面的队列方法是『不安全』的，因为在这个过程中，一个客户端可能在取出一个消息之后崩溃，而未处理完的消息也就因此丢失。

使用 [RPOPLPUSH](#rpoplpush) 命令(或者它的阻塞版本 BRPOPLPUSH source destination timeout )可以解决这个问题：因为它不仅返回一个消息，同时还将这个消息添加到另一个备份列表当中，如果一切正常的话，当一个客户端完成某个消息的处理之后，可以用 LREM key count value 命令将这个消息从备份表删除。

最后，还可以添加一个客户端专门用于监视备份表，它自动地将超过一定处理时限的消息重新放入队列中去(负责处理该消息的客户端可能已经崩溃)，这样就不会丢失任何消息了。

### 模式：循环列表

通过使用相同的 `key` 作为 [RPOPLPUSH](#rpoplpush) 命令的两个参数，客户端可以用一个接一个地获取列表元素的方式，取得列表的所有元素，而不必像 LRANGE key start stop 命令那样一下子将所有列表元素都从服务器传送到客户端中(两种方式的总复杂度都是 O(N))。

以上的模式甚至在以下的两个情况下也能正常工作：

- 有多个客户端同时对同一个列表进行旋转(rotating)，它们获取不同的元素，直到所有元素都被读取完，之后又从头开始。
- 有客户端在向列表尾部(右边)添加新元素。

这个模式使得我们可以很容易实现这样一类系统：有 N 个客户端，需要连续不断地对一些元素进行处理，而且处理的过程必须尽可能地快。一个典型的例子就是服务器的监控程序：它们需要在尽可能短的时间内，并行地检查一组网站，确保它们的可访问性。

注意，使用这个模式的客户端是易于扩展(scala)且安全(reliable)的，因为就算接收到元素的客户端失败，元素还是保存在列表里面，不会丢失，等到下个迭代来临的时候，别的客户端又可以继续处理这些元素了。

## LREM

LREM key count value

根据参数 `count` 的值，移除列表中与参数 `value` 相等的元素。

`count` 的值可以是以下几种：

- `count > 0` : 从表头开始向表尾搜索，移除与 `value` 相等的元素，数量为 `count` 。
- `count < 0` : 从表尾开始向表头搜索，移除与 `value` 相等的元素，数量为 `count` 的绝对值。
- `count = 0` : 移除表中所有与 `value` 相等的值。

### 返回值

被移除元素的数量。 因为不存在的 `key` 被视作空表(empty list)，所以当 `key` 不存在时， [LREM](#lrem) 命令总是返回 `0` 。

### 代码示例

## LLEN

LLEN key

返回列表 `key` 的长度。

如果 `key` 不存在，则 `key` 被解释为一个空列表，返回 `0` .

如果 `key` 不是列表类型，返回一个错误。

### 返回值

列表 `key` 的长度。

### 代码示例

```bash
# 空列表

redis> LLEN job
(integer) 0


# 非空列表

redis> LPUSH job "cook food"
(integer) 1

redis> LPUSH job "have lunch"
(integer) 2

redis> LLEN job
(integer) 2
```

## LINDEX

LINDEX key index

返回列表 `key` 中，下标为 `index` 的元素。

下标(index)参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示列表的第一个元素，以 `1` 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 `-1` 表示列表的最后一个元素， `-2` 表示列表的倒数第二个元素，以此类推。

如果 `key` 不是列表类型，返回一个错误。

### 返回值

列表中下标为 `index` 的元素。 如果 `index` 参数的值不在列表的区间范围内(out of range)，返回 `nil` 。

### 代码示例

```bash
redis> LPUSH mylist "World"
(integer) 1

redis> LPUSH mylist "Hello"
(integer) 2

redis> LINDEX mylist 0
"Hello"

redis> LINDEX mylist -1
"World"

redis> LINDEX mylist 3        # index不在 mylist 的区间范围内
(nil)
```

## LINSERT

LINSERT key BEFORE|AFTER pivot value

将值 `value` 插入到列表 `key` 当中，位于值 `pivot` 之前或之后。

当 `pivot` 不存在于列表 `key` 时，不执行任何操作。

当 `key` 不存在时， `key` 被视为空列表，不执行任何操作。

如果 `key` 不是列表类型，返回一个错误。

### 返回值

如果命令执行成功，返回插入操作完成之后，列表的长度。 如果没有找到 `pivot` ，返回 `-1` 。 如果 `key` 不存在或为空列表，返回 `0` 。

### 代码示例

```bash
redis> RPUSH mylist "Hello"
(integer) 1

redis> RPUSH mylist "World"
(integer) 2

redis> LINSERT mylist BEFORE "World" "There"
(integer) 3

redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"


# 对一个非空列表插入，查找一个不存在的 pivot

redis> LINSERT mylist BEFORE "go" "let's"
(integer) -1                                    # 失败


# 对一个空列表执行 LINSERT 命令

redis> EXISTS fake_list
(integer) 0

redis> LINSERT fake_list BEFORE "nono" "gogogog"
(integer) 0                                      # 失败
```

## LSET

LSET key index value

将列表 `key` 下标为 `index` 的元素的值设置为 `value` 。

当 `index` 参数超出范围，或对一个空列表( `key` 不存在)进行 [LSET](#lset) 时，返回一个错误。

关于列表下标的更多信息，请参考 LINDEX key index 命令。

### 返回值

操作成功返回 `ok` ，否则返回错误信息。

### 代码示例

```bash
# 对空列表(key 不存在)进行 LSET

redis> EXISTS list
(integer) 0

redis> LSET list 0 item
(error) ERR no such key


# 对非空列表进行 LSET

redis> LPUSH job "cook food"
(integer) 1

redis> LRANGE job 0 0
1) "cook food"

redis> LSET job 0 "play game"
OK

redis> LRANGE job  0 0
1) "play game"


# index 超出范围

redis> LLEN list                    # 列表长度为 1
(integer) 1

redis> LSET list 3 'out of range'
(error) ERR index out of range
```

## LRANGE

LRANGE key start stop

返回列表 `key` 中指定区间内的元素，区间以偏移量 `start` 和 `stop` 指定。

下标(index)参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示列表的第一个元素，以 `1` 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 `-1` 表示列表的最后一个元素， `-2` 表示列表的倒数第二个元素，以此类推。

### 注意LRANGE命令和编程语言区间函数的区别

假如你有一个包含一百个元素的列表，对该列表执行 `LRANGE list 0 10` ，结果是一个包含11个元素的列表，这表明 `stop` 下标也在 [LRANGE](#lrange) 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的 `Range.new` 、 `Array#slice` 和Python的 `range()` 函数。

### 超出范围的下标

超出范围的下标值不会引起错误。

如果 `start` 下标比列表的最大下标 `end` ( `LLEN list` 减去 `1` )还要大，那么 [LRANGE](#lrange) 返回一个空列表。

如果 `stop` 下标比 `end` 下标还要大，Redis将 `stop` 的值设置为 `end` 。

### 返回值

一个列表，包含指定区间内的元素。

### 代码示例

```bash
redis> RPUSH fp-language lisp
(integer) 1

redis> LRANGE fp-language 0 0
1) "lisp"

redis> RPUSH fp-language scheme
(integer) 2

redis> LRANGE fp-language 0 1
1) "lisp"
2) "scheme"
```

## LTRIM

LTRIM key start stop

对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。

举个例子，执行命令 `LTRIM list 0 2` ，表示只保留列表 `list` 的前三个元素，其余元素全部删除。

下标(index)参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示列表的第一个元素，以 `1` 表示列表的第二个元素，以此类推。

你也可以使用负数下标，以 `-1` 表示列表的最后一个元素， `-2` 表示列表的倒数第二个元素，以此类推。

当 `key` 不是列表类型时，返回一个错误。

[LTRIM](#ltrim) 命令通常和 [LPUSH key value value …\] 命令或 [RPUSH key value value …\] 命令配合使用，举个例子：

```
LPUSH log newest_log
LTRIM log 0 99
```

这个例子模拟了一个日志程序，每次将最新日志 `newest_log` 放到 `log` 列表中，并且只保留最新的 `100` 项。注意当这样使用 `LTRIM` 命令时，时间复杂度是O(1)，因为平均情况下，每次只有一个元素被移除。

### 注意LTRIM命令和编程语言区间函数的区别

假如你有一个包含一百个元素的列表 `list` ，对该列表执行 `LTRIM list 0 10` ，结果是一个包含11个元素的列表，这表明 `stop` 下标也在 [LTRIM](#ltrim) 命令的取值范围之内(闭区间)，这和某些语言的区间函数可能不一致，比如Ruby的 `Range.new` 、 `Array#slice` 和Python的 `range()` 函数。

### 超出范围的下标

超出范围的下标值不会引起错误。

如果 `start` 下标比列表的最大下标 `end` ( `LLEN list` 减去 `1` )还要大，或者 `start > stop` ， [LTRIM](#ltrim) 返回一个空列表(因为 [LTRIM](#ltrim) 已经将整个列表清空)。

如果 `stop` 下标比 `end` 下标还要大，Redis将 `stop` 的值设置为 `end` 。

### 返回值

命令执行成功时，返回 `ok` 。

### 代码示例

```bash
# 情况 1： 常见情况， start 和 stop 都在列表的索引范围之内

redis> LRANGE alpha 0 -1       # alpha 是一个包含 5 个字符串的列表
1) "h"
2) "e"
3) "l"
4) "l"
5) "o"

redis> LTRIM alpha 1 -1        # 删除 alpha 列表索引为 0 的元素
OK

redis> LRANGE alpha 0 -1       # "h" 被删除了
1) "e"
2) "l"
3) "l"
4) "o"


# 情况 2： stop 比列表的最大下标还要大


redis> LTRIM alpha 1 10086     # 保留 alpha 列表索引 1 至索引 10086 上的元素
OK

redis> LRANGE alpha 0 -1       # 只有索引 0 上的元素 "e" 被删除了，其他元素还在
1) "l"
2) "l"
3) "o"


# 情况 3： start 和 stop 都比列表的最大下标要大，并且 start < stop

redis> LTRIM alpha 10086 123321
OK

redis> LRANGE alpha 0 -1        # 列表被清空
(empty list or set)


# 情况 4： start 和 stop 都比列表的最大下标要大，并且 start > stop

redis> RPUSH new-alpha "h" "e" "l" "l" "o"     # 重新建立一个新列表
(integer) 5

redis> LRANGE new-alpha 0 -1
1) "h"
2) "e"
3) "l"
4) "l"
5) "o"

redis> LTRIM new-alpha 123321 10086    # 执行 LTRIM
OK

redis> LRANGE new-alpha 0 -1           # 同样被清空
(empty list or set)
```

## BLPOP

BLPOP key [key …] timeout

[BLPOP](#blpop) 是列表的阻塞式(blocking)弹出原语。

它是 LPOP key 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 [BLPOP](#blpop) 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 `key` 参数时，按参数 `key` 的先后顺序依次检查各个列表，弹出第一个非空列表的头元素。

### 非阻塞行为

当 [BLPOP](#blpop) 被调用时，如果给定 `key` 内至少有一个非空列表，那么弹出遇到的第一个非空列表的头元素，并和被弹出元素所属的列表的名字一起，组成结果返回给调用者。

当存在多个给定 `key` 时， [BLPOP](#blpop) 按给定 `key` 参数排列的先后顺序，依次检查各个列表。

假设现在有 `job` 、 `command` 和 `request` 三个列表，其中 `job` 不存在， `command` 和 `request` 都持有非空列表。考虑以下命令：

```
BLPOP job command request 0
```

[BLPOP](#blpop) 保证返回的元素来自 `command` ，因为它是按”查找 `job` -> 查找 `command` -> 查找 `request` “这样的顺序，第一个找到的非空列表。

```
redis> DEL job command request           # 确保key都被删除
(integer) 0

redis> LPUSH command "update system..."  # 为command列表增加一个值
(integer) 1

redis> LPUSH request "visit page"        # 为request列表增加一个值
(integer) 1

redis> BLPOP job command request 0       # job 列表为空，被跳过，紧接着 command 列表的第一个元素被弹出。
1) "command"                             # 弹出元素所属的列表
2) "update system..."                    # 弹出元素所属的值
```

### 阻塞行为

如果所有给定 `key` 都不存在或包含空列表，那么 [BLPOP](#blpop) 命令将阻塞连接，直到等待超时，或有另一个客户端对给定 `key` 的任意一个执行 [LPUSH key value [value …\]](#lpush) 或 [RPUSH key value [value …\]](#rpush) 命令为止。

超时参数 `timeout` 接受一个以秒为单位的数字作为值。超时参数设为 `0` 表示阻塞时间可以无限期延长(block indefinitely) 。

```bash
redis> EXISTS job                # 确保两个 key 都不存在
(integer) 0
redis> EXISTS command
(integer) 0

redis> BLPOP job command 300     # 因为key一开始不存在，所以操作会被阻塞，直到另一客户端对 job 或者 command 列表进行 PUSH 操作。
1) "job"                         # 这里被 push 的是 job
2) "do my home work"             # 被弹出的值
(26.26s)                         # 等待的秒数

redis> BLPOP job command 5       # 等待超时的情况
(nil)
(5.66s)                          # 等待的秒数
```

### 相同的key被多个客户端同时阻塞

相同的 `key` 可以被多个客户端同时阻塞。

不同的客户端被放进一个队列中，按『先阻塞先服务』(first-BLPOP，first-served)的顺序为 `key` 执行 [BLPOP](#blpop) 命令。

### 在MULTI/EXEC事务中的BLPOP

[BLPOP](#blpop) 可以用于流水线(pipline,批量地发送多个命令并读入多个回复)，但把它用在 [MULTI](#multi) / [EXEC](#exec) 块当中没有意义。因为这要求整个服务器被阻塞以保证块执行时的原子性，该行为阻止了其他客户端执行 [LPUSH key value value …\] 或 [RPUSH key value value …\] 命令。

因此，一个被包裹在 [MULTI](#multi) / [EXEC](#exec) 块内的 [BLPOP](#blpop) 命令，行为表现得就像 LPOP key 一样，对空列表返回 `nil` ，对非空列表弹出列表元素，不进行任何阻塞操作。

```bash
# 对非空列表进行操作

redis> RPUSH job programming
(integer) 1

redis> MULTI
OK

redis> BLPOP job 30
QUEUED

redis> EXEC           # 不阻塞，立即返回
1) 1) "job"
   2) "programming"


# 对空列表进行操作

redis> LLEN job      # 空列表
(integer) 0

redis> MULTI
OK

redis> BLPOP job 30
QUEUED

redis> EXEC         # 不阻塞，立即返回
1) (nil)
```

### 返回值

如果列表为空，返回一个 `nil` 。 否则，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 `key` ，第二个元素是被弹出元素的值。

### 模式：事件提醒

有时候，为了等待一个新元素到达数据中，需要使用轮询的方式对数据进行探查。

另一种更好的方式是，使用系统提供的阻塞原语，在新元素到达时立即进行处理，而新元素还没到达时，就一直阻塞住，避免轮询占用资源。

对于 Redis ，我们似乎需要一个阻塞版的 [SPOP key](#spop) 命令，但实际上，使用 [BLPOP](#blpop) 或者 [BRPOP key [key …\] timeout](#brpop) 就能很好地解决这个问题。

使用元素的客户端(消费者)可以执行类似以下的代码：

```bash
LOOP forever
    WHILE SPOP(key) returns elements
        ... process elements ...
    END
    BRPOP helper_key
END
```

添加元素的客户端(生产者)则执行以下代码：

```bash
MULTI
    SADD key element
    LPUSH helper_key x
EXEC
```

## BRPOP

BRPOP key [key …] timeout

[BRPOP](#brpop) 是列表的阻塞式(blocking)弹出原语。

它是 [RPOP key](#rpop) 命令的阻塞版本，当给定列表内没有任何元素可供弹出的时候，连接将被 [BRPOP](#brpop) 命令阻塞，直到等待超时或发现可弹出元素为止。

当给定多个 `key` 参数时，按参数 `key` 的先后顺序依次检查各个列表，弹出第一个非空列表的尾部元素。

关于阻塞操作的更多信息，请查看 [BLPOP key [key …\] timeout](#blpop) 命令， [BRPOP](#brpop) 除了弹出元素的位置和 [BLPOP key [key …\] timeout](#blpop) 不同之外，其他表现一致。

### 返回值

假如在指定时间内没有任何元素被弹出，则返回一个 `nil` 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素所属的 `key` ，第二个元素是被弹出元素的值。

### 代码示例

```bash
redis> LLEN course
(integer) 0

redis> RPUSH course algorithm001
(integer) 1

redis> RPUSH course c++101
(integer) 2

redis> BRPOP course 30
1) "course"             # 被弹出元素所属的列表键
2) "c++101"             # 被弹出的元素
```

## BRPOPLPUSH

BRPOPLPUSH source destination timeout

[BRPOPLPUSH](#brpoplpush) 是 RPOPLPUSH source destination 的阻塞版本，当给定列表 `source` 不为空时， [BRPOPLPUSH](#brpoplpush) 的表现和 RPOPLPUSH source destination 一样。

当列表 `source` 为空时， [BRPOPLPUSH](#brpoplpush) 命令将阻塞连接，直到等待超时，或有另一个客户端对 `source` 执行 [LPUSH key value value …\] 或 [RPUSH key value value …\] 命令为止。

超时参数 `timeout` 接受一个以秒为单位的数字作为值。超时参数设为 `0` 表示阻塞时间可以无限期延长(block indefinitely) 。

更多相关信息，请参考 RPOPLPUSH source destination 命令。

### 返回值

假如在指定时间内没有任何元素被弹出，则返回一个 `nil` 和等待时长。 反之，返回一个含有两个元素的列表，第一个元素是被弹出元素的值，第二个元素是等待时长。

### 代码示例

```
# 非空列表

redis> BRPOPLPUSH msg reciver 500
"hello moto"                        # 弹出元素的值
(3.38s)                             # 等待时长

redis> LLEN reciver
(integer) 1

redis> LRANGE reciver 0 0
1) "hello moto"


# 空列表

redis> BRPOPLPUSH msg reciver 1
(nil)
(1.34s)
```

### 模式：安全队列

参考 RPOPLPUSH source destination 命令的《安全队列》一节。

### 模式：循环列表

参考 RPOPLPUSH source destination 命令的《循环列表》一节。

# 集合

## SADD

SADD key member [member …]

将一个或多个 `member` 元素加入到集合 `key` 当中，已经存在于集合的 `member` 元素将被忽略。

假如 `key` 不存在，则创建一个只包含 `member` 元素作成员的集合。

当 `key` 不是集合类型时，返回一个错误。

Note

在Redis2.4版本以前， [SADD](#sadd) 只接受单个 `member` 值。

### 返回值

被添加到集合中的新元素的数量，不包括被忽略的元素。

### 代码示例

```bash
# 添加单个元素

redis> SADD bbs "discuz.net"
(integer) 1


# 添加重复元素

redis> SADD bbs "discuz.net"
(integer) 0


# 添加多个元素

redis> SADD bbs "tianya.cn" "groups.google.com"
(integer) 2

redis> SMEMBERS bbs
1) "discuz.net"
2) "groups.google.com"
3) "tianya.cn"
```

SISMEMBER

SISMEMBER key member

判断 `member` 元素是否集合 `key` 的成员。

### 返回值

如果 `member` 元素是集合的成员，返回 `1` 。 如果 `member` 元素不是集合的成员，或 `key` 不存在，返回 `0` 。

### 代码示例

```bash
redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SISMEMBER joe's_movies "bet man"
(integer) 0

redis> SISMEMBER joe's_movies "Fast Five"
(integer) 1
```

## SPOP

SPOP key

移除并返回集合中的一个随机元素。

如果只想获取一个随机元素，但不想该元素从集合中被移除的话，可以使用 [SRANDMEMBER key [count\]](#srandmember) 命令。

### 返回值

被移除的随机元素。 当 `key` 不存在或 `key` 是空集时，返回 `nil` 。

### 代码示例

```bash
redis> SMEMBERS db
1) "MySQL"
2) "MongoDB"
3) "Redis"

redis> SPOP db
"Redis"

redis> SMEMBERS db
1) "MySQL"
2) "MongoDB"

redis> SPOP db
"MySQL"

redis> SMEMBERS db
1) "MongoDB"
```

## SRANDMEMBER

SRANDMEMBER key [count]

如果命令执行时，只提供了 `key` 参数，那么返回集合中的一个随机元素。

从 Redis 2.6 版本开始， [SRANDMEMBER](#srandmember) 命令接受可选的 `count` 参数：

- 如果 `count` 为正数，且小于集合基数，那么命令返回一个包含 `count` 个元素的数组，数组中的元素**各不相同**。如果 `count` 大于等于集合基数，那么返回整个集合。
- 如果 `count` 为负数，那么命令返回一个数组，数组中的元素**可能会重复出现多次**，而数组的长度为 `count` 的绝对值。

该操作和 SPOP key 相似，但 SPOP key 将随机元素从集合中移除并返回，而 [SRANDMEMBER](#srandmember) 则仅仅返回随机元素，而不对集合进行任何改动。

### 返回值

只提供 `key` 参数时，返回一个元素；如果集合为空，返回 `nil` 。 如果提供了 `count` 参数，那么返回一个数组；如果集合为空，返回空数组。

### 代码示例

```bash
# 添加元素

redis> SADD fruit apple banana cherry
(integer) 3

# 只给定 key 参数，返回一个随机元素

redis> SRANDMEMBER fruit
"cherry"

redis> SRANDMEMBER fruit
"apple"

# 给定 3 为 count 参数，返回 3 个随机元素
# 每个随机元素都不相同

redis> SRANDMEMBER fruit 3
1) "apple"
2) "banana"
3) "cherry"

# 给定 -3 为 count 参数，返回 3 个随机元素
# 元素可能会重复出现多次

redis> SRANDMEMBER fruit -3
1) "banana"
2) "cherry"
3) "apple"

redis> SRANDMEMBER fruit -3
1) "apple"
2) "apple"
3) "cherry"

# 如果 count 是整数，且大于等于集合基数，那么返回整个集合

redis> SRANDMEMBER fruit 10
1) "apple"
2) "banana"
3) "cherry"

# 如果 count 是负数，且 count 的绝对值大于集合的基数
# 那么返回的数组的长度为 count 的绝对值

redis> SRANDMEMBER fruit -10
1) "banana"
2) "apple"
3) "banana"
4) "cherry"
5) "apple"
6) "apple"
7) "cherry"
8) "apple"
9) "apple"
10) "banana"

# SRANDMEMBER 并不会修改集合内容

redis> SMEMBERS fruit
1) "apple"
2) "cherry"
3) "banana"

# 集合为空时返回 nil 或者空数组

redis> SRANDMEMBER not-exists
(nil)

redis> SRANDMEMBER not-eixsts 10
(empty list or set)
```

## SREM

SREM key member [member …]

移除集合 `key` 中的一个或多个 `member` 元素，不存在的 `member` 元素会被忽略。

当 `key` 不是集合类型，返回一个错误。

Note

在 Redis 2.4 版本以前， [SREM](#srem) 只接受单个 `member` 值。

### 返回值

被成功移除的元素的数量，不包括被忽略的元素。

### 代码示例

```bash
# 测试数据

redis> SMEMBERS languages
1) "c"
2) "lisp"
3) "python"
4) "ruby"


# 移除单个元素

redis> SREM languages ruby
(integer) 1


# 移除不存在元素

redis> SREM languages non-exists-language
(integer) 0


# 移除多个元素

redis> SREM languages lisp python c
(integer) 3

redis> SMEMBERS languages
(empty list or set)
```

## SMOVE

SMOVE source destination member

将 `member` 元素从 `source` 集合移动到 `destination` 集合。

[SMOVE](#smove) 是原子性操作。

如果 `source` 集合不存在或不包含指定的 `member` 元素，则 [SMOVE](#smove) 命令不执行任何操作，仅返回 `0` 。否则， `member` 元素从 `source` 集合中被移除，并添加到 `destination` 集合中去。

当 `destination` 集合已经包含 `member` 元素时， [SMOVE](#smove) 命令只是简单地将 `source` 集合中的 `member` 元素删除。

当 `source` 或 `destination` 不是集合类型时，返回一个错误。

### 返回值

如果 `member` 元素被成功移除，返回 `1` 。 如果 `member` 元素不是 `source` 集合的成员，并且没有任何操作对 `destination` 集合执行，那么返回 `0` 。

### 代码示例

```bash
redis> SMEMBERS songs
1) "Billie Jean"
2) "Believe Me"

redis> SMEMBERS my_songs
(empty list or set)

redis> SMOVE songs my_songs "Believe Me"
(integer) 1

redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"
```

## SCARD

SCARD key

返回集合 `key` 的基数(集合中元素的数量)。

### 返回值

集合的基数。 当 `key` 不存在时，返回 `0` 。

### 代码示例

```bash
redis> SADD tool pc printer phone
(integer) 3

redis> SCARD tool   # 非空集合
(integer) 3

redis> DEL tool
(integer) 1

redis> SCARD tool   # 空集合
(integer) 0
```

## SMEMBERS

SMEMBERS key

返回集合 `key` 中的所有成员。

不存在的 `key` 被视为空集合。

### 返回值

集合中的所有成员。

### 代码示例

```bash
# key 不存在或集合为空

redis> EXISTS not_exists_key
(integer) 0

redis> SMEMBERS not_exists_key
(empty list or set)


# 非空集合

redis> SADD language Ruby Python Clojure
(integer) 3

redis> SMEMBERS language
1) "Python"
2) "Ruby"
3) "Clojure"
```

## SSCAN

SSCAN key cursor [MATCH pattern] [COUNT count]

详细信息请参考 SCAN cursor [MATCH pattern] [COUNT count] 命令。

## SINTER

SINTER key [key …]

返回一个集合的全部成员，该集合是所有给定集合的交集。

不存在的 `key` 被视为空集。

当给定集合当中有一个空集时，结果也为空集(根据集合运算定律)。

### 返回值

交集成员的列表。

### 代码示例

```bash
redis> SMEMBERS group_1
1) "LI LEI"
2) "TOM"
3) "JACK"

redis> SMEMBERS group_2
1) "HAN MEIMEI"
2) "JACK"

redis> SINTER group_1 group_2
1) "JACK"
```

## SINTERSTORE

SINTERSTORE destination key [key …]

这个命令类似于 [SINTER key key …\] 命令，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

如果 `destination` 集合已经存在，则将其覆盖。

`destination` 可以是 `key` 本身。

### 返回值

结果集中的成员数量。

### 代码示例

```bash
redis> SMEMBERS songs
1) "good bye joe"
2) "hello,peter"

redis> SMEMBERS my_songs
1) "good bye joe"
2) "falling"

redis> SINTERSTORE song_interset songs my_songs
(integer) 1

redis> SMEMBERS song_interset
1) "good bye joe"
```

## SUNION

**SUNION key [key …]**

返回一个集合的全部成员，该集合是所有给定集合的并集。

不存在的 `key` 被视为空集。

### 返回值

并集成员的列表。

### 代码示例

```bash
redis> SMEMBERS songs
1) "Billie Jean"

redis> SMEMBERS my_songs
1) "Believe Me"

redis> SUNION songs my_songs
1) "Billie Jean"
2) "Believe Me"
```

## SUNIONSTORE

**SUNIONSTORE destination key [key …]**

这个命令类似于 [SUNION key [key …\]](#sunion) 命令，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

如果 `destination` 已经存在，则将其覆盖。

`destination` 可以是 `key` 本身。

### 返回值

结果集中的元素数量。

### 代码示例

```bash
redis> SMEMBERS NoSQL
1) "MongoDB"
2) "Redis"

redis> SMEMBERS SQL
1) "sqlite"
2) "MySQL"

redis> SUNIONSTORE db NoSQL SQL
(integer) 4

redis> SMEMBERS db
1) "MySQL"
2) "sqlite"
3) "MongoDB"
4) "Redis"
```

## SDIFF

**SDIFF key [key …]**

返回一个集合的全部成员，该集合是所有给定集合之间的差集。

不存在的 `key` 被视为空集。

### 返回值

一个包含差集成员的列表。

### 代码示例

```bash
redis> SMEMBERS peter's_movies
1) "bet man"
2) "start war"
3) "2012"

redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SDIFF peter's_movies joe's_movies
1) "bet man"
2) "start war"
```

## SDIFFSTORE

**SDIFFSTORE destination key [key …]**

这个命令的作用和 [SDIFF key key …\] 类似，但它将结果保存到 `destination` 集合，而不是简单地返回结果集。

如果 `destination` 集合已经存在，则将其覆盖。

`destination` 可以是 `key` 本身。

### 返回值

结果集中的元素数量。

### 代码示例

```bash
redis> SMEMBERS joe's_movies
1) "hi, lady"
2) "Fast Five"
3) "2012"

redis> SMEMBERS peter's_movies
1) "bet man"
2) "start war"
3) "2012"

redis> SDIFFSTORE joe_diff_peter joe's_movies peter's_movies
(integer) 2

redis> SMEMBERS joe_diff_peter
1) "hi, lady"
2) "Fast Five"
```

# 有序集合

## ZADD

**ZADD key score member [[score member] [score member] …]**

将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中。

如果某个 `member` 已经是有序集的成员，那么更新这个 `member` 的 `score` 值，并通过重新插入这个 `member` 元素，来保证该 `member` 在正确的位置上。

`score` 值可以是整数值或双精度浮点数。

如果 `key` 不存在，则创建一个空的有序集并执行 [ZADD](#zadd) 操作。

当 `key` 存在但不是有序集类型时，返回一个错误。

对有序集的更多介绍请参见 sorted set 。

Note

在 Redis 2.4 版本以前， [ZADD](#zadd) 每次只能添加一个元素。

### 返回值

被成功添加的新成员的数量，不包括那些被更新的、已经存在的成员。

### 代码示例

```bash
# 添加单个元素

redis> ZADD page_rank 10 google.com
(integer) 1


# 添加多个元素

redis> ZADD page_rank 9 baidu.com 8 bing.com
(integer) 2

redis> ZRANGE page_rank 0 -1 WITHSCORES
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，且 score 值不变

redis> ZADD page_rank 10 google.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # 没有改变
1) "bing.com"
2) "8"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"


# 添加已存在元素，但是改变 score 值

redis> ZADD page_rank 6 bing.com
(integer) 0

redis> ZRANGE page_rank 0 -1 WITHSCORES  # bing.com 元素的 score 值被改变
1) "bing.com"
2) "6"
3) "baidu.com"
4) "9"
5) "google.com"
6) "10"
```

## ZSCORE

**ZSCORE key member**

返回有序集 `key` 中，成员 `member` 的 `score` 值。

如果 `member` 元素不是有序集 `key` 的成员，或 `key` 不存在，返回 `nil` 。

### 返回值

`member` 成员的 `score` 值，以字符串形式表示。

### 代码示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
1) "tom"
2) "2000"
3) "peter"
4) "3500"
5) "jack"
6) "5000"

redis> ZSCORE salary peter              # 注意返回值是字符串
"3500"
```

## ZINCRBY

**ZINCRBY key increment member**

为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。

可以通过传递一个负数值 `increment` ，让 `score` 减去相应的值，比如 `ZINCRBY key -5 member` ，就是让 `member` 的 `score` 值减去 `5` 。

当 `key` 不存在，或 `member` 不是 `key` 的成员时， `ZINCRBY key increment member` 等同于 `ZADD key increment member` 。

当 `key` 不是有序集类型时，返回一个错误。

`score` 值可以是整数值或双精度浮点数。

### 返回值

`member` 成员的新 `score` 值，以字符串形式表示。

### 代码示例

```bash
redis> ZSCORE salary tom
"2000"

redis> ZINCRBY salary 2000 tom   # tom 加薪啦！
"4000"
```

## ZCARD

ZCARD key

返回有序集 `key` 的基数。

### 返回值

当 `key` 存在且是有序集类型时，返回有序集的基数。 当 `key` 不存在时，返回 `0` 。

### 代码示例

## ZCOUNT

ZCOUNT key min max

返回有序集 `key` 中， `score` 值在 `min` 和 `max` 之间(默认包括 `score` 值等于 `min` 或 `max` )的成员的数量。

关于参数 `min` 和 `max` 的详细使用方法，请参考 ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT offset count]命令。

### 返回值

`score` 值在 `min` 和 `max` 之间的成员的数量。

### 代码示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES    # 测试数据
1) "jack"
2) "2000"
3) "peter"
4) "3500"
5) "tom"
6) "5000"

redis> ZCOUNT salary 2000 5000          # 计算薪水在 2000-5000 之间的人数
(integer) 3

redis> ZCOUNT salary 3000 5000          # 计算薪水在 3000-5000 之间的人数
(integer) 2
```

## ZRANGE

ZRANGE key start stop [WITHSCORES]

返回有序集 `key` 中，指定区间内的成员。

其中成员的位置按 `score` 值递增(从小到大)来排序。

具有相同 `score` 值的成员按字典序([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order) )来排列。

如果你需要成员按 `score` 值递减(从大到小)来排列，请使用 [ZREVRANGE key start stop WITHSCORES\] 命令。

下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。 你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。

超出范围的下标并不会引起错误。 比如说，当 `start` 的值比有序集的最大下标还要大，或是 `start > stop` 时， [ZRANGE](#zrange) 命令只是简单地返回一个空列表。 另一方面，假如 `stop` 参数的值比有序集的最大下标还要大，那么 Redis 将 `stop` 当作最大下标来处理。

可以通过使用 `WITHSCORES` 选项，来让成员和它的 `score` 值一并返回，返回列表以 `value1,score1, ..., valueN,scoreN` 的格式表示。 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

### 返回值

指定区间内，带有 `score` 值(可选)的有序集成员的列表。

### 代码示例

## ZREVRANGE

**ZREVRANGE key start stop [WITHSCORES]**

返回有序集 `key` 中，指定区间内的成员。

其中成员的位置按 `score` 值递减(从大到小)来排列。 具有相同 `score` 值的成员按字典序的逆序([reverse lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order))排列。

除了成员按 `score` 值递减的次序排列这一点外， [ZREVRANGE](#zrevrange) 命令的其他方面和 [ZRANGE key start stop WITHSCORES\] 命令一样。

### 返回值

指定区间内，带有 `score` 值(可选)的有序集成员的列表。

### 代码示例

## ZRANGEBYSCORE

ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。有序集成员按 `score` 值递增(从小到大)次序排列。

具有相同 `score` 值的成员按字典序([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order))来排列(该属性是有序集提供的，不需要额外的计算)。

可选的 `LIMIT` 参数指定返回结果的数量及区间(就像SQL中的 `SELECT LIMIT offset, count` )，注意当 `offset` 很大时，定位 `offset` 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。

可选的 `WITHSCORES` 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 `score` 值一起返回。 该选项自 Redis 2.0 版本起可用。

### 区间及无限

`min` 和 `max` 可以是 `-inf` 和 `+inf` ，这样一来，你就可以在不知道有序集的最低和最高 `score` 值的情况下，使用 [ZRANGEBYSCORE](#zrangebyscore) 这类命令。

默认情况下，区间的取值使用[闭区间](http://zh.wikipedia.org/wiki/區間) (小于等于或大于等于)，你也可以通过给参数前增加 `(` 符号来使用可选的[开区间](http://zh.wikipedia.org/wiki/區間) (小于或大于)。

举个例子：

```
ZRANGEBYSCORE zset (1 5
```

返回所有符合条件 `1 < score <= 5` 的成员，而

```
ZRANGEBYSCORE zset (5 (10
```

则返回所有符合条件 `5 < score < 10` 的成员。

### 返回值

指定区间内，带有 `score` 值(可选)的有序集成员的列表。

### 代码示例

## ZREVRANGEBYSCORE

**ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]**

返回有序集 `key` 中， `score` 值介于 `max` 和 `min` 之间(默认包括等于 `max` 或 `min` )的所有的成员。有序集成员按 `score` 值递减(从大到小)的次序排列。

具有相同 `score` 值的成员按字典序的逆序([reverse lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order) )排列。

除了成员按 `score` 值递减的次序排列这一点外， ZREVRANGEBYSCORE 命令的其他方面和 ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT offset count] 命令一样。

### 返回值

指定区间内，带有 `score` 值(可选)的有序集成员的列表。

### 代码示例

## ZRANK

**ZRANK key member**

返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(从小到大)顺序排列。

排名以 `0` 为底，也就是说， `score` 值最小的成员排名为 `0` 。

使用 ZREVRANK key member 命令可以获得成员按 `score` 值递减(从大到小)排列的排名。

### 返回值

如果 `member` 是有序集 `key` 的成员，返回 `member` 的排名。 如果 `member` 不是有序集 `key` 的成员，返回 `nil` 。

### 代码示例

```bash
redis> ZRANGE salary 0 -1 WITHSCORES        # 显示所有成员及其 score 值
1) "peter"
2) "3500"
3) "tom"
4) "4000"
5) "jack"
6) "5000"

redis> ZRANK salary tom                     # 显示 tom 的薪水排名，第二
(integer) 1
```

## ZREVRANK

ZREVRANK key member

返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递减(从大到小)排序。

排名以 `0` 为底，也就是说， `score` 值最大的成员排名为 `0` 。

使用 ZRANK key member 命令可以获得成员按 `score` 值递增(从小到大)排列的排名。

### 返回值

如果 `member` 是有序集 `key` 的成员，返回 `member` 的排名。 如果 `member` 不是有序集 `key` 的成员，返回 `nil` 。

### 代码示例

```bash
redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES     # 测试数据
1) "jack"
2) "2000"
3) "peter"
4) "3500"
5) "tom"
6) "5000"

redis> ZREVRANK salary peter     # peter 的工资排第二
(integer) 1

redis> ZREVRANK salary tom       # tom 的工资最高
(integer) 0
```

## ZREM

**ZREM key member [member …]**

移除有序集 `key` 中的一个或多个成员，不存在的成员将被忽略。

当 `key` 存在但不是有序集类型时，返回一个错误。

Note

在 Redis 2.4 版本以前， [ZREM](#zrem) 每次只能删除一个元素。

### 返回值

被成功移除的成员的数量，不包括被忽略的成员。

### 代码示例

## ZREMRANGEBYRANK

**ZREMRANGEBYRANK key start stop**

移除有序集 `key` 中，指定排名(rank)区间内的所有成员。

区间分别以下标参数 `start` 和 `stop` 指出，包含 `start` 和 `stop` 在内。

下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。 你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。

### 返回值

被移除成员的数量。

### 代码示例

```bash
redis> ZADD salary 2000 jack
(integer) 1
redis> ZADD salary 5000 tom
(integer) 1
redis> ZADD salary 3500 peter
(integer) 1

redis> ZREMRANGEBYRANK salary 0 1       # 移除下标 0 至 1 区间内的成员
(integer) 2

redis> ZRANGE salary 0 -1 WITHSCORES    # 有序集只剩下一个成员
1) "tom"
2) "5000"
```

## ZREMRANGEBYSCORE

ZREMRANGEBYSCORE key min max

移除有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。

自版本2.1.6开始， `score` 值等于 `min` 或 `max` 的成员也可以不包括在内，详情请参见 ZRANGEBYSCORE key min max [WITHSCORES\] [LIMIT offset count] 命令。

### 返回值

被移除成员的数量。

### 代码示例

## ZRANGEBYLEX

ZRANGEBYLEX key min max [LIMIT offset count]

当有序集合的所有成员都具有相同的分值时， 有序集合的元素会根据成员的字典序（lexicographical ordering）来进行排序， 而这个命令则可以返回给定的有序集合键 `key` 中， 值介于 `min` 和 `max` 之间的成员。

如果有序集合里面的成员带有不同的分值， 那么命令返回的结果是未指定的（unspecified）。

命令会使用 C 语言的 `memcmp()` 函数， 对集合中的每个成员进行逐个字节的对比（byte-by-byte compare）， 并按照从低到高的顺序， 返回排序后的集合成员。 如果两个字符串有一部分内容是相同的话， 那么命令会认为较长的字符串比较短的字符串要大。

可选的 `LIMIT offset count` 参数用于获取指定范围内的匹配元素 （就像 SQL 中的 `SELECT LIMIT offset count` 语句）。 需要注意的一点是， 如果 `offset` 参数的值非常大的话， 那么命令在返回结果之前， 需要先遍历至 `offset` 所指定的位置， 这个操作会为命令加上最多 O(N) 复杂度。

### 如何指定范围区间

合法的 `min` 和 `max` 参数必须包含 `(` 或者 `[` ， 其中 `(` 表示开区间（指定的值不会被包含在范围之内）， 而 `[` 则表示闭区间（指定的值会被包含在范围之内）。

特殊值 `+` 和 `-` 在 `min` 参数以及 `max` 参数中具有特殊的意义， 其中 `+` 表示正无限， 而 `-` 表示负无限。 因此， 向一个所有成员的分值都相同的有序集合发送命令 `ZRANGEBYLEX - +` ， 命令将返回有序集合中的所有元素。

### 返回值

数组回复：一个列表，列表里面包含了有序集合在指定范围内的成员。

### 代码示例

## ZLEXCOUNT

**ZLEXCOUNT key min max**

对于一个所有成员的分值都相同的有序集合键 `key` 来说， 这个命令会返回该集合中， 成员介于 `min` 和 `max` 范围内的元素数量。

这个命令的 `min` 参数和 `max` 参数的意义和 [ZRANGEBYLEX key min max [LIMIT offset count\]](#zrangebylex) 命令的 `min` 参数和 `max` 参数的意义一样。

### 返回值

整数回复：指定范围内的元素数量。

### 代码示例

## ZREMRANGEBYLEX

**ZREMRANGEBYLEX key min max**

对于一个所有成员的分值都相同的有序集合键 `key` 来说， 这个命令会移除该集合中， 成员介于 `min` 和 `max` 范围内的所有元素。

这个命令的 `min` 参数和 `max` 参数的意义和 [ZRANGEBYLEX key min max [LIMIT offset count\]](#zrangebylex) 命令的 `min` 参数和 `max` 参数的意义一样。

### 返回值

整数回复：被移除的元素数量。

### 代码示例

## ZSCAN

ZSCAN key cursor [MATCH pattern] [COUNT count]

详细信息请参考 SCAN cursor [MATCH pattern] [COUNT count] 命令。

## ZUNIONSTORE

ZUNIONSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]

计算给定的一个或多个有序集的并集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该并集(结果集)储存到 `destination` 。

默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之 *和* 。

### WEIGHTS

使用 `WEIGHTS` 选项，你可以为 *每个* 给定有序集 *分别* 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 `score` 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

如果没有指定 `WEIGHTS` 选项，乘法因子默认设置为 `1` 。

### AGGREGATE

使用 `AGGREGATE` 选项，你可以指定并集的结果集的聚合方式。

默认使用的参数 `SUM` ，可以将所有集合中某个成员的 `score` 值之 *和* 作为结果集中该成员的 `score` 值；使用参数 `MIN` ，可以将所有集合中某个成员的 *最小* `score` 值作为结果集中该成员的 `score` 值；而参数 `MAX` 则是将所有集合中某个成员的 *最大* `score` 值作为结果集中该成员的 `score` 值。

### 返回值

保存到 `destination` 的结果集的基数。

### 代码示例

## ZINTERSTORE

**ZINTERSTORE destination numkeys key [key …] [WEIGHTS weight [weight …]] [AGGREGATE SUM|MIN|MAX]**

计算给定的一个或多个有序集的交集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该交集(结果集)储存到 `destination` 。

默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和.

关于 `WEIGHTS` 和 `AGGREGATE` 选项的描述，参见 ZUNIONSTORE destination numkeys key [key …\] [WEIGHTS weight [weight …] [AGGREGATE SUM|MIN|MAX]命令。

### 返回值

保存到 `destination` 的结果集的基数。

### 代码示例

# HyperLogLog

## PFADD

**PFADD key element [element …]**

将任意数量的元素添加到指定的 HyperLogLog 里面。

作为这个命令的副作用， HyperLogLog 内部可能会被更新， 以便反映一个不同的唯一元素估计数量（也即是集合的基数）。

如果 HyperLogLog 估计的近似基数（approximated cardinality）在命令执行之后出现了变化， 那么命令返回 `1` ， 否则返回 `0` 。 如果命令执行时给定的键不存在， 那么程序将先创建一个空的 HyperLogLog 结构， 然后再执行命令。

调用 [PFADD key element element …\] 命令时可以只给定键名而不给定元素：

- 如果给定键已经是一个 HyperLogLog ， 那么这种调用不会产生任何效果；
- 但如果给定的键不存在， 那么命令会创建一个空的 HyperLogLog ， 并向客户端返回 `1` 。

要了解更多关于 HyperLogLog 数据结构的介绍知识， 请查阅 [PFCOUNT key key …\] 命令的文档。

### 返回值

整数回复： 如果 HyperLogLog 的内部储存被修改了， 那么返回 1 ， 否则返回 0 。

### 代码示例

## PFCOUNT

PFCOUNT key [key …]

当 PFCOUNT key [key …] 命令作用于单个键时， 返回储存在给定键的 HyperLogLog 的近似基数， 如果键不存在， 那么返回 0 。

当 PFCOUNT key [key …] 命令作用于多个键时， 返回所有给定 HyperLogLog 的并集的近似基数， 这个近似基数是通过将所有给定 HyperLogLog 合并至一个临时 HyperLogLog 来计算得出的。

通过 HyperLogLog 数据结构， 用户可以使用少量固定大小的内存， 来储存集合中的唯一元素 （每个 HyperLogLog 只需使用 12k 字节内存，以及几个字节的内存来储存键本身）。

命令返回的可见集合（observed set）基数并不是精确值， 而是一个带有 0.81% 标准错误（standard error）的近似值。

举个例子， 为了记录一天会执行多少次各不相同的搜索查询， 一个程序可以在每次执行搜索查询时调用一次 PFADD key element [element …] ， 并通过调用 PFCOUNT key [key …] 命令来获取这个记录的近似结果。

### 返回值

整数回复： 给定 HyperLogLog 包含的唯一元素的近似数量。

### 代码示例

## PFMERGE

PFMERGE destkey sourcekey [sourcekey …]

将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集。

合并得出的 HyperLogLog 会被储存在 `destkey` 键里面， 如果该键并不存在， 那么命令在执行之前， 会先为该键创建一个空的 HyperLogLog 。

### 返回值

字符串回复：返回 `OK` 。

### 代码示例

# 地理位置

## GEOADD

**GEOADD key longitude latitude member [longitude latitude member …]**

将给定的空间元素（纬度、经度、名字）添加到指定的键里面。 这些数据会以有序集合的形式被储存在键里面， 从而使得像 `GEORADIUS` 和 `GEORADIUSBYMEMBER` 这样的命令可以在之后通过位置查询取得这些元素。

`GEOADD` 命令以标准的 `x,y` 格式接受参数， 所以用户必须先输入经度， 然后再输入纬度。 `GEOADD` 能够记录的坐标是有限的： 非常接近两极的区域是无法被索引的。 精确的坐标限制由 EPSG:900913 / EPSG:3785 / OSGEO:41001 等坐标系统定义， 具体如下：

- 有效的经度介于 -180 度至 180 度之间。
- 有效的纬度介于 -85.05112878 度至 85.05112878 度之间。

当用户尝试输入一个超出范围的经度或者纬度时， `GEOADD` 命令将返回一个错误。

### 返回值

新添加到键里面的空间元素数量， 不包括那些已经存在但是被更新的元素。

### 代码示例

## GEOPOS

**GEOPOS key member [member …]**

从键里面返回所有给定位置元素的位置（经度和纬度）。

因为 `GEOPOS` 命令接受可变数量的位置元素作为输入， 所以即使用户只给定了一个位置元素， 命令也会返回数组回复。

### 返回值

`GEOPOS` 命令返回一个数组， 数组中的每个项都由两个元素组成： 第一个元素为给定位置元素的经度， 而第二个元素则为给定位置元素的纬度。 当给定的位置元素不存在时， 对应的数组项为空值。

### 代码示例

## GEODIST

**GEODIST key member1 member2 [unit]**

返回两个给定位置之间的距离。

如果两个位置之间的其中一个不存在， 那么命令返回空值。

指定单位的参数 `unit` 必须是以下单位的其中一个：

- `m` 表示单位为米。
- `km` 表示单位为千米。
- `mi` 表示单位为英里。
- `ft` 表示单位为英尺。

如果用户没有显式地指定单位参数， 那么 `GEODIST` 默认使用米作为单位。

`GEODIST` 命令在计算距离时会假设地球为完美的球形， 在极限情况下， 这一假设最大会造成 0.5% 的误差。

### 返回值

计算出的距离会以双精度浮点数的形式被返回。 如果给定的位置元素不存在， 那么命令返回空值。

### 代码示例

```bash
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2

redis> GEODIST Sicily Palermo Catania
"166274.15156960039"

redis> GEODIST Sicily Palermo Catania km
"166.27415156960038"

redis> GEODIST Sicily Palermo Catania mi
"103.31822459492736"

redis> GEODIST Sicily Foo Bar
(nil)
```

## GEORADIUS

**GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]**

以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。

范围可以使用以下其中一个单位：

- `m` 表示单位为米。
- `km` 表示单位为千米。
- `mi` 表示单位为英里。
- `ft` 表示单位为英尺。

在给定以下可选项时， 命令会返回额外的信息：

- `WITHDIST` ： 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
- `WITHCOORD` ： 将位置元素的经度和维度也一并返回。
- `WITHHASH` ： 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。

命令默认返回未排序的位置元素。 通过以下两个参数， 用户可以指定被返回位置元素的排序方式：

- `ASC` ： 根据中心的位置， 按照从近到远的方式返回位置元素。
- `DESC` ： 根据中心的位置， 按照从远到近的方式返回位置元素。

在默认情况下， `GEORADIUS` 命令会返回所有匹配的位置元素。 虽然用户可以使用 `COUNT ` 选项去获取前 N 个匹配元素， 但是因为命令在内部可能会需要对所有被匹配的元素进行处理， 所以在对一个非常大的区域进行搜索时， 即使只使用 `COUNT` 选项去获取少量元素， 命令的执行速度也可能会非常慢。 但是从另一方面来说， 使用 `COUNT` 选项去减少需要返回的元素数量， 对于减少带宽来说仍然是非常有用的。

### 返回值

`GEORADIUS` 命令返回一个数组， 具体来说：

- 在没有给定任何 `WITH` 选项的情况下， 命令只会返回一个像 `["New York","Milan","Paris"]` 这样的线性（linear）列表。
- 在指定了 `WITHCOORD` 、 `WITHDIST` 、 `WITHHASH` 等选项的情况下， 命令返回一个二层嵌套数组， 内层的每个子数组就表示一个元素。

在返回嵌套数组时， 子数组的第一个元素总是位置元素的名字。 至于额外的信息， 则会作为子数组的后续元素， 按照以下顺序被返回：

1. 以浮点数格式返回的中心与位置元素之间的距离， 单位与用户指定范围时的单位一致。
2. geohash 整数。
3. 由两个元素组成的坐标，分别为经度和纬度。

举个例子， `GEORADIUS Sicily 15 37 200 km withcoord withdist` 这样的命令返回的每个子数组都是类似以下格式的：

```
["Palermo","190.4424",["13.361389338970184","38.115556395496299"]]
```

### 代码示例

## GEORADIUSBYMEMBER

**GEORADIUSBYMEMBER key member radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [ASC|DESC] [COUNT count]**

这个命令和 `GEORADIUS` 命令一样， 都可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 `GEORADIUS` 那样， 使用输入的经度和纬度来决定中心点。

关于 `GEORADIUSBYMEMBER` 命令的更多信息， 请参考 `GEORADIUS` 命令的文档。

### 返回值

一个数组， 数组中的每个项表示一个范围之内的位置元素。

### 代码示例

## GEOHASH

GEOHASH key member [member …]

返回一个或多个位置元素的 [Geohash](https://en.wikipedia.org/wiki/Geohash) 表示。

### 返回值

一个数组， 数组的每个项都是一个 geohash 。 命令返回的 geohash 的位置与用户给定的位置元素的位置一一对应。

### 代码示例

```bash
redis> GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
(integer) 2

redis> GEOHASH Sicily Palermo Catania
1) "sqc8b49rny0"
2) "sqdtr74hyu0"
```

# 位图

## SETBIT

**SETBIT key offset value**

对 `key` 所储存的字符串值，设置或清除指定偏移量上的位(bit)。

位的设置或清除取决于 `value` 参数，可以是 `0` 也可以是 `1` 。

当 `key` 不存在时，自动生成一个新的字符串值。

字符串会进行伸展(grown)以确保它可以将 `value` 保存在指定的偏移量上。当字符串值进行伸展时，空白位置以 `0` 填充。

`offset` 参数必须大于或等于 `0` ，小于 2^32 (bit 映射被限制在 512 MB 之内)。

Warning

对使用大的 `offset` 的 [SETBIT](#setbit) 操作来说，内存分配可能造成 Redis 服务器被阻塞。具体参考 SETRANGE key offset value 命令，warning(警告)部分。

### 返回值

指定偏移量原来储存的位。

### 代码示例

```bash
redis> SETBIT bit 10086 1
(integer) 0

redis> GETBIT bit 10086
(integer) 1

redis> GETBIT bit 100   # bit 默认被初始化为 0
(integer) 0
```

## GETBIT

**GETBIT key offset**

对 `key` 所储存的字符串值，获取指定偏移量上的位(bit)。

当 `offset` 比字符串值的长度大，或者 `key` 不存在时，返回 `0` 。

### 返回值

字符串值指定偏移量上的位(bit)。

### 代码示例

```bash
# 对不存在的 key 或者不存在的 offset 进行 GETBIT， 返回 0

redis> EXISTS bit
(integer) 0

redis> GETBIT bit 10086
(integer) 0


# 对已存在的 offset 进行 GETBIT

redis> SETBIT bit 10086 1
(integer) 0

redis> GETBIT bit 10086
(integer) 1
```

## BITCOUNT

BITCOUNT key [start] [end]

计算给定字符串中，被设置为 `1` 的比特位的数量。

一般情况下，给定的整个字符串都会被进行计数，通过指定额外的 `start` 或 `end` 参数，可以让计数只在特定的位上进行。

`start` 和 `end` 参数的设置和 GETRANGE key start end 命令类似，都可以使用负数值： 比如 `-1` 表示最后一个字节， `-2` 表示倒数第二个字节，以此类推。

不存在的 `key` 被当成是空字符串来处理，因此对一个不存在的 `key` 进行 `BITCOUNT` 操作，结果为 `0` 。

### 返回值

被设置为 `1` 的位的数量。

### 代码示例

```
redis> BITCOUNT bits
(integer) 0

redis> SETBIT bits 0 1          # 0001
(integer) 0

redis> BITCOUNT bits
(integer) 1

redis> SETBIT bits 3 1          # 1001
(integer) 0

redis> BITCOUNT bits
(integer) 2
```

### 模式：使用 bitmap 实现用户上线次数统计

Bitmap 对于一些特定类型的计算非常有效。

假设现在我们希望记录自己网站上的用户的上线频率，比如说，计算用户 A 上线了多少天，用户 B 上线了多少天，诸如此类，以此作为数据，从而决定让哪些用户参加 beta 测试等活动 —— 这个模式可以使用 SETBIT key offset value 和 BITCOUNT key [start\] [end] 来实现。

比如说，每当用户在某一天上线的时候，我们就使用 SETBIT key offset value ，以用户名作为 `key` ，将那天所代表的网站的上线日作为 `offset` 参数，并将这个 `offset` 上的为设置为 `1` 。

举个例子，如果今天是网站上线的第 100 天，而用户 peter 在今天阅览过网站，那么执行命令 `SETBIT peter 100 1` ；如果明天 peter 也继续阅览网站，那么执行命令 `SETBIT peter 101 1` ，以此类推。

当要计算 peter 总共以来的上线次数时，就使用 BITCOUNT key [start\] [end] 命令：执行 `BITCOUNT peter` ，得出的结果就是 peter 上线的总天数。

更详细的实现可以参考博文(墙外) [Fast, easy, realtime metrics using Redis bitmaps](http://blog.getspool.com/2011/11/29/fast-easy-realtime-metrics-using-redis-bitmaps/) 。

### 性能

前面的上线次数统计例子，即使运行 10 年，占用的空间也只是每个用户 10*365 比特位(bit)，也即是每个用户 456 字节。对于这种大小的数据来说， BITCOUNT key [start\] [end] 的处理速度就像 [GET key](#get) 和 [INCR key](#incr) 这种 O(1) 复杂度的操作一样快。

如果你的 bitmap 数据非常大，那么可以考虑使用以下两种方法：

1. 将一个大的 bitmap 分散到不同的 key 中，作为小的 bitmap 来处理。使用 Lua 脚本可以很方便地完成这一工作。
2. 使用 BITCOUNT key [start\] [end] 的 `start` 和 `end` 参数，每次只对所需的部分位进行计算，将位的累积工作(accumulating)放到客户端进行，并且对结果进行缓存 (caching)。

## BITPOS

**BITPOS key bit [start] [end]**

返回位图中第一个值为 `bit` 的二进制位的位置。

在默认情况下， 命令将检测整个位图， 但用户也可以通过可选的 `start` 参数和 `end` 参数指定要检测的范围。

### 返回值

整数回复。

### 代码示例

```bash
127.0.0.1:6379> SETBIT bits 3 1    # 1000
(integer) 0

127.0.0.1:6379> BITPOS bits 0
(integer) 0

127.0.0.1:6379> BITPOS bits 1
(integer) 3
```

## BITOP

**BITOP operation destkey key [key …]**

对一个或多个保存二进制位的字符串 `key` 进行位元操作，并将结果保存到 `destkey` 上。

`operation` 可以是 `AND` 、 `OR` 、 `NOT` 、 `XOR` 这四种操作中的任意一种：

- `BITOP AND destkey key [key ...]` ，对一个或多个 `key` 求逻辑并，并将结果保存到 `destkey` 。
- `BITOP OR destkey key [key ...]` ，对一个或多个 `key` 求逻辑或，并将结果保存到 `destkey` 。
- `BITOP XOR destkey key [key ...]` ，对一个或多个 `key` 求逻辑异或，并将结果保存到 `destkey` 。
- `BITOP NOT destkey key` ，对给定 `key` 求逻辑非，并将结果保存到 `destkey` 。

除了 `NOT` 操作之外，其他操作都可以接受一个或多个 `key` 作为输入。

### 处理不同长度的字符串

当 [BITOP](#bitop) 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 `0` 。

空的 `key` 也被看作是包含 `0` 的字符串序列。

### 返回值

保存到 `destkey` 的字符串的长度，和输入 `key` 中最长的字符串长度相等。

Note

[BITOP](#bitop) 的复杂度为 O(N) ，当处理大型矩阵(matrix)或者进行大数据量的统计时，最好将任务指派到附属节点(slave)进行，避免阻塞主节点。

### 代码示例

## BITFIELD

**BITFIELD key [GET type offset] [SET type offset value] [INCRBY type offset increment] [OVERFLOW WRAP|SAT|FAIL]**

`BITFIELD` 命令可以将一个 Redis 字符串看作是一个由二进制位组成的数组， 并对这个数组中储存的长度不同的整数进行访问 （被储存的整数无需进行对齐）。 换句话说， 通过这个命令， 用户可以执行诸如 “对偏移量 1234 上的 5 位长有符号整数进行设置”、 “获取偏移量 4567 上的 31 位长无符号整数”等操作。 此外， `BITFIELD` 命令还可以对指定的整数执行加法操作和减法操作， 并且这些操作可以通过设置妥善地处理计算时出现的溢出情况。

`BITFIELD` 命令可以在一次调用中同时对多个位范围进行操作： 它接受一系列待执行的操作作为参数， 并返回一个数组作为回复， 数组中的每个元素就是对应操作的执行结果。

比如以下命令就展示了如何对位于偏移量 100 的 8 位长有符号整数执行加法操作， 并获取位于偏移量 0 上的 4 位长无符号整数：

```
> BITFIELD mykey INCRBY i8 100 1 GET u4 0
1) (integer) 1
2) (integer) 0
```

注意：

- 使用 `GET` 子命令对超出字符串当前范围的二进制位进行访问（包括键不存在的情况）， 超出部分的二进制位的值将被当做是 0 。
- 使用 `SET` 子命令或者 `INCRBY` 子命令对超出字符串当前范围的二进制位进行访问将导致字符串被扩大， 被扩大的部分会使用值为 0 的二进制位进行填充。 在对字符串进行扩展时， 命令会根据字符串目前已有的最远端二进制位， 计算出执行操作所需的最小长度。

### 支持的子命令以及数字类型

以下是 `BITFIELD` 命令支持的子命令：

- `GET ` —— 返回指定的二进制位范围。
- `SET  ` —— 对指定的二进制位范围进行设置，并返回它的旧值。
- `INCRBY  ` —— 对指定的二进制位范围执行加法操作，并返回它的旧值。用户可以通过向 `increment` 参数传入负值来实现相应的减法操作。

除了以上三个子命令之外， 还有一个子命令， 它可以改变之后执行的 `INCRBY` 子命令在发生溢出情况时的行为：

- `OVERFLOW [WRAP|SAT|FAIL]`

当被设置的二进制位范围值为整数时， 用户可以在类型参数的前面添加 `i` 来表示有符号整数， 或者使用 `u` 来表示无符号整数。 比如说， 我们可以使用 `u8` 来表示 8 位长的无符号整数， 也可以使用 `i16` 来表示 16 位长的有符号整数。

`BITFIELD` 命令最大支持 64 位长的有符号整数以及 63 位长的无符号整数， 其中无符号整数的 63 位长度限制是由于 Redis 协议目前还无法返回 64 位长的无符号整数而导致的。

### 二进制位和位置偏移量

在二进制位范围命令中， 用户有两种方法来设置偏移量：

- 如果用户给定的是一个没有任何前缀的数字， 那么这个数字指示的就是字符串以零为开始（zero-base）的偏移量。
- 另一方面， 如果用户给定的是一个带有 `#` 前缀的偏移量， 那么命令将使用这个偏移量与被设置的数字类型的位长度相乘， 从而计算出真正的偏移量。

比如说， 对于以下这个命令来说：

```
BITFIELD mystring SET i8 #0 100 i8 #1 200
```

命令会把 `mystring` 键里面， 第一个 `i8` 长度的二进制位的值设置为 `100` ， 并把第二个 `i8` 长度的二进制位的值设置为 `200` 。 当我们把一个字符串键当成数组来使用， 并且数组中储存的都是同等长度的整数时， 使用 `#` 前缀可以让我们免去手动计算被设置二进制位所在位置的麻烦。

### 溢出控制

用户可以通过 `OVERFLOW` 命令以及以下展示的三个参数， 指定 `BITFIELD` 命令在执行自增或者自减操作时， 碰上向上溢出（overflow）或者向下溢出（underflow）情况时的行为：

- `WRAP` ： 使用回绕（wrap around）方法处理有符号整数和无符号整数的溢出情况。 对于无符号整数来说， 回绕就像使用数值本身与能够被储存的最大无符号整数执行取模计算， 这也是 C 语言的标准行为。 对于有符号整数来说， 上溢将导致数字重新从最小的负数开始计算， 而下溢将导致数字重新从最大的正数开始计算。 比如说， 如果我们对一个值为 `127` 的 `i8` 整数执行加一操作， 那么将得到结果 `-128` 。
- `SAT` ： 使用饱和计算（saturation arithmetic）方法处理溢出， 也即是说， 下溢计算的结果为最小的整数值， 而上溢计算的结果为最大的整数值。 举个例子， 如果我们对一个值为 `120` 的 `i8` 整数执行加 `10` 计算， 那么命令的结果将为 `i8` 类型所能储存的最大整数值 `127` 。 与此相反， 如果一个针对 `i8` 值的计算造成了下溢， 那么这个 `i8` 值将被设置为 `-127` 。
- `FAIL` ： 在这一模式下， 命令将拒绝执行那些会导致上溢或者下溢情况出现的计算， 并向用户返回空值表示计算未被执行。

需要注意的是， `OVERFLOW` 子命令只会对紧随着它之后被执行的 `INCRBY` 命令产生效果， 这一效果将一直持续到与它一同被执行的下一个 `OVERFLOW` 命令为止。 在默认情况下， `INCRBY` 命令使用 `WRAP` 方式来处理溢出计算。

以下是一个使用 `OVERFLOW` 子命令来控制溢出行为的例子：

```bash
> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 1
2) (integer) 1

> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 2
2) (integer) 2

> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 3
2) (integer) 3

> BITFIELD mykey incrby u2 100 1 OVERFLOW SAT incrby u2 102 1
1) (integer) 0  -- 使用默认的 WRAP 方式处理溢出
2) (integer) 3  -- 使用 SAT 方式处理溢出
```

而以下则是一个因为 `OVERFLOW FAIL` 行为而导致子命令返回空值的例子：

```bash
> BITFIELD mykey OVERFLOW FAIL incrby u2 102 1
1) (nil)
```

### 作用

`BITFIELD` 命令的作用在于它能够将很多小的整数储存到一个长度较大的位图中， 又或者将一个非常庞大的键分割为多个较小的键来进行储存， 从而非常高效地使用内存， 使得 Redis 能够得到更多不同的应用 —— 特别是在实时分析领域： `BITFIELD` 能够以指定的方式对计算溢出进行控制的能力， 使得它可以被应用于这一领域。

### 性能注意事项

`BITFIELD` 在一般情况下都是一个快速的命令， 需要注意的是， 访问一个长度较短的字符串的远端二进制位将引发一次内存分配操作， 这一操作花费的时间可能会比命令访问已有的字符串花费的时间要长。

### 二进制位的排列

`BITFIELD` 把位图第一个字节偏移量 0 上的二进制位看作是 most significant 位， 以此类推。 举个例子， 如果我们对一个已经预先被全部设置为 0 的位图进行设置， 将它在偏移量 7 的值设置为 5 位无符号整数值 23 （二进制位为 `10111` ）， 那么命令将生产出以下这个位图表示：

```
+--------+--------+
|00000001|01110000|
+--------+--------+
```

当偏移量和整数长度与字节边界进行对齐时， `BITFIELD` 表示二进制位的方式跟大端表示法（big endian）一致， 但是在没有对齐的情况下， 理解这些二进制位是如何进行排列也是非常重要的。

### 返回值

`BITFIELD` 命令的返回值是一个数组， 数组中的每个元素对应一个被执行的子命令。 需要注意的是， `OVERFLOW` 子命令本身并不产生任何回复。

# STEAM流

# 数据库

## EXISTS

**EXISTS key**

检查给定 `key` 是否存在。

### 返回值

若 `key` 存在，返回 `1` ，否则返回 `0` 。

### 代码示例

```bash
redis> SET db "redis"
OK

redis> EXISTS db
(integer) 1

redis> DEL db
(integer) 1

redis> EXISTS db
(integer) 0
```

## TYPE

TYPE key

返回 `key` 所储存的值的类型。

### 返回值

- `none` (key不存在)
- `string` (字符串)
- `list` (列表)
- `set` (集合)
- `zset` (有序集)
- `hash` (哈希表)
- `stream` （流）

### 代码示例

```bash
# 字符串

redis> SET weather "sunny"
OK

redis> TYPE weather
string


# 列表

redis> LPUSH book_list "programming in scala"
(integer) 1

redis> TYPE book_list
list


# 集合

redis> SADD pat "dog"
(integer) 1

redis> TYPE pat
set
```

## RENAME

**RENAME key newkey**

将 `key` 改名为 `newkey` 。

当 `key` 和 `newkey` 相同，或者 `key` 不存在时，返回一个错误。

当 `newkey` 已经存在时， [RENAME](#rename) 命令将覆盖旧值。

### 返回值

改名成功时提示 `OK` ，失败时候返回一个错误。

### 代码示例

## RENAMENX

**RENAMENX key newkey**

当且仅当 `newkey` 不存在时，将 `key` 改名为 `newkey` 。

当 `key` 不存在时，返回一个错误。

### 返回值

修改成功时，返回 `1` ； 如果 `newkey` 已经存在，返回 `0` 。

### 代码示例

## MOVE

**MOVE key db**

将当前数据库的 `key` 移动到给定的数据库 `db` 当中。

如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 `key` ，或者 `key` 不存在于当前数据库，那么 `MOVE` 没有任何效果。

因此，也可以利用这一特性，将 [MOVE](#move) 当作锁(locking)原语(primitive)。

### 返回值

移动成功返回 `1` ，失败则返回 `0` 。

### 代码示例

## DEL

**DEL key [key …]**

删除给定的一个或多个 `key` 。

不存在的 `key` 会被忽略。

### 返回值

被删除 `key` 的数量。

### 代码示例

## RANDOMKEY

**RANDOMKEY**

从当前数据库中随机返回(不删除)一个 `key` 。

### 返回值

当数据库不为空时，返回一个 `key` 。 当数据库为空时，返回 `nil` 。

### 代码示例

## DBSIZE

**DBSIZE**

返回当前数据库的 key 的数量。

### 返回值

当前数据库的 key 的数量。

### 代码示例

```bash
redis> DBSIZE
(integer) 5

redis> SET new_key "hello_moto"     # 增加一个 key 试试
OK

redis> DBSIZE
(integer) 6
```

## KEYS

**KEYS pattern**

查找所有符合给定模式 `pattern` 的 `key` ， 比如说：

- `KEYS *` 匹配数据库中所有 `key` 。
- `KEYS h?llo` 匹配 `hello` ， `hallo` 和 `hxllo` 等。
- `KEYS h*llo` 匹配 `hllo` 和 `heeeeello` 等。
- `KEYS h[ae]llo` 匹配 `hello` 和 `hallo` ，但不匹配 `hillo` 。

特殊符号用 `\` 隔开。

Warning

[KEYS](#keys) 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如果你需要从一个数据集中查找特定的 `key` ，你最好还是用 Redis 的集合结构(set)来代替。

### 返回值

符合给定模式的 `key` 列表。

### 代码示例

```bash
redis> MSET one 1 two 2 three 3 four 4  # 一次设置 4 个 key
OK

redis> KEYS *o*
1) "four"
2) "two"
3) "one"

redis> KEYS t??
1) "two"

redis> KEYS t[w]*
1) "two"

redis> KEYS *  # 匹配数据库内所有 key
1) "four"
2) "three"
3) "two"
4) "one"
```

## SCAN

**SCAN cursor [MATCH pattern] [COUNT count]**

`SCAN` 命令及其相关的 `SSCAN` 命令、 `HSCAN` 命令和 `ZSCAN` 命令都用于增量地迭代（incrementally iterate）一集元素（a collection of elements）：

- `SCAN` 命令用于迭代当前数据库中的数据库键。
- `SSCAN` 命令用于迭代集合键中的元素。
- `HSCAN` 命令用于迭代哈希键中的键值对。
- `ZSCAN` 命令用于迭代有序集合中的元素（包括元素成员和元素分值）。

以上列出的四个命令都支持增量式迭代， 它们每次执行都只会返回少量元素， 所以这些命令可以用于生产环境， 而不会出现像 `KEYS` 命令、 `SMEMBERS` 命令带来的问题 —— 当 `KEYS` 命令被用于处理一个大的数据库时， 又或者 `SMEMBERS` 命令被用于处理一个大的集合键时， 它们可能会阻塞服务器达数秒之久。

不过， 增量式迭代命令也不是没有缺点的： 举个例子， 使用 `SMEMBERS` 命令可以返回集合键当前包含的所有元素， 但是对于 `SCAN` 这类增量式迭代命令来说， 因为在对键进行增量式迭代的过程中， 键可能会被修改， 所以增量式迭代命令只能对被返回的元素提供有限的保证 （offer limited guarantees about the returned elements）。

因为 `SCAN` 、 `SSCAN` 、 `HSCAN` 和 `ZSCAN` 四个命令的工作方式都非常相似， 所以这个文档会一并介绍这四个命令， 但是要记住：

- `SSCAN` 命令、 `HSCAN` 命令和 `ZSCAN` 命令的第一个参数总是一个数据库键。
- 而 `SCAN` 命令则不需要在第一个参数提供任何数据库键 —— 因为它迭代的是当前数据库中的所有数据库键。

### SCAN 命令的基本用法

`SCAN` 命令是一个基于游标的迭代器（cursor based iterator）： `SCAN` 命令每次被调用之后， 都会向用户返回一个新的游标， 用户在下次迭代时需要使用这个新游标作为 `SCAN` 命令的游标参数， 以此来延续之前的迭代过程。

当 `SCAN` 命令的游标参数被设置为 `0` 时， 服务器将开始一次新的迭代， 而当服务器向用户返回值为 `0` 的游标时， 表示迭代已结束。

以下是一个 `SCAN` 命令的迭代过程示例：

### SCAN 命令的保证（guarantees）

`SCAN` 命令， 以及其他增量式迭代命令， 在进行完整遍历的情况下可以为用户带来以下保证： 从完整遍历开始直到完整遍历结束期间， 一直存在于数据集内的所有元素都会被完整遍历返回； 这意味着， 如果有一个元素， 它从遍历开始直到遍历结束期间都存在于被遍历的数据集当中， 那么 `SCAN` 命令总会在某次迭代中将这个元素返回给用户。

然而因为增量式命令仅仅使用游标来记录迭代状态， 所以这些命令带有以下缺点：

- 同一个元素可能会被返回多次。 处理重复元素的工作交由应用程序负责， 比如说， 可以考虑将迭代返回的元素仅仅用于可以安全地重复执行多次的操作上。
- 如果一个元素是在迭代过程中被添加到数据集的， 又或者是在迭代过程中从数据集中被删除的， 那么这个元素可能会被返回， 也可能不会， 这是未定义的（undefined）。

### SCAN 命令每次执行返回的元素数量

增量式迭代命令并不保证每次执行都返回某个给定数量的元素。

增量式命令甚至可能会返回零个元素， 但只要命令返回的游标不是 `0` ， 应用程序就不应该将迭代视作结束。

不过命令返回的元素数量总是符合一定规则的， 在实际中：

- 对于一个大数据集来说， 增量式迭代命令每次最多可能会返回数十个元素；
- 而对于一个足够小的数据集来说， 如果这个数据集的底层表示为编码数据结构（encoded data structure，适用于是小集合键、小哈希键和小有序集合键）， 那么增量迭代命令将在一次调用中返回数据集中的所有元素。

最后， 用户可以通过增量式迭代命令提供的 `COUNT` 选项来指定每次迭代返回元素的最大值。

### COUNT 选项

虽然增量式迭代命令不保证每次迭代所返回的元素数量， 但我们可以使用 `COUNT` 选项， 对命令的行为进行一定程度上的调整。

基本上， `COUNT` 选项的作用就是让用户告知迭代命令， 在每次迭代中应该从数据集里返回多少元素。

虽然 `COUNT` 选项**只是对增量式迭代命令的一种提示**（hint）， 但是在大多数情况下， 这种提示都是有效的。

- `COUNT` 参数的默认值为 `10` 。
- 在迭代一个足够大的、由哈希表实现的数据库、集合键、哈希键或者有序集合键时， 如果用户没有使用 `MATCH` 选项， 那么命令返回的元素数量通常和 `COUNT` 选项指定的一样， 或者比 `COUNT` 选项指定的数量稍多一些。
- 在迭代一个编码为整数集合（intset，一个只由整数值构成的小集合）、 或者编码为压缩列表（ziplist，由不同值构成的一个小哈希或者一个小有序集合）时， 增量式迭代命令通常会无视 `COUNT` 选项指定的值， 在第一次迭代就将数据集包含的所有元素都返回给用户。

Note

**并非每次迭代都要使用相同的** `COUNT` **值。**

用户可以在每次迭代中按自己的需要随意改变 `COUNT` 值， 只要记得将上次迭代返回的游标用到下次迭代里面就可以了。

### MATCH 选项

和 `KEYS` 命令一样， 增量式迭代命令也可以通过提供一个 glob 风格的模式参数， 让命令只返回和给定模式相匹配的元素， 这一点可以通过在执行增量式迭代命令时， 通过给定 `MATCH ` 参数来实现。

以下是一个使用 `MATCH` 选项进行迭代的示例：

```
redis 127.0.0.1:6379> sadd myset 1 2 3 foo foobar feelsgood
(integer) 6

redis 127.0.0.1:6379> sscan myset 0 match f*
1) "0"
2) 1) "foo"
   2) "feelsgood"
   3) "foobar"
```

需要注意的是， 对元素的模式匹配工作是在命令从数据集中取出元素之后， 向客户端返回元素之前的这段时间内进行的， 所以如果被迭代的数据集中只有少量元素和模式相匹配， 那么迭代命令或许会在多次执行中都不返回任何元素。

以下是这种情况的一个例子：

```
redis 127.0.0.1:6379> scan 0 MATCH *11*
1) "288"
2) 1) "key:911"

redis 127.0.0.1:6379> scan 288 MATCH *11*
1) "224"
2) (empty list or set)

redis 127.0.0.1:6379> scan 224 MATCH *11*
1) "80"
2) (empty list or set)

redis 127.0.0.1:6379> scan 80 MATCH *11*
1) "176"
2) (empty list or set)

redis 127.0.0.1:6379> scan 176 MATCH *11* COUNT 1000
1) "0"
2)  1) "key:611"
    2) "key:711"
    3) "key:118"
    4) "key:117"
    5) "key:311"
    6) "key:112"
    7) "key:111"
    8) "key:110"
    9) "key:113"
   10) "key:211"
   11) "key:411"
   12) "key:115"
   13) "key:116"
   14) "key:114"
   15) "key:119"
   16) "key:811"
   17) "key:511"
   18) "key:11"
```

如你所见， 以上的大部分迭代都不返回任何元素。

在最后一次迭代， 我们通过将 `COUNT` 选项的参数设置为 `1000` ， 强制命令为本次迭代扫描更多元素， 从而使得命令返回的元素也变多了。

### 并发执行多个迭代

在同一时间， 可以有任意多个客户端对同一数据集进行迭代， 客户端每次执行迭代都需要传入一个游标， 并在迭代执行之后获得一个新的游标， 而这个游标就包含了迭代的所有状态， 因此， 服务器无须为迭代记录任何状态。

### 中途停止迭代

因为迭代的所有状态都保存在游标里面， 而服务器无须为迭代保存任何状态， 所以客户端可以在中途停止一个迭代， 而无须对服务器进行任何通知。

即使有任意数量的迭代在中途停止， 也不会产生任何问题。

### 使用错误的游标进行增量式迭代

使用间断的（broken）、负数、超出范围或者其他非正常的游标来执行增量式迭代并不会造成服务器崩溃， 但可能会让命令产生未定义的行为。

未定义行为指的是， 增量式命令对返回值所做的保证可能会不再为真。

只有两种游标是合法的：

1. 在开始一个新的迭代时， 游标必须为 `0` 。
2. 增量式迭代命令在执行之后返回的， 用于延续（continue）迭代过程的游标。

### 迭代终结的保证

增量式迭代命令所使用的算法只保证在数据集的大小有界（bounded）的情况下， 迭代才会停止， 换句话说， 如果被迭代数据集的大小不断地增长的话， 增量式迭代命令可能永远也无法完成一次完整迭代。

从直觉上可以看出， 当一个数据集不断地变大时， 想要访问这个数据集中的所有元素就需要做越来越多的工作， 能否结束一个迭代取决于用户执行迭代的速度是否比数据集增长的速度更快。

### 返回值

`SCAN` 命令、 `SSCAN` 命令、 `HSCAN` 命令和 `ZSCAN` 命令都返回一个包含两个元素的 multi-bulk 回复： 回复的第一个元素是字符串表示的无符号 64 位整数（游标）， 回复的第二个元素是另一个 multi-bulk 回复， 这个 multi-bulk 回复包含了本次被迭代的元素。

`SCAN` 命令返回的每个元素都是一个数据库键。

`SSCAN` 命令返回的每个元素都是一个集合成员。

`HSCAN` 命令返回的每个元素都是一个键值对，一个键值对由一个键和一个值组成。

`ZSCAN` 命令返回的每个元素都是一个有序集合元素，一个有序集合元素由一个成员（member）和一个分值（score）组成。

## SORT

**SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern …]] [ASC | DESC] [ALPHA] [STORE destination]**

返回或保存给定列表、集合、有序集合 `key` 中经过排序的元素。

排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。

### 一般 SORT 用法

最简单的 [SORT](#sort) 使用方法是 `SORT key` 和 `SORT key DESC` ：

- `SORT key` 返回键值从小到大排序的结果。
- `SORT key DESC` 返回键值从大到小排序的结果。

假设 `today_cost` 列表保存了今日的开销金额， 那么可以用 [SORT](#sort) 命令对它进行排序：

```bash
# 开销金额列表

redis> LPUSH today_cost 30 1.5 10 8
(integer) 4

# 排序

redis> SORT today_cost
1) "1.5"
2) "8"
3) "10"
4) "30"

# 逆序排序

redis 127.0.0.1:6379> SORT today_cost DESC
1) "30"
2) "10"
3) "8"
4) "1.5"
```

### 使用 ALPHA 修饰符对字符串进行排序

因为 [SORT](#sort) 命令默认排序对象为数字， 当需要对字符串进行排序时， 需要显式地在 [SORT](#sort) 命令之后添加 `ALPHA` 修饰符：

```bash
# 网址

redis> LPUSH website "www.reddit.com"
(integer) 1

redis> LPUSH website "www.slashdot.com"
(integer) 2

redis> LPUSH website "www.infoq.com"
(integer) 3

# 默认（按数字）排序

redis> SORT website
1) "www.infoq.com"
2) "www.slashdot.com"
3) "www.reddit.com"

# 按字符排序

redis> SORT website ALPHA
1) "www.infoq.com"
2) "www.reddit.com"
3) "www.slashdot.com"
```

如果系统正确地设置了 `LC_COLLATE` 环境变量的话，Redis能识别 `UTF-8` 编码。

### 使用 LIMIT 修饰符限制返回结果

排序之后返回元素的数量可以通过 `LIMIT` 修饰符进行限制， 修饰符接受 `offset` 和 `count` 两个参数：

- `offset` 指定要跳过的元素数量。
- `count` 指定跳过 `offset` 个指定的元素之后，要返回多少个对象。

以下例子返回排序结果的前 5 个对象( `offset` 为 `0` 表示没有元素被跳过)。

```bash
# 添加测试数据，列表值为 1 指 10

redis 127.0.0.1:6379> RPUSH rank 1 3 5 7 9
(integer) 5

redis 127.0.0.1:6379> RPUSH rank 2 4 6 8 10
(integer) 10

# 返回列表中最小的 5 个值

redis 127.0.0.1:6379> SORT rank LIMIT 0 5
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

可以组合使用多个修饰符。以下例子返回从大到小排序的前 5 个对象。

```bash
redis 127.0.0.1:6379> SORT rank LIMIT 0 5 DESC
1) "10"
2) "9"
3) "8"
4) "7"
5) "6"
```

### 使用外部 key 进行排序

可以使用外部 `key` 的数据作为权重，代替默认的直接对比键值的方式来进行排序。

假设现在有用户数据如下：

以下代码将数据输入到 Redis 中：

```bash
# admin

redis 127.0.0.1:6379> LPUSH uid 1
(integer) 1

redis 127.0.0.1:6379> SET user_name_1 admin
OK

redis 127.0.0.1:6379> SET user_level_1 9999
OK

# jack

redis 127.0.0.1:6379> LPUSH uid 2
(integer) 2

redis 127.0.0.1:6379> SET user_name_2 jack
OK

redis 127.0.0.1:6379> SET user_level_2 10
OK

# peter

redis 127.0.0.1:6379> LPUSH uid 3
(integer) 3

redis 127.0.0.1:6379> SET user_name_3 peter
OK

redis 127.0.0.1:6379> SET user_level_3 25
OK

# mary

redis 127.0.0.1:6379> LPUSH uid 4
(integer) 4

redis 127.0.0.1:6379> SET user_name_4 mary
OK

redis 127.0.0.1:6379> SET user_level_4 70
OK
```

### BY 选项

默认情况下， `SORT uid` 直接按 `uid` 中的值排序：

```bash
redis 127.0.0.1:6379> SORT uid
1) "1"      # admin
2) "2"      # jack
3) "3"      # peter
4) "4"      # mary
```

通过使用 `BY` 选项，可以让 `uid` 按其他键的元素来排序。比如说， 以下代码让 `uid` 键按照 `user_level_{uid}` 的大小来排序：

```
redis 127.0.0.1:6379> SORT uid BY user_level_*
1) "2"      # jack , level = 10
2) "3"      # peter, level = 25
3) "4"      # mary, level = 70
4) "1"      # admin, level = 9999
```

`user_level_*` 是一个占位符， 它先取出 `uid` 中的值， 然后再用这个值来查找相应的键。

比如在对 `uid` 列表进行排序时， 程序就会先取出 `uid` 的值 `1` 、 `2` 、 `3` 、 `4` ， 然后使用 `user_level_1` 、 `user_level_2` 、 `user_level_3` 和 `user_level_4` 的值作为排序 `uid` 的权重。

### GET 选项

使用 `GET` 选项， 可以根据排序的结果来取出相应的键值。

比如说， 以下代码先排序 `uid` ， 再取出键 `user_name_{uid}` 的值：

```bash
redis 127.0.0.1:6379> SORT uid GET user_name_*
1) "admin"
2) "jack"
3) "peter"
4) "mary"
```

### 组合使用 BY 和 GET

通过组合使用 `BY` 和 `GET` ， 可以让排序结果以更直观的方式显示出来。比如说， 以下代码先按 `user_level_{uid}` 来排序 `uid` 列表， 再取出相应的 `user_name_{uid}` 的值：

```bash
redis 127.0.0.1:6379> SORT uid BY user_level_* GET user_name_*
1) "jack"       # level = 10
2) "peter"      # level = 25
3) "mary"       # level = 70
4) "admin"      # level = 9999
```

现在的排序结果要比只使用 `SORT uid BY user_level_*` 要直观得多。

### 获取多个外部键

可以同时使用多个 `GET` 选项， 获取多个外部键的值。以下代码就按 `uid` 分别获取 `user_level_{uid}` 和 `user_name_{uid}` ：

```bash
redis 127.0.0.1:6379> SORT uid GET user_level_* GET user_name_*
1) "9999"       # level
2) "admin"      # name
3) "10"
4) "jack"
5) "25"
6) "peter"
7) "70"
8) "mary"
```

`GET` 有一个额外的参数规则，那就是 —— 可以用 `#` 获取被排序键的值。

以下代码就将 `uid` 的值、及其相应的 `user_level_*` 和 `user_name_*` 都返回为结果：

```bash
redis 127.0.0.1:6379> SORT uid GET # GET user_level_* GET user_name_*
1) "1"          # uid
2) "9999"       # level
3) "admin"      # name
4) "2"
5) "10"
6) "jack"
7) "3"
8) "25"
9) "peter"
10) "4"
11) "70"
12) "mary"
```

### 获取外部键，但不进行排序

通过将一个不存在的键作为参数传给 `BY` 选项， 可以让 `SORT` 跳过排序操作， 直接返回结果：

```bash
redis 127.0.0.1:6379> SORT uid BY not-exists-key
1) "4"
2) "3"
3) "2"
4) "1"
```

这种用法在单独使用时，没什么实际用处。

不过，通过将这种用法和 `GET` 选项配合， 就可以在不排序的情况下， 获取多个外部键， 相当于执行一个整合的获取操作（类似于 SQL 数据库的 `join` 关键字）。以下代码演示了，如何在不引起排序的情况下，使用 `SORT` 、 `BY` 和 `GET` 获取多个外部键：

```bash
redis 127.0.0.1:6379> SORT uid BY not-exists-key GET # GET user_level_* GET user_name_*
1) "4"      # id
2) "70"     # level
3) "mary"   # name
4) "3"
5) "25"
6) "peter"
7) "2"
8) "10"
9) "jack"
10) "1"
11) "9999"
12) "admin"
```

### 将哈希表作为 GET 或 BY 的参数

除了可以将字符串键之外， 哈希表也可以作为 `GET` 或 `BY` 选项的参数来使用。

比如说，对于前面给出的用户信息表：

我们可以不将用户的名字和级别保存在 `user_name_{uid}` 和 `user_level_{uid}` 两个字符串键中， 而是用一个带有 `name` 域和 `level` 域的哈希表 `user_info_{uid}` 来保存用户的名字和级别信息：

```bash
redis 127.0.0.1:6379> HMSET user_info_1 name admin level 9999
OK

redis 127.0.0.1:6379> HMSET user_info_2 name jack level 10
OK

redis 127.0.0.1:6379> HMSET user_info_3 name peter level 25
OK

redis 127.0.0.1:6379> HMSET user_info_4 name mary level 70
OK
```

之后， `BY` 和 `GET` 选项都可以用 `key->field` 的格式来获取哈希表中的域的值， 其中 `key` 表示哈希表键， 而 `field` 则表示哈希表的域：

```bash
redis 127.0.0.1:6379> SORT uid BY user_info_*->level
1) "2"
2) "3"
3) "4"
4) "1"

redis 127.0.0.1:6379> SORT uid BY user_info_*->level GET user_info_*->name
1) "jack"
2) "peter"
3) "mary"
4) "admin"
```

### 保存排序结果

默认情况下， [SORT](#sort) 操作只是简单地返回排序结果，并不进行任何保存操作。

通过给 `STORE` 选项指定一个 `key` 参数，可以将排序结果保存到给定的键上。

如果被指定的 `key` 已存在，那么原有的值将被排序结果覆盖。

```bash
# 测试数据

redis 127.0.0.1:6379> RPUSH numbers 1 3 5 7 9
(integer) 5

redis 127.0.0.1:6379> RPUSH numbers 2 4 6 8 10
(integer) 10

redis 127.0.0.1:6379> LRANGE numbers 0 -1
1) "1"
2) "3"
3) "5"
4) "7"

redis 127.0.0.1:6379> SORT numbers STORE sorted-numbers
(integer) 10

# 排序后的结果

redis 127.0.0.1:6379> LRANGE sorted-numbers 0 -1
```

可以通过将 [SORT](#sort) 命令的执行结果保存，并用 EXPIRE key seconds 为结果设置生存时间，以此来产生一个 [SORT](#sort) 操作的结果缓存。

这样就可以避免对 [SORT](#sort) 操作的频繁调用：只有当结果集过期时，才需要再调用一次 [SORT](#sort) 操作。

另外，为了正确实现这一用法，你可能需要加锁以避免多个客户端同时进行缓存重建(也就是多个客户端，同一时间进行 [SORT](#sort) 操作，并保存为结果集)，具体参见 SETNX key value 命令。

### 返回值

没有使用 `STORE` 参数，返回列表形式的排序结果。 使用 `STORE` 参数，返回排序结果的元素数量。

## FLUSHDB

**FLUSHDB**

清空当前数据库中的所有 key。

此命令从不失败。

### 返回值

总是返回 `OK` 。

### 代码示例

```bash
redis> DBSIZE    # 清空前的 key 数量
(integer) 4

redis> FLUSHDB
OK

redis> DBSIZE    # 清空后的 key 数量
(integer) 0
```

## FLUSHALL

**FLUSHALL**

清空整个 Redis 服务器的数据(删除所有数据库的所有 key )。

此命令从不失败。

### 返回值

总是返回 `OK` 。

```bash
redis> DBSIZE            # 0 号数据库的 key 数量
(integer) 9

redis> SELECT 1          # 切换到 1 号数据库
OK

redis[1]> DBSIZE         # 1 号数据库的 key 数量
(integer) 6

redis[1]> flushall       # 清空所有数据库的所有 key
OK

redis[1]> DBSIZE         # 不但 1 号数据库被清空了
(integer) 0

redis[1]> SELECT 0       # 0 号数据库(以及其他所有数据库)也一样
OK

redis> DBSIZE
(integer) 0
```

## SELECT

**SELECT index**

切换到指定的数据库，数据库索引号 `index` 用数字值指定，以 `0` 作为起始索引值。

默认使用 `0` 号数据库。

### 返回值

```
OK
```

### 代码示例

```bash
redis> SET db_number 0         # 默认使用 0 号数据库
OK

redis> SELECT 1                # 使用 1 号数据库
OK

redis[1]> GET db_number        # 已经切换到 1 号数据库，注意 Redis 现在的命令提示符多了个 [1]
(nil)

redis[1]> SET db_number 1
OK

redis[1]> GET db_number
"1"

redis[1]> SELECT 3             # 再切换到 3 号数据库
OK

redis[3]>     
```

## SWAPDB

**SWAPDB db1 db2**

> 版本要求： >= 4.0.0
> 
> 时间复杂度： O(1)

对换指定的两个数据库， 使得两个数据库的数据立即互换。

### 返回值

```
OK
```

### 代码示例

```bash
# 对换数据库 0 和数据库 1
redis> SWAPDB 0 1
OK
```

## MEMORY HELP

> **Available since 4.0.0.**

The MEMORY HELP command returns a helpful text describing the different subcommands.

### Return value

Array reply: a list of subcommands and their descriptions

## MEMORY MALLOC-STATS

# 自动过期

## EXPIRE

**EXPIRE key seconds**

为给定 `key` 设置生存时间，当 `key` 过期时(生存时间为 `0` )，它会被自动删除。

在 Redis 中，带有生存时间的 `key` 被称为『易失的』(volatile)。

生存时间可以通过使用 `DEL` 命令来删除整个 `key` 来移除，或者被 `SET` 和 `GETSET` 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 `key` 的值而不是用一个新的 `key` 值来代替(replace)它的话，那么生存时间不会被改变。

比如说，对一个 `key` 执行 `INCR` 命令，对一个列表进行 `LPUSH` 命令，或者对一个哈希表执行 `HSET` 命令，这类操作都不会修改 `key` 本身的生存时间。

另一方面，如果使用 `RENAME` 对一个 `key` 进行改名，那么改名后的 `key` 的生存时间和改名前一样。

`RENAME` 命令的另一种可能是，尝试将一个带生存时间的 `key` 改名成另一个带生存时间的 `another_key` ，这时旧的 `another_key` (以及它的生存时间)会被删除，然后旧的 `key` 会改名为 `another_key` ，因此，新的 `another_key` 的生存时间也和原本的 `key` 一样。

使用 `PERSIST` 命令可以在不删除 `key` 的情况下，移除 `key` 的生存时间，让 `key` 重新成为一个『持久的』(persistent) `key` 。

### 更新生存时间

可以对一个已经带有生存时间的 `key` 执行 `EXPIRE` 命令，新指定的生存时间会取代旧的生存时间。

### 过期时间的精确度

在 Redis 2.4 版本中，过期时间的延迟在 1 秒钟之内 —— 也即是，就算 `key` 已经过期，但它还是可能在过期之后一秒钟之内被访问到，而在新的 Redis 2.6 版本中，延迟被降低到 1 毫秒之内。

### Redis 2.1.3 之前的不同之处

在 Redis 2.1.3 之前的版本中，修改一个带有生存时间的 `key` 会导致整个 `key` 被删除，这一行为是受当时复制(replication)层的限制而作出的，现在这一限制已经被修复。

### 返回值

设置成功返回 `1` 。 当 `key` 不存在或者不能为 `key` 设置生存时间时(比如在低于 2.1.3 版本的 Redis 中你尝试更新 `key` 的生存时间)，返回 `0` 。

### 代码示例

```
redis> SET cache_page "www.google.com"
OK

redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
(integer) 1

redis> TTL cache_page    # 查看剩余生存时间
(integer) 23

redis> EXPIRE cache_page 30000   # 更新过期时间
(integer) 1

redis> TTL cache_page
(integer) 29996
```

### 模式：导航会话

假设你有一项 web 服务，打算根据用户最近访问的 N 个页面来进行物品推荐，并且假设用户停止阅览超过 60 秒，那么就清空阅览记录(为了减少物品推荐的计算量，并且保持推荐物品的新鲜度)。

这些最近访问的页面记录，我们称之为『导航会话』(Navigation session)，可以用 `INCR` 和 `RPUSH` 命令在 Redis 中实现它：每当用户阅览一个网页的时候，执行以下代码：

```
MULTI
    RPUSH pagewviews.user:<userid> http://.....
    EXPIRE pagewviews.user:<userid> 60
EXEC
```

如果用户停止阅览超过 60 秒，那么它的导航会话就会被清空，当用户重新开始阅览的时候，系统又会重新记录导航会话，继续进行物品推荐。

## EXPIREAT

**EXPIREAT key timestamp**

> 可用版本： >= 1.2.0
> 
> 时间复杂度： O(1)

`EXPIREAT` 的作用和 `EXPIRE` 类似，都用于为 `key` 设置生存时间。

不同在于 `EXPIREAT` 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。

### 返回值

如果生存时间设置成功，返回 `1` ； 当 `key` 不存在或没办法设置生存时间，返回 `0` 。

### 代码示例

```bash
redis> SET cache www.google.com
OK

redis> EXPIREAT cache 1355292000     # 这个 key 将在 2012.12.12 过期
(integer) 1

redis> TTL cache
(integer) 45081860
```

## TTL

**TTL key**

> 可用版本： >= 1.0.0
> 
> 时间复杂度： O(1)

以秒为单位，返回给定 `key` 的剩余生存时间(TTL, time to live)。

### 返回值

当 `key` 不存在时，返回 `-2` 。 当 `key` 存在但没有设置剩余生存时间时，返回 `-1` 。 否则，以秒为单位，返回 `key` 的剩余生存时间。

Note

在 Redis 2.8 以前，当 `key` 不存在，或者 `key` 没有设置剩余生存时间时，命令都返回 `-1` 。

### 代码示例

## PERSIST

**PERSIST key**

> 可用版本： >= 2.2.0
> 
> 时间复杂度： O(1)

移除给定 `key` 的生存时间，将这个 `key` 从“易失的”(带生存时间 `key` )转换成“持久的”(一个不带生存时间、永不过期的 `key` )。

### 返回值

当生存时间移除成功时，返回 `1` . 如果 `key` 不存在或 `key` 没有设置生存时间，返回 `0` 。

### 代码示例

```bash
redis> SET mykey "Hello"
OK

redis> EXPIRE mykey 10  # 为 key 设置生存时间
(integer) 1

redis> TTL mykey
(integer) 10

redis> PERSIST mykey    # 移除 key 的生存时间
(integer) 1

redis> TTL mykey
(integer) -1
```

## PEXPIRE

**PEXPIRE key milliseconds**

> 可用版本： >= 2.6.0
> 
> 时间复杂度： O(1)

这个命令和 `EXPIRE` 命令的作用类似，但是它以毫秒为单位设置 `key` 的生存时间，而不像 `EXPIRE` 命令那样，以秒为单位。

### 返回值

设置成功，返回 `1` `key` 不存在或设置失败，返回 `0`

### 代码示例

```bash
redis> SET mykey "Hello"
OK

redis> PEXPIRE mykey 1500
(integer) 1

redis> TTL mykey    # TTL 的返回值以秒为单位
(integer) 2

redis> PTTL mykey   # PTTL 可以给出准确的毫秒数
(integer) 1499
```

## PEXPIREAT

**PEXPIREAT key milliseconds-timestamp**

> 可用版本： >= 2.6.0
> 
> 时间复杂度： O(1)

这个命令和 `expireat` 命令类似，但它以毫秒为单位设置 `key` 的过期 unix 时间戳，而不是像 `expireat` 那样，以秒为单位。

### 返回值

如果生存时间设置成功，返回 `1` 。 当 `key` 不存在或没办法设置生存时间时，返回 `0` 。(查看 EXPIRE key seconds 命令获取更多信息)

### 代码示例

```bash
redis> SET mykey "Hello"
OK

redis> PEXPIREAT mykey 1555555555005
(integer) 1

redis> TTL mykey           # TTL 返回秒
(integer) 223157079

redis> PTTL mykey          # PTTL 返回毫秒
(integer) 223157079318
```

## PTTL

**PTTL key**

> 可用版本： >= 2.6.0
> 
> 复杂度： O(1)

这个命令类似于 `TTL` 命令，但它以毫秒为单位返回 `key` 的剩余生存时间，而不是像 `TTL` 命令那样，以秒为单位。

### 返回值

- 当 `key` 不存在时，返回 `-2` 。
- 当 `key` 存在但没有设置剩余生存时间时，返回 `-1` 。
- 否则，以毫秒为单位，返回 `key` 的剩余生存时间。

Note

在 Redis 2.8 以前，当 `key` 不存在，或者 `key` 没有设置剩余生存时间时，命令都返回 `-1` 。

### 代码示例

```bash
# 不存在的 key

redis> FLUSHDB
OK

redis> PTTL key
(integer) -2


# key 存在，但没有设置剩余生存时间

redis> SET key value
OK

redis> PTTL key
(integer) -1


# 有剩余生存时间的 key

redis> PEXPIRE key 10086
(integer) 1

redis> PTTL key
(integer) 6179
```

# 事务

## MULTI

**MULTI**

标记一个事务块的开始。

事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 [EXEC](#exec) 命令原子性(atomic)地执行。

- **可用版本：**
  
  > = 1.2.0

- **时间复杂度：**
  
  O(1)。

- **返回值：**
  
  总是返回 `OK` 。

```bash
redis> MULTI            # 标记事务开始
OK

redis> INCR user_id     # 多条命令按顺序入队
QUEUED

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> PING
QUEUED

redis> EXEC             # 执行
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
```

## EXEC

**EXEC**

执行所有事务块内的命令。

假如某个(或某些) key 正处于 [WATCH](#watch) 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 [EXEC](#exec) 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。

- **可用版本：**
  
  > = 1.2.0

- **时间复杂度：**
  
  事务块内所有命令的时间复杂度的总和。

- **返回值：**
  
  事务块内所有命令的返回值，按命令执行的先后顺序排列。当操作被打断时，返回空值 `nil` 。

```bash
# 事务被成功执行

redis> MULTI
OK

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> INCR user_id
QUEUED

redis> PING
QUEUED

redis> EXEC
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG


# 监视 key ，且事务成功执行

redis> WATCH lock lock_times
OK

redis> MULTI
OK

redis> SET lock "huangz"
QUEUED

redis> INCR lock_times
QUEUED

redis> EXEC
1) OK
2) (integer) 1


# 监视 key ，且事务被打断

redis> WATCH lock lock_times
OK

redis> MULTI
OK

redis> SET lock "joe"        # 就在这时，另一个客户端修改了 lock_times 的值
QUEUED

redis> INCR lock_times
QUEUED

redis> EXEC                  # 因为 lock_times 被修改， joe 的事务执行失败
(nil)
```

## DISCARD

> 可用版本： >= 2.0.0
> 
> 时间复杂度： O(1)。

取消事务，放弃执行事务块内的所有命令。

如果正在使用 `WATCH` 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 `UNWATCH` 。

### 返回值

总是返回 `OK` 。

### 代码示例

```bash
redis> MULTI
OK

redis> PING
QUEUED

redis> SET greeting "hello"
QUEUED

redis> DISCARD
OK
```

## WATCH

**WATCH key [key …]**

监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

- **可用版本：**
  
  > = 2.2.0

- **时间复杂度：**
  
  O(1)。

- **返回值：**
  
  总是返回 `OK` 。

```bash
redis> WATCH lock lock_times
OK
```

## UNWATCH

**UNWATCH**

取消 [WATCH](#watch) 命令对所有 key 的监视。

如果在执行 [WATCH](#watch) 命令之后， [EXEC](#exec) 命令或 [DISCARD](#discard) 命令先被执行了的话，那么就不需要再执行 [UNWATCH](#unwatch) 了。

因为 [EXEC](#exec) 命令会执行事务，因此 [WATCH](#watch) 命令的效果已经产生了；而 [DISCARD](#discard) 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 [UNWATCH](#unwatch) 了。

- **可用版本：**
  
  > = 2.2.0

- **时间复杂度：**
  
  O(1)

- **返回值：**
  
  总是 `OK` 。

```bash
redis> WATCH lock lock_times
OK

redis> UNWATCH
OK
```

# Lua 脚本

## EVAL

**EVAL script numkeys key [key …] arg [arg …]**

> 可用版本： >= 2.6.0
> 
> 时间复杂度： [EVAL](#eval) 和 EVALSHA 可以在 O(1) 复杂度内找到要被执行的脚本，其余的复杂度取决于执行的脚本本身。

从 Redis 2.6.0 版本开始，通过内置的 Lua 解释器，可以使用 [EVAL](#eval) 命令对 Lua 脚本进行求值。

`script` 参数是一段 Lua 5.1 脚本程序，它会被运行在 Redis 服务器上下文中，这段脚本不必(也不应该)定义为一个 Lua 函数。

`numkeys` 参数用于指定键名参数的个数。

键名参数 `key [key ...]` 从 [EVAL](#eval) 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，这些键名参数可以在 Lua 中通过全局变量 `KEYS` 数组，用 `1` 为基址的形式访问( `KEYS[1]` ， `KEYS[2]` ，以此类推)。

在命令的最后，那些不是键名参数的附加参数 `arg [arg ...]` ，可以在 Lua 中通过全局变量 `ARGV` 数组访问，访问的形式和 `KEYS` 变量类似( `ARGV[1]` 、 `ARGV[2]` ，诸如此类)。

上面这几段长长的说明可以用一个简单的例子来概括：

```
> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

其中 `"return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}"` 是被求值的 Lua 脚本，数字 `2` 指定了键名参数的数量， `key1` 和 `key2` 是键名参数，分别使用 `KEYS[1]` 和 `KEYS[2]` 访问，而最后的 `first` 和 `second` 则是附加参数，可以通过 `ARGV[1]` 和 `ARGV[2]` 访问它们。

在 Lua 脚本中，可以使用两个不同函数来执行 Redis 命令，它们分别是：

- `redis.call()`
- `redis.pcall()`

这两个函数的唯一区别在于它们使用不同的方式处理执行命令所产生的错误，在后面的『错误处理』部分会讲到这一点。

`redis.call()` 和 `redis.pcall()` 两个函数的参数可以是任何格式良好(well formed)的 Redis 命令：

```
> eval "return redis.call('set','foo','bar')" 0
OK
```

需要注意的是，上面这段脚本的确实现了将键 `foo` 的值设为 `bar` 的目的，但是，它违反了 [EVAL](#eval) 命令的语义，因为脚本里使用的所有键都应该由 `KEYS` 数组来传递，就像这样：

```
> eval "return redis.call('set',KEYS[1],'bar')" 1 foo
OK
```

要求使用正确的形式来传递键(key)是有原因的，因为不仅仅是 [EVAL](#eval) 这个命令，所有的 Redis 命令，在执行之前都会被分析，籍此来确定命令会对哪些键进行操作。

因此，对于 [EVAL](#eval) 命令来说，必须使用正确的形式来传递键，才能确保分析工作正确地执行。除此之外，使用正确的形式来传递键还有很多其他好处，它的一个特别重要的用途就是确保 Redis 集群可以将你的请求发送到正确的集群节点。(对 Redis 集群的工作还在进行当中，但是脚本功能被设计成可以与集群功能保持兼容。)不过，这条规矩并不是强制性的，从而使得用户有机会滥用(abuse) Redis 单实例配置(single instance configuration)，代价是这样写出的脚本不能被 Redis 集群所兼容。

### 在 Lua 数据类型和 Redis 数据类型之间转换

当 Lua 通过 `call()` 或 `pcall()` 函数执行 Redis 命令的时候，命令的返回值会被转换成 Lua 数据结构。同样地，当 Lua 脚本在 Redis 内置的解释器里运行时，Lua 脚本的返回值也会被转换成 Redis 协议(protocol)，然后由 [EVAL](#eval) 将值返回给客户端。

数据类型之间的转换遵循这样一个设计原则：如果将一个 Redis 值转换成 Lua 值，之后再将转换所得的 Lua 值转换回 Redis 值，那么这个转换所得的 Redis 值应该和最初时的 Redis 值一样。

换句话说， Lua 类型和 Redis 类型之间存在着一一对应的转换关系。

以下列出的是详细的转换规则：

从 Redis 转换到 Lua ：

- Redis integer reply -> Lua number / Redis 整数转换成 Lua 数字
- Redis bulk reply -> Lua string / Redis bulk 回复转换成 Lua 字符串
- Redis multi bulk reply -> Lua table (may have other Redis data types nested) / Redis 多条 bulk 回复转换成 Lua 表，表内可能有其他别的 Redis 数据类型
- Redis status reply -> Lua table with a single ok field containing the status / Redis 状态回复转换成 Lua 表，表内的 `ok` 域包含了状态信息
- Redis error reply -> Lua table with a single err field containing the error / Redis 错误回复转换成 Lua 表，表内的 `err` 域包含了错误信息
- Redis Nil bulk reply and Nil multi bulk reply -> Lua false boolean type / Redis 的 Nil 回复和 Nil 多条回复转换成 Lua 的布尔值 `false`

从 Lua 转换到 Redis：

- Lua number -> Redis integer reply / Lua 数字转换成 Redis 整数
- Lua string -> Redis bulk reply / Lua 字符串转换成 Redis bulk 回复
- Lua table (array) -> Redis multi bulk reply / Lua 表(数组)转换成 Redis 多条 bulk 回复
- Lua table with a single ok field -> Redis status reply / 一个带单个 `ok` 域的 Lua 表，转换成 Redis 状态回复
- Lua table with a single err field -> Redis error reply / 一个带单个 `err` 域的 Lua 表，转换成 Redis 错误回复
- Lua boolean false -> Redis Nil bulk reply / Lua 的布尔值 `false` 转换成 Redis 的 Nil bulk 回复

从 Lua 转换到 Redis 有一条额外的规则，这条规则没有和它对应的从 Redis 转换到 Lua 的规则：

- Lua boolean true -> Redis integer reply with value of 1 / Lua 布尔值 `true` 转换成 Redis 整数回复中的 `1`

以下是几个类型转换的例子：

```bash
> eval "return 10" 0
(integer) 10

> eval "return {1,2,{3,'Hello World!'}}" 0
1) (integer) 1
2) (integer) 2
3) 1) (integer) 3
   2) "Hello World!"

> eval "return redis.call('get','foo')" 0
"bar"
```

在上面的三个代码示例里，前两个演示了如何将 Lua 值转换成 Redis 值，最后一个例子更复杂一些，它演示了一个将 Redis 值转换成 Lua 值，然后再将 Lua 值转换成 Redis 值的类型转过程。

### 脚本的原子性

Redis 使用单个 Lua 解释器去运行所有脚本，并且， Redis 也保证脚本会以原子性(atomic)的方式执行：当某个脚本正在运行的时候，不会有其他脚本或 Redis 命令被执行。这和使用 [MULTI](#multi) / [EXEC](#exec) 包围的事务很类似。在其他别的客户端看来，脚本的效果(effect)要么是不可见的(not visible)，要么就是已完成的(already completed)。

另一方面，这也意味着，执行一个运行缓慢的脚本并不是一个好主意。写一个跑得很快很顺溜的脚本并不难，因为脚本的运行开销(overhead)非常少，但是当你不得不使用一些跑得比较慢的脚本时，请小心，因为当这些蜗牛脚本在慢吞吞地运行的时候，其他客户端会因为服务器正忙而无法执行命令。

### 错误处理

前面的命令介绍部分说过， `redis.call()` 和 `redis.pcall()` 的唯一区别在于它们对错误处理的不同。

当 `redis.call()` 在执行命令的过程中发生错误时，脚本会停止执行，并返回一个脚本错误，错误的输出信息会说明错误造成的原因：

```bash
redis> lpush foo a
(integer) 1

redis> eval "return redis.call('get', 'foo')" 0
(error) ERR Error running script (call to f_282297a0228f48cd3fc6a55de6316f31422f5d17): ERR Operation against a key holding the wrong kind of value
```

和 `redis.call()` 不同， `redis.pcall()` 出错时并不引发(raise)错误，而是返回一个带 `err` 域的 Lua 表(table)，用于表示错误：

```bash
redis 127.0.0.1:6379> EVAL "return redis.pcall('get', 'foo')" 0
(error) ERR Operation against a key holding the wrong kind of value
```

### 带宽和 EVALSHA

[EVAL](#eval) 命令要求你在每次执行脚本的时候都发送一次脚本主体(script body)。Redis 有一个内部的缓存机制，因此它不会每次都重新编译脚本，不过在很多场合，付出无谓的带宽来传送脚本主体并不是最佳选择。

为了减少带宽的消耗， Redis 实现了 EVALSHA 命令，它的作用和 [EVAL](#eval) 一样，都用于对脚本求值，但它接受的第一个参数不是脚本，而是脚本的 SHA1 校验和(sum)。

EVALSHA 命令的表现如下：

- 如果服务器还记得给定的 SHA1 校验和所指定的脚本，那么执行这个脚本
- 如果服务器不记得给定的 SHA1 校验和所指定的脚本，那么它返回一个特殊的错误，提醒用户使用 [EVAL](#eval) 代替 EVALSHA

以下是示例：

```bash
> set foo bar
OK

> eval "return redis.call('get','foo')" 0
"bar"

> evalsha 6b1bf486c81ceb7edf3c093f4c48582e38c0e791 0
"bar"

> evalsha ffffffffffffffffffffffffffffffffffffffff 0
(error) `NOSCRIPT` No matching script. Please use [EVAL](/commands/eval).
```

客户端库的底层实现可以一直乐观地使用 EVALSHA 来代替 [EVAL](#eval) ，并期望着要使用的脚本已经保存在服务器上了，只有当 `NOSCRIPT` 错误发生时，才使用 [EVAL](#eval) 命令重新发送脚本，这样就可以最大限度地节省带宽。

这也说明了执行 [EVAL](#eval) 命令时，使用正确的格式来传递键名参数和附加参数的重要性：因为如果将参数硬写在脚本中，那么每次当参数改变的时候，都要重新发送脚本，即使脚本的主体并没有改变，相反，通过使用正确的格式来传递键名参数和附加参数，就可以在脚本主体不变的情况下，直接使用 EVALSHA 命令对脚本进行复用，免去了无谓的带宽消耗。

### 脚本缓存

Redis 保证所有被运行过的脚本都会被永久保存在脚本缓存当中，这意味着，当 [EVAL](#eval) 命令在一个 Redis 实例上成功执行某个脚本之后，随后针对这个脚本的所有 EVALSHA 命令都会成功执行。

刷新脚本缓存的唯一办法是显式地调用 `SCRIPT FLUSH` 命令，这个命令会清空运行过的所有脚本的缓存。通常只有在云计算环境中，Redis 实例被改作其他客户或者别的应用程序的实例时，才会执行这个命令。

缓存可以长时间储存而不产生内存问题的原因是，它们的体积非常小，而且数量也非常少，即使脚本在概念上类似于实现一个新命令，即使在一个大规模的程序里有成百上千的脚本，即使这些脚本会经常修改，即便如此，储存这些脚本的内存仍然是微不足道的。

事实上，用户会发现 Redis 不移除缓存中的脚本实际上是一个好主意。比如说，对于一个和 Redis 保持持久化链接(persistent connection)的程序来说，它可以确信，执行过一次的脚本会一直保留在内存当中，因此它可以在流水线中使用 EVALSHA 命令而不必担心因为找不到所需的脚本而产生错误(稍候我们会看到在流水线中执行脚本的相关问题)。

### SCRIPT 命令

Redis 提供了以下几个 SCRIPT 命令，用于对脚本子系统(scripting subsystem)进行控制：

- SCRIPT FLUSH ：清除所有脚本缓存
- [SCRIPT EXISTS sha1 sha1 …\] ：根据给定的脚本校验和，检查指定的脚本是否存在于脚本缓存
- SCRIPT LOAD script ：将一个脚本装入脚本缓存，但并不立即运行它
- SCRIPT KILL ：杀死当前正在运行的脚本

### 纯函数脚本

在编写脚本方面，一个重要的要求就是，脚本应该被写成纯函数(pure function)。

也就是说，脚本应该具有以下属性：

- 对于同样的数据集输入，给定相同的参数，脚本执行的 Redis 写命令总是相同的。脚本执行的操作不能依赖于任何隐藏(非显式)数据，不能依赖于脚本在执行过程中、或脚本在不同执行时期之间可能变更的状态，并且它也不能依赖于任何来自 I/O 设备的外部输入。

使用系统时间(system time)，调用像 [RANDOMKEY](#randomkey) 那样的随机命令，或者使用 Lua 的随机数生成器，类似以上的这些操作，都会造成脚本的求值无法每次都得出同样的结果。

为了确保脚本符合上面所说的属性， Redis 做了以下工作：

- Lua 没有访问系统时间或者其他内部状态的命令
- Redis 会返回一个错误，阻止这样的脚本运行： 这些脚本在执行随机命令之后(比如 RANDOMKEY 、 [SRANDMEMBER key count\] 或 TIME 等)，还会执行可以修改数据集的 Redis 命令。如果脚本只是执行只读操作，那么就没有这一限制。注意，随机命令并不一定就指那些带 RAND 字眼的命令，任何带有非确定性的命令都会被认为是随机命令，比如 TIME 命令就是这方面的一个很好的例子。
- 每当从 Lua 脚本中调用那些返回无序元素的命令时，执行命令所得的数据在返回给 Lua 之前会先执行一个静默(slient)的字典序排序([lexicographical sorting](http://en.wikipedia.org/wiki/Lexicographical_order))。举个例子，因为 Redis 的 Set 保存的是无序的元素，所以在 Redis 命令行客户端中直接执行 [SMEMBERS key](#smembers) ，返回的元素是无序的，但是，假如在脚本中执行 `redis.call("smembers", KEYS[1])` ，那么返回的总是排过序的元素。
- 对 Lua 的伪随机数生成函数 `math.random` 和 `math.randomseed` 进行修改，使得每次在运行新脚本的时候，总是拥有同样的 seed 值。这意味着，每次运行脚本时，只要不使用 `math.randomseed` ，那么 `math.random` 产生的随机数序列总是相同的。

尽管有那么多的限制，但用户还是可以用一个简单的技巧写出带随机行为的脚本(如果他们需要的话)。

假设现在我们要编写一个 Redis 脚本，这个脚本从列表中弹出 N 个随机数。一个 Ruby 写的例子如下：

```ruby
require 'rubygems'
require 'redis'

r = Redis.new

RandomPushScript = <<EOF
    local i = tonumber(ARGV[1])
    local res
    while (i > 0) do
        res = redis.call('lpush',KEYS[1],math.random())
        i = i-1
    end
    return res
EOF

r.del(:mylist)
puts r.eval(RandomPushScript,[:mylist],[10,rand(2**32)])
```

这个程序每次运行都会生成带有以下元素的列表：

```bash
> lrange mylist 0 -1
1) "0.74509509873814"
2) "0.87390407681181"
3) "0.36876626981831"
4) "0.6921941534114"
5) "0.7857992587545"
6) "0.57730350670279"
7) "0.87046522734243"
8) "0.09637165539729"
9) "0.74990198051087"
10) "0.17082803611217"
```

上面的 Ruby 程序每次都只生成同样的列表，用途并不是太大。那么，该怎样修改这个脚本，使得它仍然是一个纯函数(符合 Redis 的要求)，但是每次调用都可以产生不同的随机元素呢？

一个简单的办法是，为脚本添加一个额外的参数，让这个参数作为 Lua 的随机数生成器的 seed 值，这样的话，只要给脚本传入不同的 seed ，脚本就会生成不同的列表元素。

以下是修改后的脚本：

```ruby
RandomPushScript = <<EOF
    local i = tonumber(ARGV[1])
    local res
    math.randomseed(tonumber(ARGV[2]))
    while (i > 0) do
        res = redis.call('lpush',KEYS[1],math.random())
        i = i-1
    end
    return res
EOF

r.del(:mylist)
puts r.eval(RandomPushScript,1,:mylist,10,rand(2**32))
```

尽管对于同样的 seed ，上面的脚本产生的列表元素是一样的(因为它是一个纯函数)，但是只要每次在执行脚本的时候传入不同的 seed ，我们就可以得到带有不同随机元素的列表。

Seed 会在复制(replication link)和写 AOF 文件时作为一个参数来传播，保证在载入 AOF 文件或附属节点(slave)处理脚本时， seed 仍然可以及时得到更新。

注意，Redis 实现保证 `math.random` 和 `math.randomseed` 的输出和运行 Redis 的系统架构无关，无论是 32 位还是 64 位系统，无论是小端(little endian)还是大端(big endian)系统，这两个函数的输出总是相同的。

### 全局变量保护

为了防止不必要的数据泄漏进 Lua 环境， Redis 脚本不允许创建全局变量。如果一个脚本需要在多次执行之间维持某种状态，它应该使用 Redis key 来进行状态保存。

企图在脚本中访问一个全局变量(不论这个变量是否存在)将引起脚本停止， [EVAL](#eval) 命令会返回一个错误：

```
redis 127.0.0.1:6379> eval 'a=10' 0
(error) ERR Error running script (call to f_933044db579a2f8fd45d8065f04a8d0249383e57): user_script:1: Script attempted to create global variable 'a'
```

Lua 的 debug 工具，或者其他设施，比如打印（alter）用于实现全局保护的 meta table ，都可以用于实现全局变量保护。

实现全局变量保护并不难，不过有时候还是会不小心而为之。一旦用户在脚本中混入了 Lua 全局状态，那么 AOF 持久化和复制（replication）都会无法保证，所以，请不要使用全局变量。

避免引入全局变量的一个诀窍是：将脚本中用到的所有变量都使用 `local` 关键字定义为局部变量。

### 库

Redis 内置的 Lua 解释器加载了以下 Lua 库：

- `base`
- `table`
- `string`
- `math`
- `debug`
- `cjson`
- `cmsgpack`

其中 `cjson` 库可以让 Lua 以非常快的速度处理 JSON 数据，除此之外，其他别的都是 Lua 的标准库。

每个 Redis 实例都保证会加载上面列举的库，从而确保每个 Redis 脚本的运行环境都是相同的。

### 使用脚本散发 Redis 日志

在 Lua 脚本中，可以通过调用 `redis.log` 函数来写 Redis 日志(log)：

```
redis.log(loglevel, message)
```

其中， `message` 参数是一个字符串，而 `loglevel` 参数可以是以下任意一个值：

- `redis.LOG_DEBUG`
- `redis.LOG_VERBOSE`
- `redis.LOG_NOTICE`
- `redis.LOG_WARNING`

上面的这些等级(level)和标准 Redis 日志的等级相对应。

对于脚本散发(emit)的日志，只有那些和当前 Redis 实例所设置的日志等级相同或更高级的日志才会被散发。

以下是一个日志示例：

```
redis.log(redis.LOG_WARNING, "Something is wrong with this script.")
```

执行上面的函数会产生这样的信息：

```
[32343] 22 Mar 15:21:39 # Something is wrong with this script.
```

### 沙箱(sandbox)和最大执行时间

脚本应该仅仅用于传递参数和对 Redis 数据进行处理，它不应该尝试去访问外部系统(比如文件系统)，或者执行任何系统调用。

除此之外，脚本还有一个最大执行时间限制，它的默认值是 5 秒钟，一般正常运作的脚本通常可以在几分之几毫秒之内完成，花不了那么多时间，这个限制主要是为了防止因编程错误而造成的无限循环而设置的。

最大执行时间的长短由 `lua-time-limit` 选项来控制(以毫秒为单位)，可以通过编辑 `redis.conf` 文件或者使用 CONFIG GET parameter 和 CONFIG SET parameter value 命令来修改它。

当一个脚本达到最大执行时间的时候，它并不会自动被 Redis 结束，因为 Redis 必须保证脚本执行的原子性，而中途停止脚本的运行意味着可能会留下未处理完的数据在数据集(data set)里面。

因此，当脚本运行的时间超过最大执行时间后，以下动作会被执行：

- Redis 记录一个脚本正在超时运行
- Redis 开始重新接受其他客户端的命令请求，但是只有 `SCRIPT KILL` 和 `SHUTDOWN NOSAVE` 两个命令会被处理，对于其他命令请求， Redis 服务器只是简单地返回 `BUSY` 错误。
- 可以使用 `SCRIPT KILL` 命令将一个仅执行只读命令的脚本杀死，因为只读命令并不修改数据，因此杀死这个脚本并不破坏数据的完整性
- 如果脚本已经执行过写命令，那么唯一允许执行的操作就是 `SHUTDOWN NOSAVE` ，它通过停止服务器来阻止当前数据集写入磁盘

### 流水线(pipeline)上下文(context)中的 EVALSHA

在流水线请求的上下文中使用 EVALSHA 命令时，要特别小心，因为在流水线中，必须保证命令的执行顺序。

一旦在流水线中因为 EVALSHA 命令而发生 NOSCRIPT 错误，那么这个流水线就再也没有办法重新执行了，否则的话，命令的执行顺序就会被打乱。

为了防止出现以上所说的问题，客户端库实现应该实施以下的其中一项措施：

- 总是在流水线中使用 [EVAL](#eval) 命令
- 检查流水线中要用到的所有命令，找到其中的 [EVAL](#eval) 命令，并使用 [SCRIPT EXISTS sha1 sha1 …\] 命令检查要用到的脚本是不是全都已经保存在缓存里面了。如果所需的全部脚本都可以在缓存里找到，那么就可以放心地将所有 [EVAL](#eval) 命令改成 EVALSHA 命令，否则的话，就要在流水线的顶端(top)将缺少的脚本用 SCRIPT LOAD script 命令加上去。

## EVALSHA

**EVALSHA sha1 numkeys key [key …] arg [arg …]**

> 可用版本： >= 2.6.0
> 
> 时间复杂度： 根据脚本的复杂度而定。

根据给定的 sha1 校验码，对缓存在服务器中的脚本进行求值。

将脚本缓存到服务器的操作可以通过 SCRIPT LOAD script 命令进行。

这个命令的其他地方，比如参数的传入方式，都和 EVAL script numkeys key [key …\] arg [arg …] 命令一样。

```bash
redis> SCRIPT LOAD "return 'hello moto'"
"232fd51614574cf0867b83d384a5e898cfd24e5a"

redis> EVALSHA "232fd51614574cf0867b83d384a5e898cfd24e5a" 0
"hello moto"
```

## SCRIPT LOAD script

> 可用版本： >= 2.6.0
> 
> 时间复杂度： O(N) , `N` 为脚本的长度(以字节为单位)。

将脚本 `script` 添加到脚本缓存中，但并不立即执行这个脚本。

EVAL script numkeys key [key …\] arg [arg …] 命令也会将脚本添加到脚本缓存中，但是它会立即对输入的脚本进行求值。

如果给定的脚本已经在缓存里面了，那么不做动作。

在脚本被加入到缓存之后，通过 EVALSHA 命令，可以使用脚本的 SHA1 校验和来调用这个脚本。

脚本可以在缓存中保留无限长的时间，直到执行 SCRIPT FLUSH 为止。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 EVAL script numkeys key [key …\] arg [arg …] 命令。

### 返回值

给定 `script` 的 SHA1 校验和。

### 代码示例

```bash
redis> SCRIPT LOAD "return 'hello moto'"
"232fd51614574cf0867b83d384a5e898cfd24e5a"

redis> EVALSHA 232fd51614574cf0867b83d384a5e898cfd24e5a 0
"hello moto"
```

## SCRIPT EXISTS

**SCRIPT EXISTS sha1 [sha1 …]**

> 可用版本： >= 2.6.0
> 
> 时间复杂度： O(N) , `N` 为给定的 SHA1 校验和的数量。

给定一个或多个脚本的 SHA1 校验和，返回一个包含 `0` 和 `1` 的列表，表示校验和所指定的脚本是否已经被保存在缓存当中。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 EVAL script numkeys key [key …\] arg [arg …] 命令。

### 返回值

一个列表，包含 `0` 和 `1` ，前者表示脚本不存在于缓存，后者表示脚本已经在缓存里面了。 列表中的元素和给定的 SHA1 校验和保持对应关系，比如列表的第三个元素的值就表示第三个 SHA1 校验和所指定的脚本在缓存中的状态。

### 代码示例

```bash
redis> SCRIPT LOAD "return 'hello moto'"    # 载入一个脚本
"232fd51614574cf0867b83d384a5e898cfd24e5a"

redis> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
1) (integer) 1

redis> SCRIPT FLUSH     # 清空缓存
OK

redis> SCRIPT EXISTS 232fd51614574cf0867b83d384a5e898cfd24e5a
1) (integer) 0
```

## SCRIPT FLUSH

> 可用版本： >= 2.6.0
> 
> 复杂度： O(N) ， `N` 为缓存中脚本的数量。

清除所有 Lua 脚本缓存。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 EVAL script numkeys key [key …\] arg [arg …] 命令。

### 返回值

总是返回 `OK`

### 代码示例

```bash
redis> SCRIPT FLUSH
OK
```

## SCRIPT KILL

> 可用版本： >= 2.6.0
> 
> 时间复杂度： O(1)

杀死当前正在运行的 Lua 脚本，当且仅当这个脚本没有执行过任何写操作时，这个命令才生效。

这个命令主要用于终止运行时间过长的脚本，比如一个因为 BUG 而发生无限 loop 的脚本，诸如此类。

SCRIPT KILL 执行之后，当前正在运行的脚本会被杀死，执行这个脚本的客户端会从 EVAL script numkeys key [key …\] arg [arg …] 命令的阻塞当中退出，并收到一个错误作为返回值。

另一方面，假如当前正在运行的脚本已经执行过写操作，那么即使执行 SCRIPT KILL ，也无法将它杀死，因为这是违反 Lua 脚本的原子性执行原则的。在这种情况下，唯一可行的办法是使用 `SHUTDOWN NOSAVE` 命令，通过停止整个 Redis 进程来停止脚本的运行，并防止不完整(half-written)的信息被写入数据库中。

关于使用 Redis 对 Lua 脚本进行求值的更多信息，请参见 EVAL script numkeys key [key …\] arg [arg …]命令。

### 返回值

执行成功返回 `OK` ，否则返回一个错误。

### 代码示例

```bash
# 没有脚本在执行时

redis> SCRIPT KILL
(error) ERR No scripts in execution right now.

# 成功杀死脚本时

redis> SCRIPT KILL
OK
(1.30s)

# 尝试杀死一个已经执行过写操作的脚本，失败

redis> SCRIPT KILL
(error) ERR Sorry the script already executed write commands against the dataset. You can either wait the script termination or kill the server in an hard way using the SHUTDOWN NOSAVE command.
(1.69s)
```

以下是脚本被杀死之后，返回给执行脚本的客户端的错误：

```bash
redis> EVAL "while true do end" 0
(error) ERR Error running script (call to f_694a5fe1ddb97a4c6a1bf299d9537c7d3d0f84e7): Script killed by user with SCRIPT KILL...
(5.00s)
```

# 持久化

## SAVE

> 可用版本： >= 1.0.0
> 
> 时间复杂度： O(N)， `N` 为要保存到数据库中的 key 的数量。

[SAVE](#save) 命令执行一个同步保存操作，将当前 Redis 实例的所有数据快照(snapshot)以 RDB 文件的形式保存到硬盘。

一般来说，在生产环境很少执行 [SAVE](#save) 操作，因为它会阻塞所有客户端，保存数据库的任务通常由 [BGSAVE](#bgsave) 命令异步地执行。然而，如果负责保存数据的后台子进程不幸出现问题时， [SAVE](#save) 可以作为保存数据的最后手段来使用。

请参考文档： [Redis 的持久化运作方式(英文)](http://redis.io/topics/persistence) 以获取更多消息。

### 返回值

保存成功时返回 `OK` 。

### 代码示例

```bash
redis> SAVE
OK
```

## BGSAVE

在后台异步(Asynchronously)保存当前数据库的数据到磁盘。

[BGSAVE](#bgsave) 命令执行之后立即返回 `OK` ，然后 Redis fork 出一个新子进程，原来的 Redis 进程(父进程)继续处理客户端请求，而子进程则负责将数据保存到磁盘，然后退出。

客户端可以通过 [LASTSAVE](l#lastsave) 命令查看相关信息，判断 [BGSAVE](#bgsave) 命令是否执行成功。

请移步 [持久化文档](http://redis.io/topics/persistence) 查看更多相关细节。

### 返回值

反馈信息。

### 代码示例

```bash
redis> BGSAVE
Background saving started
```

## BGREWRITEAOF

执行一个 [AOF文件](http://redis.io/topics/persistence#append-only-file) 重写操作。重写会创建一个当前 AOF 文件的体积优化版本。

即使 [BGREWRITEAOF](#bgrewriteaof) 执行失败，也不会有任何数据丢失，因为旧的 AOF 文件在 [BGREWRITEAOF](#bgrewriteaof) 成功之前不会被修改。

重写操作只会在没有其他持久化工作在后台执行时被触发，也就是说：

- 如果 Redis 的子进程正在执行快照的保存工作，那么 AOF 重写的操作会被预定(scheduled)，等到保存工作完成之后再执行 AOF 重写。在这种情况下， [BGREWRITEAOF](#bgrewriteaof) 的返回值仍然是 `OK` ，但还会加上一条额外的信息，说明 [BGREWRITEAOF](#bgrewriteaof) 要等到保存操作完成之后才能执行。在 Redis 2.6 或以上的版本，可以使用 [INFO [section\]](#info) 命令查看 [BGREWRITEAOF](#bgrewriteaof) 是否被预定。
- 如果已经有别的 AOF 文件重写在执行，那么 [BGREWRITEAOF](#bgrewriteaof) 返回一个错误，并且这个新的 [BGREWRITEAOF](#bgrewriteaof) 请求也不会被预定到下次执行。

从 Redis 2.4 开始， AOF 重写由 Redis 自行触发， [BGREWRITEAOF](#bgrewriteaof) 仅仅用于手动触发重写操作。

请移步 [持久化文档(英文)](http://redis.io/topics/persistence) 查看更多相关细节。

### 返回值

反馈信息。

### 代码示例

```bash
redis> BGREWRITEAOF
Background append only file rewriting started
```

## LASTSAVE

返回最近一次 Redis 成功将数据保存到磁盘上的时间，以 UNIX 时间戳格式表示。

### 返回值

一个 UNIX 时间戳。

### 代码示例

```bash
redis> LASTSAVE
(integer) 1324043588
```

# 发布与订阅

## PUBLISH

**PUBLISH channel message**

将信息 `message` 发送到指定的频道 `channel` 。

### 返回值

接收到信息 `message` 的订阅者数量。

### 代码示例

```bash
# 对没有订阅者的频道发送信息

redis> publish bad_channel "can any body hear me?"
(integer) 0


# 向有一个订阅者的频道发送信息

redis> publish msg "good morning"
(integer) 1


# 向有多个订阅者的频道发送信息

redis> publish chat_room "hello~ everyone"
(integer) 3
```

## SUBSCRIBE

**SUBSCRIBE channel [channel …]**

订阅给定的一个或多个频道的信息。

接收到的信息(请参见下面的代码说明)。

```bash
# 订阅 msg 和 chat_room 两个频道

# 1 - 6 行是执行 subscribe 之后的反馈信息
# 第 7 - 9 行才是接收到的第一条信息
# 第 10 - 12 行是第二条

redis> subscribe msg chat_room
Reading messages... (press Ctrl-C to quit)
1) "subscribe"       # 返回值的类型：显示订阅成功
2) "msg"             # 订阅的频道名字
3) (integer) 1       # 目前已订阅的频道数量

1) "subscribe"
2) "chat_room"
3) (integer) 2

1) "message"         # 返回值的类型：信息
2) "msg"             # 来源(从那个频道发送过来)
3) "hello moto"      # 信息内容

1) "message"
2) "chat_room"
3) "testing...haha"
```

## PSUBSCRIBE

**PSUBSCRIBE pattern [pattern …]**

订阅一个或多个符合给定模式的频道。

每个模式以 `*` 作为匹配符，比如 `it*` 匹配所有以 `it` 开头的频道( `it.news` 、 `it.blog` 、 `it.tweets` 等等)， `news.*` 匹配所有以 `news.` 开头的频道( `news.it` 、 `news.global.today` 等等)，诸如此类。

接收到的信息(请参见下面的代码说明)。

```bash
# 订阅 news.* 和 tweet.* 两个模式

# 第 1 - 6 行是执行 psubscribe 之后的反馈信息
# 第 7 - 10 才是接收到的第一条信息
# 第 11 - 14 是第二条
# 以此类推。。。

redis> psubscribe news.* tweet.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"                  # 返回值的类型：显示订阅成功
2) "news.*"                      # 订阅的模式
3) (integer) 1                   # 目前已订阅的模式的数量

1) "psubscribe"
2) "tweet.*"
3) (integer) 2

1) "pmessage"                    # 返回值的类型：信息
2) "news.*"                      # 信息匹配的模式
3) "news.it"                     # 信息本身的目标频道
4) "Google buy Motorola"         # 信息的内容

1) "pmessage"
2) "tweet.*"
3) "tweet.huangz"
4) "hello"

1) "pmessage"
2) "tweet.*"
3) "tweet.joe"
4) "@huangz morning"

1) "pmessage"
2) "news.*"
3) "news.life"
4) "An apple a day, keep doctors away"
```

## UNSUBSCRIBE

**UNSUBSCRIBE [channel [channel …]]**

指示客户端退订给定的频道。

如果没有频道被指定，也即是，一个无参数的 `UNSUBSCRIBE` 调用被执行，那么客户端使用 `SUBSCRIBE` 命令订阅的所有频道都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的频道。

这个命令在不同的客户端中有不同的表现。

## PUNSUBSCRIBE

**PUNSUBSCRIBE [pattern [pattern …]]**

指示客户端退订所有给定模式。

如果没有模式被指定，也即是，一个无参数的 `PUNSUBSCRIBE` 调用被执行，那么客户端使用 [PSUBSCRIBE pattern pattern …\] 命令订阅的所有模式都会被退订。在这种情况下，命令会返回一个信息，告知客户端所有被退订的模式。

### 返回值

这个命令在不同的客户端中有不同的表现。

## PUBSUB

**PUBSUB <subcommand> [argument [argument …]]**

`PUBSUB` 是一个查看订阅与发布系统状态的内省命令， 它由数个不同格式的子命令组成， 以下将分别对这些子命令进行介绍。

**PUBSUB CHANNELS [pattern]**

> 复杂度： O(N) ， `N` 为活跃频道的数量（对于长度较短的频道和模式来说，将进行模式匹配的复杂度视为常数）。

列出当前的活跃频道。

活跃频道指的是那些至少有一个订阅者的频道， 订阅模式的客户端不计算在内。

`pattern` 参数是可选的：

- 如果不给出 `pattern` 参数，那么列出订阅与发布系统中的所有活跃频道。
- 如果给出 `pattern` 参数，那么只列出和给定模式 `pattern` 相匹配的那些活跃频道。

一个由活跃频道组成的列表。

```bash
# client-1 订阅 news.it 和 news.sport 两个频道

client-1> SUBSCRIBE news.it news.sport
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news.it"
3) (integer) 1
1) "subscribe"
2) "news.sport"
3) (integer) 2

# client-2 订阅 news.it 和 news.internet 两个频道

client-2> SUBSCRIBE news.it news.internet
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news.it"
3) (integer) 1
1) "subscribe"
2) "news.internet"
3) (integer) 2

# 首先， client-3 打印所有活跃频道
# 注意，即使一个频道有多个订阅者，它也只输出一次，比如 news.it

client-3> PUBSUB CHANNELS
1) "news.sport"
2) "news.internet"
3) "news.it"

# 接下来， client-3 打印那些与模式 news.i* 相匹配的活跃频道
# 因为 news.sport 不匹配 news.i* ，所以它没有被打印

redis> PUBSUB CHANNELS news.i*
1) "news.internet"
2) "news.it"
```

PUBSUB NUMSUB [channel-1 … channel-N]

> 复杂度： O(N) ， `N` 为给定频道的数量。

返回给定频道的订阅者数量， 订阅模式的客户端不计算在内。

### 返回值

一个多条批量回复（Multi-bulk reply），回复中包含给定的频道，以及频道的订阅者数量。 格式为：频道 `channel-1` ， `channel-1` 的订阅者数量，频道 `channel-2` ， `channel-2` 的订阅者数量，诸如此类。 回复中频道的排列顺序和执行命令时给定频道的排列顺序一致。 不给定任何频道而直接调用这个命令也是可以的， 在这种情况下， 命令只返回一个空列表。

### 代码示例

```bash
# client-1 订阅 news.it 和 news.sport 两个频道

client-1> SUBSCRIBE news.it news.sport
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news.it"
3) (integer) 1
1) "subscribe"
2) "news.sport"
3) (integer) 2

# client-2 订阅 news.it 和 news.internet 两个频道

client-2> SUBSCRIBE news.it news.internet
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "news.it"
3) (integer) 1
1) "subscribe"
2) "news.internet"
3) (integer) 2

# client-3 打印各个频道的订阅者数量

client-3> PUBSUB NUMSUB news.it news.internet news.sport news.music
1) "news.it"    # 频道
2) "2"          # 订阅该频道的客户端数量
3) "news.internet"
4) "1"
5) "news.sport"
6) "1"
7) "news.music" # 没有任何订阅者
8) "0"
```

**PUBSUB NUMPAT**

> 复杂度： O(1) 。

返回订阅模式的数量。

注意， 这个命令返回的不是订阅模式的客户端的数量， 而是客户端订阅的所有模式的数量总和。

### 返回值

一个整数回复（Integer reply）。

### 代码示例

```bash
# client-1 订阅 news.* 和 discount.* 两个模式

client-1> PSUBSCRIBE news.* discount.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news.*"
3) (integer) 1
1) "psubscribe"
2) "discount.*"
3) (integer) 2

# client-2 订阅 tweet.* 一个模式

client-2> PSUBSCRIBE tweet.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "tweet.*"
3) (integer) 1

# client-3 返回当前订阅模式的数量为 3

client-3> PUBSUB NUMPAT
(integer) 3

# 注意，当有多个客户端订阅相同的模式时，相同的订阅也被计算在 PUBSUB NUMPAT 之内
# 比如说，再新建一个客户端 client-4 ，让它也订阅 news.* 频道

client-4> PSUBSCRIBE news.*
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "news.*"
3) (integer) 1

# 这时再计算被订阅模式的数量，就会得到数量为 4

client-3> PUBSUB NUMPAT
(integer) 4
```

# 复制

## SLAVEOF

**SLAVEOF host port**

[SLAVEOF](#slaveof) 命令用于在 Redis 运行时动态地修改复制(replication)功能的行为。

通过执行 `SLAVEOF host port` 命令，可以将当前服务器转变为指定服务器的从属服务器(slave server)。

如果当前服务器已经是某个主服务器(master server)的从属服务器，那么执行 `SLAVEOF host port` 将使当前服务器停止对旧主服务器的同步，丢弃旧数据集，转而开始对新主服务器进行同步。

另外，对一个从属服务器执行命令 `SLAVEOF NO ONE` 将使得这个从属服务器关闭复制功能，并从从属服务器转变回主服务器，原来同步所得的数据集*不会*被丢弃。

利用“`SLAVEOF NO ONE` 不会丢弃同步所得数据集”这个特性，可以在主服务器失败的时候，将从属服务器用作新的主服务器，从而实现无间断运行。

### 返回值

总是返回 `OK` 。

### 代码示例

```bash
redis> SLAVEOF 127.0.0.1 6379
OK

redis> SLAVEOF NO ONE
OK
```

## REPLICAOF

REPLICAOF 是 SLAVEOF的同义词。出于政治正确的考虑，自 Redis 5.0后，REPLICAOF 作为 SLAVEOF 的同义词使用，同时 SLAVEOF 仍然可用，保持向后兼容性。

## ROLE

**ROLE**

返回实例在复制中担任的角色， 这个角色可以是 `master` 、 `slave` 或者 `sentinel` 。 除了角色之外， 命令还会返回与该角色相关的其他信息， 其中：

- 主服务器将返回属下从服务器的 IP 地址和端口。
- 从服务器将返回自己正在复制的主服务器的 IP 地址、端口、连接状态以及复制偏移量。
- Sentinel 将返回自己正在监视的主服务器列表。

### 返回值

`ROLE` 命令将返回一个数组。

### 代码示例

#### 主服务器

```bash
1) "master"
2) (integer) 3129659
3) 1) 1) "127.0.0.1"
      2) "9001"
      3) "3129242"
   2) 1) "127.0.0.1"
      2) "9002"
      3) "3129543"
```

#### 从服务器

```bash
1) "slave"
2) "127.0.0.1"
3) (integer) 9000
4) "connected"
5) (integer) 3167038
```

#### Sentinel

```bash
1) "sentinel"
2) 1) "resque-master"
   2) "html-fragments-master"
   3) "stats-master"
   4) "metadata-master"
```

# 客户端与服务器

## AUTH

**AUTH password**

通过设置配置文件中 `requirepass` 项的值(使用命令 `CONFIG SET requirepass password` )，可以使用密码来保护 Redis 服务器。

如果开启了密码保护的话，在每次连接 Redis 服务器之后，就要使用 `AUTH` 命令解锁，解锁之后才能使用其他 Redis 命令。

如果 `AUTH` 命令给定的密码 `password` 和配置文件中的密码相符的话，服务器会返回 `OK` 并开始接受命令输入。

另一方面，假如密码不匹配的话，服务器将返回一个错误，并要求客户端需重新输入密码。

Warning

因为 Redis 高性能的特点，在很短时间内尝试猜测非常多个密码是有可能的，因此请确保使用的密码足够复杂和足够长，以免遭受密码猜测攻击。

### 返回值

密码匹配时返回 `OK` ，否则返回一个错误。

### 代码示例

```bash
# 设置密码

redis> CONFIG SET requirepass secret_password   # 将密码设置为 secret_password
OK

redis> QUIT                                     # 退出再连接，让新密码对客户端生效

[huangz@mypad]$ redis

redis> PING                                     # 未验证密码，操作被拒绝
(error) ERR operation not permitted

redis> AUTH wrong_password_testing              # 尝试输入错误的密码
(error) ERR invalid password

redis> AUTH secret_password                     # 输入正确的密码
OK

redis> PING                                     # 密码验证成功，可以正常操作命令了
PONG


# 清空密码

redis> CONFIG SET requirepass ""   # 通过将密码设为空字符来清空密码
OK

redis> QUIT

$ redis                            # 重新进入客户端

redis> PING                        # 执行命令不再需要密码，清空密码操作成功
PONG
```

## QUIT

请求服务器关闭与当前客户端的连接。

一旦所有等待中的回复(如果有的话)顺利写入到客户端，连接就会被关闭。

### 返回值

总是返回 `OK` (但是不会被打印显示，因为当时 Redis-cli 已经退出)。

### 代码示例

```bash
$ redis

redis> QUIT

$
```

## INFO

**INFO [section]**

以一种易于解释（parse）且易于阅读的格式，返回关于 Redis 服务器的各种信息和统计数值。

通过给定可选的参数 `section` ，可以让命令只返回某一部分的信息：

- `server` 部分记录了 Redis 服务器的信息，它包含以下域：
  
  > - `redis_version` : Redis 服务器版本
  > - `redis_git_sha1` : Git SHA1
  > - `redis_git_dirty` : Git dirty flag
  > - `os` : Redis 服务器的宿主操作系统
  > - `arch_bits` : 架构（32 或 64 位）
  > - `multiplexing_api` : Redis 所使用的事件处理机制
  > - `gcc_version` : 编译 Redis 时所使用的 GCC 版本
  > - `process_id` : 服务器进程的 PID
  > - `run_id` : Redis 服务器的随机标识符（用于 Sentinel 和集群）
  > - `tcp_port` : TCP/IP 监听端口
  > - `uptime_in_seconds` : 自 Redis 服务器启动以来，经过的秒数
  > - `uptime_in_days` : 自 Redis 服务器启动以来，经过的天数
  > - `lru_clock` : 以分钟为单位进行自增的时钟，用于 LRU 管理

- `clients` 部分记录了已连接客户端的信息，它包含以下域：
  
  > - `connected_clients` : 已连接客户端的数量（不包括通过从属服务器连接的客户端）
  > - `client_longest_output_list` : 当前连接的客户端当中，最长的输出列表
  > - `client_longest_input_buf` : 当前连接的客户端当中，最大输入缓存
  > - `blocked_clients` : 正在等待阻塞命令（BLPOP、BRPOP、BRPOPLPUSH）的客户端的数量

- `memory` 部分记录了服务器的内存信息，它包含以下域：
  
  > - `used_memory` : 由 Redis 分配器分配的内存总量，以字节（byte）为单位
  > - `used_memory_human` : 以人类可读的格式返回 Redis 分配的内存总量
  > - `used_memory_rss` : 从操作系统的角度，返回 Redis 已分配的内存总量（俗称常驻集大小）。这个值和 `top` 、 `ps` 等命令的输出一致。
  > - `used_memory_peak` : Redis 的内存消耗峰值（以字节为单位）
  > - `used_memory_peak_human` : 以人类可读的格式返回 Redis 的内存消耗峰值
  > - `used_memory_lua` : Lua 引擎所使用的内存大小（以字节为单位）
  > - `mem_fragmentation_ratio` : `used_memory_rss` 和 `used_memory` 之间的比率
  > - `mem_allocator` : 在编译时指定的， Redis 所使用的内存分配器。可以是 libc 、 jemalloc 或者 tcmalloc 。
  > 
  > 在理想情况下， `used_memory_rss` 的值应该只比 `used_memory` 稍微高一点儿。
  > 
  > 当 `rss > used` ，且两者的值相差较大时，表示存在（内部或外部的）内存碎片。
  > 
  > 内存碎片的比率可以通过 `mem_fragmentation_ratio` 的值看出。
  > 
  > 当 `used > rss` 时，表示 Redis 的部分内存被操作系统换出到交换空间了，在这种情况下，操作可能会产生明显的延迟。
  > 
  > Because Redis does not have control over how its allocations are mapped to memory pages, high `used_memory_rss` is often the result of a spike in memory usage.
  > 
  > 当 Redis 释放内存时，分配器可能会，也可能不会，将内存返还给操作系统。
  > 
  > 如果 Redis 释放了内存，却没有将内存返还给操作系统，那么 `used_memory` 的值可能和操作系统显示的 Redis 内存占用并不一致。
  > 
  > 查看 `used_memory_peak` 的值可以验证这种情况是否发生。

- `persistence` 部分记录了跟 `RDB` 持久化和 `AOF` 持久化有关的信息，它包含以下域：
  
  > - `loading` : 一个标志值，记录了服务器是否正在载入持久化文件。
  > - `rdb_changes_since_last_save` : 距离最近一次成功创建持久化文件之后，经过了多少秒。
  > - `rdb_bgsave_in_progress` : 一个标志值，记录了服务器是否正在创建 RDB 文件。
  > - `rdb_last_save_time` : 最近一次成功创建 RDB 文件的 UNIX 时间戳。
  > - `rdb_last_bgsave_status` : 一个标志值，记录了最近一次创建 RDB 文件的结果是成功还是失败。
  > - `rdb_last_bgsave_time_sec` : 记录了最近一次创建 RDB 文件耗费的秒数。
  > - `rdb_current_bgsave_time_sec` : 如果服务器正在创建 RDB 文件，那么这个域记录的就是当前的创建操作已经耗费的秒数。
  > - `aof_enabled` : 一个标志值，记录了 AOF 是否处于打开状态。
  > - `aof_rewrite_in_progress` : 一个标志值，记录了服务器是否正在创建 AOF 文件。
  > - `aof_rewrite_scheduled` : 一个标志值，记录了在 RDB 文件创建完毕之后，是否需要执行预约的 AOF 重写操作。
  > - `aof_last_rewrite_time_sec` : 最近一次创建 AOF 文件耗费的时长。
  > - `aof_current_rewrite_time_sec` : 如果服务器正在创建 AOF 文件，那么这个域记录的就是当前的创建操作已经耗费的秒数。
  > - `aof_last_bgrewrite_status` : 一个标志值，记录了最近一次创建 AOF 文件的结果是成功还是失败。
  
  如果 AOF 持久化功能处于开启状态，那么这个部分还会加上以下域：
  
  > - `aof_current_size` : AOF 文件目前的大小。
  > - `aof_base_size` : 服务器启动时或者 AOF 重写最近一次执行之后，AOF 文件的大小。
  > - `aof_pending_rewrite` : 一个标志值，记录了是否有 AOF 重写操作在等待 RDB 文件创建完毕之后执行。
  > - `aof_buffer_length` : AOF 缓冲区的大小。
  > - `aof_rewrite_buffer_length` : AOF 重写缓冲区的大小。
  > - `aof_pending_bio_fsync` : 后台 I/O 队列里面，等待执行的 `fsync` 调用数量。
  > - `aof_delayed_fsync` : 被延迟的 `fsync` 调用数量。

- `stats` 部分记录了一般统计信息，它包含以下域：
  
  > - `total_connections_received` : 服务器已接受的连接请求数量。
  > - `total_commands_processed` : 服务器已执行的命令数量。
  > - `instantaneous_ops_per_sec` : 服务器每秒钟执行的命令数量。
  > - `rejected_connections` : 因为最大客户端数量限制而被拒绝的连接请求数量。
  > - `expired_keys` : 因为过期而被自动删除的数据库键数量。
  > - `evicted_keys` : 因为最大内存容量限制而被驱逐（evict）的键数量。
  > - `keyspace_hits` : 查找数据库键成功的次数。
  > - `keyspace_misses` : 查找数据库键失败的次数。
  > - `pubsub_channels` : 目前被订阅的频道数量。
  > - `pubsub_patterns` : 目前被订阅的模式数量。
  > - `latest_fork_usec` : 最近一次 `fork()` 操作耗费的毫秒数。

- `replication` : 主/从复制信息
  
  > - `role` : 如果当前服务器没有在复制任何其他服务器，那么这个域的值就是 `master` ；否则的话，这个域的值就是 `slave` 。注意，在创建复制链的时候，一个从服务器也可能是另一个服务器的主服务器。
  
  如果当前服务器是一个从服务器的话，那么这个部分还会加上以下域：
  
  > - `master_host` : 主服务器的 IP 地址。
  > - `master_port` : 主服务器的 TCP 监听端口号。
  > - `master_link_status` : 复制连接当前的状态， `up` 表示连接正常， `down` 表示连接断开。
  > - `master_last_io_seconds_ago` : 距离最近一次与主服务器进行通信已经过去了多少秒钟。
  > - `master_sync_in_progress` : 一个标志值，记录了主服务器是否正在与这个从服务器进行同步。
  
  如果同步操作正在进行，那么这个部分还会加上以下域：
  
  > - `master_sync_left_bytes` : 距离同步完成还缺少多少字节数据。
  > - `master_sync_last_io_seconds_ago` : 距离最近一次因为 SYNC 操作而进行 I/O 已经过去了多少秒。
  
  如果主从服务器之间的连接处于断线状态，那么这个部分还会加上以下域：
  
  > - `master_link_down_since_seconds` : 主从服务器连接断开了多少秒。
  
  以下是一些总会出现的域：
  
  > - `connected_slaves` : 已连接的从服务器数量。
  
  对于每个从服务器，都会添加以下一行信息：
  
  > - `slaveXXX` : ID、IP 地址、端口号、连接状态

- `cpu` 部分记录了 CPU 的计算量统计信息，它包含以下域：
  
  > - `used_cpu_sys` : Redis 服务器耗费的系统 CPU 。
  > - `used_cpu_user` : Redis 服务器耗费的用户 CPU 。
  > - `used_cpu_sys_children` : 后台进程耗费的系统 CPU 。
  > - `used_cpu_user_children` : 后台进程耗费的用户 CPU 。

- `commandstats` 部分记录了各种不同类型的命令的执行统计信息，比如命令执行的次数、命令耗费的 CPU 时间、执行每个命令耗费的平均 CPU 时间等等。对于每种类型的命令，这个部分都会添加一行以下格式的信息：
  
  > - `cmdstat_XXX:calls=XXX,usec=XXX,usecpercall=XXX`

- `cluster` 部分记录了和集群有关的信息，它包含以下域：
  
  > - `cluster_enabled` : 一个标志值，记录集群功能是否已经开启。

- `keyspace` 部分记录了数据库相关的统计信息，比如数据库的键数量、数据库已经被删除的过期键数量等。对于每个数据库，这个部分都会添加一行以下格式的信息：
  
  > - `dbXXX:keys=XXX,expires=XXX`

除上面给出的这些值以外， `section` 参数的值还可以是下面这两个：

- `all` : 返回所有信息
- `default` : 返回默认选择的信息

当不带参数直接调用 [INFO](#info) 命令时，使用 `default` 作为默认参数。

Note

不同版本的 Redis 可能对返回的一些域进行了增加或删减。

因此，一个健壮的客户端程序在对 [INFO [section\]](#info) 命令的输出进行分析时，应该能够跳过不认识的域，并且妥善地处理丢失不见的域。

### 返回值

具体请参见下面的测试代码。

### 代码示例

```bash
redis> INFO
# Server
redis_version:2.9.11
redis_git_sha1:937384d0
redis_git_dirty:0
redis_build_id:8e9509442863f22
redis_mode:standalone
os:Linux 3.13.0-35-generic x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.8.2
process_id:4716
run_id:26186aac3f2380aaee9eef21cc50aecd542d97dc
tcp_port:6379
uptime_in_seconds:362
uptime_in_days:0
hz:10
lru_clock:1725349
config_file:

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:508536
used_memory_human:496.62K
used_memory_rss:7974912
used_memory_peak:508536
used_memory_peak_human:496.62K
used_memory_lua:33792
mem_fragmentation_ratio:15.68
mem_allocator:jemalloc-3.2.0

# Persistence
loading:0
rdb_changes_since_last_save:6
rdb_bgsave_in_progress:0
rdb_last_save_time:1411011131
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:-1
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:2
total_commands_processed:4
instantaneous_ops_per_sec:0
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:0
keyspace_misses:0
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:0
migrate_cached_sockets:0

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:0.21
used_cpu_user:0.17
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=2,expires=0,avg_ttl=0
```

## SHUTDOWN

**SHUTDOWN [SAVE|NOSAVE]**

[SHUTDOWN](#shutdown) 命令执行以下操作：

- 停止所有客户端
- 如果有至少一个保存点在等待，执行 SAVE 命令
- 如果 AOF 选项被打开，更新 AOF 文件
- 关闭 redis 服务器(server)

如果持久化被打开的话， [SHUTDOWN](#shutdown) 命令会保证服务器正常关闭而不丢失任何数据。

另一方面，假如只是单纯地执行 SAVE命令，然后再执行 QUIT命令，则没有这一保证 —— 因为在执行 SAVE 之后、执行 QUIT之前的这段时间中间，其他客户端可能正在和服务器进行通讯，这时如果执行 QUIT就会造成数据丢失。

### SAVE 和 NOSAVE 修饰符

通过使用可选的修饰符，可以修改 [SHUTDOWN](#shutdown) 命令的表现。比如说：

- 执行 `SHUTDOWN SAVE` 会强制让数据库执行保存操作，即使没有设定(configure)保存点
- 执行 `SHUTDOWN NOSAVE` 会阻止数据库执行保存操作，即使已经设定有一个或多个保存点(你可以将这一用法看作是强制停止服务器的一个假想的 ABORT 命令)

### 返回值

执行失败时返回错误。 执行成功时不返回任何信息，服务器和客户端的连接断开，客户端自动退出。

### 代码示例

```bash
redis> PING
PONG

redis> SHUTDOWN

$

$ redis
Could not connect to Redis at: Connection refused
not connected>
```

## TIME

返回当前服务器时间。

### 返回值

一个包含两个字符串的列表： 第一个字符串是当前时间(以 UNIX 时间戳格式表示)，而第二个字符串是当前这一秒钟已经逝去的微秒数。

### 代码示例

```bash
redis> TIME
1) "1332395997"
2) "952581"

redis> TIME
1) "1332395997"
2) "953148"
```

## CLIENT GETNAME

返回 `CLIENT SETNAME` 命令为连接设置的名字。

因为新创建的连接默认是没有名字的， 对于没有名字的连接， `CLIENT GETNAME` 返回空白回复。

### 返回值

如果连接没有设置名字，那么返回空白回复； 如果有设置名字，那么返回名字。

### 代码示例

```bash
# 新连接默认没有名字

redis 127.0.0.1:6379> CLIENT GETNAME
(nil)

# 设置名字

redis 127.0.0.1:6379> CLIENT SETNAME hello-world-connection
OK

# 返回名字

redis 127.0.0.1:6379> CLIENT GETNAME
"hello-world-connection"
```

## CLIENT KILL

**CLIENT KILL ip:port**

关闭地址为 `ip:port` 的客户端。

`ip:port` 应该和 `CLIENT LIST` 命令输出的其中一行匹配。

因为 Redis 使用单线程设计，所以当 Redis 正在执行命令的时候，不会有客户端被断开连接。

如果要被断开连接的客户端正在执行命令，那么当这个命令执行之后，在发送下一个命令的时候，它就会收到一个网络错误，告知它自身的连接已被关闭。

### 返回值

当指定的客户端存在，且被成功关闭时，返回 `OK` 。

### 代码示例

```bash
# 列出所有已连接客户端

redis 127.0.0.1:6379> CLIENT LIST
addr=127.0.0.1:43501 fd=5 age=10 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client

# 杀死当前客户端的连接

redis 127.0.0.1:6379> CLIENT KILL 127.0.0.1:43501
OK

# 之前的连接已经被关闭，CLI 客户端又重新建立了连接
# 之前的端口是 43501 ，现在是 43504

redis 127.0.0.1:6379> CLIENT LIST
addr=127.0.0.1:43504 fd=5 age=0 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
```

## CLIENT LIST

以人类可读的格式，返回所有连接到服务器的客户端信息和统计数据。

```bash
redis> CLIENT LIST
addr=127.0.0.1:43143 fd=6 age=183 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=32768 obl=0 oll=0 omem=0 events=r cmd=client
addr=127.0.0.1:43163 fd=5 age=35 idle=15 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=ping
addr=127.0.0.1:43167 fd=7 age=24 idle=6 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=0 oll=0 omem=0 events=r cmd=get
```

### 返回值

命令返回多行字符串，这些字符串按以下形式被格式化：

- 每个已连接客户端对应一行（以 `LF` 分割）
- 每行字符串由一系列 `属性=值` 形式的域组成，每个域之间以空格分开

以下是域的含义：

- `addr` ： 客户端的地址和端口
- `fd` ： 套接字所使用的文件描述符
- `age` ： 以秒计算的已连接时长
- `idle` ： 以秒计算的空闲时长
- `flags` ： 客户端 flag （见下文）
- `db` ： 该客户端正在使用的数据库 ID
- `sub` ： 已订阅频道的数量
- `psub` ： 已订阅模式的数量
- `multi` ： 在事务中被执行的命令数量
- `qbuf` ： 查询缓冲区的长度（字节为单位， `0` 表示没有分配查询缓冲区）
- `qbuf-free` ： 查询缓冲区剩余空间的长度（字节为单位， `0` 表示没有剩余空间）
- `obl` ： 输出缓冲区的长度（字节为单位， `0` 表示没有分配输出缓冲区）
- `oll` ： 输出列表包含的对象数量（当输出缓冲区没有剩余空间时，命令回复会以字符串对象的形式被入队到这个队列里）
- `omem` ： 输出缓冲区和输出列表占用的内存总量
- `events` ： 文件描述符事件（见下文）
- `cmd` ： 最近一次执行的命令

客户端 flag 可以由以下部分组成：

- `O` ： 客户端是 MONITOR 模式下的附属节点（slave）
- `S` ： 客户端是一般模式下（normal）的附属节点
- `M` ： 客户端是主节点（master）
- `x` ： 客户端正在执行事务
- `b` ： 客户端正在等待阻塞事件
- `i` ： 客户端正在等待 VM I/O 操作（已废弃）
- `d` ： 一个受监视（watched）的键已被修改， `EXEC` 命令将失败
- `c` : 在将回复完整地写出之后，关闭链接
- `u` : 客户端未被阻塞（unblocked）
- `A` : 尽可能快地关闭连接
- `N` : 未设置任何 flag

文件描述符事件可以是：

- `r` : 客户端套接字（在事件 loop 中）是可读的（readable）
- `w` : 客户端套接字（在事件 loop 中）是可写的（writeable）

Note

为了 debug 的需要，经常会对域进行添加和删除，一个安全的 Redis 客户端应该可以对 `CLIENT LIST` 的输出进行相应的处理（parse），比如忽略不存在的域，跳过未知域，诸如此类。

## CLIENT SETNAME

**CLIENT SETNAME connection-name**

为当前连接分配一个名字。

这个名字会显示在 `CLIENT LIST` 命令的结果中， 用于识别当前正在与服务器进行连接的客户端。

举个例子， 在使用 Redis 构建队列（queue）时， 可以根据连接负责的任务（role）， 为信息生产者（producer）和信息消费者（consumer）分别设置不同的名字。

名字使用 Redis 的字符串类型来保存， 最大可以占用 512 MB 。 另外， 为了避免和 `CLIENT LIST` 命令的输出格式发生冲突， 名字里不允许使用空格。

要移除一个连接的名字， 可以将连接的名字设为空字符串 `""` 。

使用 `CLIENT GETNAME` 命令可以取出连接的名字。

新创建的连接默认是没有名字的。

Tip

在 Redis 应用程序发生连接泄漏时，为连接设置名字是一种很好的 debug 手段。

### 返回值

设置成功时返回 `OK` 。

### 代码示例

```bash
# 新连接默认没有名字

redis 127.0.0.1:6379> CLIENT GETNAME
(nil)

# 设置名字

redis 127.0.0.1:6379> CLIENT SETNAME hello-world-connection
OK

# 返回名字

redis 127.0.0.1:6379> CLIENT GETNAME
"hello-world-connection"

# 在客户端列表中查看

redis 127.0.0.1:6379> CLIENT LIST
addr=127.0.0.1:36851
fd=5
name=hello-world-connection     # <- 名字
age=51
...

# 清除名字

redis 127.0.0.1:6379> CLIENT SETNAME        # 只用空格是不行的！
(error) ERR Syntax error, try CLIENT (LIST | KILL ip:port)

redis 127.0.0.1:6379> CLIENT SETNAME ""     # 必须双引号显示包围
OK

redis 127.0.0.1:6379> CLIENT GETNAME        # 清除完毕
(nil)
```

## CLIENT PAUSE

## CLIENT REPLY

## CLIENT UNBLOCK

# 配置选项

## CONFIG SET

**CONFIG SET parameter value**

`CONFIG SET` 命令可以动态地调整 Redis 服务器的配置(configuration)而无须重启。

你可以使用它修改配置参数，或者改变 Redis 的持久化(Persistence)方式。

`CONFIG SET` 可以修改的配置参数可以使用命令 `CONFIG GET *` 来列出，所有被 `CONFIG SET` 修改的配置参数都会立即生效。

关于 `CONFIG SET` 命令的更多消息，请参见命令 `CONFIG GET` 的说明。

关于如何使用 `CONFIG SET` 命令修改 Redis 持久化方式，请参见 [Redis Persistence](http://redis.io/topics/persistence) 。

### 返回值

当设置成功时返回 `OK` ，否则返回一个错误。

### 代码示例

```bash
redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "1024"

redis> CONFIG SET slowlog-max-len 10086
OK

redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "10086"
```

## CONFIG GET

**CONFIG GET parameter**

`CONFIG GET` 命令用于取得运行中的 Redis 服务器的配置参数(configuration parameters)，在 Redis 2.4 版本中， 有部分参数没有办法用 `CONFIG GET` 访问，但是在最新的 Redis 2.6 版本中，所有配置参数都已经可以用 `CONFIG GET` 访问了。

`CONFIG GET` 接受单个参数 `parameter` 作为搜索关键字，查找所有匹配的配置参数，其中参数和值以“键-值对”(key-value pairs)的方式排列。

比如执行 `CONFIG GET s*` 命令，服务器就会返回所有以 `s` 开头的配置参数及参数的值：

```bash
redis> CONFIG GET s*
1) "save"                       # 参数名：save
2) "900 1 300 10 60 10000"      # save 参数的值
3) "slave-serve-stale-data"     # 参数名： slave-serve-stale-data
4) "yes"                        # slave-serve-stale-data 参数的值
5) "set-max-intset-entries"     # ...
6) "512"
7) "slowlog-log-slower-than"
8) "1000"
9) "slowlog-max-len"
10) "1000"
```

如果你只是寻找特定的某个参数的话，你当然也可以直接指定参数的名字：

```bash
redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "1000"
```

使用命令 `CONFIG GET *` ，可以列出 `CONFIG GET` 命令支持的所有参数：

```bash
redis> CONFIG GET *
1) "dir"
2) "/var/lib/redis"
3) "dbfilename"
4) "dump.rdb"
5) "requirepass"
6) (nil)
7) "masterauth"
8) (nil)
9) "maxmemory"
10) "0"
11) "maxmemory-policy"
12) "volatile-lru"
13) "maxmemory-samples"
14) "3"
15) "timeout"
16) "0"
17) "appendonly"
18) "no"
# ...
49) "loglevel"
50) "verbose"
```

所有被 `CONFIG SET` 所支持的配置参数都可以在配置文件 redis.conf 中找到，不过 `CONFIG GET` 和 `CONFIG SET` 使用的格式和 redis.conf 文件所使用的格式有以下两点不同：

- `10kb` 、 `2gb` 这些在配置文件中所使用的储存单位缩写，不可以用在 `CONFIG` 命令中， `CONFIG SET` 的值只能通过数字值显式地设定。
  
  像 `CONFIG SET xxx 1k` 这样的命令是错误的，正确的格式是 `CONFIG SET xxx 1000` 。

- `save` 选项在 redis.conf 中是用多行文字储存的，但在 `CONFIG GET` 命令中，它只打印一行文字。
  
  以下是 `save` 选项在 redis.conf 文件中的表示：
  
  `save 900 1`
  
  `save 300 10`
  
  `save 60 10000`
  
  但是 `CONFIG GET` 命令的输出只有一行：
  
  `redis> CONFIG GET save`
  
  `1) "save"`
  
  `2) "900 1 300 10 60 10000"`
  
  上面 `save` 参数的三个值表示：在 900 秒内最少有 1 个 key 被改动，或者 300 秒内最少有 10 个 key 被改动，又或者 60 秒内最少有 1000 个 key 被改动，以上三个条件随便满足一个，就触发一次保存操作。

### 返回值

给定配置参数的值。

## CONFIG RESETSTAT

重置 `INFO` 命令中的某些统计数据，包括：

- Keyspace hits (键空间命中次数)
- Keyspace misses (键空间不命中次数)
- Number of commands processed (执行命令的次数)
- Number of connections received (连接服务器的次数)
- Number of expired keys (过期key的数量)
- Number of rejected connections (被拒绝的连接数量)
- Latest fork(2) time(最后执行 fork(2) 的时间)
- The `aof_delayed_fsync` counter(`aof_delayed_fsync` 计数器的值)

### 返回值

总是返回 `OK` 。

### 代码示例

## CONFIG REWRITE

`CONFIG REWRITE` 命令对启动 Redis 服务器时所指定的 `redis.conf` 文件进行改写： 因为 `CONFIG_SET` 命令可以对服务器的当前配置进行修改， 而修改后的配置可能和 `redis.conf` 文件中所描述的配置不一样， `CONFIG REWRITE` 的作用就是通过尽可能少的修改， 将服务器当前所使用的配置记录到 `redis.conf` 文件中。

重写会以非常保守的方式进行：

- 原有 `redis.conf` 文件的整体结构和注释会被尽可能地保留。
- 如果一个选项已经存在于原有 `redis.conf` 文件中 ， 那么对该选项的重写会在选项原本所在的位置（行号）上进行。
- 如果一个选项不存在于原有 `redis.conf` 文件中， 并且该选项被设置为默认值， 那么重写程序不会将这个选项添加到重写后的 `redis.conf` 文件中。
- 如果一个选项不存在于原有 `redis.conf` 文件中， 并且该选项被设置为非默认值， 那么这个选项将被添加到重写后的 `redis.conf` 文件的末尾。
- 未使用的行会被留白。 比如说， 如果你在原有 `redis.conf` 文件上设置了数个关于 `save` 选项的参数， 但现在你将这些 `save` 参数的一个或全部都关闭了， 那么这些不再使用的参数原本所在的行就会变成空白的。

即使启动服务器时所指定的 `redis.conf` 文件已经不再存在， `CONFIG REWRITE` 命令也可以重新构建并生成出一个新的 `redis.conf` 文件。

另一方面， 如果启动服务器时没有载入 `redis.conf` 文件， 那么执行 `CONFIG REWRITE` 命令将引发一个错误。

### 原子性重写

对 `redis.conf` 文件的重写是原子性的， 并且是一致的： 如果重写出错或重写期间服务器崩溃， 那么重写失败， 原有 `redis.conf` 文件不会被修改。 如果重写成功， 那么 `redis.conf` 文件为重写后的新文件。

### 返回值

一个状态值：如果配置重写成功则返回 `OK` ，失败则返回一个错误。

### 代码示例

以下是执行 `CONFIG REWRITE` 前， 被载入到 Redis 服务器的 `redis.conf` 文件中关于 `appendonly` 选项的设置：

```bash
# ... 其他选项

appendonly no

# ... 其他选项
```

在执行以下命令之后：

```bash
redis> CONFIG GET appendonly           # appendonly 处于关闭状态
1) "appendonly"
2) "no"

redis> CONFIG SET appendonly yes       # 打开 appendonly
OK

redis> CONFIG GET appendonly
1) "appendonly"
2) "yes"

redis> CONFIG REWRITE                  # 将 appendonly 的修改写入到 redis.conf 中
OK
```

重写后的 `redis.conf` 文件中的 `appendonly` 选项将被改写：

```bash
# ... 其他选项

appendonly yes

# ... 其他选项
```

# 调试

## PING

使用客户端向 Redis 服务器发送一个 `PING` ，如果服务器运作正常的话，会返回一个 `PONG` 。

通常用于测试与服务器的连接是否仍然生效，或者用于测量延迟值。

### 返回值

如果连接正常就返回一个 `PONG` ，否则返回一个连接错误。

### 代码示例

```bash
# 客户端和服务器连接正常

redis> PING
PONG

# 客户端和服务器连接不正常(网络不正常或服务器未能正常运行)

redis 127.0.0.1:6379> PING
Could not connect to Redis at 127.0.0.1:6379: Connection refused
```

## ECHO message

打印一个特定的信息 `message` ，测试时使用。

### 返回值

`message` 自身。

### 代码示例

```bash
redis> ECHO "Hello Moto"
"Hello Moto"

redis> ECHO "Goodbye Moto"
"Goodbye Moto"
```

## OBJECT

**OBJECT subcommand [arguments [arguments]]**

`OBJECT` 命令允许从内部察看给定 `key` 的 Redis 对象， 它通常用在除错(debugging)或者了解为了节省空间而对 `key` 使用特殊编码的情况。 当将Redis用作缓存程序时，你也可以通过 `OBJECT` 命令中的信息，决定 `key` 的驱逐策略(eviction policies)。

OBJECT 命令有多个子命令：

- `OBJECT REFCOUNT ` 返回给定 `key` 引用所储存的值的次数。此命令主要用于除错。
- `OBJECT ENCODING ` 返回给定 `key` 锁储存的值所使用的内部表示(representation)。
- `OBJECT IDLETIME ` 返回给定 `key` 自储存以来的空闲时间(idle， 没有被读取也没有被写入)，以秒为单位。

对象可以以多种方式编码：

- 字符串可以被编码为 `raw` (一般字符串)或 `int` (为了节约内存，Redis 会将字符串表示的 64 位有符号整数编码为整数来进行储存）。
- 列表可以被编码为 `ziplist` 或 `linkedlist` 。 `ziplist` 是为节约大小较小的列表空间而作的特殊表示。
- 集合可以被编码为 `intset` 或者 `hashtable` 。 `intset` 是只储存数字的小集合的特殊表示。
- 哈希表可以编码为 `zipmap` 或者 `hashtable` 。 `zipmap` 是小哈希表的特殊表示。
- 有序集合可以被编码为 `ziplist` 或者 `skiplist` 格式。 `ziplist` 用于表示小的有序集合，而 `skiplist` 则用于表示任何大小的有序集合。

假如你做了什么让 Redis 没办法再使用节省空间的编码时(比如将一个只有 1 个元素的集合扩展为一个有 100 万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general type)。

### 返回值

`REFCOUNT` 和 `IDLETIME` 返回数字。 `ENCODING` 返回相应的编码类型。

### 代码示例

```bash
redis> SET game "COD"           # 设置一个字符串
OK

redis> OBJECT REFCOUNT game     # 只有一个引用
(integer) 1

redis> OBJECT IDLETIME game     # 等待一阵。。。然后查看空闲时间
(integer) 90

redis> GET game                 # 提取game， 让它处于活跃(active)状态
"COD"

redis> OBJECT IDLETIME game     # 不再处于空闲状态
(integer) 0

redis> OBJECT ENCODING game     # 字符串的编码方式
"raw"

redis> SET big-number 23102930128301091820391092019203810281029831092  # 非常长的数字会被编码为字符串
OK

redis> OBJECT ENCODING big-number
"raw"

redis> SET small-number 12345  # 而短的数字则会被编码为整数
OK

redis> OBJECT ENCODING small-number
"int"
```

## SLOWLOG

**SLOWLOG subcommand [argument]**

### 什么是 SLOWLOG

Slow log 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

### 设置 SLOWLOG

Slow log 的行为由两个配置参数(configuration parameter)指定，可以通过改写 redis.conf 文件或者用 `CONFIG GET` 和 `CONFIG SET` 命令对它们动态地进行修改。

第一个选项是 `slowlog-log-slower-than` ，它决定要对执行时间大于多少微秒(microsecond，1秒 = 1,000,000 微秒)的查询进行记录。

比如执行以下命令将让 slow log 记录所有查询时间大于等于 100 微秒的查询：

```bash
CONFIG SET slowlog-log-slower-than 100
```

而以下命令记录所有查询时间大于 1000 微秒的查询：

```bash
CONFIG SET slowlog-log-slower-than 1000
```

另一个选项是 `slowlog-max-len` ，它决定 slow log *最多*能保存多少条日志， slow log 本身是一个 FIFO 队列，当队列大小超过 `slowlog-max-len` 时，最旧的一条日志将被删除，而最新的一条日志加入到 slow log ，以此类推。

以下命令让 slow log 最多保存 1000 条日志：

```
CONFIG SET slowlog-max-len 1000
```

使用 `CONFIG GET` 命令可以查询两个选项的当前值：

```bash
redis> CONFIG GET slowlog-log-slower-than
1) "slowlog-log-slower-than"
2) "1000"

redis> CONFIG GET slowlog-max-len
1) "slowlog-max-len"
2) "1000"
```

### 查看 slow log

要查看 slow log ，可以使用 `SLOWLOG GET` 或者 `SLOWLOG GET number` 命令，前者打印所有 slow log ，最大长度取决于 `slowlog-max-len` 选项的值，而 `SLOWLOG GET number` 则只打印指定数量的日志。

最新的日志会最先被打印：

```bash
# 为测试需要，将 slowlog-log-slower-than 设成了 10 微秒

redis> SLOWLOG GET
1) 1) (integer) 12                      # 唯一性(unique)的日志标识符
   2) (integer) 1324097834              # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                      # 查询执行时间，以微秒为单位
   4) 1) "CONFIG"                       # 执行的命令，以数组的形式排列
      2) "GET"                          # 这里完整的命令是 CONFIG GET slowlog-log-slower-than
      3) "slowlog-log-slower-than"

2) 1) (integer) 11
   2) (integer) 1324097825
   3) (integer) 42
   4) 1) "CONFIG"
      2) "GET"
      3) "*"

3) 1) (integer) 10
   2) (integer) 1324097820
   3) (integer) 11
   4) 1) "CONFIG"
      2) "GET"
      3) "slowlog-log-slower-than"

# ...
```

日志的唯一 id 只有在 Redis 服务器重启的时候才会重置，这样可以避免对日志的重复处理(比如你可能会想在每次发现新的慢查询时发邮件通知你)。

### 查看当前日志的数量

使用命令 `SLOWLOG LEN` 可以查看当前日志的数量。

请注意这个值和 `slower-max-len` 的区别，它们一个是当前日志的数量，一个是允许记录的最大日志的数量。

```
redis> SLOWLOG LEN
(integer) 14
```

### 清空日志

使用命令 `SLOWLOG RESET` 可以清空 slow log 。

```bash
redis> SLOWLOG LEN
(integer) 14

redis> SLOWLOG RESET
OK

redis> SLOWLOG LEN
(integer) 0
```

### 返回值

取决于不同命令，返回不同的值。

## MONITOR

实时打印出 Redis 服务器接收到的命令，调试用。

### 返回值

总是返回 `OK` 。

### 代码示例

```bash
127.0.0.1:6379> MONITOR
OK
# 以第一个打印值为例
# 1378822099.421623 是时间戳
# [0 127.0.0.1:56604] 中的 0 是数据库号码， 127... 是 IP 地址和端口
# "PING" 是被执行的命令
1378822099.421623 [0 127.0.0.1:56604] "PING"
1378822105.089572 [0 127.0.0.1:56604] "SET" "msg" "hello world"
1378822109.036925 [0 127.0.0.1:56604] "SET" "number" "123"
1378822140.649496 [0 127.0.0.1:56604] "SADD" "fruits" "Apple" "Banana" "Cherry"
1378822154.117160 [0 127.0.0.1:56604] "EXPIRE" "msg" "10086"
1378822257.329412 [0 127.0.0.1:56604] "KEYS" "*"
1378822258.690131 [0 127.0.0.1:56604] "DBSIZE"
```

## DEBUG OBJECT

**DEBUG OBJECT key**

`DEBUG OBJECT` 是一个调试命令，它不应被客户端所使用。

查看 `OBJECT` 命令获取更多信息。

### 返回值

当 `key` 存在时，返回有关信息。 当 `key` 不存在时，返回一个错误。

### 代码示例

```bash
redis> DEBUG OBJECT my_pc
Value at:0xb6838d20 refcount:1 encoding:raw serializedlength:9 lru:283790 lru_seconds_idle:150

redis> DEBUG OBJECT your_mac
(error) ERR no such key
```

## DEBUG SEGFAULT

执行一个不合法的内存访问从而让 Redis 崩溃，仅在开发时用于 BUG 模拟。

### 返回值

无

### 代码示例

```bash
redis> DEBUG SEGFAULT
Could not connect to Redis at: Connection refused

not connected>
```

# 内部命令

## MIGRATE

**MIGRATE host port key destination-db timeout [COPY] [REPLACE]**

将 `key` 原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功， `key` 保证会出现在目标实例上，而当前实例上的 `key` 会被删除。

这个命令是一个原子操作，它在执行的时候会阻塞进行迁移的两个实例，直到以下任意结果发生：迁移成功，迁移失败，等待超时。

命令的内部实现是这样的：它在当前实例对给定 `key` 执行 `DUMP` 命令 ，将它序列化，然后传送到目标实例，目标实例再使用 `RESTORE` 对数据进行反序列化，并将反序列化所得的数据添加到数据库中；当前实例就像目标实例的客户端那样，只要看到 `RESTORE` 命令返回 `OK` ，它就会调用 `DEL` 删除自己数据库上的 `key` 。

`timeout` 参数以毫秒为格式，指定当前实例和目标实例进行沟通的**最大间隔时间**。这说明操作并不一定要在 `timeout` 毫秒内完成，只是说数据传送的时间不能超过这个 `timeout` 数。

`MIGRATE` 命令需要在给定的时间规定内完成 IO 操作。如果在传送数据时发生 IO 错误，或者达到了超时时间，那么命令会停止执行，并返回一个特殊的错误： `IOERR` 。

当 `IOERR` 出现时，有以下两种可能：

- `key` 可能存在于两个实例
- `key` 可能只存在于当前实例

唯一不可能发生的情况就是丢失 `key` ，因此，如果一个客户端执行 `MIGRATE` 命令，并且不幸遇上 `IOERR` 错误，那么这个客户端唯一要做的就是检查自己数据库上的 `key` 是否已经被正确地删除。

如果有其他错误发生，那么 `MIGRATE` 保证 `key` 只会出现在当前实例中。（当然，目标实例的给定数据库上可能有和 `key` 同名的键，不过这和 `MIGRATE` 命令没有关系）。

### 可选项

- `COPY` ：不移除源实例上的 `key` 。
- `REPLACE` ：替换目标实例上已存在的 `key` 。

### 返回值

迁移成功时返回 `OK` ，否则返回相应的错误。

### 代码示例

先启动两个 Redis 实例，一个使用默认的 6379 端口，一个使用 7777 端口。

```bash
$ ./redis-server &
[1] 3557

...

$ ./redis-server --port 7777 &
[2] 3560

...
```

然后用客户端连上 6379 端口的实例，设置一个键，然后将它迁移到 7777 端口的实例上：

```bash
$ ./redis-cli

redis 127.0.0.1:6379> flushdb
OK

redis 127.0.0.1:6379> SET greeting "Hello from 6379 instance"
OK

redis 127.0.0.1:6379> MIGRATE 127.0.0.1 7777 greeting 0 1000
OK

redis 127.0.0.1:6379> EXISTS greeting                           # 迁移成功后 key 被删除
(integer) 0
```

使用另一个客户端，查看 7777 端口上的实例：

```bash
$ ./redis-cli -p 7777

redis 127.0.0.1:7777> GET greeting
"Hello from 6379 instance"
```

## DUMP

**DUMP key**

序列化给定 `key` ，并返回被序列化的值，使用 `RESTORE` 命令可以将这个值反序列化为 Redis 键。

序列化生成的值有以下几个特点：

- 它带有 64 位的校验和，用于检测错误， `RESTORE` 在进行反序列化之前会先检查校验和。
- 值的编码格式和 RDB 文件保持一致。
- RDB 版本会被编码在序列化值当中，如果因为 Redis 的版本不同造成 RDB 格式不兼容，那么 Redis 会拒绝对这个值进行反序列化操作。

序列化的值不包括任何生存时间信息。

### 返回值

如果 `key` 不存在，那么返回 `nil` 。 否则，返回序列化之后的值。

### 代码示例

```bash
redis> SET greeting "hello, dumping world!"
OK

redis> DUMP greeting
"\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

redis> DUMP not-exists-key
(nil)
```

## RESTORE

**RESTORE key ttl serialized-value [REPLACE]**

反序列化给定的序列化值，并将它和给定的 `key` 关联。

参数 `ttl` 以毫秒为单位为 `key` 设置生存时间；如果 `ttl` 为 `0` ，那么不设置生存时间。

`RESTORE` 在执行反序列化之前会先对序列化值的 RDB 版本和数据校验和进行检查，如果 RDB 版本不相同或者数据不完整的话，那么 `RESTORE` 会拒绝进行反序列化，并返回一个错误。

如果键 `key` 已经存在， 并且给定了 `REPLACE` 选项， 那么使用反序列化得出的值来代替键 `key` 原有的值； 相反地， 如果键 `key` 已经存在， 但是没有给定 `REPLACE` 选项， 那么命令返回一个错误。

更多信息可以参考 `DUMP` 命令。

### 返回值

如果反序列化成功那么返回 `OK` ，否则返回一个错误。

### 代码示例

```bash
# 创建一个键，作为 DUMP 命令的输入

redis> SET greeting "hello, dumping world!"
OK

redis> DUMP greeting
"\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"

# 将序列化数据 RESTORE 到另一个键上面

redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
OK

redis> GET greeting-again
"hello, dumping world!"

# 在没有给定 REPLACE 选项的情况下，再次尝试反序列化到同一个键，失败

redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde"
(error) ERR Target key name is busy.

# 给定 REPLACE 选项，对同一个键进行反序列化成功

redis> RESTORE greeting-again 0 "\x00\x15hello, dumping world!\x06\x00E\xa0Z\x82\xd8r\xc1\xde" REPLACE
OK

# 尝试使用无效的值进行反序列化，出错

redis> RESTORE fake-message 0 "hello moto moto blah blah"
(error) ERR DUMP payload version or checksum are wrong
```

## SYNC

用于复制功能(replication)的内部命令。

### 返回值

序列化数据。

### 代码示例

```bash
redis> SYNC
"REDIS0002\xfe\x00\x00\auser_id\xc0\x03\x00\anumbers\xc2\xf3\xe0\x01\x00\x00\tdb_number\xc0\x00\x00\x04name\x06huangz\x00\anew_key\nhello_moto\x00\bgreeting\nhello moto\x00\x05my_pc\bthinkpad\x00\x04lock\xc0\x01\x00\nlock_times\xc0\x04\xfe\x01\t\x04info\x19\x02\x04name\b\x00zhangyue\x03age\x02\x0022\xff\t\aooredis,\x03\x04name\a\x00ooredis\aversion\x03\x001.0\x06author\x06\x00huangz\xff\x00\tdb_number\xc0\x01\x00\x05greet\x0bhello world\x02\nmy_friends\x02\x05marry\x04jack\x00\x04name\x05value\xfe\x02\x0c\x01s\x12\x12\x00\x00\x00\r\x00\x00\x00\x02\x00\x00\x01a\x03\xc0f'\xff\xff"
(1.90s)
```

## PSYNC

**PSYNC master_run_id offset**

用于复制功能(replication)的内部命令。

更多信息请参考 复制（Replication） 文档。

### 返回值

序列化数据。

### 代码示例

```bash
127.0.0.1:6379> PSYNC ? -1
"REDIS0006\xfe\x00\x00\x02kk\x02vv\x00\x03msg\x05hello\xff\xc3\x96P\x12h\bK\xef"
```

其他命令(Other Commands)
sort source-key [BY pattern] [Limit offset count] [Get pattern [Get pattern ...]] [Asc|Desc] [Alpha] [Store dest-key] - 根据给定的选项，对输入的列表、集合或者有序集合进行排序，返回或存储排序的结果

发布/订阅(publish/subscribe)
subscribe channel [channel ...] - 订阅给定的频道（一个或多个）
unsubscribe [channel [channel ...]] - 退订给定的频道，如果没有给定频道，将退订所有频道
publish channel message - 向给定频道发送消息
psubscribe pattern [pattern ...] - 订阅给定模式匹配的频道
punsubscribe [pattern [pattern ...]] - 退订给定pattern匹配的所有模式，如果没有给定模式，将退订所有模式

过期时间(expiring keys)
persist key-name - 移除键的过期时间
ttl key-name - 查看给定键距离过期还有多少秒
expire key-name seconds - 让指定键在给定秒数后过期
expireat key-name timestamp - 将给定的过期时间设置为给定的Unix时间戳
pttl key-name - 查看给定键距离过期还有多少毫秒(version >= 2.6)
pexpire key-name milliseconds - 让指定键在给定毫秒数后过期(version >= 2.6)</p>
pexpireat key-name timestamp-milliseconds - 将给定的过期时间设置为给定的毫秒级精度的Unix时间戳(version >= 2.6)

# Redis部署

**单机版**：单机部署，单机redis能够承载的 QPS 大概就在上万到几万不等。这种部署方式很少使用。存在的问题：1、内存容量有限 2、处理能力有限 3、无法高可用。

**主从模式**：一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读。所有的读请求全部走从节点。这样也可以很轻松实现水平扩容，支撑读高并发。master 节点挂掉后，需要手动指定新的 master，可用性不高，基本不用。

**哨兵模式**：主从复制存在不能自动故障转移、达不到高可用的问题。哨兵模式解决了这些问题。通过哨兵机制可以自动切换主从节点。master 节点挂掉后，哨兵进程会主动选举新的 master，可用性高，但是每个节点存储的数据是一样的，浪费内存空间。数据量不是很多，集群规模不是很大，需要自动容错容灾的时候使用。

**Redis cluster**：服务端分片技术，3.0版本开始正式提供。Redis Cluster并没有使用一致性hash，而是采用slot(槽)的概念，一共分成16384个槽。将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行。主要是针对海量数据+高并发+高可用的场景，如果是海量数据，如果你的数据量很大，那么建议就用Redis cluster，所有主节点的容量总和就是Redis cluster可缓存的数据容量。

## 主从架构

单机的 redis，能够承载的 QPS 大概就在上万到几万不等。对于缓存来说，一般都是用来支撑读高并发的。因此架构做成主从(master-slave)架构，一主多从，主负责写，并且将数据复制到其它的 slave 节点，从节点负责读。所有的读请求全部走从节点。这样也可以很轻松实现水平扩容，支撑读高并发。

Redis的复制功能是支持多个数据库之间的数据同步。主数据库可以进行读写操作，当主数据库的数据发生变化时会自动将数据同步到从数据库。从数据库一般是只读的，它会接收主数据库同步过来的数据。一个主数据库可以有多个从数据库，而一个从数据库只能有一个主数据库。

**主从复制的原理？**

1. 当启动一个从节点时，它会发送一个 `PSYNC` 命令给主节点；
2. 如果是从节点初次连接到主节点，那么会触发一次全量复制。此时主节点会启动一个后台线程，开始生成一份 `RDB` 快照文件；
3. 同时还会将从客户端 client 新收到的所有写命令缓存在内存中。`RDB` 文件生成完毕后， 主节点会将`RDB`文件发送给从节点，从节点会先将`RDB`文件**写入本地磁盘，然后再从本地磁盘加载到内存中**；
4. 接着主节点会将内存中缓存的写命令发送到从节点，从节点同步这些数据；
5. 如果从节点跟主节点之间网络出现故障，连接断开了，会自动重连，连接之后主节点仅会将部分缺失的数据同步给从节点。

## 哨兵Sentinel

主从复制存在不能自动故障转移、达不到高可用的问题。哨兵模式解决了这些问题。通过哨兵机制可以自动切换主从节点。

客户端连接Redis的时候，先连接哨兵，哨兵会告诉客户端Redis主节点的地址，然后客户端连接上Redis并进行后续的操作。当主节点宕机的时候，哨兵监测到主节点宕机，会重新推选出某个表现良好的从节点成为新的主节点，然后通过发布订阅模式通知其他的从服务器，让它们切换主机。

**工作原理**

- 每个`Sentinel`以每秒钟一次的频率向它所知道的`Master`，`Slave`以及其他 `Sentinel` 实例发送一个 `PING`命令。
- 如果一个实例距离最后一次有效回复 `PING` 命令的时间超过指定值， 则这个实例会被 `Sentine` 标记为主观下线。
- 如果一个`Master`被标记为主观下线，则正在监视这个`Master`的所有 `Sentinel` 要以每秒一次的频率确认`Master`是否真正进入主观下线状态。
- 当有足够数量的 `Sentinel`（大于等于配置文件指定值）在指定的时间范围内确认`Master`的确进入了主观下线状态， 则`Master`会被标记为客观下线 。若没有足够数量的 `Sentinel` 同意 `Master` 已经下线， `Master` 的客观下线状态就会被解除。 若 `Master`重新向 `Sentinel` 的 `PING` 命令返回有效回复， `Master` 的主观下线状态就会被移除。
- 哨兵节点会选举出哨兵 leader，负责故障转移的工作。
- 哨兵 leader 会推选出某个表现良好的从节点成为新的主节点，然后通知其他从节点更新主节点信息。

## Redis cluster

哨兵模式解决了主从复制不能自动故障转移、达不到高可用的问题，但还是存在主节点的写能力、容量受限于单机配置的问题。而cluster模式实现了Redis的分布式存储，每个节点存储不同的内容，解决主节点的写能力、容量受限于单机配置的问题。

Redis cluster集群节点最小配置6个节点以上（3主3从），其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用。

Redis cluster采用**虚拟槽分区**，所有的键根据哈希函数映射到0～16383个整数槽内，每个节点负责维护一部分槽以及槽所映射的键值数据。

**工作原理：**

1. 通过哈希的方式，将数据分片，每个节点均分存储一定哈希槽(哈希值)区间的数据，默认分配了16384 个槽位
2. 每份数据分片会存储在多个互为主从的多节点上
3. 数据写入先写主节点，再同步到从节点(支持配置为阻塞同步)
4. 同一分片多个节点间的数据不保持一致性
5. 读取数据时，当客户端操作的key没有分配在该节点上时，redis会返回转向指令，指向正确的节点
6. 扩容时时需要需要把旧节点的数据迁移一部分到新节点

在 redis cluster 架构下，每个 redis 要放开两个端口号，比如一个是 6379，另外一个就是 加1w 的端口号，比如 16379。

16379 端口号是用来进行节点间通信的，也就是 cluster bus 的东西，cluster bus 的通信，用来进行故障检测、配置更新、故障转移授权。cluster bus 用了另外一种二进制的协议，`gossip` 协议，用于节点间进行高效的数据交换，占用更少的网络带宽和处理时间。

**优点：**

- 无中心架构，**支持动态扩**容；
- 数据按照`slot`存储分布在多个节点，节点间数据共享，**可动态调整数据分布**；
- **高可用性**。部分节点不可用时，集群仍可用。集群模式能够实现自动故障转移（failover），节点之间通过`gossip`协议交换状态信息，用投票机制完成`Slave`到`Master`的角色转换。

**缺点：**

- **不支持批量操作**（pipeline）。
- 数据通过异步复制，**不保证数据的强一致性**。
- **事务操作支持有限**，只支持多`key`在同一节点上的事务操作，当多个`key`分布于不同的节点上时无法使用事务功能。
- `key`作为数据分区的最小粒度，不能将一个很大的键值对象如`hash`、`list`等映射到不同的节点。
- **不支持多数据库空间**，单机下的Redis可以支持到16个数据库，集群模式下只能使用1个数据库空间。
- 只能使用0号数据库。

**哈希分区算法有哪些？**

节点取余分区。使用特定的数据，如Redis的键或用户ID，对节点数量N取余：hash（key）%N计算出哈希值，用来决定数据映射到哪一个节点上。
优点是简单性。扩容时通常采用翻倍扩容，避免数据映射全部被打乱导致全量迁移的情况。

一致性哈希分区。为系统中每个节点分配一个token，范围一般在0~232，这些token构成一个哈希环。数据读写执行节点查找操作时，先根据key计算hash值，然后顺时针找到第一个大于等于该哈希值的token节点。
这种方式相比节点取余最大的好处在于加入和删除节点只影响哈希环中相邻的节点，对其他节点无影响。

虚拟槽分区，所有的键根据哈希函数映射到0~16383整数槽内，计算公式：slot=CRC16（key）&16383。每一个节点负责维护一部分槽以及槽所映射的键值数据。**Redis Cluser采用虚拟槽分区算法。 

# Redis 集群规范

Redis 集群是一个分布式（distributed）、容错（fault-tolerant）的 Redis 实现， 集群可以使用的功能是普通单机 Redis 所能使用的功能的一个子集（subset）

Redis 集群中不存在中心（central）节点或者代理（proxy）节点， 集群的其中一个主要设计目标是达到线性可扩展性（linear scalability）

Redis 集群为了保证一致性（consistency）而牺牲了一部分容错性： 系统会在保证对网络断线（net split）和节点失效（node failure）具有有限（limited）抵抗力的前提下， 尽可能地保持数据的一致性

Note

集群将节点失效视为网络断线的其中一种特殊情况。

集群的容错功能是通过使用主节点（master）和从节点（slave）两种角色（role）的节点（node）来实现的：

- 主节点和从节点使用完全相同的服务器实现， 它们的功能（functionally）也完全一样， 但从节点通常仅用于替换失效的主节点。
- 不过， 如果不需要保证“先写入，后读取”操作的一致性（read-after-write consistency）， 那么可以使用从节点来执行只读查询。

## Redis 集群实现的功能子集

Redis 集群实现了单机 Redis 中， 所有处理单个数据库键的命令。

针对多个数据库键的复杂计算操作， 比如集合的并集操作、合集操作没有被实现， 那些理论上需要使用多个节点的多个数据库键才能完成的命令也没有被实现。

在将来， 用户也许可以通过 [MIGRATE COPY](#migrate) 命令， 在集群的计算节点（computation node）中执行针对多个数据库键的只读操作， 但集群本身不会去实现那些需要将多个数据库键在多个节点中移来移去的复杂多键命令。

Redis 集群不像单机 Redis 那样支持多数据库功能， 集群只使用默认的 `0` 号数据库， 并且不能使用 SELECT index 命令。

## Redis 集群协议中的客户端和服务器

Redis 集群中的节点有以下责任：

- 持有键值对数据。
- 记录集群的状态，包括键到正确节点的映射（mapping keys to right nodes）。
- 自动发现其他节点，识别工作不正常的节点，并在有需要时，在从节点中选举出新的主节点。

为了执行以上列出的任务， 集群中的每个节点都与其他节点建立起了“集群连接（cluster bus）”， 该连接是一个 TCP 连接， 使用二进制协议进行通讯。

节点之间使用 [Gossip 协议](http://en.wikipedia.org/wiki/Gossip_protocol) 来进行以下工作：

- 传播（propagate）关于集群的信息，以此来发现新的节点。
- 向其他节点发送 `PING` 数据包，以此来检查目标节点是否正常运作。
- 在特定事件发生时，发送集群信息。

除此之外， 集群连接还用于在集群中发布或订阅信息。

因为集群节点不能代理（proxy）命令请求， 所以客户端应该在节点返回 `-MOVED` 或者 `-ASK` 转向（redirection）错误时， 自行将命令请求转发至其他节点。

因为客户端可以自由地向集群中的任何一个节点发送命令请求， 并可以在有需要时， 根据转向错误所提供的信息， 将命令转发至正确的节点， 所以在理论上来说， 客户端是无须保存集群状态信息的。

不过， 如果客户端可以将键和节点之间的映射信息保存起来， 可以有效地减少可能出现的转向次数， 籍此提升命令执行的效率。

## 键分布模型

Redis 集群的键空间被分割为 `16384` 个槽（slot）， 集群的最大节点数量也是 `16384` 个。

Note

推荐的最大节点数量为 1000 个左右。

每个主节点都负责处理 `16384` 个哈希槽的其中一部分。

当我们说一个集群处于“稳定”（stable）状态时， 指的是集群没有在执行重配置（reconfiguration）操作， 每个哈希槽都只由一个节点进行处理。

Note

重配置指的是将某个/某些槽从一个节点移动到另一个节点。

Note

一个主节点可以有任意多个从节点， 这些从节点用于在主节点发生网络断线或者节点失效时， 对主节点进行替换。

以下是负责将键映射到槽的算法：

```
HASH_SLOT = CRC16(key) mod 16384
```

以下是该算法所使用的参数：

- 算法的名称: XMODEM (又称 ZMODEM 或者 CRC-16/ACORN)
- 结果的长度: 16 位
- 多项数（poly）: 1021 (也即是 `x16 + x12 + x5 + 1`)
- 初始化值: `0000`
- 反射输入字节（Reflect Input byte）: `False`
- 发射输出 CRC （Reflect Output CRC）: `False`
- 用于 CRC 输出值的异或常量（Xor constant to output CRC）: `0000`
- 该算法对于输入 `"123456789"` 的输出: `31C3`

附录 A 中给出了集群所使用的 CRC16 算法的实现。

CRC16 算法所产生的 16 位输出中的 14 位会被用到。

在我们的测试中， CRC16 算法可以很好地将各种不同类型的键平稳地分布到 `16384` 个槽里面。

## 集群节点属性

每个节点在集群中都有一个独一无二的 ID ， 该 ID 是一个十六进制表示的 160 位随机数， 在节点第一次启动时由 `/dev/urandom` 生成。

节点会将它的 ID 保存到配置文件， 只要这个配置文件不被删除， 节点就会一直沿用这个 ID 。

节点 ID 用于标识集群中的每个节点。 一个节点可以改变它的 IP 和端口号， 而不改变节点 ID 。 集群可以自动识别出 IP/端口号的变化， 并将这一信息通过 Gossip 协议广播给其他节点知道。

以下是每个节点都有的关联信息， 并且节点会将这些信息发送给其他节点：

- 节点所使用的 IP 地址和 TCP 端口号。
- 节点的标志（flags）。
- 节点负责处理的哈希槽。
- 节点最近一次使用集群连接发送 `PING` 数据包（packet）的时间。
- 节点最近一次在回复中接收到 `PONG` 数据包的时间。
- 集群将该节点标记为下线的时间。
- 该节点的从节点数量。
- 如果该节点是从节点的话，那么它会记录主节点的节点 ID 。 如果这是一个主节点的话，那么主节点 ID 这一栏的值为 `0000000` 。

以上信息的其中一部分可以通过向集群中的任意节点（主节点或者从节点都可以）发送 `CLUSTER NODES` 命令来获得。

以下是一个向集群中的主节点发送 `CLUSTER NODES` 命令的例子， 该集群由三个节点组成：

```
$ redis-cli cluster nodes
d1861060fe6a534d42d8a19aeb36600e18785e04 :0 myself - 0 1318428930 connected 0-1364
3886e65cc906bfd9b1f7e7bde468726a052d1dae 127.0.0.1:6380 master - 1318428930 1318428931 connected 1365-2729
d289c575dcbc4bdd2931585fd4339089e461a27d 127.0.0.1:6381 master - 1318428931 1318428931 connected 2730-4095
```

在上面列出的三行信息中， 从左到右的各个域分别是： 节点 ID ， IP 地址和端口号， 标志（flag）， 最后发送 `PING` 的时间， 最后接收 `PONG` 的时间， 连接状态， 节点负责处理的槽。

## 节点握手（已实现）

节点总是应答（accept）来自集群连接端口的连接请求， 并对接收到的 `PING` 数据包进行回复， 即使这个 `PING` 数据包来自不可信的节点。

然而， 除了 `PING` 之外， 节点会拒绝其他所有并非来自集群节点的数据包。

要让一个节点承认另一个节点同属于一个集群， 只有以下两种方法：

- 一个节点可以通过向另一个节点发送 `MEET` 信息， 来强制让接收信息的节点承认发送信息的节点为集群中的一份子。 一个节点仅在管理员显式地向它发送 `CLUSTER MEET ip port` 命令时， 才会向另一个节点发送 `MEET` 信息。
- 另外， 如果一个可信节点向另一个节点传播第三者节点的信息， 那么接收信息的那个节点也会将第三者节点识别为集群中的一份子。 也即是说， 如果 A 认识 B ， B 认识 C ， 并且 B 向 A 传播关于 C 的信息， 那么 A 也会将 C 识别为集群中的一份子， 并尝试连接 C 。

这意味着如果我们将一个/一些新节点添加到一个集群中， 那么这个/这些新节点最终会和集群中已有的其他所有节点连接起来。

这说明只要管理员使用 `CLUSTER MEET` 命令显式地指定了可信关系， 集群就可以自动发现其他节点。

这种节点识别机制通过防止不同的 Redis 集群因为 IP 地址变更或者其他网络事件的发生而产生意料之外的联合（mix）， 从而使得集群更具健壮性。

当节点的网络连接断开时， 它会主动连接其他已知的节点。

## MOVED 转向

一个 Redis 客户端可以向集群中的任意节点（包括从节点）发送命令请求。 节点会对命令请求进行分析， 如果该命令是集群可以执行的命令， 那么节点会查找这个命令所要处理的键所在的槽。

如果要查找的哈希槽正好就由接收到命令的节点负责处理， 那么节点就直接执行这个命令。

另一方面， 如果所查找的槽不是由该节点处理的话， 节点将查看自身内部所保存的哈希槽到节点 ID 的映射记录， 并向客户端回复一个 `MOVED` 错误。

以下是一个 `MOVED` 错误的例子：

```
GET x

-MOVED 3999 127.0.0.1:6381
```

错误信息包含键 `x` 所属的哈希槽 `3999` ， 以及负责处理这个槽的节点的 IP 和端口号 `127.0.0.1:6381` 。 客户端需要根据这个 IP 和端口号， 向所属的节点重新发送一次 GET key命令请求。

注意， 即使客户端在重新发送 GET key命令之前， 等待了非常久的时间， 以至于集群又再次更改了配置， 使得节点 `127.0.0.1:6381` 已经不再处理槽 `3999` ， 那么当客户端向节点 `127.0.0.1:6381` 发送 GET key 命令的时候， 节点将再次向客户端返回 `MOVED` 错误， 指示现在负责处理槽 `3999` 的节点。

虽然我们用 ID 来标识集群中的节点， 但是为了让客户端的转向操作尽可能地简单， 节点在 `MOVED` 错误中直接返回目标节点的 IP 和端口号， 而不是目标节点的 ID 。

虽然不是必须的， 但一个客户端应该记录（memorize）下“槽 `3999` 由节点 `127.0.0.1:6381` 负责处理“这一信息， 这样当再次有命令需要对槽 `3999` 执行时， 客户端就可以加快寻找正确节点的速度。

注意， 当集群处于稳定状态时， 所有客户端最终都会保存有一个哈希槽至节点的映射记录（map of hash slots to nodes）， 使得集群非常高效： 客户端可以直接向正确的节点发送命令请求， 无须转向、代理或者其他任何可能发生单点故障（single point failure）的实体（entiy）。

除了 `MOVED` 转向错误之外， 一个客户端还应该可以处理稍后介绍的 `ASK` 转向错误。

## 集群在线重配置（live reconfiguration）

Redis 集群支持在集群运行的过程中添加或者移除节点。

实际上， 节点的添加操作和节点的删除操作可以抽象成同一个操作， 那就是， 将哈希槽从一个节点移动到另一个节点：

- 添加一个新节点到集群， 等于将其他已存在节点的槽移动到一个空白的新节点里面。
- 从集群中移除一个节点， 等于将被移除节点的所有槽移动到集群的其他节点上面去。

因此， 实现 Redis 集群在线重配置的核心就是将槽从一个节点移动到另一个节点的能力。 因为一个哈希槽实际上就是一些键的集合， 所以 Redis 集群在重哈希（rehash）时真正要做的， 就是将一些键从一个节点移动到另一个节点。

要理解 Redis 集群如何将槽从一个节点移动到另一个节点， 我们需要对 `CLUSTER` 命令的各个子命令进行介绍， 这些命理负责管理集群节点的槽转换表（slots translation table）。

以下是 `CLUSTER` 命令可用的子命令：

- `CLUSTER ADDSLOTS slot1 [slot2] ... [slotN]`
- `CLUSTER DELSLOTS slot1 [slot2] ... [slotN]`
- `CLUSTER SETSLOT slot NODE node`
- `CLUSTER SETSLOT slot MIGRATING node`
- `CLUSTER SETSLOT slot IMPORTING node`

最开头的两条命令 `ADDSLOTS` 和 `DELSLOTS` 分别用于向节点指派（assign）或者移除节点， 当槽被指派或者移除之后， 节点会将这一信息通过 Gossip 协议传播到整个集群。 `ADDSLOTS` 命令通常在新创建集群时， 作为一种快速地将各个槽指派给各个节点的手段来使用。

`CLUSTER SETSLOT slot NODE node` 子命令可以将指定的槽 `slot` 指派给节点 `node` 。

至于 `CLUSTER SETSLOT slot MIGRATING node` 命令和 `CLUSTER SETSLOT slot IMPORTING node` 命令， 前者用于将给定节点 `node` 中的槽 `slot` 迁移出节点， 而后者用于将给定槽 `slot` 导入到节点 `node` ：

- 当一个槽被设置为 `MIGRATING` 状态时， 原来持有这个槽的节点仍然会继续接受关于这个槽的命令请求， 但只有命令所处理的键仍然存在于节点时， 节点才会处理这个命令请求。
  
  如果命令所使用的键不存在与该节点， 那么节点将向客户端返回一个 `-ASK` 转向（redirection）错误， 告知客户端， 要将命令请求发送到槽的迁移目标节点。

- 当一个槽被设置为 `IMPORTING` 状态时， 节点仅在接收到 `ASKING` 命令之后， 才会接受关于这个槽的命令请求。
  
  如果客户端没有向节点发送 `ASKING` 命令， 那么节点会使用 `-MOVED` 转向错误将命令请求转向至真正负责处理这个槽的节点。

上面关于 `MIGRATING` 和 `IMPORTING` 的说明有些难懂， 让我们用一个实际的实例来说明一下。

假设现在， 我们有 A 和 B 两个节点， 并且我们想将槽 `8` 从节点 A 移动到节点 B ， 于是我们：

- 向节点 B 发送命令 `CLUSTER SETSLOT 8 IMPORTING A`
- 向节点 A 发送命令 `CLUSTER SETSLOT 8 MIGRATING B`

每当客户端向其他节点发送关于哈希槽 `8` 的命令请求时， 这些节点都会向客户端返回指向节点 A 的转向信息：

- 如果命令要处理的键已经存在于槽 `8` 里面， 那么这个命令将由节点 A 处理。
- 如果命令要处理的键未存在于槽 `8` 里面（比如说，要向槽添加一个新的键）， 那么这个命令由节点 B 处理。

这种机制将使得节点 A 不再创建关于槽 `8` 的任何新键。

与此同时， 一个特殊的客户端 `redis-trib` 以及 Redis 集群配置程序（configuration utility）会将节点 A 中槽 `8` 里面的键移动到节点 B 。

键的移动操作由以下两个命令执行：

```
CLUSTER GETKEYSINSLOT slot count
```

上面的命令会让节点返回 `count` 个 `slot` 槽中的键， 对于命令所返回的每个键， `redis-trib` 都会向节点 A 发送一条 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令， 该命令会将所指定的键原子地（atomic）从节点 A 移动到节点 B （在移动键期间，两个节点都会处于阻塞状态，以免出现竞争条件）。

以下为 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令的运作原理：

```
MIGRATE target_host target_port key target_database id timeout
```

执行 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令的节点会连接到 `target` 节点， 并将序列化后的 `key` 数据发送给 `target` ， 一旦 `target` 返回 `OK` ， 节点就将自己的 `key` 从数据库中删除。

从一个外部客户端的视角来看， 在某个时间点上， 键 `key` 要么存在于节点 A ， 要么存在于节点 B ， 但不会同时存在于节点 A 和节点 B 。

因为 Redis 集群只使用 `0` 号数据库， 所以当 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令被用于执行集群操作时， `target_database` 的值总是 `0` 。

`target_database` 参数的存在是为了让 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令成为一个通用命令， 从而可以作用于集群以外的其他功能。

我们对 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 命令做了优化， 使得它即使在传输包含多个元素的列表键这样的复杂数据时， 也可以保持高效。

不过， 尽管 MIGRATE host port key destination-db timeout [COPY\] [REPLACE] 非常高效， 对一个键非常多、并且键的数据量非常大的集群来说， 集群重配置还是会占用大量的时间， 可能会导致集群没办法适应那些对于响应时间有严格要求的应用程序。

## ASK 转向

在之前介绍 `MOVED` 转向的时候， 我们说除了 `MOVED` 转向之外， 还有另一种 `ASK` 转向。

当节点需要让一个客户端长期地（permanently）将针对某个槽的命令请求发送至另一个节点时， 节点向客户端返回 `MOVED` 转向。

另一方面， 当节点需要让客户端仅仅在下一个命令请求中转向至另一个节点时， 节点向客户端返回 `ASK` 转向。

比如说， 在我们上一节列举的槽 `8` 的例子中， 因为槽 `8` 所包含的各个键分散在节点 A 和节点 B 中， 所以当客户端在节点 A 中没找到某个键时， 它应该转向到节点 B 中去寻找， 但是这种转向应该仅仅影响一次命令查询， 而不是让客户端每次都直接去查找节点 B ： 在节点 A 所持有的属于槽 `8` 的键没有全部被迁移到节点 B 之前， 客户端应该先访问节点 A ， 然后再访问节点 B 。

因为这种转向只针对 `16384` 个槽中的其中一个槽， 所以转向对集群造成的性能损耗属于可接受的范围。

因为上述原因， 如果我们要在查找节点 A 之后， 继续查找节点 B ， 那么客户端在向节点 B 发送命令请求之前， 应该先发送一个 `ASKING` 命令， 否则这个针对带有 `IMPORTING` 状态的槽的命令请求将被节点 B 拒绝执行。

接收到客户端 `ASKING` 命令的节点将为客户端设置一个一次性的标志（flag）， 使得客户端可以执行一次针对 `IMPORTING` 状态的槽的命令请求。

从客户端的角度来看， `ASK` 转向的完整语义（semantics）如下：

- 如果客户端接收到 `ASK` 转向， 那么将命令请求的发送对象调整为转向所指定的节点。
- 先发送一个 `ASKING` 命令，然后再发送真正的命令请求。
- 不必更新客户端所记录的槽 `8` 至节点的映射： 槽 `8` 应该仍然映射到节点 A ， 而不是节点 B 。

一旦节点 A 针对槽 `8` 的迁移工作完成， 节点 A 在再次收到针对槽 `8` 的命令请求时， 就会向客户端返回 `MOVED` 转向， 将关于槽 `8` 的命令请求长期地转向到节点 B 。

注意， 即使客户端出现 Bug ， 过早地将槽 `8` 映射到了节点 B 上面， 但只要这个客户端不发送 `ASKING` 命令， 客户端发送命令请求的时候就会遇上 `MOVED` 错误， 并将它转向回节点 A 。

## 容错

### 节点失效检测

以下是节点失效检查的实现方法：

- 当一个节点向另一个节点发送 [PING](#ping) 命令， 但是目标节点未能在给定的时限内返回 PING 命令的回复时， 那么发送命令的节点会将目标节点标记为 `PFAIL` （possible failure，可能已失效）。
  
  等待 PING 命令回复的时限称为“节点超时时限（node timeout）”， 是一个节点选项（node-wise setting）。

- 每次当节点对其他节点发送 PING 命令的时候， 它都会随机地广播三个它所知道的节点的信息， 这些信息里面的其中一项就是说明节点是否已经被标记为 `PFAIL` 或者 `FAIL` 。

- 当节点接收到其他节点发来的信息时， 它会记下那些被其他节点标记为失效的节点。 这称为失效报告（failure report）。

- 如果节点已经将某个节点标记为 `PFAIL` ， 并且根据节点所收到的失效报告显式， 集群中的大部分其他主节点也认为那个节点进入了失效状态， 那么节点会将那个失效节点的状态标记为 `FAIL` 。

- 一旦某个节点被标记为 `FAIL` ， 关于这个节点已失效的信息就会被广播到整个集群， 所有接收到这条信息的节点都会将失效节点标记为 `FAIL` 。

简单来说， 一个节点要将另一个节点标记为失效， 必须先询问其他节点的意见， 并且得到大部分主节点的同意才行。

因为过期的失效报告会被移除， 所以主节点要将某个节点标记为 `FAIL` 的话， 必须以最近接收到的失效报告作为根据。

在以下两种情况中， 节点的 `FAIL` 状态会被移除：

- 如果被标记为 `FAIL` 的是从节点， 那么当这个节点重新上线时， `FAIL` 标记就会被移除。
  
  保持（retaning）从节点的 `FAIL` 状态是没有意义的， 因为它不处理任何槽， 一个从节点是否处于 `FAIL` 状态， 决定了这个从节点在有需要时能否被提升为主节点。

- 如果一个主节点被打上 `FAIL` 标记之后， 经过了节点超时时限的四倍时间， 再加上十秒钟之后， 针对这个主节点的槽的故障转移操作仍未完成， 并且这个主节点已经重新上线的话， 那么移除对这个节点的 `FAIL` 标记。

在第二种情况中， 如果故障转移未能顺利完成， 并且主节点重新上线， 那么集群就继续使用原来的主节点， 从而免去管理员介入的必要。

### 集群状态检测（已部分实现）

每当集群发生配置变化时（可能是哈希槽更新，也可能是某个节点进入失效状态）， 集群中的每个节点都会对它所知道的节点进行扫描（scan）。

一旦配置处理完毕， 集群会进入以下两种状态的其中一种：

- `FAIL` ： 集群不能正常工作。 当集群中有某个节点进入失效状态时， 集群不能处理任何命令请求， 对于每个命令请求， 集群节点都返回错误回复。
- `OK` ： 集群可以正常工作， 负责处理全部 `16384` 个槽的节点中， 没有一个节点被标记为 `FAIL` 状态。

这说明即使集群中只有一部分哈希槽不能正常使用， 整个集群也会停止处理任何命令。

不过节点从出现问题到被标记为 `FAIL` 状态的这段时间里， 集群仍然会正常运作， 所以集群在某些时候， 仍然有可能只能处理针对 `16384` 个槽的其中一个子集的命令请求。

以下是集群进入 `FAIL` 状态的两种情况：

1. 至少有一个哈希槽不可用，因为负责处理这个槽的节点进入了 `FAIL` 状态。
2. 集群中的大部分主节点都进入下线状态。当大部分主节点都进入 `PFAIL` 状态时，集群也会进入 `FAIL` 状态。

第二个检查是必须的， 因为要将一个节点从 `PFAIL` 状态改变为 `FAIL` 状态， 必须要有大部分主节点进行投票表决， 但是， 当集群中的大部分主节点都进入失效状态时， 单凭一个两个节点是没有办法将一个节点标记为 `FAIL` 状态的。

因此， 有了第二个检查条件， 只要集群中的大部分主节点进入了下线状态， 那么集群就可以在不请求这些主节点的意见下， 将某个节点判断为 `FAIL` 状态， 从而让整个集群停止处理命令请求。

### 从节点选举

一旦某个主节点进入 `FAIL` 状态， 如果这个主节点有一个或多个从节点存在， 那么其中一个从节点会被升级为新的主节点， 而其他从节点则会开始对这个新的主节点进行复制。

新的主节点由已下线主节点属下的所有从节点中自行选举产生， 以下是选举的条件：

- 这个节点是已下线主节点的从节点。
- 已下线主节点负责处理的槽数量非空。
- 从节点的数据被认为是可靠的， 也即是， 主从节点之间的复制连接（replication link）的断线时长不能超过节点超时时限（node timeout）乘以 `REDIS_CLUSTER_SLAVE_VALIDITY_MULT` 常量得出的积。

如果一个从节点满足了以上的所有条件， 那么这个从节点将向集群中的其他主节点发送授权请求， 询问它们， 是否允许自己（从节点）升级为新的主节点。

如果发送授权请求的从节点满足以下属性， 那么主节点将向从节点返回 `FAILOVER_AUTH_GRANTED` 授权， 同意从节点的升级要求：

- 发送授权请求的是一个从节点， 并且它所属的主节点处于 `FAIL` 状态。
- 在已下线主节点的所有从节点中， 这个从节点的节点 ID 在排序中是最小的。
- 这个从节点处于正常的运行状态： 它没有被标记为 `FAIL` 状态， 也没有被标记为 `PFAIL` 状态。

一旦某个从节点在给定的时限内得到大部分主节点的授权， 它就会开始执行以下故障转移操作：

- 通过 `PONG` 数据包（packet）告知其他节点， 这个节点现在是主节点了。
- 通过 `PONG` 数据包告知其他节点， 这个节点是一个已升级的从节点（promoted slave）。
- 接管（claiming）所有由已下线主节点负责处理的哈希槽。
- 显式地向所有节点广播一个 `PONG` 数据包， 加速其他节点识别这个节点的进度， 而不是等待定时的 `PING` / `PONG` 数据包。

所有其他节点都会根据新的主节点对配置进行相应的更新，特别地：

- 所有被新的主节点接管的槽会被更新。
- 已下线主节点的所有从节点会察觉到 `PROMOTED` 标志， 并开始对新的主节点进行复制。
- 如果已下线的主节点重新回到上线状态， 那么它会察觉到 `PROMOTED` 标志， 并将自身调整为现任主节点的从节点。

在集群的生命周期中， 如果一个带有 `PROMOTED` 标识的主节点因为某些原因转变成了从节点， 那么该节点将丢失它所带有的 `PROMOTED` 标识。

## 发布/订阅（已实现，但仍然需要改善）

在一个 Redis 集群中， 客户端可以订阅任意一个节点， 也可以向任意一个节点发送信息， 节点会对客户端所发送的信息进行转发。

在目前的实现中， 节点会将接收到的信息广播至集群中的其他所有节点， 在将来的实现中， 可能会使用 bloom filter 或者其他算法来优化这一操作。

# 线程

讲讲Redis的线程模型？

Redis基于Reactor模式开发了网络事件处理器，这个处理器被称为文件事件处理器。它的组成结构为4部分：多个套接字、IO多路复用程序、文件事件分派器、事件处理器。因为文件事件分派器队列的消费是单线程的，所以Redis才叫单线程模型。

- 文件事件处理器使用I/O多路复用（multiplexing）程序来同时监听多个套接字， 并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接accept、read、write、close等操作时， 与操作相对应的文件事件就会产生， 这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

虽然文件事件处理器以单线程方式运行， 但通过使用 I/O 多路复用程序来监听多个套接字， 文件事件处理器既实现了高性能的网络通信模型， 又可以很好地与 redis 服务器中其他同样以单线程方式运行的模块进行对接， 这保持了 Redis 内部单线程设计的简单性。

Redis 为什么是单线程

官方 FAQ 表示，因为 Redis 是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。既然单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章地采用单线程的方案了，毕竟采用多线程会有很多麻烦。

# 持久化（persistence）

持久化就是把**内存的数据写到磁盘中**，防止服务宕机导致内存数据丢失。Redis支持两种方式的持久化，一种是`RDB`的方式，一种是`AOF`的方式。**前者会根据指定的规则定时将内存中的数据存储在硬盘上**，而**后者在每次执行完命令后将命令记录下来**。一般将两者结合使用。

## RDB

`RDB`是 Redis 默认的持久化方案。RDB持久化时会将内存中的数据写入到磁盘中，在指定目录下生成一个`dump.rdb`文件。Redis 重启会加载`dump.rdb`文件恢复数据。

`bgsave`是主流的触发 RDB 持久化的方式，执行过程如下：

- 执行`BGSAVE`命令
- Redis 父进程判断当前**是否存在正在执行的子进程**，如果存在，`BGSAVE`命令直接返回。
- 父进程执行`fork`操作**创建子进程**，fork操作过程中父进程会阻塞。
- 父进程`fork`完成后，**父进程继续接收并处理客户端的请求**，而**子进程开始将内存中的数据写进硬盘的临时文件**；
- 当子进程写完所有数据后会**用该临时文件替换旧的 RDB 文件**。

Redis启动时会读取RDB快照文件，将数据从硬盘载入内存。通过 RDB 方式的持久化，一旦Redis异常退出，就会丢失最近一次持久化以后更改的数据。

### 触发 RDB 持久化的方式

1. **手动触发**：用户执行`SAVE`或`BGSAVE`命令。`SAVE`命令执行快照的过程会阻塞所有客户端的请求，应避免在生产环境使用此命令。`BGSAVE`命令可以在后台异步进行快照操作，快照的同时服务器还可以继续响应客户端的请求，因此需要手动执行快照时推荐使用`BGSAVE`命令。

2. **被动触发**：
   
   - 根据配置规则进行自动快照，如`SAVE 100 10`，100秒内至少有10个键被修改则进行快照。
   - 如果从节点执行全量复制操作，主节点会自动执行`BGSAVE`生成 RDB 文件并发送给从节点。
   - 默认情况下执行`shutdown`命令时，如果没有开启 AOF 持久化功能则自动执行·BGSAVE·。

### 快照的运作方式

当 Redis 需要保存 `dump.rdb` 文件时， 服务器执行以下操作：

1. Redis 调用 `fork()` ，同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

### RDB 快照

在默认情况下， Redis 将数据库快照保存在名字为 `dump.rdb` 的二进制文件中。

你可以对 Redis 进行设置， 让它在“ `N` 秒内数据集至少有 `M` 个改动”这一条件被满足时， 自动保存一次数据集。

你也可以通过调用 [SAVE](#save) 或者 [BGSAVE](#bgsave) ， 手动让 Redis 进行数据集保存操作。

比如说， 以下设置会让 Redis 在满足“ `60` 秒内有至少有 `1000` 个键被改动”这一条件时， 自动保存一次数据集：

```
save 60 1000
```

这种持久化方式被称为快照（snapshot）。

### RDB 的优点

- RDB 是一个非常紧凑（compact）的文件，它保存了 Redis 在某个时间点上的数据集。 这种文件非常适合用于进行备份： 比如说，你可以在最近的 24 小时内，每小时备份一次 RDB 文件，并且在每个月的每一天，也备份一个 RDB 文件。 这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。
- RDB 非常适用于灾难恢复（disaster recovery）：它只有一个文件，并且内容都非常紧凑，可以（在加密后）将它传送到别的数据中心，或者亚马逊 S3 中。
- RDB 可以最大化 Redis 的性能：父进程在保存 RDB 文件时唯一要做的就是 `fork` 出一个子进程，然后这个子进程就会处理接下来的所有保存工作，父进程无须执行任何磁盘 I/O 操作。
- RDB 在恢复大数据集时的速度比 AOF 的恢复速度要快。

**优点**：

1. **Redis 加载 RDB 恢复数据远远快于 AOF 的方式**。
2. 使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，**保证了 Redis 的高性能**。

### RDB 的缺点

- 如果你需要尽量避免在服务器故障时丢失数据，那么 RDB 不适合你。 虽然 Redis 允许你设置不同的保存点（save point）来控制保存 RDB 文件的频率， 但是， 因为RDB 文件需要保存整个数据集的状态， 所以它并不是一个轻松的操作。 因此你可能会至少 5 分钟才保存一次 RDB 文件。 在这种情况下， 一旦发生故障停机， 你就可能会丢失好几分钟的数据。
- 每次保存 RDB 的时候，Redis 都要 `fork()` 出一个子进程，并由子进程来进行实际的持久化工作。 在数据集比较庞大时， `fork()` 可能会非常耗时，造成服务器在某某毫秒内停止处理客户端； 如果数据集非常巨大，并且 CPU 时间非常紧张的话，那么这种停止时间甚至可能会长达整整一秒。 虽然 AOF 重写也需要进行 `fork()` ，但无论 AOF 重写的执行间隔有多长，数据的耐久性都不会有任何损失。

**缺点**：

1. **RDB方式数据无法做到实时持久化**。因为`BGSAVE`每次运行都要执行`fork`操作创建子进程，属于重量级操作，频繁执行成本比较高。
2. RDB 文件使用特定二进制格式保存，Redis 版本升级过程中有多个格式的 RDB 版本，**存在老版本 Redis 无法兼容新版 RDB 格式的问题**。

## AOF方式

AOF（append only file）持久化：以独立日志的方式记录每次写命令，Redis重启时会重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是**解决了数据持久化的实时性**，AOF 是Redis持久化的主流方式。

默认情况下Redis没有开启AOF方式的持久化，可以通过`appendonly`参数启用：`appendonly yes`。开启AOF方式持久化后每执行一条写命令，Redis就会将该命令写进`aof_buf`缓冲区，AOF缓冲区根据对应的策略向硬盘做同步操作。

默认情况下系统**每30秒**会执行一次同步操作。为了防止缓冲区数据丢失，可以在Redis写入AOF文件后主动要求系统将缓冲区数据同步到硬盘上。可以通过`appendfsync`参数设置同步的时机。

```
appendfsync always //每次写入aof文件都会执行同步，最安全最慢，不建议配置
appendfsync everysec  //既保证性能也保证安全，建议配置
appendfsync no //由操作系统决定何时进行同步操作
```

接下来看一下 AOF 持久化执行流程：

1. 所有的写入命令会追加到 AOP 缓冲区中。
2. AOF 缓冲区根据对应的策略向硬盘同步。
3. 随着 AOF 文件越来越大，需要定期对 AOF 文件进行重写，达到压缩文件体积的目的。AOF文件重写是把Redis进程内的数据转化为写命令同步到新AOF文件的过程。
4. 当 Redis 服务器重启时，可以加载 AOF 文件进行数据恢复。

### 只进行追加操作的文件（append-only file，AOF）

快照功能并不是非常耐久（durable）： 如果 Redis 因为某些原因而造成故障停机， 那么服务器将丢失最近写入、且仍未保存到快照中的那些数据。

尽管对于某些程序来说， 数据的耐久性并不是最重要的考虑因素， 但是对于那些追求完全耐久能力（full durability）的程序来说， 快照功能就不太适用了。

从 1.1 版本开始， Redis 增加了一种完全耐久的持久化方式： AOF 持久化。

你可以通过修改配置文件来打开 AOF 功能：

```
appendonly yes
```

从现在开始， 每当 Redis 执行一个改变数据集的命令时（比如 SET key value [EX seconds\] [PX milliseconds] [NX|XX]）， 这个命令就会被追加到 AOF 文件的末尾。

这样的话， 当 Redis 重新启时， 程序就可以通过重新执行 AOF 文件中的命令来达到重建数据集的目的。

### AOF 重写

因为 AOF 的运作方式是不断地将命令追加到文件的末尾， 所以随着写入命令的不断增加， AOF 文件的体积也会变得越来越大。

举个例子， 如果你对一个计数器调用了 100 次 INCR key ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100 条记录（entry）。

然而在实际上， 只使用一条 SET key value [EX seconds\] [PX milliseconds] [NX|XX] 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。

为了处理这种情况， Redis 支持一种有趣的特性： 可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。

执行 BGREWRITEAOF 命令， Redis 将生成一个新的 AOF 文件， 这个文件包含重建当前数据集所需的最少命令。

Redis 2.2 需要自己手动执行 BGREWRITEAOF 命令； Redis 2.4 则可以自动触发 AOF 重写， 具体信息请查看 2.4 的示例配置文件。

### AOF 的运作方式

AOF 重写和 RDB 创建快照一样，都巧妙地利用了写时复制机制。

以下是 AOF 重写的执行步骤：

1. Redis 执行 `fork()` ，现在同时拥有父进程和子进程。
2. 子进程开始将新 AOF 文件的内容写入到临时文件。
3. 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾： 这样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。
4. 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
5. 搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

### AOF 的优点

- 使用 AOF 持久化会让 Redis 变得非常耐久（much more durable）：你可以设置不同的 `fsync` 策略，比如无 `fsync` ，每秒钟一次 `fsync` ，或者每次执行写入命令时 `fsync` 。 AOF 的默认策略为每秒钟 `fsync` 一次，在这种配置下，Redis 仍然可以保持良好的性能，并且就算发生故障停机，也最多只会丢失一秒钟的数据（ `fsync` 会在后台线程执行，所以主线程可以继续努力地处理命令请求）。
- AOF 文件是一个只进行追加操作的日志文件（append only log）， 因此对 AOF 文件的写入不需要进行 `seek` ， 即使日志因为某些原因而包含了未写入完整的命令（比如写入时磁盘已满，写入中途停机，等等）， `redis-check-aof` 工具也可以轻易地修复这种问题。
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 [FLUSHALL](#flushall) 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 [FLUSHALL](#flushall) 命令， 并重启 Redis ， 就可以将数据集恢复到 [FLUSHALL](#flushall) 执行之前的状态。

**优点**：

1. AOF可以更好的保护数据不丢失，可以配置 AOF 每秒执行一次`fsync`操作，如果Redis进程挂掉，最多丢失1秒的数据。
2. AOF以`append-only`的模式写入，所以没有磁盘寻址的开销，写入性能非常高。

### AOF 的缺点

- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。
- 根据所使用的 `fsync` 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 `fsync` 的性能依然非常高， 而关闭 `fsync` 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。
- AOF 在过去曾经发生过这样的 bug ： 因为个别命令的原因，导致 AOF 文件在重新载入时，无法将数据集恢复成保存时的原样。 （举个例子，阻塞命令 BRPOPLPUSH source destination timeout 就曾经引起过这样的 bug 。） 测试套件里为这种情况添加了测试： 它们会自动生成随机的、复杂的数据集， 并通过重新载入这些数据来确保一切正常。 虽然这种 bug 在 AOF 文件中并不常见， 但是对比来说， RDB 几乎是不可能出现这种 bug 的。
  **缺点**：
1. 对于同一份文件AOF文件比RDB数据快照要大。
2. 数据恢复比较慢。

## RDB和AOF如何选择？

通常来说，应该同时使用两种持久化方案，以保证数据安全。

- 如果数据不敏感，且可以从其他地方重新生成，可以关闭持久化。
- 如果数据比较重要，且能够承受几分钟的数据丢失，比如缓存等，只需要使用RDB即可。
- 如果是用做内存数据，要使用Redis的持久化，建议是RDB和AOF都开启。
- 如果只用AOF，优先使用everysec的配置选择，因为它在可靠性和性能之间取了一个平衡。

当RDB与AOF两种方式都开启时，Redis会优先使用AOF恢复数据，因为AOF保存的文件比RDB文件更完整。

Redis 提供了多种不同级别的持久化方式：

- RDB 持久化可以在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）。
- AOF 持久化记录服务器执行的所有写操作命令，并在服务器启动时，通过重新执行这些命令来还原数据集。 AOF 文件中的命令全部以 Redis 协议的格式来保存，新命令会被追加到文件的末尾。 Redis 还可以在后台对 AOF 文件进行重写（rewrite），使得 AOF 文件的体积不会超出保存数据集状态所需的实际大小。
- Redis 还可以同时使用 AOF 持久化和 RDB 持久化。 在这种情况下， 当 Redis 重启时， 它会优先使用 AOF 文件来还原数据集， 因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。
- 你甚至可以关闭持久化功能，让数据只在服务器运行时存在。

了解 RDB 持久化和 AOF 持久化之间的异同是非常重要的， 以下几个小节将详细地介绍这这两种持久化功能， 并对它们的相同和不同之处进行说明。

一般来说， 如果想达到足以媲美 PostgreSQL 的数据安全性， 你应该同时使用两种持久化功能。

如果你非常关心你的数据， 但仍然可以承受数分钟以内的数据丢失， 那么你可以只使用 RDB 持久化。

有很多用户都只使用 AOF 持久化， 但我们并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快， 除此之外， 使用 RDB 还可以避免之前提到的 AOF 程序的 bug 。

Note

因为以上提到的种种原因， 未来我们可能会将 AOF 和 RDB 整合成单个持久化模型。 （这是一个长期计划。）

接下来的几个小节将介绍 RDB 和 AOF 的更多细节。

## RDB 持久化切换 AOF 持久化

在 Redis 2.2 或以上版本，可以在不重启的情况下，从 RDB 切换到 AOF ：

1. 为最新的 `dump.rdb` 文件创建一个备份。
2. 将备份放到一个安全的地方。
3. 执行以下两条命令：

> ```
> redis-cli> CONFIG SET appendonly yes
> 
> redis-cli> CONFIG SET save ""
> ```

1. 确保命令执行之后，数据库的键的数量没有改变。
2. 确保写命令会被正确地追加到 AOF 文件的末尾。

步骤 3 执行的第一条命令开启了 AOF 功能： Redis 会阻塞直到初始 AOF 文件创建完成为止， 之后 Redis 会继续处理命令请求， 并开始将写入命令追加到 AOF 文件末尾。

步骤 3 执行的第二条命令用于关闭 RDB 功能。 这一步是可选的， 如果你愿意的话， 也可以同时使用 RDB 和 AOF 这两种持久化功能。

Note

别忘了在 `redis.conf` 中打开 AOF 功能！ 否则的话， 服务器重启之后， 之前通过 `CONFIG SET` 设置的配置就会被遗忘， 程序会按原来的配置来启动服务器。

Note

译注： 原文这里还有介绍 2.0 版本的切换方式， 考虑到 2.0 已经很老旧了， 这里省略了对那部分文档的翻译， 有需要的请参考原文。

## RDB 和 AOF 之间的相互作用

在版本号大于等于 2.4 的 Redis 中， BGSAVE 执行的过程中， 不可以执行 BGREWRITEAOF 。 反过来说， 在 BGREWRITEAOF 执行的过程中， 也不可以执行 BGSAVE 。

这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。

如果 BGSAVE 正在执行， 并且用户显示地调用 BGREWRITEAOF 命令， 那么服务器将向用户回复一个 `OK` 状态， 并告知用户， BGREWRITEAOF 已经被预定执行： 一旦 BGSAVE 执行完毕， BGREWRITEAOF 就会正式开始。

当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

## 备份 Redis 数据

在阅读这个小节前， 先将下面这句话铭记于心： 一定要备份你的数据库！

磁盘故障， 节点失效， 诸如此类的问题都可能让你的数据消失不见， 不进行备份是非常危险的。

Redis 对于数据备份是非常友好的， 因为你可以在服务器运行的时候对 RDB 文件进行复制： RDB 文件一旦被创建， 就不会进行任何修改。 当服务器要创建一个新的 RDB 文件时， 它先将文件的内容保存在一个临时文件里面， 当临时文件写入完毕时， 程序才使用 `rename(2)` 原子地用临时文件替换原来的 RDB 文件。

这也就是说， 无论何时， 复制 RDB 文件都是绝对安全的。

以下是我们的建议：

- 创建一个定期任务（cron job）， 每小时将一个 RDB 文件备份到一个文件夹， 并且每天将一个 RDB 文件备份到另一个文件夹。
- 确保快照的备份都带有相应的日期和时间信息， 每次执行定期任务脚本时， 使用 `find` 命令来删除过期的快照： 比如说， 你可以保留最近 48 小时内的每小时快照， 还可以保留最近一两个月的每日快照。
- 至少每天一次， 将 RDB 备份到你的数据中心之外， 或者至少是备份到你运行 Redis 服务器的物理机器之外。

## 容灾备份

Redis 的容灾备份基本上就是对数据进行备份， 并将这些备份传送到多个不同的外部数据中心。

容灾备份可以在 Redis 运行并产生快照的主数据中心发生严重的问题时， 仍然让数据处于安全状态。

因为很多 Redis 用户都是创业者， 他们没有大把大把的钱可以浪费， 所以下面介绍的都是一些实用又便宜的容灾备份方法：

- Amazon S3 ，以及其他类似 S3 的服务，是一个构建灾难备份系统的好地方。 最简单的方法就是将你的每小时或者每日 RDB 备份加密并传送到 S3 。 对数据的加密可以通过 `gpg -c` 命令来完成（对称加密模式）。 记得把你的密码放到几个不同的、安全的地方去（比如你可以把密码复制给你组织里最重要的人物）。 同时使用多个储存服务来保存数据文件，可以提升数据的安全性。
- 传送快照可以使用 SCP 来完成（SSH 的组件）。 以下是简单并且安全的传送方法： 买一个离你的数据中心非常远的 VPS ， 装上 SSH ， 创建一个无口令的 SSH 客户端 key ， 并将这个 key 添加到 VPS 的 authorized_keys 文件中， 这样就可以向这个 VPS 传送快照备份文件了。 为了达到最好的数据安全性，至少要从两个不同的提供商那里各购买一个 VPS 来进行数据容灾备份。

需要注意的是， 这类容灾系统如果没有小心地进行处理的话， 是很容易失效的。

最低限度下， 你应该在文件传送完毕之后， 检查所传送备份文件的体积和原始快照文件的体积是否相同。 如果你使用的是 VPS ， 那么还可以通过比对文件的 SHA1 校验和来确认文件是否传送完整。

另外， 你还需要一个独立的警报系统， 让它在负责传送备份文件的传送器（transfer）失灵时通知你。

# 发布与订阅（pub/sub）

SUBSCRIBE channel channel …\] 、 [UNSUBSCRIBE [channel [channel …\]] 和 PUBLISH channel message 三个命令实现了[发布与订阅信息泛型](http://en.wikipedia.org/wiki/Publish/subscribe)（Publish/Subscribe messaging paradigm）， 在这个实现中， 发送者（发送信息的客户端）不是将信息直接发送给特定的接收者（接收信息的客户端）， 而是将信息发送给频道（channel）， 然后由频道将信息转发给所有对这个频道感兴趣的订阅者。

发送者无须知道任何关于订阅者的信息， 而订阅者也无须知道是那个客户端给它发送信息， 它只要关注自己感兴趣的频道即可。

对发布者和订阅者进行解构（decoupling）， 可以极大地提高系统的扩展性（scalability）， 并得到一个更动态的网络拓扑（network topology）。

比如说， 要订阅频道 `foo` 和 `bar` ， 客户端可以使用频道名字作为参数来调用 [SUBSCRIBE channel channel …\] 命令：

```
redis> SUBSCRIBE foo bar
```

当有客户端发送信息到这些频道时， Redis 会将传入的信息推送到所有订阅这些频道的客户端里面。

正在订阅频道的客户端不应该发送除 SUBSCRIBE channel channel …\] 和 UNSUBSCRIBE [channel [channel …\]] 之外的其他命令。 其中， [SUBSCRIBE channel channel …\] 可以用于订阅更多频道， 而 [UNSUBSCRIBE [channel [channel …\]] 则可以用于退订已订阅的一个或多个频道。

SUBSCRIBE channel channel …\] 和 UNSUBSCRIBE [channel [channel …\]] 的执行结果会以信息的形式返回， 客户端可以通过分析所接收信息的第一个元素， 从而判断所收到的内容是一条真正的信息， 还是 [SUBSCRIBE channel channel …\] 或 [UNSUBSCRIBE [channel [channel …\]] 命令的操作结果。

## 信息的格式

频道转发的每条信息都是一条带有三个元素的多条批量回复（multi-bulk reply）。

信息的第一个元素标识了信息的类型：

- `subscribe` ： 表示当前客户端成功地订阅了信息第二个元素所指示的频道。 而信息的第三个元素则记录了目前客户端已订阅频道的总数。
- `unsubscribe` ： 表示当前客户端成功地退订了信息第二个元素所指示的频道。 信息的第三个元素记录了客户端目前仍在订阅的频道数量。 当客户端订阅的频道数量降为 `0` 时， 客户端不再订阅任何频道， 它可以像往常一样， 执行任何 Redis 命令。
- `message` ： 表示这条信息是由某个客户端执行 PUBLISH channel message 命令所发送的， 真正的信息。 信息的第二个元素是信息来源的频道， 而第三个元素则是信息的内容。

举个例子， 如果客户端执行以下命令：

```
redis> SUBSCRIBE first second
```

那么它将收到以下回复：

```
1) "subscribe"
2) "first"
3) (integer) 1

1) "subscribe"
2) "second"
3) (integer) 2
```

如果在这时， 另一个客户端执行以下 PUBLISH channel message 命令：

```
redis> PUBLISH second Hello
```

那么之前订阅了 `second` 频道的客户端将收到以下信息：

```
1) "message"
2) "second"
3) "hello"
```

当订阅者决定退订所有频道时， 它可以执行一个无参数的 UNSUBSCRIBE [channel [channel …\]] 命令：

```
redis> UNSUBSCRIBE
```

这个命令将接到以下回复：

```
1) "unsubscribe"
2) "second"
3) (integer) 1

1) "unsubscribe"
2) "first"
3) (integer) 0
```

## 订阅模式

Redis 的发布与订阅实现支持模式匹配（pattern matching）： 客户端可以订阅一个带 `*` 号的模式， 如果某个/某些频道的名字和这个模式匹配， 那么当有信息发送给这个/这些频道的时候， 客户端也会收到这个/这些频道的信息。

比如说，执行命令

```
redis> PSUBSCRIBE news.*
```

的客户端将收到来自 `news.art.figurative` 、 `news.music.jazz` 等频道的信息。

客户端订阅的模式里面可以包含多个 glob 风格的通配符， 比如 `*` 、 `?` 和 `[...]` ， 等等。

执行命令

```
redis> PUNSUBSCRIBE news.*
```

将退订 `news.*` 模式， 其他已订阅的模式不会被影响。

通过订阅模式接收到的信息， 和通过订阅频道接收到的信息， 这两者的格式不太一样：

- 通过订阅模式而接收到的信息的类型为 `pmessage` ： 这代表有某个客户端通过 PUBLISH channel message 向某个频道发送了信息， 而这个频道刚好匹配了当前客户端所订阅的某个模式。 信息的第二个元素记录了被匹配的模式， 第三个元素记录了被匹配的频道的名字， 最后一个元素则记录了信息的实际内容。

客户端处理 PSUBSCRIBE pattern pattern …\] 和 PUNSUBSCRIBE pattern [pattern …\]] 返回值的方式， 和客户端处理 [SUBSCRIBE channel channel …\] 和 [UNSUBSCRIBE [channel [channel …\]] 的方式类似： 通过对信息的第一个元素进行分析， 客户端可以判断接收到的信息是一个真正的信息， 还是 [PSUBSCRIBE pattern pattern …\] 或 [PUNSUBSCRIBE [pattern [pattern …\]] 命令的返回值。

## 通过频道和模式接收同一条信息

如果客户端订阅的多个模式匹配了同一个频道， 或者客户端同时订阅了某个频道、以及匹配这个频道的某个模式， 那么它可能会多次接收到同一条信息。

举个例子， 如果客户端执行了以下命令：

```
SUBSCRIBE foo
PSUBSCRIBE f*
```

那么当有信息发送到频道 `foo` 时， 客户端将收到两条信息： 一条来自频道 `foo` ，信息类型为 `message` ； 另一条来自模式 `f*` ，信息类型为 `pmessage` 。

## 订阅总数

在执行 SUBSCRIBE channel channel …\] 、 UNSUBSCRIBE [channel [channel …\]] 、 [PSUBSCRIBE pattern pattern …\] 和 [PUNSUBSCRIBE [pattern [pattern …\]] 命令时， 返回结果的最后一个元素是客户端目前仍在订阅的频道和模式总数。

当客户端退订所有频道和模式， 也即是这个总数值下降为 `0` 的时候， 客户端将退出订阅与发布状态。

## 编程示例

Pieter Noordhuis 提供了一个使用 EventMachine 和 Redis 编写的 [高性能多用户网页聊天软件](https://gist.github.com/348262) ， 这个软件很好地展示了发布与订阅功能的用法。

## 客户端库实现提示

因为所有接收到的信息都会包含一个信息来源：

- 当信息来自频道时，来源是某个频道；
- 当信息来自模式时，来源是某个模式。

因此， 客户端可以用一个哈希表， 将特定来源和处理该来源的回调函数关联起来。 当有新信息到达时， 程序就可以根据信息的来源， 在 O(1) 复杂度内， 将信息交给正确的回调函数来处理。

# Redis 架构模式

### 单机版

特点：简单

问题：

1、内存容量有限 2、处理能力有限 3、无法高可用。

### 主从复制

Redis 的复制（replication）功能允许用户根据一个 Redis 服务器来创建任意多个该服务器的复制品，其中被复制的服务器为主服务器（master），而通过复制创建出来的服务器复制品则为从服务器（slave）。 只要主从服务器之间的网络连接正常，主从服务器两者会具有相同的数据，主服务器就会一直将发生在自己身上的数据更新同步 给从服务器，从而一直保证主从服务器的数据相同。

特点：

1、master/slave 角色

2、master/slave 数据相同

3、降低 master 读压力在转交从库

问题：

无法保证高可用

没有解决 master 写的压力

### 哨兵

Redis sentinel 是一个分布式系统中监控 redis 主从服务器，并在主服务器下线时自动进行故障转移。其中三个特性：

监控（Monitoring）：  Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。

提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作。

特点：

1、保证高可用

2、监控各个节点

3、自动故障迁移

缺点：主从模式，切换需要时间丢数据

没有解决 master 写的压力

### 集群（proxy 型）

Twemproxy 是一个 Twitter 开源的一个 redis 和 memcache 快速/轻量级代理服务器； Twemproxy 是一个快速的单线程代理程序，支持 Memcached ASCII 协议和 redis 协议。

特点：1、多种 hash 算法：MD5、CRC16、CRC32、CRC32a、hsieh、murmur、Jenkins

2、支持失败节点自动删除

3、后端 Sharding 分片逻辑对业务透明，业务方的读写方式和操作单个 Redis 一致

缺点：增加了新的 proxy，需要维护其高可用。

failover 逻辑需要自己实现，其本身不能支持故障的自动转移可扩展性差，进行扩缩容都需要手动干预

### 集群（直连型）

从redis 3.0之后版本支持redis-cluster集群，Redis-Cluster采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

特点：

1、无中心架构（不存在哪个节点影响性能瓶颈），少了 proxy 层。

2、数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布。

3、可扩展性，可线性扩展到 1000 个节点，节点可动态添加或删除。

4、高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做备份数据副本

5、实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave到 Master 的角色提升。

缺点：

1、资源隔离性较差，容易出现相互影响的情况。

2、数据通过异步复制,不保证数据的强一致性

# Sentinel

Sentinel（哨岗、哨兵）是Redis的高可用性（high avail-ability）解决方案：由一个或多个Sentinel实例（instance）组成的Sentinel系统（system）对主从服务器进行监视。<!--more-->

所谓Sentinel['sentɪnl]的高可用性指**系统无中断地执行其功能的能力**。Sentinel的主要功能是：

1. 监控Redis整体是否正常运行。
2. 某个节点出问题时，**通知给其他进程**（比如他的客户端）。
3. 主服务器下线时，在从服务器中**选举**出一个新的主服务器。

# 1. 启动与初始化

启动命令：

```
$ redis-sentinel /path/to/your/sentinel.conf
$ redis-server /path/to/your/sentinel.conf --sentinel
```

一个Sentinel启动时，需要执行以下步骤：

1. 初始化服务器
2. 将普通Redis服务器使用的代码替换成Sentinel专用代码。
3. 初始化Sentinel状态
4. 根据给定的配置文件，初始化Sentinel的监视主服务器列表
5. 创建连向主服务器的网络连接

**（1）初始化服务器**

Sentinel**本质上只是一个运行在特殊模式下的Redis服务器**，所以启动Sentinel的第一步，就是初始化一个普通的Redis服务器。

由于Sentinel执行的工作和普通的Redis服务器执行的工作不同，所以初始化也不相同。首先先来回忆一下普通服务器的初始化过程：

1. 初始化服务器状态结构
2. 载入配置选项
3. 初始化服务器数据结构
4. 还原数据库状态
5. 执行事件循环

而Sentinel初始化时，并不会使用AOF和RDB来还原数据库，此外很多命令比如WATCH,EVAL等都不会使用。

**（2）使用Sentinel专用代码**

主要分两部分：**端口**和**命令集**

普通Redis服务器使用`redis.h/REDIS_SERVERPORT`常量的值作为服务器端口：

```C
#define REDIS_SERVERPORT 6379
```

而Sentinel则使用`sentinel.c/REDIS_SENTINEL_POR`T常量的值作为服务器端口：

```C
#define REDIS_SENTINEL_PORT 26379
```

普通Redis服务器使用`redis.c/redisCom-mandTable`作为服务器的命令表。而Sentinel则使用`sentinel.c/sentinelcmds`作为服务器的命令表。

`sentinelcmds`命令表也解释了为什么在Sentinel模式下，Redis服务器不能执行诸如SET、DBSIZE、EVAL等等这些命令，因为服务器**根本没有在命令表中载入这些命令**。PING、SEN-TINEL、INFO、SUBSCRIBE、UNSUBSCRIBE、PSUBSCRIBE和PUNSUBSCRIBE这七个命令就是客户端可以对Sentinel执行的全部命令了。

**（3）初始化Sentinel状态**

在应用了Sentinel的专用代码之后，接下来，服务器会初始化一个`sentinel.c/sentinelState`结构，也就是Sentinel状态。服务器的一般状态仍然由`redis.h/redisServer`结构保存。

**（4）初始化Sentinel状态的masters属性**

Sentinel状态中的masters字典记录了所有被Sentinel监视的主服务器的相关信息，其中：

- 字典的**键**是**被监视主服务器**的名字。
- 而字典的**值**则是**被监视主服务器**对应的`sentinel.c/sentinelRedisInstance`结构。

每个`sentinelRedisInstance`结构（后面简称“实例结构”）**代表一个被Sentinel监视的Redis服务器实例（instance）**，这个实例可以是主服务器、从服务器，或者另外一个Sentinel。

对Sentinel状态的初始化将引发对masters字典的初始化，而masters字典的初始化是根据被载入的Sentinel配置文件来进行的。

**（5）创建连向主服务器的网络连接**

Sentinel会创建两个连向主服务器的异步网络连接：

- 一个是命令连接，这个连接专门用于向主服务器发送命令，并接收命令回复。
- 另一个是订阅连接，这个连接专门用于订阅主服务器的`__sentinel__:hello`频道。

Redis目前的发布与订阅功能中，被发送的信息都不会保存在Redis服务器里面，如果在信息发送时，想要接收信息的客户端不在线或者断线，那么这个客户端就会丢失这条信息。因此，为了不丢失`__sentinel__:hello`频道的任何信息，Sentinel必须专门用一个订阅连接来接收该频道的信息。

因为Sentinel需要与多个实例创建多个网络连接，所以Sentinel使用的是异步连接。

# 2. 获取服务器信息

## 2.1 获取主服务器信息

Sentinel默认会以每十秒一次的频率，通过命令连接向被监视的**主服务器发送INFO命令**，并通过分析INFO命令的回复来获取主服务器的当前信息。

通过分析主服务器返回的INFO命令回复，Sentinel可以明确：主服务器本身的信息，从服务器的信息。之后进行更新：

- 根据run_id域和role域记录的信息，**更新Sentinel主服务器的实例结构**。
- 返回的从服务器信息，则会被用于**更新主服务器实例结构的slaves字典**，这个字典记录了主服务器属下从服务器的名单。

## 2.2 获取从服务器信息

当Sentinel发现主服务器有新的从服务器出现时，Sentinel除了会为这个新的从服务器创建相应的实例结构之外，Sentinel还会创建连接到从服务器的命令连接和订阅连接。

在创建命令连接之后，Sentinel在默认情况下，会以每十秒一次的频率通过命令连接向**从服务器发送INFO命令**，并获得类似于以下内容的回复：

利用这些信息更新从服务器的实例结构：

## 2.3 接收主从服务器的频道

当Sentinel与一个主服务器或者从服务器建立起订阅连接之后，Sentinel就会通过订阅连接，向服务器发送以下命令：

```C
SUBSCRIBE __sentinel__:hello
```

Sentinel对`__sentinel__:hello`频道的订阅会一直持续到Sentinel与服务器的连接断开为止。

对于每个与Sentinel连接的服务器，Sentinel既通过命令连接向服务器的`__sentinel__:hello`频道发送信息，又通过订阅连接从服务器的`__sentinel__:hello`频道接收信息

假设现在有sentinel1、sentinel2、sentinel3三个Sentinel在监视同一个服务器，那么当sentinel1向服务器的`__sentinel__:hello`频道发送一条信息时，所有订阅了`__sen-tinel__:hello`频道的Sentinel都会收到这条信息。

# 3. Sentinel互相监督

## 3.1 Sentinel字典

Sentinel为主服务器创建的实例结构中的**sentinels字典**保存了除Sentinel本身之外，所有同样**监视这个主服务器的其他Sentinel的资料。**

当一个Sentinel接收到其他Sentinel发来的信息时，会提取出两方面信息：

- **与Sentinel有关的参数**：源Sentinel的IP，port，ID和配置参数
- **与主服务器有关的参数**：源Sentinel正在监视的主服务器的名字，IP，端口号和配置参数。

提取以后，Sentinel会在自己的Sentinel状态的masters字典中查找相应的主服务器实例结构，**检查源Sentinel的实例结构是否存在：**

- 存在，则更新
- 不存在，则创建

Sentinel可以通过分析接收到的频道信息来获知其他Sentinel的存在，并通过发送频道信息来让其他Sentinel知道自己的存在**，监视同一个主服务器的多个Sentinel可以自动发现对方**。

## 3.2 连接其他Sentinel

当Sentinel通过频道信息发现一个新的Sentinel时，它不仅会为新Sentinel在sentinels字典中创建相应的实例结构**，还会创建一个连向新Sentinel的命令连接**，而**新Sentinel也同样会创建连向这个Sentinel的命令连接**，最终监视同一主服务器的多个Sentinel将形成相互连接的**环形网络**。

注意：Sentinel之间**只会创建命令连接，但不会创建订阅**。Sentinel需要通过接收主服务器或者从服务器发来的频道信息来发现未知的新Sentinel，所以才需要建立订阅连接。相互已知的Sentinel只要使用命令连接来进行通信就足够了。

# 4. 检测下线状态

## 4.1 检测主观下线状态

Sentinel会以每秒一次的频率向所有与它创建了命令连接的实例（包括**主服务器、从服务器、其他Sentinel在内**）发送PING命令，并通过实例返回的PING命令回复来**判断对方是否在线。**

当对方超过一段时间不向Sentinel回复时（比如超时5000毫秒）则Sentinel1就会**将对方标记为主观下线**。

## 4.2 检查客观下线状态

当Sentinel将一个主服务器判断为主观下线之后，为了确认这个主服务器是否真的下线了，它会**向同样监视这一主服务器的其他Sentinel进行询问，看它们是否也认为主服务器已经进入了下线状态**。当Sentinel从其他Sentinel那里接收到足够数量的已下线判断之后，Sentinel就会将从服务器**判定为客观下线**，并对主服务器执行故障转移操作。

# 5. 下线后的补救

## 5.1 选举Sentinel领袖

一个主服务器被判断为客观下线时，监视这个下线主服务器的各个Sentinel会进行协商，**选举出一个领头Sentinel，并由领头Sentinel对下线主服务器执行故障转移操作。**

选举策略：

- **候选人**：所有在线的Sentinel
- **选举过程**：一个Sentinel向另一个Sentinel发送设置请求`SENTINEL is-master-down-by-addr`命令
- **胜选条件**：
  - **局部领头Sentinel：先到先得**，最先向目标Sentinel发送设置要求的源Sentinel将成为目标Sen-tinel的局部领头Sentinel，而之后接收到的所有设置要求都会被目标Sentinel拒绝。
  - **领头Sentinel：**如果有某个Sentinel**被半数**以上的Sentinel设置成了局部领头Sentinel，那么这个Sentinel成为领头Sentinel。
- **重选条件：**如果没有过半，则再次投票，知道选出过半的为止。

## 5.1 故障转移

在选举产生出领头Sentinel之后，领头Sentinel将对已下线的主服务器执行故障转移操作，该操作包含以下三个步骤：

1. 从已下线的主服务器的从服务器中**拔举**一个作为主服务器。
2. 让已下线主服务器属下的所有从服务器改为复制新的主服务器
3. 将已下线主服务器设置为新的主服务器的从服务器，当这个旧的主服务器重新上线时，它就会成为新的主服务器的从服务器。

**（1）选出新的主服务器**

选择的标准是：状态良好、数据完整的从服务器。在满足基本连接需求后，判断偏移量，**选出偏移量最大（保存最新数据）的从服务器作为新的主服务器。**然后向这个从服务器发送**SLAVEOF no one**命令，将这个从服务器转换为主服务器。

在发送**SLAVEOF no one**命令之后，领头Sentinel会以每秒一次的频率（平时是每十秒一次），**向被升级的从服务器发送INFO命令**，并观察命令回复中的角色（role）信息，当被升级服务器的role从原来的slave变为master时，领头Sentinel就知道被选中的从服务器已经顺利升级为主服务器了。

**（2）修改复制目标**

之后需要让已下线主服务器属下的所有从服务器去复制新的主服务器，这一动作可以通过向从服务器发送SLAVEOF命令来实现。

**（3）将旧的主服务器变为从服务器**

当原来的主服务器重新上线时，Sentinel就会向它发送SLAVEOF命令，让它成为新主服务器的从服务器。

# 集群

Redis集群是Redis提供的分布式数据库方案，集群通过分片（sharding）来进行数据共享，并提供复制和故障转移功能。<!--more-->

# 1. 节点

## 1.1 节点是什么

节点时Redis中数据存储的单位。一个Redis集群通常由多个节点（node）组成，**在刚开始的时候，每个节点都是相互独立的**，它们都处于一个只包含自己的集群当中，要组建一个真正可工作的集群，我们必须**将各个独立的节点连接起来，构成一个包含多个节点的集群。**

使用`CLUSTER MEET`命令来完成，该命令的格式如下：

```C
CLUSTER MEET <ip> <port>
```

当前节点发送`CLUSTER MEET`命令，可以与**ip和port所指定的节点进行握手（handshake）**，当握手成功时，node节点就会将ip和port所指定的节点添加到node节点当前所在的集群中。

## 1.2 启动节点

**一个节点就是一个运行在集群模式下的Redis服务器**，Redis服务器在启动时会根据`cluster-enabled`配置选项是否为yes来决定是否开启服务器的集群模式:

## 1.3 集群数据结构

`clusterNode`结构保存了一个节点的当前状态，比如节点的创建时间、节点的名字、节点当前的配置纪元、节点的IP地址和端口号等等。**每个节点都会使用一个`clusterNode`结构来记录自己的状态，并为集群中的所有其他节点（包括主节点和从节点）都创建一个相应的clusterNode结构，以此来记录其他节点的状态：**

```C
struct clusterNode {

    // 创建节点的时间
    mstime_t ctime;

    // 节点的名字，由 40 个十六进制字符组成
    // 例如 68eef66df23420a5862208ef5b1a7005b806f2ff
    char name[REDIS_CLUSTER_NAMELEN];

    // 节点标识
    // 使用各种不同的标识值记录节点的角色（比如主节点或者从节点），
    // 以及节点目前所处的状态（比如在线或者下线）。
    int flags;

    // 节点当前的配置纪元，用于实现故障转移
    uint64_t configEpoch;

    // 节点的 IP 地址
    char ip[REDIS_IP_STR_LEN];

    // 节点的端口号
    int port;

    // 保存连接节点所需的有关信息
    clusterLink *link;
    // ...
};
```

`clusterNode` 结构的 `link` 属性是一个 `clusterLink` 结构， 该结构保存了**连接节点所需的有关信息**， 比如套接字描述符， 输入缓冲区和输出缓冲区：

```C
typedef struct clusterLink {

    // 连接的创建时间
    mstime_t ctime;

    // TCP 套接字描述符
    int fd;

    // 输出缓冲区，保存着等待发送给其他节点的消息（message）。
    sds sndbuf;

    // 输入缓冲区，保存着从其他节点接收到的消息。
    sds rcvbuf;

    // 与这个连接相关联的节点，如果没有的话就为 NULL
    struct clusterNode *node;

} clusterLink;
```

- clusterState保存**集群状态**，**每个节点都保存着一个这样的状态**，记录了它们眼中的集群的样子。
- clusterNode保存了**某个节点的状态**，同时通过指针指向其他节点，达到关联的目的
- clusterLink保存了**和其他节点通信的信息**。

## 1.4 实现CLUSTER MEET命令

**向A**发送命令，**希望A和B形成集群**，则：

1. 节点A会为节点B创建一个clusterNode结构，并将该结构添加到自己的`clusterState.nodes`字典里面。
2. 根据命令给定的IP和端口，节点A向B发送一条MEET消息
3. B收到以后，为节点A创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面。
4. B向A返回一条PONG消息
5. A收到PONG后向B返回一条PING
6. B收到PING后，握手完成。

# 2. 槽指派

Redis集群通过分片的方式来保存数据库中的键值对：**集群的整个数据库被分为16384个槽（slot）**，**数据库中的每个键都属于这16384个槽的其中一个**，集群中的每个节点可以处理0个或最多16384个槽。

当数据库中的16384个槽**都有**节点在处理时，**集群处于上线状态（ok）**；相反地，如果数据库中有任何一个槽没有得到处理，那么集群处于下线状态（fail）。

换句话说，只有完全分配了16384个槽才会进入上线状态。

## 2.1 当前节点的槽指派信息

clusterNode结构的`slots`属性和`numslot`属性记录了节点负责处理哪些槽：

```C
struct clusterNode 
{ 
    // ... 
    unsigned char slots[16384/8]; 
    int numslots; 
    // ...
};
```

`slots`属性是一个二进制位数组（bit array），这个数组的长度为16384/8=2048个字节，共包含16384个二进制位。通过判断二进制的01状态来判断，此槽是否属于。

## 2.2 传播节点的槽指派信息

一个节点除了会将自己负责处理的槽记录在clusterNode结构的slots属性和numslots属性之外，它**还会将自己的slots数组通过消息发送给集群中的其他节点，以此来告知其他节点自己目前负责处理哪些槽。**

当节点A通过消息从节点B那里接收到节点B的`slots`数组时，节点A会在自己的`clusterState.nodes`字典中**查找节点B对应的clusterNode结构，并对结构中的`slots`数组进行保存或者更新。**

因此，集群中的**每个节点**都会知道数据库中的16384个槽分别被指派给了集群中的哪些节点。

在clusterState中有一个myself指针，指向当前节点clusterNode，这个结构中包含了一个二进制数组，记录当前节点的槽指派情况。而clusterState中还有一个`clusterNode *slots[REDIS_CLUSTER_SLOTS];`，记录了其他节点的槽指派情况。例如 `slots[i] = clusterNode_A` 表示槽 i 由节点 A 处理

## 2.3 集群所有槽的指派信息

clusterState结构中的`slots`数组记录了集群中所有16384个槽的指派信息：

```C
typedef struct clusterState 
{ 
    // ... 
    clusterNode *slots[16384]; 
    // ...
} clusterState;
```

每个数组都是指向`clusterNode`结构的指针：

- 如果指向NULL，表示尚未指派给任何节点。
- 如果指向某一个clusterNode，则表示槽i指派给了某一个节点。

如果只将槽指派信息保存在各个节点的`clusterNode.slots`数组里，会出现一些无法高效地解决的问题，而`clusterState.slots`数组的存在解决了这些问题：

- 如果想知道某个槽是否被指派以及被指派给了谁，需要遍历所有clusterNode结构。
- 通过`clusterState`保存的数组，可以以$O(1)$的时间取得结果。

## 2.4 槽的保存方式

节点还会用`clusterState`结构中的`slots_to_keys`**跳跃表**来保存槽和键之间的关系:

```C
typedef struct clusterState 
{ 
    // ... 
    zskiplist *slots_to_keys; 
    // ...
} clusterState;
```

`slots_to_keys`跳跃表每个节点的分值（score）都是一个槽号，而每个节点的成员（member）都是一个数据库键：

# 3. MOVED错误

前面提到，指派完槽以后，集群会进入上线状态，此时客户端可以向集群中的节点发送数据命令。

当客户端向节点发送与数据库键有关的命令时，**接收命令的节点会计算出命令要处理的数据库键属于哪个槽，并检查这个槽是否指派给了自己：**

- 如果指派给自己，执行
- 否则，返回MOVED错误，并**将客户端指向正确的节点**。

比如，date键所在的槽正好是节点7000负责，正常执行。

```C
127.0.0.1:7000> SET date "2013-12-31"
OK
```

如果是如下的情况，客户端会被自动转到正确的节点。

```c
127.0.0.1:7000> SET msg "happy new year!"
-> Redirected to slot [6257] located at 127.0.0.1:7001
OK

127.0.0.1:7001> GET msg
"happy new year!"
```

要完成上面的操作，需要至少两步：

- 判断槽是否自己负责
- MOVED错误的实现方法

## 3.1 判断槽是否自己负责

键属于哪个槽需要用到CRC-16校验的办法：

```C
slot_number = CRC16(key)&16383
```

这样就可以计算出一个介于0至16383之间的整数作为键key的槽号。

当节点计算出键所属的槽i之后，节点就会检查自己在`clusterState.slots`数组中的项i，判断键所在的槽是否由自己负责：

- 如果`clusterState.slots[i]`等于`clusterState.myself`，那么说明槽i由当前节点负责，节点可以执行客户端发送的命令。
- 反之，则记下指向clusterNode结构所记录的IP和端口号

## 3.2 MOVED错误实现

MOVED错误的格式为：

```
MOVED <slot> <ip>:<port>
```

其中slot为键所在的槽，而ip和port则是负责处理槽slot的节点的IP地址和端口号。

当客户端接收到节点返回的MOVED错误时，客户端会根据MOVED错误中提供的IP地址和端口号，转向至负责处理槽slot的节点，并向该节点重新发送之前想要执行的命令。

```C
127.0.0.1:7000> SET msg "happy new year!"
-> Redirected to slot [6257] located at 127.0.0.1:7001
OK
```

# 4. 重新分片

Redis集群的重新分片操作可以将**任意数量已经指派给某个节点（源节点）的槽改为指派给另一个节点（目标节点）**，并且相关槽所属的键值对也会从源节点被移动到目标节点。

重新分片操作可以**在线（online）进行**，在重新分片的过程中，集群不需要下线，并且**源节点和目标节点都可以继续处理命令请求**。

Redis集群的重新分片操作是由Redis的集群管理软件**redis-trib**负责执行的，**Redis提供了进行重新分片所需的所有命令**，而redis-trib则**通过向源节点和目标节点发送命令来进行重新分片操作。**

# 5. ASK错误

**正在重新分片时**，属于被迁移槽的一部分键值对保存在源节点里面，而另一部分键值对则保存在目标节点里面。

当客户端向源节点发送一个与数据库键有关的命令，并且命令要处理的数据库键恰好就属于正在被迁移的槽时：

- 槽在自己这里，执行客户端命令。
- 槽不在，返回ASK错误，指引客户端转向正在导入槽的目标节点。

下面讲解ASK错误的实现原理：

## 5.1 关键命令

**（1）CLUSTER SETSLOT IMPORTING命令的实现**

`clusterState`结构的`importing_slots_from`数组记录了当前节点正在从其他节点导入的槽：

```c
typedef struct clusterState 
{ 
    // ... 
    clusterNode *importing_slots_from[16384]; 
    // ...
} clusterState;
```

如果`importing_slots_from[i]`的值不为NULL，而是指向一个clusterNode结构，那么表示**当前节点正在从clusterNode所代表的节点导入槽i。**

在对集群进行重新分片的时候，向目标节点发送命令：

```C
CLUSTER SETSLOT <i> IMPORTING <source_id>
```

假如，客户端向节点7003发送命令：

```C
# 9dfb... 是节点7002 的ID
127.0.0.1:7003> CLUSTER SETSLOT 16198 IMPORTING 
9dfb4c4e016e627d9769e4c9bb0d4fa208e65c26OK
```

**（2）CLUSTER SETSLOT MIGRATING命令的实现**

`clusterState`结构的`migrating_slots_to`数组记录了当前节点正在迁移至其他节点的槽：

```C
typedef struct clusterState 
{ 
    // ... 
    clusterNode *migrating_slots_to[16384]; 
    // ...
} clusterState;
```

同理，如果索引i不为NULL，则表示当前节点正在将槽i迁移至目标节点。

在对集群进行重新分片的时候，向源节点发送命令：

```C
CLUSTER SETSLOT <i> MIGRATING <target_id>
```

可以将源节点`clusterState.migrating_slots_to[i]`的值设置为target_id所代表节点的clusterNode结构。

## 5.2 ASKING命令

通过`migrating_slots_to`这个数组，我们知道当前节点的某个键是否正在迁移。如果是则返回ASK错误。

```C
ASK 16198 127.0.0.1:7003
```

接到ASK错误的客户端会根据错误提供的IP地址和端口号，**转向至正在导入槽的目标节点，然后首先向目标节点发送一个ASKING命令**，之后再重新发送原本想要执行的命令。

ASKING命令的目的就是打开REDIS_ASKING标识，而且**是一次性的打开**，意味着使用完后会被关闭。

ASK错误和MOVED错误的区别：

- **相同点**：都会导致客户端转向
- **不同点：**
  - MOVED错误代表槽的**负责权已经从一个节点转移到了另一个节点**：在客户端收到关于槽i的MOVED错误之后，客户端**每次遇到**关于槽i的命令请求时，都可以直接将命令请求发送至MOVED错误所指向的节点。
  - ASK错误只是两个节点在迁移槽的**过程中使用的一种临时措施**：在客户端收到关于槽i的ASK错误之后，客户端只会在**接下来的一次命令请求中将关于槽i的命令请求发送至ASK错误所指示的节点**，但这种转向不会对客户端今后发送关于槽i的命令请求产生任何影响，**客户端仍然会将关于槽i的命令请求发送至目前负责处理槽i的节点**，除非ASK错误再次出现。

# 6. 节点的复制与故障转移

和主从服务器的关系非常相似，不过**在集群模式下服务器被替换为节点。**Redis集群中的节点分为主节点（master）和从节点（slave），其中主节点用于处理槽，而从节点则用于复制某个主节点，并在被复制的主节点下线时，代替下线主节点继续处理命令请求。

## 6.1 设置从节点

向一个节点发送命令：

```C
CLUSTER REPLICATE <node_id>
```

可以让接收命令的节点成为node_id所指定节点的从节点，并开始对主节点进行复制，步骤如下：

**（1）修改指针指向**

接收到该命令的节点首先会在自己的`clusterState.nodes`字典中找到node_id所对应节点的clusterNode结构，**并将自己的`clusterState.myself.slaveof`指针指向这个结构**，以此来记录这个节点正在复制的主节点：

```C
struct clusterNode 
{ 
    // ... 
    // 如果这是一个从节点，那么指向主节点 
    struct clusterNode *slaveof;
};
```

**（2）修改标识**

然后节点会修改自己在`clusterState.myself.flags`中的属性，关闭原本的`REDIS_NODE_MASTER`标识，打开`REDIS_NODE_SLAVE`标识。

**（3）调用复制代码**

根据`clusterState.my-self.slaveof`指向的clusterNode结构所保存的IP地址和端口号，对主节点进行复制。因为**节点的复制功能和单机Redis服务器的复制功能使用了相同的代码**，所以让从节点复制主节点相当于向从节点发送命令SLAVEOF。

一个节点成为从节点，并开始复制某个主节点这一信息**会通过消息发送给集群中的其他节点，最终集群中的所有节点都会知道某个从节点正在复制某个主节点。**

集群中的所有节点都会在代表主节点的clusterNode结构的`slaves`属性和`numslaves`属性中记录正在复制这个主节点的从节点名单：

```C
struct clusterNode 
{ 
    // ... 
    // 正在复制这个主节点的从节点数量
    int numslaves;
    // 一个数组 
    // 每个数组项指向一个正在复制这个主节点的从节点的clusterNode结构 
    struct clusterNode **slaves;
    //....
}
```

## 6.2 故障检测

集群中的每个节点都会定期地向集群中的其他节点发送PING消息，如果接收PING消息的节点没有在规定的时间内返回PONG，那么发送PING消息的节点就会将接收PING消息的节点**标记为疑似下线（probable fail，PFAIL）**

集群中的各个节点会通过**互相发送消息**的方式来交换集群中各个节点的状态信息，例如某个节点是处于**在线状态、疑似下线状态（PFAIL），还是已下线状态（FAIL）。**

当一个主节点A通过消息得知主节点B认为主节点C进入了疑似下线状态时，主节点A会在自己的`clusterState.nodes`字典中找到主节点C所对应的clusterNode结构，**并将主节点B的下线报告（failure report）添加到clusterNode结构的`fail_reports`链表里面：**

```C
struct clusterNode 
{ 
    // ...
    // 一个链表，记录了所有其他节点对该节点的下线报告 
    list *fail_reports;
     // ...
};
```

如果在一个集群里面，**半数以上负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点x将被标记为已下线（FAIL**），将主节点x标记为已下线的节点**会向集群广播一条关于主节点x的FAIL消息，所有收到这条FAIL消息的节点都会立即将主节点x标记为已下线。**

## 6.3 故障转移

当一个从节点发现自己正在复制的主节点进入了已下线状态时，从节点将开始对下线主节点进行故障转移，以下是故障转移的执行步骤：

1. 从已下线主节点中选出一个从节点
2. 从节点执行SLAVEOF no one命令，成为新的主节点
3. 新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己。
4. 新的主节点向集群广播一条PONG消息，这条PONG消息可以让集群中的其他节点立即知道这个节点已经由从节点变成了主节点。
5. 新的主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。

# Sentinel

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

- **监控（Monitoring）**： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- **提醒（Notification）**： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- **自动故障迁移（Automatic failover）**： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

Redis Sentinel 是一个分布式系统， 你可以在一个架构中运行多个 Sentinel 进程（progress）， 这些进程使用流言协议（gossip protocols)来接收关于主服务器是否下线的信息， 并使用投票协议（agreement protocols）来决定是否执行自动故障迁移， 以及选择哪个从服务器作为新的主服务器。

虽然 Redis Sentinel 释出为一个单独的可执行文件 `redis-sentinel` ， 但实际上它只是一个运行在特殊模式下的 Redis 服务器， 你可以在启动一个普通 Redis 服务器时通过给定 `--sentinel` 选项来启动 Redis Sentinel 。

Warning

Redis Sentinel 目前仍在开发中， 这个文档的内容可能随着 Sentinel 实现的修改而变更。

Redis Sentinel 兼容 Redis 2.4.16 或以上版本， 推荐使用 Redis 2.8.0 或以上的版本。

## 获取 Sentinel

目前 Sentinel 系统是 Redis 的 `unstable` 分支的一部分， 你必须到 [Redis 项目的 Github 页面](https://github.com/antirez/redis) 克隆一份 `unstable` 分值， 然后通过编译来获得 Sentinel 系统。

Sentinel 程序可以在编译后的 `src` 文档中发现， 它是一个命名为 `redis-sentinel` 的程序。

你也可以通过下一节介绍的方法， 让 `redis-server` 程序运行在 Sentinel 模式之下。

另外， 一个新版本的 Sentinel 已经包含在了 Redis 2.8.0 版本的释出文件中。

## 启动 Sentinel

对于 `redis-sentinel` 程序， 你可以用以下命令来启动 Sentinel 系统：

```
redis-sentinel /path/to/sentinel.conf
```

对于 `redis-server` 程序， 你可以用以下命令来启动一个运行在 Sentinel 模式下的 Redis 服务器：

```
redis-server /path/to/sentinel.conf --sentinel
```

两种方法都可以启动一个 Sentinel 实例。

启动 Sentinel 实例必须指定相应的配置文件， 系统会使用配置文件来保存 Sentinel 的当前状态， 并在 Sentinel 重启时通过载入配置文件来进行状态还原。

如果启动 Sentinel 时没有指定相应的配置文件， 或者指定的配置文件不可写（not writable）， 那么 Sentinel 会拒绝启动。

## 配置 Sentinel

Redis 源码中包含了一个名为 `sentinel.conf` 的文件， 这个文件是一个带有详细注释的 Sentinel 配置文件示例。

运行一个 Sentinel 所需的最少配置如下所示：

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

第一行配置指示 Sentinel 去监视一个名为 `mymaster` 的主服务器， 这个主服务器的 IP 地址为 `127.0.0.1` ， 端口号为 `6379` ， 而将这个主服务器判断为失效至少需要 `2` 个 Sentinel 同意 （只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行）。

不过要注意， 无论你设置要多少个 Sentinel 同意才能判断一个服务器失效， **一个 Sentinel 都需要获得系统中多数（majority） Sentinel 的支持， 才能发起一次自动故障迁移，** 并预留一个给定的配置纪元 （configuration Epoch ，一个配置纪元就是一个新主服务器配置的版本号）。

换句话说， **在只有少数（minority） Sentinel 进程正常运作的情况下， Sentinel 是不能执行自动故障迁移的。**

其他选项的基本格式如下：

```
sentinel <选项的名字> <主服务器的名字> <选项的值>
```

各个选项的功能如下：

- `down-after-milliseconds` 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。
  
  如果服务器在给定的毫秒数之内， 没有返回 Sentinel 发送的 PING命令的回复， 或者返回一个错误， 那么 Sentinel 将这个服务器标记为**主观下线**（subjectively down，简称 `SDOWN` ）。
  
  不过只有一个 Sentinel 将服务器标记为主观下线并不一定会引起服务器的自动故障迁移： 只有在足够数量的 Sentinel 都将一个服务器标记为主观下线之后， 服务器才会被标记为**客观下线**（objectively down， 简称 `ODOWN` ）， 这时自动故障迁移才会执行。
  
  将服务器标记为客观下线所需的 Sentinel 数量由对主服务器的配置决定。

- `parallel-syncs` 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。
  
  如果从服务器被设置为允许使用过期数据集（参见对 `redis.conf` 文件中对 `slave-serve-stale-data` 选项的说明）， 那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求， 因为尽管复制过程的绝大部分步骤都不会阻塞从服务器， 但从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求： 如果全部从服务器一起对新的主服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。
  
  你可以通过将这个值设为 `1` 来保证每次只有一个从服务器处于不能处理命令请求的状态。

本文档剩余的内容将对 Sentinel 系统的其他选项进行介绍， 示例配置文件 `sentinel.conf` 也对相关的选项进行了完整的注释。

## 主观下线和客观下线

前面说过， Redis 的 Sentinel 中关于下线（down）有两个不同的概念：

- 主观下线（Subjectively Down， 简称 SDOWN）指的是单个 Sentinel 实例对服务器做出的下线判断。
- 客观下线（Objectively Down， 简称 ODOWN）指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 `SENTINEL is-master-down-by-addr` 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 `SENTINEL is-master-down-by-addr` 命令来询问对方是否认为给定的服务器已下线。）

如果一个服务器没有在 `master-down-after-milliseconds` 选项所指定的时间内， 对向它发送 PING命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线。

服务器对 PING 命令的有效回复可以是以下三种回复的其中一种：

- 返回 `+PONG` 。
- 返回 `-LOADING` 错误。
- 返回 `-MASTERDOWN` 错误。

如果服务器返回除以上三种回复之外的其他回复， 又或者在指定时间内没有回复 PING 命令， 那么 Sentinel 认为服务器返回的回复无效（non-valid）。

注意， 一个服务器必须在 `master-down-after-milliseconds` 毫秒内， 一直返回无效回复才会被 Sentinel 标记为主观下线。

举个例子， 如果 `master-down-after-milliseconds` 选项的值为 `30000` 毫秒（`30` 秒）， 那么只要服务器能在每 `29` 秒之内返回至少一次有效回复， 这个服务器就仍然会被认为是处于正常状态的。

从主观下线状态切换到客观下线状态并没有使用严格的法定人数算法（strong quorum algorithm）， 而是使用了流言协议： 如果 Sentinel 在给定的时间范围内， 从其他 Sentinel 那里接收到了足够数量的主服务器下线报告， 那么 Sentinel 就会将主服务器的状态从主观下线改变为客观下线。 如果之后其他 Sentinel 不再报告主服务器已下线， 那么客观下线状态就会被移除。

客观下线条件**只适用于主服务器**： 对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。

只要一个 Sentinel 发现某个主服务器进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

## 每个 Sentinel 都需要定期执行的任务

- 每个 Sentinel 以每秒钟一次的频率向它所知的主服务器、从服务器以及其他 Sentinel 实例发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 `down-after-milliseconds` 选项所指定的值， 那么这个实例会被 Sentinel 标记为主观下线。 一个有效回复可以是： `+PONG` 、 `-LOADING` 或者 `-MASTERDOWN` 。
- 如果一个主服务器被标记为主观下线， 那么正在监视这个主服务器的所有 Sentinel 要以每秒一次的频率确认主服务器的确进入了主观下线状态。
- 如果一个主服务器被标记为主观下线， 并且有足够数量的 Sentinel （至少要达到配置文件指定的数量）在指定的时间范围内同意这一判断， 那么这个主服务器被标记为客观下线。
- 在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有主服务器和从服务器发送 [INFO section\] 命令。 当一个主服务器被 Sentinel 标记为客观下线时， Sentinel 向下线主服务器的所有从服务器发送 [INFO section\] 命令的频率会从 10 秒一次改为每秒一次。
- 当没有足够数量的 Sentinel 同意主服务器已经下线， 主服务器的客观下线状态就会被移除。 当主服务器重新向 Sentinel 的 PING 命令返回有效回复时， 主服务器的主管下线状态就会被移除。

## 自动发现 Sentinel 和从服务器

一个 Sentinel 可以与其他多个 Sentinel 进行连接， 各个 Sentinel 之间可以互相检查对方的可用性， 并进行信息交换。

你无须为运行的每个 Sentinel 分别设置其他 Sentinel 的地址， 因为 Sentinel 可以通过发布与订阅功能来自动发现正在监视相同主服务器的其他 Sentinel ， 这一功能是通过向频道 `__sentinel__:hello` 发送信息来实现的。

与此类似， 你也不必手动列出主服务器属下的所有从服务器， 因为 Sentinel 可以通过询问主服务器来获得所有从服务器的信息。

- 每个 Sentinel 会以每两秒一次的频率， 通过发布与订阅功能， 向被它监视的所有主服务器和从服务器的 `__sentinel__:hello` 频道发送一条信息， 信息中包含了 Sentinel 的 IP 地址、端口号和运行 ID （runid）。
- 每个 Sentinel 都订阅了被它监视的所有主服务器和从服务器的 `__sentinel__:hello` 频道， 查找之前未出现过的 sentinel （looking for unknown sentinels）。 当一个 Sentinel 发现一个新的 Sentinel 时， 它会将新的 Sentinel 添加到一个列表中， 这个列表保存了 Sentinel 已知的， 监视同一个主服务器的所有其他 Sentinel 。
- Sentinel 发送的信息中还包括完整的主服务器当前配置（configuration）。 如果一个 Sentinel 包含的主服务器配置比另一个 Sentinel 发送的配置要旧， 那么这个 Sentinel 会立即升级到新配置上。
- 在将一个新 Sentinel 添加到监视主服务器的列表上面之前， Sentinel 会先检查列表中是否已经包含了和要添加的 Sentinel 拥有相同运行 ID 或者相同地址（包括 IP 地址和端口号）的 Sentinel ， 如果是的话， Sentinel 会先移除列表中已有的那些拥有相同运行 ID 或者相同地址的 Sentinel ， 然后再添加新 Sentinel 。

## Sentinel API

在默认情况下， Sentinel 使用 TCP 端口 `26379` （普通 Redis 服务器使用的是 `6379` ）。

Sentinel 接受 Redis 协议格式的命令请求， 所以你可以使用 `redis-cli` 或者任何其他 Redis 客户端来与 Sentinel 进行通讯。

有两种方式可以和 Sentinel 进行通讯：

- 第一种方法是通过直接发送命令来查询被监视 Redis 服务器的当前状态， 以及 Sentinel 所知道的关于其他 Sentinel 的信息， 诸如此类。
- 另一种方法是使用发布与订阅功能， 通过接收 Sentinel 发送的通知： 当执行故障转移操作， 或者某个被监视的服务器被判断为主观下线或者客观下线时， Sentinel 就会发送相应的信息。

### Sentinel 命令

以下列出的是 Sentinel 接受的命令：

- PING ：返回 `PONG` 。
- `SENTINEL masters` ：列出所有被监视的主服务器，以及这些主服务器的当前状态。
- `SENTINEL slaves ` ：列出给定主服务器的所有从服务器，以及这些从服务器的当前状态。
- `SENTINEL get-master-addr-by-name ` ： 返回给定名字的主服务器的 IP 地址和端口号。 如果这个主服务器正在执行故障转移操作， 或者针对这个主服务器的故障转移操作已经完成， 那么这个命令返回新的主服务器的 IP 地址和端口号。
- `SENTINEL reset ` ： 重置所有名字和给定模式 `pattern` 相匹配的主服务器。 `pattern` 参数是一个 Glob 风格的模式。 重置操作清除主服务器目前的所有状态， 包括正在执行中的故障转移， 并移除目前已经发现和关联的， 主服务器的所有从服务器和 Sentinel 。
- `SENTINEL failover ` ： 当主服务器失效时， 在不询问其他 Sentinel 意见的情况下， 强制开始一次自动故障迁移 （不过发起故障转移的 Sentinel 会向其他 Sentinel 发送一个新的配置，其他 Sentinel 会根据这个配置进行相应的更新）。

### 发布与订阅信息

客户端可以将 Sentinel 看作是一个只提供了订阅功能的 Redis 服务器： 你不可以使用 PUBLISH channel message 命令向这个服务器发送信息， 但你可以用 [SUBSCRIBE channel channel …\] 命令或者 [PSUBSCRIBE pattern pattern …\] 命令， 通过订阅给定的频道来获取相应的事件提醒。

一个频道能够接收和这个频道的名字相同的事件。 比如说， 名为 `+sdown` 的频道就可以接收所有实例进入主观下线（SDOWN）状态的事件。

通过执行 `PSUBSCRIBE *` 命令可以接收所有事件信息。

以下列出的是客户端可以通过订阅来获得的频道和信息的格式： 第一个英文单词是频道/事件的名字， 其余的是数据的格式。

注意， 当格式中包含 `instance details` 字样时， 表示频道所返回的信息中包含了以下用于识别目标实例的内容：

```
<instance-type> <name> <ip> <port> @ <master-name> <master-ip> <master-port>
```

`@` 字符之后的内容用于指定主服务器， 这些内容是可选的， 它们仅在 `@` 字符之前的内容指定的实例不是主服务器时使用。

- `+reset-master ` ：主服务器已被重置。
- `+slave ` ：一个新的从服务器已经被 Sentinel 识别并关联。
- `+failover-state-reconf-slaves ` ：故障转移状态切换到了 `reconf-slaves` 状态。
- `+failover-detected ` ：另一个 Sentinel 开始了一次故障转移操作，或者一个从服务器转换成了主服务器。
- `+slave-reconf-sent ` ：领头（leader）的 Sentinel 向实例发送了 SLAVEOF host port 命令，为实例设置新的主服务器。
- `+slave-reconf-inprog ` ：实例正在将自己设置为指定主服务器的从服务器，但相应的同步过程仍未完成。
- `+slave-reconf-done ` ：从服务器已经成功完成对新主服务器的同步。
- `-dup-sentinel ` ：对给定主服务器进行监视的一个或多个 Sentinel 已经因为重复出现而被移除 —— 当 Sentinel 实例重启的时候，就会出现这种情况。
- `+sentinel ` ：一个监视给定主服务器的新 Sentinel 已经被识别并添加。
- `+sdown ` ：给定的实例现在处于主观下线状态。
- `-sdown ` ：给定的实例已经不再处于主观下线状态。
- `+odown ` ：给定的实例现在处于客观下线状态。
- `-odown ` ：给定的实例已经不再处于客观下线状态。
- `+new-epoch ` ：当前的纪元（epoch）已经被更新。
- `+try-failover ` ：一个新的故障迁移操作正在执行中，等待被大多数 Sentinel 选中（waiting to be elected by the majority）。
- `+elected-leader ` ：赢得指定纪元的选举，可以进行故障迁移操作了。
- `+failover-state-select-slave ` ：故障转移操作现在处于 `select-slave` 状态 —— Sentinel 正在寻找可以升级为主服务器的从服务器。
- `no-good-slave ` ：Sentinel 操作未能找到适合进行升级的从服务器。Sentinel 会在一段时间之后再次尝试寻找合适的从服务器来进行升级，又或者直接放弃执行故障转移操作。
- `selected-slave ` ：Sentinel 顺利找到适合进行升级的从服务器。
- `failover-state-send-slaveof-noone ` ：Sentinel 正在将指定的从服务器升级为主服务器，等待升级功能完成。
- `failover-end-for-timeout ` ：故障转移因为超时而中止，不过最终所有从服务器都会开始复制新的主服务器（slaves will eventually be configured to replicate with the new master anyway）。
- `failover-end ` ：故障转移操作顺利完成。所有从服务器都开始复制新的主服务器了。
- `+switch-master   ` ：配置变更，主服务器的 IP 和地址已经改变。 **这是绝大多数外部用户都关心的信息。**
- `+tilt` ：进入 tilt 模式。
- `-tilt` ：退出 tilt 模式。

## 故障转移

一次故障转移操作由以下步骤组成：

- 发现主服务器已经进入客观下线状态。
- 对我们的当前纪元进行自增（详情请参考 [Raft leader election](https://www.google.com.hk/search?q=Raft+leader+election&client=ubuntu&channel=cs&oq=Raft+leader+election&aqs=chrome..69i57&sourceid=chrome&ie=UTF-8) ）， 并尝试在这个纪元中当选。
- 如果当选失败， 那么在设定的故障迁移超时时间的两倍之后， 重新尝试当选。 如果当选成功， 那么执行以下步骤。
- 选出一个从服务器，并将它升级为主服务器。
- 向被选中的从服务器发送 `SLAVEOF NO ONE` 命令，让它转变为主服务器。
- 通过发布与订阅功能， 将更新后的配置传播给所有其他 Sentinel ， 其他 Sentinel 对它们自己的配置进行更新。
- 向已下线主服务器的从服务器发送 SLAVEOF host port 命令， 让它们去复制新的主服务器。
- 当所有从服务器都已经开始复制新的主服务器时， 领头 Sentinel 终止这次故障迁移操作。

Note

每当一个 Redis 实例被重新配置（reconfigured） —— 无论是被设置成主服务器、从服务器、又或者被设置成其他主服务器的从服务器 —— Sentinel 都会向被重新配置的实例发送一个 `CONFIG REWRITE` 命令， 从而确保这些配置会持久化在硬盘里。

Sentinel 使用以下规则来选择新的主服务器：

- 在失效主服务器属下的从服务器当中， 那些被标记为主观下线、已断线、或者最后一次回复 PING 命令的时间大于五秒钟的从服务器都会被淘汰。
- 在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 `down-after` 选项指定的时长十倍的从服务器都会被淘汰。
- 在经历了以上两轮淘汰之后剩下来的从服务器中， 我们选出复制偏移量（replication offset）最大的那个从服务器作为新的主服务器； 如果复制偏移量不可用， 或者从服务器的复制偏移量相同， 那么带有最小运行 ID 的那个从服务器成为新的主服务器。

### Sentinel 自动故障迁移的一致性特质

Sentinel 自动故障迁移使用 Raft 算法来选举领头（leader） Sentinel ， 从而确保在一个给定的纪元（epoch）里， 只有一个领头产生。

这表示在同一个纪元中， 不会有两个 Sentinel 同时被选中为领头， 并且各个 Sentinel 在同一个纪元中只会对一个领头进行投票。

更高的配置纪元总是优于较低的纪元， 因此每个 Sentinel 都会主动使用更新的纪元来代替自己的配置。

简单来说， 我们可以将 Sentinel 配置看作是一个带有版本号的状态。 一个状态会以最后写入者胜出（last-write-wins）的方式（也即是，最新的配置总是胜出）传播至所有其他 Sentinel 。

举个例子， 当出现网络分割（[network partitions](http://en.wikipedia.org/wiki/Network_partition)）时， 一个 Sentinel 可能会包含了较旧的配置， 而当这个 Sentinel 接到其他 Sentinel 发来的版本更新的配置时， Sentinel 就会对自己的配置进行更新。

如果要在网络分割出现的情况下仍然保持一致性， 那么应该使用 `min-slaves-to-write` 选项， 让主服务器在连接的从实例少于给定数量时停止执行写操作， 与此同时， 应该在每个运行 Redis 主服务器或从服务器的机器上运行 Redis Sentinel 进程。

### Sentinel 状态的持久化

Sentinel 的状态会被持久化在 Sentinel 配置文件里面。

每当 Sentinel 接收到一个新的配置， 或者当领头 Sentinel 为主服务器创建一个新的配置时， 这个配置会与配置纪元一起被保存到磁盘里面。

这意味着停止和重启 Sentinel 进程都是安全的。

### Sentinel 在非故障迁移的情况下对实例进行重新配置

即使没有自动故障迁移操作在进行， Sentinel 总会尝试将当前的配置设置到被监视的实例上面。 特别是：

- 根据当前的配置， 如果一个从服务器被宣告为主服务器， 那么它会代替原有的主服务器， 成为新的主服务器， 并且成为原有主服务器的所有从服务器的复制对象。
- 那些连接了错误主服务器的从服务器会被重新配置， 使得这些从服务器会去复制正确的主服务器。

不过， 在以上这些条件满足之后， Sentinel 在对实例进行重新配置之前仍然会等待一段足够长的时间， 确保可以接收到其他 Sentinel 发来的配置更新， 从而避免自身因为保存了过期的配置而对实例进行了不必要的重新配置。

## TILT 模式

Redis Sentinel 严重依赖计算机的时间功能： 比如说， 为了判断一个实例是否可用， Sentinel 会记录这个实例最后一次相应 PING 命令的时间， 并将这个时间和当前时间进行对比， 从而知道这个实例有多长时间没有和 Sentinel 进行任何成功通讯。

不过， 一旦计算机的时间功能出现故障， 或者计算机非常忙碌， 又或者进程因为某些原因而被阻塞时， Sentinel 可能也会跟着出现故障。

TILT 模式是一种特殊的保护模式： 当 Sentinel 发现系统有些不对劲时， Sentinel 就会进入 TILT 模式。

因为 Sentinel 的时间中断器默认每秒执行 10 次， 所以我们预期时间中断器的两次执行之间的间隔为 100 毫秒左右。 Sentinel 的做法是， 记录上一次时间中断器执行时的时间， 并将它和这一次时间中断器执行的时间进行对比：

- 如果两次调用时间之间的差距为负值， 或者非常大（超过 2 秒钟）， 那么 Sentinel 进入 TILT 模式。
- 如果 Sentinel 已经进入 TILT 模式， 那么 Sentinel 延迟退出 TILT 模式的时间。

当 Sentinel 进入 TILT 模式时， 它仍然会继续监视所有目标， 但是：

- 它不再执行任何操作，比如故障转移。
- 当有实例向这个 Sentinel 发送 `SENTINEL is-master-down-by-addr` 命令时， Sentinel 返回负值： 因为这个 Sentinel 所进行的下线判断已经不再准确。

如果 TILT 可以正常维持 30 秒钟， 那么 Sentinel 退出 TILT 模式。

## 处理 `-BUSY` 状态

Warning

该功能尚未实现

当 Lua 脚本的运行时间超过指定时限时， Redis 就会返回 `-BUSY` 错误。

当出现这种情况时， Sentinel 在尝试执行故障转移操作之前， 会先向服务器发送一个 SCRIPT KILL 命令， 如果服务器正在执行的是一个只读脚本的话， 那么这个脚本就会被杀死， 服务器就会回到正常状态。

## Sentinel 的客户端实现

关于 Sentinel 客户端的实现信息可以参考 [Sentinel 客户端指引手册](http://redis.io/topics/sentinel-clients) 。

# 集群

Redis 集群是一个可以**在多个 Redis 节点之间进行数据共享**的设施（installation）。

Redis 集群不支持那些需要同时处理多个键的 Redis 命令， 因为执行这些命令需要在多个 Redis 节点之间移动数据， 并且在高负载的情况下， 这些命令将降低 Redis 集群的性能， 并导致不可预测的行为。

Redis 集群**通过分区（partition）来提供一定程度的可用性**（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。

Redis 集群提供了以下两个好处：

- 将数据自动切分（split）到多个节点的能力。
- 当集群中的一部分节点失效或者无法进行通讯时， 仍然可以继续处理命令请求的能力。

## Redis 集群数据共享

Redis 集群使用数据分片（sharding）而非一致性哈希（consistency hashing）来实现： 一个 Redis 集群包含 `16384` 个哈希槽（hash slot）， 数据库中的每个键都属于这 `16384` 个哈希槽的其中一个， 集群使用公式 `CRC16(key) % 16384` 来计算键 `key` 属于哪个槽， 其中 `CRC16(key)` 语句用于计算键 `key` 的 [CRC16 校验和](http://zh.wikipedia.org/wiki/循環冗餘校驗) 。

集群中的每个节点负责处理一部分哈希槽。 举个例子， 一个集群可以有三个哈希槽， 其中：

- 节点 A 负责处理 `0` 号至 `5500` 号哈希槽。
- 节点 B 负责处理 `5501` 号至 `11000` 号哈希槽。
- 节点 C 负责处理 `11001` 号至 `16384` 号哈希槽。

这种将哈希槽分布到不同节点的做法使得用户可以很容易地向集群中添加或者删除节点。 比如说：

- 如果用户将新节点 D 添加到集群中， 那么集群只需要将节点 A 、B 、 C 中的某些槽移动到节点 D 就可以了。
- 与此类似， 如果用户要从集群中移除节点 A ， 那么集群只需要将节点 A 中的所有哈希槽移动到节点 B 和节点 C ， 然后再移除空白（不包含任何哈希槽）的节点 A 就可以了。

因为将一个哈希槽从一个节点移动到另一个节点不会造成节点阻塞， 所以无论是添加新节点还是移除已存在节点， 又或者改变某个节点包含的哈希槽数量， 都不会造成集群下线。

## Redis 集群中的主从复制

为了使得集群在一部分节点下线或者无法与集群的大多数（majority）节点进行通讯的情况下， 仍然可以正常运作， Redis 集群对节点使用了主从复制功能： 集群中的每个节点都有 `1` 个至 `N` 个复制品（replica）， 其中一个复制品为主节点（master）， 而其余的 `N-1` 个复制品为从节点（slave）。

在之前列举的节点 A 、B 、C 的例子中， 如果节点 B 下线了， 那么集群将无法正常运行， 因为集群找不到节点来处理 `5501` 号至 `11000` 号的哈希槽。

另一方面， 假如在创建集群的时候（或者至少在节点 B 下线之前）， 我们为主节点 B 添加了从节点 B1 ， 那么当主节点 B 下线的时候， 集群就会将 B1 设置为新的主节点， 并让它代替下线的主节点 B ， 继续处理 `5501` 号至 `11000` 号的哈希槽， 这样集群就不会因为主节点 B 的下线而无法正常运作了。

不过如果节点 B 和 B1 都下线的话， Redis 集群还是会停止运作。

## Redis 集群的一致性保证（guarantee）

Redis 集群**不保证数据的强一致性**（strong consistency）： 在特定条件下， Redis 集群可能会丢失已经被执行过的写命令。

使用异步复制（asynchronous replication）是 Redis 集群可能会丢失写命令的其中一个原因。 考虑以下这个写命令的例子：

- 客户端向主节点 B 发送一条写命令。
- 主节点 B 执行写命令，并向客户端返回命令回复。
- 主节点 B 将刚刚执行的写命令复制给它的从节点 B1 、 B2 和 B3 。

如你所见， 主节点对命令的复制工作发生在返回命令回复之后， 因为如果每次处理命令请求都需要等待复制操作完成的话， 那么主节点处理命令请求的速度将极大地降低 —— 我们必须在性能和一致性之间做出权衡。

如果真的有必要的话， Redis 集群可能会在将来提供同步地（synchronou）执行写命令的方法。

Redis 集群另外一种可能会丢失命令的情况是， 集群出现网络分裂（[network partition](http://en.wikipedia.org/wiki/Network_partition)）， 并且一个客户端与至少包括一个主节点在内的少数（minority）实例被孤立。

举个例子， 假设集群包含 A 、 B 、 C 、 A1 、 B1 、 C1 六个节点， 其中 A 、B 、C 为主节点， 而 A1 、B1 、C1 分别为三个主节点的从节点， 另外还有一个客户端 Z1 。

假设集群中发生网络分裂， 那么集群可能会分裂为两方， 大多数（majority）的一方包含节点 A 、C 、A1 、B1 和 C1 ， 而少数（minority）的一方则包含节点 B 和客户端 Z1 。

在网络分裂期间， 主节点 B 仍然会接受 Z1 发送的写命令：

- 如果网络分裂出现的时间很短， 那么集群会继续正常运行；
- 但是， 如果网络分裂出现的时间足够长， 使得大多数一方将从节点 B1 设置为新的主节点， 并使用 B1 来代替原来的主节点 B ， 那么 Z1 发送给主节点 B 的写命令将丢失。

注意， 在网络分裂出现期间， 客户端 Z1 可以向主节点 B 发送写命令的最大时间是有限制的， 这一时间限制称为**节点超时时间**（node timeout）， 是 Redis 集群的一个重要的配置选项：

- 对于大多数一方来说， 如果一个主节点未能在节点超时时间所设定的时限内重新联系上集群， 那么集群会将这个主节点视为下线， 并使用从节点来代替这个主节点继续工作。
- 对于少数一方， 如果一个主节点未能在节点超时时间所设定的时限内重新联系上集群， 那么它将停止处理写命令， 并向客户端报告错误。

## 创建集群

## 集群的客户端

Redis 集群现阶段的一个问题是客户端实现很少。 以下是一些我知道的实现：

- `redis-rb-cluster` 是我（@antirez）编写的 Ruby 实现， 用于作为其他实现的参考。 该实现是对 `redis-rb` 的一个简单包装， 高效地实现了与集群进行通讯所需的最少语义（semantic）。
- `redis-py-cluster` 看上去是 `redis-rb-cluster` 的一个 Python 版本， 这个项目有一段时间没有更新了（最后一次提交是在六个月之前）， 不过可以将这个项目用作学习集群的起点。
- 流行的 Predis 曾经对早期的 Redis 集群有过一定的支持， 但我不确定它对集群的支持是否完整， 也不清楚它是否和最新版本的 Redis 集群兼容 （因为新版的 Redis 集群将槽的数量从 4k 改为 16k 了）。
- Redis `unstable` 分支中的 `redis-cli` 程序实现了非常基本的集群支持， 可以使用命令 `redis-cli -c` 来启动。

## 对集群进行重新分片

## 添加新节点到集群

根据新添加节点的种类， 我们需要用两种方法来将新节点添加到集群里面：

- 如果要添加的新节点是一个主节点， 那么我们需要创建一个空节点（empty node）， 然后将某些哈希桶移动到这个空节点里面。
- 另一方面， 如果要添加的新节点是一个从节点， 那么我们需要将这个新节点设置为集群中某个节点的复制品（replica）。

本节将对以上两种情况进行介绍， 首先介绍主节点的添加方法， 然后再介绍从节点的添加方法。

无论添加的是那种节点， 第一步要做的总是添加一个空节点。

## 移除节点

# 事务（transaction）

Redis事务

事务的原理是将一个事务范围内的若干命令发送给Redis，然后再让Redis依次执行这些命令。

事务的生命周期：

1. 使用MULTI开启一个事务

2. 在开启事务的时候，每次操作的命令将会被插入到一个队列中，同时这个命令并不会被真的执行

3. EXEC命令进行提交事务

![](http://img.topjavaer.cn/img/redis-multi.jpg)

一个事务范围内某个命令出错不会影响其他命令的执行，不保证原子性：

```java
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set a 1
QUEUED
127.0.0.1:6379> set b 1 2
QUEUED
127.0.0.1:6379> set c 3
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR syntax error
3) OK
```

**WATCH命令**

`WATCH`命令可以监控一个或多个键，一旦其中有一个键被修改，之后的事务就不会执行（类似于乐观锁）。执行`EXEC`命令之后，就会自动取消监控。

```java
127.0.0.1:6379> watch name
OK
127.0.0.1:6379> set name 1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set name 2
QUEUED
127.0.0.1:6379> set gender 1
QUEUED
127.0.0.1:6379> exec
(nil)
127.0.0.1:6379> get gender
(nil)
```

比如上面的代码中：

1. `watch name`开启了对`name`这个`key`的监控
2. 修改`name`的值
3. 开启事务a
4. 在事务a中设置了`name`和`gender`的值
5. 使用`EXEC`命令进提交事务
6. 使用命令`get gender`发现不存在，即事务a没有执行

使用`UNWATCH`可以取消`WATCH`命令对`key`的监控，所有监控锁将会被取消。

## Redis事务支持隔离性吗？

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

## Redis事务保证原子性吗，支持回滚吗？

Redis单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。

MULTI 、 EXEC 、 DISCARD 和 WATCH 是 Redis 事务的基础。

事务可以一次执行多个命令， 并且带有以下两个重要的保证：

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。
  
  EXEC 命令负责触发并执行事务中的所有命令：
  
  - 如果客户端在使用 MULTI 开启了一个事务之后，却因为断线而没有成功执行 EXEC ，那么事务中的所有命令都不会被执行。
  - 另一方面，如果客户端成功在开启事务之后执行 EXEC ，那么事务中的所有命令都会被执行。
  
  当使用 AOF 方式做持久化的时候， Redis 会使用单个 `write(2)` 命令将事务写入到磁盘中。
  
  然而，如果 Redis 服务器因为某些原因被管理员杀死，或者遇上某种硬件故障，那么可能只有部分事务命令会被成功写入到磁盘中。
  
  如果 Redis 在重新启动时发现 AOF 文件出了这样的问题，那么它会退出，并汇报一个错误。
  
  使用 `redis-check-aof` 程序可以修复这一问题：它会移除 AOF 文件中不完整事务的信息，确保服务器可以顺利启动。

从 2.2 版本开始，Redis 还可以通过乐观锁（optimistic lock）实现 CAS （check-and-set）操作，具体信息请参考文档的后半部分。

## 用法

MULTI 命令用于开启一个事务，它总是返回 OK 。

MULTI 执行之后， 客户端可以继续向服务器发送任意多条命令， 这些命令不会立即被执行， 而是被放到一个队列中， 当 EXEC 命令被调用时， 所有队列中的命令才会被执行。

另一方面， 通过调用 DISCARD ， 客户端可以清空事务队列， 并放弃执行事务。

以下是一个事务例子， 它原子地增加了 `foo` 和 `bar` 两个键的值：

```
> MULTI
OK

> INCR foo
QUEUED

> INCR bar
QUEUED

> EXEC
1) (integer) 1
2) (integer) 1
```

EXEC 命令的回复是一个数组， 数组中的每个元素都是执行事务中的命令所产生的回复。 其中， 回复元素的先后顺序和命令发送的先后顺序一致。

当客户端处于事务状态时， 所有传入的命令都会返回一个内容为 `QUEUED` 的状态回复（status reply）， 这些被入队的命令将在 EXEC 命令被调用时执行。

## 事务中的错误

使用事务时可能会遇上以下两种错误：

- 事务在执行 EXEC 之前，入队的命令可能会出错。比如说，命令可能会产生语法错误（参数数量错误，参数名错误，等等），或者其他更严重的错误，比如内存不足（如果服务器使用 `maxmemory` 设置了最大内存限制的话）。
- 命令可能在 EXEC 调用之后失败。举个例子，事务中的命令可能处理了错误类型的键，比如将列表命令用在了字符串键上面，诸如此类。

对于发生在 EXEC 执行之前的错误，客户端以前的做法是检查命令入队所得的返回值：如果命令入队时返回 `QUEUED` ，那么入队成功；否则，就是入队失败。如果有命令在入队时失败，那么大部分客户端都会停止并取消这个事务。

不过，从 Redis 2.6.5 开始，服务器会对命令入队失败的情况进行记录，并在客户端调用 EXEC 命令时，拒绝执行并自动放弃这个事务。

在 Redis 2.6.5 以前， Redis 只执行事务中那些入队成功的命令，而忽略那些入队失败的命令。 而新的处理方式则使得在流水线（pipeline）中包含事务变得简单，因为发送事务和读取事务的回复都只需要和服务器进行一次通讯。

至于那些在 EXEC 命令执行之后所产生的错误， 并没有对它们进行特别处理： 即使事务中有某个/某些命令在执行时产生了错误， 事务中的其他命令仍然会继续执行。

从协议的角度来看这个问题，会更容易理解一些。 以下例子中， LPOP key 命令的执行将出错， 尽管调用它的语法是正确的：

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.

MULTI
+OK

SET a 3
abc

+QUEUED
LPOP a

+QUEUED
EXEC

*2
+OK
-ERR Operation against a key holding the wrong kind of value
```

EXEC 返回两条批量回复（bulk reply）： 第一条是 `OK` ，而第二条是 `-ERR` 。 至于怎样用合适的方法来表示事务中的错误， 则是由客户端自己决定的。

最重要的是记住这样一条， 即使事务中有某条/某些命令执行失败了， 事务队列中的其他命令仍然会继续执行 —— Redis 不会停止执行事务中的命令。

以下例子展示的是另一种情况， 当命令在入队时产生错误， 错误会立即被返回给客户端：

```
MULTI
+OK

INCR a b c
-ERR wrong number of arguments for 'incr' command
```

因为调用 INCR key 命令的参数格式不正确， 所以这个 INCR key 命令入队失败。

## 为什么 Redis 不支持回滚（roll back）

如果你有使用关系式数据库的经验， 那么 “Redis 在事务失败时不进行回滚，而是继续执行余下的命令”这种做法可能会让你觉得有点奇怪。

以下是这种做法的优点：

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 INCR key 命令将键的值加上 `1` ， 却不小心加上了 `2` ， 又或者对错误类型的键执行了 INCR key ， 回滚是没有办法处理这些情况的。

鉴于没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。

## 放弃事务

当执行 DISCARD 命令时， 事务会被放弃， 事务队列会被清空， 并且客户端会从事务状态中退出：

```
redis> SET foo 1
OK

redis> MULTI
OK

redis> INCR foo
QUEUED

redis> DISCARD
OK

redis> GET foo
"1"
```

## 使用 check-and-set 操作实现乐观锁

WATCH 命令可以为 Redis 事务提供 check-and-set （CAS）行为。

被 WATCH 的键会被监视，并会发觉这些键是否被改动过了。 如果有至少一个被监视的键在 EXEC 执行之前被修改了， 那么整个事务都会被取消， EXEC 返回空多条批量回复（null multi-bulk reply）来表示事务已经失败。

举个例子， 假设我们需要原子性地为某个值进行增 `1` 操作（假设 INCR key 不存在）。

首先我们可能会这样做：

```
val = GET mykey
val = val + 1
SET mykey $val
```

上面的这个实现在只有一个客户端的时候可以执行得很好。 但是， 当多个客户端同时对同一个键进行这样的操作时， 就会产生竞争条件。

举个例子， 如果客户端 A 和 B 都读取了键原来的值， 比如 `10` ， 那么两个客户端都会将键的值设为 `11` ， 但正确的结果应该是 `12` 才对。

有了 WATCH ， 我们就可以轻松地解决这类问题了：

```
WATCH mykey

val = GET mykey
val = val + 1

MULTI
SET mykey $val
EXEC
```

使用上面的代码， 如果在 WATCH 执行之后， EXEC 执行之前， 有其他客户端修改了 `mykey` 的值， 那么当前客户端的事务就会失败。 程序需要做的， 就是不断重试这个操作， 直到没有发生碰撞为止。

这种形式的锁被称作乐观锁， 它是一种非常强大的锁机制。 并且因为大多数情况下， 不同的客户端会访问不同的键， 碰撞的情况一般都很少， 所以通常并不需要进行重试。

## 了解 WATCH

WATCH 使得 EXEC 命令需要有条件地执行： 事务只能在所有被监视键都没有被修改的前提下执行， 如果这个前提不能满足的话，事务就不会被执行。

Note

如果你使用 WATCH 监视了一个带过期时间的键， 那么即使这个键过期了， 事务仍然可以正常执行， 关于这方面的详细情况，请看这个帖子： http://code.google.com/p/redis/issues/detail?id=270

WATCH 命令可以被调用多次。 对键的监视从 WATCH 执行之后开始生效， 直到调用 EXEC 为止。

用户还可以在单个 WATCH 命令中监视任意多个键， 就像这样：

```
redis> WATCH key1 key2 key3
OK
```

当 EXEC 被调用时， 不管事务是否成功执行， 对所有键的监视都会被取消。

另外， 当客户端断开连接时， 该客户端对键的监视也会被取消。

使用无参数的 UNWATCH 命令可以手动取消对所有键的监视。 对于一些需要改动多个键的事务， 有时候程序需要同时对多个键进行加锁， 然后检查这些键的当前值是否符合程序的要求。 当值达不到要求时， 就可以使用 UNWATCH 命令来取消目前对键的监视， 中途放弃这个事务， 并等待事务的下次尝试。

## 使用 WATCH 实现 ZPOP

WATCH 可以用于创建 Redis 没有内置的原子操作。

举个例子， 以下代码实现了原创的 `ZPOP` 命令， 它可以原子地弹出有序集合中分值（score）最小的元素：

```
WATCH zset
element = ZRANGE zset 0 0
MULTI
    ZREM zset element
EXEC
```

程序只要重复执行这段代码， 直到 EXEC 的返回值不是空多条回复（null multi-bulk reply）即可。

## Redis 脚本和事务

从定义上来说， Redis 中的脚本本身就是一种事务， 所以任何在事务里可以完成的事， 在脚本里面也能完成。 并且一般来说， 使用脚本要来得更简单，并且速度更快。

因为脚本功能是 Redis 2.6 才引入的， 而事务功能则更早之前就存在了， 所以 Redis 才会同时存在两种处理事务的方法。

不过我们并不打算在短时间内就移除事务功能， 因为事务提供了一种即使不使用脚本， 也可以避免竞争条件的方法， 而且事务本身的实现并不复杂。

不过在不远的将来， 可能所有用户都会只使用脚本来实现事务也说不定。 如果真的发生这种情况的话， 那么我们将废弃并最终移除事务功能。

# 使用pipelining加速Redis查询

## Request/Response protocols and RTT

Redis是一种基于客户端-服务端模型以及请求/响应协议的TCP服务。

这意味着通常情况下一个请求会遵循以下步骤：

客户端向服务端发送一个查询请求，并监听Socket返回，通常是以阻塞模式，等待服务端响应。
服务端处理命令，并将结果返回给客户端。
因此，例如下面是4个命令序列执行情况：

- *Client:* INCR X
- *Server:* 1
- *Client:* INCR X
- *Server:* 2
- *Client:* INCR X
- *Server:* 3
- *Client:* INCR X
- *Server:* 4

客户端和服务器通过网络进行连接。这个连接可以很快（loopback接口）或很慢（建立了一个多次跳转的网络连接）。无论网络延如何延时，数据包总是能从客户端到达服务器，并从服务器返回数据回复客户端。

这个时间被称之为 RTT (Round Trip Time - 往返时间). 当客户端需要在一个批处理中执行多次请求时很容易看到这是如何影响性能的（例如添加许多元素到同一个list，或者用很多Keys填充数据库）。例如，如果RTT时间是250毫秒（在一个很慢的连接下），即使服务器每秒能处理100k的请求数，我们每秒最多也只能处理4个请求。

如果采用loopback接口，RTT就短得多（比如我的主机ping 127.0.0.1只需要44毫秒），但在一次批量写入操作中，仍然是一笔很大的开销。

幸运的是有一种方法可以改善这种情况。

## Redis 管道化

一次请求/响应服务器能实现处理新的请求即使旧的请求还未被响应。这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。

这就是管道（pipelining），是一种几十年来广泛使用的技术。例如许多POP3协议已经实现支持这个功能，大大加快了从服务器下载新邮件的过程。

Redis很早就支持管道（pipelining）技术，因此无论你运行的是什么版本，你都可以使用管道（pipelining）操作Redis。下面是一个使用的例子：

```
$ (printf "PING\r\nPING\r\nPING\r\n"; sleep 1) | nc localhost 6379
+PONG
+PONG
+PONG
```

# 对象

前一章介绍了Redis的主要数据结构，但Redis并没有直接使用这些数据结构来实现键值对数据库， 而是基于这些数据结构创建了一个对象系统<!--more--> ，**这个系统包含字符串对象、列表对象、哈希对象、集合对象和有序集合对象**这五种类型的对象。

使用对象有两个好处：

- 执行命令前，根据对象类型来**判断是否可以执行此命令**。
- 针对不同使用场景，为对象**设置多种不同的数据结构实现**，达到优化的目的。

此外，对象系统还引入了**引用计数实现内存回收机制**，以及**对象共享**。

## 1. 对象的类型与编码

Redis 中的**每个键值对的键和值都是一个对象，每个对象都由一个 `redisObject` 结构表示**， 该结构中和保存数据有关的三个属性分别是 `type` 属性、 `encoding` 属性和 `ptr` 属性：

```C
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 指向底层实现数据结构的指针
    void *ptr;
    ...

} robj;
```

> 结构体的冒号表示位域，表示该变量占用的二进制位数

对象的 `type` 属性记录了**对象的类型**，属性的值如下表所示：

| 类型常量           | 对象的名称  |
|:--------------:|:------:|
| `REDIS_STRING` | 字符串对象  |
| `REDIS_LIST`   | 列表对象   |
| `REDIS_HASH`   | 哈希对象   |
| `REDIS_SET`    | 集合对象   |
| `REDIS_ZSET`   | 有序集合对象 |

对于 Redis 数据库保存的键值对来说， **键总是一个字符串对象**， 而值则可以是上表中的其中一个。

所以，当我们称呼一个数据库键为“字符串键”时， 我们指的是“这个数据库键所对应的**值**为字符串对象”；同理，当我们称呼一个键为“列表键”时， 我们指的是“这个数据库键所对应的**值**为列表对象”。

---

对象的 `ptr` 指针指向对象的底层实现数据结构， 而这些数据结构由对象的 `encoding` 属性决定。`encoding` 属性如下表所示：

| 编码常量                        | 编码所对应的底层数据结构        |
|:---------------------------:|:-------------------:|
| `REDIS_ENCODING_INT`        | `long` 类型的整数        |
| `REDIS_ENCODING_EMBSTR`     | `embstr` 编码的简单动态字符串 |
| `REDIS_ENCODING_RAW`        | 简单动态字符串             |
| `REDIS_ENCODING_HT`         | 字典                  |
| `REDIS_ENCODING_LINKEDLIST` | 双端链表                |
| `REDIS_ENCODING_ZIPLIST`    | 压缩列表                |
| `REDIS_ENCODING_INTSET`     | 整数集合                |
| `REDIS_ENCODING_SKIPLIST`   | 跳跃表和字典              |

## 2. 字符串对象

### 2.1 编码方式

字符串对象的编码可以是 `int` 、 `raw` 或者 `embstr` 。

（1）如果一个字符串对象保存的是**整数值**， 并且这个整数值可以用 `long` 类型来表示， 那么字符串对象会**将整数值保存在字符串对象结构的 `ptr`属性里面**（将 `void*` 转换成 `long` ）， 并将字符串对象的编码设置为 `int` 。

（2）如果字符串对象保存的是一个字符串值， 并且这个字符串值的长度**大于** `39` 字节， 那么字符串对象将**使用一个简单动态字符串（SDS）来保存这个字符串值**， 并将对象的编码设置为 `raw` 。

（3）如果字符串对象保存的是一个字符串值， 并且这个字符串值的长度**小于等于** `39` 字节， 那么字符串对象将使用 `embstr` 编码的方式来保存这个字符串值。

`embstr` 编码是**专门用于保存短字符串**的一种优化编码方式。这种编码和 `raw` 编码一样， 都使用 `redisObject` 结构和 `sdshdr` 结构来表示字符串对象，区别在于：

- `raw` 编码会**调用两次内存分配**函数来**分别**创建 `redisObject` 结构和 `sdshdr` 结构。
- `embstr` 编码则通过**调用一次**内存分配函数来分配一块**连续的空间**。

由于减少了内存分配的次数，以及将零散的内存整合到一起，这种编码的字符串对象比起 `raw` 编码能够**更好地利用缓存带来的优势**。

如果保存浮点数，则会先转化为字符串类型保存。比如保存3.14就会先转化为`"3.14"`。下表是值和对应的编码类型

| 值                             | 编码                |
|:-----------------------------:|:-----------------:|
| 可以用 `long` 类型保存的整数。           | `int`             |
| 可以用 `long double` 类型保存的浮点数。   | `embstr` 或者 `raw` |
| 字符串值，长度太大没办法用 `long` 类型表示的整数。 | `embstr` 或者 `raw` |

### 2.2 编码转换

编码之间也会有**互相转换**的情况。对于 `int` 编码的字符串对象来说，如果因为命令导致这个对象保存的不再是整数值， 而是一个字符串值， 那么字符串对象的编码将从 `int` 变为 `raw` 。

```C
redis> SET number 10086
OK

redis> OBJECT ENCODING number
"int"

redis> APPEND number " is a good number!"
(integer) 23

redis> GET number
"10086 is a good number!"

redis> OBJECT ENCODING number
"raw"
```

 因为 Redis 没有为 `embstr` 编码的字符串对象编写任何相应的修改程序 ， 所以 `embstr` 编码的字符串对象**实际上是只读的**。当修改`embstr` 编码的字符串对象， 程序会**先将对象的编码从 `embstr` 转换成 `raw` ， 然后再执行修改命令**。

```C
redis> SET msg "hello world"
OK

redis> OBJECT ENCODING msg
"embstr"

redis> APPEND msg " again!"
(integer) 18

redis> OBJECT ENCODING msg
"raw"
```

## 3. 列表对象

### 3.1 编码方式

列表对象的编码可以是 `ziplist` 或者 `linkedlist`。

如同前面提到的，**压缩列表每个节点(entry)只保存一个列表元素**。下面例子中，我们输入数字1，字符"three"和数字5，

```C
redis> RPUSH numbers 1 "three" 5
(integer) 3
```

如果使用的不是 `ziplist` 编码， 而是 `linkedlist`双端链表 编码， 那么 

这其实是一个**嵌套编码**，Redis使用了一个带有 `StringObject` 来表示一个字符串对象，编码方式如同上面提到的那三种。**如果编码对象时字符串值**

### 3.2 编码转换

当列表对象可以同时满足以下两个条件时， 列表对象使用 `ziplist` 编码：

1. 列表对象保存的**所有**字符串元素的长度都小于 `64` 字节；
2. 列表对象保存的元素数量小于 `512` 个；

当上述条件任意一个不满足时，就会执行**转换操作**： 原本保存在压缩列表里的所有列表元素都会被转移并保存到双端链表里面， 对象的编码也会从 `ziplist` 变为 `linkedlist` 。

## 4. 哈希对象

### 4.1 编码方式

哈希对象的编码可以是 `ziplist` 或者 `hashtable` 。

`ziplist` 编码时，每当有新的键值对要加入到哈希对象时， 程序会**先将保存了键**的压缩列表节点推入到压缩列表表尾， 然后**再将保存了值**的压缩列表节点推入到压缩列表表尾， 因此：

- 保存了同一键值对的两个节点总是**紧挨在一起， 键前值后。**
- **先添加**到哈希对象中的键值对会被放在压缩列表的**表头方向**， 而**后来添加**到哈希对象中的键值对会被放在压缩列表的**表尾方向**。

比如：

```C
redis> HSET profile name "Tom"
(integer) 1

redis> HSET profile age 25
(integer) 1

redis> HSET profile career "Programmer"
(integer) 1
```

`hashtable` 编码时， 哈希对象中的每个键值对都使用一个字典键值对来保存：

- 字典的每个键都是一个**字符串对象**， 对象中保存了键值对的键；
- 字典的每个值都是一个**字符串对象**， 对象中保存了键值对的值。

### 4.2 编码转换

当哈希对象可以同时满足以下两个条件时， 哈希对象使用 `ziplist` 编码：

1. 哈希对象保存的所有键值对的键和值的字符串长度都小于 `64` 字节；
2. 哈希对象保存的键值对数量小于 `512` 个；

和列表对象一样，不满足条件时原本保存在压缩列表里的所有键值对都会被转移并保存到字典里面， 对象的编码也会从 `ziplist` 变为 `hashtable` 。

## 5. 集合对象

Redis中集合和列表结构相似，但**集合具有唯一性，列表不具有**。

### 5.1 编码方式

集合对象的编码可以是 `intset` 或者 `hashtable` 。

`intset` 编码时，元素将被密集得堆叠在位上，比如

```C
redis> SADD numbers 1 3 5
(integer) 3
```

另一方面， `hashtable` 编码的集合对象使用字典作为底层实现， 字典的**每个键都是一个字符串对象**， 每个字符串对象包含了一个集合元素， 而**字典的值则全部被设置为 `NULL` 。**

### 5.2 编码的转换

当集合对象可以同时满足以下两个条件时， 对象使用 `intset` 编码：

1. 集合对象保存的所有元素**都是整数值**；
2. 集合对象保存的元素数量不超过 `512` 个；

当使用 `intset` 编码所需的两个条件的任意一个不能被满足时， 对象的编码转换操作就会被执行： 原本保存在整数集合中的所有元素都会被转移并保存到字典里面， 并且对象的编码也会从 `intset` 变为 `hashtable` 。

## 6. 有序集合对象

### 6.1 编码方式

有序集合的编码可以是 `ziplist` 或者 `skiplist` 。

`ziplist` 编码的有序集合对象使用压缩列表作为底层实现， 每个集合元素使用两个紧挨在一起的压缩列表节点来保存， 第一个节点保存元素的成员（member）， 而第二个元素则保存元素的分值（score）。

```C
redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
(integer) 3
```

`skiplist` 编码的有序集合对象使用 `zset` 结构作为底层实现， 一个 `zset` 结构同时**包含一个字典和一个跳跃表**：

```C
typedef struct zset {
    zskiplist *zsl;
    dict *dict;
} zset;
```

起作用主要是跳跃表，字典是辅助加速用。**字典的键记录了元素的成员，而值则保存了元素的分值**。通过字典，能实现$O(1)$复杂度的查找给定成员分值。

有序集合**每个元素的成员都是一个字符串对象**， 而每个元素的**分值都是一个 `double` 类型的浮点数**。

虽然 `zset` 结构同时使用跳跃表和字典来保存有序集合元素， 但这两种数据结构都会**通过指针来共享相同元素的成员和分值**， 所以同时使用跳跃表和字典来保存集合元素不会产生任何重复成员或者分值， 也不会因此而浪费额外的内存。

### 6.2 编码转换

当有序集合对象可以同时满足以下两个条件时， 对象使用 `ziplist` 编码：

1. 有序集合保存的元素数量小于 `128` 个；
2. 有序集合保存的所有元素成员的长度都小于 `64` 字节；

不能满足以上两个条件的有序集合对象将使用 `skiplist` 编码。

## 7. 内存回收、对象共享和空转时长

对象中包括了一个引用计数器：

```C
typedef struct redisObject {
    // ...
    // 引用计数
    int refcount;
    // ...
} robj;
```

对象的引用计数信息会随着对象的使用状态而不断变化：

- 在创建一个新对象时， 引用计数的值会被初始化为 `1` ；
- 当对象被引用时，计数值+1；
- 当对象不再被引用时，计数值-1；
- 当对象的引用计数值变为 `0` 时， 对象所占用的内存会被释放。

| 函数              | 作用                                                    |
|:---------------:|:-----------------------------------------------------:|
| `incrRefCount`  | 将对象的引用计数值增一。                                          |
| `decrRefCount`  | 将对象的引用计数值减一， 当对象的引用计数值等于 `0` 时， 释放对象。                 |
| `resetRefCount` | 将对象的引用计数值设置为 `0` ， 但并不释放对象， 这个函数通常在需要重新设置对象的引用计数值时使用。 |

---

通过引用机制，还能实现对象共享。共享**只针对整数值对象，不针对包含字符串的对象。**

 假设键 A 创建了一个包含整数值 `100` 的字符串对象作为值对象，

如果这时键 B 也要创建一个同样保存了整数值 `100` 的字符串对象作为值对象， 那么服务器有以下两种做法：

1. 为键 B 新创建一个包含整数值 `100` 的字符串对象；
2. 让键 A 和键 B 共享同一个字符串对象；

以上两种方法很明显是第二种方法更节约内存。在 Redis 中， 让多个键共享同一个值对象需要执行以下两个步骤：

1. 将数据库键的值指针指向一个现有的值对象；

2. 将被共享的值对象的引用计数增一。
   
   Redis 会在初始化服务器时， 创建一万个**字符串对象**， 这些对象包含了从 `0` 到 `9999` 的所有整数值， 当服务器需要用到值为 `0`到 `9999` 的字符串对象时， 服务器就会使用这些共享对象， 而不是新创建对象。

为什么 Redis 不共享包含字符串的对象？

判断是否共享时要检验**共享对象和目标对象是否相同**。复杂度如下，

| 共享对象             | 复杂度      |
|:----------------:|:--------:|
| 保存整数值字符串对象       | $O( 1)$  |
| 保存字符串值的字符串对象     | $O(N)$   |
| 包含了多个值的对象（列表或哈希） | $O(N^2)$ |

---

`redisObject` 结构包含的最后一个属性为 `lru` 属性， 该属性记录了对象最后一次被命令程序访问的时间：

```C
typedef struct redisObject {
    // ...
    unsigned lru:22;
    // ...

} robj;
```

`OBJECT IDLETIME` 命令可以打印出给定键的空转时长， 这一空转时长就是通过将当前时间减去键的值对象的 `lru` 时间计算得出的：

```C
redis> SET msg "hello world"
OK

# 等待一小段时间
redis> OBJECT IDLETIME msg
(integer) 20
```

注意`OBJECT IDLETIME` 命令的实现是特殊的， 这个命令在访问键的值对象时， **不会修改值对象的 `lru` 属性**。这类似于`std::weak_ptr`的作用。

当内存满时，空转时长较长的键会被优先释放。

# 缓存穿透

#### 解决办法

1.缓存空对象：如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。

缓存空对象带来的问题：

```
1.空值做了缓存，意味着缓存中存了更多的键，需要更多的内存空间，比较有效的方法是针对这类数据设置一个较短的过期时间，让其自动剔除。

2.缓存和存储的数据会有一段时间窗口的不一致，可能会对业务有一定影响。例如:过期时间设置为 5分钟，如果此时存储添加了这个数据，那此段时间就会出现缓存和存储数据的不一致，此时可以利用消息系统或者其他方式清除掉缓存层中的空对象。
```

2.布隆过滤器：将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力。

### 缓存雪崩

​    如果缓存集中在一段时间内失效，发生大量的缓存穿透，所有的查询都落在数据库上，造成了缓存雪崩。

#### 解决办法

1.加锁排队：在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量.比如对某个 key 只允许一个线程查询数据和写缓存，其他线程等待；

2.数据预热：可以通过缓存 reload 机制，预先去更新缓存，再即将发生大并发访问前手动触发加载缓存不同的 key，设置不同的过期时间，让缓存失效的时间点尽量        均匀；

3.做二级缓存，或者双缓存策略：Cache1 为原始缓存，Cache2 为拷贝缓存，Cache1 失效时，可以访问 Cache2，Cache1 缓存失效时间设置为短期，Cache2 设置为长期。

4.在缓存的时候给过期时间加上一个随机值，这样就会大幅度的减少缓存在同一时间过期。

# 保证缓存和数据库数据的一致性

如题，现在很多架构都采用了Redis+MySQL来进行存储，但是由于多方面的原因，总会导致Redis和MySQL之间出现数据的不一致性.

例如如果一个事务执行失败回滚了，但是如果采取了先写Redis的方式，就会造成Redis和MySQL数据库的不一致，再比如说，一个事务写入了MySQL，但是此时还未写入Redis，如果这时候有用户访问Redis，则此时就会出现数据不一致。

为了解决这些问题，本文将着重讨论，如何保证MySQL和Redis之间存在一个合理的数据一致性方案。

### 1.分别处理

针对某些对数据一致性要求不是特别高的情况下，可以将这些数据放入Redis，请求来了直接查询Redis，例如近期回复、历史排名这种实时性不强的业务。而针对那些强实时性的业务，例如虚拟货币、物品购买件数等等，
则直接穿透Redis至MySQL上，等到MySQL上写入成功，再同步更新到Redis上去。这样既可以起到Redis的分流大量查询请求的作用，又保证了关键数据的一致性。

### 2.高并发情况下

此时如果写入请求较多，则直接写入Redis中去，然后间隔一段时间，批量将所有的写入请求，刷新到MySQL中去； 如果此时写入请求不多，则可以在每次写入Redis，都立刻将该命令同步至MySQL中去。这两种方法有利有弊，需要根据不同的场景来权衡.

### 3.基于订阅binlog的同步机制

阿里巴巴的一款开源框架canal，提供了一种发布/ 订阅模式的同步机制，通过该框架我们可以对MySQL的binlog进行订阅，这样一旦MySQL中产生了新的写入、更新、删除等操作，就可以把binlog相关的消息推送至Redis，
Redis再根据binlog中的记录，对Redis进行更新。值得注意的是，binlog需要手动打开，并且不会记录关于MySQL查询的命令和操作。
其实这种机制，很类似MySQL的主从备份机制，因为MySQL的主备也是通过binlog来实现的数据一致性。
而canal正是模仿了slave数据库的备份请求，使得Redis的数据更新达到了相同的效果。
如下图就可以看到Slave数据库中启动了2个线程，一个是MySQL SQL线程，这个线程跟Matser数据库中起的线程是一样的，
负责MySQL的业务率执行，而另外一个线程就是MySQL的I/O线程，这个线程的主要作用就是同步Master 数据库中的binlog，达到数据备份的效果。而binlog就可以理解为一堆SQL语言组成的日志。

# Redis 持久化

持久化就是把内存的数据写到磁盘中去，防止服务宕机了内存数据丢失。Redis 提供了两种持久化方式：RDB（默认） 和 AOF。

## RDB

RDB 是 Redis DataBase 的缩写。按照一定的时间周期策略把内存的数据以快照的形式保存到硬盘的二进制文件。即 Snapshot 快照存储，对应产生的数据文件为 dump.rdb，通过配置文件中的 save 参数来定义快照的周期。核心函数：rdbSave（生成 RDB 文件）和 rdbLoad（从文件加载内存）两个函数。

## AOF

AOF 是 Append-only file 的缩写。Redis会将每一个收到的写命令都通过 Write 函数追加到文件最后，类似于 MySQL 的 binlog。当 Redis 重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。每当执行服务器（定时）任务或者函数时，flushAppendOnlyFile 函数都会被调用， 这个函数执行以下两个工作：

WRITE：根据条件，将 aof_buf 中的缓存写入到 AOF 文件；

SAVE：根据条件，调用 fsync 或 fdatasync 函数，将 AOF 文件保存到磁盘中。

#### RDB 和 AOF 的区别：

```
1.AOF 文件比 RDB 更新频率高，优先使用 AOF 还原数据；

2.AOF比 RDB 更安全也更大；

3.RDB 性能比 AOF 好；

4.如果两个都配了优先加载 AOF。
```

title: Redis设计与实现4-RDB和AOF持久化
category:

- 数据库
  tags:
  - 计算机网络
  - 数据库
  - Redis
  - 读书笔记
    mathjax: true
    date: 2020-01-04 14:10:19

---

持久化的意思是将数据永久保存在磁盘中。Redis采用RDB和AOF两种策略。<!--more-->

## 1. RDB持久化

将服务器中的非空数据库以及它们的键值对统称为**数据库状态**。下图三个非空数据库，以及其中的键值对就是该服务器的数据库状态。

在Redis中，只有将数据保存在内存磁盘里才会永久保存，**如果服务器进程退出，服务器中的数据库状态就会消失。**为了解决这个问题，Redis提供了RDB持久化功能，这个功能可以将Redis在内存中的数据库状态保存到磁盘里面。

RDB持久化产生的RDB文件(Redis Database)是一个**经过压缩的二进制文件，该文件可以被还原为数据库状态**，所以即使服务器停机，服务器的数据还是被安全保存在硬盘中。

## 1.1 RDB文件的创建与载入

有两个Redis命令可以用于生成RDB文件，一个是**SAVE**，另一个是**BGSAVE **(BackGround SAVE)。SAVE命令会**阻塞Redis服务器进程**，直到RDB文件创建完毕为止，在服务器进程阻塞期间，服务器不能处理任何命令请求：

```C
redis> SAVE //等待直到RDB文件创建完毕
OK
```

而BGSAVE命令会**增加一个子进程**，负责创建RDB文件。

```C
redis> BGSAVE //派生子进程，并由子进程创建RDB文件
Background saving started
```

BGSAVE执行时，会阻止SAVE、其他BGSAVE和BGREWRITEAOF这三个命令执行，防止竞争。

创建RDB文件的实际工作由`rdb.c/rdbSave`函数完成，SAVE命令和BGSAVE命令会以不同的方式调用这个函数。

Redis并没有载入RDB文件的命令，只要服务器启动时**检测到RDB文件存在，他就会自动载入。

## 1.2 自动间隔性保存

由于BGSAVE可以不阻塞服务器执行，所以我们可以**设置条件**，让服务器每隔一段时间自动保存。举个例子：

```C
save 900 1
save 300 10
save 60 10000
```

这些条件的意思是：900秒内对数据库至少进行了1次修改，300秒内对数据库进行了10次修改....

服务器程序会根据save选项所设置的保存条件，设置服务器状态`redisServer`结构的`saveparams`属性：

```C
struct redisServer 
{ 
    // ... 
    // 记录了保存条件的数组 
    struct saveparam *saveparams; 
    // ...
};
```

`saveparams`属性是一个数组，数组中的每个元素都是一个`saveparam`结构，每个`saveparam`结构都保存了一个save选项设置的保存条件：

```c
struct saveparam 
{ 
    // 秒数 
    time_t seconds; 
    // 修改数 
    int changes;
};
```

除了设置保存条件的saveparams数组外，服务器状态还维持着一个**dirty计数器**，以及一个**lastsave属性**：

- dirty计数器记录自上一次SAVE和BGSAVE以后，服务器对数据库状态进行了多少次修改（包括增删改）
- lastsave则是一个时间戳，记录了上一次执行保存的时间。

当服务器执行修改命令一次以后，dirty计数器就加一。如果是一次性修改多个元素，计数器此时加N

```C
redis->SADD database0 apple orange watermelon
```

Redis的服务器**周期性操作函数serverCron默认每隔100毫秒就会执行一次**，该函数用于对正在运行的服务器进行维护，它的其中一项工作就是检查save选项所设置的保存条件是否已经满足，如果满足的话，就执行BGSAVE命令。

执行完以后，dirty清0，lastsave更新。

## 1.3 RDB文件结构

完整的RDB文件如下，

RDB是一个二进制文件而不是文本文件。

> 广义来说，所有文件都是二进制文件。狭义来说，文本文件是基于字符编码的文件，常见的编码有**ASCII编码，UNICODE编码**等等；二进制文件是基于值编码的文件，也可以理解为**自定义编码。**

- 开头的REDIS占5个字节，这5个字符**用于检查是不是RDB文件。**
- db_version长度为4字节，值被解析为**RDB版本**，比如"0006"就代表第6版。
- database部分包含着**多个数据库的键值对数据**，根据大小不同，长度有所不同。
- EOF占1个字节，结束位标志。
- check_sum是占8字节，保存**校验和**。服务器在载入时会根据读入的实际数据计算出一个数来和校验值比较，以此来检查是否有损坏。

### 1.3.1 database部分

每个非空数据库在RDB文件中都可以保存为`SELECTDB`、`db_number`、`key_value_pairs`三个部分，如图所示。

- `SELECTDB`，1字节，当读取到此值时，程序知道接下来要读入一个数据库号码。
- `db_number`，1、2、5字节，保存数据库号码。
- `key_value_pairs`，保存键值对，包括过期时间。

### 1.3.2 key_value_pairs部分

不带过期时间的键值对在RDB文件中由TYPE、key、value三部分组成，

带有过期时间的键值对在RDB中的结构如下

- EPIRETIME_MS，1字节，告诉程序接下来读取一个以毫秒为单位的过期时间。
- ms，8字节带符号整数，记录一个以毫秒为单位的UNIX时间戳。

### 1.3.3 value部分

**（1）字符串对象**

如果TYPE的值为`REDIS_RDB_TYPE_STRING`，那么value保存的就是一个字符串对象，字符串对象的编码可以是`REDIS_ENCODING_INT`或者`REDIS_ENCODING_RAW`。

如果是INT，则表示对象是一个**长度不超过32位的整数**，保存方式如下：

其中，`ENCODING`的值可以是`REDIS_RDB_ENC_INT8`、`REDIS_RDB_ENC_INT16`或者`REDIS_RDB_ENC_INT32`三个常量的其中一个，它们分别代表RDB文件使用8位、16位或者32位来保存整数值integer。

如果是RAW格式，则说明对象是一个**字符串值**，有压缩和不压缩两种方法来保存。对于没有压缩的字符串，保存格式如下：

压缩后的字符串，保存格式如下：

- REDIS_RDB_ENC_LZF，表明已被LZF算法压缩
- compressed_len，被压缩后的字符串长度
- origin_len，原来的长度
- compressed_string，被压缩后的字符串

**（2）列表对象**

如果TYPE的值为`REDIS_RDB_TYPE_LIST`，那么value保存的就是一个`REDIS_ENCODING_LINKEDLIST`编码的列表对

**每一个列表项都是一个字符串对象**，所以程序会以字符串对象的方式来保存。

**（3）集合对象**

如果TYPE的值为`REDIS_RDB_TYPE_SET`，那么value保存的就是一个`REDIS_ENCODING_HT`编码的集合对象

**（4）哈希表对象**

如果TYPE的值为`REDIS_RDB_TYPE_HASH`，那么value保存的就是一个`REDIS_ENCODING_HT`编码的集合对象，RDB文件保存这种对象的结构如图所示。

哈希表长度为2，第一个键值对，键长度为1的字符串"a"，值为5的字符串"apple"。

**（5）有序集合对象**

如果TYPE的值为`REDIS_RDB_TYPE_ZSET`，那么value保存的就是一个`REDIS_ENCODING_SKIPLIST`编码的有序集合对象，RDB文件保存这种对象的结构如图所示。

大小为2，第一个元素是长度为2的字符串"pi"，分值被转换为长度为4的字符串"3.14"。

**（6）INTSET编码的集合**

如果TYPE的值为`REDIS_RDB_TYPE_SET_INTSET`，那么value保存的就是一个**整数集合对象**，RDB文件保存这种对象的方法是，**先将整数集合转换为字符串对象**，然后将这个字符串对象保存到RDB文件里面。

**（7）ZIPLIST编码的列表、哈希表和有序集合**

如果TYPE的值为`REDIS_RDB_TYPE_LIST_ZIPLIST`、`REDIS_RDB_TYPE_HASH_ZIPLIST`或者`REDIS_RDB_TYPE_ZSET_ZIPLIST`，那么value保存的就是一个**压缩列表对象**，保存策略和上面一一样：先转化为字符串对象。

## 2. AOF持久化

RDB持久化记录的是数据库本身，而AOF(Append Only File)则**记录Redis服务器所执行的写命令**。

假如使用如下命令:

```C
redis> SET msg "hello"
OK
```

则AOF记录形式如下：

```
*2\r\n$6\r\nSELECT\r\n$1\r\n0\r\n
*3\r\n$3\r\nSET\r\n$3\r\nmsg\r\n$5\r\nhello\r\n
```

## 2.1 AOF实现原理

AOF如其名所示，Append Only File，AOF持久化功能的实现可以分为**命令追加（append）、文件写入与同步（sync）**

**（1）命令追加**

如果AOF被打开，则服务器执行完一个命令后，会以协议格式将命令**追加到服务器状态aof_buf缓冲区的结尾**：

```C
struct redisServer 
{ 
    // ...  
    sds aof_buf;     // AOF缓冲区
    // ...
};
```

比如执行了`SET KEY VALUE`后，会将以下协议内容加载到aof_buf缓冲区：

```
*3\r\n$3\r\nSET\r\n$3\r\nKEY\r\n$5\r\nVALUE\r\n
```

**（2）AOF文件的写入与同步**

Redis的服务器进程就是一个**事件循环（loop）**，这个循环中的**文件事件负责接收客户端的命令请求**，**以及向客户端发送命令回复**，而**时间事件则负责执行像`serverCron`函数这样需要定时运行的函数**。

服务器每次结束一个事件循环之前，它都会调用`flushAppendOnlyFile`函数，**考虑是否需要将`aof_buf`缓冲区中的内容写入和保存到AOF文件里面**。

这个函数的行为有服务器配置的`appendfsync`选项来设置，默认为`everysec`：

默认情况下，距离上次同步过了一秒钟，则服务器会将aof_buf内容写入AOF文件中。

## 2.2 AOF文件的载入与数据还原

因为AOF文件里面包含了重建数据库状态所需的所有写命令，所以服务器只要**读入并重新执行一遍AOF文件里面保存的写命令**，就可以还原服务器关闭之前的数据库状态。

AOF还原数据库的步骤如下：

1. 创建一个不带网络连接的**伪客户端（fake client）**：因为Redis的命令只能在客户端上下文中执行。
2. 从AOF中读出一条命令。
3. 使用伪客户端执行被读出的写命令。
4. 重复23步

## 2.3 AOF重写

随着时间的增长，AOF文件的大小将会越来越大。为了解决这个问题，Redis提供了**AOF重写**功能。

重写后，Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个**AOF文件保存的数据库状态完全相同**。

如果要保存一个键值对，我们其实只关心它当前的状态。所以重写策略是：首先**从数据库中读取键现在的值，然后用一条命令去记录键值对**，用到了`aof_rewrite`函数。

比如，对list进行`RPUSH`操作填入"A"、"B"、"C"，然后再`LPOP`一次，我们操作了4次，但其实用`RPUSH list A B`这一条指令就可以代替。

`aof_rewrite`函数包含了大量写入操作，调用时会导致线程被长时间阻塞，所以Redis**将AOF重写放入子进程里**。

----

还有一个问题：子进程AOF重写时，主进程也在写命令，导致两者状态不一致。因此，**Redis服务器设置了一个AOF重写缓冲区**，这个缓冲区在服务器创建子进程之后开始使用，当Redis服务器执行完一个写命令之后，它会**同时**将这个写命令发送给**AOF缓冲区**和**AOF重写缓冲区**。

换句话说，子进程执行AOF期间，服务器进程需要：

- 执行客户端指令
- 将执行后的命令追加到AOF缓冲区
- 将执行后的命令追加到AOF重写缓冲区

---

子进程执行完AOF后，向父进程发送一个信号。父进程接收后：

1. 将AOF重写缓冲区的内容写入AOF文件中，保证一致性。
2. 对新AOF文件改名，原子的(atomic)覆盖现有AOF文件。

在整个AOF后台重写过程中，**只有信号处理函数执行时会对服务器进程（父进程）造成阻塞**，在其他时候，AOF后台重写都不会阻塞父进程，这将AOF重写对服务器性能造成的影响降到了最低。

# 事件

---

title: Redis设计与实现5-事件
category:

- 数据库
  tags:
  - 计算机网络
  - 数据库
  - Redis
  - 读书笔记
    mathjax: true
    date: 2020-01-05 13:17:25

---

Redis是一个**事件驱动程序**，前面提到，服务器需要处理文件事件和时间事件。<!--more-->

- **文件事件**：Redis服务器通过套接字与客户端（或者其他Redis服务器）进行连接，而**文件事件就是服务器对套接字操作的抽象**。
- **时间事件**：些操作会在给定的时间点进行，对这类**定时操作的抽象就是时间事件。**

## 1. 文件事件

Redis基于Reactor模式开发了自己的网络事件处理器：这个处理器被称为**文件事件处理器（file event handler）**

> Reactor模式用于高并发，依靠事件驱动。传统的线程连接中，IO连接后需要等待客户的请求。而事件驱动中，IO可以干别的事情，等客户发来请求后再处理。

在Redis中，

- 文件事件处理器使用I/O多路复用（multiplexing）程序来同时监听多个套接字，并根据套接字目前执行的任务来为套接字关联不同的事件处理器。
- 当被监听的套接字准备好执行连接应答（accept）、读取（read）、写入（write）、关闭（close）等操作时，与操作相对应的文件事件就会产生，这时文件事件处理器就会调用套接字之前关联好的事件处理器来处理这些事件。

虽然文件事件处理器**以单线程方式运**行，但通过使用**I/O多路复用**程序来监听多个套接字，文件事件处理器既实现了高性能的网络通信模型，又可以很好地与Redis服务器中其他**同样以单线程方式运行的模块进行对接**，这保持了**Redis内部单线程设计的简单性。**

## 1.1 构成

文件事件处理器的四个组成部分，它们分别是**套接字**、**I/O多路复用程序**、**文件事件分派器（dispatcher）**，以及**事件处理器**。

前面提到，文件事件是对套接字操作的抽象，**当一个套接字准备好后，就会产生一个文件事件。**

尽管多个文件事件可能会并发地出现，但I/O多路复用程序总是会**将所有产生事件的套接字都放到一个队列里面**，然后通过这个队列，以**有序（sequentially）**、**同步（synchronously）**、**每次一个套接字**的方式向文件事件分派器传送套接字。**只有当上一个套接字处理完毕后，复用程序才会向分派器传送下一个套接字。**

## 1.2 IO多路复用程序的实现

Redis的I/O多路复用程序的所有功能都是**通过包装常见的select、epoll、evport和kqueue这些I/O多路复用函数库来实现的**，每个I/O多路复用函数库在Redis源码中都对应一个单独的文件，比如ae_select.c、ae_epoll.c、ae_kqueue.c。

> ae表示A simple event-driven programming library，一个简单的事件驱动程序库

由于IO复用程序提供了统一的接口，所以**底层实现方法可以互换。**

## 1.3 事件类型

I/O多路复用程序可以**同时**监听多个套接字的`ae.h/AE_READABLE`和`ae.h/AE_WRITABLE`这两种事件，这两类事件和套接字操作之间的对应关系如下：

- 客户端对套接字执行write操作，客户端对服务器的监听套接字执行connect操作。此时套接字对服务器**变为可读状态**，就会产生`AE_READABLE`事件。
- 客户端对套接字执行read操作。此时套接字对服务器变为**可写状态**，就会产生`AR_WRITABLE`事件。

虽然是可以同时处理这两种事件，但**优先处理可写事件。**

## 1.4 事件处理器

事件处理器有很多，最常用的是**通信的连接应答处理器**、**命令请求处理器**和**命令回复处理器**。

**（1）连接应答处理器**

`networking.c/acceptTcpHandler`函数是Redis的连接应答处理器，具体实现为`sys/socket.h/accept`函数的包装。

当Redis服务器进行**初始化**的时候，程序会将**连接应答处理器**和**服务器监听套接字的`AE_READABLE`事件**关联起来，当有客户端用`sys/socket.h/connec`t函数连接服务器监听套接字的时候，**套接字就会产生`AE_READABLE`事件，引发连接应答处理器执行**。

**（2）命令请求处理器**

`networking.c/readQueryFromClient`函数是Redis的命令请求处理器，这个处理器负责从套接字中读入客户端发送的命令请求内容，具体实现为`unistd.h/read`函数的包装。

和上面一样，当客户端**通过连接应答处理器成功连接到服务器后**，服务器会将**客户端套接字的AE_READABLE事件**和**命令请求处理器**关联起来，当客户端向服务器发送命令请求的时候，**套接字就会产生AE_READABLE事件**，**引发命令请求处理器执行**。

**（3）命令回复处理器**

`networking.c/sendReplyToClient`函数是Redis的命令回复处理器，这个处理器**负责将服务器执行命令后得到的命令通过套接字返回给客户端**，具体实现为`unistd.h/write`函数的包装。

当服务器有命令回复需要传送给客户端的时候，服务器会将**客户端套接字的AE_WRITABLE事件**和**命令回复处理器**关联起来，当客户端准备好接收服务器传回的命令回复时，就会**产生AE_WRITABLE事件，引发命令回复处理器执行**。

## 2. 时间事件

Redis时间事件分为两类：

- **定时事件**：程序在指定时间后执行一次。
- **周期性事件**：每隔一段时间就执行，循环往复。

一个时间事件主要由以下三个属性组成：

- **id**：服务器为时间事件创造全局唯一ID作为识别，新事件比旧事件号码要大。
- **when**：毫秒级UNIX时间戳，记录时间事件到达时间。
- **timeProc**：时间事件处理器，到时间后处理事件。

## 2.1 构成

服务器将所有时间事件都放在一个**无序链表**中，每当时间事件执行器运行时，它就**遍历整个链表**，**查找所有已到达的时间事件**，并调用相应的事件处理器。注意，我们说保存时间事件的链表为无序链表，指的不是链表不按ID排序，而是说，**该链表不按when属性的大小排序**。

## 2.2 API

`ae.c/aeCreateTimeEvent`函数接受一个毫秒数milliseconds和一个时间事件处理器proc作为参数，**将一个新的时间事件添加到服务器**。

`ae.c/aeDeleteFileEvent`函数接受一个时间事件ID作为参数，然后从服务器中**删除**该ID所对应的时间事件。

`ae.c/aeSearchNearestTimer`函数返回到达时间距离当前时间最接近的那个时间事件。

`ae.c/processTimeEvents`函数是时间事件的执行器，这个函数会**遍历所有已到达的时间事件**，并调**用这些事件的处理器**。已到达指的是，时间事件的when属性记录的UNIX时间戳等于或小于当前时间的UNIX时间戳。

## 2.3 severCron函数

持续运行的Redis服务器需要**定期**对自身的资源和状态进行检查和调整，这些定期操作由`redis.c/serverCron`函数负责执行，它的主要工作包括：

- 更新服务器统计信息，包括事件、内存占用等情况
- 清理过期键值对
- 关闭和清理失效的客户端连接
- AOF和RDB持久化操作
- 如果sever是主服务器，则对从服务器进行定期同步
- 如果是集群模式，对集群进行定期同步和连接测试

> cron在unix中表示计划任务，计时程序

默认频率是100毫秒一次，用户可以在redis.conf中修改hz选项来改变。

## 2.3 事件的调度与执行

因为服务器中同时存在文件事件和时间事件两种事件类型，所以服务器必须对这两种事件进行调度，**决定何时应该处理什么文件，以及花多少时间来处理它们等等。**事件的调度和执行由`ae.c/aeProcessEvents`函数负责。

对事件处理的原则是：

- 如果等待并处理完一次文件事件之后，仍未有任何时间事件到达，那么服务器将**再次等待并处理文件事件。**
- 对两种事件处理都是**同步、有序、原子**地执行的，服务器**不会中途中断事件处理，也不会对事件进行抢占**，因此需要尽可能地减少程序的阻塞时间，并在有需要时主动让出执行权。（比如写入字节太长，命令回复处理器就会break跳出，将余下的数据留到下次）
- 由于不能抢占，时间事件到达后需要等待文件事件处理完成，所以**一般会稍晚于到达时间。**

# 事务

---

title: Redis设计与实现11-事务
category:

- 数据库
  tags:
  - 计算机网络
  - 数据库
  - Redis
  - 读书笔记
    mathjax: true
    date: 2020-01-07 13:54:21

---

Redis通过MULTI、EXEC、WATCH等命令来实现事务（transaction）功能。事务将**一种将多个命令请求打包，然后一次性、按顺序地执行多个命令的机制**，并且在事务执行期间，**服务器不会中断事务**而改去执行其他客户端的命令请求。<!--more-->

```C
redis->MULTI
OK

redis->SET "name" "hellp"
QUEUED

redis->GET "name"
QUEUED

redis->EXEC
1)OK
2)"hellp"
```

# 1. 事务的实现

一个事务从开始到结束会经历三个阶段：

- 事务开始
- 命令入队
- 事务执行

**（1）事务开始**

通过MULTI命令可以将执行该命令的客户端**从非事务状态切换至事务状态**，这一切换是通过在客户端状态的flags属性中打开REDIS_MULTI标识来完成的。

当一个客户端已经处于非事务状态时，这个客户端发送的**命令会被服务器执行**。然而当切换到事务状态后，服务器会根据这个客户端发来的不同命令执行不同的操作：

- 如果客户端发送EXEC，DISCARD，WATCH，MULTI这四个命令，则立即执行。
- 如果发送的是其他命令，则放到事务队列里面，向客户端返回QUEUED回复。

**（2）命令入队**

事务的关键实现在于**命令入队**，每个Redis客户端都有自己的**事务状态**，这个事务状态保存在客户端状态的mstate属性里面：

```C
typedef struct redisClient
{
    //...
    multiState mstate;
    //...
}
```

而**事务状态结构体**又包含了一个**事务队列**，以及一个**已入队命令的计数器**。

```C
typedef struct multiState 
{ 
    // 事务队列，FIFO顺序 
    multiCmd *commands; 
    // 已入队命令计数 
    int count;
} multiState;
```

**事务队列**是一个结构体，实现了队列数据结构，执行FIFO先进先出的策略。真实结构是一个数组。

```c
typedef struct multiCmd 
{ 
    // 参数 
    robj **argv; 
    // 参数数量
    int argc; 
    // 命令指针 
    struct redisCommand *cmd;
} multiCmd;
```

事务结构具体的包含逻辑是：**客户端->事务状态multiState->事务队列multiCmd->具体命令cmd**

**（3）执行事务**

当一个处于事务状态的客户端向服务器发送EXEC命令时，这个EXEC命令将立即被服务器执行。**服务器会遍历这个客户端的事务队列，执行队列中保存的所有命令，最后将执行命令所得的结果全部返回给客户端。**

1. 创建空白回复队列
2. 抽取一条命令，读取参数、参数个数以及要执行的函数
3. 执行命令，取得返回值
4. 将返回值追加到1中的队列末尾，重复步骤2

完成所有命令后，将**清除REDIS_MULTI标志**，让客户端变为非事务状态，同时清**零入队命令计数器，并释放事务队列。**

# 2. WATCH命令的实现

WATCH可以别翻译为**监视器**。WATCH命令是一个**乐观锁（optimistic locking）**。

> 悲观锁：有罪推定原则，每次有人操作数据时都会假定他要修改，每次都会上互斥锁。
> 
> 乐观锁：无罪推定原则，每次别人拿数据都假定他不修改，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据。

它可以**在EXEC命令执行之前**，**监视任意数量的数据库键**，并在EXEC命令执行时，**检查被监视的键是否至少有一个已经被修改过了**，**如果是的话，服务器将拒绝执行事务**，并向客户端返回代表事务执行失败的空回复。

```C
redis-> WATCH "name"
OK

redis-> MULTI
OK

redis-> SET "name" "peter"
QUEUED

redis-> EXEC
(nil)
```

上面的例子中，WATCH监视器发现了事务修改了name的值，因此拒绝执行该事务，返回空回复。

## 2.1 监视原理

每个Redis数据库都保存着一个`watched_keys`字典，这个字典的**键是某个被WATCH命令监视的数据库键**，而**字典的值则是一个链表，链表中记录了所有监视相应数据库键的客户端：**

```C
typedef struct redisDb 
{ 
    // ... 
    // 正在被WATCH命令监视的键 
    dict *watched_keys;
    // ...
} redisDb;
```

下图说明：c1和c2客户端正在监视键"name"，c3客户端正在监视"age"....

## 2.2 监视触发

对数据库**执行修改命令**时，比如SET、LPUSH、SADD、ZREM、DEL、FLUSHDB等等，在执行之后都会调用`multi.c/touchWatchKey`函数对`watched_keys`字典进行检查。**查看当前命令修改的键是否在`watched_keys`字典中**，如果有，则客户端的`REDIS_DIRTY_CAS`标识打开**，表示该客户端的事务安全性已经被破坏**。

## 2.3 判断事务是否安全

当服务器接收到一个客户端发来的EXEC命令时，服务器会根据这个客户端是否打开了`REDIS_DIRTY_CAS`标识来决定是否执行事务。

- 如果标志被打开，则说明哨兵监视的键中被修改过了，所以当前提交的事务不再安全，拒绝执行客户端提交的事务。
- 反之，是安全的，继续执行。

# 3. 事务的ACID性质

所谓ACID性质是指：**有原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、耐久性（Durability）。**

**（1）原子性**

所谓原子性就是某个操作不可再分，比如汇编语言里面的`MOV DST,SRC`。事务的定义就是：将多个命令打包成一个实现，**要么全部执行，要么都不执行**。**在命令入队的时候，用WATCH进行检查**，不符合要求就直接返回。

Redis的事务和传统的关系型数据库事务的最大区别在于，**Redis不支持事务回滚机制（rollback）**。作者认为Redis追求简单高效，回滚机制太复杂。

> 回滚(rollback)是指当事务中某一条命令执行出错时，意味着前面的命令可能也不安全，这时候就会释放掉前面的操作，恢复到执行事务之前的状态。MySQL数据库支持回滚操作。

**（2）一致性**

“**一致”指的是数据符合数据库本身的定义和要求，没有包含非法或者无效的错误数据。**如果数据库在执行事务之前是一致的，那么在事务执行之后，无论事务是否执行成功，数据库也应该仍然是一致的。Redis保证一致性的方法如下：

- **入队错误**：事务入队时命令格式不正确，则Redis拒绝执行
- **执行错误**：执行时操作不正确，会被服务器识别，并做错误处理，所以这些出错命令不会对数据库做任何修改
- **服务器停机**：停机分三种情况，
  - 无持久化：重启后清空，数据总是一致的
  - RDB模式：根据RDB恢复数据，还原为一致状态
  - AOF模式：根据AOF恢复数据，还原为一致状态

**（3）隔离性**

隔离性也可被理解为**不存在竞争**。即使数据库中有多个事务并发地执行，各个事务之间也不会互相影响。

因为Redis使用**单线程**的方式来执行事务（以及事务队列中的命令），并且服务器保证，在执行事务期间**不会对事务进行中断**。这种**串行**的方式保证了事务也总是具有隔离性的。

**（4）耐久性**

事务的耐久性指的是，当一个事务执行完毕时，执行这个事务所得的结果已经**被保存到永久性存储介质**（比如硬盘）里面了，即使服务器在事务执行完毕之后停机，执行事务所得的结果也不会丢失。

Redis事务只是简答包裹了一组Redis命令，耐久性由持久化实现。前面提到持久化分不同的情况

- RDB模式下，只有特定条件被满足时才会执行BGSAVE，具有耐久性。
- AOF模式根据appendfsync选项来决定
  - always，每次执行命令后都会调用同步函数，具有耐久性。
  - everysec，每一秒才会同步到硬盘，不具有耐久性。
  - no，程序会交由操作系统来决定何时将命令数据同步到硬盘。不具有耐久性。

---

总的来说，Redis事务**一定**具有原子性，一致性和隔离性，但**只有在特定条件**下才具有耐久性。  

# 复制

在Redis中，用户可以通过执行SLAVEOF命令或者设置slaveof选项，让一个服务器去复制另一个服务器，我们称呼被复制的服务器为**主服务器（master）**，而对主服务器进行复制的服务器则被称为**从服务器（slave）**。搞清楚关系，如果服务器A输入指令SLAVEOF，则A变成B的从服务器。

进行复制中的主从服务器双方的数据库将保存相同的数据，概念上将这种现象称作“**数据库状态一致**”，或者简称“一致”。比如，在**主**服务器上执行命令，

```c
127.0.0.1:6379> SET msg "hello world"
OK
```

则同时可以在**从**服务器上获取msg键的值，

```C
127.0.0.1:12345> GET msg
"hello world"
```

## 1. 旧版复制

### 1.1 旧版复制的实现

Redis的复制功能分为**同步（sync）**和**命令传播（commandpropagate）**两个操作：

- 同步操作用于将从服务器的数据库状态更新至主服务器当前所处的数据库状态。
- 命令传播操作则用于在**主服务器的数据库状态被修改**，导致主从服务器的数据库状态出现不一致时，让主从服务器的数据库重新回到一致状态。

**（1）同步**

从服务器对主服务器的同步操作需要通过**向主服务器**发送**SYNC命令**来完成，以下是SYNC命令的执行步骤：

1. 从服务器向主服务器发送SYNC命令
2. 主服务器收到后，执行BGSAVE命令，生成RDB文件，并使用缓冲区记录现在开始执行的所有写命令。
3. 将RDB文件发送给从服务器，从服务器收到后更新
4. 主服务器将缓冲区的内容发送给从服务器，从服务器收到后更新。

> BGSAVE命令会**增加一个子进程**，负责创建RDB文件。

**（2）命令传播**

主服务器会将自己执行的**写命令**，也即是造成主从服务器不一致的那条写命令，发送给从服务器执行，当从服务器执行了相同的写命令之后，主从服务器将再次回到一致状态。

## 1.2 旧版复制的缺陷

在Redis中，从服务器对主服务器的复制可以分为以下两种情况：

- **初次复制**：从服务器以前没有复制过任何主服务器，或者要复制的主服务器和上一次复制的主服务器不同。
- **断线后重复制**：处于命令传播阶段的主从服务器因为网络原因而中断了复制，但从服务器通过自动重连接重新连上了主服务器，并继续复制主服务器。

初次复制效果挺好的，但断线后重新复制效率就很低。因为执行SYNC命令是非常消耗资源的行为。

## 2. 新版复制

### 2.1 新版复制功能的实现

Redis从2.8版本开始，使用**PSYNC命令**代替SYNC命令来执行复制时的同步操作。

PSYNC命令具有**完整重同步（full resynchronization）**和**部分重同步（partial resynchronization）**两种模式：

- 完整重同步用于初次复制，和SYNC命令完全一致
- 部分重同步，将断线后的命令发送给从服务器。

---

要实现部分重同步，需要完成三个部分：

- 主服务器的**复制偏移量**（replication offset）和从服务器的复制偏移量。
- 主服务器的**复制积压缓冲区**（replication backlog）。
- 服务器的**运行ID**（run ID）。

**（1）复制偏移量**

主服务器和从服务器会分别维护一个复制偏移量：

- 主服务器每次向从服务器传播N个字节的数据时，就将自己的复制偏移量的值加上N。
- 从服务器每次收到主服务器传播来的N个字节的数据时，就将自己的复制偏移量的值加上N。

**通过对比主从服务器的复制偏移量，程序可以很容易地知道主从服务器是否处于一致状态**：

- 如果主从服务器处于一致状态，那么主从服务器两者的偏移量总是相同的。
- 相反，如果主从服务器两者的偏移量并不相同，那么说明主从服务器并未处于一致状态。

**（2）复制积压缓冲区**

复制积压缓冲区是由主服务器维护的一个**固定长度**（fixed-size）先进先出（FIFO）队列，默认大小为1MB。当主服务器进行命令传播时，它不仅会将写命令发送给所有从服务器，还会**将写命令入队到复制积压缓冲区里面**。

与此同时，主服务器也会向积压缓冲区添加偏移量，

当服务器重新连接上主服务器时，从服务器会通过PSYNC命令将自己的复制偏移量offset发送给主服务器，**主服务器会根据这个复制偏移量来决定对从服务器执行何种同步操作**：

- offset偏移量之后的数据仍然存在于复制积压缓冲区中，主服务器执行部分重同步操作
- 反之，偏移量之后的数据已不存在于复制积压缓冲区，则执行完整重同步。

复制积压缓冲区作为一个**限制性容器**保证了复制的高效性：

- 如果断线时间短，错过的命令少，则直接调用偏移量为从服务器补上命令
- 反之，则直接完全重同步。

**（3）服务器运行ID**

每个Redis服务器，不论主服务器还是从服务，都会有自己的运行ID，运行ID在服务器启动时自动生成，由40个随机的十六进制字符组成。

当从服务器对主服务器进行初次复制时，主服务器会将自己的运行ID传送给从服务器，而从服务器则会将这个运行ID保存起来。

当从服务器断线并重新连上一个主服务器时，从服务器将向当前连接的主服务器发送之前保存的运行ID：

- 如果ID相同，则表示**之前同步的主服务器就是这个**，执行部分重同步。
- 如果ID不同，则表明从**服务器断线之前复制的主服务器并不是当前连接的这个主服务器**，执行完整重同步操作。

## 2.2 PSYNC命令的实现

PSYNC命令的调用方法有两种：

- 如果是初次复制，则从服务器发送`PSYNC?-1`命令，主动请求主服务器进行完整重同步。
- 如果已经复制过，则从服务器发送`PSYNC<runid><offset>`命令。即：上一次复制的主服务器ID+当前的复制偏移量。

根据情况，接收到PSYNC命令的主服务器会向从服务器返回以下三种回复的其中一种：

**（1）**如果主服务器返回`+FULLRESYNC <runid> <offset>`回复，那么表示主服务器将与从服务器执行完整重同步操作。从服务器将ID保存起来，在下一次PSYNC命令时使用，同时将offset的值当做自己的初始化偏移量。

**（2）**如果主服务器返回+CONTINUE回复，那么表示主服务器将与从服务器执行部分重同步操作，从服务器只要等着主服务器将自己缺少的那部分数据发送过来就可以了。

**（3）**如果主服务器返回-ERR回复，那么表示主服务器的版本低于Redis2.8，它识别不了PSYNC命令，从服务器将向主服务器发送SYNC命令，并与主服务器执行完整同步操作。

## 2.3 新版复制的完整流程

本节主要展示新版复制操作的全过程，假设主服务器IP地址为127.0.0.1端口号为6379，从服务器IP为127.0.0.1端口号12345.

**（1）设置主服务器地址和端口**

当**客户端**向**从服务器**发送以下命令时：

```C
127.0.0.1:12345> SLAVEOF 127.0.0.1 6379
OK
```

从服务器首先要做的就是将客户端给定的主服务器IP地址127.0.0.1以及端口6379保存到服务器状态的masterhost属性和masterport属性里面：

```C
struct redisServer{
    //...
    char *masterhost;
    int masterport;
    //...
};
```

**（2）建立套接字连接**

**从服务器**将根据命令所设置的IP地址和端口，创建**连向主服务器的**套接字连接

此时，**从服务器变为了主服务器的客户端。**从服务器**同时具备**服务器和客户端的两个身份。

**（3）发送PING命令**

连接成功后，从服务器立马发送一个PING命令，主要作用是：

- 检查套接字读写是否正常
- 检查主服务器能否正常处理命令

回复有三种可能：

- 主服务器**返回了命令回复**，但**从服务器不能再规定的时间内读出**，表明主从之间**网络连接不佳**。从服务器**断开并重新创建**连向主服务器的套接字。
- 主服务器**返回一个错误**，表示主服务器暂时无法处理请求（比如正在处理一个超时运行脚本），从服务器**断开并重新创建**连向主服务器的套接字。
- 从服务器收到PONG回复，表示主从之间连接正常。

**（4）身份验证**

收到pong的回复后，下一步是确定是否进行身份验证：如果从服务器设置了masterauth选项，那么进行身份验证；反之则不进行。

**（5）发送端口信息**

从服务器向主服务器发送从服务器的监听端口号。主服务器在接收到这个命令之后，会将端口号记录在从服务器所对应的客户端状态的`slave_listening_port`属性中：

```C
typedef struct redisClient 
{ 
    // ... 
    // 从服务器的监听端口号 
    int slave_listening_port; 
    // ...
} redisClient;
```

**（6）同步**

在这一步，从服务器将向主服务器发送PSYNC命令，执行同步操作，并将自己的数据库更新至主服务器数据库当前所处的状态。

在同步操作执行之前，只有从服务器是主服务器的客户端，但是**在执行同步操作之后，主服务器也会成为从服务器的客户端**。

**（7）命令传播**

主服务器只要一直将自己执行的写命令发送给从服务器，而从服务器只要一直接收并执行主服务器发来的写命令，就可以保证主从服务器一直保持一致了。

## 4. 心跳检测

在**命令传播阶段**，从服务器默认会以每秒一次的频率，向主服务器发送命令：

```C
REPLCONF ACK <replication_offset>
```

其中`replication_offset`是从服务器当前的复制偏移量。发送`REPLCONF ACK`命令对于主从服务器有三个作用： 

- 检测主从服务器的网络连接状态。 
- 辅助实现min-slaves选项。
- 检测命令丢失。

**（1）检测连接状态**

如果主服务器超过一秒钟没有收到从服务器发来的REPLCONF ACK命令，那么主服务器就知道主从服务器之间的连接出现问题了。

**（2）辅助实现min-slaves选项**

Redis的`min-slaves-to-write`和`min-slaves-max-lag`两个选项可以防止主服务器在不安全的情况下执行写命令。

举个例子，如果我们向主服务器提供以下设置：

```C
min-slaves-to-write 3
min-slaves-max-lag 10
```

那么在从服务器的数量少于3个，或者三个从服务器的延迟（lag）值都大于或等于10秒时，主服务器将拒绝执行写命令。

**（3）检测命令丢失**

假如主服务器的向从服务器发送的传播命令因为网络问题丢失，会导致二者偏移量不一致。这是心跳检测命令会侦察到这种情况，于是主服务器会补发。

# 7. 消息

集群中的各个节点通过发送和接收消息（message）来进行通信，节点发送的消息主要有以下五种：

- **MEET消息：**当发送者接到客户端发送的CLUSTER MEET命令时，发送者会**向接收者**发送MEET消息，请求接收者加入到发送者当前所处的集群里面。
- **PING消息：**集群里的每个节点默认每隔一秒钟就会从**已知节点列表中随机选出五个节点**，然后对这五个节点中最长时间没有发送过PING消息的节点发送PING消息，以此来检测被选中的节点是否在线。
- **PONG消息：**当接收者收到发送者发来的MEET消息或者PING消息时，为了向发送者确认这条MEET消息或者PING消息已到达，接收者**会向发送者**返回一条PONG消息。另外，一个节点也可以通过向**集群广播**自己的PONG消息来让集群中的其他节点立即刷新关于这个节点的认识。
- **FAIL消息：**当一个主节点A判断另一个主节点B已经进入FAIL状态时，节点A会**向集群广播**一条关于节点B的FAIL消息，所有收到这条消息的节点都会立即将节点B标记为已下线。
- **PUBLISH消息**：当节点接收到一个PUBLISH命令时，节点会执行这个命令，并**向集群广播**一条PUBLISH消息，所有接收到这条PUBLISH消息的节点都会执行相同的PUBLISH命令。

## 7.1 消息的结构

一条消息由消息头（header）和消息正文（data）组成

每个消息头都由一个`cluster.h/clusterMsg`结构表示：

```C
// 用来表示集群消息的结构（消息头，header）
typedef struct {
    //...
    // 消息的长度（包括这个消息头的长度和消息正文的长度）
    uint32_t totlen;   

    // 消息的类型
    uint16_t type;    

    // 消息正文包含的节点信息数量
    // 只在发送 MEET 、 PING 和 PONG 这三种 Gossip 协议消息时使用
    uint16_t count;   

    // 消息发送者的配置纪元
    uint64_t currentEpoch;  

    // 如果消息发送者是一个主节点，那么这里记录的是消息发送者的配置纪元
    // 如果消息发送者是一个从节点，那么这里记录的是消息发送者正在复制的主节点的配置纪元
    uint64_t configEpoch; 

    // 节点的复制偏移量
    uint64_t offset;   

    // 消息发送者的名字（ID）
    char sender[REDIS_CLUSTER_NAMELEN]; /

    // 消息发送者目前的槽指派信息
    unsigned char myslots[REDIS_CLUSTER_SLOTS/8];

    // 如果消息发送者是一个从节点，那么这里记录的是消息发送者正在复制的主节点的名字
    // 如果消息发送者是一个主节点，那么这里记录的是 REDIS_NODE_NULL_NAME
    // （一个 40 字节长，值全为 0 的字节数组）
    char slaveof[REDIS_CLUSTER_NAMELEN];

    // 消息发送者的端口号
    uint16_t port;      /* Sender TCP base port */

    // 消息发送者的标识值
    uint16_t flags;     /* Sender node flags */

    // 消息发送者所处集群的状态
    unsigned char state; /* Cluster state from the POV of the sender */

    // 消息标志
    unsigned char mflags[3]; 

    // 消息的正文（或者说，内容）
    union clusterMsgData data;

} clusterMsg;
```

datas属性指向联合`cluster.h/clusterMsgData`，也就是消息的正文

```C
nion clusterMsgData {

    /* PING, MEET and PONG */
    struct {
        // 每条消息都包含两个 clusterMsgDataGossip 结构
        clusterMsgDataGossip gossip[1]; //gossip八卦，小道消息，闲话的意思
    } ping;

    /* FAIL */
    struct {
        clusterMsgDataFail about;
    } fail;

    /* PUBLISH */
    struct {
        clusterMsgDataPublish msg;
    } publish;
    //...

};
```

## 7.2 消息的实现

**（1）Gossip消息的实现**

gossip是八卦，小道消息，闲话的意思，专指MEET、PING、PONG这几个消息。这三种消息的正文都由**两个**`cluster.h/clusterMsgDataGossip`结构组成。

因为MEET、PING、PONG三种消息都使用相同的消息正文，所以节点通过消息头的type属性来判断一条消息是MEET消息、PING消息还是PONG消息。

每次发送MEET、PING、PONG消息时，发送者都从自己的已知节点列表中**随机选出两个节点**（可以是主节点或者从节点），并将这两个被选中节点的信息分别保存到两个clusterMsg-DataGossip结构里面。

接受者接收到MEET、PING、PONG消息时，根据保存的两个节点是否认识来选择进行哪种操作：

- 不认识，说明接收者**第一次接触被选中节点**，则接收者与被选中节点握手
- 认识，根据结构信息进行更新。

比如A节点发送的PING给B，携带了CD两个节点，然后B回复PONG携带了EF两个节点，这样就完成了ABCDEF六个节点的信息交换。**每个节点按照周期向不同节点传播PING-PONG信息，就能完成整个集群的状态更新。**

**（2）FAIL消息的实现**

Gossip协议传播速度很慢，而主节点下线的消息需要立即通知给所有人。

FAIL消息的正文由`cluster.h/clusterMsgDataFail`结构表示，这个结构只包含一个nodename属性，该属性记录了已下线节点的名字：

```C
typedef struct 
{ 
    char nodename[REDIS_CLUSTER_NAMELEN];
} clusterMsgDataFail;
```

因为集群里的所有节点都有一个独一无二的名字，所以FAIL消息里面只需要保存下线节点的名字，接收到消息的节点就可以根据这个名字来判断是哪个节点下线了。

**（3）PUBLISH消息的实现**

当客户端向集群中的某个节点发送命令：

```C
PUBLISH <channel> <message>
```

接收到PUBLISH命令的节点**不仅会向channel频道发送消息message，它还会向集群广播一条PUBLISH消息，所有接收到这条PUBLISH消息的节点都会向channel频道发送message消息**。

也就是说，向集群发送`PUBLISH <channel> <message>`，会导致集群所有节点都向channel发送message消息。

# Redis 异步队列

一般使用list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。

缺点：

在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。

# 17.Redis相比memcached有哪些优势？

(1) memcached所有的值均是简单的字符串，redis作为其替代者， 支持更为丰富的数据类型
(2) redis的速度比memcached快很多
(3) redis可以持久化其数据<u></u>

# 缓存常见问题

### 缓存穿透

缓存穿透是指查询一个**不存在的数据**，由于缓存是不命中时被动写的，如果从DB查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到DB去查询，失去了缓存的意义。在流量大时，可能DB就挂掉了。

怎么解决？

1. **缓存空值**，不会查数据库。
2. 采用**布隆过滤器**，将所有可能存在的数据哈希到一个足够大的`bitmap`中，查询不存在的数据会被这个`bitmap`拦截掉，从而避免了对`DB`的查询压力。

布隆过滤器的原理：当一个元素被加入集合时，通过K个哈希函数将这个元素映射成一个位数组中的K个点，把它们置为1。查询时，将元素通过哈希函数映射之后会得到k个点，如果这些点有任何一个0，则被检元素一定不在，直接返回；如果都是1，则查询元素很可能存在，就会去查询Redis和数据库。

布隆过滤器一般用于在大数据量的集合中判定某元素是否存在。

### 缓存雪崩

Redis缓存雪崩是指在某个时间点，缓存中大量的数据同时过期或者缓存服务器发生故障，导致大量的请求直接访问数据库，从而造成数据库负载过高，甚至崩溃的情况。缓存雪崩是指在我们设置缓存时采用了相同的过期时间，**导致缓存在某一时刻同时失效**，请求全部转发到DB，DB瞬时压力过重挂掉。

缓存雪崩通常发生在以下情况下：

1. 缓存数据过期时间集中：如果大量的缓存数据在同一时间点过期，那么在这些数据重新加载到缓存之前，所有的请求都会直接访问数据库，导致数据库压力剧增。

2. 缓存服务器故障：如果缓存服务器发生故障，无法提供缓存服务，那么所有的请求都会直接访问数据库，造成数据库压力过大。

为了避免Redis缓存雪崩，可以采取以下措施：

1. 设置合理的缓存过期时间：将缓存数据的过期时间分散开，避免大量数据在同一时间点过期。可以通过为每个缓存数据设置一个随机的过期时间或者在过期时间上加上一个随机值来实现。

2. 实现缓存数据的自动预加载：在缓存数据过期之前，提前异步加载数据到缓存中，避免在数据过期时才去加载，减少数据库的压力。

3. 使用多级缓存架构：引入多级缓存架构，例如将热点数据缓存在内存中的Redis中，将冷数据缓存在持久化存储中，这样可以减轻Redis的负载，提高系统的稳定性。

4. 设置缓存穿透保护机制：在请求数据时，如果发现缓存中不存在该数据，可以设置一个空值或者默认值存入缓存，避免大量的请求直接访问数据库。

5. 做好缓存和数据库的容灾备份：定期备份缓存数据和数据库数据，以防止数据丢失或者发生故障时能够快速恢复。

通过以上措施的综合应用，可以有效地预防和应对Redis缓存雪崩问题。

1. 在原有的失效时间基础上**增加一个随机值**，使得过期时间分散一些。这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
2. **加锁排队可以起到缓冲的作用**，防止大量的请求同时操作数据库，但它的缺点是**增加了系统的响应时间**，**降低了系统的吞吐量**，牺牲了一部分用户体验。当缓存未查询到时，对要请求的 key 进行加锁，只允许一个线程去数据库中查，其他线程等候排队。
3. 设置二级缓存。二级缓存指的是除了 Redis 本身的缓存，**再设置一层缓存**，当 Redis 失效之后，先去查询二级缓存。例如可以设置一个本地缓存，在 Redis 缓存失效的时候先去查询本地缓存而非查询数据库。

### 缓存击穿

缓存击穿：大量的请求同时查询一个 key 时，此时这个 key 正好失效了，就会导致大量的请求都落到数据库。**缓存击穿是查询缓存中失效的 key，而缓存穿透是查询不存在的 key。**

解决方法：

1、**加互斥锁**。在并发的多个请求中，只有第一个请求线程能拿到锁并执行数据库查询操作，其他的线程拿不到锁就阻塞等着，等到第一个线程将数据写入缓存后，直接走缓存。可以使用Redis分布式锁实现，代码如下：

```java
public String get(String key) {
    String value = redis.get(key);
    if (value == null) { //缓存值过期
        String unique_key = systemId + ":" + key;
        //设置30s的超时
        if (redis.set(unique_key, 1, 'NX', 'PX', 30000) == 1) {  //设置成功
            value = db.get(key);
            redis.set(key, value, expire_secs);
            redis.del(unique_key);
        } else {  //其他线程已经到数据库取值并回写到缓存了，可以重试获取缓存值
            sleep(50);
            get(key);  //重试
        }
    } else {
        return value;
    }
}
```

2、**热点数据不过期**。直接将缓存设置为不过期，然后由定时任务去异步加载数据，更新缓存。这种方式适用于比较极端的场景，例如流量特别特别大的场景，使用时需要考虑业务能接受数据不一致的时间，还有就是异常情况的处理，保证缓存可以定时刷新。

### 缓存穿透

### 缓存预热

缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！

解决方案：

1. 直接写个缓存刷新页面，上线时手工操作一下；
2. 数据量不大，可以在项目启动的时候自动进行加载；
3. 定时刷新缓存；

### 缓存降级

当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。

缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。

在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

1. 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
2. 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；
3. 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
4. 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。

# 18.Redis如何存储复杂的数据？

### 复杂数据存储

一般的数据存储，就是一个key对应一个简单字符串，可以要想向mysql一样，保存或者获取某一列的所有值呢，
例如,用户表结构为

| id  | name | age |
| --- | ---- | --- |
| 1   | 张三   | 19  |
| 2   | 李四   | 23  |

这种情况一般可以利用json存储或者是hash

#### json存储

```
redis 127.0.0.1:6379> set charmtest:user:1 '{"name":"张三","age":"19"}'OK

redis 127.0.0.1:6379> set charmtest:user:2 '{"name":"李四","age":"23"}'OK

redis 127.0.0.1:6379> get charmtest:user:1"{"name":"张三","age":"19"}"

redis 127.0.0.1:6379> get charmtest:user:2"{"name":"李四","age":"23"}"
```

#### hash存储

```
redis 127.0.0.1:6379:0>hset 'charmhash:user:1' name '张三'

"1"

redis 127.0.0.1:6379:0>hset 'charmhash:user:1' age 19

"1"
```

#### 获取数据

```
redis 127.0.0.1:6379:0>hget 'charmhash:user:1' name

"张三"

redis 127.0.0.1:6379:0>hget 'charmhash:user:1' age

"19"
```

# Redis 过期策略

我们都知道，Redis是key-value数据库，我们可以设置Redis中缓存的key的过期时间。Redis的过期策略就是指当Redis中缓存的key过期了，Redis如何处理。

过期策略通常有以下三种

· 定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。

· 惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

· 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。
(expires字典会保存所有设置了过期时间的key的过期时间数据，其中，key是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。)

Redis中同时使用了惰性过期和定期过期两种过期策略。

### Redis 为什么这么快

1.完全基于内存，绝大部分请求是纯粹的内存操作，非常快速

2.数据结构简单，对数据操作也简单

3.采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程 导致的切换而消耗 CPU，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有 因为可能出现死锁而导致的性能消耗
4.使用多路 I/O 复用模型，非阻塞 IO，Redis 采用 IO 多路复用技术。Redis 使用单线程来轮询描述符，将数据库的操作都转换成了事件，不在网络I/O上浪费过多的时间。

## 既然Redis那么快，为什么不用它做主数据库，只用它做缓存

虽然Redis非常快，但它也有一些局限性，不能完全替代主数据库。有以下原因：

**事务处理：**Redis只支持简单的事务处理，对于复杂的事务无能为力，比如跨多个键的事务处理。

**数据持久化：**Redis是内存数据库，数据存储在内存中，如果服务器崩溃或断电，数据可能丢失。虽然Redis提供了数据持久化机制，但有一些限制。

**数据处理：**Redis只支持一些简单的数据结构，比如字符串、列表、哈希表等。如果需要处理复杂的数据结构，比如关系型数据库中的表，那么Redis可能不是一个好的选择。

**数据安全：**Redis没有提供像主数据库那样的安全机制，比如用户认证、访问控制等等。

因此，虽然Redis非常快，但它还有一些限制，不能完全替代主数据库。所以，使用Redis作为缓存是一种很好的方式，可以提高应用程序的性能，并减少数据库的负载。

SortedSet和List异同点？

**相同点**：

1. 都是有序的；
2. 都可以获得某个范围内的元素。

**不同点：**

1. 列表基于链表实现，获取两端元素速度快，访问中间元素速度慢；
2. 有序集合基于散列表和跳跃表实现，访问中间元素时间复杂度是OlogN；
3. 列表不能简单的调整某个元素的位置，有序列表可以（更改元素的分数）；
4. 有序集合更耗内存。

## Redis的内存用完了会怎样？

如果达到设置的上限，Redis的写命令会返回错误信息（但是读命令还可以正常返回）。

也可以配置内存淘汰机制，当Redis达到内存上限时会冲刷掉旧的内容。

## Redis如何做内存优化？

可以好好利用Hash,list,sorted set,set等集合类型数据，因为通常情况下很多小的Key-Value可以用更紧凑的方式存放到一起。尽可能使用散列表（hashes），散列表（是说散列表里面存储的数少）使用的内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比如你的web系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的key，而是应该把这个用户的所有信息存储到一张散列表里面。

## keys命令存在的问题？

redis的单线程的。keys指令会导致线程阻塞一段时间，直到执行完毕，服务才能恢复。scan采用渐进式遍历的方式来解决keys命令可能带来的阻塞问题，每次scan命令的时间复杂度是`O(1)`，但是要真正实现keys的功能，需要执行多次scan。

scan的缺点：在scan的过程中如果有键的变化（增加、删除、修改），遍历过程可能会有以下问题：新增的键可能没有遍历到，遍历出了重复的键等情况，也就是说scan并不能保证完整的遍历出来所有的键。

## 过期键的删除策略？

1、**被动删除**。在访问key时，如果发现key已经过期，那么会将key删除。

2、**主动删除**。定时清理key，每次清理会依次遍历所有DB，从db随机取出20个key，如果过期就删除，如果其中有5个key过期，那么就继续对这个db进行清理，否则开始清理下一个db。

3、**内存不够时清理**。Redis有最大内存的限制，通过maxmemory参数可以设置最大内存，当使用的内存超过了设置的最大内存，就要进行内存释放， 在进行内存释放的时候，会按照配置的淘汰策略清理内存。

## 内存淘汰策略有哪些？

当Redis的内存超过最大允许的内存之后，Redis 会触发内存淘汰策略，删除一些不常用的数据，以保证Redis服务器正常运行。

**Redisv4.0前提供 6 种数据淘汰策略**：

- **volatile-lru**：LRU（`Least Recently Used`），最近使用。利用LRU算法移除设置了过期时间的key
- **allkeys-lru**：当内存不足以容纳新写入数据时，从数据集中移除最近最少使用的key
- **volatile-ttl**：从已设置过期时间的数据集中挑选将要过期的数据淘汰
- **volatile-random**：从已设置过期时间的数据集中任意选择数据淘汰
- **allkeys-random**：从数据集中任意选择数据淘汰
- **no-eviction**：禁止删除数据，当内存不足以容纳新写入数据时，新写入操作会报错

**Redisv4.0后增加以下两种**：

- **volatile-lfu**：LFU，Least Frequently Used，最少使用，从已设置过期时间的数据集中挑选最不经常使用的数据淘汰。
- **allkeys-lfu**：当内存不足以容纳新写入数据时，从数据集中移除最不经常使用的key。

**内存淘汰策略可以通过配置文件来修改**，相应的配置项是`maxmemory-policy`，默认配置是`noeviction`。

## MySQL 与 Redis 如何保证数据一致性

**缓存不一致是如何产生的**

如果数据一直没有变更，那么就不会出现缓存不一致的问题。

通常缓存不一致是发生在数据有变更的时候。 因为每次数据变更你需要同时操作数据库和缓存，而他们又属于不同的系统，无法做到同时操作成功或失败，总会有一个时间差。在并发读写的时候可能就会出现缓存不一致的问题（理论上通过分布式事务可以保证这一点，不过实际上基本上很少有人这么做）。

虽然没办法在数据有变更时，保证缓存和数据库强一致，但对缓存的更新还是有一定设计方法的，遵循这些设计方法，能够让这个不一致的影响时间和影响范围最小化。

缓存更新的设计方法大概有以下四种：

- 先删除缓存，再更新数据库（这种方法在并发下最容易出现长时间的脏数据，不可取）
- 先更新数据库，删除缓存（Cache Aside Pattern）
- 只更新缓存，由缓存自己同步更新数据库（Read/Write Through Pattern）
- 只更新缓存，由缓存自己异步更新数据库（Write Behind Cache Pattern）

**先删除缓存，再更新数据库**

**先更新数据库，再让缓存失效**

**只更新缓存，由缓存自己同步更新数据库（Read/Write Through Pattern）**

**只更新缓存，由缓存自己异步更新数据库（Write Behind Cache Pattern）**

```java
BLPOP queue 0  //0表示不限制等待时间
```

> BLPOP和LPOP命令相似，唯一的区别就是当列表没有元素时BLPOP命令会一直阻塞连接，直到有新元素加入。

redis可以通过pub/sub**主题订阅模式**实现一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢失。

```java
PUBLISH channel1 hi
SUBSCRIBE channel1
UNSUBSCRIBE channel1 //退订通过SUBSCRIBE命令订阅的频道。
```

> `PSUBSCRIBE channel?*` 按照规则订阅。 `PUNSUBSCRIBE channel?*` 退订通过PSUBSCRIBE命令按照某种规则订阅的频道。其中订阅规则要进行严格的字符串匹配，`PUNSUBSCRIBE *`无法退订`channel?*`规则。

## Redis 怎么实现延时队列

使用sortedset，拿时间戳作为score，消息内容作为key，调用zadd来生产消息，消费者用`zrangebyscore`指令获取N秒之前的数据轮询进行处理。

## pipeline的作用？

redis客户端执行一条命令分4个过程： 发送命令、命令排队、命令执行、返回结果。使用`pipeline`可以批量请求，批量返回结果，执行速度比逐条执行要快。

使用`pipeline`组装的命令个数不能太多，不然数据量过大，增加客户端的等待时间，还可能造成网络阻塞，可以将大量命令的拆分多个小的`pipeline`命令完成。

原生批命令（mset和mget）与`pipeline`对比：

1. 原生批命令是原子性，`pipeline`是**非原子性**。pipeline命令中途异常退出，之前执行成功的命令**不会回滚**。

2. 原生批命令只有一个命令，但`pipeline`**支持多命令**。

## LUA脚本

Redis 通过 LUA 脚本创建具有原子性的命令： 当lua脚本命令正在运行的时候，不会有其他脚本或 Redis 命令被执行，实现组合命令的原子操作。

在Redis中执行Lua脚本有两种方法：`eval`和`evalsha`。`eval`命令使用内置的 Lua 解释器，对 Lua 脚本进行求值。

```java
//第一个参数是lua脚本，第二个参数是键名参数个数，剩下的是键名参数和附加参数
> eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

**lua脚本作用**

1、Lua脚本在Redis中是原子执行的，执行过程中间不会插入其他命令。

2、Lua脚本可以将多条命令一次性打包，有效地减少网络开销。

**应用场景**

举例：限制接口访问频率。

在Redis维护一个接口访问次数的键值对，`key`是接口名称，`value`是访问次数。每次访问接口时，会执行以下操作：

- 通过`aop`拦截接口的请求，对接口请求进行计数，每次进来一个请求，相应的接口访问次数`count`加1，存入redis。
- 如果是第一次请求，则会设置`count=1`，并设置过期时间。因为这里`set()`和`expire()`组合操作不是原子操作，所以引入`lua`脚本，实现原子操作，避免并发访问问题。
- 如果给定时间范围内超过最大访问次数，则会抛出异常。

```java
private String buildLuaScript() {
    return "local c" +
        "\nc = redis.call('get',KEYS[1])" +
        "\nif c and tonumber(c) > tonumber(ARGV[1]) then" +
        "\nreturn c;" +
        "\nend" +
        "\nc = redis.call('incr',KEYS[1])" +
        "\nif tonumber(c) == 1 then" +
        "\nredis.call('expire',KEYS[1],ARGV[2])" +
        "\nend" +
        "\nreturn c;";
}

String luaScript = buildLuaScript();
RedisScript<Number> redisScript = new DefaultRedisScript<>(luaScript, Number.class);
Number count = redisTemplate.execute(redisScript, keys, limit.count(), limit.period());
```

PS：这种接口限流的实现方式比较简单，问题也比较多，一般不会使用，接口限流用的比较多的是令牌桶算法和漏桶算法。

## 什么是RedLock？

Redis 官方站提出了一种权威的基于 Redis 实现分布式锁的方式名叫 *Redlock*，此种方式比原先的单节点的方法更安全。它可以保证以下特性：

1. 安全特性：互斥访问，即永远只有一个 client 能拿到锁
2. 避免死锁：最终 client 都可能拿到锁，不会出现死锁的情况，即使原本锁住某资源的 client 挂掉了
3. 容错性：只要大部分 Redis 节点存活就可以正常提供服务

## Redis大key怎么处理？

通常我们会将含有较大数据或含有大量成员、列表数的Key称之为大Key。

以下是对各个数据类型大key的描述：

- value是STRING类型，它的值超过5MB
- value是ZSET、Hash、List、Set等集合类型时，它的成员数量超过1w个

上述的定义并不绝对，主要是根据value的成员数量和大小来确定，根据业务场景确定标准。

怎么处理：

1. 当vaule是string时，可以使用序列化、压缩算法将key的大小控制在合理范围内，但是序列化和反序列化都会带来更多时间上的消耗。或者将key进行拆分，一个大key分为不同的部分，记录每个部分的key，使用multiget等操作实现事务读取。
2. 当value是list/set等集合类型时，根据预估的数据规模来进行分片，不同的元素计算后分到不同的片。

## Redis常见性能问题和解决方案？

1. Master最好不要做任何持久化工作，包括内存快照和AOF日志文件，特别是不要启用内存快照做持久化。
2. 如果数据比较关键，某个Slave开启AOF备份数据，策略为每秒同步一次。
3. 为了主从复制的速度和连接的稳定性，Slave和Master最好在同一个局域网内。
4. 尽量避免在压力较大的主库上增加从库
5. Master调用BGREWRITEAOF重写AOF文件，AOF在重写的时候会占大量的CPU和内存资源，导致服务load过高，出现短暂服务暂停现象。
6. 为了Master的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关系为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现Slave对Master的替换，也即，如果Master挂了，可以立马启用Slave1做Master，其他不变。

## 说说为什么Redis过期了为什么内存没释放？

第一种情况，可能是覆盖之前的key，导致key过期时间发生了改变。

当一个key在Redis中已经存在了，但是由于一些误操作使得key过期时间发生了改变，从而导致这个key在应该过期的时间内并没有过期，从而造成内存的占用。

第二种情况是，Redis过期key的处理策略导致内存没释放。

一般Redis对过期key的处理策略有两种：惰性删除和定时删除。

先说惰性删除的情况

当一个key已经确定设置了xx秒过期同时中间也没有修改它，xx秒之后它确实已经过期了，但是惰性删除的策略它并不会马上删除这个key，而是当再次读写这个key时它才会去检查是否过期，如果过期了就会删除这个key。也就是说，惰性删除策略下，就算key过期了，也不会立刻释放内容，要等到下一次读写这个key才会删除key。

而定时删除会在一定时间内主动淘汰一部分已经过期的数据，默认的时间是每100ms过期一次。因为定时删除策略每次只会淘汰一部分过期key，而不是所有的过期key，如果redis中数据比较多的话要是一次性全量删除对服务器的压力比较大，每一次只挑一批进行删除，所以很可能出现部分已经过期的key并没有及时的被清理掉，从而导致内存没有即时被释放。

## Redis突然变慢，有哪些原因？

1. **存在bigkey**。如果Redis实例中存储了 bigkey，那么在淘汰删除 bigkey 释放内存时，也会耗时比较久。应该避免存储 bigkey，降低释放内存的耗时。

2. 如果Redis 实例**设置了内存上限 maxmemory**，有可能导致 Redis 变慢。当 Redis 内存达到 maxmemory 后，每次写入新的数据之前，Redis 必须先从实例中踢出一部分数据，让整个实例的内存维持在 maxmemory 之下，然后才能把新数据写进来。

3. **开启了内存大页**。当 Redis 在执行后台 RDB 和 AOF rewrite 时，采用 fork 子进程的方式来处理。但主进程 fork 子进程后，此时的主进程依旧是可以接收写请求的，而进来的写请求，会采用 Copy On Write（写时复制）的方式操作内存数据。
   
   什么是写时复制？
   
   这样做的好处是，父进程有任何写操作，并不会影响子进程的数据持久化。
   
   不过，主进程在拷贝内存数据时，会涉及到新内存的申请，如果此时操作系统开启了内存大页，那么在此期间，客户端即便只修改 10B 的数据，Redis 在申请内存时也会以 2MB 为单位向操作系统申请，申请内存的耗时变长，进而导致每个写请求的延迟增加，影响到 Redis 性能。
   
   解决方案就是关闭内存大页机制。

4. **使用了Swap**。操作系统为了缓解内存不足对应用程序的影响，允许把一部分内存中的数据换到磁盘上，以达到应用程序对内存使用的缓冲，这些内存数据被换到磁盘上的区域，就是 Swap。当内存中的数据被换到磁盘上后，Redis 再访问这些数据时，就需要从磁盘上读取，访问磁盘的速度要比访问内存慢几百倍。尤其是针对 Redis 这种对性能要求极高、性能极其敏感的数据库来说，这个操作延时是无法接受的。解决方案就是增加机器的内存，让 Redis 有足够的内存可以使用。或者整理内存空间，释放出足够的内存供 Redis 使用

5. **网络带宽过载**。网络带宽过载的情况下，服务器在 TCP 层和网络层就会出现数据包发送延迟、丢包等情况。Redis 的高性能，除了操作内存之外，就在于网络 IO 了，如果网络 IO 存在瓶颈，那么也会严重影响 Redis 的性能。解决方案：1、及时确认占满网络带宽 Redis 实例，如果属于正常的业务访问，那就需要及时扩容或迁移实例了，避免因为这个实例流量过大，影响这个机器的其他实例。2、运维层面，需要对 Redis 机器的各项指标增加监控，包括网络流量，在网络流量达到一定阈值时提前报警，及时确认和扩容。

6. **频繁短连接**。频繁的短连接会导致 Redis 大量时间耗费在连接的建立和释放上，TCP 的三次握手和四次挥手同样也会增加访问延迟。应用应该使用长连接操作 Redis，避免频繁的短连接。

## 为什么 Redis 集群的最大槽数是 16384 个？

Redis Cluster 采用数据分片机制，定义了 16384个 Slot槽位，集群中的每个Redis 实例负责维护一部分槽以及槽所映射的键值数据。

Redis每个节点之间会定期发送ping/pong消息（心跳包包含了其他节点的数据），用于交换数据信息。

Redis集群的节点会按照以下规则发ping消息：

- (1)每秒会随机选取5个节点，找出最久没有通信的节点发送ping消息
- (2)每100毫秒都会扫描本地节点列表，如果发现节点最近一次接受pong消息的时间大于cluster-node-timeout/2 则立刻发送ping消息

心跳包的消息头里面有个myslots的char数组，是一个bitmap，每一个位代表一个槽，如果该位为1，表示这个槽是属于这个节点的。

接下来，解答为什么 Redis 集群的最大槽数是 16384 个，而不是65536 个。

1、如果采用 16384 个插槽，那么心跳包的消息头占用空间 2KB （16384/8）；如果采用 65536 个插槽，那么心跳包的消息头占用空间 8KB (65536/8)。可见采用 65536 个插槽，**发送心跳信息的消息头达8k，比较浪费带宽**。

2、一般情况下一个Redis集群**不会有超过1000个master节点**，太多可能导致网络拥堵。

3、哈希槽是通过一张bitmap的形式来保存的，在传输过程中，会对bitmap进行压缩。bitmap的填充率越低，**压缩率**越高。其中bitmap 填充率 = slots / N (N表示节点数)。所以，插槽数越低， 填充率会降低，压缩率会提高。

## Redis存在线程安全的问题吗

首先从Redis 服务端层面来看。

Redis Server本身是一个线程安全的K-V数据库，也就是说在Redis Server上执行的指令，不需要任何同步机制，不会存在线程安全问题。

虽然Redis 6.0里面，增加了多线程的模型，但是增加的多线程只是用来处理网络IO事件，对于指令的执行过程，仍然是由主线程来处理，所以不会存在多个线程通知执行操作指令的情况。

然后从Redis客户端层面来看。

虽然Redis Server中的指令执行是原子的，但是如果有多个Redis客户端同时执行多个指令的时候，就无法保证原子性。

假设两个redis client同时获取Redis Server上的key1， 同时进行修改和写入，因为多线程环境下的原子性无法被保障，以及多进程情况下的共享资源访问的竞争问题，使得数据的安全性无法得到保障。

对于客户端层面的线程安全性问题，解决方法有很多，比如尽可能的使用Redis里面的原子指令，或者对多个客户端的资源访问加锁，或者通过Lua脚本来实现多个指令的操作等等。

### 1.在分布式数据库中CAP原理CAP+BASE

### CAP

```
C:Consistency（强一致性）

A:Availability（可用性）

P:Partition tolerance（分区容错性）

注意：分布式架构的时候必须做出取舍。
一致性和可用性之间取一个平衡。多余大多数web应用，其实并不需要强一致性。因此牺牲C换取P，这是目前分布式数据库产品的方向
```

### 经典CAP图

```
CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，
最多只能同时较好的满足两个。
因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三大类：
CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
CP - 满足一致性，分区容忍必的系统，通常性能不是特别高。
AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。
```

### Base

BASE就是为了解决关系数据库强一致性引起的问题而引起的可用性降低而提出的解决方案。

BASE其实是下面三个术语的缩写：
 基本可用（Basically Available）
 软状态（Soft state）
 最终一致（Eventually consistent）

它的思想是通过让系统放松对某一时刻数据一致性的要求来换取系统整体伸缩性和性能上改观。为什么这么说呢，缘由就在于大型系统往往由于地域分布和极高性能的要求，不可能采用分布式事务来完成这些指标，要想获得这些指标，我们必须采用另外一种方式来完成，这里BASE就是解决这个问题的办法

### 分布式+集群简介

分布式系统

分布式系统（distributed system）
 由多台计算机和通信的软件组件通过计算机网络连接（本地网络或广域网）组成。分布式系统是建立在网络之上的软件系统。正是因为软件的特性，所以分布式系统具有高度的内聚性和透明性。因此，网络和分布式系统之间的区别更多的在于高层软件（特别是操作系统），而不是硬件。分布式系统可以应用在在不同的平台上如：Pc、工作站、局域网和广域网上等。

简单来讲：
1分布式：不同的多台服务器上面部署不同的服务模块（工程），他们之间通过Rpc/Rmi之间通信和调用，对外提供服务和组内协作。

2集群：不同的多台服务器上面部署相同的服务模块，通过分布式调度软件进行统一的调度，对外提供服务和访问。

## AOF 的耐久性如何？

你可以配置 Redis 多久才将数据 `fsync` 到磁盘一次。

有三个选项：

- 每次有新命令追加到 AOF 文件时就执行一次 `fsync` ：非常慢，也非常安全。
- 每秒 `fsync` 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
- 从不 `fsync` ：将数据交给操作系统来处理。更快，也更不安全的选择。

推荐（并且也是默认）的措施为每秒 `fsync` 一次， 这种 `fsync` 策略可以兼顾速度和安全性。

总是 `fsync` 的策略在实际使用中非常慢， 即使在 Redis 2.0 对相关的程序进行了改进之后仍是如此 —— 频繁调用 `fsync` 注定了这种策略不可能快得起来。

## 如果 AOF 文件出错了，怎么办？

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。

当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

1. 为现有的 AOF 文件创建一个备份。
2. 使用 Redis 附带的 `redis-check-aof` 程序，对原来的 AOF 文件进行修复。

> ```
> $ redis-check-aof --fix
> ```

1. （可选）使用 `diff -u` 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。
2. 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。

# 11.Redis 怎么实现分布式锁？

Redis 为单线程模式，采用队列模式将并发访问变成串行访问，且多客户端对 Redis 的连接并不存在竞争关系。Redis 中可以使用 SETNX 命令实现分布式锁。一般使用 setnx(set if not exists) 指令，只允许被一个程序占有，使用完调用 del 释放锁。

# Redis 淘汰策略

```
1.volatile-lru：从已设置过期时间的数据集（server. db[i]. expires）中挑选最近最少使用的数据淘汰；

2.volatile-random：从已设置过期时间的数据集（server. db[i]. expires）中任意选择数据淘汰。

3.allkeys-lru：从数据集（server. db[i]. dict）中挑选最近最少使用的数据淘汰。

4.allkeys-random：从数据集（server. db[i]. dict）中任意选择数据淘汰。

5.volatile-ttl：从已设置过期时间的数据集（server. db[i]. expires）中挑选将要过期的数据淘汰。

6.no-enviction（驱逐）：禁止驱逐数据。
```

# 13.Redis的原子性如何保证

对于Redis而言，命令的原子性指的是：一个操作不可再分，操作要么执行要么不执行，Redis之所以是原子的，是因为Redis是单线程的。

Redis所有单个命令的执行都是原子的，Redis是实现事务的原理：

1.批量操作在发送EXEC（执行）命令前被放入队列缓存；

2.收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令都不会被执行；

3.在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。
