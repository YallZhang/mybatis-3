源码入口：SqlSession类
SqlSession内包含增删改查的方法、以及获取配置文件的方法、以及获取Mapper接口代理实例的方法。
SqlSession内部是通过Executor执行器执行sql语句的。
    
    1.根据mapper文件中的唯一sql-id进行增删改查[Mapper接口代理最终也是通过这几个方法实现的];
    2.根据类类型获取Mapper接口实例[MapperRegistry.getMapper(..)];
    3.获取Configuration和Connection。

A. SqlSession下四个重要的核心类：
    
    1.ParameterHandler
    2.ResultHandler
    3.Executor[BaseExecutor、SimpleExecutor、BatchExecutor、ReuseExecutor、CachingExecutor]
      BaseExecutor内包含事务、Configuration、缓存处理等机制，并实现了部分接口方法，其余的为模板模式，交由子类实现
    4.StatementHandler[RoutingStatementHandler->SimpleStatementHandler、PrepareStatement、CallableStatementHandler]
      4.1. RoutingStatementHandler主要用于生成具体的StatementHandler,并将其存在Routing中的delegate属性中
         StatementHandler内部去构建java.sql.Statement对象以及执行sql语句，并经由ParameterHandler和ResultSetHandler处理入参和返回值。


B. 跟配置相关的类：
    
    1. Configuration类：解析mybatis-config.xml文件的类。
       包含Environment、TransactionFactory和数据源javax.sql.DataSouce；
       包含settings:下划线转驼峰命名；
       各种别名typeAliases、typeHandlers
       mappers文件集合[里面的sql语句被解析为MappedStatement对象]
    2. MappedStatement ： 读取mybatis-mapper.xml配置文件后的一个sql节点对象

C. 跟自定义Mapper接口相关的类：
   
    1. MapperRegistry:读取配置文件configuration，为每个ClassDao接口注册代理工厂Map<Class<?>, MapperProxyFactory<?>> knownMappers
    2. MapperProxy impl InvocationHandler
    3. MapperMethod是真正的代理方法实行者，内部调用sqlSession.curd
    4. 通过MapperProxyFactory动态创建各种DAO层代理类

C. spring事务实现原理：
      参看项目spring-transaction,DataSourceTransactionManager.java 


D. mybatis多源数据:
      参看项目mybatis-multi-datasource

常见异常：
1. ### Error updating database.  Cause: java.lang.IllegalArgumentException: Mapped Statements collection does not contain value for com.tujia.train.mybatis.po.Goods.insertOneGood
   检查mapper文件的namespace是否写错了，路径是否正确
2. 如果开启MyBatis事务管理，则需要手动进行事务提交，否则事务会回滚到原状态;