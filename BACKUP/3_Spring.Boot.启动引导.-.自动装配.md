# [Spring Boot 启动引导 - 自动装配](https://github.com/EruDev/blog/issues/3)

# Spring Boot 入门程序原理概述和包扫描
> 掘金小册《SpringBoot 源码解读与原理分析》笔记
##  ch03
1. `@SpringBootApplication` 是组合注解；
2. `@ComponScan` 默认扫描当前配置类所在包以及包下所有组件，`exclude` 属性会将主启动类、自动配置类屏蔽掉；
3. `@Configuration` 可标注配置类，`SpringBootConfiguration ` 并没有对其做实质性扩展