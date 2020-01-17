---
title: " Spring boot集成Quartz\t\t"
tags:
  - spring
url: 392.html
id: 392
comments: false
categories:
  - spring boot
date: 2019-04-04 10:24:07
---
说明：
Spring boot2.x之后Quartz有Springboot的Starter，这里分两部分，使用`2.0.5.RELEASE`作为Spring boot版本

- 不使用Starter集成
- 使用Starter集成

## 不使用Starter集成
引入pom

```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
    <!--使用Springboot中的Spring版本-->
    <version>${spring.version}</version>
</dependency>
```

### 在resource下新建quartz.properties

```properties
org.quartz.scheduler.instanceName=MyScheduler
org.quartz.scheduler.instanceId=AUTO
org.quartz.threadPool.threadCount=5
org.quartz.scheduler.rmi.export = false
org.quartz.scheduler.rmi.proxy = false
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.scheduler.wrapJobExecutionInUserTransaction = false
# 默认存储在内存中
#org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore
# 这里使用数据库存储，需导入sql，sql位置：quartz包下（官网下载）org/quartz/impl/jdbcjobstore
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.tablePrefix=QRTZ_
org.quartz.jobStore.dataSource=myDS
# 使用Druid替换默认的C3P0
org.quartz.dataSource.myDS.connectionProvider.class=com.cc.fortest.common.config.DruidConnectionProvider
org.quartz.dataSource.myDS.driver=com.mysql.jdbc.Driver
org.quartz.dataSource.myDS.URL=jdbc:mysql://127.0.0.1:3306/quartz?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false
org.quartz.dataSource.myDS.user=root
org.quartz.dataSource.myDS.password=123456
org.quartz.dataSource.myDS.maxConnection=5
```

### 配置类

```java
import org.quartz.Scheduler;
import org.springframework.beans.factory.config.PropertiesFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;

import java.io.IOException;
import java.util.Properties;

@Configuration
public class SchedledConfiguration {

    @Bean(name = "SchedulerFactory")
    public SchedulerFactoryBean schedulerFactoryBean() throws IOException {
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        factory.setQuartzProperties(quartzProperties());
        return factory;
    }

    @Bean
    public Properties quartzProperties() throws IOException {
        PropertiesFactoryBean propertiesFactoryBean = new PropertiesFactoryBean();
        propertiesFactoryBean.setLocation(new ClassPathResource("/quartz.properties"));
        //在quartz.properties中的属性被读取并注入后再初始化对象
        propertiesFactoryBean.afterPropertiesSet();
        return propertiesFactoryBean.getObject();
    }

    @Bean(name = "Scheduler")
    public Scheduler scheduler(SchedulerFactoryBean SchedulerFactory) {
        return SchedulerFactory.getScheduler();
    }
}
```

### 新建一个Job

```java
import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

import java.util.Date;

public class MyJob implements Job {
    public static final String CRON = "0/5 * * * * ?";

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("任务执行进入,执行时间, "+new Date());
        JobDataMap dataMap = jobExecutionContext.getMergedJobDataMap();
        String name = dataMap.getString("name");
        System.out.println("Hello "+name);
    }
}
```

### 测试类

```java
	@Autowired
    @Qualifier("Scheduler")
    private Scheduler scheduler;

	@Test
    public void quartzTest() throws SchedulerException, IOException {
        scheduler.start();
        JobDataMap jobDataMap = new JobDataMap();
        jobDataMap.put("name", "John");
        JobDetail jobDetail = JobBuilder.newJob(MyJob.class)
                .withIdentity(MyJob.class.getSimpleName(), "group")
                .setJobData(jobDataMap)
                .build();
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(MyJob.CRON);
        CronTrigger trigger = TriggerBuilder.newTrigger()
                .withIdentity(MyJob.class.getSimpleName(), "group")
                .withSchedule(cronScheduleBuilder)
                .build();
        try {
            scheduler.scheduleJob(jobDetail, trigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
        }
        System.in.read();
    }
```

### 使用Druid替换Quartz默认连接池C3P0

新建一个类实现`ConnectionProvider`

```java
import com.alibaba.druid.pool.DruidDataSource;
import lombok.Data;
import org.quartz.SchedulerException;
import org.quartz.utils.ConnectionProvider;

import java.sql.Connection;
import java.sql.SQLException;

/**
 * 将Quartz的连接池改为Druid，常量Quartz自动注入
 */
@Data
public class DruidConnectionProvider implements ConnectionProvider {
    //JDBC驱动
    private String driver;
    //JDBC连接串
    private String URL;
    //数据库用户名
    private String user;
    //数据库用户密码
    private String password;
    //数据库最大连接数
    private int maxConnection;
    //数据库SQL查询每次连接返回执行到连接池，以确保它仍然是有效的。
    private String validationQuery;
    private boolean validateOnCheckout;
    private int idleConnectionValidationSeconds;
    private String maxCachedStatementsPerConnection;
    private String discardIdleConnectionsSeconds;
    private static final int DEFAULT_DB_MAX_CONNECTIONS = 10;
    private static final int DEFAULT_DB_MAX_CACHED_STATEMENTS_PER_CONNECTION = 120;
    //Druid连接池
    private DruidDataSource datasource;
    
    public Connection getConnection() throws SQLException {
        return datasource.getConnection();
    }
    public void shutdown() throws SQLException {
        datasource.close();
    }
    public void initialize() throws SQLException{
        if (this.URL == null) {
            throw new SQLException("DBPool could not be created: DB URL cannot be null");
        }
        if (this.driver == null) {
            throw new SQLException("DBPool driver could not be created: DB driver class name cannot be null!");
        }
        if (this.maxConnection < 0) {
            throw new SQLException("DBPool maxConnectins could not be created: Max connections must be greater than zero!");
        }
        datasource = new DruidDataSource();
        try{
            datasource.setDriverClassName(this.driver);
        } catch (Exception e) {
            try {
                throw new SchedulerException("Problem setting driver class name on datasource: " + e.getMessage(), e);
            } catch (SchedulerException e1) {
            }
        }
        datasource.setUrl(this.URL);
        datasource.setUsername(this.user);
        datasource.setPassword(this.password);
        datasource.setMaxActive(this.maxConnection);
        datasource.setMinIdle(1);
        datasource.setMaxPoolPreparedStatementPerConnectionSize(DEFAULT_DB_MAX_CONNECTIONS);
        if (this.validationQuery != null) {
            datasource.setValidationQuery(this.validationQuery);
            if(!this.validateOnCheckout)
                datasource.setTestOnReturn(true);
            else
                datasource.setTestOnBorrow(true);
            datasource.setValidationQueryTimeout(this.idleConnectionValidationSeconds);
        }
    }
}
```

## 使用Starter集成

引入pom

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

```java
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;

import java.util.Date;

public class MyJob extends QuartzJobBean {
    public static final String CRON = "0/5 * * * * ?";

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        System.out.println("任务执行进入,执行时间, "+new Date());
        JobDataMap dataMap = jobExecutionContext.getMergedJobDataMap();
        String name = dataMap.getString("name");
        System.out.println("Hello "+name);
    }
}
```