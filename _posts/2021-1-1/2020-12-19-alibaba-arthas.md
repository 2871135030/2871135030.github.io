---
layout: post
title: Alibaba-arthas在线诊断工具基本使用方法
date:   2021-11-12 14:15:24
categories: [tools]
---

##### Alibaba-arthas在线诊断工具基本使用方法

> 从[github](https://github.com/alibaba/arthas)中下载最新的发布包`arthas-bin.zip`，解压到指定目录后使用命令`java -jar arthas-boot.jar`并指定`java`进程`id`即可使用

###### 1、使用`options`命令设置输入输出或异常的信息使用`json`格式化

```
options json-format true
```

> 设置成功后可使用`options`命令检查是否生效，但`json`格式对异常的输出不大友好

###### 2、使用`watch`命令观察方法的输入输出参数或异常信息

* 查看输入参数，*号表示匹配任意的方法(如指定test则为所有方法名为test方法，不区分方法的参数)

```
watch com.freeok.controller.FreeController * params
```

* 查看输入输出及异常`-n`表示只执行一次，`-x`表示入参和返回结果的展开层次为5层

```
watch com.freeok.controller.FreeController * "{params,returnObj,throwExp}" -x 5 -n 1
```

* 查看请求耗时超过100ms时的请求信息

```
watch com.freeok.controller.FreeController send "{params,returnObj,throwExp}" '#cost>100' -x 5 -n 1
```

* 增加条件第一个参数为free时才输出

```
watch com.freeok.controller.FreeController send "{params,returnObj,throwExp}" '#cost>100 && params[0]=="free"' -x 5 -n 1
```

* 增加第二个参数大于10时才输出

```
watch com.freeok.controller.FreeController send "{params,returnObj,throwExp}" '#cost>100 && params[0]=="free" && params[1]>10' -x 5 -n 1
```

> 其他条件`&& (params[1]>50 || params[1]<10 )`


* 观察方法调用前后参数的区别

```
whatch --help 可以看到 -b 指调用前 -s 指调用成功后
watch com.freeok.controller.FreeController send "{params,returnObj,throwExp}" -b -s
```

> 若输入输出为对象，需要`options`配置为`json`格式，才能在控制台直观地查看参数，此时`-x`参数意义不大，但如需要观察方法的调用异常堆栈，最好将`json-format`设置为`false`且设置`-x 5`

> 慎用通配符，将需`watch`的类及方法限制在预知且可按的范围内


###### 3、使用`tt(timetunnel)`命令观察方法的耗时

> `tt`命令会记录每次方法调用的信息，和`watch`有些相似但是它能记录下各个时间点的调用信息，之后随时查看，还可以`replay`重放这次调用。

```
tt  -t com.freeok.controller.FreeController send

重放请求
tt -play -i 1010
```

###### 4、使用`trace`命令观察方法调用栈的耗时

```
打印超过100ms的请求方法栈
trace com.freeok.controller.FreeController send '#cost>100'  
```

> 可灵活运用按指定的一些入参判断是否打印方法栈耗时，准确定位出现性能问题的方法


###### 5、使用`monitor`命令观察方法调用情况

> `monitor`命令可以监控方法的执行情况，比如调用成功次数、失败次数、失败率、平均执行时间等，可通过`-c`参数修改输出频率，`5s`输出一次结果，默认为`120s`

```
monitor -c 5 com.freeok.controller.FreeController send
```

###### 6、ognl(Object-Graph Navigation Language)表达式使用

```
可通过ognl调用类的静态方法
ognl '@com.freeok.controller.FreeController@getI()'
ognl '@com.freeok.controller.FreeController@setI(2)'

访问静态变量
ognl '@com.freeok.controller.FreeController@i'
```

###### 7、redefine实现热加载

> `redefine`可以实现代码热更新，动态给运行中的程序增加代码段（但不能改变方法名或参数名或类的字段），即类似`idea`中的`Reload Changed Classes`功能

```
[arthas@7276]$ redefine c:\\arthas-bin\\FreeController.class
redefine success, size: 1, classes:
com.freeok.controller.FreeController
[arthas@7276]$
```

> 如修改字段会报出`redefine error! java.lang.UnsupportedOperationException: class redefinition failed: attempted to change the schema (add/remove fields)`异常



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

