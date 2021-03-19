---
layout: post
title: jvm常见的几种异常与参数限制
date:   2021-11-12 14:15:25
categories: [tools]
---

##### jvm常见的几种异常与参数限制

###### 1、java.lang.StackOverflowError

> `StackOverflowError`一般是由于方法的递归层次太深，导致方法栈溢出异常，可以通过调整参数`-Xss2048k`增加栈空间，但并不能真正解决问题的根源，需从业务场景上分析调用是否合理

* 示例代码

```
public static void main(String[] args) throws Exception {
    //-Xss2048k 改变栈空间大小，默认为1024k
    //java.lang.StackOverflowError
    try {
        stackOverflow();
    }finally {
        System.out.println(i);
    }
}

public static void stackOverflow() {
    i++;
    stackOverflow();
}
```

> 在通过调整参数`-Xss`大小后，可以观察到调用栈的深度有较明显的变化（`i`的值可以侧面反映栈的深度）

###### 2、java.lang.OutOfMemoryError: Java heap space

> 堆内存溢出，最简单的方式就是定义一个大对象，在堆内存中放不下即可显现

* 示例代码

```
//定义一个2M的数组，若限定-Xmx12m时即会出现大对象溢出的问题
int[] i = new int[2*1024*1024];
```

###### 3、java.lang.OutOfMemoryError: GC overhead limit exceeded

* 示例代码

> 从字面意思可以看出是`GC`花销超出了限制（由于`JVM`花费太长时间执行`GC`且只能回收很少的堆内存时抛出的）

```
//限制参数-Xmx12m
HashMap map = new HashMap();
for(int i=Integer.MIN_VALUE;i<Integer.MAX_VALUE;i++){
    map.put(i,i);
}
```

> 可以调大-Xmx参数，则等待发生此异常的时间变大（延后触发gc），所以在实际应用中要仔细检查对象是否不可回收，循环依赖等

###### 4、java.lang.OutOfMemoryError: Direct buffer memory

> 直接内存溢出，可使用`sun.misc.VM.maxDirectMemory()`获取到当前直接内存的大小，非固定值，根据`-Xmx`取值的大小变化计算

* 示例代码

```
//可通过参数 -XX:MaxDirectMemorySize=512M 改变直接内存大小
Map map = new HashMap<>();
while(true) {
    System.out.println(sun.misc.VM.maxDirectMemory() / 1024 / 1024 + "M"); //默认大小
    ByteBuffer buf = ByteBuffer.allocateDirect(1024 * 1024 * 1024);
    map.put(buf.toString(),buf);
    Thread.sleep(1000);
}
```

###### 5、java.lang.OutOfMemoryError: unable to create new native thread

> 无法创建更多的线程，超出线程创建的总数，总的线程数可根据各内存计算大致得出，需要调大`jvm`内存才行模拟，不然会先报内存不足`java.lang.OutOfMemoryError: Java heap space`

* 示例代码

```
//jvm参数调整为-Xmx1024m
int i=1;
while(true){
    new Thread(()->{
        try {
            Thread.sleep(Integer.MAX_VALUE);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();
    System.out.println(i++);
}
```

###### 6、java.lang.OutOfMemoryError: Metaspace

> `-XX:MaxMetaspaceSize=512M`Metaspace空间越小可容纳的产生的类就越少

* 示例代码

```
//-XX:MaxMetaspaceSize=512M
for (int i = 0; i < 1000000000; i++) {
    Class c = cp.makeClass("com.freeok.Meta" + i).toClass();
            System.out.println(c.getName());
}
```

###### 7、java.lang.OutOfMemoryError: Requested array size exceeds VM limit

> 数组最大长度限制

* 示例代码

```
Integer[] array = new Integer[Integer.MAX_VALUE];
```




#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

