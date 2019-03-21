---
layout: post
title: "读HeadFirst设计模式"
date: 2018-12-09 10:25:25 +0800
comments: true
categories: 设计模式
---

以往虽也看过相关设计模式的书籍，但能和与`HeadFirt设计模式`这本书相比不是缺乏严谨性就是缺乏具体应用实例，还有幽默生动以及引人启发的观点。

## 设计原则

设计原则并不能光靠死记硬背，我们需要通过具体的设计模式来反思：该模式符合与违背了哪些设计原则。
不过在进入下文讨论之前，让我们先大概看看设计原则都有哪些。

- 封装变化
- 针对接口编程，而不是实现
- 多用组合少用继承
- 松耦合
- 对修改关闭，对扩展开放
- 依赖抽象不依赖具体
- 最少知识原则：只和亲密对象交谈
- 好莱坞原则：别打电话给我，我会打给你
- 单一职责

## 谈谈模板方法

我们都知道在`GoF`里面的设计模式含有23种，在`HeadFirst设计模式`一书中主要介绍了11种常见的设计模式。
本文挑选`模板方法`这个设计模式着重介绍下，主要是该模式在前后端的开发中经常遇到，也便于理解。

为了探析`模板方法`，让我们先看看日常生活中我们泡茶与泡咖啡的步骤。

<!--more-->

**泡茶**

- 把水烧开
- 放茶包
- 往杯子中加水
- 加入柠檬

用Java实现如下：

```java
public class Tea {
    public void prepareRecipe() {
        this.boilWater();
        this.steepTeaBag();
        this.pourInCup();
        this.addLemon();
    }

    private void addLemon() {
        System.out.println("add lemon...");
    }

    private void pourInCup() {
        System.out.println("pour in cup...");
    }

    private void steepTeaBag() {
        System.out.println("steep tea bag...");
    }

    private void boilWater() {
        System.out.println("boil water...");
    }
}
```

**泡咖啡**

- 把水烧开
- 研磨咖啡
- 往杯子中加水
- 加入糖和牛奶

用Java实现如下：

```java
public class Coffee {
    public void prepareRecipe() {
        this.boilWater();
        this.brewCoffeeGrinds();
        this.pourInCup();
        this.addSugarAndMilk();
    }

    private void addSugarAndMilk() {
        System.out.println("add sugar and milk...");
    }

    private void pourInCup() {
        System.out.println("pour in cup...");
    }

    private void brewCoffeeGrinds() {
        System.out.println("brew coffee grinds...");
    }

    private void boilWater() {
        System.out.println("boil Water....");
    }
}
```

### 探析

当写完代码，不管你记不记得`封装变化`这条设计原则，你肯定觉得泡茶喝泡咖啡有些地方代码非常的相似，例如：`烧水`, `把水倒入被子`, 对于`研磨咖啡豆`和`放茶包`其本质都是：研磨喝的固体物质，我们可以把这些不变的部分提取出来。而把哪些变化的部分：`添加糖和牛奶`和`添加柠檬`封装起来。
我相信大多数人都会想到基础，构造一个`饮料`基类。但是如何让设计更加灵活呢？
让我们一起来看看Java版的`模板方法`实现：

```java
public abstract class Beverage {

    public void prepareRecipe() {
        this.boilWater();
        this.brew();
        this.pourInCup();
        if (customerWantsCondiments()) {
            this.addCondiments();
        }
    }

    public abstract void addCondiments();

    public abstract void brew();

    public void boilWater() {
        System.out.println("boil water...");
    }

    public void pourInCup() {
        System.out.println("pour in cup...");
    }

    public boolean customerWantsCondiments() {
        return true;
    }
}
```

在`模板方法`中，我们定义了一系列的泡饮料的步骤，我们将变化的部分交给其子类去实现，可以说对变化进行了封装。而且在基类`Beverage`中的`customerWantsCondiments`方法就很巧妙，用户想不想加调料可以让他们自己决定。当然我们默认用户是想要加调料的。

我们再来看看两个实现类：`Tea`和`Coffee`。

```java
public class Tea extends Beverage {

    @Override
    public void addCondiments() {
        System.out.println("add lemon...");
    }

    @Override
    public void brew() {
        System.out.println("steep tea bag...");
    }
}

public class Coffee extends Beverage {

    @Override
    public void addCondiments() {
        System.out.println("add sugar and milk...");
    }

    @Override
    public void brew() {
        System.out.println("brew coffee grinds...");
    }
}
```

这两个子类的实现都很简单，我们再来看看如何给出用户想不想在`Coffee`中加入调料的选择条件。

```java
public class CoffeeWithHook extends Beverage {

    public void addCondiments() {
        System.out.println("add sugar and milk...");
    }

    public void brew() {
        System.out.println("brew coffee grinds...");
    }

    @Override
    public boolean customerWantsCondiments() {
        String answer = getUserInput();

        if (answer.toLowerCase().startsWith("y")) {
            return true;
        } else {
            return false;
        }
    }

    private String getUserInput() {
        String answer = null;
        System.out.println("Would you like milk and sugar with your coffee (y/n) ?");
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        try {
            answer = reader.readLine();
        } catch (IOException e) {
            System.out.println(e);
        }
        if (answer == null) {
            return "no";
        }

        return answer;
    }
}
```

是不是感觉很巧妙呢:D

### 总结模板方法

那么什么是`模板方法`模式呢，其实我们只需记住两点就好了：

- 封装算法(也就是一些列步骤)
- 制造钩子

这时候我们再来看看`Tempate Method`的定义：

> 模板方法模式在一个方法中定义了一个算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以在不改变算法结构的情况下，重新定义算法中的某些步骤。

其实在我们的前端开发中，我们可以把Android、iOS开发等App的开发，Windows窗体，Unity游戏客户端，React等等GUI开发统称为`前端开发`。我记得很多朋友喜欢叫啥`前台`和`后台`开发。。。我感觉不是很准确，首先`后台`是什么？我们知道后台就是管理员操作的一个台子。私以为用来作为`frontend`以及`backend`的中文译语不是很准确也不专业的~

## 设计模式类目

总之，`HeadFirst设计模式`强烈推荐给大家去看。不会错的，看的时候别忘了写实现，做笔记，思考。
这里在对设计模式做个大总结：

![](/images/dp/dp_category.png)

最后再把《HeadFirst》设计模式的开场图送给大家👏

![](/images/dp/start.png)

EOF