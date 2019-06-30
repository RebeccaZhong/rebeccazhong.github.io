Java中创建和销毁对象需要注意些什么？

Java中创建对象之静态工厂方法

# 考虑用静态工厂方法代替构造器

在`Boolean`类中，有这样一个方法：

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

这就是JDK用静态工厂方法代替构造器的一个使用。那么这样做有什么好处呢？
1. 静态工厂方法有名称，可以明显区分出不同构造方法的作用，构造器则体现不出来；
2. 不必在每次调用它时都创建一个新对象。在静态工厂方法中我们可以手动控制创建对象的个数。
3. 静态工厂方法可以返回原返回类型的任何子类型的对象。可以使返回的对象更加灵活。静态工厂方法构成了服务者提供框架(Service Provider Framework)的基础，例如JDBC API。服务者提供框架是指这样一个系统：多个服务提供者实现一个服务，系统为服务者提供者的客户端实现多个实现，并把他们从多个实现中解耦出来。
4. 在创建参数化类型实例的时候，他们使代码变得更加简洁。<br>通常我们在构造Map时，会这样写：`Map<String, List<String>> m = new HashMap<String, List<String>>();`，虽然在JDK7之后可以简化为`Map<String, List<String>> m = new HashMap<>();`，但假设一下，如果HashMap提供了一个这样的静态工厂：
```java
public static <K, V> HashMap<K, V> newInstance() {
  return HashMap<K, V>();
}
```
那么在创建时就可以写为：`Map<String, List<String>> m = HashMap.newInstance();`

静态工厂方法并不是没有任何缺点的：

1. 类如果不含公有的或受保护的构造器，就不能被子类化。
2. 静态工厂方法无法与其他的静态方法区分。如果命名不规范，很容易被误导。但我们可以用一些规范的命名来弥补这一劣势，如 `valueOf` 、 `getInstance` 、 `newInstance` 、 `getType` 等。

总结：

静态工厂方法和构造器各有优劣。通常静态工厂更合适，切忌第一反应就是提供公有的构造器，而不是优先考虑静态工厂。

# 参考

《Effective Java中文版 第2版》
