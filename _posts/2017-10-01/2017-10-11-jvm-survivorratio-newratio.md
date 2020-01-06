---
layout: post
title: JVM SurvivorRatio NewRatio参数配置说明
date:   2021-11-12 14:13:00
categories: [jvm]
---

## JVM SurvivorRatio NewRatio参数配置说明


1、SurvivorRatio参数说明

JVM参数`-XX:SurvivorRatio`控制了两个survivor空间(from&to)的大小，比如`-XX:SurvivorRatio=6`表示每个survivor空间和eden空间的比例为1:6，每个survivor空间则占年轻代的八分之一,
下面用代码演示一下。

MainTest.java
```
public class MainTest{
	
	public static void main(String[] args){
		System.out.println("test is ok");
	}
	
}
```

windows下执行以下命令
```
C:\temp1>javac MainTest.java & java -XX:NewSize=80M -XX:SurvivorRatio=6 -XX:+PrintGCDetails MainTest
test is ok
Heap
 PSYoungGen      total 71680K, used 3686K [0x00000000eb380000, 0x00000000f0380000, 0x0000000100000000)
  eden space 61440K, 6% used [0x00000000eb380000,0x00000000eb719b30,0x00000000eef80000)
  from space 10240K, 0% used [0x00000000ef980000,0x00000000ef980000,0x00000000f0380000)
  to   space 10240K, 0% used [0x00000000eef80000,0x00000000eef80000,0x00000000ef980000)
 ParOldGen       total 6144K, used 0K [0x00000000c1a00000, 0x00000000c2000000, 0x00000000eb380000)
  object space 6144K, 0% used [0x00000000c1a00000,0x00000000c1a00000,0x00000000c2000000)
 Metaspace       used 2618K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 283K, capacity 386K, committed 512K, reserved 1048576K

C:\temp1>
```

可以看到eden区为60M，Survivor区from为10M，Survivor区to为10M。



2、NewRatio参数说明

依然使用上面的例子


```
C:\temp1>javac MainTest.java & java -XX:SurvivorRatio=6 -XX:+PrintGCDetails  -XX:NewRatio=3 MainTest
test is ok
Heap
 PSYoungGen      total 14336K, used 1809K [0x00000000f0680000, 0x00000000f1680000, 0x0000000100000000)
  eden space 12288K, 14% used [0x00000000f0680000,0x00000000f0844638,0x00000000f1280000)
  from space 2048K, 0% used [0x00000000f1480000,0x00000000f1480000,0x00000000f1680000)
  to   space 2048K, 0% used [0x00000000f1280000,0x00000000f1280000,0x00000000f1480000)
 ParOldGen       total 49152K, used 0K [0x00000000c1a00000, 0x00000000c4a00000, 0x00000000f0680000)
  object space 49152K, 0% used [0x00000000c1a00000,0x00000000c1a00000,0x00000000c4a00000)
 Metaspace       used 2619K, capacity 4486K, committed 4864K, reserved 1056768K
  class space    used 283K, capacity 386K, committed 512K, reserved 1048576K

C:\temp1>
```

可以看到设置了参数-XX:NewRatio=3表示年轻代(eden+from+to)与老年代的比较为1:3

结束。。。
