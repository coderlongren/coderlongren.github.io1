---
title: Mybatis源码分析2
date: 2018-7-12 23:48:40
categories:
	- Mybatis
tags:
	- Mybatis
---

摘要：Mybatis源码学习2
<!-- more -->

* Executor 执行器，来调度StatementHandle， ParameterHandle，resultHandle，来执行相应的SQL
* StatementHandle是核心，使用数据库的Statement来操作
* parameterHandle SQL参数处理
* ResultHandle 对结果集进行封装处理
### 执行器(Executot)，执行Java和数据库交互的东西，存在三种执行器，可以再配置文件中进行选择。
* SIMPLE 简单执行器，默认的
* REUSE 重用预处理
* BATCH 批量更新的执行器

简单的执行器SimpleExecutor的执行过程
```java
 @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      stmt = prepareStatement(handler, ms.getStatementLog());
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }

```
里面的prepareStatement方法
```java
 private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
    Statement stmt;
    Connection connection = getConnection(statementLog);
    stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
    return stmt;
  }
  ```
进过Cnofigration构建StatementHandle，然后再经过里面的prepareStatement方法，
对SQL进行编译处理，他调用了StatementHandle的prepare（）进行编译和基础设置，在通过parameterrize()设置参数，
```
    stmt = handler.prepare(connection, transaction.getTimeout());
    handler.parameterize(stmt);
```
#### StatementHandle
完成参数的设置之后，跟着就是执行查询，update也是这样的，如果需要结果，再用ParameterHandle和resultHandle在包装结果集。
#### parameterHandle
前面看到了StatementHandle使用ParameterHandle进行编译和参数设置
```java
public interface ParameterHandler {

  Object getParameterObject();

  void setParameters(PreparedStatement ps)
      throws SQLException;

}
```

getParameterObject是获取参书，setParameters设置参数，这两个方法都留给默认实现类DefaultParameterHandler去完成。

```java
public class DefaultParameterHandler implements ParameterHandler {

  .......
  @Override
  public Object getParameterObject() {
    return parameterObject;
  }

  @Override
  public void setParameters(PreparedStatement ps) {
    ErrorContext.instance().activity("setting parameters").object(mappedStatement.getParameterMap().getId());
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    if (parameterMappings != null) {
      for (int i = 0; i < parameterMappings.size(); i++) {
        ParameterMapping parameterMapping = parameterMappings.get(i);
        if (parameterMapping.getMode() != ParameterMode.OUT) {
          Object value;
          String propertyName = parameterMapping.getProperty();
          if (boundSql.hasAdditionalParameter(propertyName)) { // issue #448 ask first for additional params
            value = boundSql.getAdditionalParameter(propertyName);
          } else if (parameterObject == null) {
            value = null;
          } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
            value = parameterObject;
          } else {
            MetaObject metaObject = configuration.newMetaObject(parameterObject);
            value = metaObject.getValue(propertyName);
          }
          TypeHandler typeHandler = parameterMapping.getTypeHandler();
          JdbcType jdbcType = parameterMapping.getJdbcType();
          if (value == null && jdbcType == null) {
            jdbcType = configuration.getJdbcTypeForNull();
          }
          try {
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
          } catch (TypeException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          } catch (SQLException e) {
            throw new TypeException("Could not set parameters for mapping: " + parameterMapping + ". Cause: " + e, e);
          }
        }
      }
    }
  }

}
```

### SqlSession运行总结
![mybatis](/images/mybatis.png)

SqlSession通过Executor创建StatementHandle开始，
Statement需要经过散步来进行SQL语句的处理：
* prepared预编译SQL
* parameterrize设置参数
* query/update执行SQL
其中的parameterrize调用parameterHandle的方法设置的，参数是根据类型处理器typeHandle处理的。query/update是通过resultHandle进行结果封装。

