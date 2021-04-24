# Redis

## 1. 小知识点

- docker进入redis内部

  docker exec -it redis210420 redis-cli

- redis命令不区分大小写，而Key是区分的

- 可以使用命令`help @数据类型` 获取帮助

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

- 最常用： set key1 value1/  get key1
- 批量操作(more): mset k1 v1 k2 v2 [...] /  mget k1 k2 [...]
- 数值增减：
  - ++ : INCR key
  - += : INCRBY key increment
  - -- : DECR key
  - -= : DECRBY key decrement
- 获取字符串长度： STRLEN key
- 分布式锁：
  - setnx key value
  - set key value [EX seconds]\[PX milliseconds] [NX|XX]
  - ttl key 看到期时间
- 应用场景：
  -  商品品号/订单号采用INCR生成
  - like点赞

### 3.2 hash

- 对应java的Map<String, Map<String,String>>
- get/set: 
  - hset key field value
  - hget key field
- mget/mset:
  - hmset key field1 value1  field2 value2 [...]
  - hmget key field1 field2 [...]
- += ： hincrby key field increment
- getall: hgetall key
- length : hlen key
- delete: hdel key field
- 应用场景：购物车
  - 添加商品 : hset shopcart:UUID01 goods1 1
  - 增加数量  : hincrby shopcart:UUID01 goods1 5
  - 商品类型数： hlen shopcart:UUID01
  - 遍历购物车： hgetall shopcart:UUID01

### 3.3 list

- 添加: 
  - 向左： lpush key element1 element2 [...]
  - 向右： rpush key element1 element2 [...]
- 查看： LRange key start end
- 长度： LLen
- 应用： 微信文章订阅公众号，关注的账号发布新文章，就把新文章的id lpush到文章集合中，就可以看到最新的更新文章

### 3.4 set(常用)

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

- 添加： Zadd key score1 member1 score2 member2 [...]
- 排序查询索引间的元素： Zrange key start end [withscores]
- 获取元素分数： zscore key  member
- 删除元素： zrem key member [...]
- 获取指定分数范围：Zrangebyscore key min max [withscore]\[limit offset count]
- 增加元素的分数： Zincrby key incrementScore member
- 长度： zcard key
- 获取指定分数范围的个数： zcount key min max
- 按照排名范围删除： zremrangebyrank key start end
- 应用场景:
  - 微博热搜：score为热度
  - 根据商品销售量对商品排序

## 4. Redis事务（不支持回滚）

- MULTI    ： 开启事务，类似begin
- EXEC： 提交事务，类似Commit
- DISCard: 清除所有先前放入队列的命令
- Watch key： 当一个事务需要按条件执行时，就要用该命令将给定的键设置为受监控状态。（受监控的对象，事务中的队列EXEC不会修改该对象）

## 5.Redis分布式锁

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

