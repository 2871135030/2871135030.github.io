---
layout: post
title: Spring Solr客户端应用
date:   2016-05-29 14:32:58
categories: [spring,tools]
---

## Spring Solr客户端应用


本文按<a href="/tools/2016/05/28/solr-6.html">上篇</a>配置好环境，创建一个core，命令如下
{% highlight ruby %}
[root@localhost2 bin]# ./solr create -c article -force
{% endhighlight %}

1、创建core后初始化schema，分别有标题、作者、内容、发布日期、浏览量、评分。如下：
{% highlight ruby %}
curl -X POST -H 'Content-type:application/json' --data-binary '{    
"add-field":{"name":"title","type":"string","stored":true,"multiValued":false,"indexed":true},
"add-field":{"name":"author","type":"string","stored":true,"multiValued":false,"indexed":true},
"add-field":{"name":"content","type":"string","stored":true,"multiValued":false,"indexed":true},
"add-field":{"name":"publishDate","type":"date","stored":true,"multiValued":false,"indexed":true},
"add-field":{"name":"click","type":"int","stored":true,"multiValued":false,"indexed":true},
"add-field":{"name":"score","type":"double","stored":true,"multiValued":false,"indexed":true}
}' http://192.168.245.133:8983/solr/article/schema
{% endhighlight %}

2、编写pom.xml，内容如下

{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>spring-solr</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<spring.version>4.3.8.RELEASE</spring.version>
		<junit.version>4.12</junit.version>
		<log4j.version>1.2.17</log4j.version>
		<mongodb.version>1.9.0.RELEASE</mongodb.version>
		<slf4j.version>1.7.12</slf4j.version>
	</properties>

	<dependencies>

		<dependency>
		    <groupId>org.apache.solr</groupId>
		    <artifactId>solr-core</artifactId>
		    <version>5.5.4</version>
		</dependency>

		<dependency>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
			<version>1.1.1</version>
		</dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-solr</artifactId>
			<version>2.1.3.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.8.0</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
		</dependency>

		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.17</version>
		</dependency>

	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>

		</plugins>
	</build>

</project>
{% endhighlight %}

2、创建类Article，ArticleRepository，其中ArticleRepository类继承SolrCrudRepository，并定义一个方法（注意：springdata solr支持关键字在方法名中），代码如下：
{% highlight ruby %}
package com.hode.solr.model;

import java.io.Serializable;
import java.util.Date;

import org.apache.solr.client.solrj.beans.Field;
import org.springframework.data.annotation.Id;
import org.springframework.data.solr.core.mapping.Indexed;
import org.springframework.data.solr.core.mapping.SolrDocument;

@SolrDocument(solrCoreName = "article")
public class Article implements Serializable {

	@Id
	@Indexed
	private String id;
	
	@Indexed
	@Field
	private String title;
	
	@Indexed
	@Field
	private String author;
	
	@Indexed
	@Field
	private String content;
	
	@Indexed
	@Field
	private Date publishDate;
	
	@Indexed
	@Field
	private int click;
	
	@Indexed
	@Field
	private double score;
	
	//省略getter/setter

}

package com.hode.solr.repository;

import java.io.Serializable;
import java.util.List;

import org.springframework.data.solr.repository.SolrCrudRepository;

import com.hode.solr.model.Article;

public interface ArticleRepository extends SolrCrudRepository<Article, Serializable>{

	public List<Article> findByContentContaining(String keyword); //表示查找content字段包含keyword的记录
	
}

{% endhighlight %}

3、编写配置文件applicationContext.xml
{% highlight ruby%}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util" xmlns:solr="http://www.springframework.org/schema/data/solr"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd
        http://www.springframework.org/schema/data/solr http://www.springframework.org/schema/data/solr/spring-solr.xsd">

	<context:component-scan base-package="com.hode" />

	<solr:repositories base-package="com.hode.solr.repository" multicore-support="true"/>

	<solr:solr-client id="solrClient" url="http://192.168.245.133:8983/solr" />
	
	<bean id="solrTemplate" class="org.springframework.data.solr.core.SolrTemplate">
		<constructor-arg ref="solrClient" />
	</bean>
	

</beans>
{% endhighlight %}

4、编写ArticleTest类，完成相关功能测试，代码如下：
{% highlight ruby %}
package com.hode;

import java.util.Date;
import java.util.List;

import org.apache.log4j.Logger;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.hode.solr.model.Article;
import com.hode.solr.repository.ArticleRepository;

@ContextConfiguration(locations={"classpath*:application*.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class ArticleTest {

	private Logger log = Logger.getLogger(getClass());
	
	@Autowired
	private ArticleRepository repository;
	
	@Test
	public void testSave(){
		Article a = new Article();
		//a.setId(); 无需设置
		a.setTitle("this is a testing title");
		a.setAuthor("hode");
		a.setContent("this is content,testing content is here");
		a.setPublishDate(new Date());
		a.setClick(1);
		a.setScore(1.0);
		repository.save(a );
	}
	
	@Test
	public void clear(){
		repository.deleteAll();
	}
	
	@Test
	public void testFindByContent(){
		List<Article> list = repository.list("testing");
		for(Article a:list){
			log.info(a);
		}
	}
	
}
{% endhighlight %}
 
结束。
