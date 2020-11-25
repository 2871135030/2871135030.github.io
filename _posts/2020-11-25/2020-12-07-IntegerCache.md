---
layout: post
title: IntegerCache使用的问题与源码解读
date:   2021-11-12 14:15:12
categories: [java]
---


#### IntegerCache使用的问题与源码解读

##### 1、使用中的代码演示
```
class IntegerTest {

    public static void main(String[] args) {
        Integer a = 88;
        Integer b = new Integer(88);
        Integer c = Integer.valueOf(88);
        System.out.println(a == b);
        System.out.println(a == c);
        System.out.println(a.equals(b));
        System.out.println(a.equals(c));
    }

}
```
执行结果如下

```
false
true
true
true
```
我们知道`Integer`的`equals`方法是通过`intValue`值进行判断，`a.equals(b)`与`a.equals(c)`结果均为`true`没有异议，但其中为什么`a==b`与`a==c`的结果不一致呢？

##### 2、原因分析
等号`==`是进行对象的比较，从结果说明a与c为同一个对象，b为不同的一个对象，可以通过`debug`调试代码得到验证，定义`Integer a = 88;`与`Integer c = Integer.valueOf(88);`的对象为同一个。

![image.png](https://upload-images.jianshu.io/upload_images/24398792-0167594e5e59a03b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

继续调试以下代码观察

```
    public static void main(String[] args) {
        Integer a = 88;
        Integer b = 88;
        Integer c = 88;
        Integer d = 188;
        Integer e = 188;
        Integer f = 188;
        System.out.println();
    }
```
可以发现`a、b、c`的对象为同一个，而`d、e、f`的对象则不是同一个。
![image.png](https://upload-images.jianshu.io/upload_images/24398792-507565512880009e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 3、源码解读（jdk8）
是什么原因造成这样的结果呢？我们看一下`Integer.valueOf()`的源码

```
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
> 这段代码意思就是判断传入值是否在缓存的区间范围内，可以理解为判断是否为热点数据，如果是直接返回缓存对象，否则`new`一个对象返回。

再继续看类`IntegerCache`源码，看看`low`及`high`这两个区间值是如何定义

```
private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```

从源码中我们可以看到`low`为-128是个固定值，而`high`的值默认为127但却可以通过参数的方式配置`java.lang.Integer.IntegerCache.high`，按这个逻辑我们把`high`配置到大于188的话，对象就会是同一个了，需注意`high`的最大取值。

> 源码中可以找到配置`high`的参数可以通过属性参数或`jvm`参数完成配置，即`-Djava.lang.Integer.IntegerCache.high=189`或`-XX:AutoBoxCacheMax=189`。
##### 4、实践总结
在`idea`中设置属性参数或`jvm`参数调试代码，可以看到设置`high`参数后`d、e、f`也为同一个对象了
![image.png](https://upload-images.jianshu.io/upload_images/24398792-a981699ccaf02f92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>通过一个整型对象的大小比较引申出系列问题的探讨以及思考，其实在`IntegerCache`类中我们还可以看到为什么只允许设置上限`high`呢？下限`low`为什么不能同样设置呢？是基于什么样的考虑呢？有官方的 [issue](https://bugs.openjdk.java.net/browse/JDK-6968657?page=com.atlassian.streams.streams-jira-plugin%3Aactivity-stream-issue-tab)，简单回复`没有必要`


#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

