---
title: spring
tags:
---

# Container Extension Points

* Customizing Beans by Using a BeanPostProcessor
Object postProcessBeforeInitialization(Object bean, String beanName) 作用域是容器级的
取到bean 可以取里面的Method Annotation 做特殊处理

<!-- more -->

* Customizing **Configuration Metadata** with a BeanFactoryPostProcessor
有个子接口BeanDefinitionRegistryPostProcessor  比方dubbo的Service注解的类要初始化，就是实现的这个接口
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException
**和BeanPostProcessor的区别，个人理解是强调configuration metadata的扩展**，拿到beanFactory基本上感觉什么都可以做了

* Customizing Instantiation Logic with a FactoryBean
复杂的实例初始化过程 可以复写这个类，比方dubbo reference这个注解，引用一个远程服务


# 没搞明白的
ApplicationContext implementations also permit the registration of existing objects that are created outside the container (by users).
	getBeanFactory() registerSingleton	registerBeanDefinition

