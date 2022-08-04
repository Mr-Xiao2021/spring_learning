# 一、概述

> 概念

![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3x97hobk0j30v107on2h.jpg)

> 底层原理：动态代理

**接口及实现类：**

![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3yha58u7sj30i808rtaw.jpg)

**动态代理：**

```java
public class DynamicProxyTest {
    public static void main(String[] args) {
        Dao dao = new DaoImpl();
        Dao daoInstance = (Dao) Proxy.newProxyInstance(DynamicProxyTest.class.getClassLoader(), dao.getClass().getInterfaces(), new MyInHandler(dao));
        int add = daoInstance.add(1, 2);
        System.out.println(add);
    }

}
class MyInHandler implements InvocationHandler{
    private Object obj;

    public MyInHandler(Object obj) {
        this.obj = obj;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法执行之前:  "+method.getName()+"("+ Arrays.toString(args)+")");

        Object resInvoke = method.invoke(obj, args);

        System.out.println("方法执行结束... "+resInvoke);

        return resInvoke;
    }
}
```



> AOP术语

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3yopvb1g2j30q909otdc.jpg)

* **连接点**

  类里面可增强的方法，这些方法即为连接点

* **切入点**

  实际被增强的方法即为切入点

* **通知（增强）**

  实际增强的逻辑部分即为通知；

  通知有多种类型：

  1. 前置通知
  2. 后置通知
  3. 环绕通知
  4. 异常通知
  5. 最终通知

* **切面（是动作）**

  1. 把通知应用到切入点的过程

![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3yhhszcs8j30zx0ej43q.jpg)







# 二、AOP操作

**推荐的dependency**

````java
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.9</version>
    </dependency>
````



权限修饰符可以不写，但是返回值类型为必填

权限修饰符默认是public  *代表返回类型可以是任意的







代理混用,也可以尝试在aopConfig标签下方添加<aop:aspectj-autoproxy  proxy-target-class="true"/>

如果是被代理类实现了接口用的是jdk代理，如果没有实现接口用的是cglib



动态代理根据接口字节码创建的对象 且该对象是接口类型的 所以要用接口类型接受





## 2.1、尚硅谷AOP讲解

### 2.1.1、注解开发

> **User类**

````java
@Component("user")
public class User {
    public void add(){
        System.out.println("\nadd method executing... \n");
    }
}
````

> **代理类之一: UserProxy**

``````java
@Component
@Aspect//表明切面类
@Order(0)
public class UserProxy {

    @Pointcut("execution(* com.xxr.aop.User.add(..))")
    public void pointDemo(){
        System.out.println("pointDemo... ");
    }
    @Before("pointDemo()")
    public void before(){
        System.out.println("before... ");
    }
    @After("execution(* com.xxr.aop.User.add(..))")
    public void after(){
        System.out.println("after... ");
    }
    @AfterReturning("execution(* com.xxr.aop.User.add(..))")
    public void afterReturning(){
        System.out.println("afterReturning... ");
    }
    @AfterThrowing("execution(* com.xxr.aop.User.add(..))")
    public void afterThrowing(){
        System.out.println("afterThrowing... ");
    }
    @Around("execution(* com.xxr.aop.User.add(..))")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("Before Around... ");
        proceedingJoinPoint.proceed();
        System.out.println("After Around... ");
    }
}
``````

> **代理类二: PersonProxy**

```````java
@Component
@Aspect
//代理的优先级，数值越小优先级越大
@Order(1)
public class PersonProxy {

    @Pointcut("execution(* com.xxr.aop.User.add(..))")
    public void pointDemo(){
        System.out.println("PersonProxy pointDemo... ");
    }
    @Before("pointDemo()")
    public void before(){
        System.out.println("PersonProxy before... ");
    }
}
```````

> **applicationContext.xml**

````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
<!--    开启注解扫描-->
    <context:component-scan base-package="com.xxr.aop"/>

<!--    开启Aspect生成代理对象-->
    <aop:aspectj-autoproxy/>


</beans>
````

> **pom.xml**

```````xml
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
      <version>2.7.0</version>
    </dependency>
```````



> **AopTest.java**

``````java
public class AopTest {
    @Test
    public void test1(){
        ApplicationContext app = new ClassPathXmlApplicationContext("applicationContext.xml");
        User user = app.getBean(User.class);
        user.add();
    }
}

``````

> **Output**

![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3yozhx76wj30lj06w75d.jpg)



### 2.1.2、配置文件开发

> **Book.java**

```````````java
public class Book {
    public void buy(){
        System.out.println("buy... ");
    }
}
```````````

> **BookProxy.java**

``````java
public class BookProxy {
    public void beforeBuy(){
        System.out.println("preparation for buy ... ");
    }

    public void afterBuy(){
        System.out.println("deliver after purchase... ");
    }
}
``````

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

    
    <bean id="book" class="com.xxr.xmlaop.Book"/>
    <bean id="bookProxy" class="com.xxr.xmlaop.BookProxy"/>
    
    <aop:config>
<!--        切入点-->
        <aop:pointcut id="pointcut" expression="execution(* com.xxr.xmlaop.Book.*(..))"/>
<!--        配置切面-->
        <aop:aspect ref="bookProxy">
            <aop:before method="beforeBuy" pointcut-ref="pointcut"/>
            <aop:after method="afterBuy" pointcut-ref="pointcut"/>
        </aop:aspect>
    </aop:config>


</beans>
``````









### 2.1.3、对应关系

> 1、@Aspect

标志此类为代理类，在`xml`文件中为

````xml
<aop:aspect ref="bookProxy">
            <aop method....../>
</aop:aspect>
````

> 2、@Pointcut("execution(切入点表达式)")

在注解中

````java
@Pointcut("execution(* com.xxr.aop.User.add(..))")
 public void pointDemo(){
     System.out.println("pointDemo... ");
}
````

对应`xml`文件

````xml
<aop:pointcut id="pointcut" expression="execution(* com.xxr.xmlaop.Book.*(..))"/>
````

> 3、标记切点并声明通知

注解中：

````````java
@Before("pointDemo()")
public void before(){
    System.out.println("before... ");
}
````````

对应`xml`文件

````````xml
<aop:pointcut id="pointcut" expression="execution(* com.xxr.xmlaop.Book.*(..))"/>
<!--        配置切面-->
<aop:aspect ref="bookProxy">
     <aop:before method="beforeBuy" pointcut-ref="pointcut"/>
     <aop:after method="afterBuy" pointcut-ref="pointcut"/>
</aop:aspect>
````````





![img](https://wx1.sinaimg.cn/mw2000/008rcJvVly1h3ypsq4hnwj31ce0kpke8.jpg)













## 2.2、狂神版AOP讲解

![img](https://wx2.sinaimg.cn/mw2000/008rcJvVly1h3yrfzxt3cj30f909a0u6.jpg)

![img](https://wx3.sinaimg.cn/mw2000/008rcJvVly1h3yrfx5f7rj319h0evao8.jpg)



### 2.2.1、AOP方式一

> **UserService和UserServiceImpl**

```````java
public interface UserService {
    void mainMethod();
}
public class UserServiceImpl implements UserService{
    @Override
    public void mainMethod() {
        System.out.println("method executing... ");
    }
}
```````

> **MyBeforeAdvice类**

``````java
public class MyBeforeAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println(o.getClass().getSimpleName()+"\t"+method.getName()+"("+ Arrays.toString(objects));
    }
}
``````

> **MyAfterReturnAdvice类**

```java
public class MyAfterReturnAdvice implements AfterReturningAdvice {
    @Override
    public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
        System.out.println("执行后：\t\t\t返回值："+o+"\t方法名："+method.getName()+"\t参数为："+ Arrays.toString(objects));
    }
}
```

> **applicationContext.xml**

```````````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                          http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                          http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    
    <bean id="userService" class="com.xxr.ksaop.UserServiceImpl"/>
    <bean id="before" class="com.xxr.ksaop.MyBeforeAdvice"/>
    <bean id="after" class="com.xxr.ksaop.MyAfterReturnAdvice"/>


    <aop:config>
        <aop:pointcut id="cut" expression="execution(* com.xxr.ksaop.UserServiceImpl.*(..))"/>
        <aop:advisor advice-ref="before" pointcut-ref="cut"/>
        <aop:advisor advice-ref="after" pointcut-ref="cut"/>
    </aop:config>


</beans>
```````````

> **Output**

```````````txt
UserServiceImpl	mainMethod([]
method executing... 
执行后：			返回值：null	方法名：mainMethod	参数为：[]
```````````



### 2.2.3、方法二

**同尚硅谷**
