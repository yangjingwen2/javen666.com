# Apache Server搭建图片服务器

```
导读：公司来了几个应届生，经常问我图片应该上传到哪里，是直接在工程下面创建一个upload_image文件夹，然后将图片上传到这个upload_image文件夹下吗？ 怎么回答呢，说“不是这样操作”，那肯定问为什么，然后就是循环的为什么了。 说“是这样操作也可以，但是有需要注意的地方”，那还是会出现循环的为什么。于是，我还不如把自己的理解写出来，免得后面一个个解释。（以下都是以JavaEE环境为基础）
```

## 一、单服务器

```
场景：直接在工程下面新建一个图片文件夹，比如：/images/，然后所有的图片都上传到这个文件夹下。
```

### 1.1、图片文件夹在项目内部（包含关系）

这种存储图片的方式，是初学者首先接触到的方式，因为简单易操作，也确实能到达目的。但是存在一个问题，就是在进行项目版本升级的时候，有可能会直接将之前的项目删掉，重新部署新版本的项目，这样就会导致老版本项目下的图片全部被删除。所以，将图片文件夹和项目单独分开可以解决这个问题。

### 1.2、图片文件夹和项目都放在服务器的根目录下（兄弟关系）

（图片文件夹和项目是兄弟关系，而非包含关系）将图片文件夹和项目文件夹单独分开，最简单的做法就是在服务器的根目录下新建一个images文件夹。这样，图片文件夹和项目文件夹就是兄弟文件夹关系了，删除项目的时候，不会影响图片文件夹。

比如，服务器是tomcat，在tomcat服务器的webapp文件夹下，创建一个images文件夹。这样，项目路径和图片路径如下：

图片地址：{tomcat}/webapp/images

项目地址：{tomcat}/webapp/项目名称

## 二、Apache Server和Tomcat

如第一点所述，图片和项目都放在一个tomcat中，虽然解决了图片被误删除的可能。但是，Tomcat是一个Java应用服务器，主要用来处理动态资源，比如servlet和jsp。Tomcat是Servlet的容器，处理静态资源（HTML、图片等）效率没有apache server的效率高。为了提升项目中静态资源的访问速度，现在流行的服务架构是“动静分离架构”。比如将servlet放在tomcat中，将html、图片等放在apache server中。



## 三、独立图片服务器的优势

```
搭建独立图片服务器的原因：
1、动静分离
2、分布式架构中，独立的图片服务器可以被共享。
```

分布式架构中的图片服务器：

![20180124-fenbushi-images-server](https://github.com/yangjingwen2/images/blob/master/20180124-fenbushi-images-server.png?raw=true)

如上图，独立的图片服务器，在分布式架构中，可以做到多个服务器共享。

## 四、Apache Server搭建独立图片服务器

```
基于windows环境的安装配置过程
```

### 4.1、下载Apache Server

下载地址:  http://httpd.apache.org/docs/current/platform/windows.html#down 

![20180124-apache-download](https://github.com/yangjingwen2/images/blob/master/20180124-apache-download.png?raw=true)

### 4.2、解压

将下载的压缩文件解压，我解压之后放在E盘，并且修改了文件夹的名称（<u>可以不修改</u>）,我的apache解压地址如下：E:\apache-httpd-2.4.29-o110g-x86-vc14\Apache24

### 4.3、配置

1、找到E:\apache-httpd-2.4.29-o110g-x86-vc14\Apache24\conf\httpd.conf文件，打开。配置如下内容：

![20180124-apache-settting1](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting1.png?raw=true)

如上图，找到38行，修改SRVROOT后面的地址为解压后的apache目录。

2、修改apache的端口

默认端口是80，也可以不修改。如果80端口被占用，可以修改端口号，配置如下：

![20180124-apache-settting2](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting2.png?raw=true)

![20180124-apache-settting3](F:\javaee\我的案例\我的备课\Apache server图片服务器\20180124-apache-settting3.png)

3、安装apache server

a、管理员身份打开cmd命令。

b、执行如下命令：

```
E:\apache-httpd-2.4.29-o110g-x86-vc14\Apache24\bin>httpd.exe -k install -n apache-server
```

其中apache-server是自定义的服务名称。然后等着安装成功，提示如下：“Errors reported here must be corrected before the service can be started”。如果“Errors reported here must be corrected before the service can be started”此句下方有错误代码，表示安装失败。通过“sc delete apache-server”命令删除服务，解决异常之后，然后重新安装。

4、启动服务

在E:\apache-httpd-2.4.29-o110g-x86-vc14\Apache24\bin\下，双击ApacheMonitor.exe，运行后，出现如下界面：

![20180124-apache-settting4](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting4.png?raw=true)

点击，右边的“start”启动服务。

5、测试

打开浏览器，输入http://localhost:83 就会出现如下界面：

![20180124-apache-settting5](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting5.png?raw=true)

到此，apache服务安装成功。



6、配置图片文件夹

在E盘创建文件夹：E:/apache/images，用来存放上传的图片。然后打开httpd.conf配置如下：

![20180124-apache-settting6](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting6.png?raw=true)

说明：Directory标签下的AllowOverride none 和Require all granted是访问权限的配置。

![20180124-apache-settting7](https://github.com/yangjingwen2/images/blob/master/20180124-apache-settting7.png?raw=true)

说明：Alias /images E:/apache/images   其中E:/apache/images是真实的图片地址，/images是用户访问的地址。配置后，用户访问路径如下：http://localhost:83/images/ddd.png

## 五、总结

apache图片服务器的搭建就到此。但是，不仅仅只有apache server能作为图片服务器，可以作为图片服务器的还有nginx、ftp、fastdfs等等。后面慢慢自己搭建~    

在公司，搭建服务器这种事，一般是运维做的事情。不过懂一点，也是好事。至少跟运维沟通比较顺畅了。