# [SpringBoot 源码解读与原理分析小册笔记](https://github.com/EruDev/blog/issues/3)

# Spring Boot 入门程序原理概述和包扫描
> 掘金小册《SpringBoot 源码解读与原理分析》笔记
##  ch03
1. `@SpringBootApplication` 是组合注解；
2. `@ComponScan` 默认扫描当前配置类所在包以及包下所有组件，`exclude` 属性会将主启动类、自动配置类屏蔽掉；
3. `@Configuration` 可标注配置类，`SpringBootConfiguration ` 并没有对其做实质性扩展

# Spring Boot 自动装配
## ch04-ch05
AutoConfigurationPackages.Registrar
```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        register(registry, new PackageImport(metadata).getPackageName());
    }

    @Override
    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new PackageImport(metadata));
    }

}
```
> ImportBeanDefinitionRegistrar 用于保存导入的配置类所在的根包

`AnnotationMetadata` 类的注解元数据，个人理解其实就是对类的包装

AutoConfigurationPackages#register
```java
public static void register(BeanDefinitionRegistry registry, String... packageNames) {
  // 判断 BeanFactory 中是否包含 AutoConfigurationPackages
  if (registry.containsBeanDefinition(BEAN)) {
	  BeanDefinition beanDefinition = registry.getBeanDefinition(BEAN);
	  ConstructorArgumentValues constructorArguments = beanDefinition.getConstructorArgumentValues();
          // addBasePackages：添加根包扫描包
	  constructorArguments.addIndexedArgumentValue(0, addBasePackages(constructorArguments, packageNames));
  }
  else {
	  GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
	  beanDefinition.setBeanClass(BasePackages.class);
	  beanDefinition.getConstructorArgumentValues().addIndexedArgumentValue(0, packageNames);
	  beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
	  registry.registerBeanDefinition(BEAN, beanDefinition);
  }
}
```
小结：
1. SpringFramework 提供了模式注解、@EnableXXX + @Import 的组合手动装配；
2. `@SpringBootApplication` 标注的主启动类所在包会被视为扫描包的根包

## Java 的 SPI
> SPI 全称 Service Provider Interface，是 JDK 内置的一种服务提供发现机制。简单来说，它就是一种动态替换发现的机制。

SPI规定，所有要预先声明的类都应该放在 META-INF/services 中。配置的文件名是接口/抽象类的全限定名，文件内容是抽象类的子类或接口的实现类的全限定类名，如果有多个，借助换行符，一行一个。

SpringFramework的SpringFactoriesLoader

1. `AutoConfigurationImportSelector` 配合 `SpringFactoriesLoader` 可加载 “META-INF/spring.factories” 中配置的 `@EnableAutoConfiguration` 对应的自动配置类。
`DeferredImportSelector` 的执行时机比 `ImportSelector` 更晚。
SpringFramework 实现了自己的SPI技术，相比较于Java原生的SPI更灵活。

# Spring Boot IOC

## Spring Boot 准备 IOC 容器
```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        this.resourceLoader = resourceLoader;
        Assert.notNull(primarySources, "PrimarySources must not be null");
        this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
        this.webApplicationType = WebApplicationType.deduceFromClasspath();
        // 设置初始化器
        setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
        // 设置监听器
        setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        this.mainApplicationClass = deduceMainApplicationClass();
}
```
1. `SpringApplication` 的创建和运行是两个不同的步骤，上面代码是创建。
2. Spring Boot 会根据当前 classpath 下的类来决定应用类型。
3. Spring Boot 中两个关键的组件：`ApplicationContextInitializer` 和 `ApplicationListener `，分别是初始化器和监听器，它们都在 构建 `SpringApplication` 时注册。

【至此，`SpringApplication` 初始化完成，下面会真正启动 SpringApplication】

## IOC: 准备运行时环境
![code](https://raw.githubusercontent.com/EruDev/md-picture/master/img/1624255046.png)

```java
public ConfigurableApplicationContext run(String... args) {
    // 4.1 创建StopWatch对象
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 4.2 创建空的IOC容器，和一组异常报告器
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    // 4.3 配置与awt相关的信息
    configureHeadlessProperty();
    // 4.4 获取SpringApplicationRunListeners，并调用starting方法（回调机制）
    SpringApplicationRunListeners listeners = getRunListeners(args);
    // 【回调】首次启动run方法时立即调用。可用于非常早期的初始化（准备运行时环境之前）。
    listeners.starting();
    try {
        // 将main方法的args参数封装到一个对象中
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        // 4.5 准备运行时环境
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        //.............
    }
```
**Environment**
> 它是 IOC 容器的运行环境，它包括 Profile 和 Properties 两大部分，它可由一个到几个激活的 Profile 共同配置，它的配置可在应用级 Bean 中获取。

可以这样理解：
![Environment](https://raw.githubusercontent.com/EruDev/md-picture/master/img/1624258459.png)

1. SpringApplication 应用中可以使用 `SpringApplicationRunListener` 来监听 SpringBoot 应用的启动过程。
2. 在创建 IOC 容器前，SpringApplication 会准备运行时环境 `Environment`