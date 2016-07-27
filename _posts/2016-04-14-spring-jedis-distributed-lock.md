---
layout: post
title: Spring4+jedis实现分布式锁
date:   2016-04-14 15:32:57
categories: [java,spring,redis]
---

## Spring4+jedis实现分布式锁

本文实现接<a href="/java/spring/redis/2016/04/13/spring-jedis.html">Spring4+jedis基本操作</a>

1、编写一个简单的锁控制类，此处简单实现，暂忽略异常处理，释放资源异常等等。

{% highlight ruby %}
package com.hode;

import org.springframework.dao.DataAccessException;
import org.springframework.data.redis.connection.RedisConnection;
import org.springframework.data.redis.core.RedisCallback;
import org.springframework.data.redis.core.RedisTemplate;

public class RedisLock {
	
	private final static String key = "lock_key";
	
	public synchronized boolean acquire(RedisTemplate<String,Object> template) throws Exception {
		int timeout = 10;
		
		while(timeout>=0){
			boolean result = template.execute(new RedisCallback<Boolean>(){
				@Override
				public Boolean doInRedis(RedisConnection conn) throws DataAccessException {
					boolean r = conn.setNX(key.getBytes(),new byte[]{0});
					conn.expire(key.getBytes(), 10); //设置超时(秒)
					return r;
				}
			});
			if(result){
				return true;
			}
			timeout--;
			Thread.sleep(100);
		}
		return false;
	}

	public synchronized void release(RedisTemplate<String, Object> template) {
		template.delete(key);
	}

}

{% endhighlight %}


2、编写一个测试用例，完成锁的基本使用：

{% highlight ruby %}
package com.hode;

import org.apache.log4j.Logger;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JedisLockTest {

	static Logger log = Logger.getLogger(JedisLockTest.class);
	
	@Autowired
	RedisTemplate<String,Object> template; //此处RedisTemplate的key,value分别定义为String,Object
	
	@Test
	public void test() throws Exception{
		RedisLock rl = new RedisLock();
		if(rl.acquire(template)){
			log.info(Thread.currentThread().getName()+"获取锁成功");
			rl.release(template);
		}
	}
	
	@SuppressWarnings({ "resource", "unchecked" })
	public static void main(String[] args) throws Exception{
		ApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext.xml");
		RedisTemplate<String,Object> t = context.getBean(RedisTemplate.class);
		
		for(int i=0;i<1000;i++){
			new Thread(new Runnable(){
				@Override
				public void run() {
					try {
						RedisLock rl = new RedisLock();
						
						if(rl.acquire(t)){
							log.info(Thread.currentThread().getName()+"获取锁成功");
							rl.release(t);
						}else{
							//log.warn(Thread.currentThread().getName()+"获取锁失败");
						}
					} catch (Exception e) {
						e.printStackTrace();
					}
				}
			}).start();
		}
	}
	
}

{% endhighlight %}

结束。
