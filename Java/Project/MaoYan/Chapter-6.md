### 第6章知识总结

#### 1. 熟悉影院业务逻辑（电影信息，电影场次，座位情况等）

#### 2. `Mybatis`中一对多的查询

#### 3. `Mybatis`中 美元符号与 # 的区别
***
##### 3.1 概述
这个老问题面试经常会在问，网上的答案也有很多。但最近再问问自己，脑海中居然整理不出来答案；因此站在源码的角度来分析一下这两个符号的区别吧。(能比较深刻地记忆下来)

##### 3.2 源码解析
以下是一个普通条件查询的时序图：
![](https://ws1.sinaimg.cn/large/6b297ce5ly1g3zujbft3cj20pv0793ys.jpg)
```java
@Test
  public void parameterTest() {
    SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
    SqlSession sqlSession = null;
    try {
      Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
      SqlSessionFactory factory = sqlSessionFactoryBuilder.build(reader);
      sqlSession = factory.openSession();
      UserMapper mapper = sqlSession.getMapper(UserMapper.class);
      User user = mapper.getUserById(1);
      System.out.println(user);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      sqlSession.close();
    }
  }
```
UserMapper.xml:
```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.jason.mapper.UserMapper">
  <select id="getUserById" resultType="com.mybatis.jason.model.User">
     select id, userName from user where id = ${id}
  </select>
</mapper>
```

我们都知道`Mybatis`中的mapper是使用了代理模式的动态代理，因此`UserMapper`会委托`MapperProxy`帮我们进行查询。
```java
//MapperProxy
@Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else if (isDefaultMethod(method)) {
        return invokeDefaultMethod(proxy, method, args);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
    // 注意哦：这里会有一个缓存(俗称一级缓存),每次查询会先缓存里面找。
    final MapperMethod mapperMethod = cachedMapperMethod(method);
    return mapperMethod.execute(sqlSession, args);
  }
  //根据key去缓存中查，如果没有的话会新建一份并保存在缓存中
  private MapperMethod cachedMapperMethod(Method method) {
    return methodCache.computeIfAbsent(method, k -> new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
  }
```
`MapperMethod`的execute是一个入点，在这个方法里面会判断我们的sql语句是INSERT,UPDATE,DELETE,SELECT的哪一种。在这里只针对SELECT的情况。
SELECT后还会判断是查询一个还是多个，我们这里是查询一个。因此会调用到sqlSession的selectOne方法。
```java
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
      case INSERT: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.insert(command.getName(), param));
        break;
      }
      case UPDATE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.update(command.getName(), param));
        break;
      }
      case DELETE: {
        Object param = method.convertArgsToSqlCommandParam(args);
        result = rowCountResult(sqlSession.delete(command.getName(), param));
        break;
      }
      case SELECT:
        if (method.returnsVoid() && method.hasResultHandler()) {
          executeWithResultHandler(sqlSession, args);
          result = null;
        } else if (method.returnsMany()) {
          result = executeForMany(sqlSession, args);
        } else if (method.returnsMap()) {
          result = executeForMap(sqlSession, args);
        } else if (method.returnsCursor()) {
          result = executeForCursor(sqlSession, args);
        } else {
          Object param = method.convertArgsToSqlCommandParam(args);
          result = sqlSession.selectOne(command.getName(), param);
          if (method.returnsOptional()
              && (result == null || !method.getReturnType().equals(result.getClass()))) {
            result = Optional.ofNullable(result);
          }
        }
        break;
      case FLUSH:
        result = sqlSession.flushStatements();
        break;
      default:
        throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
      throw new BindingException("Mapper method '" + command.getName()
          + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
    }
    return result;
  }
```
`SqlSession`是`Mybatis`中十分重要的对象，几乎所有操作都是依靠`SqlSession`的。默认是会使用`DefaultSqlSession`.
```java
@Override
  public <T> T selectOne(String statement) {
    return this.selectOne(statement, null);
  }

  @Override
  public <T> T selectOne(String statement, Object parameter) {
    // Popular vote was to return null on 0 results and throw exception on too many.
    List<T> list = this.selectList(statement, parameter);
    if (list.size() == 1) {
      return list.get(0);
    } else if (list.size() > 1) {
      throw new TooManyResultsException("Expected one result (or null) to be returned by selectOne(), but found: " + list.size());
    } else {
      return null;
    }
  }
  //即使查询一个也还是调用selectList这个方法的
  @Override
  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
```
接下来就到了executor(执行器)这一步了，来到这一步就能看到#与$的区别了.`Mybatis`对数据库的所有操作最终都会由executor(执行器来执行).
默认是会使用SimpleExecutor来执行的.针对查询的这种情况,会调用到doQuery这个方法.
```java
public class SimpleExecutor extends BaseExecutor {

  public SimpleExecutor(Configuration configuration, Transaction transaction) {
    super(configuration, transaction);
  }
  
  @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.query(stmt, resultHandler);
     } finally {
       closeStatement(stmt);
    }
  }
}
```
然后的步骤就是跟原生的jdbc没什么区别了,就先把statement创建出来,然后再由resultSet处理结果.而statement也分为很多种,因此要先通过RoutingStatementHandler来判断要创建哪一种类型的statement.而这个statement如果没有指定的话,默认是PREPARED.
```java
// XMLStatementBuilder 该类用来构建MapperStatement的.
  StatementType statementType = StatementType.valueOf(nodeToHandle.getStringAttribute("statementType", StatementType.PREPARED.toString()));
```
```java
public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
    switch (ms.getStatementType()) {
      case STATEMENT:
        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case PREPARED:
        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      case CALLABLE:
        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
        break;
      default:
        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
    }
  }
```


4. `Mybatis-Plus`中的分页使用

5. 异常处理

6. 在数据库中存储中文字符是非常耗资源的，因此不建议将两张有中文数据的表合在一起查询，一般来说连接查询效率会比其他查询高。（例如：子查询，子查询的过程中还需要生成一张临时表）

7. `Dubbo`特性： 结果缓存和并发控制，连接控制
