---
title: spring-boot-ioc 解析
date: 2019/5/22/
categories:
  - springboot
tags:
  - ioc
comments: true
abbrlink: 49197
img:
---

spring boot ioc 启动调用堆栈

```java 
inject:177, InjectionMetadata$InjectedElement (org.springframework.beans.factory.annotation)
inject:90, InjectionMetadata (org.springframework.beans.factory.annotation)
postProcessProperties:321, CommonAnnotationBeanPostProcessor (org.springframework.context.annotation)
populateBean:1395, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
doCreateBean:592, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
createBean:515, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
lambda$doGetBean$0:320, AbstractBeanFactory (org.springframework.beans.factory.support)
getObject:-1, 2018260103 (org.springframework.beans.factory.support.AbstractBeanFactory$$Lambda$144)
getSingleton:222, DefaultSingletonBeanRegistry (org.springframework.beans.factory.support)
doGetBean:318, AbstractBeanFactory (org.springframework.beans.factory.support)
getBean:199, AbstractBeanFactory (org.springframework.beans.factory.support)
resolveCandidate:277, DependencyDescriptor (org.springframework.beans.factory.config)
doResolveDependency:1247, DefaultListableBeanFactory (org.springframework.beans.factory.support)
resolveDependency:1167, DefaultListableBeanFactory (org.springframework.beans.factory.support)
inject:593, AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement (org.springframework.beans.factory.annotation)
inject:90, InjectionMetadata (org.springframework.beans.factory.annotation)
postProcessProperties:374, AutowiredAnnotationBeanPostProcessor (org.springframework.beans.factory.annotation)
populateBean:1395, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
doCreateBean:592, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
createBean:515, AbstractAutowireCapableBeanFactory (org.springframework.beans.factory.support)
lambda$doGetBean$0:320, AbstractBeanFactory (org.springframework.beans.factory.support)
getObject:-1, 2018260103 (org.springframework.beans.factory.support.AbstractBeanFactory$$Lambda$144)
getSingleton:222, DefaultSingletonBeanRegistry (org.springframework.beans.factory.support)
doGetBean:318, AbstractBeanFactory (org.springframework.beans.factory.support)
getBean:199, AbstractBeanFactory (org.springframework.beans.factory.support)
preInstantiateSingletons:849, DefaultListableBeanFactory (org.springframework.beans.factory.support)
finishBeanFactoryInitialization:877, AbstractApplicationContext (org.springframework.context.support)
refresh:549, AbstractApplicationContext (org.springframework.context.support)
refresh:142, ServletWebServerApplicationContext (org.springframework.boot.web.servlet.context)
refresh:775, SpringApplication (org.springframework.boot)
refreshContext:397, SpringApplication (org.springframework.boot)
run:316, SpringApplication (org.springframework.boot)
run:1260, SpringApplication (org.springframework.boot)
run:1248, SpringApplication (org.springframework.boot)
main:26, ZhxtApplication (com.zwkj.zhxt)
```


AbstractAutowireCapableBeanFactory 320 
```java
@Override
	public Object configureBean(Object existingBean, String beanName) throws BeansException {
		markBeanAsCreated(beanName);
		BeanDefinition mbd = getMergedBeanDefinition(beanName);
		RootBeanDefinition bd = null;
		if (mbd instanceof RootBeanDefinition) {
			RootBeanDefinition rbd = (RootBeanDefinition) mbd;
			bd = (rbd.isPrototype() ? rbd : rbd.cloneBeanDefinition());
		}
		if (bd == null) {
			bd = new RootBeanDefinition(mbd);
		}
		if (!bd.isPrototype()) {
			bd.setScope(BeanDefinition.SCOPE_PROTOTYPE);
			bd.allowCaching = ClassUtils.isCacheSafe(ClassUtils.getUserClass(existingBean), getBeanClassLoader());
		}
		BeanWrapper bw = new BeanWrapperImpl(existingBean);
		initBeanWrapper(bw);
		populateBean(beanName, bd, bw);
		return initializeBean(beanName, existingBean, bd);
	}
```

AbstractAutowireCapableBeanFactory 1336 

```java

		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME || mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
			if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (mbd.getResolvedAutowireMode() == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}

		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}

```






