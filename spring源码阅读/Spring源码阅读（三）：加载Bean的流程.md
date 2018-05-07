# Spring源码阅读（三）：加载Bean的流程

上一篇文章主阅读了Spring解析XML文件的流程。当XML解析完成之后，Spring就需要获取指定名称的Bean对象。那么，Spring是怎么获取Bean对象的呢？ 我们可以看看源码的具体实现。



## 一、源码流程

Spring中的Bean容器可以说就是BeanFactory，BeanFactory有被封装到ApplicationContext中。所以，我们从ApplicationContext中找到BeanFactory的痕迹，也就找到了Bean加载的流程。

在Spring的AbstractRefreshableApplicationContext类中有如下方法：

```
@Override
	protected final void refreshBeanFactory() throws BeansException {
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		try {
			//创建BeanFactory对象。
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
			//加载并解析XML配置文件。BeanFactory作为Bean容器传入解析方法中，便于存放解析后的对象。
			loadBeanDefinitions(beanFactory);
			//Spring初始化过程中是线程安全的
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
```

此方法中创建了BeanFactory，并且将BeanFactory传入loadBeanDefinitions方法中，loadBeanDefinitions方法是用来解析XML配置文档。  BeanFactory是用来存放SpringBean对象的容器，BeanFactory的类图如下：

![018050716280](https://github.com/yangjingwen2/javen666.com/blob/master/spring%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB/images/20180507162807.png?raw=true)



如上图，AbstractBeanFactory类中的getBean方法和doGetBean方法就是Spring中加载Bean的核心方法。其中doGetBean方法内容如下：

```
protected <T> T doGetBean(
			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
			throws BeansException {
         //获取Bean的名称
		final String beanName = transformedBeanName(name);
		Object bean;
       
         //从缓存中获取Bean对象
         //此方法用来解决Spring对象中的依赖对象问题，比如循坏依赖问题。
		// Eagerly check singleton cache for manually registered singletons.
		Object sharedInstance = getSingleton(beanName);
		if (sharedInstance != null && args == null) {
			//返回对应的Bean实例
			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
		}

		else {
			// Fail if we're already creating this bean instance:
			// We're assumably within a circular reference.
			if (isPrototypeCurrentlyInCreation(beanName)) {
				throw new BeanCurrentlyInCreationException(beanName);
			}
			...
			...
		}
		return (T) bean;
	}
```

其中用来解决循环依赖的代码：

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return (singletonObject != NULL_OBJECT ? singletonObject : null);
	}
```

以上代码是解决循环依赖的关键代码。

在各个BeanFactory的子类中，用来缓存Bean对象的容器是ConcurrentHashmap。这样保证了Spring的线程安全。

## 二、总结

1、Spring Bean加载的过程比较复杂。一篇文章不可能将Bean的加载讲解得非常细致。本篇文章在于给大家阅读源码得过程中能找到关键得代码。通过对关键代码的阅读，能有所收获。

2、Spring Bean的创建使用了反射机制。

