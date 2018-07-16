---
layout: post
title: SpringBoot防止重复请求，重复表单提交超级简单的注解实现之四（终极版）
tags:
- spring-boot 
- annotation
- 防止重复请求

categories: spring-boot 
description:  spring-boot 
---
前言：上篇文章有的童鞋说不行啊,怎么不能防止重复提交呢？哈哈，来来来我们一起来解惑！
<!-- more -->

首先需要说明的是之前的防止重复提交是指：一次请求完成之前防止重复提交，当然扩展下就可以做到会话间防止重复提交，还可以扩展为某个时间段或者永久防止重复提交（这个我就不实现了），下面我来扩展一下相同会话防止重复提交其实很简单

在上一篇的基础上DuplicateAspect不移除标记为SESSION的token就可以了！

#### 1.DuplicateSubmitToken.java添加属性type，默认为一次请求完成之前防止重复提交 ####
```
/**
 * @description 防止表单重复提交注解
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface DuplicateSubmitToken {
    /**一次请求完成之前防止重复提交*/
    public static final int REQUEST=1;
    /**一次会话中防止重复提交*/
    public static final int SESSION=2;

    /**保存重复提交标记 默认为需要保存*/
    boolean save() default true;

    /**防止重复提交类型，默认：一次请求完成之前防止重复提交*/
    int type() default REQUEST;
}
```

#### 2.DuplicateSubmitAspect.java方法doAfterReturing判断如果REQUEST才移除防止重复提交，因为SESSION标记在会话结束 时或失效时会自动移除标记 ####
```
/**
 * @description 防止表单重复提交拦截器
 */
@Aspect
@Component
@Slf4j
public class DuplicateSubmitAspect {
    public static final String  DUPLICATE_TOKEN_KEY="duplicate_token_key";

    @Pointcut("execution(public * cn.test.controller..*(..))")

    public void webLog() {
    }

    @Before("webLog() && @annotation(token)")
    public void before(final JoinPoint joinPoint, DuplicateSubmitToken token){
        if (token!=null){
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();

            boolean isSaveSession=token.save();
            if (isSaveSession){
                String key = getDuplicateTokenKey(joinPoint);
                Object t = request.getSession().getAttribute(key);
                if (null==t){
                    String uuid= UUID.randomUUID().toString();
                    request.getSession().setAttribute(key.toString(),uuid);
                    log.info("token-key="+key);
                    log.info("token-value="+uuid.toString());
                }else {
                    throw new DuplicateSubmitException(TextConstants.REQUEST_REPEAT);
                }
            }

        }
    }

    /**
     * 获取重复提交key
     * @param joinPoint
     * @return
     */
    public String getDuplicateTokenKey(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        StringBuilder key=new StringBuilder(DUPLICATE_TOKEN_KEY);
        key.append(",").append(methodName);
        return key.toString();
    }

    @AfterReturning("webLog() && @annotation(token)")
    public void doAfterReturning(JoinPoint joinPoint,DuplicateSubmitToken token) {
        // 处理完请求，返回内容
        log.info("出方法：");
        if (token!=null){
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            boolean isSaveSession=token.save();
            if (isSaveSession){
                String key = getDuplicateTokenKey(joinPoint);
                Object t = request.getSession().getAttribute(key);
                if (null!=t&&token.type()==DuplicateSubmitToken.REQUEST){
                    request.getSession(false).removeAttribute(key);
                }
            }
        }
    }

    /**
     * 异常
     * @param joinPoint
     * @param e
     */
    @AfterThrowing(pointcut = "webLog()&& @annotation(token)", throwing = "e")
    public void doAfterThrowing(JoinPoint joinPoint, Throwable e, DuplicateSubmitToken token) {
        if (null!=token
                && e instanceof DuplicateSubmitException==false){
            //处理处理重复提交本身之外的异常
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            boolean isSaveSession=token.save();
            //获得方法名称
            if (isSaveSession){
                String key=getDuplicateTokenKey(joinPoint);
                Object t = request.getSession().getAttribute(key);
                if (null!=t){
                    //方法执行完毕移除请求重复标记
                    request.getSession(false).removeAttribute(key);
                    log.info("异常情况--移除标记！");
                }
            }
        }
    }
}
```
#### 3.使用标记为会话防止重复提交 #### 
```
/**
 * @description
 */
@RestController
public class TestController {

    @DuplicateSubmitToken(type = DuplicateSubmitToken.SESSION)
    @RequestMapping(value = "/test/d", method = RequestMethod.GET)
    public Map<String, Object> test (HttpServletRequest request){

        Random r=new Random();
        int i = r.nextInt(3);
        if (i==2){
            throw new CustomException("有异常");
        }

        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        Map<String, Object> map = new HashMap<>();
        request.getSession().setAttribute("request Url", request.getRequestURL());
        map.put("request Url", request.getRequestURL());
        return map;
    }

}
```
#### 4.做了这些可能还是会遇到数据重复插入问题 ####
我们可以根据给创建唯一索引给某个字段，防止数据不重复插入，或者给方法加锁（同步）


<br/>
<br/>

###### 作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">天空蓝蓝的</a> ######
###### 网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> ######
###### 个人博客：<a href="http://www.lskyf.xyz" target="_blank">http://www.lskyf.xyz</a> ######
###### 版权所有，欢迎保留原文链接进行转载：) ######