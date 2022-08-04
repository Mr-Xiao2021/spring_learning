# 一、Spring的快速入门

[黑马程序员最全SSM框架教程|Spring+SpringMVC+MyBatis全套教程(spring+springmvc+mybatis)](https://www.bilibili.com/video/BV1WZ4y1P7Bp?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click)

* **大致步骤**

  ![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3uq18zwsfj30qv0eatcj.jpg)

  ![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3uq4menwdj30ih0g2jtd.jpg)


* **具体代码**

  `·applicationContext.xml`

  `````xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" scope="prototype"></bean>
  </beans>
  `````

  `UserdaoTest测试类`

  `````java
  public class UserDaoTest {
      @Test
      public void test1(){
          ClassPathXmlApplicationContext app =
                  new ClassPathXmlApplicationContext("applicationContext.xml");
          //利用反射调用空参构造器创建对象意味着该类一定要提供一个无参构造其器
          UserDao userDao1 = (UserDao) app.getBean("userDao");
          UserDao userDao2 = (UserDao) app.getBean("userDao");
          userDao1.testMethod();
          System.out.println("is the same one ? "+(userDao1==userDao2));
      }
  }
  `````

  

  ![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3ur2pz8wyj31m10ko4fh.jpg)

* **注意**

  注意在创建`applicationContext.xml`文件时的细节：

  ![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3uqgeht08j30l906cwgi.jpg)

  选项如果没有SpringConfig，就右击你的module选第二个add..下拉找到Spring，选SpringMVC点OK就完事了

  ![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3uqj0lpsgj30l006odi4.jpg)



# 二、Spring配置文件

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3w3iow090j30qb0d840s.jpg)

## 2.1、scope

**指对象的作用范围**



![img](https://wx4.sinaimg.cn/mw2000/008rcJvVly1h3ur4ykm1vj30x90fktfb.jpg)



````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans ...>
    <bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" scope="singleton"></bean>
</beans>
````

* **singleton 单例模式**（默认配置）

  ![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3ur7u3hh4j30qn0hv46v.jpg)

* **prototype 多例模式**

  ![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3ura6n1d3j30rd0iiakd.jpg)





## 2.2、Bean生命周期配置

* `init-method`：指定类中初始化方法的名称
* `destroy-method`：指定类中销毁方法的名称

`````xml
<bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" init-method="init" destroy-method="destroy"></bean>
`````

`````java
public class UserDaoImpl implements UserDao {

    @Override
    public void testMethod() {
        System.out.println("testMethod...");
    }

    public UserDaoImpl() {
        System.out.println("created... ");
    }

    public void init(){
        System.out.println("init... ");
    }
    public void destroy(){
        System.out.println("destroy... ");
    }
}

`````

**Result:**

![img](https://wx4.sinaimg.cn/mw2000/008rcJvVly1h3urrr3o5mj31iv0oke2v.jpg)





## 2.3、实例化三种方式

* 无参构造方法
* 工厂静态方法实例化
* 工厂实例方法实例化

**`UserDaoFactory`**

````java
public class UserDaoFactory {
    private static UserDao instance = null;
    public static UserDao getUserDao(){
        return new UserDaoImpl();
//        if(instance == null){
//            instance = new UserDaoImpl();
//        }
//        return instance;
    }


    public UserDao getNonStaticUserDao(){
        return new UserDaoImpl();
    }
}
````

**`applicationContext.xml`**

`````xml
<!--静态-->   
<bean id="userDao1" class="com.xxr.factory.UserDaoFactory" factory-method="getUserDao"></bean>

<!--非静态-->   
    <bean id="factory" class="com.xxr.factory.UserDaoFactory"></bean>
    <bean id="userDao3" factory-bean="factory" factory-method="getNonStaticUserDao"></bean>
`````

**测试类**

````java
@Test
    public void test2(){
        ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao) app.getBean("userDao1");
        UserDao userDao1 = (UserDao) app.getBean("userDao1");
        userDao.testMethod();
        //居然都是true，我不理解
        System.out.println((userDao == userDao1));

    }




    @Test
    public void test3(){
        ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao) app.getBean("userDao3");
        UserDao userDao1 = (UserDao) app.getBean("userDao3");
        System.out.println(userDao);
        System.out.println(userDao1);
        userDao.testMethod();
        //也是true...
        System.out.println((userDao == userDao1));
    }
````



# 三、Spring依赖注入



**针对Service中注入Dao，常规做法：**

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3uz6pthjzj30vc08udkj.jpg)

## 3.1、引用数据类型的注入

### 3.1.1、属性注入

**`UserServiceImpl.java`**

``````java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void testMethod() {
        userDao.testMethod();
    }
}
``````

**`applicationContext.xml`**

````xml
<bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>
    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
    </bean>
````

**`test`**

`````java
public class UserServiceTest {
    @Test
    public void test1(){
        ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService) app.getBean("userService");
        userService.testMethod();
    }
}
`````

**`Result`**

> UserDaoImpl created... 
> UserDaoImpl init... 
> testMethod...

### 3.1.2、构造器注入

**`UserServiceImpl.java`**

``````java
public class UserServiceImpl implements UserService {

    private UserDao userDao;

    public UserServiceImpl() {
    }

    public UserServiceImpl(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void testMethod() {
        userDao.testMethod();
    }
}
``````

**`applicationContext.xml`**

* 方式一：

````xml
    <bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>
    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl">
        <constructor-arg name="userDao" ref="userDao"/>
    </bean>
````

* 方式二：

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3uzzywtoaj311x0e2gry.jpg)

`````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" init-method="init" destroy-method="destroy"/>
    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>
</beans>
`````




**`test`**

`````java
public class UserServiceTest {
    @Test
    public void test1(){
        ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = (UserService) app.getBean("userService");
        userService.testMethod();
    }
}
`````

**`Result`**

> UserDaoImpl created... 
> UserDaoImpl init... 
> testMethod...



弹幕：

你拿到了对象spring容器会把实例通过构造器或者set方法提前给成员变量，你从容器中拿到的对象的成员变量是有实例的

xml中配置的是接口实现类，不是接口，sping加载创建实体类对象

controller的set方法注入service   必须在set方法上注解resource！！！







## 3.2、普通数据、集合类型的注入

案例：

* `UserDaoImpl`

  `````java
  public class UserDaoImpl implements UserDao {
      private String name;
      private int age;
      private List<String> stringList;
      private Map<String, User> userMap;
      private Properties properties;
      private List<User> userList;
  
      public void setUserList(List<User> userList) {
          this.userList = userList;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public void setAge(int age) {
          this.age = age;
      }
  
      public void setStringList(List<String> stringList) {
          this.stringList = stringList;
      }
  
      public void setUserMap(Map<String, User> userMap) {
          this.userMap = userMap;
      }
  
      public void setProperties(Properties properties) {
          this.properties = properties;
      }
  
      public UserDaoImpl() {
          System.out.println("UserDaoImpl created... ");
      }
  
      public void init(){
          System.out.println("UserDaoImpl init... ");
      }
  
      public void destroy(){
          System.out.println("destroy... ");
      }
  
      @Override
      public void testMethod() {
          System.out.println(name+"--->"+age);
          System.out.println(stringList);
          System.out.println(userMap);
          System.out.println(properties);
          System.out.println("testMethod...");
      }
  }
  `````

* `UserServiceImpl`

  ``````java
  public class UserServiceImpl implements UserService {
  
      private UserDao userDao;
  
      public void setUserDao(UserDao userDao) {
          this.userDao = userDao;
      }
  
      @Override
      public void testMethod() {
          userDao.testMethod();
      }
  }
  ``````

* `applicationContest.xml`

  ```````java
  <bean id="userDao" class="com.xxr.dao.impl.UserDaoImpl" init-method="init" destroy-method="destroy">
          <property name="name" value="xxr"/>
          <property name="age" value="19"/>
  
  
          <property name="stringList">
              <list>
                  <value>aaa</value>
                  <value>bbb</value>
                  <value>ccc</value>
                  <value>ddd</value>
              </list>
          </property>
  
          <property name="userMap">
              <map>
                  <entry key="u1" value-ref="user1"/>
                  <entry key="u2" value-ref="user2"/>
              </map>
          </property>
  
  
          <property name="properties">
              <props>
                  <prop key="p1">ppp1</prop>
                  <prop key="p2">ppp2</prop>
                  <prop key="p3">ppp3</prop>
              </props>
          </property>
  
      </bean>
      <bean id="userService" class="com.xxr.service.impl.UserServiceImpl" p:userDao-ref="userDao"/>
  
  <!--    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl">-->
  <!--        <property name="userDao" ref="userDao"/>-->
  <!--    </bean>-->
      <!--    <bean id="userService" class="com.xxr.service.impl.UserServiceImpl">-->
      <!--        <constructor-arg name="userDao" ref="userDao"/>-->
      <!--    </bean>-->
  
      <bean id="user1" class="com.xxr.domain.User">
          <property name="age" value="18"/>
          <property name="name" value="xxr"/>
      </bean>
      <bean id="user2" class="com.xxr.domain.User">
          <property name="age" value="16"/>
          <property name="name" value="tom"/>
      </bean>
  ```````

* `UserServiceTest`

  ```````java
  public class UserServiceTest {
      @Test
      public void test1(){
          ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
          UserService userService = (UserService) app.getBean("userService");
          userService.testMethod();
      }
  }
  ```````

## 3.3、配置文件的注入

`````xml
在applicationContext.xml中加上这句即可
<import resource="applicationContext-Xxx.xml"/>
`````

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3w3g9qsxpj30vw04qn0b.jpg)





# 四、Spring相关API

![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3w3opm1rbj30y20k37bm.jpg)



![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3w3p1u2s2j30qi09d42s.jpg)



![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3w3uilvwwj31090iwn3n.jpg)

````java
public class UserServiceTest {
    @Test
    public void test1(){
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
//        UserService userService = (UserService) app.getBean("userService");
        //xml中有两个同类型的bean就不行
        UserService userService = app.getBean(UserService.class);
        userService.testMethod();
    }
}
````

