# springboot2.X整合redis分布式

背景：因项目生成单号，分布式锁，限流

------

pom依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
            <version>${spring-boot-starter-redis-version}</version>
        </dependency>
```

### <u>1.实现分布式环境生成单号,实现代码如下</u>

```java
   @Autowired
   private StringRedisTemplate redisTemplate;

   @GetMapping("/api/getBillNo")
   public String billno() {
        StringBuffer prefix = new StringBuffer("ZB-");
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMdd");
        String day = sdf.format(new Date());
        String key = prefix.append(day).toString();
        RedisAtomicLong ra = new 	                    RedisAtomicLong(key,redisTemplate.getConnectionFactory());
        Long inc = ra.incrementAndGet();
        // 设置过期时间
        redisTemplate.expire(key,24l, TimeUnit.HOURS);
        return prefix.append(String.format("%05d",inc)).toString();
    }
```

### <u>2.实现分布式锁</u>

[分布式锁参考]: https://blog.csdn.net/new_com/article/details/104045501
[redis分布式锁实现参考]: https://blog.csdn.net/qq_28397259/article/details/80839072?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&amp;depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase

> #### 2.1 简单实现分布式锁(非原子性，非集群操作)

```java
    static final String DELIMITER = "|";
    @Autowired
    private StringRedisTemplate redisTemplate;

 /**
     * 获取锁
     * @param lockKey lockKey
     * @return true or false
     */
    @GetMapping("/api/lock")
    public boolean lock(String lockKey) {
        // uuid类似业务中数据值唯一
        String uuid = String.valueOf(UUID.randomUUID());
        long timeout = 90;
        final long milliseconds = Expiration.from(timeout, TimeUnit.SECONDS).getExpirationTimeInMilliseconds();
        boolean success = redisTemplate.opsForValue().setIfAbsent(lockKey, (System.currentTimeMillis() + milliseconds) + DELIMITER + uuid);
        if (success) {//非原子性可能导致key不过期
            redisTemplate.expire(lockKey, timeout, TimeUnit.SECONDS);
        } else {
            String oldVal = redisTemplate.opsForValue().getAndSet(lockKey, (System.currentTimeMillis() + milliseconds) + DELIMITER + uuid);
            final String[] oldValues = oldVal.split(Pattern.quote(DELIMITER));
            if (Long.parseLong(oldValues[0]) + 1 <= System.currentTimeMillis()) {
                return true;
            }
        }
        return success;
    }

    /**
     * 释放锁
     * @param lockKey lockKey
     * @return true or false
     */
    @GetMapping("/api/unlock")
    public boolean unlock(String lockKey , final String uuid) {
        String val = redisTemplate.opsForValue().get(lockKey);
        if(val.isEmpty()){
            return false;
        }
        final String[] values = val.split(Pattern.quote(DELIMITER));
        if (values.length <= 0) {
            return false;
        }
        if (uuid.equals(values[1])) {
            redisTemplate.delete(lockKey);
        }
        return true;
    }

```

> #### 2.2 实现分布式锁(原子性，redis集群操作)

```java
//使用lua+redis 实现原子性操作
 private static final Long SUCCESS = 1L;
    public static final String UNLOCK_LUA;
    public static final String LOCK_LUA;
    static {
        StringBuffer sb = new StringBuffer();
        sb.append("if redis.call(\"get\",KEYS[1]) == ARGV[1] ")
            .append("then  return redis.call(\"del\",KEYS[1]) ")
            .append("else  return 0 end ");
        UNLOCK_LUA = sb.toString();
        sb.setLength(0);
        sb.append("if redis.call(\"setNx\",KEYS[1],ARGV[1]) == 1 ")
                .append("then if redis.call(\"get\",KEYS[1])==ARGV[1] ")
                .append("then return redis.call('expire',KEYS[1],ARGV[2]) ")
                .append("else return 0 end end");
        LOCK_LUA = sb.toString();
    }

/**
     * 获取锁
     * @param lockKey lockKey
     * @return true or false
     */
    @GetMapping("/api/atomiclock")
    public boolean atomiclock(String lockKey) {
        String value = String.valueOf(UUID.randomUUID());
        long expireTime = 90;
        List<String> args = Arrays.asList(value,String.valueOf(expireTime));
        Long result = (Long)redisTemplate.execute((RedisCallback<Long>) connection -> {
            Object nativeConnection = connection.getNativeConnection();
            if (nativeConnection instanceof JedisCluster) {// 集群
                return (Long) ((JedisCluster) nativeConnection).eval(LOCK_LUA, Collections.singletonList(lockKey), args);
            } else if (nativeConnection instanceof Jedis) {// 非集群
                return (Long) ((Jedis) nativeConnection).eval(LOCK_LUA, Collections.singletonList(lockKey), args);
            }
            return null;
        });
        if(SUCCESS.equals(result)){
            return true;
        }else{
            return false;
        }
    }

    /**
     * 释放锁
     * @param lockKey lockKey
     * @return true or false
     */
    @GetMapping("/api/atomicunlock")
    public boolean atomicunlock(String lockKey , final String uuid) {
        List<String> args = Arrays.asList(uuid);
        Long result = (Long)redisTemplate.execute((RedisCallback<Long>) connection -> {
            Object nativeConnection = connection.getNativeConnection();
            if (nativeConnection instanceof JedisCluster) {// 集群
                return (Long) ((JedisCluster) nativeConnection).eval(UNLOCK_LUA, Collections.singletonList(lockKey), args);
            } else if (nativeConnection instanceof Jedis) {// 非集群
                return (Long) ((Jedis) nativeConnection).eval(UNLOCK_LUA, Collections.singletonList(lockKey), args);
            }
            return null;
        });
        if(SUCCESS.equals(result)){
            return true;
        }else{
            return false;
        }
    }

```

### <u>3.秒杀限流</u>

[高并发之API接口，分布式，防刷限流]: https://mp.weixin.qq.com/s/N_qfjniWaDqQ1z2mVgA-TA

```java
// TODO 待整理
```

