# Java 自定义集合常量

> 作者：[YD_1989](https://blog.csdn.net/m0_58680865)、来源：Java 自定义集合常量：https://blog.csdn.net/m0_58680865/article/details/140238107

## 1、普通方法自定义集合常量信息

### 1、定义 Map 集合信息

方法一：使用静态代码块

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class JavaAPITest {
    public static final Map<String, String> CONSTANT_MAP_1 = new HashMap<>();
    static {
        CONSTANT_MAP_1.put("01", "同意");
        CONSTANT_MAP_1.put("02", "不同意");
        CONSTANT_MAP_1.put("03", "待定");
    }

    public static void main(String[] args) {
        Set<String> keys1 = CONSTANT_MAP_1.keySet();
        for (String key : keys1) {
            System.out.println("常量 Map 的 key：" + key + " ，值是：" + CONSTANT_MAP_1.get(key));
        }
    }
}
```

```
常量 Map 的 key：01 ，值是：同意
常量 Map 的 key：02 ，值是：不同意
常量 Map 的 key：03 ，值是：待定
```

***

方法二：简单定义 Map 常量

```java
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class JavaAPITest {
    public static final Map<String, String> CONSTANT_MAP_2 = new HashMap<>() {
        {
            put("A", "父母");
            put("B", "姑侄");
            put("C", "爷孙");
        }
    };

    public static void main(String[] args) {
        Set<String> keys2 = CONSTANT_MAP_2.keySet();
        for (String key : keys2) {
            System.out.println("常量 Map 的 key：" + key + " ，值是：" + CONSTANT_MAP_2.get(key));
        }
    }
}
```

```
常量 Map 的 key：A ，值是：父母
常量 Map 的 key：B ，值是：姑侄
常量 Map 的 key：C ，值是：爷孙
```



### 2、定义 List 集合信息

```java
import java.util.*;

public class JavaAPITest {
    public static final List<String> CONSTANT_LIST_1 = Arrays.asList("01", "02", "03");

    public static void main(String[] args) {
        String relationType = "01";
        if (CONSTANT_LIST_1.contains(relationType)) {
            System.out.println("集合包含 relationType：" + relationType);
        }
    }
}
```

```
集合包含 relationType：01
```



### 3、定义 Set 集合信息

```java
import java.util.*;

public class JavaAPITest {
    public static final Set<Integer> CONSTANT_SET_1 = new HashSet<>(Arrays.asList(1, 2, 3));

    public static void main(String[] args) {
        Integer num = 1;
        if (CONSTANT_SET_1.contains(num)) {
            System.out.println("集合包含数字：" + num);
        }
    }
}
```

```
集合包含数字：1
```



### 4、定义常量集合的漏洞

需要注意的是：普通的集合对象（如ArrayList、HashMap、HashSet等）都是可变集合对象，即便是定义为静态常量，也可以通过操作方法进行修改。所以，上面方法定义的集合常量，并不是真正意义上的集合常量。

比如，Arrays.asList 方法生成的 ArrayList 不能执行 add/remove/clear方法，但是可以执行 set 方法，也属于可变集合对象。

在上面的定义 List 集合信息时，定义了集合常量 CONSTANT_LIST_1，但是在使用时，可以通过 set 方法去改变集合的值，如下：

```java
import java.util.*;

public class JavaAPITest {
    public static final List<String> CONSTANT_LIST_1 = Arrays.asList("01", "02", "03");

    public static void main(String[] args) {
        CONSTANT_LIST_1.set(0, "09"); // 通过 set 方法修改集合下标是 0 的值
        if (CONSTANT_LIST_1.contains("01")) {
            System.out.println("集合值没被改变");
        } else {
            System.out.println("集合值被改变");
        }
    }
}
```

```
集合值被改变
```

由打印结果可知，集合的值已经被改变了。为了避免这种情况发生，在JDK中，Collections 工具类中提供一套方法，用于把可变集合对象变为不可变 (不可修改，修改时会抛出 UnsupportedOperationException 异常）集合对象。所以，可以利用这套方法定义集合静态常量。



## 2、通过 Collections 定义集合常量

```java
import java.util.*;

public class JavaAPITest {

    // 定义 List 集合常量
    public static final List<String> CONSTANT_LIST_2 = 
        Collections.unmodifiableList(Arrays.asList("01", "02", "03"));

    // 定义 Set 集合常量
    public static final Set<Integer> CONSTANT_SET_2 = 
        Collections.unmodifiableSet(new HashSet<>(Arrays.asList(1, 2, 3)));

    // 定义 Map 集合常量
    public static final Map<String, String> CONSTANT_MAP_3;

    static {
        CONSTANT_MAP_3 = Collections.unmodifiableMap(new HashMap<String, String>() {{
            put("01", "同意");
            put("02", "不同意");
        }});
    }

    public static void main(String[] args) {
        CONSTANT_LIST_2.set(0, "09");
        if (CONSTANT_LIST_2.contains("01")) {
            System.out.println("集合值没被改变");
        } else {
            System.out.println("集合值被改变");
        }

    }
}
```

```
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.base/java.util.Collections$UnmodifiableList.set(Collections.java:1349)
	at com.example.springboot.tech.test.JavaAPITest.main(JavaAPITest.java:24)
```



## 3、拓展：定义数组常量的最佳方式

一般情况下，定义数组常量如下：

```java
public static final int[] CONSTANT_ARRAY = new int[]{1,2,3};
```

但是，**可以通过下标修改数组值**，导致数组常量的值可变。所以，这种方法定义的数组常量，并不是一个真正意义上的数组常量。最佳法 “**私有数组常量 + 公有克隆方法**” 的解决方案。

如下代码所示：先定义一个私有数组常量，保证不会被外部类使用；在定义一个获取数组常量方法，并返回一个数组常量的克隆值。

```java
private static final int[] CONSTANT_ARRAY_2 = new int[]{1, 2, 3};

// 由于每次返回的是一个克隆数组，即便修改了克隆数组的常量值，也不会导致原始数组常量值的修改。
public static int[] getConstantArray() {
    return CONSTANT_ARRAY_2.clone();
}
```

