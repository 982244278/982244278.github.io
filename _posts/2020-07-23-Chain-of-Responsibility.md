---
layout: post
title: 责任链(Chain of Responsibility)模式
date: 2020-07-23
Author: shope
categories: Design patterns
tags: [责任链模式]
---

**定义：**

将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。

**优点：**

1、可以动态拓展，满足开闭原则。

2、责任分担，每个类只负责自己要处理的工作。

3、简化对象之间的连接，降低耦合度。

**缺点：**

1、不确定每一个请求都被处理。

2、责任链过长会影响系统的性能。

**实现方案：**

1、对象数组

2、对象链表

**结构：**

1、抽象处理者（Handler）

​	定义处理类父类，抽象处理方法.

2、具体处理者（Concrete Handler）

​	实现Handler，对请求进行判断并处理，如果可以则处理请求，不处理传递给下一个处理者.

**抽象处理者**

```java
public abstract class Handler{
    private Handler next;
    
    public void setNextHandler(Handler h){
        this.next = h;
    }
    
    public Handler getNext(){
        return next;
    }
    
    public abstract void process(String request);
}
```

**具体处理者**

```java
public class FirstHandler extends Handler {
    
    @Override
    public void process(String request){
        if(request.equals("first")){
            System.out.println("FirstHandler process");
        } else{
            if(null != getNext()){
                getNext().process(request);
            }
        }
    }
}

public class SecondHandler extends Handler {
    
    @Override
    public void process(String request){
        if(request.equals("second")){
            System.out.println("SecondHandler process");
        } else{
            if(null != getNext()){
                getNext().process(request);
            }
        }
    }
}
```

**运行示例:**

```java
public class Main {
    public static void main(String args[]){
        Handler first = new FirstHander();
        Handler second = new SecondHandler();
        first.setNext(second);
        first.process("second");
    }
}
// output: SecondHandler process
```