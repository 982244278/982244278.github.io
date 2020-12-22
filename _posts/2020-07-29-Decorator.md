---
layout: post
title: 装饰器(Decorator)模式
date: 2020-07-29
Author: shope
categories: Design patterns
tags: [装饰器模式]
---

**定义：**

在不改变现有的对象结构，能够给对象动态的添加一些功能，符合开闭原则。

**优点：**

1、在不改变原有对象的情况下，动态的给一个对象扩展功能。

2、遵循开闭原则。

**缺点：**

1、装饰模式会增加许多子类，过度使用会增加程序的复杂性。

**结构图：**

1. 抽象构件（Component）角色：定义一个抽象接口以规范准备接收附加责任的对象。
2. 具体构件（ConcreteComponent）角色：实现抽象构件，通过装饰角色为其添加一些职责。
3. 抽象装饰（Decorator）角色：继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
4. 具体装饰（ConcreteDecorator）角色：实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

![Docorator.PNG](https://i.loli.net/2020/12/21/AcfUg7zhYHioBtj.png)

**抽象构件**

```java
public abstract class Component {
    private String desc = "no action";

    public Component() {
    }

    public Component(String desc) {
        this.desc = desc;
    }

    public String getDesc(){
        return  desc;
    }
}
```

**具体构件**

```java
//  dog构件
public class DogComponent extends Component{
    public DogComponent() {
        super("狗子");
    }
}
//  cat构件
public class CatComponent extends Component {
    public CatComponent() {
        super("喵星人");
    }
}
```

**抽象装饰器**

```java
public abstract class Docorator extends Component {
    protected Component component;
    
    public abstract String getDesc();
}
```

**具体装饰器**

```java
//  会喵喵喵的叫---行为
public class MiaoMIao extends Docorator {
    
    public MiaoMIao(Component component) {
        this.component = component;
    }

    @Override
    public String getDesc() {
        return component.getDesc()+"会叫 喵喵喵";
    }
}

//  会汪汪汪的叫---行为
public class WangWang extends Docorator {

    WangWang( Component component){
        this.component = component;
    }
    @Override
    public String getDesc() {
        return component.getDesc()+"会叫 汪汪汪 ";
    }
}

// 猫和狗共同的行为---会刨土
public class SameAction extends Docorator {

    public SameAction(Component component) {
        this.component = component;
    }

    @Override
    public String getDesc() {
        return component.getDesc() + " 还会 刨土";
    }
}
```

**运行示例**

```java
public class Main {
    public static void main(String[] args) {
        Component dog = new DogComponent();
        dog = new WangWang(dog);
        dog = new SameAction(dog);
        System.out.println(dog.getDesc());

        Component miao = new CatComponent();
        miao = new MiaoMIao(miao);
        miao = new SameAction(miao);
        System.out.println(miao.getDesc());
   }
}

/**
 * output:
 * 狗子会叫 汪汪汪  还会 刨土
 * 喵星人会叫 喵喵喵 还会 刨土
 */
```

