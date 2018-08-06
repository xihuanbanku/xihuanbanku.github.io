---
layout: post
title: "springMVC流程图"
date: 2018-8-6 11:37:09 +0800
comments: true
---

## springMVC各阶段流程图

前段时间面试, 问到了`springMVC`的知识, 很久没用过了, 回答的不好, 所以把`springMVC`的原理和流程图记录一下, 方便以后查阅:

* 流程图

![流程图]({{ site.baseurl}}/images/2018-08-06-springMVC/SpringBoot2.jpg)

* 详细原理

1. 用户发起请求, 由前端控制器`DispatcherServlet`处理
1. 前端控制器通过处理器映射器查找`hander`, 可以根据`XML`或者注解去找
1. 处理器映射器返回执行链
1. 前端控制器请求处理器适配器来执行`hander`
1. 处理器适配器来执行`handler`
1. 处理业务完成后，会给处理器适配器返回`ModeAndView`对象, 其中有视图名称, 模型数据
1. 处理器适配器将视图名称和模型数据返回到前端控制器
1. 前端控制器通过视图解析器来对视图进行解析
1. 视图解析器返回真正的视图给前端控制器
1. 前端控制器通过返回的视图和数据进行渲染
1. 返回渲染完成的视图
1. 将最终的视图返回给用户, 产生响应

***

> 参考资料

1. [从 SpringBoot 到 SpringMVC](https://www.506064.com/programming/2550.html)