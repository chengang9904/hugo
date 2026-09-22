+++
date = '2026-09-22T21:30:00+08:00'
draft = false
title = 'Java 单例模式的几种写法'
description = 'Java · 设计模式、并发'
categories = ['Java 学习']
tags = ['Java', '设计模式', '并发', 'JVM']
+++

单例要保证一个类在整个进程里只有一个实例，并提供一个全局访问点。写法本身只有几行，真正的分歧在于实例什么时候创建、多线程同时创建怎么办、以及创建出来之后还能不能被别的手段绕过。

> 单例的难点从来不是"只 new 一次"，而是在多线程、反射、反序列化三种场景下依然只有一个实例。

所有写法的共同前提是构造器私有，否则外部随时可以 `new` 出第二个。

## 方法一：饿汉式

类加载时就把实例创建好，之后直接返回。

JVM 保证类的初始化过程（执行静态变量赋值和静态代码块）是线程安全的，所以这里不需要任何加锁。

```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

代价是没有懒加载：只要这个类被加载，实例就一定会创建。如果实例的构造很重（读配置、建连接），而程序跑完可能根本用不上它，这部分开销就浪费了。

## 方法二：懒汉式加同步方法

第一次调用时才创建，用 `synchronized` 保证只有一个线程能走进创建逻辑。

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

问题是锁加在了整个方法上。实例只需要创建一次，但之后每次读取都要竞争同一把锁，高并发下这是一个明确的瓶颈。

## 方法三：双重检查锁

思路是把锁的范围缩到只有"可能需要创建"的那一次：

- 第一次判空在锁外，实例已经存在时直接返回，不走同步。
- 进入同步块后再判一次空，防止多个线程同时通过了第一次检查。

```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

`volatile` 不能省。`instance = new Singleton()` 不是一个原子操作，它至少包含三步：

1. 分配内存空间。
2. 执行构造方法，初始化对象。
3. 把引用指向这块内存。

步骤 2 和 3 之间没有数据依赖，JIT 和 CPU 允许重排成 1、3、2。一旦发生重排，另一个线程在锁外做第一次判空时会读到一个非 null 但尚未初始化完成的引用，拿去用就是各种莫名其妙的空字段。`volatile` 禁止了这段重排，同时保证写入对其他线程立即可见。

## 方法四：静态内部类

把懒加载交给 JVM 的类加载机制，写法比双重检查锁干净得多。

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

`Holder` 只有在第一次被真正引用时才会初始化，也就是第一次调用 `getInstance()` 的时候，所以实例是懒加载的。而类初始化的线程安全由 JVM 保证，代码里一把锁都不需要。

缺点是没法给构造过程传参。如果实例必须依赖运行时才能拿到的参数，只能回到双重检查锁。

## 方法五：枚举

上面四种写法都挡不住两件事：

- 反射。`setAccessible(true)` 之后可以直接调用私有构造器，造出第二个实例。
- 反序列化。默认的反序列化过程会绕过构造器生成新对象，除非自己实现 `readResolve()`。

枚举天然免疫这两点：`Constructor.newInstance()` 对枚举类型会直接抛 `IllegalArgumentException`，序列化时 JVM 也只按名字查找已有的枚举常量，不会新建对象。

```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        // 业务方法
    }
}
```

调用方写 `Singleton.INSTANCE.doSomething()`。

代价是枚举不能继承其他类，而且实例在类加载时就创建，同样没有懒加载。

## 防住反射

如果不用枚举又想挡一下反射，可以在构造器里检查实例是否已存在：

```java
private Singleton() {
    if (Holder.INSTANCE != null) {
        throw new IllegalStateException("单例已存在");
    }
}
```

这只是一道补丁：攻击者仍然可以先反射把静态字段置空再创建。想彻底堵死就用枚举。

## 小结

- 实例构造很轻、必然会用到，直接写饿汉式，最简单也最不容易出错。
- 需要懒加载且构造不依赖参数，用静态内部类，兼顾简洁和性能。
- 构造需要传运行时参数，用 `volatile` + 双重检查锁，注意 `volatile` 不能省。
- 要求防住反射和反序列化，用枚举。
- 懒汉式加同步方法只适合面试时讲演进过程，实际不要用。

面试里被问到单例，重点通常不在于能背出几种写法，而在于能不能说清楚双重检查锁为什么需要 `volatile`，以及静态内部类的懒加载是靠什么机制实现的。

实际项目里如果已经在用 Spring，单例交给容器管理即可，不必自己手写。
