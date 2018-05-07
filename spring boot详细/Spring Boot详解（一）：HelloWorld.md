# Spring Boot详解（一）：HelloWorld

Spring Boot使您可以轻松创建可以运行的独立的，生产级的基于Spring的应用程序。Spring Boot的目的是为了简化Spring繁琐的配置，让spring的开发变得更加简单（划重点：更加简单了。所以不要有心理障碍）。个人认为从来没有接触过spring的小白，更容易学习spring boot。



## 一、Spring Boot项目的创建

Spring Boot项目创建的方式有三种。（其中第三种方式是最简单的方式，通用与任何开发工具）

### 第一种：纯手工打造

纯手工方式不推荐。 不过，如果以下第二、三种方式不能满足你的要求。则按照下图目录结构创建Maven项目：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-4.png?raw=true)

手动添加图上序号3、4所指目录。然后在pom.xml中添加如下内容：

~~~
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.javen</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>


</project>

~~~

此种手动创建项目方式不推荐，以下两种方式更加方便。

### 第二种：通过idea创建spring boot项目

1、打开idea，新建一个项目，进入如下界面：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-1.png?raw=true)

2、第一步点击next按钮之后，进入如下界面：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-2.png?raw=true)

上图的group和artifact可以自行修改（当然也没有默认），然后点击next按钮进入下一步。

3、选择要集成的spring的模块，如下：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-3.png?raw=true)

因为本案例只是演示spring web在spring boot种的使用方式，所以只选择了web（如图箭头处）。之后，点击next按钮（或者finish按钮），等一会，项目就创建完成。  完成之后的项目结构如下：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-4.png?raw=true)

图上序号1：源文件所在的目录，没有什么特别之处。

图上序号2：spring boot的启动类，里面之后一个main方法。不需要修改。

图上序号3：静态资源文件夹，是springboot的默认文件夹。

图上序号4：html页面所在的文件夹，也是springboot的默认文件夹。因为springboot不建议使用jsp，所以页面一般是freemarker或者thymeleaf。

图上序号5：spring的配置文档，有一些需要自定义的配置信息可以在此文档种设置，比如jdbc的url等。



### 第三种：在Spring官网上自动生成

这是最简单的一种方式（需要有网）。

1、进入如下网站：http://start.spring.io/，如图：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-6.png?raw=true)

按照上图说明进行配置。最后点击”Generate Project“按钮生成并且下载项目。下载完成之后解压，通过eclipse或者idea打开项目即可。

## 二、利用Spring Boot编写一个Spring MVC的接口

按照上面的方式创建了一个集成springweb的springboot项目之后。接下来，我们创建一个controller接口进行测试。

1、创建如下controller类：

~~~
package com.javen.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/test1")
    @ResponseBody
    public String test2(Model model){

        return "hello";
    }
    
}

~~~

2、执行springboot启动类的main方法。如图：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-8.png?raw=true)



3、控制台如下信息则表示启动成功：

![pringboot-](https://github.com/yangjingwen2/javen666.com/blob/master/spring%20boot%E8%AF%A6%E7%BB%86/image/springboot-7.png?raw=true)

注意：如果在最后一行之后，还有error信息则表示启动失败。根据提示修改正确之后，可以再重新启动。

4、测试，打开浏览器。在地址栏输入：http://localhost:8080/user/test1 ，能显示hello文字，表示测试成功。



## 说明

1、springboot自带web服务器插件。不需要配置tomcat。

2、传统意义上的spring的配置方式再springboot中基本都省略了。所以上面的步骤中没有添加什么spring的配置内容。
