# 一、Spring配置数据源

## 1.1、数据源（连接池）的作用

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3w40c8yxtj30lk087ta8.jpg)

> 常见数据源：DBCP、C3P0、Druid等

## 1.2、数据源的开发步骤

![img](https://wx4.sinaimg.cn/mw2000/008rcJvVly1h3w437ual7j30hk08mta6.jpg)

## 1.3、案例



> 获取Druid连接池

`druid.properties`

`````properties
url=jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true
name=root
password=sll520
driverClassName=com.mysql.cj.jdbc.Driver

initialSize=10
maxActive=20
maxWait=1000
filters=wall
`````

`Test.java`

```````java
//硬编码   
@Test
    public void testDruid() throws SQLException {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUsername("root");
        dataSource.setPassword("sll520");
//        dataSource.setInitialSize(8);
        DruidPooledConnection connection = dataSource.getConnection();
        System.out.println(connection);
    }

//加载properties文件
    @Test
    public void testDruid1() throws Exception {
        Properties pro = new Properties();
        pro.load(this.getClass().getClassLoader().getResourceAsStream("druid.properties"));
        DataSource ds = DruidDataSourceFactory.createDataSource(pro);
        Connection conn = ds.getConnection();
        System.out.println(conn);
    }
```````





> 获取C3P0连接池

`c3p0.properties`

````properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test
name=root
password=sll520
````

`c3p0-config.xml`

``````xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <named-config name="helloc3p0">
        <!-- 获取连接的4个基本信息 -->
        <property name="user">root</property>
        <property name="password">sll520</property>
        <property name="jdbcUrl">jdbc:mysql:///test</property>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>

        <!-- 涉及到数据库连接池的管理的相关属性的设置 -->
        <!-- 若数据库中连接数不足时, 一次向数据库服务器申请多少个连接 -->
        <property name="acquireIncrement">5</property>
        <!-- 初始化数据库连接池时连接的数量 -->
        <property name="initialPoolSize">5</property>
        <!-- 数据库连接池中的最小的数据库连接数 -->
        <property name="minPoolSize">5</property>
        <!-- 数据库连接池中的最大的数据库连接数 -->
        <property name="maxPoolSize">10</property>
        <!-- C3P0 数据库连接池可以维护的 Statement 的个数 -->
        <property name="maxStatements">20</property>
        <!-- 每个连接同时可以使用的 Statement 对象的个数 -->
        <property name="maxStatementsPerConnection">5</property>

    </named-config>
</c3p0-config>
``````

`Test.java`

```````java
//硬编码
@Test
    public void testC3p0() throws Exception {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setUser("root");
        dataSource.setPassword("sll520");
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }

//加载c3p0.properties
    @Test
    public void testC3p01() throws Exception {
        ResourceBundle bundle = ResourceBundle.getBundle("c3p0");
        String driver = bundle.getString("driver");
        String url = bundle.getString("url");
        String username = bundle.getString("name");
        String password = bundle.getString("password");
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        Connection connection = dataSource.getConnection();
        System.out.println(connection);

    }

//加载c3p0-config.xml
    @Test
    public void testC3p02() throws Exception {
        ComboPooledDataSource dataSource = new ComboPooledDataSource("helloc3p0");
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }
```````





-----

**用Spring容器获取dataSource**

----

> **c3p0连接池**

**`applicationContext.xml`**

`````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:c3p0.properties"/>
    
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${driver}"/>
        <property name="jdbcUrl" value="${url}"/>
<!--        用spel表达式时，${username}会产生歧义以为是计算机系统用户名-->
        <property name="user" value="${name}"/>
        <property name="password" value="${password}"/>
    </bean>

</beans>
`````

**其中`c3p0.properties`**

`````properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test
name=root
password=sll520
`````

**测试：**

`````java
    @Test
    public void testC3p03() throws Exception {
        //通过Spring容器实例化对象
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        ComboPooledDataSource bean = app.getBean(ComboPooledDataSource.class);
        System.out.println(bean.getConnection());
    }
`````



---

> **Druid连接池**

**`applicationContext.xml`**

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:druid.properties"/>
    
     <bean id="dataSource1" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${driverClassName}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${name}"/>
        <property name="password" value="${password}"/>
    </bean>

</beans>
````

**其中`druid.properties`**

````properties
url=jdbc:mysql://localhost:3306/test
name=root
password=sll520
driverClassName=com.mysql.cj.jdbc.Driver

initialSize=10
maxActive=20
maxWait=1000
filters=wall
````

**测试：**

````java
    @Test
    public void testDruid3() throws Exception {
        //通过Spring容器实例化对象
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        DruidDataSource dataSource = app.getBean(DruidDataSource.class);
        System.out.println(dataSource.getConnection());
    }
````







# 二、Spring注解开发

## 2.1、原始注解

![img](https://wx4.sinaimg.cn/mw2000/008rcJvVly1h3x0e1vcmgj30pm0eu7aq.jpg)



> **UserDaoImpl类**

``````java
@Component("userDao")
public class UserDaoImpl implements UserDao {
    @Override
    public void testMethod() {
        System.out.println("testMethod... ");
    }
}
``````

> **UsrServiceImpl类**

```java
@Component("userService")
public class UserServiceImpl implements UserService {
    @Value("${str}") //el表达式加载test.properties中的键值对进行值注入
    String str;

//    方法一：@Autowired  根据数据类型从容器中进行匹配

    //方法二：
//    @Autowired
//    @Qualifier("userDao")  根据id值从容器中进行匹配，但必须与@Autowired结合使用

    //方法三：
    @Resource(name = "userDao")  //@Autowired + @Qualifier("userDao")
    private UserDao userDao;
    
//	可省略
//    public void setUserDao(UserDao userDao) {
//        this.userDao = userDao;
//    }


    @PostConstruct
    public void init(){
        System.out.println("init... ");
    }


    @Override
    public void testMethod() {
        System.out.println(str);
        userDao.testMethod();
    }


    @PreDestroy
    public void destroy(){
        System.out.println("destroy...");
    }
}
```

> **test.properties**

```````properties
str=testStr
```````

> **applicationContext.xml**

**组件扫描和properties文件加载仍然依赖配置文件**

``````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

<!--    component-scan-->
    <context:component-scan base-package="com.xxr"/>

<!--    load properties-->
    <context:property-placeholder location="classpath:test.properties"/>
</beans>
``````



## 2.2、完全注解开发

1. **原始注解的局限性：**

   ![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3x0hy7h5vj30ko086dhx.jpg)

2. **新注解**

   ![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3x0i03fj2j30jb093772.jpg)

3. **示例**

   > **目录结构**

   ![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3x3p8u116j30dn0lm0x3.jpg)

   > **Configuration包**

   `1、DataSourceConfiguration类`

   `````````java
   //标记是配置文件
   @Configuration
   //<context:component-scan base-package="com.xxr"/>
   @ComponentScan("com.xxr")
   //<context:property-placeholder location="classpath:c3p0.properties"/>
   //@PropertySource("classpath:c3p0.properties")
   @PropertySources({
           @PropertySource("classpath:c3p0.properties"),
           @PropertySource("classpath:test.properties")
   })
   public class DataSourceConfiguration {
       @Value("${driver}")
       private String driver;
       @Value("${url}")
       private String url;
       @Value("${name}")
       private String name;
       @Value("${password}")
       private String password;
   
       @Bean("dataSource")  //Spring会将当前方法的返回值以指定名称存储到Spring容器中
       public DataSource getDataSource() throws PropertyVetoException {
           System.out.println(driver+"\t"+url+"\t"+name+"\t"+password);
           ComboPooledDataSource dataSource = new ComboPooledDataSource();
           dataSource.setDriverClass(driver);
           dataSource.setJdbcUrl(url);
           dataSource.setUser(name);
           dataSource.setPassword(password);
           return dataSource;
       }
   }
   `````````

   `2、SpringConfiguration类`

   `````````java
   @Configuration
   //<context:component-scan base-package="com.xxr"/>
   @ComponentScan("com.xxr")
   //<import resource = ""/>
   @Import({DataSourceConfiguration.class})
   public class SpringConfiguration {
   
   }
   `````````

   > **测试类**

   `````java
       @Test
       public void test2() throws SQLException {
           ApplicationContext app = new AnnotationConfigApplicationContext(SpringConfiguration.class);
           UserService userService = app.getBean(UserService.class);
   //        DataSource dataSource = (DataSource) app.getBean("dataSource");
   //        System.out.println(dataSource);
   //        System.out.println(dataSource.getConnection());
           DataSource dataSource = app.getBean(DataSource.class);
           System.out.println(dataSource);
           System.out.println(dataSource.getConnection());
           userService.testMethod();
       }
   `````

   > **配置文件**

   ````properties
   # c3p0.properties
   driver=com.mysql.cj.jdbc.Driver
   url=jdbc:mysql://localhost:3306/test
   name=root
   password=sll520
   ````

4. 注记

   * 标志是配置类

     `````java
     @Configuration
     `````

   * 扫描注解所在的包

     ````java
     //<context:component-scan base-package="com.xxr"/>
     @ComponentScan("com.xxr")
     ````

   * 加载配置`.properties`文件

     `````````````java
     //<context:property-placeholder location="classpath:c3p0.properties"/>
     //@PropertySource("classpath:c3p0.properties")
     @PropertySources({
             @PropertySource("classpath:c3p0.properties"),
             @PropertySource("classpath:test.properties")
     })
     `````````````

   * 核心配置类导入其他配置类

     `````java
     //<import resource = ""/>
     @Import({DataSourceConfiguration.class})
     `````

   * 使用

     ````java
         @Test
         public void test2() throws SQLException {
             ApplicationContext app = new AnnotationConfigApplicationContext(SpringConfiguration.class);
             UserService userService = app.getBean(UserService.class);
     //        DataSource dataSource = (DataSource) app.getBean("dataSource");
     //        System.out.println(dataSource);
     //        System.out.println(dataSource.getConnection());
             DataSource dataSource = app.getBean(DataSource.class);
             System.out.println(dataSource);
             System.out.println(dataSource.getConnection());
             userService.testMethod();
         }
     ````



## 2.3、Spring集成Junit环境

`````````java
@RunWith(SpringJUnit4ClassRunner.class)
//@ContextConfiguration("classpath:applicationContext.xml")
@ContextConfiguration(classes = {SpringConfiguration.class})
public class SpringJunitTest {


    @Autowired
    public UserService userService;

    @Autowired
    public DataSource dataSource;

    @Test
    public void test1() throws SQLException {
        userService.testMethod();
        System.out.println(dataSource.getConnection());
    }
}
`````````

`注意：Junit应该不低于4.1.12版本`

 













