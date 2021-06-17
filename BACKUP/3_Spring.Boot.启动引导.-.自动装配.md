# [Spring Boot 启动引导 - 自动装配](https://github.com/EruDev/blog/issues/3)

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