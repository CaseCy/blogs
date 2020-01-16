---
title: " 使用Jedis连接Redis\t\t"
tags:
  - redis
url: 93.html
id: 93
comments: false
categories:
  - redis
date: 2018-10-19 08:47:33
---

tags： Redis Jedis pool

* * *

如果是通过外网访问Redis一定要开放防火墙的端口权限，不然不能访问 导入pom.xml

        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
    

Java代码

    Jedis jedis = new Jedis("127.0.0.1",6379);      //新建一个到指定ip和端口的Jedis对象
    jedis.set("test","这是一个测试");
    

创建对象还有一些其他方法，这里不赘述

* * *

通过JedisPool连接
-------------

设置JedisPool，参照[JedisPoolConfig配置](http://www.cnblogs.com/tankaixiong/p/4048167.html)

    JedisPoolConfig config = new JedisPoolConfig();
    
    //连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true
    config.setBlockWhenExhausted(true);
    
    //设置的逐出策略类名, 默认DefaultEvictionPolicy(当连接超过最大空闲时间,或连接数超过最大空闲连接数)
    config.setEvictionPolicyClassName("org.apache.commons.pool2.impl.DefaultEvictionPolicy");
    
    //是否启用pool的jmx管理功能, 默认true
    config.setJmxEnabled(true);
    
    //MBean ObjectName = new ObjectName("org.apache.commons.pool2:type=GenericObjectPool,name=" + "pool" + i); 默 认为"pool", JMX不熟,具体不知道是干啥的...默认就好.
    config.setJmxNamePrefix("pool");
    
    //是否启用后进先出, 默认true
    config.setLifo(true);
    
    //最大空闲连接数, 默认8个
    config.setMaxIdle(8);
    
    //最大连接数, 默认8个
    config.setMaxTotal(8);
    
    //获取连接时的最大等待毫秒数(如果设置为阻塞时BlockWhenExhausted),如果超时就抛异常, 小于零:阻塞不确定的时间,  默认-1
    config.setMaxWaitMillis(-1);
    
    //逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
    config.setMinEvictableIdleTimeMillis(1800000);
    
    //最小空闲连接数, 默认0
    config.setMinIdle(0);
    
    //每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
    config.setNumTestsPerEvictionRun(3);
    
    //对象空闲多久后逐出, 当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出,不再根据MinEvictableIdleTimeMillis判断  (默认逐出策略)   
    config.setSoftMinEvictableIdleTimeMillis(1800000);
    
    //在获取连接的时候检查有效性, 默认false
    config.setTestOnBorrow(false);
    
    //在空闲时检查有效性, 默认false
    config.setTestWhileIdle(false);
    
    //逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
    config.setTimeBetweenEvictionRunsMillis(-1);
    

注：Jedis低版本可能不存在`maxTotal`和`maxWaitMillis`，使用`maxActive`和`maxWait`

    JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            jedisPoolConfig.setMaxIdle(5);      //设置最大空闲数
            jedisPoolConfig.setBlockWhenExhausted(true);    //设置连接耗尽时是否阻塞，true为阻塞到超时（默认），false抛出异常
            jedisPoolConfig.setEvictionPolicyClassName("org.apache.commons.pool2.impl.DefaultEvictionPolicy");//设置逐出策略
            jedisPoolConfig.setJmxEnabled(true);        //是否启用pool的jmx
            jedisPoolConfig.setMinIdle(2);      //初始化连接数
            jedisPoolConfig.setMaxWaitMillis(3000);//设置资源耗尽时最大阻塞时间
            JedisPool jedisPool = new JedisPool(jedisPoolConfig,"127.0.0.1",6379,3000);
            Jedis jedis = jedisPool.getResource();
            String aa = jedis.get("aa");
            System.out.println(aa);
    

* * *

Jedis集成Spring
-------------

配置文件加载

    <context:property-placeholder location="classpath:config/redis.properties" ignore-unresolvable="true" file-encoding="utf-8"/>
    <!--配置jedispoolconfig-->
        <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
            <property name="minIdle" value="${jedis.minIdle}"></property>
            <property name="lifo" value="${jedis.lifo}"></property>
            <property name="maxIdle" value="${jedis.maxIdle}"></property>
            <property name="maxTotal" value="${jedis.maxTotal}"></property>
            <property name="maxWaitMillis" value="${jedis.maxWaitMillis}"></property>
            <property name="minEvictableIdleTimeMillis" value="${jedis.minEvictableIdleTimeMillis}"></property>
            <property name="numTestsPerEvictionRun" value="${jedis.numTestsPerEvictionRun}"></property>
            <property name="softMinEvictableIdleTimeMillis" value="${jedis.softMinEvictableIdleTimeMillis}"></property>
            <property name="testOnBorrow" value="${jedis.testOnBorrow}"></property>
            <property name="testWhileIdle" value="${jedis.testWhileIdle}"></property>
            <property name="timeBetweenEvictionRunsMillis" value="${jedis.timeBetweenEvictionRunsMillis}"></property>
            <property name="blockWhenExhausted" value="${jedis.blockWhenExhausted}"></property>
            <property name="jmxEnabled" value="${jedis.jmxEnabled}"></property>
        </bean>
        <!--配置jedispool-->
        <bean id="jedisPool" class="redis.clients.jedis.JedisPool" scope="singleton">
            <constructor-arg index="0" ref="jedisPoolConfig" />
            <constructor-arg index="1" value="${jedis.address}" />
            <constructor-arg index="2" value="${jedis.port}" type="int" />
            <constructor-arg index="3" value="0" type="int" />
        </bean>
    

通过上面的方式，使用`@Autowired`注入的时候会自动注入`jedispool`并且使用配置文件中的配置 redis.properties:

    #设置初始化实例数
    jedis.minIdle=1
    #设置最大空闲数
    jedis.maxIdle=5
    #设置进出策略，默认为true,LIFO（后进先出）false为FIFO
    jedis.lifo=true
    #设置最大连接数
    jedis.maxTotal=10
    #连接耗尽时。最大等待时间
    jedis.maxWaitMillis=3000
    #逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
    jedis.minEvictableIdleTimeMillis=1800000
    #每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
    jedis.numTestsPerEvictionRun=3
    #对象空闲多久后逐出, 当空闲时间>该值 且 空闲连接>最大空闲数 时直接逐出,不再根据MinEvictableIdleTimeMillis判断  (默认逐出策略)
    jedis.softMinEvictableIdleTimeMillis=1800000
    #在获取连接的时候检查有效性, 默认false
    jedis.testOnBorrow=true
    #在空闲时检查有效性, 默认false
    jedis.testWhileIdle=true
    #逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
    jedis.timeBetweenEvictionRunsMillis=1000
    #连接耗尽时是否阻塞, false报异常,ture阻塞直到超时, 默认true
    jedis.blockWhenExhausted=true
    #是否启用pool的jmx管理功能, 默认true
    jedis.jmxEnabled=true
    
    ######################JedisPool################################
    jedis.address=127.0.0.1
    jedis.port=6379
    

上面设置只能为参考，不具有实际项目的应用能力  
Java代码

        @Autowired
        private JedisPool jedisPool;
    
        public String get(String key){
            try {
                Jedis jedis = jedisPool.getResource();
                return jedis.get(key);
            } catch (Exception e) {
                e.printStackTrace();
               log.error("get Error!"+e);
            }
            return null;
        }
    

也可以只注入JedisPoolConfig，手动创建JedisPool

    @Component
    public class JedisUtil {
        @Autowired
        JedisPoolConfig jedisPoolConfig;
    
        private static JedisPool jedisPool;
    
        @Value("${jedis.address}")
        private String redisAddress;
    
        @Value("${jedis.port}")
        private int redisPort;
    
        private static final Log log = LogFactory.getLog(JedisUtil.class);
    
        //单例
        private JedisPool getPool(){
            if (jedisPool==null){
                synchronized (JedisUtil.class){
                    if (jedisPool==null){
                        jedisPool = new JedisPool(jedisPoolConfig,redisAddress,redisPort);
                    }
                }
            }
            return jedisPool;
        }
    
        public String get(String key){
            try{
                Jedis jedis = getPool().getResource();
                return jedis.get(key);
            }catch (Exception e){
                log.error("get Error"+e);
            }
            return null;
        }
    }