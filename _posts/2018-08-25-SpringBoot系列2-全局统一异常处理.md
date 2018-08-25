---
layout: post
title: SpringBoot系列2-全局统一异常处理
tags:
- spring-boot


categories: spring-boot
description:  SpringBoot系列2-全局统一异常处理
---
 springboot做全局统一异常处理
<!-- more -->
SpringBoot系列2-全局统一异常处理

###	为什么要全局统一异常处理呢？ ###
如果系统发生了异常，不做统一异常处理，前端会给用户展示一大片看不懂的文字。做统一异常处理后当异常发生后可以给用户一个温馨的提示，不至于使用户满头雾水，所以一方面是为了更好的用户体验
如果不统一全局异常，服务端和前端在遇到异常的时候处理起来杂乱无章非常费力。所以另一方面是为了制定规范提高工作效率

###	SpringBoot如何实现全局统一异常处理呢？ ###

项目目录结构
 
<img src="{{ site.assets }}/images/2018-08-25/20180825132413.png"/>

### 自定义异常类 ###
pom.xml加入lombok
```
<dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
</dependency>

```
MyException.java
```
@Getter
@Setter
public class MyException extends RuntimeException {
    private Integer code;
    public MyException(String msg){
        super(msg);
    }

    public MyException(Integer code,String msg){
        super(msg);
        this.code=code;
    }
}
```

### 自定义枚举类 ###
```
public enum ResultEnum {
    SUCCESS(200,"成功"),
    FAIL(100,"失败"),
    EXCEPTION(300,"系统异常"),
    UNLOGIN(201,"未登录");

    private Integer code;
    private String msg;

    private ResultEnum(Integer code,String msg){
        this.code=code;
        this.msg=msg;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }
```
### 自定义全局异常捕获类 ###
```
@ControllerAdvice
public class MyExceptionAdvice {

    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result defaultException(HttpServletRequest request,Exception e){
        e.printStackTrace();
        return Result.builder()
                .code(ResultEnum.EXCEPTION.getCode())
                .message(ResultEnum.EXCEPTION.getMsg())
                .build();
    }

    @ExceptionHandler(value = MyException.class)
    @ResponseBody
    public Result myException(HttpServletRequest request,MyException e){
        e.printStackTrace();
        Integer code=e.getCode();
        String message=e.getMessage();

        if (e.getCode()==null){
            code=ResultEnum.EXCEPTION.getCode();
        }
        if (e.getMessage()==null){
            message=ResultEnum.EXCEPTION.getMsg();
        }
        return Result.builder()
                .code(code)
                .message(message)
                .build();
    }
}
```
### 进行测试 ###
```
@RestController
public class ExceptionController {
    @RequestMapping("/exception")
    public Result exception(String name,String pwd) throws Exception {
            String realname="tiankonglanlande";
            String realPwd="123";

            if(null!=name&&name.equals("xx")){
                throw new Exception("系统异常！");
            }
            if(StringUtils.isEmpty(name)||StringUtils.isEmpty(pwd)){
                throw new MyException("参数必须传！");
            }else if (!name.equals(realname)||!pwd.equals(realPwd)){
                throw new MyException("用户名或密码不正确！");
            }else if (name.equals("aa")){
                throw new MyException(200,"用户名存在！");
            }
            String info="你好["+name+"]!";
        return ResultUtils.success(info);
    }
    @RequestMapping("/unlogin")
    public Result unlogin() throws Exception {
        return ResultUtils.success(ResultEnum.UNLOGIN);
    }
    @RequestMapping("/success")
    public Result success() throws Exception {
        return ResultUtils.success(200,"自定义消息");
    }
} 
```
### 运行测试在浏览器访问 ###
 可以看到浏览器返回对应的json字符串
  <img src="{{ site.assets }}/images/2018-08-25/20180825132412.png"/>

### 源码地址：
<a href="https://github.com/tiankonglanlande/springboot" target="_blank">https://github.com/tiankonglanlande/springboot</a> <br>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">天空蓝蓝的</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> <br>
版权所有，欢迎保留原文链接进行转载：)
关注我们的公众号了解更多<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>