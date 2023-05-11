# SpringBoot框架简介

#### 架构简介

- 采用SpringBoot+Redis+MySQL（可迁移Oracle，需要预研Mybatis-Generator与Oracle如何配置）

- 中间件配置

    - Mybatis作为ORM框架，可手写复杂SQL
    - Shiro作为权限控制框架
    - Druid作为数据库连接池（Druid采用Log4J2作为日志框架，需要等其修复完毕更新版本）
    - Redis连接采用Lettuce

---

#### 配置项简述

- 目前仅有数据库配置与Redis配置

    - 数据库配置

        - 配置项目简介

          ```properties
          spring.datasource.driver-class-name	//数据库连接驱动配置，MySQL为com.mysql.cj.jdbc.Driver
          spring.datasource.type	// 数据库连接类型配置, Druid为com.alibaba.druid.pool.DruidDataSource
          spring.datasource.url	// 数据库连接URL
          spring.datasource.username	// 数据库连接用户名
          spring.datasource.password	// 数据库连接密码
          ```

        - 在org.springframework.boot.autoconfigure.jdbc.DataSourceProperties中，可以可以看到各个配置项的含义以及默认值

        - 存在问题：

          ```
          数据库连接配置中，密码配置需要加密
          ```


- Redis

    - 配置项简介

      ```properties
      spring.redis.host	// Reids地址
      spring.redis.port	// Redis端口
      spring.redis.lettuce.pool.max-active	//Redis连接池最大活跃数
      spring.redis.lettuce.pool.max-idle		// 池中“空闲”连接的最大数量。 使用负值表示无限数量的空闲连接。
      spring.redis.lettuce.pool.min-idle		// 池中要维护的最小空闲连接数的目标。 此设置仅在驱逐运行之间的时间和时间均为正时才有效。
      spring.redis.lettuce.pool.max-wait		// 当池耗尽时，在抛出异常之前连接分配应该阻塞的最长时间。 使用负值无限期阻止。
      ```

    - 在 org.springframework.boot.autoconfigure.data.redis.RedisProperties中，可以看到各个配置项的含义以及默认值

    - TODO

      ```
      同数据库配置，需要将密码加密
      ```

---

#### 代码框架简介

- ***common模块***
    - common模块为项目基础包，其余模块全部依赖此模块，用于存放其他模块共有Maven依赖，项目整体配置，其他模块共用接口，数据结构，以及工具类

- ***所包含子模块简介：***
    - config包：用于存放项目整体配置，如数据库与Redis（Shiro由于存在拦截器，且ShiroRealm需要依赖Service层，因此该配置存放在了web模块中）
    - intf包：用于存放各个模块可能会共用的基础接口，如各个Template，持久化校验接口等
    - struct包：用于存放各个模块用到的数据结构，注解，常量枚举，异常等
    - util包：存放各种工具类

---

- ***repository模块***

    - 数据数据访问层，存放访问数据库的Mapper，实体类，实体转换器

- ***所包含子模块简介：***

    - assembler包：实体转换器，用于某个实体的转换工作，如DO转换成BO，BO转换成DTO等
    - dao包：存放访问数据库的Mapper接口以及repository仓储类（repository输出转换好的，Service层需要的实体）
    - entity包：存放各个实体类
        - bo包：Business Object：业务对象，具有行为，继承实体对应的Data Object，可以访问数据库（通过SpringContextUtils来获取访问数据库的Repo对象，不被Spring容器管理）
        - pojo包：包含Data Object与DataExample
            - Data Object：数据对象，不具有行为，一般作为与数据库交互的对象
            - DataExample：数据对象对应的Example，由Mybatis-Generator生成
        - dto包：Data Transfer Object 数据传输对象，一般需要实现Java中的Serializable接口，用于串行化传输对象，包含View Object与Param
            - View Object：视图对象，一般作为回传前端的实体对象，不具有行为
            - Param：参数对象，用于接收前端传参，可具有一些简单的行为，如校验参数是否合法等，复杂校验需要交给BO
    - resource文件：存放Mapper的XML文件

---

- ***service模块***
    - 业务层，用于处理业务逻辑

- ***所包含子模块简介：***
    - 自身下面之间存放Service接口，用于对外提供服务。小项目中，由于一个Service只对应一个接口实现类，且业务之间耦合不严重，可以省略Service接口
    - impl包：存放Service接口实现。
    - intf包：存放接口，**注意不是Service接口，是模块中用到的其他接口（如用户信息补全器IUserInfoCompleter）**

---

- ***web模块***

    - 访问控制层，用于接收前端请求，原则上不处理业务逻辑

- ***所包含子模块简介：***

    - controller包：存放Controller。
    - aspect包：存放切面。
    - interceptor：存放拦截器及其配置，以及Shiro相关配置。
    - resource文件：存放项目配置，后续如果后端需要提供HTML文件，也应该存放在这里。

---

#### 细节简述

- ***Mybatis-Generator使用简介：***

    - 主要需要修改其generatorConfig.xml文件

  ![image-20211224150238071](img/video_thumb.png)

---

- BaseRepo简介

    - 功能说明

        - 利用反射封装了Mybatis-Generator自动生成的Mapper中常用的方法， 待实现的有分页查询（已经整合PageHepler）

    - 泛型参数说明

        - **BOClass：** 继承Mybatis-Generator自动生成的POJO类
        - **POJOClass：** Mybatis-Generator自动生成的POJO类

    - 构造函数参数说明

        - **Class<BO> boClazz：** 继承Mybatis-Generator自动生成的POJO类的Class
        - **Class<POJO> pojoClazz：** Mybatis-Generator自动生成的数据库POJO的Class
        - **Class<?> exampleClazz：** Mybatis-Generator自动生成的Example的Class
        - **pojoMapper：**  Mybatis-Generator自动生成的Mapper， 注意此处并非Class

        - 构造函数参数比较多，且不容易记忆，是否需要考虑定义一个参数类， 或者再封装一下， 利用注解等形式更方便地初始化该类

    - 代码示例

      ```java
      @Repository
      public class OperationLogRepo extends BaseRepo<OperationLogBO, OperationLog> {
      
          private final OperationLogMapper operationLogMapper;
      
          public OperationLogRepo(OperationLogMapper operationLogMapper) {
              super(OperationLogBO.class, OperationLog.class, OperationLogExample.class, operationLogMapper);
              this.operationLogMapper = operationLogMapper;
          }
      
          public PageInfo<OperationLog> queryLogByPage(LogPageQueryParam param) {
              OperationLogExample example = new OperationLogExample();
              return PageHelper.startPage(param)
                      .doSelectPageInfo(() -> queryByExample(example));
          }
      }
      ```

---

- 操作日志记录功能说明

    - 功能说明

        - 记录用户请求参数，返回值等信息
        - 使用SpringAOP，在需要记录的请求URI上加注解，即可实现记录日志功能
        - 目前操作日志记录表设计略为简单，可根据业务需要，增加记录字段，并修改切面落库方法

    - 使用说明

        - **moduleType：**模块名称， String类型，如可输入"用户管理， 权限管理等"，可根据业务需要，定义枚举类

        - **operationType：**操作类型，已封装成枚举类***OperationType***，目前有增删改查，以及一个其他，可根据业务需要，向该枚举类中添加操作类型。

          ```java
          @RequestMapping(value = "/test", method = RequestMethod.GET)
          @SaveLog(moduleType = "测试", operationType = OperationType.QUERY)
          public Object testLog() {
              return ApiResult.getSuccess("请求成功");
          }
          ```

    - 存在的问题：

        - MySQL存储容量有限，目前日志记录功能没有定期清库的功能，是否需要考虑添加。
        - 落库操作是否可调整为异步？并在落库失败后记录日志，以便排除。

---

- 健康检查说明：

    - 功能说明

        - 使用Spring-Boot自带的Actuator提供项目健康检查功能

    - 使用说明

        - 项目启动后，访问/actuator/health检查健康状态

    - 配置项说明

      ```
      management.endpoint.health.show-details=always	// 显示所有的状态信息，否则，默认只显示服务总体状态，不方便排查
      // 其他待补充
      ```

---

