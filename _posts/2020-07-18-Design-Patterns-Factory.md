---
layout: post
title: Factory Design patterns(工厂模式)
date: 2020-7-18
Author: shope
categories: Design patterns
tags: [工厂模式]
---

### 简单工厂模式

简单工厂模式又叫静态工厂方法模式如果要创建的产品不多，只要一个工厂类就可以完成。用户只需要知道参数，调用工厂方法创建对象。最主要的是简单工厂方法模式的工厂方法为静态方法。

**缺点：**

1、在工厂方法中定义逻辑判断，要拓展只能修改工厂方法逻辑，不符合开闭原则。

2、正因为在工厂法中定义逻辑判断，所以拓展工厂方法后，导致工厂方法过于臃肿。

3、工厂方法为静态方法，无法定多种工厂角色。

**定义类型接口**（抽象）

```java
public interface Pet{
    void action();
}
```

**具体类**

```java
public class Cart implements Pet {
    void action(){
        System.out.println("喵喵!!");
    }
}


public class Dog implements Pet {
    void action(){
        System.out.println("汪汪!!");
    }
}
```

**工厂方法**

```java
public class Factory{
    static class petFactory {
        public static Pet createPet(int kind) {
            switch (kind) {
                case 1:
                    return new Cart();
                case 2:
                    return new Dog();
            }
            return null;
        }
    }
}
```



------

### 工厂方法模式