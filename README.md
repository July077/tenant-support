## 1. 项目概述

用于 Spring Boot 应用的多租户插件，支持以低侵入式的方式对现有应用进行多租户改造。目前已经接入线上服务在测试中。

支持特性如下:

- 低侵入式让现在项目支持多租户。
- 采用注解方式对常见业务进行多租户改造。
- 支持多租户模式，同时代码可以无缝切换回单租户模式下。
-  SpringCache 相关注解的多租户支持。
-  RedisTemplate 、 StringRedisTemplate 大部分函数多租户支持。
-  Redisson 中大部分函数的多租户支持。
-  Quartz 定时任务的多租户支持。
-  @Async 异步业务逻辑的多租户支持。
-  Spring Task 的多租户支持。
- 线程池执行业务场景下的多租户支持。





## 2. 项目背景

由于我司现在项目需要多租户支持，在改造过程中对相关功能单独抽取出来了一个模块，用于多个服务进行共享。插件是根据我司的项目来进行定制的，故不一定适合所有的业务场景。

我们现在面临的问题:

1. 应用是 Spring Boot + Mybatis + PostgreSQL (主要数据库) + MySQL (辅助数据库)， 故进行多租户改造需围绕现在技术栈进行。
2. 由于数据库是 PostgreSQL ，网上的多租户文章基本以 MySQL为主，虽说可以参考思路，但总归是需要做些改动的。
3. 解决多租户 SQL 查询问题只是其中的一部分，我们还有其它的场景也需要解决多租户的，例如比较常见的缓存、定时任务。
4. 由于项目时间也比较久了，积累的业务代码还是稍微比较多，后端代码预计  20w +。如果大面积的改造代码，对系统测试来说是非常困难的。
5. 由于是业务系统，很多功能调用大量报表统计的 SQL 。 不同于常规的互联网应用，管理系统中的报表 SQL 基本都是非常复杂，单表查询的 SQL 基本是看不到的，故如果是对 SQL 加 tenant_id 字段实行起来比较困难。
6. 租户数量不会过多，会根据租户的规模大小进行不同的方案处理，如大租户的话会考虑直接内网单独部署一套，无需考虑太多租户数量过多的问题。
7. 再一个就是任务时间紧且参与人数少。



目前 Github 中已经有不少多租户支持相关的插件了，但是大部分看下来，总结起来如下:

- 采用拦截器对 SQL 进行特殊处理 。 这种做法网上已经有了现成的插件，如 MyBatis Plus 中已经有了多租户插件，开启后即可以对 **简单并且是符合条件** 的SQL 进行多租户条件的拼接。 这种方法我们在以前的电商项目中已经实验过了，由于是对 C 端用户提供的接口，故SQL查询都比较简单一点，这种方案在 C端用户接口下是可行的。
- 使用中间件进行 SQL 路由，如使用 mycat 、shardingsphere以及其它类似的插件进行中间层处理。这种不太合适，我们 SQL 中有跨库关联SQL的情况 ( MySQL 中叫跨 database , PostgreSQL 中叫跨 schema )
- 暂没有看到有提供对其它开源组件的多租户支持方案。 企业项目中定时任务的场景是不可避免的，如 quartz 等。
- 直接对项目进行硬编码的，如原先函数中入参是一个，改造后就将函数的入参改造成两个，每一个函数都侵入式的加一个 tenantId 参数。
- 基于服务网关进行请求转发，网关后面每个租户对应着一整套的服务，根据请求中的信息来转发到不同的租户服务上。
- 挂羊头卖狗肉，写着多租户支持，点进去看代码压根就没有多租户相关的代码。





## 3. 设计思路

### 3.1 多租户实现方式

业界常用的几种多租户方式: 

-  **独立数据库实例**

  优点:  数据最安全，租户的数据在物理层级隔离。 性能很好。

  缺点:  维护成本很高。

  SQL 调整:  不需要对SQL进行调整。

  

- **独立数据库**

  优点: 数据相对较安全， 在各方面都比较均衡。 

  s: 维护成本较高

  SQL 调整:  不需要对SQL进行调整。

  

- **共享数据库表**

  优点:  维护成本最低，不需要同步多套数据库表等信息。

  缺点: 扩展起来较为困难、数据很不安全、性能较差、需要在所有数据库表中增加 tenant_id 字段。

  SQL 调整: 需要对 SQL 进行调整。



### 3.2 多租户数据库落地方案



### 3.3 租户切换思路

















## 4. 插件依赖

开发语言:  JAVA 

运行环境:  JDK8+

插件依赖的框架: 

- Spring Boot 2.4.1   强依赖，因为我们项目都是基于 Spring Boot 开发的。
- MyBatis      DAO 层框架
- Dynamic Datasource   3.3.2         Mybatis Plus 团队开发的一款多数据源插件，项目原有引用插件，故继续沿用。
- TTL   阿里团队开源的组件，用于线程池模式下变量传递。
- Spring JDBC   数据源部分功能会使用到。





## 5. 快速接入

 由于暂时未发布到 Maven 中央仓库，故暂时需要手动打包到本地 Maven 仓库。

1. 下载本项目:  

   ````shell
   gie clone https://github.com/July077/tenant-support
   ````

   

2. 下载后使用 IDEA 或者 Eclipse 中打开，如果打开后 IDE 没有识别到项目，在 IDEA 中可以找到  tenant-support / pom.xml ，然后对着 pom.xml 右键可以找到 **Add as Maven Project** ，点击后 IDEA 将开始索引该项目，稍等片刻可完成索引。

   

3. 在 tenant-support 目录下执行下列命令

   ````shell
   mvn clean install -Dmaven.test.skip=true -Pprod
   ````

   执行成功后可在本地 Maven 仓库中   ``` com > suny > tenant-support ```   中找到对应版本的 Jar 包。

   

4. 在需要接入多租户支持的项目 pom.xml 中添加依赖: 

   ````xml
           <dependency>
               <groupId>com.suny.tenant</groupId>
               <artifactId>tenant-support</artifactId>
               <version>对应的版本号</version>
           </dependency>
   ````

   

5. 在 Spring Boot 启动类中添加注解  **@EnabledMultipleTenant**

   

6. 配置好所有的数据源信息。

   

7. 配置默认的租户ID属性，找到 application.yml ，加入以下内容 :

   ````yaml
   tenant:
     defaultTenantId: 默认的租户ID
   ````

   

8. 对相应的业务代码进行微调，如添加注解或者其它相应的改动。具体的用法看下面章节介绍。

   

9. 点击启动项目，查看控制台中是否输出租户相关的日志，同时对相关的业务接口测试。





## 6. 具体用法

### 6.1 切换多租户、单租户模式

- 多租户模式

  在项目启动类上添加 **@EnabledMultipleTenant** 注解。

  

- 单租户模式

​       在项目启动类上添加  **@EnableSingleTenant** 注解

  

### 6.2 Spring Cache 支持

- @Cacheable   支持，无需额外调整。
- @CachePut    支持，无需额外调整。
- @CacheEvict  支持，无需额外调整。 allEntries = true 时只会移除当前租户相关的缓存，不会影响到其他租户缓存。



### 6.3 Spring Task 支持

 在执行业务的函数上额外添加  **@TenantScheduledTask** 注解，注解有以下可选择的参数: 

- onlyMaster    默认值为 **fasle** 。 当将值调整为 **true** 时将只会以主库租户身份去执行业务逻辑。

  

````java
@Component
public class DemoTask {
  
    @TenantScheduledTask
    @Scheduled
    public void execute() {
       // 执行业务
    }
}

````





### 6.4 Spring Data Redis  支持

主要是支持以下两个常用的工具类中大部分函数，少部分函数不支持！！！

#### 6.4.1 RedisTemplate 支持

##### 6.4.1.1 基础函数支持  
  - [x] hasKey()

  - [x] countExistingKeys()

  - [x] delete()

  - [x] unlink()

  - [x] type()

  - [x] keys()

  - [x] rename()

  - [x] renameIfAbsent()

  - [x] expire()

  - [x] expireAt()

  - [x] persist()

  - [x] move()

  - [x] dump()

  - [x] restore()

  - [x] getExpire()

  - [x] watch()

  - [ ] convertAndSend  暂时不支持，需要手动调整代码适配。

##### 6.4.1.2.  valueOps  支持函数
- [x] get()
- [x] append()
- [x] size()
- [x] increment()
- [x] set()
- [x] getAndSet()
- [x] setBit()
- [x] getBit()
- [x] bitField()
- [x] decrement()
- [x] multiSet()
- [x] setIfAbsent()
- [x] multiSetIfAbsent()


##### 6.4.1.3.  listOps  支持函数
- [x] index()
- [x] remove()
- [x] size()
- [x] trim()
- [x] set()
- [x] range()
- [x] getOperations()
- [x] rightPopAndLeftPush()
- [x] rightPopAndLeftPush()
- [x] rightPopAndLeftPush()
- [x] rightPushIfPresent()
- [x] leftPushIfPresent()
- [x] rightPush()
- [x] rightPush()
- [x] rightPop()
- [x] rightPop()
- [x] rightPop()
- [x] leftPush()
- [x] leftPush()
- [x] leftPop()
- [x] leftPop()
- [x] leftPop()
- [x] leftPushAll()
- [x] leftPushAll()
- [x] rightPushAll()
- [x] rightPushAll()

##### 6.4.1.4.  setOps  支持函数
- [x] remove()
- [x] size()
- [x] pop()
- [x] pop()
- [x] members()
- [x] union()
- [x] union()
- [x] union()
- [x] move()
- [x] scan()
- [x] difference()
- [x] difference()
- [x] difference()
- [x] isMember()
- [x] getOperations()
- [x] distinctRandomMembers()
- [x] intersectAndStore()
- [x] intersectAndStore()
- [x] intersectAndStore()
- [x] differenceAndStore()
- [x] differenceAndStore()
- [x] differenceAndStore()
- [x] unionAndStore()
- [x] unionAndStore()
- [x] unionAndStore()
- [x] randomMember()
- [x] randomMembers()
- [x] intersect()
- [x] intersect()
- [x] intersect()

##### 6.4.1.5.  streamOps  应用场景少，暂不适配



##### 6.4.1.6.  zSetOps
- [x] add()
- [x] add()
- [x] remove()
- [x] count()
- [x] size()
- [x] removeRange()
- [x] range()
- [x] scan()
- [x] reverseRange()
- [x] getOperations()
- [x] score()
- [x] zCard()
- [x] intersectAndStore()
- [x] intersectAndStore()
- [x] intersectAndStore()
- [x] intersectAndStore()
- [x] rangeByScoreWithScores()
- [x] rangeByScoreWithScores()
- [x] removeRangeByScore()
- [x] reverseRangeByScore()
- [x] reverseRangeByScore()
- [x] reverseRangeWithScores()
- [x] reverseRangeByScoreWithScores()
- [x] reverseRangeByScoreWithScores()
- [x] unionAndStore()
- [x] unionAndStore()
- [x] unionAndStore()
- [x] unionAndStore()
- [x] rangeByLex()
- [x] rangeByLex()
- [x] incrementScore()
- [x] reverseRank()
- [x] rangeByScore()
- [x] rangeByScore()
- [x] rank()
- [x] rangeWithScores()

##### 6.4.1.7.  geoOps  支持函数
- [x] add()
- [x] add()
- [x] add()
- [x] add()
- [x] remove()
- [x] hash()
- [x] position()
- [x] geoRadiusByMember()
- [x] geoRadiusByMember()
- [x] geoRadiusByMember()
- [x] distance()
- [x] distance()
- [x] geoAdd()
- [x] geoAdd()
- [x] geoAdd()
- [x] geoAdd()
- [x] geoDist()
- [x] geoDist()
- [x] radius()
- [x] radius()
- [x] radius()
- [x] radius()
- [x] radius()
- [x] geoHash()
- [x] geoPos()
- [x] geoRadius()
- [x] geoRadius()
- [x] geoRemove()

##### 6.4.1.8.  hllOps  支持函数
- [x] add()
- [x] size()
- [x] delete()
- [x] union()



#### 6.4.2 StringRedisTemplate 支持

StringRedisTemplate 跟上面的 RedisTemplate 一致的，除了序列化方式并无其他很大差异。




### 6.5 Redisson 支持

 Redisson 目前的版本并不支持多租户，故需单独适配。

issue： https://github.com/redisson/redisson/issues/3462

answer :  dynamic change of configuration is not supported

### 6.6 Quartz 支持



### 6.7 本地 Map 等缓存支持



### 6.8 切换数据源支持



### 6.9 @Async 业务支持



### 6.10 自定义线程池执行业务支持







## 7. 插件预留扩展点



### 7.1 租户、数据源数据加载扩展



#### 7.1.1 自定义加载数据源信息



#### 7.1.2 动态添加数据源



#### 7.1.3 自定义加载租户信息



#### 7.1.4 动态添加租户





### 7.2  业务数据接口扩展点

#### 7.2.1 自定义授权信息扩展



#### 7.2.2 自定义缓存 key 生成策略



#### 7.2.3 自定义 Quartz job key、Trigger Key、Identity Key 生成策略









## 8. 编译以及开发调试





## 9. 插件限制

### 9.1 @PostConstruct 初始化限制



### 9.2 Aop 限制









## 10. FAQ

- 如何卸载插件

  插件支持多租户以及单租户模式的快速切换，卸载插件有以下两种方法: 

  1. 直接将启动类上的注解  `@EnabledMultipleTenant` 切换成  `@EnableSingleTenant` 即可恢复到多租户前的状态。此方法接近于无改造。 (建议使用)

  2. 将 pom 文件中的依赖移除，并且将业务代码中的租户相关的类移除，即可切换到多租户前的状态，这种方法需要稍微调整下代码。(不建议使用)

     






## 11. 参考资料



