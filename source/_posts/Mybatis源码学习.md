---
title: Mybatis源码学习
date: 2018-2-25 21:02:17
categories:
	- Mybatis
tags:
	- Mybatis
---
## Mybatis的初始化
× configuration 配置
    × properties 属性
    × settings 设置
   × typeAliases 类型命名
   × typeHandlers 类型处理器
   × objectFactory 对象工厂
   × plugins 插件
   × environments 环境
       ×environment 环境变量
       × transactionManager 事务管理器
       ×dataSource 数据源
×映射器
MyBatis采用了一个非常直白和简单的方式---使用 org.apache.ibatis.session.Configuration 对象作为一个所有配置信息的容器

* mybatis的初始化的过程，就是创建Configration对象的过程 *
1. 基于XML配置文件：基于XML配置文件的方式是将MyBatis的所有配置信息放在XML文件中，MyBatis通过加载并XML配置文件，将配置文信息组装成内部的Configuration对象
2. 基于Java API：这种方式不使用XML配置文件，需要MyBatis使用者在Java代码中，手动创建Configuration对象，然后将配置参数set 进入Configuration对象中

### 基于xml配置文件创建Configration对象的过程
```java
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sqlSession = sqlSessionFactory.openSession();
List list = sqlSession.selectList("com.foo.bean.BlogMapper.queryAllBlogInfo");
```
上面就是 执行 queryAllBloginfo()的过程
经历了mybatis初始化 -->创建SqlSession -->执行SQL语句 返回结果三个过程。
真实创建就是那一句 `SqlSessionFactoryBuilder().build(inputStream);`
SqlSessionFactoryBuilder根据传入的数据流生成Configuration对象，然后根据Configuration对象创建默认的SqlSessionFactory实例。
![](http://img.blog.csdn.net/20140718131020765)
`
1. 调用 SqlSessionFactoryBuilder对象的build(inputStream)方法
2. SqlSessionFactoryBuilder会根据inputStream流创建XMLConfigBuilder对象
3. SqlSessionFactoryBuilder调用XMLConfigBuilder的parse（）方法解析
4. XMLConfigBuilder对象返回 Configration对象
5. SqlSessionFactoryBuilder根据Configuration对象创建一个DefaultSessionFactory对象
6. SqlSessionFactoryBuilder返回DefaultSessionFactory对象给Client，供Client使用。
`
SqlSessionFactoryBuilder的构造方法
```java
public SqlSessionFactory build(InputStream inputStream)
{
    return build(inputStream, null, null);
}
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties)
{
    try
    {
        //2. 创建XMLConfigBuilder对象用来解析XML配置文件，生成Configuration对象
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        //3. 将XML配置文件内的信息解析成Java对象Configuration对象
        Configuration config = parser.parse();
        //4. 根据Configuration对象创建出SqlSessionFactory对象
        return build(config);
    }
    catch (Exception e)
    {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    }
    finally
    {
        ErrorContext.instance().reset();
        try
        {
            inputStream.close();
        }
        catch (IOException e)
        {
            // Intentionally ignore. Prefer previous error.
        }
    }
}

public SqlSessionFactory build(Configuration config)
{
    return new DefaultSqlSessionFactory(config);
}

```


