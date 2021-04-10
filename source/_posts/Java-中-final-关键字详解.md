---
title: Java 中 final 关键字详解
date: 2021-04-03 00:31:31
categories: Java
tags: Java
---

在 Java 中，final 关键字可以修饰的东西比较多，很容易混淆，在这里记录一下。主要从功能上说明一下，不做过多的代码演示。

### final 关键字用途
#### 1. final 变量
凡是对成员变量或者本地变量(在方法中的或者代码块中的变量称为本地变量)声明为final的都叫作final变量。final变量经常和static关键字一起使用，作为常量。用final关键字修饰的变量，只能进行一次赋值操作，并且在生存期内不可以改变它的值。

#### 2. final方法参数
如果 final 关键字修饰方法参数时，方法中是不能改变该参数的，举例如下：

```java
public void testFunc(final Integer i) {
    i = 20;  // 报错: Cannot assign a value to final variable 'i'
    System.out.println(i);
}
```

#### 3. final 方法
final也可以声明方法。方法前面加上final关键字，代表这个方法不可以被子类的方法重写。如果你认为一个方法的功能已经足够完整了，子类中不需要改变的话，你可以声明此方法为final。final方法比非final方法要快，因为在编译的时候已经静态绑定了，不需要在运行时再动态绑定。


#### 4. final 类
使用final来修饰的类叫作final类。final类通常功能是完整的，它们不能被继承。Java 中有许多类是final的，譬如String, Interger以及其他包装类。

#### 5. final 关键字好处
- final关键字提高了性能。JVM和Java应用都会缓存final变量。
- final变量可以安全的在多线程环境下进行共享，而不需要额外的同步开销。
- 使用final关键字，JVM会对方法、变量及类进行优化。

#### final 和 static
两者得分开解释，不能混为一谈，这样更容易理解，static 作用于成员变量用来表示只保存一份副本，是类变量而已，而final的作用是用来保证变量不可变。两者用在一起表示类的不可以修改值的类变量。

```java
public final class Employee {
    public static final String SERVER = "example.com:9090";
    ...
}
```

### 总结
![在这里插入图片描述](/images/java-final-keyword.png)
### 参考文档
[https://www.jb51.net/article/157603.htm](https://note.youdao.com/)

