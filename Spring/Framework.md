# 1. Framework

## 1.1. The IoC Container(IoC 容器)

### 1.1.1. IoC Container and Beans 的介绍

**IoC** :控制反转又叫依赖注入（**DI**），是对象定义其依赖的过程，可通过构造器参数、 工厂方法参数、或是在该对象的属性，容器在创建这些Bean时注入这些依赖。

org.springframework.beans 和 org.springframework.context 包，是Spring Framework 的基础**IoC**容器

**Beans**: 在应用中受IoC容器管理的对象

### 1.1.2. 容器总览

org.springframework.context.ApplicationContext 接口，代表着Spring IoC 容器，其责任是根据配置的元数据，实例化、配置、和组装Bean，配置元数据可通过XML、Java annotations,或是Java code。

ApplicationContext为BeanFactory 的子接口，BeanFactory提供了一种配置机制，能管理任何类型的对象

#### 1.1.2.1. 配置元数据

**Java-based configuration**

* **@Bean**: @Bean注解表明该用于实例化、配置和初始化对象的方法由Spring IoC 容器管理

* **@Configuration** ：@Configuration注解主要用于表明这是一个定义bean的资源文件，在该文件当中可定义bean间依赖


#### 1.1.2.2. 实例化Spring 容器

使用**AnnotationConfigApplicationContext** 可实例化Spring 容器

通过构造方法将配置类传入
```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

使用无参构造方法实例化，通过register方法传入配置类

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
通过组件扫描传入配置类

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}

public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```
蚌埠
#### 1.1.2.3. 使用@Bean 注解

* @Bean 是方法级别的注解，可用在@Configuration注释或@Componet注释过的类中的方法，注释的方法，实例化对象并且返回对像

* Bean创建时所需的依赖：通过方法的参数传入，传入参数自动实例化，如下accountRepository会自动实例化

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

* 生命周期回调：对象创建和毁灭时调用的方法，init-method和destroyMethod，如果Bean类中有public的close或shutdown方法，容器关闭时会自动调用bean中以close或shutdown命名的方法，不想被自动调用的话，设@Bean(destroyMethod="")

```java
public class BeanOne {
蚌埠
    public void init() {
        // initialization logic
    }
}
public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}
@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

* 指定Bean的 Scope（范围）：通过@Scope ，eg. @Scope("prototype")

* 自定义Bean的名字：@Bean(name = "myThing")

* Bean的别名：给一个Bean多个名字，eg. @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})

* Bean描述: @Description("Provides a basic example of a bean")


#### 1.1.2.4. 使用@Configuartion

@Configuration 是提供类级别的注释

* 注入Bean之间的依赖

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo()); // beanTwo被直接实例化并注入
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```
* Lookup Method Injection

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
通过 Java Configuration，可以生成CommandManager的子类，并重写createCommand()方法

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}
@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```


### 1.1.3. 依赖

#### 1.1.3.1. 依赖注入

* 基于构造器的依赖注入： 容器调用带参数的构造器，完成依赖注入

* 基于Setter的依赖注入：容器调用无参构造器实例化bean后，调用setter方法完成依赖注入

Spring team 比较主张构造器注入，这可让对象不可变，和确定依赖非空

setter方法注入的优点是，能让对象在合适的情况下，重新配置或重新被依赖注入

#### 1.1.3.2. Lazy-initalized Beans

Bean 默认是急促实例化且为单例的，通过@Lazy或xml lazy-init="true"可使该bean在请求时才被实例化。

```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

#### 1.1.3.3. Method Injection

当一个单例对象依赖一个非单例对象时，每次获得单例对象里面的非单例不会更新，利用Method Injection 可解决

不再使用以来注入，而是实现BeanFactoryAware接口之后每一次通过getBean的方法获取一个新的实例B，但是这通常不是一个理想的解决方案，因为bean代码耦合到Spring中。

* Lookup Method injection

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
```java
public abstract class UserService {
    public abstract WalletService createWalletService() ;
    public UserService() {
        System.out.println("UserService 正在实例化!");
    }
    public void login(String userName, String passWord) {
        createWalletService().run();
        System.out.println(userName + "正在登陆！");
    }
}
```

```java
public  class WalletService {
    public WalletService(){
        System.out.println("CarService");
    }
    public void run() {
        System.out.println("this = " + this);
    }
}
```

```xml
<bean id="walletService" class="com.daxin.service.WalletService" singleton="false"/>
<bean id="userService" class="com.daxin.service.UserService">
    <lookup-method name="createWalletService" bean="walletService"></lookup-method>
 </bean>
```
测试

```java
public static void main(String[] args) throws CloneNotSupportedException {
    ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
    for (int i = 0; i < 5; i++) {
        UserService userService = (UserService) ctx.getBean("userService");
        userService.login("daxin", "root");
    }
    ctx.close();
}
daxin正在登陆！
CarService
this = com.daxin.service.WalletService@40e6dfe1
daxin正在登陆！
CarService
this = com.daxin.service.WalletService@1b083826
daxin正在登陆！
CarService
this = com.daxin.service.WalletService@105fece7
daxin正在登陆！
CarService
this = com.daxin.service.WalletService@3ec300f1
daxin正在登陆
```

通过@Lookup

```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        MyCommand command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup
    protected abstract MyCommand createCommand();
}
```

### 1.1.4. Bean Scopes

* singleton ：默认的Scopes，对在Spring IoC 容器中单例对象的定义

* protoype ： 该定义下的对象，可在容器中存在多个实例

* request ： 定义了单个HTTP请求的生命周期，每一个HTTP请求都有属于它的一个实例

* session ： HTTP会话

* application ： 定义ServletContext的生命周期

* websocket ： 定义websocket的生命周期


## 1.2. Aspect Oriented Programming with Spring (AOP)

AOP 是 OOP的补充，OOP模块化的关键单元是类，AOP模块化的单元是切面。AOP为Spring IoC提供了一个非常有效的中间件解决方案

### 1.2.1. AOP概念

* Aspect：关注涉及多个类的模块化，事物管理就是很好的例子，一个aspect 可通过一个常规的类并通过xml配置（schema-based approach）实现，或通过@AspectJ注解的类实现

* Join point:  程序运行过程中的一点，一个连接点总是代表一个方法的运行

* Advice：Aspect在特定的连接点上采取的操作，许多AOP 框架将Advice建模为拦截器，并且在Join point周围维护了一条拦截器连

* Pointcut：匹配多个连接点（方法）并抽象成一个方法，供Advice调用

* Introduction： 对于一个已有的类引入新的接口，将当前对象转型成另一对象时，就可调用另一对象的方法

Spring AOP 的 advice

* Before advice ：在join point 之前运行
* After returning advice： 在join point 运行结束后运行
* After throwing advice：当该join point运行时抛出异常运行
* After advice ： 不管是正常退出还是抛出异常后都运行
* Around advice：join point 运行前运行一次，结束也运行一次



