---
layout: post
title: SpringBoot防止重复请求，重复表单提交超级简单的注解实现之一(初级版)
tags:
- spring-boot 
- annotation
- 防止重复请求

categories: spring-boot 
description:  spring-boot 
---
##  需求说明：想要实现一个防止重复提交的功能：网上的实例也很多个人觉得很麻烦，要是能简单到只需要添加一个标记到需要防止重复提交的方法上，不管你请求来自移动端还是pc端，而且不需要防止重复的地方我就不添加标记那该多好啊！
<!-- more -->


需求：想要实现一个防止重复提交的功能：网上的实例也很多个人觉得很麻烦，要是能简单到只需要添加一个标记到需要防止重复提交的方法上，不管你请求来自移动端还是pc端，而且不需要防止重复的地方我就不添加标记那该多好啊！
实现思路：
注解的方式+拦截器
1. 注解接口 
/**
 * @description 防止表单重复提交注解
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface DuplicateSubmitToken {

    //保存重复提交标记 默认为需要保存
    boolean save() default true;

}
2. 拦截器
 

/**
 * @description 防止表单重复提交拦截器
 */
@Aspect
@Component
@Slf4j
public class DuplicateSubmitAspect {
    public static final String  DUPLICATE_TOKEN_KEY="duplicate_token_key";

    @Pointcut("execution(public * cn.test.core.controller..*(..))")

    public void webLog() {
    }

    @Before("webLog() && @annotation(token)")
    public void before(final JoinPoint joinPoint, DuplicateSubmitToken token){
        if (token!=null){
            Object[]args=joinPoint.getArgs();
            HttpServletRequest request=null;
            HttpServletResponse response=null;
            for (int i = 0; i < args.length; i++) {
                if (args[i] instanceof HttpServletRequest){
                    request= (HttpServletRequest) args[i];
                }
                if (args[i] instanceof HttpServletResponse){
                    response= (HttpServletResponse) args[i];
                }
            }

            boolean isSaveSession=token.save();
            if (isSaveSession){
                Object t = request.getSession().getAttribute(DUPLICATE_TOKEN_KEY);
                if (null==t){
                    String uuid= UUID.randomUUID().toString();
                    request.getSession().setAttribute(DUPLICATE_TOKEN_KEY,uuid);
                    log.info("进入方法：token="+uuid);
                }else {
                    throw new CustomException("请不要重复请求！");
                }
            }

        }
    }

    @AfterReturning("webLog() && @annotation(token)")
    public void doAfterReturning(JoinPoint joinPoint,DuplicateSubmitToken token) {
        // 处理完请求，返回内容
        log.info("出来方法：");
        if (token!=null){
            Object[]args=joinPoint.getArgs();
            HttpServletRequest request=null;
            for (int i = 0; i < args.length; i++) {
                if (args[i] instanceof HttpServletRequest){
                    request= (HttpServletRequest) args[i];
                }
            }
            boolean isSaveSession=token.save();
            if (isSaveSession){
                Object t = request.getSession().getAttribute(DUPLICATE_TOKEN_KEY);
                if (null!=t){
                    //方法执行完毕移除请求重复标记
                    request.getSession(false).removeAttribute(DUPLICATE_TOKEN_KEY);
                    log.info("方法执行完毕移除请求重复标记！");
                }
            }
        }
    }
}
3. 控制器 
在你要使用的控制器要防止重复提交的方法添加注解@DuplicateSubmitToken即可
 

@RestController
public class TestController {
    @DuplicateSubmitToken
    @RequestMapping(value = "/test/d", method = RequestMethod.GET)
    public Map<String, Object> test (HttpServletRequest request){

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
4. 测试

5.总结：初步实现了防止重复提交的功能，但是这只是考虑到了正常的逻辑，异常情况是怎么样的呢？异常情况会出现重复标记无法移除，从而影响后面的请求，如何是好呢？请看下一个版本！