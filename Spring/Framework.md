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







