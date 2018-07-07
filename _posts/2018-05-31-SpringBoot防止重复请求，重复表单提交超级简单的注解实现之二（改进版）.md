---
layout: post
title: SpringBoot防止重复请求，重复表单提交超级简单的注解实现之二（改进版）
tags:
- spring-boot 
- annotation
- 防止重复请求

categories: spring-boot 
description:  spring-boot 
---
##  接着上一篇的问题：异常情况的处理，首先我想到了设定一个时间点，当在拦截器检测如果过了这个时间点我就将重复标记删除不就不影响后边的请求了吗？说着就开始干了！
<!-- more -->


#### 1. 注解接口 ####
 
```
/**
 * @description 防止表单重复提交注解
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface DuplicateSubmitToken {

    //保存重复提交标记 默认为需要保存
    boolean save() default true;
    //超时时间：默认5秒
    long timeout() default 5000;

}
```

#### 2. 拦截器 ####
```
/**
 * @description 防止表单重复提交拦截器
 */
@Aspect
@Component
@Slf4j
public class DuplicateSubmitAspect {
    public static final String  DUPLICATE_TOKEN_KEY="duplicate_token_key";
    public static final String  DUPLICATE_TOKEN_SAVE_TIME_KEY="duplicate_token_save_time_key";

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
                if (null!=t){
                    Object oldTime = request.getSession().getAttribute(DUPLICATE_TOKEN_SAVE_TIME_KEY);
                    if (oldTime!=null){
                        long otime=Long.parseLong(oldTime+"");
                        long now = System.currentTimeMillis();
                        if (now-otime>token.timeout()){
                            //清除超时数据 避免异常等情况造成的旧数据遗留问题
                            request.getSession(false).removeAttribute(DUPLICATE_TOKEN_KEY);
                            t=null;
                        }
                    }
                }

                if (null==t){
                    String uuid= UUID.randomUUID().toString();
                    long now = System.currentTimeMillis();
                    request.getSession().setAttribute(DUPLICATE_TOKEN_KEY,uuid);
                    request.getSession().setAttribute(DUPLICATE_TOKEN_SAVE_TIME_KEY,now);
                    log.info("进入方法：token="+uuid);
                }else {
                    throw new CustomException(TextConstants.REQUEST_REPEAT);
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
```

#### 3. 控制器 ####
```
/**
 * @description
 */
@RestController
public class TestController {
    
    @DuplicateSubmitToken
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

#### 4.总结：####
功能异常情况就处理了，但是总觉得这也太不智能了。而且麻烦，总觉得绕圈子了，能不能一出现异常就减标记移除呢？请看下一篇

作者：@天空蓝蓝的    www.lskyf.xyz 
版权所有，欢迎保留原文链接进行转载：)