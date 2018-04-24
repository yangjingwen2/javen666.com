# Spring Boot详解（五）：通过XML整合mybatis

本小节介绍Spring boot和mybatis的整合，采用xml方式。上一篇已经介绍了通过注解整合MyBatis[Spring Boot详解（四）：简洁整合mybatis](Spring Boot详解（四）：简洁整合mybatis.md)。

## 一、导入依赖

~~~
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
<dependency>
    <groupId>com.mchange</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.5.2</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
~~~

以上仅仅是将mybatis所需要的依赖写进来了。相信你看到这篇文章之前已经对springboot有所了解。如果不是很了解的同学，可以先看看另外一篇文章[《Spring Boot详解（一）：HelloWorld》](Spring Boot详解（一）：HelloWorld.md)。

## 二、创建Mapper接口和映射文件

###2.1、mapper接口

~~~
package com.qianfeng.springbootdemo.mapper;

import com.qianfeng.springbootdemo.dto.DemoDTO;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;
import org.springframework.data.annotation.Id;

public interface DemoMapper {

    DemoDTO selectOneById(@Param("id") Integer id);

}

~~~

### 2.2、映射文件（demo.mapper.xml）

~~~
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.qianfeng.springbootdemo.mapper.DemoMapper">
​
    <resultMap id="demoResultMap" type="com.qianfeng.springbootdemo.dto.DemoDTO">
        <id property="demoId" column="demo_id" javaType="java.lang.Integer"/>
        <result property="demoName" column="demo_name" javaType="java.lang.String"/>
    </resultMap>
​
    <select id="selectOneById" resultMap="demoResultMap">
        SELECT demo_id , demo_name FROM tb_demo WHERE demo_id=#{id}
    </select>
</mapper>
~~~





## 三、创建Java Bean类

~~~
package com.qianfeng.springbootdemo.dto;

import java.io.Serializable;

public class DemoDTO implements Serializable {

    private Integer demoId;

    private String demoName;

    public DemoDTO() {
    }

    public DemoDTO(Integer demoId, String demoName) {
        this.demoId = demoId;
        this.demoName = demoName;
    }

    public Integer getDemoId() {
        return demoId;
    }

    public void setDemoId(Integer demoId) {
        this.demoId = demoId;
    }

    public String getDemoName() {
        return demoName;
    }

    public void setDemoName(String demoName) {
        this.demoName = demoName;
    }
}

~~~

对应的数据库结构如下：

+-----------+-------------+------+-----+---------+----------------+
| Field     | Type        | Null | Key | Default | Extra          |
+-----------+-------------+------+-----+---------+----------------+
| demo_id   | int(11)     | NO   | PRI | NULL    | auto_increment |
| demo_name | varchar(22) | YES  |     | NULL    |                |
+-----------+-------------+------+-----+---------+----------------+

## 四、application.properties配置

~~~
mybatis.mapper-locations=classpath:mapper/*.mapper.xml
spring.datasource.type=com.mchange.v2.c3p0.ComboPooledDataSource
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/java1706
spring.datasource.username=root
spring.datasource.password=123456
~~~

以上只是配置了数据源，跟传统意义上的mybatis的配置简化了很多。Spring boot会自动将数据源注入到SqlSessionFactory中，并且自动将SqlSessionFactory注入到mapper中，不需要我们操心，简化了配置过程，这也是Spring Boot的目的。



**注意：这个位置的spring.datasource.username，不要写成了spring.datasource.user，否则会出现连接数据库的异常 **



spring.datasource.type：指定采用数据源的类型，也可以不配置，Spring Boot有默认的数据源。





## 五、测试

~~~
package com.qianfeng.springbootdemo;

import com.qianfeng.springbootdemo.dto.DemoDTO;
import com.qianfeng.springbootdemo.mapper.DemoMapper;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringbootdemoApplicationTests {


	@Autowired
	private DemoMapper demoMapper;


	@Test
	public void testCase1(){
		System.out.println(demoMapper);
		DemoDTO demoDTO = demoMapper.selectOneById(1);
		System.out.println(demoDTO.getDemoName());
	}
}

~~~



简单XML方式和纯注解的方式大体上一样。



