# Spring Boot详解（二）：thymeleaf

因为Spring Boot不建议使用jsp，可能是因为jsp糟糕的可读性。而thymeleaf的可读性是非常好，乍一看，感觉就是一个html页面。不需要web服务器也可以通过浏览器显示效果。 类似thymeleaf的还有freemarker，是目前使用比较多的两种模板引擎。

## 一、配置

~~~
#thymeleaf
spring.thymeleaf.suffix=.html
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.mode=HTML5
spring.thymeleaf.servlet.content-type=text/html
~~~

spring.thymeleaf.suffix=.html  表示页面后缀是html格式

spring.thymeleaf.prefix=classpath:/templates/  表示html页面在resources文件夹下的templates文件夹中。

spring.thymeleaf.encoding=UTF-8 采用utf-8编码，避免页面乱码。

spring.thymeleaf.mode=HTML5  页面是html5类型。

spring.thymeleaf.servlet.content-type=text/html 文档MIME类型是text/html，也就是html格式的文档。



### 1.1、基本语法

~~~
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Thymeleaf template</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
</head>
<body>


<p th:text="${msg}">Hello World</p>

</body>
</html>
~~~

整体上跟html语法一样，注意这个位置<html xmlns:th="http://www.thymeleaf.org">就行。

## 二、取值

~~~
<p th:text="${msg}">Hello World</p>
~~~

如上，获取后端接口传递的msg参数。基本都是html标签，除了th:text="“。th:text是thymeleaf模板标签，用来显示后端传递的数据，并且替换Hello World文字。上面这句话的意思是如果msg有值则显示msg内容，否则显示Hello World。



### 2.1、引入url

~~~
<a th:href="@{http://javen666.com}">课堂笔记</a>
~~~



## 三、条件分支

### 3.1、条件

th:if     如果条件成立

th:unless 如果条件不成立

比如：

~~~
<a th:href="@{/login}" th:unless=${session.user != null}>Login</a>
~~~

### 3.2、分支

th:switch

比如：

~~~
<div th:switch="${user.role}">
  <p th:case="'admin'">User is an administrator</p>
  <p th:case="#{roles.manager}">User is a manager</p>
</div>
~~~



## 四、循环

th:each

比如：

~~~~
<tr th:each="prod : ${prods}">
      <td th:text="${prod.name}">Onions</td>
      <td th:text="${prod.price}">2.41</td>
      <td th:text="${prod.inStock}? #{true} : #{false}">yes</td>
    </tr>
~~~~



## 五、运算

${ }表达式中可以进行+-*/运算。



## 六、其他

thymeleaf提供了最基本的操作之外，对string字符串、date格式化、集合空判断等也有一定的支持。比如：

~~~
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}

${#strings.isEmpty(name)}
~~~



说明：thymeleaf不是一定要在spring boot环境下才能使用。 一般springmvc整合thymeleaf也可以使用，用法一样。

