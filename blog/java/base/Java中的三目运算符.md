某日，同事给我展示了一块代码，问我有没有什么问题。（代码如下 代码块1）：
```java
int i = 1;
Boolean a = null;
boolean b = false;
System.out.println(i == 1 ? a : b);
```
集智慧与美貌于一身的我一眼就发现了其中的端倪：当然会报 NPE 了！

但是如果代码变成这样的呢（如下 代码块2）？
```java
int i = 1;
boolean a = true;
Boolean b = null;
System.out.println(i == 1 ? a : b);
```
上述代码块就是正常的。

为什么呢？

这就涉及到了Java中 三目运算符 和 自动装箱/拆箱 的问题了。

代码块1会报错是因为条件满足，取 a 的值。在取 a 的值时，三目运算符中既有基本类型，又有引用类型；JVM会帮我们把引用类型的数据转换为基本类型。通过查看 class 文件看到如下代码（代码块3）：
```java
int i = 1;
Boolean a = null;
boolean b = false;
System.out.println(i == 1 ? a.booleanValue() : b);
```
因为 a 是引用类型，此时调用 a 的方法就会报NPE了。

代码块2为什么不会报错呢？

此时三目运算符中的条件满足，直接就取 a 的值了，因为 a 是基本类型的，直接就拿来用了，所以程序正常。可以看到 class 文件如下（代码块4）：
```java
int i = 1;
boolean a = true;
Boolean b = null;
System.out.println(i == 1 ? a : b.booleanValue());
```

但在实际的业务场景中，我们的代码不会上上述一样是 i == 1 的写法，大部分情况下是动态判断的。如何避免踩进上述的坑中呢？

其实要解决这个问题也很简单，就是不让JVM帮我们拆箱。即把 a 和 b 两个对象都改为 Boolean 或 boolean，只要两个对象统一就好了。

避免采坑的代码：
```java
int i = 1;
Boolean a = Boolean.TRUE;
Boolean b = Boolean.FALSE;
System.out.println(i == 1 ? a : b);
```
或
```java
int i = 1;
boolean a = true;
boolean b = false;
System.out.println(i == 1 ? a : b);
```
##### 总结：
不止 Boolean/boolean 有这个问题，像 Byte / byte、Integer / int、Short / short、Long / long、Character / char、Float / float、Double / double 都会有这样的问题。只要代码规范一点，就不会坑到队友啦  ╮(￣▽￣)╭


最后祝大家码出好心情，码出幸福人生！
