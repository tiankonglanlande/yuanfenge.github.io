---
layout: post
title: java redis模糊删除key
tags:
- java 
- redis
- 模糊删除key


categories: java  redis  模糊删除key 
description:  java redis模糊删除key
---
两个java redis模糊删除key实例
<!-- more -->
注释比较清楚直接看代码
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisTest {
     @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Autowired
    private RedisTemplate redisTemplate;

    
    /**
     * 使用redis模糊清除缓存
     */
    @Test
    public void testRedisCache(){
        String keys="test:";
        redisTemplate.opsForValue().set(keys+1,"this is a test content!",1000,TimeUnit.SECONDS);
        String content=redisTemplate.opsForValue().get(keys+1).toString();
        System.out.println("---------》获取到缓存的内容为："+content);
        redisTemplate.delete(redisTemplate.keys(keys+"*"));
        Object msg=redisTemplate.opsForValue().get(keys+1);
        if (msg==null){
            msg="没有了";
        }
        System.out.println("---------》移除后内容为："+msg);
    }
}
```
注意：删除的前缀应该是就近一级  eg:  key=test:aa:bb:12345 那么他的前缀应该是test:aa:bb:*        这样才能删除


##### 续集 #####

1.思路：使用模糊获取相关的key，然后根据key做更新删除操作

2.伪代码
```
String keys="test:group:user*";
Set<String> keysList = stringRedisTemplate.keys(keys);

keysList.forEach(i->{
    String key = i;
    String value=stringRedisTemplate.opsForValue().get(key);
    LogUtils.info("---------->"+value+"\n");
});
```
3.拿到key了，删除更新都就迎刃而解了

<br/>
<br/>

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：<a href="#">猿份哥</a> <br>
网址导航：<a href="http://www.lskyf.com" target="_blank">http://www.lskyf.com</a> <br>
个人博客：<a href="yuanfenge.lskyf.com" target="_blank">yuanfenge.lskyf.com</a> <br>
版权所有，欢迎保留原文链接进行转载：) 
还可以关注我们的公众号<br>
<img src="{{ site.assets }}/images/gongzonghao/天空唯美.jpg"/>