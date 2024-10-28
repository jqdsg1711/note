# 黑马点评

redis部署

```
docker run -d --name redis-6381 \
  -p 6379:6379 \
  -v /root/redis/data:/data \
  -v /root/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  redis:6.2.6 \
  redis-server /usr/local/etc/redis/redis.conf


```

```
# Redis服务器配置 
 
# 绑定IP地址
#解除本地限制 注释bind 127.0.0.1  
#bind 127.0.0.1  
 
# 服务器端口号  
port 6379 
 
#配置密码，不要可以删掉
requirepass syf133618
  
 
 
#这个配置不要会和docker -d 命令 冲突
# 服务器运行模式，Redis以守护进程方式运行,默认为no，改为yes意为以守护进程方式启动，可后台运行，除非kill进程，改为yes会使配置文件方式启动redis失败，如果后面redis启动失败，就将这个注释掉
daemonize no
 
#当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定(自定义)
#pidfile /data/dockerData/redis/run/redis6379.pid  
 
#默认为no，redis持久化，可以改为yes
appendonly yes
 
 
#当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能
timeout 60
# 服务器系统默认配置参数影响 Redis 的应用
maxclients 10000
tcp-keepalive 300
 
#指定在多长时间内，有多少次更新操作，就将数据同步到数据文件，可以多个条件配合（分别表示900秒（15分钟）内有1个更改，300秒（5分钟）内有10个更改以及60秒内有10000个更改）
save 900 1
save 300 10
save 60 10000
 
# 按需求调整 Redis 线程数
tcp-backlog 511
 
 
 
  
 
 
# 设置数据库数量，这里设置为16个数据库  
databases 16
 
 
 
# 启用 AOF, AOF常规配置
appendonly yes
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
 
 
# 慢查询阈值
slowlog-log-slower-than 10000
slowlog-max-len 128
 
 
# 是否记录系统日志，默认为yes  
syslog-enabled yes  
 
#指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose
loglevel notice
  
# 日志输出文件，默认为stdout，也可以指定文件路径  
logfile stdout
 
# 日志文件
#logfile /var/log/redis/redis-server.log
 
 
# 系统内存调优参数   
# 按需求设置
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-entries 512
list-max-ziplist-value 64
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
 
 
 
 
```

## 01短信登陆

### 1.基于session实现登陆

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240917211248460.png" alt="image-20240917211248460" style="zoom:50%;" />

存储在session应该存储部分信息而不是全部信息

### 2.session共享问题

描述：多台Tomcat并不共享session存储空间，当请求切换到不同tomcat服务时导致数据丢失的问题。

session的替代方案应该满足：

- 数据共享
- 内存存储
- key、value结构

### 3.基于Redis实现共享session

用redis要考虑两个地方，第一个是用什么来做**key**，另一个是用什么**结构**来做value

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240917213545902.png" alt="image-20240917213545902" style="zoom:50%;" />

为什么用phone来做key，因为在短信验证码登陆时需要手机号，可以二次利用

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240917213429082.png" alt="image-20240917213429082" style="zoom:50%;" />

为什么用token来做key？

第一，安全性，把手机号作为key存储在游览器中不安全，因为游览器需要一个唯一标识来获取用户，如果用手机号不安全

第二，token安全且唯一，

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240917213041416.png" alt="image-20240917213041416" style="zoom: 50%;" />



解决状态登陆刷新问题：

描述，只有用户访问拦截器拦截得页面才会刷新页面

解决思路：新增一个拦截器，拦截一切路径。作用：更新刷新时间





<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240919164100861.png" alt="image-20240919164100861" style="zoom: 80%;" />

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        //登陆拦截器
        registry.addInterceptor(new LoginInterceptor())
                .excludePathPatterns(
                        "/user/code",
                        "/shop/**",
                        "/upload/**",
                        "/voucher/**",
                        "/shop-type/**",
                        "/blog/hot",
                        "/user/login"

                ).order(1);//order代表优先级，数字越小，优先级越高
        //token刷新的拦截器
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate))
                .addPathPatterns("/**").order(0);
    }
}

```

## 02商户缓存

### 什么是缓存？

缓存就是数据交换得缓冲区（Cache),是存储数据得临时地方，一般读写性能较高。

**缓存作用：**降低后端负载；提高读写的效率，降低响应时间。

**缓存成本：**数据一致性成本（数据库里的数据如果发生变化，容易与缓存中的数据形成不一致）。代码维护成本高（搭建集群）。运营成本高。

### 添加redis缓存

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240921205909480.png" alt="image-20240921205909480" style="zoom:50%;" />

在shopserviceImpl修改

```java
@Service
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result queryById(Long id) {
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {
            //3.存在，直接返回
            Shop shop = JSONUtil.toBean(shopJson, Shop.class);
            return Result.ok(shop);
        }
        //4.不存在，根据id查询数据库
        Shop shop = getById(id);
        if (shop == null) {
            //5.不存在，返回错误
            return Result.fail("店铺不存在");
        }
        //6.存在，写入redis
        stringRedisTemplate.opsForValue().set(key,JSONUtil.toJsonStr(shop));
        //7.返回
        return Result.ok(shop);
    }
}
```

### 缓存更新策略

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922104018800.png" alt="image-20240922104018800" style="zoom:50%;" />

#### 主动更新

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922104436179.png" alt="image-20240922104436179" style="zoom:50%;" />

#### 01 Cashe Aside Pattern

操作缓存和数据库时有三个问题需要考虑：

1.删除缓存还是更新缓存

- [ ] 更新缓存：每次更新数据库都更新缓存，无效写操作比较多
- [x] 删除缓存：更新数据库时让缓存失效，查询时再更新缓存

2.如何保证缓存与数据库的操作同时成功与失败

- 单体系统，将缓存与数据库操作放在一个事务
- 分布式系统，利用TCC等分布式事务方案

3.先操作缓存还是先操作数据库

- [ ] 先删除缓存，再操作数据库
- [x] 先操作数据库，再删除缓存

多线程异常情况下：方案二情况概率更低，而且可以给写缓存设置一个超时时间。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922110116641.png" alt="image-20240922110116641" style="zoom:50%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922110134149.png" alt="image-20240922110134149" style="zoom:50%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922110311731.png" alt="image-20240922110311731" style="zoom:50%;" />

```java
    @Override
    @Transactional
    public Result update(Shop shop) {
        Long id = shop.getId();
        if (id == null) {
            return Result.fail("店铺id不能为空");
        }
        //1.更新数据库
        updateById(shop);
        //2.删除缓存
        stringRedisTemplate.delete(CACHE_SHOP_KEY+shop.getId());
        return Result.ok();


    }
}

```

### 缓存穿透

缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。



常见的解决方案有两种：

- ##### **缓存空对象**

  - 优点：实现简单，维护方便
  - 缺点：
    - 额外的内存消耗（解决：设置TTL)
    - 可能造成短期的不一致(解决：数据库更新的时候，主动覆盖缓存)

- ##### **布隆过滤**

  - 优点：内存占用少，没有多余的key

  - 缺点：

    - 实现复杂

    - 存在误判可能

      

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922112747271.png" alt="image-20240922112747271" style="zoom:50%;" />

##### 业务逻辑改变

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922112914771.png" alt="image-20240922112914771" style="zoom: 50%;" />

```java
    @Override
    public Result queryById(Long id) {
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {

            //3.不是空值，返回店铺信息
            Shop shop = JSONUtil.toBean(shopJson, Shop.class);
            return Result.ok(shop);
        }
        //判断命中是否为空值
        if(shopJson != null){
            return Result.fail("店铺不存在");
        }
        //4.不存在，根据id查询数据库
        Shop shop = getById(id);
        if (shop == null) {
            //5.不存在，将空值写入redis
            stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return Result.ok();
        }
        //6.存在，写入redis
        stringRedisTemplate.opsForValue().set(key,JSONUtil.toJsonStr(shop),CACHE_SHOP_TTL, TimeUnit.MINUTES);
        //7.返回
        return Result.ok(shop);
    }

```

##### 总结

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922120853900.png" alt="image-20240922120853900" style="zoom: 50%;" />

### 缓存雪崩

**缓存雪崩**是指同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库，带来巨大压力。

**解决方案：**

- 给不同的key的TTL添加随机值
- 利用Redis集群提高服务的可用性
- 给缓存业务添加降级限流策略
- 给业务添加多级缓存

### 缓存击穿

**缓存击穿问题**也叫热点key问题，就是一个高并发访问并且缓存重建业务较复杂的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922152816615.png" alt="image-20240922152816615" style="zoom: 33%;" />

**解决方案：**

- 互斥锁
- 逻辑过期

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922153642765.png" alt="image-20240922153642765" style="zoom: 33%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922153937561.png" alt="image-20240922153937561" style="zoom: 33%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922154108397.png" alt="image-20240922154108397" style="zoom:33%;" />

##### 互斥解决方式-代码实现

```java
   private boolean tryLock(String  key)
    {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", LOCK_SHOP_TTL, TimeUnit.SECONDS);
        return BooleanUtil.isTrue(flag);
    }

    private void unLock(String  key)
    {
        stringRedisTemplate.delete(key);
    }
   public Shop queryWithMutex(Long id) {
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isNotBlank(shopJson)) {

            //3.不是空值，返回店铺信息
            return JSONUtil.toBean(shopJson, Shop.class);
        }
        //判断命中是否为空值
        if(shopJson != null){
            return null;
        }

        //4.实现缓存重建
        //4.1获取互斥锁
        String lockKey = LOCK_SHOP_KEY + id;
        Shop shop = null;
        try {
            boolean isLokc = tryLock(lockKey);
            //4.2判断是否获取锁
            if(!isLokc){
                //4.3如果失败，则失眠并重试
                Thread.sleep(50);
                return queryWithMutex(id);
            }
            //4.4成功，根据id查询数据库
            shop = getById(id);
            //模拟重建延迟
            Thread.sleep(200);
            if (shop == null) {
                //5.不存在，将空值写入redis
                stringRedisTemplate.opsForValue().set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
                return null;
            }
            //6.存在，写入redis
            stringRedisTemplate.opsForValue().set(key,JSONUtil.toJsonStr(shop),CACHE_SHOP_TTL, TimeUnit.MINUTES);

        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }finally {
            //7.释放互斥锁
            unLock(lockKey);
        }
        //8.返回
        return shop;
    }

```

![image-20240922165202692](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240922165202692.png)

##### 基于逻辑过期-代码实现

```java
  public Shop queryWithLogicalExpire(Long id) {
        String key = CACHE_SHOP_KEY + id;
        //1.从redis查商铺缓存
        String shopJson = stringRedisTemplate.opsForValue().get(key);
        //2.判断是否存在
        if (StrUtil.isBlank(shopJson)) {

            //3.不是空值，返回店铺信息
            return null;
        }
        //命中,需要先把json反序列化
        RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
        JSONObject data = (JSONObject) redisData.getData();
        Shop shop = JSONUtil.toBean(data, Shop.class);
        LocalDateTime expireTime = redisData.getExpireTime();

        //5,判断是否过期
        if (expireTime.isAfter(LocalDateTime.now())) {
            //5.1，未过期，直接返回店铺信息
            return shop;
        }
        //5.2已过期，需要缓存重建

        //6.缓存重建
        //6.1获取互斥锁
        String lockKey = LOCK_SHOP_KEY + id;
        boolean islock = tryLock(lockKey);
        //6.2判断是否获取到锁
        if (islock) {
            // 6.3成功获取到锁，开启独立线程，实现缓存重建
            CACHE_REBUILD_EXECUTOR.submit(() -> {

                try {
                    //重建
                    this.saveShop2Redis(id, 20L);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    //释放锁
                    unLock(lockKey);
                }

            });
        }
        //6.4返回过期商铺信息
        return shop;


    }

```

### 缓存工具封装





## 03优惠券秒杀

#### 全局唯一ID

全局ID生成器，是一种在分布式系统下用来生成全局唯一ID的工具，一般要满足下列特性：

- 唯一性

- 高可用

- 高性能

- 递增性

- 安全性

  ![image-20240923203751341](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240923203751341.png)

##### **code**

```java
    public long nextId(String keyPrefix) {
        //1.生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSeconds = now.toEpochSecond(ZoneOffset.UTC);
        long timestamp = nowSeconds - BEGIN_TIMESTAMP;


        //2.生成序列号
        //2.1获取当前的日期，精确到天
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        //2.2自增长
        long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);
        //3.拼接并返回

        return timestamp << COUNT_BITS | count ;
    }
```

##### 总结

全局唯一ID生成策略：

- UUID
- Redis自增
- snowflake算法
- 数据库自增

Redis自增ID策略：

- 每天一个key,方便统计订单量
- ID构造是 时间戳+计数器





### 实现优惠券秒杀下单

下单时需要判断两点：

- 秒杀是否开始或结束，如果尚未开始或已经结束则无法下单
- 库存是否充足，不足则无法下单

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240924225330605.png" alt="image-20240924225330605" style="zoom: 50%;" />





### 超卖问题

**问题描述**：

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240926102347632.png" alt="image-20240926102347632" style="zoom:33%;" />

超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案是加锁。

- **悲观锁**：认为线程安全问题一定会发送，因此在操作数据之前要先获取锁，确保线程串行执行。像Synchronized、Lock都属于悲观锁。
- **乐观锁**：认为线程安全问题不一定会发生，因此不加锁，只是在更新数据时去判断有没有其它线程对数据做了修改。
  - 如果没有修改则认为是安全的，自己才更新数据。
  - 如果已经被其它线程修改说明发生了安全问题，此时可以重试或异常。

**乐观锁关键是判断之前查询得到的数据是否被修改过，常见的方法有2种：**

1. 版本号法：添加一个版本号，在数据库操作时，要对版本号进行核对，一致才能执行sql语句

![image-20240926104536160](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240926104536160.png)

2.CAS法（版本号法的简化版）：查询的时候把库存查出来，更新的时候判断库存和之前查到的库存是否一致，如果一致则更新数据。

![image-20240926104824652](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240926104824652.png)



code

```java
 //5.扣减库存
        boolean success = seckillVoucherService.update()
                .setSql("stock = stock - 1" )
                .eq("voucher_id", voucherId).eq("stock",voucher.getStock()).update();

```



测试结果发现：200个并发只成功21个，**成功率太低**

原理是因为：假如一次有30个线程涌入，查询到库存值为100，只有1个线程能把值改为99，其它29个线程比对库存值99发现和自己查询到的库存值100不同，所以都认为数据已经被修改过，所以都失败了。

改进：

```java
boolean success = seckillVoucherService.update()
        .setSql("stock = stock - 1") //相当于set条件 set stock = stock - 1
        .eq("voucher_id", voucherId) //相当于where条件 where id = ? and stock = ?
        .gt("stock",0).update();
```

**乐观锁缺陷：**需要大量对数据库进行访问，容易导致数据库的崩溃。

<img src="https://i-blog.csdnimg.cn/blog_migrate/6a5877b6520a15c240ddbcdf47090c56.png" alt="img" style="zoom: 67%;" />



### 一人一单

需要大量对数据库进行访问，容易导致数据库的崩溃。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240926110153318.png" alt="image-20240926110153318" style="zoom:50%;" />







![image-20240929123313963](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240929123313963.png)

### 分布式锁

**分布式锁**：满足分布式系统或集群模式下多进程可见并且互斥的锁。

synchronized只能保证单个JVM内部的多个线程之间的互斥，而没法让集群下多个JVM进程间的线程互斥。

 要让多个JVM进程能看到同一个锁监视器，而且同一时间只有一个线程能拿到锁监视器。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240929124151187.png" alt="image-20240929124151187" style="zoom:50%;" />

**分布式**要满足

1. 多进程可见
2. 互斥
3. 高可用
4. 高新能
5. 安全性
6. ....（功能性特性）



<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20240929124952149.png" alt="image-20240929124952149" style="zoom:33%;" />

#### 基于Redis的分布式锁

![image-20241002103314269](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241002103314269.png)



**code**

```java
public class SimpleRedisLock implements ILock{

    private String name;

    private StringRedisTemplate stringRedisTemplate;

    private static final String KEY_PREFIX = "lock:";

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean tryLock(long timeoutSec) {
        //获取线程标识
        long threadId = Thread.currentThread().getId();
        //获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId + "", timeoutSec, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }

    @Override
    public void unlock() {
        //释放锁
        stringRedisTemplate.delete(KEY_PREFIX + name);
    }
}

```



```java
    public Result seckillVoucher(Long voucherId) {
        //1.查询优惠券信息
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
        //2.判断秒杀是否开始
        //2.1秒杀尚未开始返回异常
        if(voucher.getBeginTime().isAfter(LocalDateTime.now())){
            return Result.fail("秒杀尚未开始");
        }
        //2.2秒杀已结束返回异常
        if(voucher.getEndTime().isBefore(LocalDateTime.now())){
            return Result.fail("秒杀已经结束");
        }
        voucher = seckillVoucherService.getById(voucherId);
        //3.判断库存是否充足
        if(voucher.getStock()<1){
            //3.1库存不足返回异常
            return Result.fail("库存不足！");
        }
        Long userId = UserHolder.getUser().getId();
        //创建锁对
        SimpleRedisLock lock = new SimpleRedisLock("order" + userId, stringRedisTemplate);
        //获取锁
        boolean isLock = lock.tryLock(1200);
        if(!isLock){
            //获取锁失败,返回错误或失败
            return Result.fail("不允许重复下单");


        }try{
        //获取代理对象(事务）
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
            return proxy.createVoucherOrder(voucherId);
        }finally {
            //释放锁
            lock.unlock();
        }
    }
```

**安全问题**

由于业务阻塞，可能线程1被超时释放锁，然后线程2乘虚而入，进行自己的业务。由于线程1业务完成，就会释放锁，此时释放的锁是线程2的锁。而后线程3可以获取锁并且执行业务导致安全问题。



<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241002110038974.png" alt="image-20241002110038974" style="zoom: 50%;" />

**问题关键：**释放锁时要确定是否是自己锁，需要获取锁标识，并判断是否一致

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241002110157205.png" alt="image-20241002110157205" style="zoom:50%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241002110228801.png" alt="image-20241002110228801" style="zoom:50%;" />

##### 案例：改进Redis的分布式锁

需求：修改之前的分布式锁实现，满足：

1. 在获取锁时存入线程表示（可以用UUID表示）(在集群模式下可能线程id会重复)
2. 在释放锁时先获取锁中的线程标示，判断是否与当前标示一致
   - 如果一致则释放锁
   - 如果不一致则不释放锁

**code**

```java
   @Override
    public boolean tryLock(long timeoutSec) {
        //获取线程标识
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        //获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        return Boolean.TRUE.equals(success);
    }

    @Override
    public void unlock() {
        //获取线程标示
        String threadId = ID_PREFIX + Thread.currentThread().getId();
        //获取锁中的标示
        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
        //判断是否一致
        if (threadId.equals(id)) {
            //释放锁
            stringRedisTemplate.delete(KEY_PREFIX + name);
        }
    }
```



判断与释放锁可能产生阻塞（JVM的垃圾回收），就会产生超时释放锁

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003102822732.png" alt="image-20241003102822732" style="zoom:33%;" />



**解决：**必须让判断锁和释放锁有原子性。

##### Redis的lua脚本

Redis提供Lua脚本功能，在一个脚本中编写多条Redis命令，**确保多条命令执行时的原子性**。

Lua是一种编程语言，它的基本语法可以参考网站：https://www.runoob.com/lua/lua-tutorial.html

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003103336480.png" alt="image-20241003103336480" style="zoom:33%;" />



**执行脚本**



繁琐的lua脚本

```lua
-- 锁的key
local key = KEYS[1]
-- 当前线程标示
local threadId = ARGV[1]
 
--获取锁中的线程标示
local id = redis.call('get',key)
--比较线程标示与锁中的标示是否一致
if(id == threadId) then
    --释放锁 del key
    return redis.call('del',key)
end
return 0
```

简化版lua脚本

```lua
--比较线程标示与锁中的标示是否一致
if(redis.call('get',KEYS[1]) == ARGV[1]) then
    --释放锁 del key
    return redis.call('del',KEYS[1])
end
return 0
```

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003104612285.png" alt="image-20241003104612285" style="zoom:33%;" />

###### **code**

```java
    private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;
    static {
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }
   @Override
    public void unlock() {
     //调用lua脚本
       stringRedisTemplate.execute(
                UNLOCK_SCRIPT,
                Collections.singletonList(KEY_PREFIX + name),
                ID_PREFIX + Thread.currentThread().getId()
        );
    }
```

##### 总结

**基于Redis的分布式锁的实现思路：**

1.利用set nx ex获取锁，并设置过期时间，保存线程标示。

2.释放锁时先判断线程标示是否与自己一致，一致则删除锁。

**特性：**

1.利用set nx满足互斥性。

2.利用set nx保障故障时锁依然能够释放，避免死锁，提高安全性。

3.利用Redis集群保障高可用和高并发的特性。

#### 基于Redis的分布式锁优化

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003110632157.png" alt="image-20241003110632157" style="zoom: 33%;" />

##### **Redisson**

Redisson是一个在Redis的基础上实现的Java驻内存数据网格。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务，其中包含了各种分布式锁的实现。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003111205323.png" alt="image-20241003111205323" style="zoom:25%;" />

官网地址：https://redisson.org

GitHub：https://github.com/redisson/redisson

##### Redisson入门

1.引入依赖

```java
<!--redisson-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.13.6</version>
</dependency>
```

2.配置Redisson客户端：在config包下创建RedissonConfig类

```java
@Configuration
public class RedissonConfig{
    @Bean
    public RedissonClient redissonClient(){
        //配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379").setPassword("");
        //创建RedissonClient对象
        return Redisson.create(config);
    }
}
```

3.使用Redission的分布式锁

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241003111935178.png" alt="image-20241003111935178" style="zoom:33%;" />

##### Redisson可重入锁原理

获取锁的时候在判断这个锁已经被占有的情况下，会检查占有锁的是否是当前线程，如果是当前线程，也会获取锁成功。会有一个计数器记录重入的次数。

value的结构string就不能满足需求，故使用hash结构。

![image-20241004101247613](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004101247613.png)

每释放一次锁采用的策略是把重入次数减1。

加锁和释放锁是成对出现的，因此当方法执行到最外层结束时，重入的次数一定会减为0。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004101811497.png" alt="image-20241004101811497" style="zoom: 50%;" /><img 

![image-20241004102107138](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004102107138.png)

源码：

加锁逻辑

<img src="https://i-blog.csdnimg.cn/blog_migrate/3deab72cd73f80454e3a4b94322eead0.png" alt="img" style="zoom: 50%;" />

解锁逻辑

<img src="https://i-blog.csdnimg.cn/blog_migrate/9fda7c2ec1270b8490e5db5dd73d41e3.png" alt="img" style="zoom:50%;" />

![image-20241004104538183](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004104538183.png)

获取锁机制：

1.判断ttl是否为null

        1.1 为null，获取锁成功（涉及自动更新锁过期时间），判断leaseTime是否为-1
    
                1.1.1 为-1自动开启看门狗机制，定时更新锁的过期时间
    
                        看门狗默认30秒，每隔10秒会更新一次过期时间。
    
                1.1.2 不为-1返回true
    
        1.2 不为null，获取锁失败（涉及获取锁的失败重试），判断剩余等待时间是否大于0
    
                1.2.1 大于0，订阅并等待释放锁的信号
    
                        在受到释放信号后会判断是否超时，如未超时继续尝试获取锁
    
                1.2.2 不大于0，获取锁失败

释放锁机制：

1.尝试释放锁，判断是否成功

        1.1 释放成功。
    
                发送锁释放的消息（与获取锁的失败重试关联）
    
                取消看门狗机制（与自动更新锁过期时间关联）
    
        1.2 释放失败。返回异常。


##### 总结

1. 可重入：利用hash结构纪录线程id和可重入次数
2. 可重试：利用信号量和PubSub功能实现等待、唤醒、获取锁失败的重试机制
3. 超时续约：利用watchDog，每隔一段时间（releaseTime/3)，重置超时时间

#### 主从一致性问题

主节点负责写，从节点负责读，主节点和从节点间需要同步，会存在延迟。

如果主节点宕机，会从从节点中选拔一个新的节点作为主节点。如果主从同步尚未完成，会出现锁失效的问题。

使用联锁来满足需求，现在在所有主节点中都存放一份锁，要求一个线程必须从所有主节点中获取锁，才算真正获取锁。

![image-20241004131200917](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004131200917.png)



##### 总结

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004131452997.png" alt="image-20241004131452997" style="zoom:33%;" />

### Redis优化秒杀

库存：KEY用string类型，VALUE用数值类型。

一人一单：KEY用string类型，VALUE用set集合类型。（set中存放的用户id，故具有唯一性，使用set集合）

由于整个操作流程长，且具有原子性，故使用lua脚本。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241004162230003.png" alt="image-20241004162230003" style="zoom: 33%;" />

##### 案例：改进秒杀业务，提交并发性能

需求：

1. 新增秒杀优惠券的同时，将优惠券信息保存在Redis中
2. 基于lua脚本，判断秒杀库存、一人一单，决定用户是否抢购成功
3. 如果抢购成功，将优惠券id和用户id封装后存入阻塞队列、
4. 开启线程，不断从阻塞队列中获取信息，实现异步下单功能

##### 总结

秒杀业务的优化思路：

1.先利用Redis完成库存量、一人一单的判断，完成抢单业务。

2.将下单业务放入阻塞队列，利用独立线程异步下单。

基于阻塞队列的异步秒杀存在哪些问题：

1.内存限制问题。使用的是jdk提供的阻塞队列，使用的是JVM的内存，在一开始写死了队列空间的大小，如果在高并发的情况下，队列很快会被占满，如果不对队列的空间加以限制，很容易造成内存的溢出。

2.数据安全问题。缺乏持久化机制，是基于内存来保存信息，如果服务突然宕机，内存中保存的信息都会丢失。如果任务被取出，但由于突然发生事故异常，导致任务没有被消费，任务丢失，会造成数据不一致问题。

### Redis消息队列实现异步秒杀

消息队列（message Queue),字面意思就是存放消息的队列。最简单的消息队列模型包括3个角色：

- **消息队列**：存储和管理消息，也被称为消息代理（Message Broker)
- **生产者**：发送消息到消息队列
- **消费者**：从消息队列获取消息并处理消息

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005122536100.png" alt="image-20241005122536100" style="zoom:50%;" />



Redis提供了3种不同的方式来实现消息队列：

1.**list结构**：基于List结构模拟消息队列。

2.**PubSub**（发布订阅）：基本的点对点消息模型。

3.**Stream**：比较完善的功能强大的消息队列模型。

##### 基于List结构模拟消息队列

**消息队列（message Queue)**,字面意思就是存放消息的队列。Redis的list数据结构是一个双向链表，很容易模拟出队列效果。

队列的入口和出口不在一边，可以利用：LPUSH结合RPOP，RPUSH结合LPOP来实现。

如果队列中没有消息时RPOP或LPOP的操作会返回null，不会像JVM的阻塞队列那样阻塞并等待消息，因此这里应该用**BRPOP**或**BLPOP**来实现阻塞效果。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005122905961.png" alt="image-20241005122905961" style="zoom:50%;" />

List消息队列

**优点：**

1.利用Redis存储，不受限于JVM内存上限。

2.基于Redis的持久化机制，数据安全性有保证。

3.可以保证消息的有序性。

**缺点：**

1.无法避免消息丢失。

2.只支持单消费者。

##### 基于PubSub的消息队列

**PubSub**（Publish Subscribe **发布订阅**）：Redis2.0版本引入的消息传递模型，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。

- SUBSCRIBE channel[channel]  ：  订阅一个或多个频道。
- PUBLISH channel msg  ：  向一个频道发送消息。
- PSUBSCRIBE pattern[pattern]  ：  订阅与pattern格式匹配的所有频道。

基于PubSub的消息队列有哪些优缺点：

**优点：**

1.采用发布订阅模型，支持多生产、多消费。

**缺点：**

1.不支持数据持久化

2.无法避免消息丢失（如果发布的消息没有人订阅，消息直接丢失）安全性无法保障

3.消息堆积有上限（消费者缓存的空间有上限），超出时数据丢失。

##### 基于Stream的消息队列

Stream是Redis5.0引入的一种新数据类型，可以实现一个功能非常完善的消息队列。

发送消息的命令：

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005124754048.png" alt="image-20241005124754048" style="zoom:50%;" />

例如：

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005124808803.png" alt="image-20241005124808803" style="zoom:50%;" />

读取消息的方式之一：XREAD

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005125041526.png" alt="image-20241005125041526" style="zoom:50%;" />

例如，使用XREAD读取第一个消息：

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005125254857.png" alt="image-20241005125254857" style="zoom:33%;" />

XREAD阻塞方式，读取最新的消息：

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005125339068.png" alt="image-20241005125339068" style="zoom: 50%;" />

在业务开发中，我们可以循环的调用XREAD阻塞方式来查询最新消息，来实现持续监听的效果，伪代码如下。

![image-20241005125409029](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005125409029.png)

STREAM类型消息队列的XREAD命令特点：

- 消息可回溯。
- 一个消息可以被多个消费者读取。
- 可以阻塞读取。
- 有消息漏读风险。

##### 基于stream的消息队列-消费组

**消费组（Consumer Group)**：将多个消费者划分到一个组中，监听同一个队列。具备下列特点：

**![image-20241005151008329](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005151008329.png)**



<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005153035084.png" alt="image-20241005153035084" style="zoom:33%;" />

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005153054785.png" alt="image-20241005153054785" style="zoom:33%;" />

消费者监听消息的基本思路：

![image-20241005153200331](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005153200331.png)











###### 总结

STREAM类型消息队列的XREADGROUP命令特点：

1.消息可回溯。

2.可以多消费者争抢消息，加快消费速度。

3.可以阻塞读取。

4.没有消息漏读的风险。

5.有消息确认机制，消息至少被消费一次。

6.支持消息持久化

##### 总结

![image-20241005152731866](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241005152731866.png)

##### 基于Redis的stream结构作为消息队列，实现异步秒杀下单

需求：

1. 创建一个Stream类型的消息队列，名为stream.order
2. 修改之前的秒杀下单Lua脚本，在认定有抢购资格后，直接向stream.order中添加消息，内容包含voucherId，userId,orderId
3. 项目启动时，开启一个线程任务，尝试获取stream.orders中的消息，完成下单

实现：

1.

```
redis:db0> XGROUP CREATE stream.orders g1 0 MKSTREAM
OK
```









## 04达人探店

#### 发布探店笔记

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241006111632113.png" alt="image-20241006111632113" style="zoom:33%;" />















































#### 点赞

##### 完善点赞功能

需求

同一个用户只能点赞一个，再次点击则取消点赞

如果当前用户已经点赞，则点赞按钮高亮显示（前段已实现，判断字段Blog类的isLike属性）



实现步骤：

1. 给Blog类中添加一个isLike字段，标示是否被当前用户点赞
2. 修改点赞功能，利用redis的set集合判断是否点赞过，未点赞过则点赞数+1，已点赞过则点赞过-1
3. 修改根据id查询Blog的业务，判断当前登录用户是否点赞过，赋值给isLike字段
4. 修改分页查询Blog业务，判断当前登录用户是否点赞过，赋值给isLike字段

#### 点赞排行榜

在探店笔记的详情页面，应该把给该笔记点赞的人显示出来，比如最早的点赞

需求：按照点赞时间先后排序，返回Top5的用户

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007142851717.png" alt="image-20241007142851717" style="zoom: 50%;" />

## 05好友关注

### 关注和取关

**<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007145755520.png" alt="image-20241007145755520" style="zoom:50%;" />**



<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007153850791.png" alt="image-20241007153850791" style="zoom:50%;" />

```java
@Service
public class FollowServiceImpl extends ServiceImpl<FollowMapper, Follow> implements IFollowService {

    @Override
    public Result follow(Long followUserId, Boolean isFollow) {
        Long userId = UserHolder.getUser().getId();
        //1.判断是关注还是取关
        if(isFollow){
            //2.关注，新增数据
            Follow follow = new Follow();
            follow.setUserId(userId);
            follow.setFollowUserId(followUserId);
            save(follow);
        }else{
            //3.取关，删除记录
            remove(new QueryWrapper<Follow>()
                    .eq("user_id", userId).eq("follow_user_id", followUserId));
        }
        return Result.ok();
    }

    @Override
    public Result isFollow(Long followUserId) {
        Long userId = UserHolder.getUser().getId();
        //1.查询是否关注
        Long count = query().eq("user_id", userId).eq("follow_user_id", followUserId).count();
        //2.判断是否关注
        return Result.ok(count>0);
    }
}
```



### 共同关注





### 关注推送

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007160734145.png" alt="image-20241007160734145" style="zoom: 33%;" />

Feed流产品有2种常见模式：

1.**Timeline**：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈。

优点：信息全面，不会有缺失。并且实现也相对简单。

缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低。

2.**智能排序**：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣的信息来吸引用户。

优点：投喂用户感兴趣的信息，用户粘度高，容易沉迷。

缺点：如果算法不精确，可能起反作用。

本例中的个人页面，是基于关注的好友来做feed流，因此采用Timeline的模式。该模式的实现方案有三种：

1. 拉模式
2. 推模式
3. 推拉结合



**拉模式**：也叫读扩散

每个人发消息都会带上时间戳到发件箱，读取的人有一个收件箱，读取的时候会根据时间戳来把所有的新的消息拉取。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007161412280.png" alt="image-20241007161412280" style="zoom:50%;" />

缺点：操作耗费时间多，存在延迟。

**推模式**：写扩散。

信息的发送端会直接把信息发送到所有接收方的收件箱。接收方收件箱内的消息按时间逐个进行排序

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007161522763.png" alt="image-20241007161522763" style="zoom:50%;" />

缺点：需要保存大量的消息。

**推拉结合模式**：读写混合，兼具推和拉两种模式的优点。

当普通人发消息，因为粉丝少，直接采用推模式。

当大V发消息，因为粉丝多，对于活跃粉丝（数量少）采用推模式，对于普通粉丝僵死粉（数量多）采用拉模式。



<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007161902999.png" alt="image-20241007161902999" style="zoom:50%;" />



![image-20241007161917386](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007161917386.png)

### 基于推模式实现关注推送功能

![image-20241007162324873](C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007162324873.png)

Feed流中的数据会不断更新，所以数据的角标也在变化，因此不能采用传统的分页模式。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007162434573.png" alt="image-20241007162434573" style="zoom:33%;" />

滚动分页模式：纪录每次最后一个查询的id。

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007162546644.png" alt="image-20241007162546644" style="zoom: 33%;" />

List不支持滚动分页，score范围进行查询，记住最小的时间戳，下次找比这个更小的时间戳。

```java
    @Override
    public Result saveBlog(Blog blog) {
        //1.获取登录用户
        UserDTO user = UserHolder.getUser();
        blog.setUserId(user.getId());
        //2.保存探店笔记
        boolean isSuccess = save(blog);
        if(!isSuccess){
            return Result.fail("新增笔记失败！");
        }
        //3.查询笔记作业的所有粉丝
        //select * from tb_follow where follow_user_id = ?
        List<Follow> follows = followService.query().eq("follow_user_id", user.getId()).list();
        //4.推送笔记id给粉丝
        for(Follow follow : follows){
            //4.1.获取粉丝id
            Long userId = follow.getUserId();
            //4.2.推送到粉丝收件箱是sortedSet
            String key = "feed::"+userId;
            stringRedisTemplate.opsForZSet().add(key,blog.getId().toString(),System.currentTimeMillis());
        }
        //返回id
        return Result.ok(blog.getId());

    }
```

### 实现关注推送页面的分页查询

<img src="C:\Users\jqdsg\AppData\Roaming\Typora\typora-user-images\image-20241007163324195.png" alt="image-20241007163324195" style="zoom:50%;" />

ZRANGE是按照角标从小到大排序：
