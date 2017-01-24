---
layout: post
title: Springmvc restful使用swagger生成在线文档
date:   2016-04-17 23:23:34
categories: [java,spring,webservice]
---

## Spring4 mvc构建restful webservice应用使用swagger生成在线文档

本文接<a href="/java/spring/webservice/2016/04/17/spring-restful.html">上一篇已实现spring构建restful webservice应用</a>，本篇代码在上一篇的基础上完成，最后同样给出demo代码。

1、下载上文中的demo代码，或点此下载<a href="/rar/spring-restful.rar"><b>Demo代码下载</b></a>
{% highlight ruby %}
为了测试方便将demo代码中pom.xml配置文件中的标签 <scope>provided</scope> 去掉，直接执行以下命令运行服务端，可查看demo代码是否正常运行。
mvn clean compile exec:java -Dexec.mainClass="com.hode.JettyServer"
{% endhighlight %}

2、在pom.xml中添加swagger及相关依赖，如下
{% highlight ruby %}
		<dependency>
			<groupId>com.mangofactory</groupId>
			<artifactId>swagger-springmvc</artifactId>
			<version>0.9.4</version>
		</dependency>
		
		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>${jackson.version}</version>
		</dependency>

		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.6.5</version>
		</dependency>
		
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.6.5</version>
		</dependency>
{% endhighlight %}

3、编写一个swagger配置类，SwaggerConfig.java内容如下(注意目录结构src/main/java/com/hode/swagger/SwaggerConfig.java)
{% highlight ruby %}
package com.hode.swagger;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import com.mangofactory.swagger.configuration.SpringSwaggerConfig;
import com.mangofactory.swagger.models.dto.ApiInfo;
import com.mangofactory.swagger.plugin.EnableSwagger;
import com.mangofactory.swagger.plugin.SwaggerSpringMvcPlugin;

@Configuration
@EnableSwagger
public class SwaggerConfig {

	@Autowired
	private SpringSwaggerConfig springSwaggerConfig;
	
	public SwaggerSpringMvcPlugin customImplementation(){
		return new SwaggerSpringMvcPlugin(this.springSwaggerConfig)
				.apiInfo(apiInfo()).includePatterns(".*?");
	}
	
	private ApiInfo apiInfo(){
		ApiInfo apiInfo = new ApiInfo("Title","Description","Service","Email","Licence Type","Licence URL");
		return apiInfo;
	}
}

{% endhighlight %}


4、UserRestController修改如下，增加@ApiOperation及@ApiParam注解
{% highlight ruby %}
package com.hode.rest;

import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import com.wordnik.swagger.annotations.ApiOperation;
import com.wordnik.swagger.annotations.ApiParam;

@RestController
@RequestMapping("user")
public class UserRestController extends BaseController {

	@RequestMapping("getUser")
	@ApiOperation(value = "根据用户名获取用户",httpMethod="POST")
	public String getUser(@ApiParam(value="用户名") @RequestBody String username){
		return String.format("success[%s]",username);
	}

}

{% endhighlight %}


5、在applicationContext-mvc.xml文件中添加如下配置
{% highlight ruby %}
	<bean class="com.mangofactory.swagger.configuration.SpringSwaggerConfig" />
	
	<bean class="com.hode.swagger.SwaggerConfig" />
	
	<mvc:resources location="/" mapping="/**"/>
{% endhighlight %}

6、由于swagger的限制，需先将web.xml中配置的*.rest改为/*，结果如下（注：由于本人参与的项目restful接口后缀为.rest，如果全部去掉.rest涉及比较多所以考虑修改swagger源码，文章尾部介绍）
{% highlight ruby %}
	<servlet-mapping>
		<servlet-name>DispatcherServlet</servlet-name>
		<url-pattern>/*</url-pattern>
	</servlet-mapping>
{% endhighlight %}

7、下载swagger ui，下载地址https://github.com/swagger-api/swagger-ui，下载后将dist改为swagger并拷贝到webapp目录下，修改webapp/swagger/index.html
{% highlight ruby %}
url = "http://petstore.swagger.io/v2/swagger.json";
改为
url = "http://localhost/api-docs";
{% endhighlight %}

8、运行命令
{% highlight ruby %}
mvn clean compile exec:java -Dexec.mainClass="com.hode.JettyServer"
{% endhighlight %}

使用http://localhost/swagger/index.html即可以看到结果，可以Parameters中的value输入"hode"查看结果。截图如下
![tomcat配置ssl单向认证](/assets/20170124114945.png)，点击try it out可模拟请求。

到此简单使用结束，<a href="/rar/spring-restful-swagger.rar"><b>Demo代码下载</b></a>


{% highlight ruby %}
---------------分割线---------------
{% endhighlight %}


9、由于本人参与的项目restful接口均为.rest后缀，不能直接使用swagger，官网也未找到对应的修改参数，所以动手调整下源码，以便不修改原项目代码。

将web.xml中改回.rest，表示.rest后缀的请求才由mvc拦截，如下
{% highlight ruby %}
	<servlet-mapping>
		<servlet-name>DispatcherServlet</servlet-name>
		<url-pattern>*.rest</url-pattern>
	</servlet-mapping>
{% endhighlight %}
删除applicationContext-mvc.xml配置文件中添加的行
{% highlight ruby %}
	<mvc:resources location="/" mapping="/**"/>
{% endhighlight %}
修改webapp/swagger/index.html
{% highlight ruby %}
url = "http://localhost/api-docs";
改为
url = "http://localhost/api-docs.rest";
{% endhighlight %}

基本内容已修改完，再修改webapp/swagger/swagger-ui.js
查找 SwaggerSpecConverter.prototype.getAbsolutePath，修改此方法，在var location = docLocation;定义行前添加判断逻辑，
{% highlight ruby %}
  if(docLocation.endsWith('.rest')) {
      // remove .rest
	  docLocation = docLocation.replace('.rest','');
	  path += '.rest';
    }
var location = docLocation;
{% endhighlight %}
查找Operation.prototype.urlify，在此方法的 return url + requestUrl + querystring; 前添加如下
{% highlight ruby %}
requestUrl += '.rest';
return url + requestUrl + querystring;
{% endhighlight %}

<h4><a href="/rar/spring-restful-swagger-.rest.rar"><b>支持.rest后Demo代码下载</b></a></h4>

结束。
