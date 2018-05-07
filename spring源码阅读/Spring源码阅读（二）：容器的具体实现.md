# Spring源码阅读（二）：配置文件的解析过程

Spring Bean是Spring最核心的内容。Spring Bean更像是一个Bean工厂，这个Bean工厂就是Bean容器。使用过Spring的同学都知道，Spring通过XML文档对Bean进行定义。本篇文章就是要介绍Spring是如何解析XML配置文件。带着如下问题来阅读源码：

问题1：Spring是怎么获取Bean对象？

## 一、什么是Bean？Spring怎么获取Bean对象？

再阅读源码之前，我们需要回顾一下Spring的基本操作过程。

### 1.1、什么是Bean？

~~~
package com.qianfeng;

public class Student {
    
    private String name="zhangsan";

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
~~~

以上就是一个简单的Bean。是一个很普通的Java类。通常创建一个类的对象直接使用new关键字就行，但是在有Spring框架的项目中，我们是通过Spring来管理和创建Java对象。下面，我们看看Spring怎么获取Java Bean对象。

### 1.2、Spring获取Bean对象

- spring配置文档(spring.xml)

``` 
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
        
    <bean name="student" class="com.qianfeng.Student"></bean>

</beans>
```

以上配置文档就是Spring中Bean的声明方式。这样配置之后，就可以将Student类交给Spring来创建并获取对象了。

- 获取Bean的测试代码

```
@Test
    public void testCase1(){
        //初始化Spring
        ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        Student stu = (Student) applicationContext.getBean("student");
        String name = stu.getName();
        System.out.println(name);
    }
```

执行以上代码之后，会在控制台输出“zhangsan”。

以上就是一个简单的Spring案例，通过这个案例结合文章开头提出的两个问题，思考如下：

* 1、Spring是怎么创建的Student，代码applicationContext.getBean("student")直接能获取一个Student对象，他是怎么做到的？



## 二、源码分析

结合上面Spring的基本用法来看，Spring容器应该至少具备两个核心的功能：一个是xml解析的能力，另一个是反射生成对象的能力。结合这亮点，带着以上的疑问，接下来就是查看源码了。在查看源码的过程中可能会碰到一个问题：

``` 从入口开始追踪源码，可能会追踪到接口中，这是看不到具体的内部实现；这时，大家需要结合API文档，看看当前接口的实现类，找到实现类然后再继续追踪代码的具体实现。```



### 2.1、解析XML配置文件、加载Bean

看源码的过程中，会有很多类和接口。最好使用时序图和类图将关键的步骤绘制出来。我们先来看看Spring初始化Bean解析XML文档的时许图：

![pring加载bean的流](images\spring加载bean的流程.png)

如上图，Spring从ClasspathXMLApplicaltionContext类的初始化开始，一步步执行到XmlBeanDefinitionRader类。在XmlBeanDefinitionRader类中的解析XML的代码如下：

```
/**
	 * Actually load bean definitions from the specified XML file.
	 * 从指定的XML文档中加载Bean的定义。
	 * @param inputSource the SAX InputSource to read from SAX API中的一个类，用户辅助XML的解析。
	 * @param resource the resource descriptor for the XML file Spring配置文件XML的封装类。
	 * @return the number of bean definitions found 返回最后解析bean的个数。
	 * @throws BeanDefinitionStoreException in case of loading or parsing errors
	 * @see #doLoadDocument
	 * @see #registerBeanDefinitions
	 */
	protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
			throws BeanDefinitionStoreException {
		try {
			//DOM解析XML文件，并且获取到Document对象。PS：Document是DOM解析中的核心类。
			Document doc = doLoadDocument(inputSource, resource);
			//从document中解析bean
			return registerBeanDefinitions(doc, resource);
		}
		catch (BeanDefinitionStoreException ex) {
			throw ex;
		}
		catch (SAXParseException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"Line " + ex.getLineNumber() + " in XML document from " + resource + " is invalid", ex);
		}
		catch (SAXException ex) {
			throw new XmlBeanDefinitionStoreException(resource.getDescription(),
					"XML document from " + resource + " is invalid", ex);
		}
		catch (ParserConfigurationException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Parser configuration exception parsing XML from " + resource, ex);
		}
		catch (IOException ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"IOException parsing XML document from " + resource, ex);
		}
		catch (Throwable ex) {
			throw new BeanDefinitionStoreException(resource.getDescription(),
					"Unexpected exception parsing XML document from " + resource, ex);
		}
	}
```

其中doLoadDocument方法代码如下：

```
/**
	 * Actually load the specified document using the configured DocumentLoader.
	 * @param inputSource the SAX InputSource to read from
	 * @param resource the resource descriptor for the XML file
	 * @return the DOM Document
	 * @throws Exception when thrown from the DocumentLoader
	 * @see #setDocumentLoader
	 * @see DocumentLoader#loadDocument
	 */
	protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
				getValidationModeForResource(resource), isNamespaceAware());
	}
```

其中registerBeanDefinitions方法的代码如下：

```
/**
	 * Register the bean definitions contained in the given DOM document.
	 * Called by {@code loadBeanDefinitions}.
	 * <p>Creates a new instance of the parser class and invokes
	 * {@code registerBeanDefinitions} on it.
	 * @param doc the DOM document
	 * @param resource the resource descriptor (for context information)
	 * @return the number of bean definitions found
	 * @throws BeanDefinitionStoreException in case of parsing errors
	 * @see #loadBeanDefinitions
	 * @see #setDocumentReaderClass
	 * @see BeanDefinitionDocumentReader#registerBeanDefinitions
	 */
	public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		int countBefore = getRegistry().getBeanDefinitionCount();
		documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```

以上代码可以看出，Spring在进行XML配置文档的解析。 因为Spring的配置比较多，所以整个解析过程也是比较复杂，在此不做展开说明。下面附上Spring中XML解析中设计到的主要类的类图：

![018050713564](images\20180507135644.png)



## 三、总结

Spring中解析XML的主要是采用Java中的DOM解析，同时也借助了SAX API中的InputSource类完成XML的验证和解析。解析XML的核心类是XmlBeanDefinitionReader。