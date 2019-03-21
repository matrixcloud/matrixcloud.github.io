---
layout: post
title: "Zuul 1.x的工作原理"
date: 2018-03-18 10:17:34 +0800
comments: true
categories: Java
---

## Zuul简介

Zuul在微服务架构中，可以作为提供动态路由，监控，弹性，安全等边缘服务的框架。在Netflix，被用作所有请求到达`streaming application`的前门。Zuul使用一系列不同的`Filter`可以提供各种功能：

- 安全与认证
- 监控
- 动态路由
- 压测
- 限流
- 静态响应
- 多区域弹性负载均衡

> 可参考：[How We Use Zuul At Netflix](https://github.com/Netflix/zuul/wiki/How-We-Use-Zuul-At-Netflix)

## Zuul执行流程

先从github把代码clone下来：`git clone https://github.com/Netflix/zuul.git`，切换至`1.x`分支，核心代码为`zuul-core`模块，几个重要的类清单如下：

| 类名 | 描述 |
|-----|---------|
| RequestContext | 继承至ConcurrentHashMap；被保存在ThreadLocal中；单例 |
| ZuulServlet | 继承至HttpServlet；核心方法为service()|
| ZuulFilter | 抽象类，实现了Comparable接口，自定义Filter需从它继承 |
| ZuulRunner | ZuulServlet中用于代为执行route方法；核心方法为init()，初始化RequestContext中的request, response |
| FilterProcessor | Filter执行器，Filter的runFilter方法再该类的processZuulFilter被调用；单例 |
| FilterRegistry | Filter注册器；一个(K, V)的ZuulFilter容器。单例 |
| FilterLoader | Filter管理器；主要提供了getFiltersByType()，putFilter()方法和初始化FilterFileManager；单例 | 
| FilterFileManager | 声明了poller用于监听groovy文件夹的变化并通过FilterLoader.putFilter加入filter到FilterFileManager中。单例|

具体执行流程如图所示：

<!--more-->

{% img fancybox https://ws4.sinaimg.cn/large/006tKfTcgy1fph3cqu4ghj31kw0mwtbg.jpg %}
## ZuulFilter

ZuulFilter本身是个抽象类，通过继承ZuulFilter并实现以下三个方法，就可以定义各种Filter了：

```
/**
* 定义了同类型下filter的执行顺序：值越小优先被执行
*/
@Override
public int filterOrder() { 
    return 50000; 
}

/**
* 共四种类型：pre, route, post, error
*/
@Override
public String filterType() {
    return "pre";
}

/**
* 该filter是否能够被执行：抽象类ZuulFilter的runFilter有体现
*/
@Override
public boolean shouldFilter() {
    return true;
}

/**
/* filter的具体运行逻辑
*/
@Override
public Object run() {
    return null;
}
```

**Filter的生命周期**

![](https://ww1.sinaimg.cn/large/006tNc79gy1foqihwvs72j30qo0k0aao.jpg)

- `PRE`：在路由至`origin server`之前被执行，可用于请求认证、选择`origin server`、日志记录。
- `ROUTING`：在路由至`origin server`的过程中被执行。此时，原始的HTTP请求被构建，并且被Apache HttpClient或者Netflix Ribbon转发。
- `POST`：当请求被路由至`origin server`后被执行，可用于添加标准的HTTP头至响应中、或者收集统计信息的纬度、以及流转该响应至客户端。
- `ERROR`：当错误在任何其他阶段发生时被执行。

最后，来自Zuul的凝视😏

![](https://ww1.sinaimg.cn/large/006tNc79gy1fordkbvh7pj305k02wq2r.jpg)

> 参考：[官方wiki](https://github.com/Netflix/zuul/wiki)

EOF