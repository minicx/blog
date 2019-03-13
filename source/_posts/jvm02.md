---
title: jvm02-OOM
date: 2019-03-13 12:24:33
tags: JVM
---
## 常见OOM简介

### 1.堆溢出

限制堆大小
`-verbose:gc -Xms2M -Xmx2M -Xmn1M -Xss300K -XX:+PrintGCDetails -XX:SurvivorRatio=6 -XX:+HeapDumpOnOutOfMemoryError`
无限制填充堆
```
List<String> list = new ArrayList<>();
while (true) {
    list.add(String.valueOf(Math.random()));
}
```
出现 Java heap spac
```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at sun.misc.FloatingDecimal$BinaryToASCIIBuffer.toJavaFormatString(FloatingDecimal.java:302)
	at sun.misc.FloatingDecimal.toJavaFormatString(FloatingDecimal.java:70)
	at java.lang.Double.toString(Double.java:204)
	at java.lang.String.valueOf(String.java:3141)
	at com.shitu.common.base.JvmTest.main(JvmTest.java:16)
```

### 2.虚拟机栈和本地方法溢出

无限制调用自身
```$xslt
    public static void main(String[] args) {
        for (;;)
            new JvmTest().getSelf();
    }

    public void getSelf() {
        this.getSelf();
    }
```
出现StackOverflowError
```$xslt
Exception in thread "main" java.lang.StackOverflowError
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
	at com.shitu.common.base.JvmTest.getSelf(JvmTest.java:25)
```
无限线程
```$xslt
 public static void main(String[] args) {
        for (; ; ) {
            new Thread(() -> {
                try {
                    Thread.sleep(100000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
```
结果
```
Exception in thread "main" [Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4199K->4199K(1056768K)], 0.0356043 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4199K->4199K(1056768K)], 0.0424894 secs] [Times: user=0.02 sys=0.00, real=0.05 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4199K->4199K(1056768K)], 0.0416804 secs] [Times: user=0.03 sys=0.00, real=0.04 secs] 
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message can't create byte arrau at JPLISAgent.c line: 813
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1227K(1536K), [Metaspace: 4200K->4200K(1056768K)], 0.0290505 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 512K->511K(512K)] 1228K->1227K(1536K), [Metaspace: 4202K->4202K(1056768K)], 0.0254690 secs] [Times: user=0.03 sys=0.00, real=0.02 secs] 
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4202K->4202K(1056768K)], 0.0395328 secs] [Times: user=0.02 sys=0.01, real=0.04 secs] 
[Full GC (Allocation Failure) *** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message can't create byte arrau at JPLISAgent.c line: 813
[PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4202K->4202K(1056768K)], 0.0297203 secs] [Times: user=0.02 sys=0.00, real=0.03 secs] 
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1227K(1536K), [Metaspace: 4217K->4217K(1056768K)], 0.0535506 secs] [Times: user=0.03 sys=0.00, real=0.05 secs] 
[Full GC (Ergonomics) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4217K->4217K(1056768K)], 0.0346813 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1228K(1536K), [Metaspace: 4217K->4217K(1056768K)], 0.0429982 secs] [Times: user=0.03 sys=0.00, real=0.05 secs] 
[Full GC (Ergonomics) *** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message can't create byte arrau at JPLISAgent.c line: 813
[PSYoungGen: 716K->716K(1024K)] [ParOldGen: 511K->511K(512K)] 1228K->1227K(1536K), [Metaspace: 4223K->4223K(1056768K)], 0.0235572 secs] [Times: user=0.02 sys=0.01, real=0.03 secs] 
[Full GC (Ergonomics) java.lang.OutOfMemoryError: Java heap space
```
### 3.常量池溢出

### 4.方法区溢出

### 5.本机直接内存溢出

结果测试,3,4,5在jdk8中不会java.lang.OutOfMemoryError: PermGen space错误了.
因为在jdk8中使用了元空间代替了永久代.

元空间是方法区的在HotSpot jvm 中的实现，方法区主要用于存储类的信息、常量池、方法数据、方法代码等。方法区逻辑上属于堆的一部分，但是为了与堆进行区分，通常又叫“非堆”。

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用本地内存。，理论上取决于32位/64位系统可虚拟的内存大小。可见也不是无限制的，需要配置参数。

测试限制
`-XX:MetaspaceSize=10M -XX:MaxMetaspaceSize=10M -verbose:gc -Xms200M -Xmx200M -Xmn10M -Xss300K -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError`
测试代码
```$xslt
 List<String> list = new ArrayList();
 int i = 0;
 for (; ; ) {
     list.add(String.valueOf(i++).intern());
 }
```
结果抛出GC overhead limit exceeded
```markdown
[Full GC (Ergonomics) [PSYoungGen: 4095K->4095K(7168K)] [ParOldGen: 194273K->194273K(194560K)] 198369K->198369K(201728K), [Metaspace: 3277K->3277K(1056768K)], 0.7411740 secs] [Times: user=1.89 sys=0.00, real=0.74 secs] 
[Full GC (Ergonomics) [PSYoungGen: 4095K->4095K(7168K)] [ParOldGen: 194275K->194275K(194560K)] 198371K->198371K(201728K), [Metaspace: 3277K->3277K(1056768K)], 0.7874470 secs] [Times: user=2.03 sys=0.00, real=0.79 secs] 
java.lang.OutOfMemoryError: GC overhead limit exceeded
Dumping heap to java_pid11091.hprof ...
Heap dump file created [261373312 bytes in 1.441 secs]
Exception in thread "main" [Full GC (Ergonomics) java.lang.OutOfMemoryError: GC overhead limit exceeded
	at java.lang.Integer.toString(Integer.java:401)
[PSYoungGen: 4096K->0K(7168K)] [ParOldGen: 194292K->456K(194560K)] 198388K->456K(201728K), [Metaspace: 3303K->3303K(1056768K)], 0.0868391 secs] [Times: user=0.08 sys=0.00, real=0.09 secs] 
	at java.lang.String.valueOf(String.java:3099)
Heap
	at com.shitu.common.base.JvmTest.main(JvmTest.java:45)
 PSYoungGen      total 7168K, used 116K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 4096K, 2% used [0x00000000ff600000,0x00000000ff61d3c8,0x00000000ffa00000)
  from space 3072K, 0% used [0x00000000ffd00000,0x00000000ffd00000,0x0000000100000000)
  to   space 3072K, 0% used [0x00000000ffa00000,0x00000000ffa00000,0x00000000ffd00000)
 ParOldGen       total 194560K, used 456K [0x00000000f3800000, 0x00000000ff600000, 0x00000000ff600000)
  object space 194560K, 0% used [0x00000000f3800000,0x00000000f3872318,0x00000000ff600000)
 Metaspace       used 3309K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 363K, capacity 388K, committed 512K, reserved 1048576K
```

