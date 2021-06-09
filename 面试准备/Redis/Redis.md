# Redis

## 1. 小知识点

- 默认端口6379

- docker进入redis内部

  docker exec -it redis210420 redis-cli

- redis命令不区分大小写，而Key是区分的

- 可以使用命令`help @数据类型` 获取帮助

- 使用单线程+多路IO复用

  ![1620781135251](E:\SoftwareNote\面试准备\Redis\Redis的单线程+多路IO复用.png)

## 2. redis常见类型

- 传统

  - string 字符类型
  - list 列表类型
  - hash 散列类型
  - set 集合类型
  - zset(sorted set)  有序集合类型
- 其他
  - bitmap 位图
  - HyperLogLog 统计
  - GEO 地理
  - Stream 

## 3. 各类型命令及应用场景

### 3.1 String

string类型是二进制安全的，可以包含任何数据。一个redis的value最多可以包含512M数据。

扩容：value小于1M时，扩容为*2；value大于1M时，扩容为+1M

- 最常用： set key1 value1/  get key1

- 批量操作(more): mset k1 v1 k2 v2 [...] /  mget k1 k2 [...]

  msetnx 也是批量操作，但是是原子性的，一个存在则全部失败

- 追加： append key value2

- 数值增减：
  - ++ : INCR key
  - += : INCRBY key increment
  - -- : DECR key
  - -= : DECRBY key decrement

- 获取字符串长度： STRLEN key

- 范围操作： setrange start end  / getrange start end    [start, end]

- 设置过期时间： setex key 时间（单位秒）

- 分布式锁：
  - setnx key value  :  **只有key不存在才能设置**
  - set key value [EX seconds]\[PX milliseconds] [NX|XX]
  - ttl key 看到期时间

- 应用场景：
  -  商品品号/订单号采用INCR生成
  - like点赞

### 3.2 hash

- 对应java的Map<String, Map<String,String>>，双层Map，相比直接用JSON存储对象，双层Map更方便更新内部数据。

- 数据结构：

  对应两种：ziplist和hashtable，当field-value长度短且数量少，用ziplist，否则hashtable

- get/set: 
  - hset key field value
  - hget key field
  - hsetnx: 不存在才操作

- mget/mset:
  - hmset key field1 value1  field2 value2 [...]
  - hmget key field1 field2 [...]

- += ： hincrby key field increment

- getall: hgetall key

- length : hlen key

- delete: hdel key field

- contains：hexists key field 查看key中有没有field字段

- keyEntry： hkeys key 列出key中所有filed

- Values: hvals key 列出key中所有values

- 应用场景：购物车
  - 添加商品 : hset shopcart:UUID01 goods1 1
  - 增加数量  : hincrby shopcart:UUID01 goods1 5
  - 商品类型数： hlen shopcart:UUID01
  - 遍历购物车： hgetall shopcart:UUID01

### 3.3 list

list底层结构为快速链表quicklist。在元素较少的情况，会使用一块连续的内存存储，即压缩链表ziplist，它将所有元素挨在一起，分配的是连续的内存空间。当数据量较多，就会改成quicklist（多个ziplist链起来）。

- 添加: 
  - 向左： lpush key element1 element2 [...]
  - 向右： rpush key element1 element2 [...]
- 取值并删除：  lpop / rpop
- 不同列表中替换：rpoplpush key1 key2 从key1右边弹出一个添加到key2的左边
- 查看： LRange key start end
- 通过索引获取值(左到右)：lindex key index
- 插入：linsert key before value newValue 在value前面插入newValue
- 按value删除：lrem key count value , 从左往右删除count个value
  - 替换: lset key index newValue
- 长度： LLen
- 应用： 微信文章订阅公众号，关注的账号发布新文章，就把新文章的id lpush到文章集合中，就可以看到最新的更新文章

### 3.4 set(常用)

可以自动去重，底层是dict字典(用hash表实现的），可以快速查找删除。

- 添加元素： sadd key member1 member2
- 删除元素： srem key member1
- 获取全部： smembers key
- contains： sismember key member1
- 长度： scard key
- 随机弹出count个，不删除： srandmember key [count]
- 随机弹出count个，删除： spop key [count]
- 集合运算
  - 差集A-B： Sdiff A B
  - 交集A∩B： SInter A B
  - 并集A∪B： SUnion A B
- 运用场景
  - 微信抽奖： spop users 3
  - 朋友圈点赞： 点/取消/展示/个数
  - 微博好友社交圈关系
    - 共同关注 SInter A B
    - 我关注的人也关注他
  - QQ内推可能认识的人：A和B是好友，Ａ－B的人可以推荐给B认识

### 3.5 zset

底层数据结构：hash（存储Map格式数据，保证唯一等）+跳跃表（快速查找）

- 添加： zadd key score1 member1 score2 member2 [...]

- 排序查询索引间的元素： zrange key start end [withscores] （withscores展示分数）

- 获取元素分数： zscore key  member

- 删除元素： zrem key member [...]

- 获取指定分数范围：Zrangebyscore key min max [withscore]\[limit offset count]

  zrevrangebyscore 反序

- 增加元素的分数： Zincrby key incrementScore member

- 长度： zcard key

- 获取指定分数范围的个数： zcount key min max

- 按照排名范围删除： zremrangebyrank key start end

- 应用场景:
  - 微博热搜：score为热度
  - 根据商品销售量对商品排序

## 4. Redis事务（不支持回滚）和锁机制

### 4.1 事务的定义

- Redis事务时一个单独的隔离操作：事务中所有的命令都会序列化，按照顺序执行。事务在执行过程中，不会被其他客户端发来的命令打断。
- Redis事务的主要作用是串联多个命令防止插队发生

### 4.2 事务方法

- MULTI： 开启事务，类似begin
- EXEC： 提交事务，类似Commit
- DISCard: 清除所有先前放入队列的命令
- Watch key（乐观锁的体现）： 当一个事务需要按条件执行时，就要用该命令将给定的键设置为受监控状态。（受监控的对象，事务外的命令不会修改该对象）

### 4.3 事务异常处理

- 组队阶段异常，exec全部命令不会提交

- 执行阶段异常，exec**只有异常的命令不会提交**，其余正常的命令还是会正常提交

  ![1620807390485](E:\SoftwareNote\面试准备\Redis\Redis事务.png)

### 4.4 Redis事务三大特性

- 单独的隔离操作：事务中所有的命令都会序列化，按照顺序执行。事务在执行过程中，不会被其他客户端发来的命令打断
- 没有隔离级别概念：队列中的命令没有提交之前，都不会被执行，
- **不保证原子性** ：执行线段可能会出现部分成功部分失败

### 4.5 乐观锁和悲观锁

![1620809650448](E:\SoftwareNote\面试准备\Redis\乐观锁.png)

![1620809636155](E:\SoftwareNote\面试准备\Redis\悲观锁.png)	

### 4.6 乐观锁带来的库存遗留问题

- 版本号不同，则不会执行，因此会有一些数据没被修改

### 4.6 lua脚本解决事务问题

- 将复杂的或者多步骤的redis操作，写成一个脚本，一次提交给redis执行，减少返回连接redis的次数，提升性能。
- lua脚本是类似redis事务，有一定的原子性，不会被其他命令插队，可以完成一些redis事务性的操作。
- redis的lua脚本功能，只有在redis 2.6以上版本才能使用
- 其本质是利用redis的单线程特性，用任务队列的方式解决多任务并发问题。

```lua
local userid=KEYS[1]; 
local prodid=KEYS[2];
local qtkey="sk:"..prodid..":qt";
local usersKey="sk:"..prodid.":usr'; 
local userExists=redis.call("sismember",usersKey,userid);
if tonumber(userExists)==1 then 
  return 2;
end
local num= redis.call("get" ,qtkey);
if tonumber(num)<=0 then 
  return 0; 
else 
  redis.call("decr",qtkey);
  redis.call("sadd",usersKey,userid);
end
return 1;
```

- jedis调用lua

```java
String sha1 = jedis.scriptLoad(String script);
Object result = jedis.evalsha(sha1, 2, userid, prodid)
```

## 5.Redis分布式锁

- 分布式事务问题

  随着业务发展的需要，原单体单机部署的系统被演化成分布式集群系统后，由于分布式系统多线程、多进程并且分布在不同机器上，这将使原单机部署情况下的并发控制锁策略失效，**单纯的Java API并不能提供分布式锁的能力(lock/synchronized只支持单进程，分布式是多进程)**。为了解决这个问题就需要一种跨JVM的互斥机制来控制共享资源的访问，这就是分布式锁要解决的问题！

  分布式锁主流的实现方案：

  \1. 基于数据库实现分布式锁

  \2. 基于缓存（Redis等）

  \3. 基于Zookeeper

  每一种分布式锁解决方案都有各自的优缺点：

  \1. 性能：redis最高

  \2. 可靠性：zookeeper最高

  这里，我们就基于redis实现分布式锁。

- ### 分布式锁需满足四个条件

  首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

  1. **互斥性**。在任意时刻，只有一个客户端能持有锁。
  2. **不会发生死锁**。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
  3. **解铃还须系铃人**。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了，即不能误解锁。
  4. **具有容错性**。只要大多数Redis节点正常运行，客户端就能够获取和释放锁。

- 各分布式事务实现对比

| 分类                              | 方案                                                         | 实现原理                                                     | 优点                                                         | 缺点                                                         |
| --------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 基于数据库                        | 基于mysql 表唯一索引                                         | 1.表增加唯一索引  2.加锁：执行insert语句，若报错，则表明加锁失败  3.解锁：执行delete语句 | 完全利用DB现有能力，实现简单                                 | 1.锁**无超时自动失效机制，有死锁风险**  2.**不支持锁重入，不支持阻塞等待**  3.操作数据库开销大，性能不高 |
| 基于MongoDB findAndModify原子操作 | 1.加锁：执行findAndModify原子命令查找document，若不存在则新增  2.解锁：删除document | 实现也很容易，较基于MySQL唯一索引的方案，性能要好很多        | 1.大部分公司数据库用MySQL，可能缺乏相应的MongoDB运维、开发人员  2.锁无超时自动失效机制 |                                                              |
| 基于分布式协调系统                | 基于ZooKeeper                                                | 1.加锁：在/lock目录下创建临时有序节点，判断创建的节点序号是否最小。若是，则表示获取到锁；否，则则watch /lock目录下序号比自身小的前一个节点  2.解锁：删除节点 | 1.由zk保障系统高可用  2.Curator框架已原生支持系列分布式锁命令，使用简单 | 需**单独维护一套zk集群**，维保成本高                         |
| 基于缓存                          | 基于redis命令                                                | 1. 加锁：执行setnx，若成功再执行expire添加过期时间  2. 解锁：执行delete命令 | 实现简单，相比数据库和分布式系统的实现，该方案最轻，性能最好 | 1.setnx和expire分2步执行，**非原子操作**；若setnx执行成功，但expire执行失败，就**可能出现死锁**  2**.delete命令存在误删除非当前线程持有的锁**的可能  3.**不支持阻塞等待、不可重入** |
| 基于redis Lua脚本能力             | 1. 加锁：执行SET lock_name random_value EX seconds NX 命令   2. 解锁：执行Lua脚本，释放锁时验证random_value   -- ARGV[1]为random_value,  KEYS[1]为lock_name if redis.call("get", KEYS[1]) == ARGV[1] then     return redis.call("del",KEYS[1]) else     return 0 end | 同上；实现逻辑上也更严谨，除了单点问题，生产环境采用用这种方案，问题也不大。 | 不支持锁重入，不支持阻塞等待                                 |                                                              |
| redisson                          | redis+lua基本可应付工作中分布式锁的需求。然而，redission更强 |                                                              | 保持了**简单易用、支持锁重入、支持阻塞等待、Lua脚本原子**操作 |                                                              |

- 普通方案：

```java
@Autowired
private StringRedisTemplate redisTemplate;

private final Lock lock = new ReentrantLock();

private final static String REDIS_LOCK = "REDIS_LOCK";

@Value("${server.port}")
private String serverPort;

@GetMapping("/buy/{id}")
public String buyGoods(@PathVariable Integer id) {

    // 相当于SetNx操作，如果这个key没有被使用则set(有使用说明有锁)，设置默认解锁(超时)时间为10S，避免死锁
    String keyValue = UUID.randomUUID() + Thread.currentThread().getName();
    Boolean flag = redisTemplate.opsForValue().setIfAbsent(REDIS_LOCK,
                                                           keyValue,
                                                           10, TimeUnit.SECONDS);

    if (!flag) {
        return "锁被占用";
    }

    try {
        String key = "Goods:" + id;
        String s = redisTemplate.opsForValue().get(key);
        int count = Integer.parseInt(Optional.ofNullable(s).orElseGet(() -> "0"));
        if (count > 0) {
            return doLog("买到商品"+s+"，库存为：" + redisTemplate.opsForValue().decrement(key));
        } else {
            return doLog("库存不足。");
        }
    } finally {
        // 需要释放锁，
        // 方法1：直接删除锁可能会存在，当本次业务处理超过redis锁的超时时间，那么就可能被其他人解锁
        // redisTemplate.delete(REDIS_LOCK);
        // 方法2： 官方推荐lua脚本，但是面试可能会问非lua的实现。可以采用redis的事务机制保证原子性
        //            if (redisTemplate.opsForValue().get(REDIS_LOCK).equals(keyValue)) {
        //                redisTemplate.watch(REDIS_LOCK);
        //                redisTemplate.setEnableTransactionSupport(true);
        //                redisTemplate.multi();
        //                redisTemplate.delete(REDIS_LOCK);
        //                redisTemplate.exec();
        //                redisTemplate.setEnableTransactionSupport(false);
        //            }

        // 方法3： lua脚本
        Jedis jedis = new Jedis("192.168.1.170", 6379);
        String scipt = "if redis.call('get',KEY[1]) == ARGC[1] " +
            "then  return redis.call('del',KEY[1]) else return 0 end ";
        Object o = jedis.eval(scipt, Collections.singletonList(REDIS_LOCK), Collections.singletonList(keyValue));

        try {
            if (o.equals("1")) {
                doLog("删除Lock成功");
            } else {
                doLog("删除Lock失败");
            }
        } finally {
            // 资源关闭
            if (jedis != null) {
                jedis.close();
            }
        }
    }

}
```



- 最优方案：redission

```java
@Bean
public Redisson redisson() {
    Config config = new Config();
    config.useSingleServer().setAddress("redis://192.168.1.170:6379").setDatabase(0);
    return (Redisson) Redisson.create(config);
}


@Autowired
private StringRedisTemplate redisTemplate;
@Autowired
private Redisson redisson;

private final static String REDIS_LOCK = "REDIS_LOCK";

private final Lock lock = new ReentrantLock();

@Value("${server.port}")
private String serverPort;

@GetMapping("/buy/{id}")
public String buyGoods(@PathVariable Integer id) {
    RLock lock = redisson.getLock(REDIS_LOCK);
    lock.lock();
    try {
        String key = "Goods:" + id;
        String s = redisTemplate.opsForValue().get(key);
        int count = Integer.parseInt(Optional.ofNullable(s).orElseGet(() -> "0"));
        if (count > 0) {
            return doLog("买到商品"+s+"，库存为：" + redisTemplate.opsForValue().decrement(key));
        } else {
            return doLog("库存不足。");
        }
    } finally {
        if (lock.isLocked() && lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

- AOP+Redission+SpEL

```java
class {
    @RedisLock(lockName = "insertUser", key = "#appConnect.appId + ':' + #appConnect.bizUserId")
    @Caching(evict = {
			@CacheEvict(cacheNames = "yami_user", key = "#appConnect.appId + ':' + 
                        #appConnect.bizUserId"),
			@CacheEvict(cacheNames = "AppConnect", key = "#appConnect.appId + ':' + 
                        #appConnect.bizUserId")
	})
    public void insertUserIfNecessary(AppConnect appConnect) {
        
    }
}

@Aspect
@Component
public class RedisLockAspect {

	@Autowired
	private RedissonClient redissonClient;

	private static final String REDISSON_LOCK_PREFIX = "redisson_lock:";

    // 拦截自定义注解@redisLock
	@Around("@annotation(redisLock)")
	public Object around(ProceedingJoinPoint joinPoint, RedisLock redisLock) throws Throwable {
		String spel = redisLock.key();
		String lockName = redisLock.lockName();

		RLock rLock = redissonClient.getLock(getRedisKey(joinPoint,lockName,spel));

		rLock.lock(redisLock.expire(),redisLock.timeUnit());

		Object result = null;
		try {
			//执行方法
			result = joinPoint.proceed();

		} finally {
			rLock.unlock();
		}
		return result;
	}

	/**
	 * 将spel表达式转换为字符串
	 * @param joinPoint 切点
	 * @return redisKey
	 */
	private String getRedisKey(ProceedingJoinPoint joinPoint,String lockName,String spel) {
		Signature signature = joinPoint.getSignature();
		MethodSignature methodSignature = (MethodSignature) signature;
		Method targetMethod = methodSignature.getMethod();
		Object target = joinPoint.getTarget();
		Object[] arguments = joinPoint.getArgs();
		return REDISSON_LOCK_PREFIX + lockName + StrUtil.COLON + SpelUtil.parse(target,spel, targetMethod, arguments);
	}
}
```



- 分布式锁的续期

## 6. 内存 

- Redis内存设置

  - redis.conf 参数 maxmemory <bytes> 不设置和0都是默认最大，32位为3G
  - 生产一般设置为最大内存的3/4
  - 修改可以直接修改配置文件，也可通过命令。`config set maxmemory xx(bytes)`
  - `config get maxmemory` 查看设置的最大内存
  - `info memory ` 查看内存情况

- 内存超出

  `(error) OOM command not allowed when used memory > 'maxmemory'.`

- 内存淘汰策略: 如果一个键是过期的，那么过期之后并不立马从内存中删除, 而是通过缓存过期淘汰策略

  - 缓存过期淘汰策略

    LRU：Least Recently Used （手写算法见算法篇）

    LFU： Least Frequently Used

    -  noeviction(默认): 不会驱逐任何key
    - allkeys-lru: 对所有key使用LRU算法进行删除
    - volatile-lru: 对所有设置了过期时间的key使用LRU算法删除
    - allkeys-random: 对所有key随机删除
    - volatile-random: 对设置了过期时间的key随机删除
    - volatile-ttl: 删除马上要过期的key
    - allkeys-lfu: 对所有key使用LFU算法删除
    - volatile-lfu: 对所有设置了过期时间的key使用LFU算法删除

		-  设置

		- 配置文件： `maxmemory-policy allkeys-lru`

		- 命令： `set maxmemory-policy allkeys-lru`

- 立即删除：能保证内存中数据的最大新鲜度，但频繁删除会占用CPU的时间，产生大量性能损耗，从而也影响到查询操作。

- 惰性删除：数据达到过期时间不删除，而是等下次访问，如果下次访问时已过期则删除，返回空；存在则返回数据。

  对内存不友好，可能会产生一个key早已过期却迟迟没被删除，占用大量内存空间(除非用户手动执行FLUSHDB)。

- 定期删除：是“立即删除”和"惰性删除"的折中。每隔一段时间执行一次删除过期操作，通过设定删除操作执行的时长和频率来减少删除操作对CPU时间的影响。

  但是策略是定期抽样key，会有漏网之鱼。

## 7. NoSql的应用（Not Only Sql 非关系型数据库）

- 不支持ACID（原子性/一致性/隔离性/持久性）
- 远超SQL性能
- 不遵循SQL标准

### 7.1 集群session的共享

### 7.2 数据库缓存

### 7.3 分布式锁



## 8. Redis命令

### 8.1 切换数据库

`select <dbid> 0-15`

### 8.2 查询key的个数

`dbsize`

### 8.2 删除库

- 删除当前库

  `flushdb`

- 删除全部库

  `flushall`

### 8.3 查看key

- 查看当前库所有key

  `keys *`

- 判断key是否存在

  `exists key`

- 查看key类型

  `type key`

- 删除key

  `del key`

- 根据value选择非阻塞删除: 仅将key从keyspace元数据中删除，真正的删除会在后续异步操作中删除

  `unlink key`

- 设置过期时间

  `expire key 10`

- 查看剩余过期时间

  `ttl key`

### 8.4 查看服务器信息

info replication

打印主从复制的相关信息

## 9. docker相关

### 9.1 挂载配置文件启动

- 在外部创建配置文件，并修改配置文件参数

```shell
#注释掉这部分，这是限制redis只能本地访问
bind 127.0.0.1 
#默认yes，开启保护模式，限制为本地访问
protected-mode no 
#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败
daemonize no
#数据库个数（可选），我修改了这个只是查看是否生效。。
databases 16 
#输入本地redis数据库存放文件夹（可选）
dir  ./ 
#redis持久化（可选）
appendonly yes 
```

- docker命令

```shell
docker run -p 6379:6379 --name myredis -v /usr/local/docker/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data --privileged=true -d redis  redis-server /etc/redis/redis.conf --appendonly yes  

docker run 
-p 6379:6379 
--name myredis 
-v /usr/local/docker/redis.conf:/etc/redis/redis.conf -v /usr/local/docker/data:/data #挂载目录
--privileged=true   #权限开关 centos7中安全模块selinux把权限禁掉了
-d redi  redis-server /etc/redis/redis.conf  # -d redis 表示后台启动redis
# redis-server /etc/redis/redis.conf 以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录/usr/local/docker/redis.conf
--appendonly yes   # 开启redis 持久化
```

### 9.2进入容器内部

`docker exec -it redis_outside_config /bin/bash`

### 9.3 docker 主从设置

- 在容器外准备各自的配置文件。修改各自个性化信息。**端口号6379不能改**(或者改了，然后run命令也相应的改？，非docker启动需要设置) 。

- 分别启动docker容器

  ```shell
  docker run -p 6382:6379 --name redis6382 -v /home/ap/docker_config/redis/test/redis6382.conf:/etc/redis/redis.conf  --privileged=true -d redis  redis-server /etc/redis/redis.conf
  ```

- 设置主从

  `slaveof 192.168.1.170 6380` 因127.0.0.1被注释，设置时需要写实际ip

## 10. 配置文件详解

- include ： 可以引入其他配置文件
- network
  - bind 127.0.0.1 表示只能通过本地连接，远程无法连接
  - timeout xx：会话过期时间 ，默认0永不过期
  - tcp-keepalive 300： 检测心跳，300second
- general
  - **daemonize no**#默认no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，**改为yes会使配置文件方式启动redis失败** 

## 11. Redis的发布订阅

## 12. 新数据类型

### 12.1 bitmaps： 一堆二进制值0000000，像biz_character

type bitmaps => string 。说明bitmaps不是数据类型，而是复用string，只是存的都是 01

- 优点：节省空间，就0和1（set占用64位，bitmaps就1位）
- 缺点：不适用非活跃的场景，因为不管活不活跃，总是要用0去填充位置，空间就占用很大了

- 设值：

  `setbit key offset value` offset为位置，value为0/1

- 取值：

  `getbit key offset`

- 计数：

  `bitcount key [start, end]  `

- 交/并/非/异或and / or /not/ xor

  `bitop and(or/xor) newkey key1 key2`

### 12.2 HyperLogLog

- 统计相关的功能，如页面访问量

- 添加

  `pfadd key value  `

- 计数

  `pfcount key`

- 合并

  `pfmerge newkey key1 key2`

### 12.3 GEO

- Geographic地理信息。二位坐标经纬度。支持范围查询/距离查询等

- 添加

  `geoadd key 经 纬 名称`

- 获取

  `geopos key 名称`

- 计算距离

  `geodist key value1 value2 单位(m/km/ft/mi)`  value1到value2的距离

- 找出某半径范围元素

  `georadius key 经 纬 半径 单位`



## 13 SpringBoot集成Redis

### 13.1 pom

```java
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```



### 13.2 configuration

```java
@Configuration
@EnableCaching
public class JedisTemplate extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(factory);

        // 使用Jackson2JsonRedisSerialize 替换默认序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置value的序列化规则和 key的序列化规则
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        // 解决查询缓存转换异常的问题
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);

        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 配置序列化，解决乱码。设置过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(
                        RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```

## 14. Redis的持久化操作

### 14.1 RDB (Redis DataBase)

- 在指定的时间内，将内存中的数据集快照snapshot写入到磁盘。恢复则是相反，将快照写入内存。
- 命令save和bgsave的区别
  - save: 只管保存，其他不管，因此会阻塞。手动保存不建议。
  - bgsave： redis会在后台异步进行快照操作，异步，所有可以异步持久化一边响应其他命令
- 备用是如何执行的

```tex
Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。
```

- fork

  - Fork的作用是**复制一个与当前进程一样的进程**。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个**全新的进程**，并作为原进程的子进程
  - 在Linux程序中，fork()会产生一个和父进程完全相同的子进程，但子进程在此后多会exec系统调用，出于效率考虑，Linux中引入了**“写时复制技术”**
  - 一般情况父进程和子进程会共用同一段物理内存，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程。

- RDB持久化流程

  ![1620820930454](E:\SoftwareNote\面试准备\Redis\RDB持久化流程.png)



- 备份的恢复
  - 关闭Redis
  - 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb
  - 启动Redis, 备份数据会直接加载
- 优势
  - 适合大规模的数据恢复
  - **对数据完整性和一致性要求不高（不敏感的数据）** 更适合使用
  - 节省磁盘空间
  - 恢复速度快
- 劣势
  - Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
  - 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。
  - 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

### 14.2 AOF（Append Only File）

- 是什么

  以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

- RDB和AOF同时开启，默认读AOF的数据

- AOF持久化流程

  （1）客户端的请求写命令会被append追加到AOF缓冲区内；

  （2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；

  （3）AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量；

  （4）Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的；

  ![1620821409867](E:\SoftwareNote\面试准备\Redis\AOF持久化流程.png)

  AOF 的异常恢复

  - 修改默认的appendonly no，改为yes
  - 如遇到**AOF文件损坏**，通过/usr/local/bin/**redis-check-aof--fix appendonly.aof**进行恢复
  - 备份被写坏的AOF文件
  - 恢复：重启redis，然后重新加载

- AOF同步频率设置

  - appendfsync always

    始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

  - appendfsync everysec

    每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

  - appendfsync no

    redis不主动进行同步，把同步时机交给操作系统。

- AOF的重写

  AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof

  如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。 

- 优势

  - **备份机制更稳健** ，丢失数据概率更低。
  - 可读的日志文本，通过操作AOF稳健，**可以处理误操作** 。

- 劣势

  -  比起RDB**占用更多的磁盘空间**（多一个.aof日志）。
  - **恢复备份速度要慢**（多一个.aof日志）。
  - 每次读写都同步的话，有一定的性能压力。
  - 存在个别Bug，造成恢复不能。

- 官方推荐

  官方推荐两个都启用。

  如果对数据不敏感，可以选单独用RDB。

  不建议单独用 AOF，因为可能会出现Bug。

  如果只是做纯内存缓存，可以都不用。 

## 15. Redis主从

### 15.1 主从设置

- 在从服务器上，``slaveof 主host 主port  ``

- 从服务器不允许更新操作

  (error) READONLY You can't write against a read only replica.

### 15.2 查看主从关系

- info replication 

  ```shell
  # Replication
  role:master # 主
  connected_slaves:2 # 2从
  slave0:ip=172.17.0.1,port=6379,state=online,offset=910,lag=0
  slave1:ip=172.17.0.1,port=6379,state=online,offset=896,lag=1
  master_failover_state:no-failover
  master_replid:6c72085743f812574beec8e209d7b23ada3c80a7
  master_replid2:0000000000000000000000000000000000000000
  master_repl_offset:910
  second_repl_offset:-1
  repl_backlog_active:1
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:1
  repl_backlog_histlen:910
  
  # Replication
  role:slave # 从
  master_host:192.168.1.170
  master_port:6380
  master_link_status:up ## up
  master_last_io_seconds_ago:1
  master_sync_in_progress:0
  slave_repl_offset:0
  slave_priority:100
  slave_read_only:1
  connected_slaves:0
  master_failover_state:no-failover
  master_replid:6c72085743f812574beec8e209d7b23ada3c80a7
  master_replid2:0000000000000000000000000000000000000000
  master_repl_offset:0
  second_repl_offset:-1
  repl_backlog_active:1
  repl_backlog_size:1048576
  repl_backlog_first_byte_offset:1
  repl_backlog_histlen:0
  
  ```

### 15.3 down机的效果

- 主机挂掉，重启就行，一切如初

- 从机重启需重设：slaveof 192.167.1.170 6380

  可以将配置增加到文件中。永久生效。 

### 15.4 配置文件

- 从机优先级: 设置从机的优先级，值越小，优先级越高，用于选举主机时使用。默认100

  `slave-priority 10`

### 15.5 常用主从模式

#### 15.5.1 **一主二仆**

#### 15.5.2 薪火相传（主的从是别人的主）

- 主 -> 从 -> 从的从（但从的从不是主的从，因此如果从挂了，主无法复制给从的从）

- 可以有效减轻master的写压力,去中心化降低风险。

- 中途变更转向:会清除之前的数据，重新建立拷贝最新的

  风险是一旦某个slave宕机，后面的slave都没法备份

  主机挂了，从机还是从机，无法写数据了

#### 15.5.3 反客为主

- 当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。
- 手动： 某个从机输入命令`slaveof  no one`  将从机变为主机

###15.6 复制原理

- Slave启动成功连接到master后会发送一个sync命令
-  Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

### 15.7 哨兵模式 sentinel

**反客为主的自动版**，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

- 新建***sentinel.conf***文件（注意拼写）

- 配置文件内容

  `sentinel monitor mymaster 127.0.0.1 6379 1`

  其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。 

- 启动命令

  `执行redis-sentinel  /myredis/sentinel.conf`

- (大概10秒左右可以看到哨兵窗口日志，切换了新的主机)

  哪个从机会被选举为主机呢？根据优先级别：slave-priority 

  原主机重启后会变为从机。

- 复制延迟

  由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。

### 15.8 Java中的主从实现

```java
private static JedisSentinelPool jedisSentinelPool=null;

public static  Jedis getJedisFromSentinel(){
    if(jedisSentinelPool==null){
        Set<String> sentinelSet=new HashSet<>();
        sentinelSet.add("192.168.11.103:26379");

        JedisPoolConfig jedisPoolConfig =new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(10); //最大可用连接数
        jedisPoolConfig.setMaxIdle(5); //最大闲置连接数
        jedisPoolConfig.setMinIdle(5); //最小闲置连接数
        jedisPoolConfig.setBlockWhenExhausted(true); //连接耗尽是否等待
        jedisPoolConfig.setMaxWaitMillis(2000); //等待时间
        jedisPoolConfig.setTestOnBorrow(true); //取连接的时候进行一下测试 ping pong

        jedisSentinelPool=new JedisSentinelPool("mymaster",sentinelSet,jedisPoolConfig);
        return jedisSentinelPool.getResource();
    }else{
   	 	return jedisSentinelPool.getResource();
    }
}
```

## 16. Redis集群

### 16.1 非集群存在的问题

- 容量不够，redis如何进行扩容？
- 并发写操作， redis如何分摊？
- 另外，主从模式，薪火相传模式，主机宕机，导致ip地址发生变化，应用程序中配置需要修改对应的主机地址、端口等信息。
- 之前通过代理主机来解决，但是redis3.0中提供了解决方案。就是无中心化集群配置。

### 16.2 集群的概念 

Redis 集群实现了对Redis的**水平扩容** ，即启动N个redis节点，将整个数据库**分布存储** 在这N个节点中，每个节点存储总数据的1/N。

Redis 集群通过分区（partition）来提供一定程度的可用性（availability）： 即使集群中有一部分节点失效或者无法进行通讯， 集群也可以继续处理命令请求。



### 16.3 集群配置（Docker）

- 集群前

  **redis cluster配置修改**

  cluster-enabled yes    打开集群模式

  cluster-config-file nodes-6379.conf  设定节点配置文件名

  cluster-node-timeout 15000   设定节点失联时间，超过该时间（毫秒），集群自动进行主从切换。

  如果是已存在容器，需要删除旧数据

- 步骤

```shell
# 1. redis.conf中，集群开关开启
cluster-enabled yes

# 2.启动容器，注意暴露+10000端口（如果是已存在容器，需要删除旧数据）
docker run -p 6392:6379 -p 16392:16379 --name redis6392 -v /home/ap/docker_config/redis/test/redis6392.conf:/etc/redis/redis.conf  --privileged=true -d redis  redis-server /etc/redis/redis.conf

# 3.进入容器后，执行命令配置集群（使用真实ip）
# --replicas 1 采用最简单的方式配置集群，一台主机，一台从机，正好三组。
redis-cli --cluster create 192.168.1.170:6380 192.168.1.170:6381 192.168.1.170:6382 192.168.1.170:6390 192.168.1.170:6391 192.168.1.170:6392 --cluster-replicas 1

# 4. 以集群策略启动客户端 -c
docker exec -it redis6391 redis-cli -c

127.0.0.1:6379> get k4 # 会跳转到相应的服务器上 
-> Redirected to slot [8455] located at 172.17.0.3:6379
"1"
```

- 过程错误

```shell
# 配置文件中cluster-enabled yes 的注释打开即可
[ERR] Node 192.168.1.170:6380 is not configured as a cluster node.

# 集群前先删除旧数据
[ERR] Node 192.168.1.170:6380 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.

## 没有暴露相应的+10000端口
Waiting for the cluster to join .... # 配置集群时，一直在run

# 启动集群报错，因为之前已经又配置了，需要删除重新配置（注意删除后要关闭服务器，如果一半开一半关，那么马上又被复制了）
# 解决方法：
# 1)、需要新增的节点下aof、rdb等本地备份文件删除；
# 2)、同时将新Node的集群配置文件删除,即：删除你redis.conf里面cluster-config-file所在的文件；
# 3)、再次添加新节点如果还是报错，则登录新Node,./redis-cli–h x –p对数据库进行清除：
# 172.168.63.201:7001>  flushdb      #清空当前数据库
[ERR] Node 192.168.1.170:6390 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0


 # move集群错误，没有以集群的方式启动 docker exec -it redis6391 redis-cli -c
 127.0.0.1:6379> set k1 v1
(error) MOVED 12706 172.17.0.4:6379
```

### 16.4 集群中的slot

**[OK] All nodes agree about slots configuration.**

**>>> Check for open slots...**

**>>> Check slots coverage...**

**[OK] All** **16384** **slots covered.**

一个 Redis 集群包含 16384 个插槽（hash slot）， 数据库中的每个键都属于这 16384 个插槽的其中一个， 

集群使用公式 CRC16(key) % 16384 来计算键 key 属于哪个槽， 其中 CRC16(key) 语句用于计算键 key 的 CRC16 校验和 。

集群中的每个节点负责处理一部分插槽。 举个例子， 如果一个集群可以有主节点， 其中：

节点 A 负责处理 0 号至 5460 号插槽。

节点 B 负责处理 5461 号至 10922 号插槽。

节点 C 负责处理 10923 号至 16383 号插槽。

- 批量操作 mset: 因为每个key都需要查询slot才能录入，批量的key可能各不相同，就会报错

  在redis-cli每次录入、查询键值，redis都会计算出该key应该送往的插槽，如果不是该客户端对应服务器的插槽，redis会报错，并告知应前往的redis实例地址和端口。

  redis-cli客户端提供了 –c 参数实现自动重定向。

  如 redis-cli  -c –p 6379 登入后，再录入、查询键值对可以自动重定向。

  不在一个slot下的键值，是不能使用mget,mset等多键操作。

  `172.17.0.3:6379> mset k1 v1 k2 v2 k5 v5`
  `(error) CROSSSLOT Keys in request don't hash to the same slot` 

  可以通过{}来定义组的概念，从而使key中{}内相同内容的键值对放到一个slot中去。

  ```shell
  172.17.0.3:6379> mset k1{user} v1 k2{user} v2 k5{user} v5
  OK
  ```

### 16.5 一些集群命令

- 通过 cluster nodes 命令查看集群信息

  ```shell
  # 通过 cluster nodes 命令查看集群信息
  29da3e60f3db4dfe193284c80652c3e819810961 172.17.0.3:6379@16379 myself,master - 0 1620873348000 2 connected 5461-10922
  e66576e12609ca49fd2d0d64888954be199fd095 172.17.0.6:6379@16379 slave 29da3e60f3db4dfe193284c80652c3e819810961 0 1620873346000 2 connected
  02f3bdf883646f331f960ee6955f6f821ad9d37e 172.17.0.7:6379@16379 slave 97339f8690335975a017dbb199db5104c165814e 0 1620873350071 3 connected
  da45ff4c1812960b267153115398c9eb667aeef4 172.17.0.2:6379@16379 master - 0 1620873349067 1 connected 0-5460
  97339f8690335975a017dbb199db5104c165814e 172.17.0.4:6379@16379 master - 0 1620873349000 3 connected 10923-16383
  cc934145920b7cf27310968435f1c8697846e7ef 172.17.0.5:6379@16379 slave da45ff4c1812960b267153115398c9eb667aeef4 0 1620873348060 1 connected
  ```

- 查询集群中key所对应的slot

  ```shell
  172.17.0.3:6379> CLUSTER keyslot user
  (integer) 5474
  ```

- 查询集群中slot中包含的个数

  ```shell
  172.17.0.3:6379> CLUSTER countkeysinslot 5474
  (integer) 3
  ```

- 查询集群中slot中的元素

  ```shell
  172.17.0.3:6379> CLUSTER GETKEYSINSLOT 5474 19
  1) "k1{user}"
  2) "k2{user}"
  3) "k5{user}"
  ```

### 16.6 集群故障恢复

如果主节点下线？从节点能否自动升为主节点？注意：**15秒超时** 

主节点恢复后，主从关系会如何？主节点回来变成从机。 

如果所有某一段插槽的主从节点都宕掉，redis服务是否还能继续?

如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为yes ，那么 ，整个集群都挂掉

如果某一段插槽的主从都挂掉，而cluster-require-full-coverage 为no ，那么，该插槽数据全都不能使用，也无法存储。

redis.conf中的参数  cluster-require-full-coverage

### 16.7 Jedis操作集群

即使连接的不是主机，集群会自动切换主机存储。主机写，从机读。

无中心化主从集群。无论从哪台主机写的数据，其他主机上都能读到数据。

```java
public class JedisClusterTest {
  public static void main(String[] args) { 
     Set<HostAndPort>set =new HashSet<HostAndPort>();
     set.add(new HostAndPort("192.168.31.211",6379));
     JedisCluster jedisCluster=new JedisCluster(set);
     jedisCluster.set("k1", "v1");
     System.out.println(jedisCluster.get("k1"));
  }
}
```

### 16.8 Redis集群提供了以下好处

- 实现扩容
- 分摊压力
- 无中心配置相对简单

### 16.9 Redis 集群的不足

- 多键操作是不被支持的 
- 多键的Redis事务是不被支持的。lua脚本不被支持
- 由于集群方案出现较晚，很多公司已经采用了其他的集群方案，而代理或者客户端分片的方案想要迁移至redis cluster，需要整体迁移而不是逐步过渡，复杂度较大。

## 17 .Redis应用问题

### 17.1 缓存穿透

- 问题描述

  **key对应的数据在数据源并不存在**，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。比如用一个**不存在的用户id获取用户信息**，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

- 解决方案

  一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

  解决方案：

  （1） **对空值缓存：**如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的**过期时间会很短**，最长不超过五分钟

  （2） **设置可访问的名单（白名单）：**

  使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。

  （3） **采用布隆过滤器**：(布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。

  布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)

  将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被 这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。

  （4） **进行实时监控：**当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务 

### 17.2 缓存击穿（单key过期，热点key）

- 问题描述

  key对应的数据存在，但在**redis中过期**，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。 

- 解决方案

  key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

  解决问题：

  **（1）预先设置热门数据：**在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长

  **（2）实时调整：**现场监控哪些数据热门，实时调整key的过期时长

  **（3）使用锁：**

  ​	1） 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。

  ​	2） 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key

  ​	3） 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key；

  ​		当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法。

### 17.3 缓存雪崩（多key过期）

- 问题描述

  key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

  缓存雪崩与缓存击穿的**区别**在于这里**针对很多key缓存**，前者则是某一个key

- 解决方案

  缓存失效时的雪崩效应对底层系统的冲击**非常可怕！**

  解决方案：

  （1） **构建多级缓存架构：**nginx缓存 + redis缓存 +其他缓存（ehcache等）

  （2） **使用锁或队列****：**

  用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况

  （3） **设置过期标志更新缓存：**

  记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。

  （4） **将缓存失效时间分散开(随机)：**

  比如我们可以在原有的失效时间基础上增加一个**随机值**，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

## 18. Redis应用

### 18.1 Key过期监听

可以应用在一些定时通知，比如外卖超过30分钟没有付款自动失效，外卖30分钟没有送达提示用户等等。

如果是常规的思路，会遍历表中的数对比时间，然后挨个通知。但是外卖的数据量是非常大的，存表以及读表效率比较底下。

```java
@Component
@Slf4j
public class RedisKeyExpiredUtils {

    @Bean
    RedisMessageListenerContainer redisMessageListenerContainer(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        // 监听所有库的key过期事件
        container.setConnectionFactory(connectionFactory);
        return container;
    }

    @Bean
    public RedisKeyExpiredImpl redisKeyExpired(RedisMessageListenerContainer redisMessageListenerContainer) {
        return new RedisKeyExpiredImpl(redisMessageListenerContainer);
    }

    class RedisKeyExpiredImpl/* implements RedisKeyExpirationListener  */ extends KeyExpirationEventMessageListener   {

        /**
         *
         * @param listenerContainer must not be {@literal null}.
         */
        public RedisKeyExpiredImpl(RedisMessageListenerContainer listenerContainer) {
            super(listenerContainer);
        }


        @Override
        public void onMessage(Message message, byte[] pattern) {
//            super.onMessage(message, pattern);
            // 监听事件业务逻辑
            log.info("redis key过期监听{},{}" , message.toString(), new String(pattern));
        }
    }
}
```

## 19. SpringBoot整合Redis缓存

- 配置文件：CacheManager / 缓存过期时间 / key策略等

```java
{
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        return new RedisCacheManager(
                RedisCacheWriter.nonLockingRedisCacheWriter(redisConnectionFactory),
                this.getRedisCacheConfigurationWithTtl(30*60), // 默认策略，未配置的 key 会使用这个
                this.getRedisCacheConfigurationMap() // 指定 key 策略
        );
    }

    private Map<String, RedisCacheConfiguration> getRedisCacheConfigurationMap() {
        Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();
        //SsoCache和BasicDataCache进行过期时间配置
        redisCacheConfigurationMap.put("SsoCache", this.getRedisCacheConfigurationWithTtl(24*60*60));
        redisCacheConfigurationMap.put("BasicDataCache", this.getRedisCacheConfigurationWithTtl(30*60));
        return redisCacheConfigurationMap;
    }

    private RedisCacheConfiguration getRedisCacheConfigurationWithTtl(Integer seconds) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        redisCacheConfiguration = redisCacheConfiguration.serializeValuesWith(
                RedisSerializationContext
                        .SerializationPair
                        .fromSerializer(jackson2JsonRedisSerializer)
        ).entryTtl(Duration.ofSeconds(seconds));

        return redisCacheConfiguration;
    }

    @Bean
    public KeyGenerator wiselyKeyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append("." + method.getName());
                if(params==null||params.length==0||params[0]==null){
                    return null;
                }
                String join = String.join("&", Arrays.stream(params).map(Object::toString).collect(Collectors.toList()));
                String format = String.format("%s{%s}", sb.toString(), join);
                //log.info("缓存key：" + format);
                return format;
            }
        };
    }
}


// SPringle注解@CacheConfig/ @Cacheable / @CachePut / @CacheEvit / @Caching
{
    @CacheConfig(cacheNames = "SsoCache")
public class SsoCache{
	@Cacheable(keyGenerator = "wiselyKeyGenerator")
	public String getTokenByGsid(String gsid) 
}
//二者选其一,可以使用value上的信息，来替换类上cacheNames的信息
@Cacheable(value = "BasicDataCache",keyGenerator = "wiselyKeyGenerator")
public String getTokenByGsid(String gsid) {} 

}
```

