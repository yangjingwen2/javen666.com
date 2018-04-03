# Dubbo分布式服务框架

Dubbo是是一个高性能，基于Java的RPC框架，由阿里巴巴开源。一个分布式的服务框架。可以实现SOA（面向服务的架构）架构。 Dubbo使用的公司：京东、当当、阿里巴巴、中国电信等等。

## 分布式服务架构的由来

```
问题：比如电信的计费系统提供了最原始的扣费功能，需要接入此计费系统的应用比较多，比如营业厅的系统（办理普通用户需要扣费），比如打电话需要计费、比如流量需要计费、比如宽带需要计费、比如ITV需要计费等等。  如果在每一个系统中都写一套计费的逻辑就会出现代码冗余过大，并且逻辑无法做到统一可维护。 所以需要将计费功能单独提炼出来，作为一个业务模块提供给其他应用使用。
```

如图：

![布式服务架构演](F:/javaee/1706/%E7%AC%94%E8%AE%B0/%E5%88%86%E5%B8%83%E5%BC%8F%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E6%BC%94%E5%8F%98.png)

如上图，电信业务最早只有打电话的功能，所以开发一个应用用来进行打电话并且扣费就行了。但是随着社会发展越来越快，又多了宽带和流量功能。使用宽带和流量也是需要扣费的。 业务最早没有想到社会出现“流量“、”宽带“这些需求东西，所以原始的电话扣费应用中的打电话和扣费是整合到一起了，也就是在一个应用里面。后面出现了宽带，但是宽带很多逻辑跟打电话不一样，甚至开发的人员和公司都不一样，所以也不能直接在原有系统中增加一个”宽带“模块；再者，考虑到后期可能还会出现其他业务计费功能，不可能在一个项目中无止境的增加业务功能模块。所以需要独立开发一个新的项目单独用来进行宽带计费。  这样导致如下问题：

1、这样会导致”电话“和”宽带“都需要计费模块。如果在两个应用中都继承一个计费模块，那么又会导致维护困难、不能统一维护。

2、再者程序中存在一个魔性的数字：65535（16bit最大值）限制，（因为调用方法的指令容量只有16bit，65535正好是16bit能容纳的最大数字）。重复的方法数太多，会加速达到这个上限。（比如Android 应用65535很容易就上限了）。

所以需要将计费模块独立出来（独立于任何的应用，不在属于”打电话“或者”宽带“中的一个集成模块了，而成为一个独立不受任何业务影响的模块）。改造如下：

![布式服务架构演变](F:/javaee/1706/%E7%AC%94%E8%AE%B0/%E5%88%86%E5%B8%83%E5%BC%8F%E6%9C%8D%E5%8A%A1%E6%9E%B6%E6%9E%84%E6%BC%94%E5%8F%982.png)

如上图，每一个方块就是一个独立的应用。这样解决了业务复杂度，将业务模块化、独立化，方便共享和扩展。扩展之后的系统存在一个问题：

1、各个独立app之间的通信问题怎么解决？

2、怎么做到统一调度、协调处理。

3、如果计费模块是并发最大的模块，但是其他模块并发不是很大。则需要对计费进行负载均衡，怎么实现？



## 架构演变过程

![dubbo-17071-20160628000111859-650529644.jpg](https://github.com/yangjingwen2/images/blob/master/dubbo-17071-20160628000111859-650529644.jpg?raw=true)





## 什么是RPC？
RPC（Remote Procedure Call Protocol）远程过程调用协议。服务器A调用服务器B上的方法的一种技术。Dubbo就是一个RPC框架，实现了远程过程调用。



## Dubbo的原理图
![image](http://dubbo.io/images//dubbo-architecture.png)


dubbo主要的三个要素：
1、接口的远程调用
2、负载均衡。
3、服务自动注册和发现

## Dubbo的使用
### 1、说明
Dubbo框架需要有注册中心，本案例中使用Redis作为Dubbo的注册中心。除了Redis外，Zookeeper等也可以作为Dubbo的注册中心。

### 2、环境要求
JDK 1.6以上。

### 3、添加依赖
```
<dependencies>
        <!-- https://mvnrepository.com/artifact/com.alibaba/dubbo -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.5.5</version>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>1.8.4.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>4.3.11.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>4.3.11.RELEASE</version>
        </dependency>
    </dependencies>
```

### 4、定义Dubbo服务接口
其实就是创建一个独立的module（并且在pom.xml导入以上依赖），在module中创建一些接口和方法（也叫服务）。
比如在dubbo_service_interface中创建一个接口IDubboService，代码如下:
```
package com.javen.dubbo.service;

import java.util.List;

/**
 * Dubbo的服务
 * @author yangjw 
 */
public interface IDubboService {

    List<String> getData(String data);

}

```

### 5、定义服务提供者（接口具体实现者）
创建另外一个module，命名dubbo_provider1（并且在pom.xml导入以上依赖）。将dubbo_service_interface作为依赖添加进来。

1、在其中创建类DubboService，并且继承IDubboService接口。代码如下：
```
package com.javen.dubbo.provider;

import com.javen.dubbo.service.IDubboService;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * 服务提供者
 * @author yangjw 
 */
@Component("dubboService")
public class DubboService implements IDubboService{


    public List<String> getData(String data) {

        ArrayList<String> list = new ArrayList<String>();
        list.add("这是Dubbo中Provider返回的数据：" + data);
        return list;
    }
}

```
2、配置spring-dubbo.xml,代码如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">

    <context:annotation-config/>
    <context:component-scan base-package="com.javen.dubbo.provider"/>
    <!--配置dubbo服务的唯一名称 -->
    <dubbo:application name="dubbo_provider1"/>
    <!--将服务注册到redis中，并且配置协议和端口为20880-->
    <dubbo:registry address="redis://192.168.72.188:6379"/>
    <dubbo:protocol name="dubbo" port="20880"/>
    <!--配置服务接口，ref关联到服务实现类-->
    <dubbo:service interface="com.javen.dubbo.service.IDubboService" ref="dubboService"/>

</beans>
```

3、启动Provider,代码如下：
```
package com.javen.dubbo.provider;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

public class StartProvider {

    public static void main(String[] args) throws IOException {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
                new String[] {"spring-dubbo.xml"});
        System.out.println("服务1启动~~~");
        context.start();
        //线程阻塞：保证服务一直存在，如果线程结束，服务终止
        System.in.read(); // press any key to exit
    }
}

```


### 6、定义服务消费者（接口的具体调用者）

再创建一个module，命名dubbo_consumer（并且在pom.xml导入以上依赖）。将dubbo_service_interface作为依赖添加进来。

1、配置spring-dubbo.xml,代码如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                             http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">


    <dubbo:application name="demo-consumer"/>
    <dubbo:registry address="redis://192.168.72.188:6379"/>
    <dubbo:reference id="demoService" interface="com.javen.dubbo.service.IDubboService"/>
</beans>
```

2、测试
```
package com.test;

import com.javen.dubbo.service.IDubboService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import java.util.List;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-dubbo.xml")
public class DubboTest {

    @Resource(name = "demoService")
    private IDubboService dubboService;
    @Test
    public void testDubbo(){
        List<String> haha = dubboService.getData("haha");
        System.out.println(haha.get(0));
    }
}

```

## Dubbo、MyCat、主从配置\读写分离、redis分布式、JTA分布式事务的关系。
![image](https://github.com/yangjingwen2/images/blob/master/dubbo_redis_jiagou.png?raw=true)

dubbo默认每次只访问一个服务器，需要主从配合完成数据同步。
```
<!--replicate可以实现所有服务器同步写，但是只读取单台服务器。默认是failover，读写都是单台服务器-->
    <dubbo:registry cluster="replicate" address="redis://192.168.72.188:6379"/>
```