## 1.讲一下字典树结构，如何实现敏感词过滤

首先，在进行敏感词过滤之前我们需要有一个敏感词库，用这个敏感词库来建立字典树，比如我们现在有两个敏感词：“de”, “bca”，建立字典树，根节点不存放任何东西，其他每个子节点存放一个字符，深红色代表单词结尾，如下：
![![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713205451638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70#pic_center](https://img-blog.csdnimg.cn/20190713211809447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70#pic_center)
建立完字典树之后，我们就要利用这棵由敏感词组成的字典树匹配字符串了。

现在，我们有一段文本 “abcadef" ，目的是将其中的 “de”, “bca”,过滤掉，具体算法如下：

为了遍历字符串和字典树，我们假设有三个指针，p1,p2,p3，其中p1指向字典树根节点，p2 和 p3 指向字符串的第一个字符，如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713212113327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

然后从字符串的 a 开始，检测有没有以 a 作为前缀的敏感词，直接判断 p1 的孩子节点中是否有 a 这个节点就可以了，显然这里没有。接着把指针 p2 和 p3 向右移动一格。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713212000687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

然后从字符串 b 开始查找，看看是否有以 b 作为前缀的字符串，p1 的孩子节点中有 b，这时，我们把 p1 指向节点 b，由于此时 b 不是单词的结尾，所以p3 向右移动一格，不过，p2 不动。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713212338870.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

判断 p1 的孩子节点中是否存在 p3 指向的字符 c，显然有。我们把 p1 指向节点 c，p3 向右移动一格，p2 不动。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713212537429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

判断 p1 的孩子节点中是否存在 p3 指向的字符 a，显然有，且 a 是字符串 “bca” 的结尾。这意味着，p2 到 p3之间为敏感词 “bca”，把 p2 和 p3 指向的区间那些字符替换成 *。这时我们把 p2 和 p3 都移向字符 d，p1 还是还原到最开始指向 root。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713212936755.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

和前面的步骤一样，判断有没以 d 作为前缀的字符串，显然这里有 “de”，所以把 p2 和 p3 移到字符 f。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713213317876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTAwMDgx,size_16,color_FFFFFF,t_70)

因为根节点没有子节点 f，所以匹配结束。

在 Java 中可以利用 HashMap 来存放一层树结点，则每个敏感词的查找时间复杂度是 O (1)，字符串的长度为 n，我们需要遍历 1 遍，所以敏感词查找这个过程的时间复杂度是 O (n * 1)。如果每个敏感词的平均长度为 m，有 t 个敏感词的话，构建 trie 树的时间复杂度是 O (t * m)。



##  2、如何使用[redis]()实现关注、点赞的功能 

### 数据库设计

- Redis数据库设计
   `Redis`是`K-V`数据库，没有统一的数据结构，针对不同的功能点设计了不同的`K-V`存储结构

  - 用户某篇文章的点赞数，都有谁点赞我的某篇文章
     使用`HashMap`数据结构，`HashMap`中的`key`为`articleId`，`value`为`Set`，`Set`中的值为用户`ID`，即`HashMap<String, Set<String>>`
  - 用户总的点赞数
     使用`HashMap`数据结构，`HashMap`中的`key`为`userId`，`value`为`String`记录总的点赞数
  - 用户点赞的文章，我都点赞了哪些篇文章
     使用`HashMap`数据结构，`HashMap`中的`key`为`userId`，`value`为`Set`，`Set`中的值为文章`ID`，即`HashMap<String, Set<String>>`

- MySQL数据库设计
   最主要的两张表，`article`表和`user_like_article`

  - `article`表结构

  |      字段值      | 字段类型 |     说明     |
  | :--------------: | :------: | :----------: |
  |   article_name   | varchar  |   文章名字   |
  |     content      |   blob   |   文章内容   |
  | total_like_count |  bigint  | 文章总点赞数 |

  文章总的点赞数需要和`Redis`中的点赞数进行同步

  - `user_like_article`表结构

  |   字段值   | 字段类型 |  说明  |
  | :--------: | :------: | :----: |
  |  user_id   |  bigint  | 用户ID |
  | article_id |  bigint  | 文章ID |

  记录用户点赞文章的信息，是一张中间表

说明：表结构设计省略了`id`、`deleted`、`gmt_create`、`gmt_modified`字段

### 流程图

![img](https:////upload-images.jianshu.io/upload_images/9358011-5251b67dd6264394.png?imageMogr2/auto-orient/strip|imageView2/2/w/742)

流程图

流程图比较简单，点赞和取消点赞基本实现步骤相同

- 参数校验
   对传入的参数进行`null`值判断
- 逻辑校验
   对于用户点赞，用户不能重复点赞相同的文章
   对于取消点赞，用户不能取消未点赞的文章
- 存入`Redis`
   存入的数据主要有所有文章的点赞数（我的所有文章的点赞数）、某篇文章的点赞数、用户点赞的文章
- 定时任务
   通过定时【1小时执行一次】，从`Redis`读取数据持久化到`MySQL`中

## 3、redis的数据结构有哪些，分别使用于哪些场景 

#### 一、String（字符串）

在任何一种编程语言里，字符串`String`都是最基础的数据结构， 那你有想过`Redis`中存储一个字符串都进行了哪些操作嘛？

在`Redis`中`String`是可以修改的，称为`动态字符串`(`Simple Dynamic String` 简称 `SDS`)（**快拿小本本记名词，要考的**），说是字符串但它的内部结构更像是一个 `ArrayList`，内部维护着一个字节数组，并且在其内部预分配了一定的空间，以减少内存的频繁分配。

`Redis`的内存分配机制是这样：

- 当字符串的长度小于 1MB时，每次扩容都是加倍现有的空间。
- 如果字符串长度超过 1MB时，每次扩容时只会扩展 1MB 的空间。

这样既保证了内存空间够用，还不至于造成内存的浪费，**字符串最大长度为 `512MB`.**。

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/25/171101faad0e645c?imageView2/0/w/1280/h/960/ignore-error/1)

> 以上图片源自网络，如有侵权联系删除

上图就是字符串的基本结构，其中 `content` 里面保存的是字符串内容，`0x\0`作为结束字符不会被计算`len`中。

分析一下字符串的数据结构

```
struct SDS{
  T capacity;       //数组容量
  T len;            //实际长度
  byte flages;  //标志位,低三位表示类型
  byte[] content;   //数组内容
}

```

`capacity` 和 `len`两个属性都是泛型，为什么不直接用`int类型`？因为`Redis`内部有很多优化方案，为更合理的使用内存，不同长度的字符串采用不同的数据类型表示，且在创建字符串的时候 `len` 会和 `capacity` 一样大，不产生冗余的空间，所以`String`值可以是字符串、数字（整数、浮点数) 或者 二进制。

**1、应用场景：**

**存储key-value键值对，这个比较简单不细说了**

**2、字符串（String）常用的命令：**

```
set   [key]  [value]   给指定key设置值（set 可覆盖老的值）

get  [key]   获取指定key 的值

del  [key]   删除指定key

exists  [key]  判断是否存在指定key

mset  [key1]  [value1]  [key2]  [value2] ...... 批量存键值对

mget  [key1]  [key2] ......   批量取key

expire [key]  [time]    给指定key 设置过期时间  单位秒

setex    [key]  [time]  [value]  等价于 set + expire 命令组合

setnx  [key]  [value]   如果key不存在则set 创建，否则返回0

incr   [key]           如果value为整数 可用 incr命令每次自增1

incrby  [key] [number]  使用incrby命令对整数值 进行增加 number
复制代码
```

------

#### 二、list(列表)

`Redis`中的`list`和`Java`中的`LinkedList`很像，底层都是一种链表结构， `list`的插入和删除操作非常快，时间复杂度为 0(1)，不像数组结构插入、删除操作需要移动数据。

像归像，但是`redis`中的`list`底层可不是一个双向链表那么简单。

**当数据量较少的时候它的底层存储结构为一块连续内存，称之为`ziplist(压缩列表)`，它将所有的元素紧挨着一起存储，分配的是一块连续的内存；当数据量较多的时候将会变成`quicklist(快速链表)`结构。**

可单纯的链表也是有缺陷的，链表的前后指针 `prev` 和 `next` 会占用较多的内存，会比较浪费空间，而且会加重内存的碎片化。在redis 3.2之后就都改用`ziplist+链表`的混合结构，称之为 `quicklist(快速链表)`。

下面具体介绍下两种链表

#### ziplist(压缩列表)

先看一下`ziplist`的数据结构，

```
struct ziplist<T>{
    int32 zlbytes;            //压缩列表占用字节数
    int32 zltail_offset;    //最后一个元素距离起始位置的偏移量,用于快速定位到最后一个节点
    int16 zllength;            //元素个数
    T[] entries;            //元素内容
    int8 zlend;                //结束位 0xFF
}
复制代码
```

`int32 zlbytes`： 压缩列表占用字节数 `int32 zltail_offset`： 最后一个元素距离起始位置的偏移量,用于快速定位到最后一个节点 `int16 zllength`：元素个数 `T[] entries`：元素内容 `int8 zlend`：结束位 0xFF

压缩列表为了支持双向遍历，所以才会有 `ztail_offset` 这个字段，用来快速定位到最后一 个元素，然后倒着遍历

![在这里插入图片描述](https://user-gold-cdn.xitu.io/2020/3/25/171101faad102812?imageView2/0/w/1280/h/960/ignore-error/1)

> 以上图片源自网络，如有侵权联系删除

`entry`的数据结构：

```
struct entry{
    int<var> prevlen;            //前一个 entry 的长度
    int<var> encoding;            //元素类型编码
    optional byte[] content;    //元素内容
}
复制代码
```

`entry`它的 `prevlen` 字段表示前一个 `entry` 的字节长度，当压缩列表倒着遍历时，需要通过这 个字段来快速定位到下一个元素的位置。

**1、应用场景：**

由于list它是一个按照插入顺序排序的列表，所以应用场景相对还较多的，例如：

- **消息队列：`lpop`和`rpush`（或者反过来，`lpush`和`rpop`）能实现队列的功能**
- **朋友圈的点赞列表、评论列表、排行榜：`lpush`命令和`lrange`命令能实现最新列表的功能，每次通过`lpush`命令往列表里插入新的元素，然后通过`lrange`命令读取最新的元素列表。**

**2、list操作的常用命名：**

```
rpush  [key] [value1] [value2] ......    链表右侧插入

rpop    [key]  移除右侧列表头元素，并返回该元素

lpop   [key]    移除左侧列表头元素，并返回该元素

llen  [key]     返回该列表的元素个数

lrem [key] [count] [value]  删除列表中与value相等的元素，count是删除的个数。 count>0 表示从左侧开始查找，删除count个元素，count<0 表示从右侧开始查找，删除count个相同元素，count=0 表示删除全部相同的元素

(PS:   index 代表元素下标，index 可以为负数， index= 表示倒数第一个元素，同理 index=-2 表示倒数第二 个元素。)

lindex [key] [index]  获取list指定下标的元素 （需要遍历，时间复杂度为O(n)）

lrange [key]  [start_index] [end_index]   获取list 区间内的所有元素 （时间复杂度为 O（n））

ltrim  [key]  [start_index] [end_index]   保留区间内的元素，其他元素删除（时间复杂度为 O（n））
复制代码
```

------

#### 三、hash （字典）

`Redis` 中的 `Hash`和 Java的 `HashMap` 更加相似，都是`数组+链表`的结构，当发生 hash 碰撞时将会把元素追加到链表上，值得注意的是在 `Redis` 的 `Hash` 中 `value` 只能是字符串.

```
hset books java "Effective java" (integer) 1
hset books golang "concurrency in go" (integer) 1
hget books java "Effective java"
hset user age 17 (integer) 1
hincrby user age 1	#单个 key 可以进行计数 和 incr 命令基本一致 (integer) 18
复制代码
```

**`Hash` 和`String`都可以用来存储用户信息 ，但不同的是`Hash`可以对用户信息的每个字段单独存储；`String`存的是用户全部信息经过序列化后的字符串，如果想要修改某个用户字段必须将用户信息字符串全部查询出来，解析成相应的用户信息对象，修改完后在序列化成字符串存入。而 hash可以只对某个字段修改，从而节约网络流量，不过hash内存占用要大于 `String`，这是 `hash` 的缺点。**

**1、应用场景：**

- 购物车：`hset [key] [field] [value]` 命令， 可以实现以`用户Id`，`商品Id`为`field`，商品数量为`value`，恰好构成了购物车的3个要素。
- 存储对象：`hash`类型的`(key, field, value)`的结构与对象的`(对象id, 属性, 值)`的结构相似，也可以用来存储对象。

**2、hash常用的操作命令：**

```
hset  [key]  [field] [value]    新建字段信息

hget  [key]  [field]    获取字段信息

hdel [key] [field]  删除字段

hlen  [key]   保存的字段个数

hgetall  [key]  获取指定key 字典里的所有字段和值 （字段信息过多,会导致慢查询 慎用：亲身经历 曾经用过这个这个指令导致线上服务故障）

hmset  [key]  [field1] [value1] [field2] [value2] ......   批量创建

hincr  [key] [field]   对字段值自增

hincrby [key] [field] [number] 对字段值增加number
复制代码
```

------

#### 四、set(集合)

`Redis` 中的 `set`和`Java`中的`HashSet` 有些类似，它内部的键值对是无序的、唯一 的。它的内部实现相当于一个特殊的字典，字典中所有的value都是一个值 NULL。当集合中最后一个元素被移除之后，数据结构被自动删除，内存被回收。

**1、应用场景：**

- **好友、关注、粉丝、感兴趣的人集合：**
  1. `sinter`命令可以获得A和B两个用户的共同好友；
  2. `sismember`命令可以判断A是否是B的好友；
  3. `scard`命令可以获取好友数量；
  4. 关注时，`smove`命令可以将B从A的粉丝集合转移到A的好友集合
- **首页展示随机：美团首页有很多推荐商家，但是并不能全部展示，set类型适合存放所有需要展示的内容，而`srandmember`命令则可以从中随机获取几个。**
- **存储某活动中中奖的用户ID ，因为有去重功能，可以保证同一个用户不会中奖两次。**

**2、set的常用命令：**

```
sadd  [key]  [value]  向指定key的set中添加元素

smembers [key]    获取指定key 集合中的所有元素

sismember [key] [value]   判断集合中是否存在某个value

scard [key]    获取集合的长度

spop  [key]   弹出一个元素

srem [key] [value]  删除指定元素
复制代码
```

------

#### 五、zset(有序集合)

`zset`也叫`SortedSet`一方面它是个 `set` ，保证了内部 value 的唯一性，另方面它可以给每个 value 赋予一个`score`，代表这个value的排序权重。它的内部实现用的是一种叫作“`跳跃列表`”的数据结构。

**1、应用场景：**

**`zset` 可以用做排行榜，但是和`list`不同的是`zset`它能够实现动态的排序，例如： 可以用来存储粉丝列表，value 值是粉丝的用户 ID，score 是关注时间，我们可以对粉丝列表按关注时间进行排序。**

**`zset` 还可以用来存储学生的成绩， `value` 值是学生的 ID, `score` 是他的考试成绩。 我们对成绩按分数进行排序就可以得到他的名次。**

**2、zset有序集合的常用操作命令：**

```
zadd [key] [score] [value] 向指定key的集合中增加元素

zrange [key] [start_index] [end_index] 获取下标范围内的元素列表，按score 排序输出

zrevrange [key] [start_index] [end_index]  获取范围内的元素列表 ，按score排序 逆序输出

zcard [key]  获取集合列表的元素个数

zrank [key] [value]  获取元素再集合中的排名

zrangebyscore [key] [score1] [score2]  输出score范围内的元素列表

zrem [key] [value]  删除元素

zscore [key] [value] 获取元素的score
```



##  4、redis的过期机制了解么 

在聊这个问题之前，一定要明确的一件事情是，如非必要，任何进入缓存的数据都应该设置过期时间，因为内存的大小是有限的，一台机器可能就那么几十个 G ，你不能拿内存和硬盘比，一台机器硬盘几个 T 都是洒洒水，只要想装，几十个 T 都装得下，关键还不贵。

Redis 设置删除策略，主要有两种思路，一种是定期删除，另一种是惰性删除。

**定期删除：**

设定一个时间，在 Redis 中默认是每隔 100ms 就随机抽取一些设置了过期时间的 key，检查其是否过期，如果过期就删除。

注意这里是随机抽取一些设置了过期时间的 key ，而是扫描所有，试想这样一个场景，如果我有 100w 个设置了过期时间的 key ，如果每次都全部扫描一遍，基本上 Redis 就死了， CPU 的负载会非常的高，全部都消耗在了检查过期 key 上面。

**惰性删除：**

惰性删除的意思就是当 key 过期后，不做删除动作，等到下次使用的时候，发现 key 已经过期，这时不在返回这个 key 对应的 value ，直接将这个 key 删除掉。

这种方式有一个致命的弱点，就是会有很多过期的 key-value 明明已经到了过期时间，缺还在内存中占着使用空间，大大降低了内存使用效率。

**所以 Redis 的过期策略是：定期删除 + 惰性删除。**

**实际上简单的定期删除 + 惰性删除还是会存在问题，定期删除可能会导致很多过期 key 到了时间并没有被删除掉，然后我们也没有及时的去做检查，也没有做惰性删除，此时的结果就是大量过期 key 堆积在内存里，导致 Redis 的内存被耗尽。**

咋整？答案是**走内存淘汰机制。**

### 内存淘汰机制都有哪些？

Redis 内存淘汰机制有以下几个：

- **noeviction**: 当内存不足以容纳新写入数据时，新写入操作会报错。这个一般没啥人用吧，太傻了。
- **allkeys-lru**：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的 key（这个是最常用的）。
- **allkeys-random**：当内存不足以容纳新写入数据时，在键空间中，随机移除某个 key。这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。
- **volatile-lru**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key（这个一般不太合适）。
- **volatile-random**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key。这个一般也没人用吧。
- **volatile-ttl**：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除。

### LRU算法

LRU全称是Least Recently Used，即最近最久未使用的意思。
LRU算法的设计原则是：如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小。也就是说，当限定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。

1. 用一个数组来存储数据，给每一个数据项标记一个访问时间戳，每次插入新数据项的时候，先把数组中存在的数据项的时间戳自增，并将新数据项的时间戳置为0并插入到数组中。每次访问数组中的数据项的时候，将被访问的数据项的时间戳置为0。当数组空间已满时，将时间戳最大的数据项淘汰。
2. 利用一个链表来实现，每次新插入数据的时候将新数据插到链表的头部；每次缓存命中（即数据被访问），则将数据移到链表头部；那么当链表满的时候，就将链表尾部的数据丢弃。

3. 利用链表和hashmap。当需要插入新的数据项的时候，如果新数据项在链表中存在（一般称为命中），则把该节点移到链表头部，如果不存在，则新建一个节点，放到链表头部，若缓存满了，则把链表最后一个节点删除即可。在访问数据的时候，如果数据项在链表中存在，则把该节点移到链表头部，否则返回-1。这样一来在链表尾部的节点就是最近最久未访问的数据项。

对于第一种方法， 需要不停地维护数据项的访问时间戳，另外，在插入数据、删除数据以及访问数据时，时间复杂度都是O(n)。对于第二种方法，链表在定位数据的时候时间复杂度为O(n)。所以在一般使用第三种方式来是实现LRU算法。
实现方案

**使用LinkedHashMap实现**
     LinkedHashMap底层就是用的HashMap加双链表实现的，而且本身已经实现了按照访问顺序的存储。此外，LinkedHashMap中本身就实现了一个方法removeEldestEntry用于判断是否需要移除最不常读取的数，方法默认是直接返回false，不会移除元素，所以需要重写该方法。即当缓存满后就移除最不常用的数。
    

    public class LRU<K,V> {
     
      private static final float hashLoadFactory = 0.75f;
      private LinkedHashMap<K,V> map;
      private int cacheSize;
     
      public LRU(int cacheSize) {
        this.cacheSize = cacheSize;
        int capacity = (int)Math.ceil(cacheSize / hashLoadFactory) + 1;
        map = new LinkedHashMap<K,V>(capacity, hashLoadFactory, true){
          private static final long serialVersionUID = 1;
     
          @Override
          protected boolean removeEldestEntry(Map.Entry eldest) {
            return size() > LRU.this.cacheSize;
          }
        };
      }
     
      public synchronized V get(K key) {
        return map.get(key);
      }
     
      public synchronized void put(K key, V value) {
        map.put(key, value);
      }
     
      public synchronized void clear() {
        map.clear();
      }
     
      public synchronized int usedSize() {
        return map.size();
      }
     
      public void print() {
        for (Map.Entry<K, V> entry : map.entrySet()) {
          System.out.print(entry.getValue() + "--");
        }
        System.out.println();
      }
    }



当存在热点数据时，LRU的效率很好，**但偶发性的、周期性的批量操作会导致LRU命中率急剧下降，缓存污染情况比较严重。**