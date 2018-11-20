---
title:  thymeleaf介绍
tag: java
---
<!-- toc -->
#  thymeleaf介绍

在Java世界的MVC框架里，使用的视图技术不少，最基本的是JSP，还有知名的FreeMarker和Velocity等模板引擎。Thymeleaf也是一款优秀的模板引擎，它在HTML5/XHTML的视图层表现的很好，也能在离线情况下处理任何XML文件。它是完全可以替代JSP+JSTL的。

下面是来自于Thymeleaf官方的Q&A：

Q: 和FreeMarker,Velocity相比，Thymeleaf表现得怎样呢？

A：FreeMarker和Velocity都是软件领域杰出的作品，但它们在解决模板问题上的处理哲学和Thymeleaf不一样。

Thymeleaf强调的是自然模板，也就是允许模板作为产品原型使用(笔者注:因为其后缀名就是.html，可以直接在浏览器打开)，而FreeMarker和Velocity不行。并且Thymeleaf的语法更简洁、更和目前Web开发的趋势相一致。其次，从架构的角度看，FreeMarker和Velocity更像个文本处理器，所以它们能够处理很多类型的内容，而Thymeleaf是基于XML的，只能处理XML格式的数据。因此这样看，FreeMarker和Velocity更通用些，但恰恰如此，Thymeleaf更能利用XML的特性，尤其是在Web应用中。

[官网地址](http://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#the-controller)

[spring-boot示例](https://spring.io/guides/gs/serving-web-content/)


