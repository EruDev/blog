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