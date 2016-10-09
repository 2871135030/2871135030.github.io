---
layout: post
title: Gradle构建spring项目
date:   2016-05-19 12:20:58
categories: [tools,spring]
---

## Gradle构建spring项目

接上一篇<a href="/tools/spring/2016/05/19/gradle-spring-project.html">Gradle安装与简单使用介绍</a>，本文介绍gradle构建一个spring项目

1、首先编写build.gradle配置文件，内容如下

```markdown
apply plugin:'java'
apply plugin:'eclipse'

repositories{
	mavenCentral()
}

dependencies{
	compile(
		'org.springframework:spring-core:4.0.7.RELEASE'
		'org.springframework:spring-context:4.0.7.RELEASE'
	)
	testCompile(
		'junit:junit:4.11'
	)
}
```

<font color="red">注意：使用了maven的中央仓库，以及依赖了spring-core、junit。注意相应的格式</font>

执行命令完成目录创建
mkdir -p src/{main,test}/{java,resources}

2、执行命令 gradle eclipse 完成初始化。

3、编写类完成测试（按照包的结构保存类）

UserService.java(目录/src/main/java/com/hode/service/)

```markdown
package com.hode.service;

import org.springframework.stereotype.Service;

@Service
public class UserService{
	
	public void info(){
		System.out.println("this is UserService info function");
	}
	
}
```

Config.java(目录/src/main/java/com/hode/)

```markdown
package com.hode;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("com.hode")
public class Config {

}
```

TestClazz.java(目录/src/test/java/com/hode/)

```markdown
package com.hode;

import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;

import com.hode.service.UserService;

public class TestClazz{
	
	@Test
	public void test(){
		AbstractApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
		System.out.println(context);
		UserService service = context.getBean(UserService.class);
		service.info();
	}
	
}
```

4、执行命令gradle test 完成测试。

测试结果可查看文件build/test-results/test/binary/output.bin

完毕。

