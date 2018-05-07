# Spring Boot详解（六）：简洁整合Shiro

Shiro是一个系统安全框架，Spring boot整合Shiro的过程也是非常简介。

## 一、导包

```
		<dependency>
			<groupId>org.apache.shiro</groupId>
			<artifactId>shiro-spring</artifactId>
			<version>1.4.0</version>
		</dependency>
```

shiro并没有提供spring boot的专属包，所以导入的依赖跟非spring boot项目的依赖是一致的。



## 二、Spring Boot中Shiro的配置

```
package com.qianfeng._50_springboot.config;

import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.realm.jdbc.JdbcRealm;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.servlet.ShiroFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;

@Configuration
public class ShiroConfig {

    @Bean
    public SecurityManager provideSecurityManager(){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(new JdbcRealm());
        return securityManager;
    }

    /**
     * shiro过滤器
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean provideShiroFilter(SecurityManager securityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        //指定登录页面
        shiroFilterFactoryBean.setLoginUrl("/user/login");
        //过滤条件-----
        LinkedHashMap<String, String> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put("/user/login","anon");
        linkedHashMap.put("/**","authc");

        shiroFilterFactoryBean.setFilterChainDefinitionMap(linkedHashMap);

        return shiroFilterFactoryBean;

    }
}

```

添加如上代码到spring boot项目中，Shiro框架会自动完成安全拦截效果。其中采用linkedHashMap是因为采用其有序的特征。因为shiro的拦截规则是从上往下范围依此递增。切记！不能使用HashMap无序集合，否则拦截规则会混乱。



