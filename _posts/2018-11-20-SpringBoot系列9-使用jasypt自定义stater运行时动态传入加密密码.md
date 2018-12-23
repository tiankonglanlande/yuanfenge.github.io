---
layout: post
title: SpringBoot系列9-使用jasypt自定义stater运行时动态传入加密密码
tags:
- spring-boot
- jasypt

categories: spring-boot
description: 
---
 
  介绍如何自定义一个stater将jasypt封装为一个stater供多个项目使用，同时运行时动态传入加密密码
<!-- more -->

#### 新建springboot-encryption-configuration项目实现stater

#### pom文件引入jasypt
```xml
   <dependencies>
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>com.github.ulisesbocchio</groupId>
			<artifactId>jasypt-spring-boot-starter</artifactId>
			<version>2.1.0</version>
		</dependency>
   </dependencies>
```
说明：

spring-boot-starter 自定义stater引入则具备stater的能力；

jasypt-spring-boot-starter自定义stater引入则具备加密解密能力

#### 在resources/support/下配置application.properties文件
```properties
jasypt.encryptor.password=${PWD}
```
说明：接收运行时传入的解密密码

#### 在resources/support/下根据不同环境配置文件，此处我的开发和测试环境application-${spring.profiles.active}.properties文件的加密内容一样。请你根据你的情况来配置
application-dev.properties和application-test.properties
```
u.username=ENC(uF42uO9PxXDQYtnYe5++ebszGCkWXdY1NynJ+5Lptsg=)
u.pwd=ENC(+v5XN2Tv+d6VDfpeapME+S6vw2nOfE9L/1sjh6UFzso=)
```
#### 定义SupportConfiguration.java
```java
/**
 * @author tiankonglanlande
 * @description
 * @createTime 2018 - 11 - 20 11:53
 */
@EnableEncryptableProperties
@EncryptablePropertySources({@EncryptablePropertySource(value = "classpath:support/application.properties", ignoreResourceNotFound = true),
        @EncryptablePropertySource(value = "classpath:support/application-${spring.profiles.active}.properties", ignoreResourceNotFound = true)})
public class SupportConfiguration {

}
```
说明：

@EnableEncryptableProperties启用属性加密

@EncryptablePropertySource指定加密文件（这里的application-${spring.profiles.active}.properties是根据环境动态加载配置文件）

#### 将上面项目安装到maven库
我这里是使用的idea工具选择Maven Project-->springboot-encryption-configuration --> clean --> 然后compile --> 最后install

#### 新建springboot-encryption-configuration-demo测试stater

#### pom文件引入stater依赖
```xml
    <!--引入stater-->
    <dependency>
        <groupId>com.tiankonglanlande.cn.springboot.stater</groupId>
        <artifactId>springboot-encryption-configuration-stater</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
```
#### 编写测试类DemoApplicationTests.java
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {
	@Value("${u.username}")
	private String username;
	@Value("${u.pwd}")
	private String pwd;

	@Test
	public void contextLoads() {
		System.out.println("读取username:"+username);
		System.out.println("读取pwd:"+pwd);

	}
}
```
#### 运行
说明：
点击运行左侧栏的绿色按钮：此时会报错

idea选择 Edit Configurations--> JUnit -->选择 DemoApplicationTests.contextLoads --> 在右边的VM options 填入启动的环境和加密密码 -Dspring.profiles.active=test -DPWD=tiankonglanlande

#### 大功告成：运行测试类DemoApplicationTests的contextLoads方法console打印出原内容如下
```
读取username:tiankonglanlande
读取pwd:tiankonglanlande_pwd

```

[源码下载链接](https://github.com/tiankonglanlande/springboot)

[原文链接: http://www.lskyf.com/view/27](http://www.lskyf.com/view/27)

作者：天空蓝蓝的，版权所有，欢迎保留原文链接进行转载：)

