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

## 平时都拿redis做什么？ 

string 

* **存储key-value键值对，这个比较简单不细说了**

list 

* **消息队列：`lpop`和`rpush`（或者反过来，`lpush`和`rpop`）能实现队列的功能**
* **朋友圈的点赞列表、评论列表、排行榜：`lpush`命令和`lrange`命令能实现最新列表的功能，每次通过`lpush`命令往列表里插入新的元素，然后通过`lrange`命令读取最新的元素列表。**

hash 

- 购物车：`hset [key] [field] [value]` 命令， 可以实现以`用户Id`，`商品Id`为`field`，商品数量为`value`，恰好构成了购物车的3个要素。
- 存储对象：`hash`类型的`(key, field, value)`的结构与对象的`(对象id, 属性, 值)`的结构相似，也可以用来存储对象。

set

- **好友、关注、粉丝、感兴趣的人集合：**
  1. `sinter`命令可以获得A和B两个用户的共同好友；
  2. `sismember`命令可以判断A是否是B的好友；
  3. `scard`命令可以获取好友数量；
  4. 关注时，`smove`命令可以将B从A的粉丝集合转移到A的好友集合
- **首页展示随机：美团首页有很多推荐商家，但是并不能全部展示，set类型适合存放所有需要展示的内容，而`srandmember`命令则可以从中随机获取几个。**
- **存储某活动中中奖的用户ID ，因为有去重功能，可以保证同一个用户不会中奖两次。**

zset

应用场景：

* **`zset` 可以用做排行榜，但是和`list`不同的是`zset`它能够实现动态的排序，例如： 可以用来存储粉丝列表，value 值是粉丝的用户 ID，score 是关注时间，我们可以对粉丝列表按关注时间进行排序。**

* **`zset` 还可以用来存储学生的成绩， `value` 值是学生的 ID, `score` 是他的考试成绩。 我们对成绩按分数进行排序就可以得到他的名次。**

## redis缓存与数据库的一致性问题？ 

**针对读场景**

不管是从缓存中读还是从硬盘中读，数据没有经过修改，读取的值自然是一样的。不存在数据不一致问题。

**针对写场景**

1. 如果redis中本身不存在缓存数据，则直接修改db中的数据即可，不会产生数据不一致问题。
2. 如果redis中已存在缓存数据，则需要同时修改db和redis中的数据，但是二者修改操作的执行必然存在先后顺序。在高并发的场景下，就有可能产生数据不一致的情况。

方案1: 淘汰缓存策略

    优点：操作简单，直接将对应缓存删除即可
    缺点：由于缓存被删除后，下次的读请求无法命中缓存，需回源db，将数据重新写入redis

方案2: 更新缓存策略

    优点：缓存命中率高，只要缓存进行了更新，后续的读请求不会出现缓存未命中(cache miss)的情况
    缺点：在某些业务场景下，缓存更新的成本过大。且更新后的缓存不一定会被使用。

其实业界一般采用的都是**缓存淘汰策略**，而非缓存更新策略。原因有三：

* 大多数情况下，redis缓存中的数据并不是完全copy db中的数据，而是将db中多张表的数据进行了重新计算，筛选后更新到redis。如果在db某一张表的数据发生了变化的情况下，需要同步重新计算redis中的值，成本过高。
* 缓存更新后的新值，无法保证一定会有读请求命中，如果一直没有请求命中该部分冷数据，其实是产生了一定的资源浪费（计算成本+存储成本）。
* 相较于淘汰缓存策略中，仅有一次读请求cache miss的结果来说，淘汰缓存策略的缺点完全可以容忍。

**redis和db写操作，谁先谁后？**

注意：以下的方案讨论都是基于redis淘汰缓存操作以及数据库更新操作保证成功（可通过重试机制解决）的情况下，高并发的业务场景中的解决方案。
**写1：先更新数据库，再更新缓存(普通低并发)**

![image-20201106184914749](https://img-blog.csdnimg.cn/img_convert/bb518142aec2d0199c8ed0b0bcda3e57.png)

更新数据库信息，再更新Redis缓存。这是常规做法，缓存基于数据库，取自数据库。

但是其中可能遇到一些问题，例如上述如果更新缓存失败(宕机等其他状况)，将会使得数据库和Redis数据不一致。造成DB新数据，缓存旧数据。
**写2：先删除缓存，再写入数据库(低并发优化)**

![image-20201106184958339](https://img-blog.csdnimg.cn/img_convert/65286c874254e5ea1f599a5b76525c20.png)

解决的问题

这种情况能够有效避免写1中防止写入Redis失败的问题。将缓存删除进行更新。理想是让下次访问Redis为空去MySQL取得最新值到缓存中。但是这种情况仅限于低并发的场景中而不适用高并发场景。

存在的问题

写2虽然能够看似写入Redis异常的问题。看似较为好的解决方案但是在高并发的方案中其实还是有问题的。我们在写1讨论过如果更新库成功，缓存更新失败会导致脏数据。我们理想是删除缓存让下一个线程访问适合更新缓存。问题是：如果这下一个线程来的太早、太巧了呢？
![image-20201106191042265](https://img-blog.csdnimg.cn/img_convert/af20f12b114d64195439fc3091594209.png)

因为多线程你也不知道谁先谁后，谁快谁慢。如上图所示情况，将会出现Redis缓存数据和MySQL不一致。当然你可以对key进行上锁。但是锁这种重量级的东西对并发功能影响太大，能不用锁就别用！上述情况就高并发下依然会造成缓存是旧数据，DB是新数据。并且如果缓存没有过期这个问题会一直存在。
**写3：延时双删策略**

![image-20201106191310072](https://img-blog.csdnimg.cn/img_convert/b29dca00bfb541e806b2f5f911e0ac83.png)

这个就是延时双删策略，能过缓解在写2中在更新MySQL过程中有读的线程进入造成Redis缓存与MySQL数据不一致。方法就是删除缓存->更新缓存->延时(几百ms)(可异步)再次删除缓存。即使在更新缓存途中发生写2的问题。造成数据不一致，但是延时(具体实间根据业务来，一般几百ms)再次删除也能很快的解决不一致。

**延时再删除的目的就是为了规避，写操作刚完成，读操作就把读取到的旧数据更新到缓存中。**

但是就写的方案其实还是有漏洞的，比如第二次删除错误、多写多读高并发情况下对MySQL访问的压力等等。当然你可以选择用MQ等消息队列异步解决。
**写4：直接操作缓存，定期写入sql(适合高并发)**

当有一堆并发(写)扔过来的后，前面几个方案即使使用消息队列异步通信但也很难给用户一个舒适的体验。并且对大规模操作sql对系统也会造成不小的压力。所以还有一种方案就是直接操作缓存，将缓存定期写入sql。因为Redis这种非关系数据库又基于内存操作KV相比传统关系型要快很多。

![image-20201106192531468](https://img-blog.csdnimg.cn/img_convert/a483bad583e1f07b3c1a19a7150dc1f3.png)

上面适用于高并发情况下业务设计，这个时候以Redis数据为主，MySQL数据为辅助。定期插入(好像数据备份库一样)。当然，这种高并发往往会因为业务对读、写的顺序等等可能有不同要求，可能还要借助消息队列以及锁完成针对业务上对数据和顺序可能会因为高并发、多线程带来的不确定性和不稳定性，提高业务可靠性。


## 缓存穿透？如何解决？ 

#### 16.1. 什么是缓存穿透？

缓存穿透说简单点就是大量请求的 key 根本不存在于缓存中，导致请求直接到了数据库上，根本没有经过缓存这一层。举个例子：某个黑客故意制造我们缓存中不存在的 key 发起大量请求，导致大量请求落到数据库。

#### 

#### 16.2. 缓存穿透情况的处理流程是怎样的？

如下图所示，用户的请求最终都要跑到数据库中查询一遍。

[![缓存穿透情况](https://github.com/Snailclimb/JavaGuide/raw/master/docs/database/Redis/images/redis-all/%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F%E6%83%85%E5%86%B5.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/Redis/images/redis-all/缓存穿透情况.png)

#### 

#### 16.3. 有哪些解决办法？

最基本的就是首先做好参数校验，一些不合法的参数请求直接抛出异常信息返回给客户端。比如查询的数据库 id 不能小于 0、传入的邮箱格式不对的时候直接返回错误消息给客户端等等。

**1）缓存无效 key**

如果缓存和数据库都查不到某个 key 的数据就写一个到 Redis 中去并设置过期时间，具体命令如下： `SET key value EX 10086` 。这种方式可以解决请求的 key 变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求 key，会导致 Redis 中缓存大量无效的 key 。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效的 key 的过期时间设置短一点比如 1 分钟。

另外，这里多说一嘴，一般情况下我们是这样设计 key 的： `表名:列名:主键名:主键值` 。

如果用 Java 代码展示的话，差不多是下面这样的：

```
public Object getObjectInclNullById(Integer id) {
    // 从缓存中获取数据
    Object cacheValue = cache.get(id);
    // 缓存为空
    if (cacheValue == null) {
        // 从数据库中获取
        Object storageValue = storage.get(key);
        // 缓存空对象
        cache.set(key, storageValue);
        // 如果存储数据为空，需要设置一个过期时间(300秒)
        if (storageValue == null) {
            // 必须设置过期时间，否则有被攻击的风险
            cache.expire(key, 60 * 5);
        }
        return storageValue;
    }
    return cacheValue;
}
```

**2）布隆过滤器**

布隆过滤器是一个非常神奇的数据结构，通过它我们可以非常方便地判断一个给定数据是否存在于海量数据中。我们需要的就是判断 key 是否合法，有没有感觉布隆过滤器就是我们想要找的那个“人”。

具体是这样做的：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程。

加入布隆过滤器之后的缓存处理流程图如下。

[![image](https://github.com/Snailclimb/JavaGuide/raw/master/docs/database/Redis/images/redis-all/%E5%8A%A0%E5%85%A5%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8%E5%90%8E%E7%9A%84%E7%BC%93%E5%AD%98%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)](https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/Redis/images/redis-all/加入布隆过滤器后的缓存处理流程.png)

但是，需要注意的是布隆过滤器可能会存在误判的情况。总结来说就是： **布隆过滤器说某个元素存在，小概率会误判。布隆过滤器说某个元素不在，那么这个元素一定不在。**

*为什么会出现误判的情况呢? 我们还要从布隆过滤器的原理来说！*

我们先来看一下，**当一个元素加入布隆过滤器中的时候，会进行哪些操作：**

1. 使用布隆过滤器中的哈希函数对元素值进行计算，得到哈希值（有几个哈希函数得到几个哈希值）。
2. 根据得到的哈希值，在位数组中把对应下标的值置为 1。

我们再来看一下，**当我们需要判断一个元素是否存在于布隆过滤器的时候，会进行哪些操作：**

1. 对给定元素再次进行相同的哈希计算；
2. 得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

然后，一定会出现这样一种情况：**不同的字符串可能哈希出来的位置相同。** （可以适当增加位数组大小或者调整我们的哈希函数来降低概率）

## 持久化机制、淘汰策略？ 

很多时候我们需要持久化数据也就是将内存中的数据写入到硬盘里面，大部分原因是为了之后重用数据（比如重启机器、机器故障之后恢复数据），或者是为了防止系统故障而将数据备份到一个远程位置。

Redis 不同于 Memcached 的很重要一点就是，Redis 支持持久化，而且支持两种不同的持久化操作。**Redis 的一种持久化方式叫快照（snapshotting，RDB），另一种方式是只追加文件（append-only file, AOF）**。这两种方法各有千秋，下面我会详细这两种持久化方法是什么，怎么用，如何选择适合自己的持久化方法。

**快照（snapshotting）持久化（RDB）**

**Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本**。Redis  创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构，主要用来提高 Redis  性能），还可以将快照留在原地以便重启服务器的时候使用。

快照持久化是 Redis 默认采用的持久化方式，在 Redis.conf 配置文件中默认有此下配置：

```
save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
```

**AOF（append-only file）持久化**

与快照持久化相比，AOF 持久化 的实时性更好，因此已成为主流的持久化方案。默认情况下 Redis 没有开启 AOF（append only file）方式的持久化，可以通过 appendonly 参数开启：

```
appendonly yes
```

**开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件**。AOF 文件的保存位置和 RDB 文件的位置相同，都是通过 dir 参数设置的，默认的文件名是 appendonly.aof。

在 Redis 的配置文件中存在三种不同的 AOF 持久化方式，它们分别是：

```
appendfsync always    #每次有数据修改发生时都会写入AOF文件,这样会严重降低Redis的速度
appendfsync everysec  #每秒钟同步一次，显示地将多个写命令同步到硬盘
appendfsync no        #让操作系统决定何时进行同步
```

为了兼顾数据和写入性能，用户可以考虑 appendfsync everysec 选项 ，让 Redis 每秒同步一次 AOF  文件，Redis 性能几乎没受到任何影响。而且这样即使出现系统崩溃，用户最多只会丢失一秒之内产生的数据。当硬盘忙于执行写入操作的时候，Redis 还会优雅的放慢自己的速度以便适应硬盘的最大写入速度。

**拓展：Redis 4.0 对于持久化机制的优化**

Redis 4.0 开始支持 RDB 和 AOF 的混合持久化（默认关闭，可以通过配置项 `aof-use-rdb-preamble` 开启）。

如果把混合持久化打开，**fork 一个子线程将内存数据以RDB二进制格式写入AOF文件头部，那些在重写操作执行之后执行的 Redis 命令，则以AOF持久化的方式追加到AOF文件的末尾。**这样做的好处是可以结合 RDB 和 AOF  的优点, 快速加载同时避免丢失过多的数据。当然缺点也是有的， AOF 里面的 RDB 部分是压缩格式不再是 AOF 格式，可读性较差。

**补充内容：AOF 重写**

AOF 重写可以产生一个新的 AOF 文件，这个新的 AOF 文件和原有的 AOF 文件所保存的数据库状态一样，但体积更小。

AOF 重写是一个有歧义的名字，该功能是通过读取数据库中的键值对来实现的，程序无须对现有 AOF 文件进行任何读入、分析或者写入操作。

在执行 BGREWRITEAOF 命令时，Redis 服务器会维护一个 AOF 重写缓冲区，该缓冲区会在子进程创建新 AOF  文件期间，记录服务器执行的所有写命令。当子进程完成创建新 AOF 文件的工作之后，服务器会将重写缓冲区中的所有内容追加到新 AOF  文件的末尾，使得新旧两个 AOF 文件所保存的数据库状态一致。最后，服务器用新的 AOF 文件替换旧的 AOF 文件，以此来完成 AOF  文件重写操作



## setnx怎么用的？redisson锁原理？ 

### Setnx

目前通常所说的 Setnx 命令，并非单指 Redis 的 setnx key value 这条命令。

一般代指 Redis 中对 Set 命令加上 NX 参数进行使用，Set 这个命令，目前已经支持这么多参数可选：

```
SET key value [EX seconds|PX milliseconds] [NX|XX] [KEEPTTL]
```

当然了，就不在文章中默写 API 了，基础参数还有不清晰的，可以蹦到官网。



  ![0b8d57ab88fa1c9b09b95b248a9cb505.png](https://img-blog.csdnimg.cn/img_convert/0b8d57ab88fa1c9b09b95b248a9cb505.png) 

上图是笔者画的 Setnx 大致原理，主要依托了它的 Key 不存在才能 Set 成功的特性，进程 A 拿到锁，在没有删除锁的 Key 时，进程 B 自然获取锁就失败了。

那么为什么要使用 PX 30000 去设置一个超时时间？是怕进程 A 不讲道理啊，锁没等释放呢，万一崩了，直接原地把锁带走了，导致系统中谁也拿不到锁。



  ![6050edb39edccd0faa921a6cd635b6bd.png](https://img-blog.csdnimg.cn/img_convert/6050edb39edccd0faa921a6cd635b6bd.png) 

就算这样，还是不能保证万无一失。如果进程 A 又不讲道理，操作锁内资源超过笔者设置的超时时间，那么就会导致其他进程拿到锁，等进程 A 回来了，回手就是把其他进程的锁删了，如图：



  ![88de3f98e74e21892f519182d8c752ee.png](https://img-blog.csdnimg.cn/img_convert/88de3f98e74e21892f519182d8c752ee.png) 

还是刚才那张图，将 T5 时刻改成了锁超时，被 Redis 释放。

进程 B 在 T6 开开心心拿到锁不到一会，进程 A 操作完成，回手一个 Del，就把锁释放了。

当进程 B 操作完成，去释放锁的时候（图中 T8 时刻）：



  ![3ae496b5b4934cd970b506814bf481cc.png](https://img-blog.csdnimg.cn/img_convert/3ae496b5b4934cd970b506814bf481cc.png) 

找不到锁其实还算好的，万一 T7 时刻有个进程 C 过来加锁成功，那么进程 B 就把进程 C 的锁释放了。

以此类推，进程 C 可能释放进程 D 的锁，进程 D....(禁止套娃)，具体什么后果就不得而知了。

所以在用 Setnx 的时候，Key 虽然是主要作用，但是 Value 也不能闲着，可以设置一个唯一的客户端 ID，或者用 UUID 这种随机数。

当解锁的时候，先获取 Value 判断是否是当前进程加的锁，再去删除。伪代码：

```
String uuid = xxxx;
// 伪代码，具体实现看项目中用的连接工具
// 有的提供的方法名为set 有的叫setIfAbsent
set Test uuid NX PX 3000
try{
// biz handle....
} finally {
    // unlock
    if(uuid.equals(redisTool.get('Test')){
        redisTool.del('Test');
    }

}
```

这回看起来是不是稳了？相反，这回的问题更明显了，在 Finally 代码块中，Get 和 Del 并非原子操作，还是有进程安全问题。



下面给大家看一段简单的使用代码片段，先直观的感受一下：

![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205258812-1901485246.png)

 

 

 

怎么样，上面那段代码，是不是感觉简单的不行！

 

此外，人家还支持redis单实例、redis哨兵、redis cluster、redis master-slave等各种部署架构，都可以给你完美实现。

 

 

### Redisson实现Redis分布式锁的底层原理

好的，接下来就通过一张手绘图，给大家说说Redisson这个开源框架对Redis分布式锁的实现原理。![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205347970-1529033883.png)  

**（1）加锁机制** 

咱们来看上面那张图，现在某个客户端要加锁。如果该客户端面对的是一个redis cluster集群，他首先会根据hash节点选择一台机器。 

**这里注意**，仅仅只是选择一台机器！这点很关键！ 

紧接着，就会发送一段lua脚本到redis上，那段lua脚本如下所示：

![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205357505-171041515.png)

为啥要用lua脚本呢？

因为一大坨复杂的业务逻辑，可以通过封装在lua脚本中发送给redis，保证这段复杂业务逻辑执行的**原子性**。

那么，这段lua脚本是什么意思呢？

**KEYS[1]**代表的是你加锁的那个key，比如说：

RLock lock = redisson.getLock("myLock");

这里你自己设置了加锁的那个锁key就是“myLock”。 

**ARGV[1]**代表的就是锁key的默认生存时间，默认30秒。

**ARGV[2]**代表的是加锁的客户端的ID，类似于下面这样：

8743c9c0-0795-4907-87fd-6c719a6b4586:1

给大家解释一下，第一段if判断语句，就是用“exists myLock”命令判断一下，如果你要加锁的那个锁key不存在的话，你就进行加锁。

如何加锁呢？很简单，用下面的命令：

hset myLock 

  8743c9c0-0795-4907-87fd-6c719a6b4586:1 1

通过这个命令设置一个hash数据结构，这行命令执行后，会出现一个类似下面的数据结构：

![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205406379-341931221.png)

上述就代表“8743c9c0-0795-4907-87fd-6c719a6b4586:1”这个客户端对“myLock”这个锁key完成了加锁 

接着会执行“pexpire myLock 30000”命令，设置myLock这个锁key的生存时间是30秒。

好了，到此为止，ok，加锁完成了

**（2）锁互斥机制**

那么在这个时候，如果客户端2来尝试加锁，执行了同样的一段lua脚本，会咋样呢？

很简单，第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在了。

接着第二个if判断，判断一下，myLock锁key的hash数据结构中，是否包含客户端2的ID，但是明显不是的，因为那里包含的是客户端1的ID。

所以，客户端2会获取到pttl myLock返回的一个数字，这个数字代表了myLock这个锁key的**剩余生存时间。**比如还剩15000毫秒的生存时间。

此时客户端2会进入一个while循环，不停的尝试加锁

**（3）watch dog自动延期机制**

客户端1加锁的锁key默认生存时间才30秒，如果超过了30秒，客户端1还想一直持有这把锁，怎么办呢？ 

简单！只要客户端1一旦加锁成功，就会启动一个watch dog看门狗，**他是一个后台线程，会每隔10秒检查一下**，如果客户端1还持有锁key，那么就会不断的延长锁key的生存时间。  

**（4）可重入加锁机制** 

那如果客户端1都已经持有了这把锁了，结果可重入的加锁会怎么样呢？

比如下面这种代码：

![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205416247-221660573.png)

这时我们来分析一下上面那段lua脚本。

第一个if判断肯定不成立，“exists myLock”会显示锁key已经存在了。

第二个if判断会成立，因为myLock的hash数据结构中包含的那个ID，就是客户端1的那个ID，也就是“8743c9c0-0795-4907-87fd-6c719a6b4586:1”

此时就会执行可重入加锁的逻辑，他会用：

incrby myLock 

 8743c9c0-0795-4907-87fd-6c71a6b4586:1 1

通过这个命令，对客户端1的加锁次数，累加1。

此时myLock数据结构变为下面这样：

![img](https://img2018.cnblogs.com/i-beta/1377406/202001/1377406-20200101205424854-570061070.png)

大家看到了吧，那个myLock的hash数据结构中的那个客户端ID，就对应着加锁的次数 

**（5）释放锁机制**

 如果执行lock.unlock()，就可以释放分布式锁，此时的业务逻辑也是非常简单的。

其实说白了，就是每次都对myLock数据结构中的那个加锁次数减1。

 如果发现加锁次数是0了，说明这个客户端已经不再持有锁了，此时就会用：

“del myLock”命令，从redis里删除这个key。

 然后呢，另外的客户端2就可以尝试完成加锁了。

 这就是所谓的**分布式锁的开源Redisson框架的实现机制。**

一般我们在生产系统中，可以用Redisson框架提供的这个类库来基于redis进行分布式锁的加锁与释放锁。

**（6）上述Redis分布式锁的缺点**

其实上面那种方案最大的问题，就是如果你对某个redis master实例，写入了myLock这种锁key的value，此时会异步复制给对应的master slave实例。

但是这个过程中一旦发生redis master宕机，主备切换，redis slave变为了redis master。

接着就会导致，客户端2来尝试加锁的时候，在新的redis master上完成了加锁，而客户端1也以为自己成功加了锁。

此时就会导致多个客户端对一个分布式锁完成了加锁。

这时系统在业务语义上一定会出现问题，**导致各种脏数据的产生**。

所以这个就是redis cluster，或者是redis master-slave架构的**主从异步复制**导致的redis分布式锁的最大缺陷：在redis master实例宕机的时候，可能导致多个客户端同时完成加锁。