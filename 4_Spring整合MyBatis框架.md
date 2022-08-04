# 一、回顾普通Mabatis应用

> **项目结构**

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3zoiqh3woj30c00ktdiy.jpg)

> **pom.dependencies**

`````xml
 <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>



    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.6</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.29</version>
    </dependency>



  </dependencies>
`````



> **mybatis-config.xml**

`````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <package name="com.xxr.pojo"/>
    </typeAliases>

    <!--    配置数据库环境，设置默认的数据库连接-->
    <environments default="development">


        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis?userSSL=false"/>
                <property name="username" value="root"/>
                <property name="password" value="sll520"/>
            </dataSource>
        </environment>

        <environment id="remote">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://120.77.92.117:3306/Examination"/>
                <property name="username" value="dev"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>


    </environments>


    <mappers>
        <package name="com.xxr.mapper"/>
    </mappers>
</configuration>
`````

> **UserMapper.xml && UserMapper.interface**

``````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.xxr.mapper.UserMapper">
    <resultMap id="userResultMapper" type="user">
        <result property="name" column="username"/>
        <result property="pwd" column="password"/>
    </resultMap>
</mapper>
``````

```````java
public interface UserMapper {
    @Select("select * from mybatis.tb_user;")
    @ResultMap("userResultMapper")
    List<User> selectAll();

    @Select("select * from mybatis.tb_user where id = #{id};")
    @ResultMap("userResultMapper")
    User selectById(int id);
}
```````






> **SqlSessionFactoryUtils.java**

````````java
public class SqlSessionFactoryUtils {

    private static SqlSessionFactory sqlSessionFactory;
    static {
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }


    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }

}
````````




> **UserServiceImpl.java**

``````````java
public class UserServiceImpl implements UserService {
    private SqlSessionFactory sqlSessionFactory = SqlSessionFactoryUtils.getSqlSessionFactory();
    @Override
    public List<User> selectAll() {
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = userMapper.selectAll();
        sqlSession.close();
        return users;
    }
}
``````````

# 二、Spring整合MyBatis



[---> mybatis-spring 官方文档](https://mybatis.org/spring/zh/getting-started.html)

[【狂神说视频讲解】](https://www.bilibili.com/video/BV1WE411d7Dv?p=26&vd_source=fbab16b01174b8c671633151f543a4c7)



## 2.1、XML文件注入

> **项目结构**

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3zplgvjr3j30c60mlwir.jpg)

> **jdbc.properties**

`````properties
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
name=root
password=sll520
driverClassName=com.mysql.cj.jdbc.Driver
`````

> **pom.dependencies**

```xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>



    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.6</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.29</version>
    </dependency>

    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.3.20</version>
    </dependency>

    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.9</version>
    </dependency>


    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>2.0.7</version>
    </dependency>

  </dependencies>
```

> **daoContext.xml**   (即上图中的`spring-dao.xml`文件)

`````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:property-placeholder location="classpath:jdbc.properties"/>
    
<!--    获取dataSource-->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${driverClassName}"/>
        <property name="password" value="${password}"/>
        <property name="username" value="${name}"/>
        <property name="url" value="${url}"/>
    </bean>


<!--获取sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--绑定Mybatis配置文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <!--定位mapper配置文件-->
        <property name="mapperLocations" value="classpath:com/xxr/mapper/*.xml"/>
        <!--设置别名-->
        <property name="typeAliasesPackage" value="com.xxr.pojo"/>
    </bean>


<!--    获取SqlSession即SqlSessionTemplate-->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg index="0" ref="sqlSessionFactory"/>
    </bean>


</beans>
`````

> **UserServiceImpl.java**

````java
public class UserServiceImpl implements UserService {
    private SqlSessionTemplate sqlSession;


    public void setSqlSession(SqlSessionTemplate sqlSession) {
        this.sqlSession = sqlSession;
    }

    @Override
    public List<User> selectAll() {
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        return userMapper.selectAll();
    }

    @Override
    public User selectById(int id) {
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
        return userMapper.selectById(id);
    }
}
````

> **applicationContext.xml**

``````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

<!--    引入其他配置文件，例如dao配置文件-->
    <import resource="classpath:daoContext.xml"/>
    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl">
        <property name="sqlSession" ref="sqlSession"/>
    </bean>

</beans>
``````

> **mybatis-config.xml**

``````xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
``````

>  **测试**

````java
@Test
    public void test2(){
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService) context.getBean("userService");
        userService.selectAll().forEach(System.out::println);
        System.out.println();
        int id =3;
        User user = userService.selectById(3);
        System.out.println(user);
    }
````





> **与未整合时的对比**

![img](https://wx4.sinaimg.cn/mw2000/008rcJvVly1h3zprah6gyj31mp0t74qp.jpg)

(左图为MyBatis整合前的`mybatis-config.xml`，右图为整合后的`daoContext.xml`)







# 三、Spring声明式事务

![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3zuijwt2hj30o00ddtai.jpg)

在上述提供的`daoContext.xml`文件（即图中的`spring-dao.xml`文件）中添加以下片段即可：

`````xml
<!--    配置声明式事务-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<!--        <constructor-arg ref="dataSource"/>-->
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    结合AOP实现事务的注入-->

    <!-- 配置事务通知-->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>

    </tx:advice>


    <!-- 配置事务切入-->
    <aop:config>
        <aop:pointcut id="txPointCut" expression="execution(* com.xxr.service.impl.UserServiceImpl.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
    </aop:config>
`````

保证事务安全、稳定。

`<tx:method name="*" propagation="REQUIRED"/>`的name就是方法名，在这里是UserService的要增强的方法，配置事务管理里面的方法一定要与`UserService`接口中的方法一样，要么你就用*,要么就保证方法名一致。
