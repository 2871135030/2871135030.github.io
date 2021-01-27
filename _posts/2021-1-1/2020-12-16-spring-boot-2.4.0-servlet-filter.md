---
layout: post
title: spring-boot-2.4.0中添加servlet filter的执行顺序问题
date:   2021-11-12 14:15:21
categories: [spring]
---

#### spring-boot-2.4.0中添加servlet filter的执行顺序问题

##### 1、先看源码中类`ServletContextInitializerBeans`初始化的过程

* 构造方法

```
	public ServletContextInitializerBeans(ListableBeanFactory beanFactory,
			Class<? extends ServletContextInitializer>... initializerTypes) {
		... //省略非相关代码
		addServletContextInitializerBeans(beanFactory);
		addAdaptableBeans(beanFactory);
		List<ServletContextInitializer> sortedInitializers = this.initializers.values().stream()
				.flatMap((value) -> value.stream().sorted(AnnotationAwareOrderComparator.INSTANCE))
				.collect(Collectors.toList());
		this.sortedList = Collections.unmodifiableList(sortedInitializers);
		logMappings(this.initializers);
	}
```

* 查看构造方法中调用的方法`addAdaptableBeans(beanFactory);`

```
	protected void addAdaptableBeans(ListableBeanFactory beanFactory) {
		MultipartConfigElement multipartConfig = getMultipartConfig(beanFactory);
		addAsRegistrationBean(beanFactory, Servlet.class, new ServletRegistrationBeanAdapter(multipartConfig));
		addAsRegistrationBean(beanFactory, Filter.class, new FilterRegistrationBeanAdapter());
		...... //省略非相关代码
	}
```

> 可以看到实现Filter.class的bean注入主要是通过`addAsRegistrationBean(beanFactory, Filter.class, new FilterRegistrationBeanAdapter());`这段代码实现的

* 继续查看方法`addAsRegistrationBean`

```
	protected <T> void addAsRegistrationBean(ListableBeanFactory beanFactory, Class<T> type,
			RegistrationBeanAdapter<T> adapter) {
		addAsRegistrationBean(beanFactory, type, type, adapter);
	}
	
	private <T, B extends T> void addAsRegistrationBean(ListableBeanFactory beanFactory, Class<T> type,
			Class<B> beanType, RegistrationBeanAdapter<T> adapter) {
		List<Map.Entry<String, B>> entries = getOrderedBeansOfType(beanFactory, beanType, this.seen);
        ...... //省略非相关代码
	}
```

* 继续查看方法`getOrderedBeansOfType`，此方法实现了`filter`的排序功能

```
	private <T> List<Entry<String, T>> getOrderedBeansOfType(ListableBeanFactory beanFactory, Class<T> type,
			Set<?> excludes) {
		String[] names = beanFactory.getBeanNamesForType(type, true, false);
		Map<String, T> map = new LinkedHashMap<>();
		for (String name : names) {
			if (!excludes.contains(name) && !ScopedProxyUtils.isScopedTarget(name)) {
				T bean = beanFactory.getBean(name, type);
				if (!excludes.contains(bean)) {
					map.put(name, bean);
				}
			}
		}
		List<Entry<String, T>> beans = new ArrayList<>(map.entrySet());
		beans.sort((o1, o2) -> AnnotationAwareOrderComparator.INSTANCE.compare(o1.getValue(), o2.getValue()));  //实现filter的排序功能
		return beans;
	}
	
```
	
* 继续查看`AnnotationAwareOrderComparator`的`compare`方法
	
```
	public int compare(@Nullable Object o1, @Nullable Object o2) {
		return doCompare(o1, o2, null);
	}

	private int doCompare(@Nullable Object o1, @Nullable Object o2, @Nullable OrderSourceProvider sourceProvider) {
	    //判断是否实现优先排序接口，若其中一个实现另一个未实现则可直接返回优先级
		boolean p1 = (o1 instanceof PriorityOrdered);
		boolean p2 = (o2 instanceof PriorityOrdered);
		if (p1 && !p2) {
			return -1;
		}
		else if (p2 && !p1) {
			return 1;
		}

        //均实现优先排序接口或均未实现排序接口
		int i1 = getOrder(o1, sourceProvider);
		int i2 = getOrder(o2, sourceProvider);
		return Integer.compare(i1, i2);
	}
```

> 大致的意思就是从`BeanFactory`中获取到`filter`对应的`bean`，使用`AnnotationAwareOrderComparator`类进行比较排序，接下来再看看排序的细节。



##### 2、`Ordered`接口与`@Order`注解的解读

> 从源码中可以看出使用`AnnotationAwareOrderComparator`对`filter`的排序是通过对实现了`Ordered`接口、`@Order`注解的解析出`order`值完成排序，如果两个都定义了的话优先取实现接口`Ordered`的排序值，示例如下:

```
@Order(20)
class DemoOrder implements Ordered{

    @Override
    public int getOrder() {
        return 10;
    }
    
}
```

> `DemoOrder`中既实现了`Ordered`接口也定义了`@Order`注解，其结果是取10，实现接口的优先判断返回，查看`findOrder`源码可以看到如果能从实现接口中获取到排序就已`return`返回了

* `AnnotationAwareOrderComparator`的`findOrder`

```
	protected Integer findOrder(Object obj) {
		Integer order = super.findOrder(obj);
		if (order != null) {
			return order;
		}
		return findOrderFromAnnotation(obj);
	}
```

##### 3、自定义`Filter`的两种方式

* 3.1、使用`@Component`注册为普通的`RegistrationBean`

```
@Order(10)
@Component
public class DemoFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        chain.doFilter(request, response);
    }

}
```

* 3.2、使用`@Webfilter`注解以及`@ServletComponentScan`注解

```
@WebFilter
public class DemoWebFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        chain.doFilter(request, response);
    }

}

//需同时在启动类中加入@ServletComponentScan
```

> 需注意，在使用`@Webfilter`注解时，若使用了`@Order`或继承了`Ordered`接口均无法改变`Filter`的顺序的，只能改变类的命名调整`Filter`的顺序

* 3.3、分析`@Webfilter`增加`@Order`注解或实现`Ordered`接口无效原因

> 我们回看类`ServletContextInitializerBeans`的构造方法中的`addServletContextInitializerBeans(beanFactory);`可以跟踪到若使用`@Webfilter`注册的bean，其`order`值相同均为默认优先级最低值`order=2147483647`，是由`AnnotationAwareOrderComparator`类的`findOrder`方法获取，那为什么不会获取到类中定义的实现`Ordered`的接口呢？接着分析一下类的注册过程

* 3.4、`addAdaptableBeans(beanFactory);`方法获取`FilterRegistrationBean`是通过`ServletContextInitializerBeans`的方法`addAsRegistrationBean`，如下
 ```
 	private <T, B extends T> void addAsRegistrationBean(ListableBeanFactory beanFactory, Class<T> type,
			Class<B> beanType, RegistrationBeanAdapter<T> adapter) {
			    // 省略非关键代码
				RegistrationBean registration = adapter.createRegistrationBean(beanName, bean, entries.size());
				int order = getOrder(bean);
				registration.setOrder(order); //获取到order值重新设置
				this.initializers.add(type, registration);
				// 省略非关键代码
			}
		}
	}
 ```

* 3.5、`addServletContextInitializerBeans(beanFactory);`方法获取`FilterRegistrationBean`是通过`AbstractBeanFactory`类的方法，如下

```
	public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		        // 省略非关键代码
				try {
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				// 省略非关键代码
		}
	}
```

> 可以看到两种方式的不同，`addAsRegistrationBean`重新设置了`order`排序

> 如果使用了`Ordered`接口或者`@Order`注解，则按优先级排序，如果没有则排到最后，如果有多个没有`Order`的接口，则按类名的字典排序。
 
 





#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

