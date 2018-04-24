## Spring Boot详解（三）：Thymeleaf和Spring boot整合
本篇仅仅介绍spring boot和thymeleaf整合的一个小案例。具体thymeleaf的详解参看[Spring Boot详解（二）：thymeleaf](Spring Boot详解（二）：thymeleaf.md)。

### 导入依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
	<version>1.5.8.RELEASE</version>
</dependency>
```

### 页面hello.html(在resources/templates文件夹下)
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Thymeleaf template</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>

<p>Hello World</p>

<h1 th:text="${book.bookName}">ssss</h1>

</body>
</html>
```

### Controller层代码 BookController.java 
```
package com.javen.demo_springboot.web;

import com.javen.demo_springboot.bean.Book;
import com.javen.demo_springboot.dao.BookDAO;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@Controller
public class BookController {

    @Autowired
    private BookDAO bookDAO;

    @RequestMapping("/find")
    public String findBook(Map<String,Object> map){
        Book book = bookDAO.queryBookById();
        map.put("book",book);
        System.out.println("-----------------------------" + book.getBookName());
        return "/hello";
    }
}

```