# redis

## 简介

~~~c
NoSQL--非关系型数据库，not only sql
解决高并发，磁盘IO慢的问题；
redis(Remote Dictionary Service 远程字典服务器) //基于key-value型，不区分大小写
https://www.redis.io --官网
启动：redis-cli
select 0 //切换数据库，默认支持16个数据库(0~15)

~~~



## 五种数据类型

~~~c
1.基本操作
set k1 100  //设置键值
get k1      //获取值
del k1      //删除键值
    
keys *    //打印所有的key
keys ?    //匹配k1这种
keys ??   //匹配k12这种
dbsize    //key值数量

flushdb   //清空当前数据库
flushall  //清空所有数据库
    
move key [num]  //把key从当前库移到目标库
expire k1 10    //设置k1为10s后过期
ttl k1          //看剩余时间
    
2.对string类型的操作
getrange k1 0 -1       //返回字符串字串，0表示开头，-1表示尾
setrange k1 0 hello    //从头开始，替换对应长度的字符串
mget/mset              //获取设定多个key值
getset k1 123          //获取旧值，设定新值
setex k1 seconds value //将value关联到key，并设定过期时间
//key对应的value要为数值型
incr k1                //k1加1
incrby k1 100          //k1加100
    
3.对list类型的操作
lpush key value1 value2 ... //头插元素
rpush key value1 value2 ... //尾插元素
lpop key  //队头出队
rpop key  //队尾出队
lrange key start stop  //获取指定范围内的元素  (头0，尾-1)
lset key index value   //通过下标(从0开始)设置列表元素的值
lindex key index       //通过下标获取列表中的元素
lrem key count value   //从队头开始移除count个值为value的列表元素
ltrim key start stop   //列表只保留指定区间范围内的元素
linsert key before|after pivot value //在列表的元素(pivot)前或后插入元素(value)
    
4.对set类型的操作
无序/不重复，哈希表的实现
sadd key member1 member2 ...   //向集合中添加一个或多个成员
scard key    //获取集合的成员数
smembers key  //返回集合中的所有成员
sismember key member  //判断member元素是否是集合key的成员
smove source destination member   //将member元素从source集合移动到destination集合
srem key value  //删除集合中值为value的元素
srand member key num  //在集合中随机选出num个数(公司抽奖)
spop key [num]    //移除并返回集合中的一个/num个随机元素
sdiff key1 key2   //在key1中但不在key2中的元素
sinter key1 key2  //求交集
sunin key1 key2   //求并集
    
5.对sorted set(set)类型的操作
每个元素都会关联一个double类型的分数，redis正是通过分数来为集合中的成员排序；
有序集合中的成员是唯一的，但分数去可以重复；
zadd key score1 member1 [score2 member2] //向有序集合中添加一个或多个成员，或更新已存在成员的分数
zcard key  //获取有序集合的成员数
zcount key min max  //计算有序集合指定区间分数的成员数
zrange key start syop [withscores]  //通过索引区间返回有序集合指定区间内的成员
          (start stop)  表示开区间
zrangebylex key min max [limit offset count]  //通过字典序返回有序集合的成员
                 -   +   --头到尾
               [str1  [str2  --闭区间
zrangebyscore key min max [withscores] [limit]  //通过分数区间返回有序集合指定区间内的成员
zscore key member  //返回有序集合中成员的分数值
zrevrangebyscore key max min [withscores]  //返回有序集合中指定分数区间内的成员，分数从高到低排列
                       
6.hash
是一个string 类型的fiel(字段)和value(值)的映射表，hash适合用于存储对象；
key-value的模式不变，但value是一个键值对，map<key,map<key1,value>>
hset key field value  //将哈希表中的字段field的值设为value
hmset ....            //设置多个
hget key field/hmget ... //获取给定字段的值
hgetall key  //获取在哈希表中指定key的所有字段和值
hkeys key    //获取所有哈希表中的字段
hvals key   //获取哈希表中的所有值
hlen key    //获取哈希表字段的数量                   
  
7.配置文件
cd /etc/redis
~~~

## 持久化

~~~c
1.概念
需要定期将redis中的数据以某种形式(数据或命令)从内存保存到硬盘；
RDB是将当前数据保存到硬盘(原理是将redis在内存中的数据库记录定时dump到磁盘上的RDB持久化)。
AOF则是将每次执行的写命令保存到硬盘(原理是将Reids的操作日志以追加的方式写入文件，类似于MySQL的binlog).
 
//redis启动的时候先以AOF方式加载
//启动：sudo redis-server /etc/redis/6379.conf
2.RDB
a.执行save（阻塞,只管保存快照,其他的等待）或者是bgsave（异步）命令;
b.在指定的时间间隔内，执行指定次数的写操作;
c. 执行flushall 命令，清空数据库所有数据，意义不大;
d. 执行shutdown 命令，保证服务器正常关闭且不丢失任何数据，意义不大。
//优点：
1.适合大规模的数据恢复，与AOF相比，在恢复大的数据集的时候，RDB方式会更快一些。
2.如果业务对数据完整性和一致性要求不高，RDB是很好的选择。
//缺点：
1.数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
2.备份时占用内存，因为Redis在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍），最后再将临时文件替换之前的备份文件。所以Redis的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。

3.AOF
a.AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序在将指令都执行一遍。
b.AOF的运作方式是不断的将命令追加到文件的末尾，所以随着写入命令的不断增加，AOF文件的体积会变得越来越大。为了处理这种情况，Redis支持一种有趣的特性：可以在不打断服务客户端的情况下，对AOF文件进行重写（rewrite）
    redis会记录上次重写的时候AOF的大小，默认配置的AOF的大小是上次重写后大小的一倍，且文件大于64M的时候触发。
    	auto-aof-rewrite-min-size 64mb
    如果appendonly.aof由于之前shutdown的时候坏了，修复命令如下：sudo redis-check-aof --fix 文件名
//优点：
aof方式数据完整性和一致性更好。
//缺点：
因为记录了每条写数据的指令，数据恢复时把所有指令执行一次，开销比较大，aof文件的体积也容易比较大。
~~~

## redis中的事务

~~~c
//redis中的事务没有原子性，但每条命令本身具有原子性
redis的事务可以一次执行多个命令，本质是一组命令的集合；一个事务中的所有命令都会序列化，按顺序的串行化执行，而不会被其他命令插入。
//redis事务中的命令
multi  //开始事务
set k2 200  //命令入队
exec   //执行事务
discard  //取消事务
watch key [key ...] //监视一个或多个key，如果在事务执行之前这个key被其他命令所改动，那么事务将被打断。
unwatch  //取消watch命令对所有key的监视
一旦执行exec,watch监控就会取消。
//watch举例
#右边执行操作
127.0.0.1:6379> watch k1
OK 
127.0.0.1:6379> multi
OK 
127.0.0.1:6379> set k1 200
QUEUED 
127.0.0.1:6379> get k1
QUEUED

#左边进行操作
127.0.0.1:6379> set k1 300
OK
127.0.0.1:6379> 

#在接着在右边
127.0.0.1:6379> exec
(nil)
//命令出错
事务执行中有错误的命令：部分执行成功or全部不执行？
	如果输入命令后显示的是QUEUED，表示入队成功，部分执行。
	如果提示ERROR，事务被放弃，全部不执行。
	总结：编译时出错和运行时出错的区别。

~~~

## 乐观锁和悲观锁

~~~c
1.悲观锁
每次去拿别人的数据的时候都会认为别人会修改，所以都会上锁，其他线程想要访问数据的时候只能阻塞挂起。
2.乐观锁(冲突检测和数据更新)
a.概念
每次去拿数据的时候都认为别人不会修改，所以不会上锁;
但是在更新的时候会判断一下在此期间别人有没有去更新这个数据;
可以使用版本号等机制,乐观锁适用于写比较少的情况下(冲突发生的少点)，这样可以提高吞吐量。
b.乐观锁策略
提交版本必须大于记录当前版本才能执行更新，一般会使用版本号机制或CAS操作实现：
(1).version方式
一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加一。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。
(2).CAS(Check And Set先检查再设置) //也叫无锁编程
涉及到三个操作数，数据所在的内存值V，预期值A，新值B。当需要更新时，判断当前内存值V与之前取到的值A是否相等，若相等，则用新值更新，若失败则重试，一般情况下是一个自旋操作，即不断的重试。

 
~~~

## 主从复制

~~~c
1.概念
持久化侧重解决的是Redis数据的单机备份问题（从内存到硬盘的备份）；而主从复制则侧重解决数据的多机热备。此外，主从复制还可以实现负载均衡和故障恢复。
主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。
    
2.作用
a.数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
b.故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
c.负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
d.高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

3.配置
a.修改三个配置文件
/etc/redis/6379.conf    
/etc/redis/6380.conf
/etc/redis/6381.conf
// 368行 dir/var/lib/redis/6379 不用换，其他地方的6379都换
b.分别启动三个配置文件
// 文件权限是只读的时候，可以 chmod 777 6380.conf 修改权限
 redis-server /etc/redis/6379.conf 
 redis-server /etc/redis/6380.conf 
 redis-server /etc/redis/6381.conf 
c.分别启动三个客户端
redis-cli -p  6379
redis-cli -p  6380
redis-cli -p  6381
d.在主机和从机上输入info  replication 
e.在从机上面配置相应主机服务器   slaveof 127.0.0.1 6379
f.操作
操作1：主机上写数据，从机上都可以看到，但是从机写数据，会报错，读写分离了；
操作2：主机挂掉(shutdown)，从机不会成为新的主机，当主机进来启动后，从机主动连接到主机。
    
    
~~~

## 哨兵模式

~~~c
Redis Sentinel是一个分布式系统， 你可以在一个架构中运行多个 Sentinel 进程，这些进程使用流言协议来接收关于主服务器是否下线的信息， 并使用投票协议来决定是否执行自动故障迁移，以及选择哪个从服务器作为新的主服务器。
虽然 Redis Sentinel 释出为一个单独的可执行文件 redis-sentinel ， 但实际上它只是一个运行在特殊模式下的 Redis 服务器， 你可以在启动一个普通 Redis 服务器时通过给定 –sentinel 选项来启动 Redis Sentinel 。

哨兵节点的启动有两种方式，二者作用是完全相同的：
redis-sentinel  /etc/redis/sentinel.conf  //一般用这个
redis-server /path/to/sentinel.conf –sentinel
启动 Sentinel 实例必须指定相应的配置文件，系统会使用配置文件来保存 Sentinel 的当前状态， 并在 Sentinel 重启时通过载入配置文件来进行状态还原。如果启动 Sentinel 时没有指定相应的配置文件， 或者指定的配置文件不可写（not writable），那么 Sentinel 会拒绝启动。

~~~

### sentinel.conf

~~~c
#sentinel monitor {masterName}  {masterIp}  {masterPort}  {quorum}
#sentinel monitor  主节点名称    主节点ip    主节点端口    投票数
#quorum是判断主节点客观下线的哨兵数量阈值
#当判定主节点下线的哨兵数量达到quorum时，对主节点进行客观下线
#建议取值为哨兵数量的一半加1
sentinel myid 73093787e1b1290060e1b32155f6f0d2fe913477
# Generated by CONFIG REWRITE
sentinel monitor master6379 127.0.0.1 6379 1 //哨兵个数
~~~

## redis常见问题

~~~c
1.缓存雪崩
大量数据在同一时刻过期，并且很多请求同时产生，这样在缓存里面找不到，请求会到底层数据库上；
解决方案：数据不过期、分散数据过期时间。
2.缓存击穿
某个热点数据过期后，有大量请求不能在缓存中找到，必须到底层数据库进行访问；
解决方案：不设置过期时间，同一时刻只让一个请求访问底层数据库。
3.缓存穿透
访问的数据本来就不存在数据库，有大量请求不能在缓存中找到，但是也不能在底层数据库进行访问。
解决办法：
a.查询时做一些校验和过滤（权限校验，参数校验等等），判断这是一次正常的查询，还是异常的查询或者是攻击，如果是不合法的参数或者查询，直接返回. 
b.缓存空对象，如果数据库中不存在这个数据，我们也在缓存中保存这个key，只是把val值记录为“不存在”，“空”这样的数据，下次再访问这个key时，就不会到数据库中做无用的查找了。
c.我们可以预先将数据库里面所有的key全部存到一个大的map里面,然后在过滤器中过滤掉那些不存在的key.但是需要考虑数据库的key是会更新的,此时需要考虑数据库 --> map的更新频率问题。类似于位图。
~~~

