| 语句     | 普通for循环                | for-each                   | Java8-forEach              |
| -------- | -------------------------- | -------------------------- | -------------------------- |
| return   | 跳出循环，直接返回方法     | 跳出循环，直接返回方法     | 跳过当前元素，继续执行循环 |
| break    | 跳出循环，继续执行方法     | 跳出循环，继续执行方法     | 不可用                     |
| continue | 跳过当前元素，继续执行循环 | 跳过当前元素，继续执行循环 | 不可用                     |



## for 循环

1. return：跳出循环，直接返回方法
2. break：跳出循环，继续执行方法
3. continue：跳过当前元素，继续执行循环

```java
List<Integer> integers = new ArrayList<>();
for (int i = 0 ; i < 10 ; i++) {
    if (i == 3) return; // 跳出循环，直接返回方法
    // if (i == 3) break; // 跳出循环，继续执行方法
    // if (i == 3) continue; // 跳过当前元素，继续执行循环
    integers.add(i);
    System.out.println(i);
}

for (int item : integers){
    System.out.println(item);
}
```



## for-each

同普通for循环，测试代码：

```java
List<Integer> integers = new ArrayList<>();
for (int i = 0 ; i < 5 ; i++) {
    integers.add(i);
    System.out.println(i);
}

for (int item : integers){
    if (item == 3) return;     // 跳出循环，直接返回方法
    // if (item == 3) break;   // 跳出循环，继续执行方法
    // if (item==3) continue;  // 跳过当前元素，继续执行循环
    System.out.println(item);
}
```



## forEach()【Java8】

不能使用 continue、break；可以使用 return：跳过当前元素，继续执行循环

```java
integers.forEach(integer -> {
    if (integer == 3) return;      // 跳过当前元素，继续执行循环
    // if (integer == 3) continue; // 编译错误：continue outside of loop
    // if (integer == 3) break;    // 编译错误：Break outside switch or loop
    System.out.println(integer);
});
```

