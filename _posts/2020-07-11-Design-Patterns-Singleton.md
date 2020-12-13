---
layout: post
title: Singleton Design patterns(单例设计模式)
date: 2020-7-11
Author: shope
categories: Design patterns
tags: [单例设计模式]
---

### 单例模式--懒汉式

该懒汉模式在多线程情况下，返回的实例并不一定是唯一的。

在两个线程A、B同时获取实例，就不能确保返回的是同一个实例。

```Java
public class LazySingleton{
    private static LazySingleton INSTANCE;
    private LazySingleton(){}
    public static LazySingleton getInstance(){
        if(null == INSTANCE){
            INSTANCE = new LazySingleton();
        }
        return INSTANCE;
    }
}
```

### 单例模式--懒汉式--Double Check

使用Double Check实现在多线程环境下保证实例的唯一性。在未实例化前会做Double Check，实例化后程序就不会再进入同步代码块。

```java
public class LazySingleton{
    
    private static LazySingleton INSTANCE;
    
    private LazySingleton(){}
    
    public static LazySingleton getInstance(){
        if(null == INSTANCE){
            synchronized(LazySingleton.class){
                if(null == INSTANCE){
                    INSTANCE = new LazySingleton();
                 }
            }
        }
        return INSTANCE;
    }
}
```

### 单例模式--饿汉式

饿汉式在多线程环境下能够保证实例的唯一性。该知识点涉及到 JVM的类加载机制 。

1、首先会加载类的.class文件到内存中

2、接着对文件进行验证以确保文件是正确的

3、接下来JVM会给**类变量**（静态变量）分配内存空间 (解析过程忽略)

4、给类变量赋值

JVM确保加载的唯一性，从而确保实例化的唯一性。

```java
public class HangrySingleton{
    private static HangrySingleton INSTANCE = new HangrySingleton();
    
    private HangrySingleton(){}
    
    public static HangrySingleton getInstance(){
        return INSTANCE;
    }
}
```

### 单例模式--内部类

使用内部内可以实现

```java
public class InnerClassSingleton{
    
    private InnerClassSingleton(){
        if(InnerClassHolder.INSTANCE != null){
            throw new RuntimeException("singleton can not instance mutli instance.");
        }
    }
    
    private static class InnerClassHolder{
        private static INSTANCE = new InnerClassSingleton();
    }
    
    public static InnerClassSingleton getInstance(){
        return InnerClassHolder.INSTANCE;
    }
}
```

