> 来源：7 段小代码，玩转Java程序常见的崩溃场景：https://mp.weixin.qq.com/s/6fGlAdfm0f3tnbiDyGvqoA

Java程序是基于GC的，在启动初始，就申请了足量的内存池，再加上JIT等编译器的实时优化，速度并不比直接用C++语言写的慢。Java语言同时由于反射和可观测等特点，再加上JFR这种神器，在发生问题的时候比二进制文件更容易找到它的根源。

最近在看RCA（Root Cause Analysis）的东西，不小心发现了yCrash这么个东西。它的几段问题小代码写的非常典型，我们可以稍微看一下，来看看Java应用程序常见的几个崩溃场景。



# 1、堆空间溢出

OOM 一般是内存泄漏引起的，表现在 GC 日志里，一般情况下就是 GC 的时间变长了，而且每次回收的效果都非常一般。GC 后，堆内存的实际占用呈上升趋势。

下面的代码是死循环，持续向HashMap里塞数据，由于myMap属于GCRoots，始终得不到释放，所以它最终的结果就是OOM。

```java
import java.util.HashMap;
public class OOMDemo {
    static HashMap<Object, Object> myMap = new HashMap<>();
    public static void start() throws Exception { 
        while (true) {
            myMap.put("key" + counter, "Large stringgggggggggggggggggggggggggggg"
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + "ggggggggggggggggggggggggggggggggggggggggggggggggggggg" 
                      + counter);         
            ++counter;
        }
    }   
}
```



# 2、内存泄漏

内存泄漏和内存溢出是一个道理，不同的是它的语意。

内存溢出可能是由于请求量过高，或者真实的业务需求需要所造成的后果，而内存溢出属于未知的、超出期望的OOM情况。

我们可以使用上面同样的代码达到这个目的。

在现实情况中，内存泄漏通常都非常的隐蔽，需要借助Mat等工具才能找到根本原因。jmap、pmap等是常用的工具。

比如，如果你忘记了重写对象的hashCode和equals方法，就会产生内存泄漏。

```java
import java.util.HashMap;
import java.util.Map;
public class HashMapLeakDemo {
    public static class Key {
        String title;
        public Key(String title) {
            this.title = title;
        }
    }
    public static void main(String[] args) {
        Map<Key, Integer> map = new HashMap<>();
        map.put(new Key("1"), 1);
        map.put(new Key("2"), 2);
        map.put(new Key("3"), 2);
        Integer integer = map.get(new Key("2"));
        System.out.println(integer);
    }
}
```



# 3、CPU飙升

直接一个死循环，就可以把CPU干死。

```java
public class CPUSpikeDemo {
    public static void start() {
        new CPUSpikerThread().start();
        new CPUSpikerThread().start();
        new CPUSpikerThread().start();
        new CPUSpikerThread().start();
        new CPUSpikerThread().start();
        new CPUSpikerThread().start();
        System.out.println("6 threads launched!");
    }
}
public class CPUSpikerThread extends Thread {
    @Override
    public void run() {
        while (true) {
            // Just looping infinitely
        }
    }
}
```

获取问题代码通常可以使用下面的方法。

- （1）使用 top 命令，查找到使用 CPU 最多的某个进程，记录它的 pid。使用 Shift + P 快捷键可以按 CPU 的使用率进行排序。
- （2）再次使用 top 命令，加 -H 参数，查看某个进程中使用 CPU 最多的某个线程，记录线程的 ID。
- （3）使用 printf 函数，将十进制的 tid 转化成十六进制。
- （4）使用 jstack 命令，查看 Java 进程的线程栈。
- （5）使用 less 命令查看生成的文件，并查找刚才转化的十六进制 tid，找到发生问题的线程上下文。



# 4、线程泄漏

线程资源是昂贵的。如果你不停的创建线程，系统资源很快就会被耗尽。下面的代码一直不停的创建线程，如果同时请求压力比较大的话，多数能搞死宿主机。

```java
public class ThreadLeakDemo {
    public static void start() {
        while (true) {
            new ForeverThread().start();
        }
    }
}
public class ForeverThread extends Thread {
    @Override
    public void run() {
        // Put the thread to sleep forever, so they don't die.
        while (true) {
            try {
                // Sleeping for 10 minutes repeatedly
                Thread.sleep(10 * 60 * 1000);
            } catch (Exception e) {}
        }
    }
}
```

这是暴力啊，这和每个请求创建一个线程，或者创建一个线程池的后果是一样的。

```
java.lang.OutOfMemoryError: unable to create new native thread
```



# 5、死锁

死锁代码一般不会发生，但一旦发生还是非常严重的，相关的业务可能就跑不动了。

```java
public class DeadLockDemo {
    public static void start() {
        new ThreadA().start();
        new ThreadB().start();
    }
}

public class ThreadA extends Thread {
    @Override 
    public void run() {
        CoolObject.method1();
    }
}

public class ThreadB extends Thread {
    @Override 
    public void run() {
        HotObject.method2();
    } 
}

public class CoolObject {
    public static synchronized void method1() {
        try {
            // Sleep for 10 seconds
            Thread.sleep(10 * 1000);
        } catch (Exception e) {}
        HotObject.method2();
    }
}

public class HotObject {
    public static synchronized void method2() {
        try {
            // Sleep for 10 seconds
            Thread.sleep(10 * 1000);
        } catch (Exception e) {}
        CoolObject.method1();
    } 
}
```

死锁属于比较严重的一种情况，jstack 会以明显的信息进行提示。当然，关于线程的 dump，也有一些线上分析工具可以使用。比如fastthread，但也需要你先了解这些情况发生的意义。



# 6、栈溢出

栈溢出不会造成 JVM 进程死亡，危害“相对较小”。下面是一个简单的模拟栈溢出的代码，只需要递归调用就可以了。

```java
public class StackOverflowDemo {
    public void start() {
        start();
    }
}
```

通过 -Xss 参数可以设置虚拟机栈的大小。比如下面的命令就是设置栈大小为 128K。

```bash
-Xss128K
```

如果你的应用经常发生这种情况，可以试着调大这个值。但一般都是因为程序错误引起的，最好检查一下自己的代码。



# 7、Blocked线程

BLOCKED是一个比较严重的线程状态，当后端的服务处理时间非常长，请求的线程就会进入等待状态。这时候通过jstack来获取堆栈，就会发现线程处于阻塞状态。它阻塞在对锁的获取上（wating to lock）

```java
public class BlockedAppDemo {
    public static void start() {
        for (int counter = 0; counter < 10; ++counter) {
            // Launch 10 threads.
            new AppThread().start();
        }
    }
}
public class AppThread extends Thread {
    @Override
    public void run() {
        AppObject.getSomething();
    }
}
public class AppObject {
    public static synchronized void getSomething() {
        while (true) {
            try {
                Thread.sleep(10 * 60 * 1000);
            } catch (Exception e) {}
        }
    }
}
```

一旦频繁发生这种情况，就证明你的程序相应太慢了。如果CPU资源还有剩余，可以尝试着增加请求的线程数，比如tomcat的最大线程数。