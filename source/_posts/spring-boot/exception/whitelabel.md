---
title:  自定义'白标'（whitelabel，可以了解下相关理念）错误页面
tag: java
---
<!-- toc -->
#  自定义'白标'（whitelabel，可以了解下相关理念）错误页面

Spring Boot安装了一个'whitelabel'错误页面，如果你遇到一个服务器错误（机器客户端消费的是JSON，其他媒体类型则会看到一个具有正确错误码的合乎情理的响应），那就能在客户端浏览器中看到该页面。你可以设置error.whitelabel.enabled=false来关闭该功能，但通常你想要添加自己的错误页面来取代whitelabel。确切地说，如何实现取决于你使用的模板技术。例如，你正在使用Thymeleaf，你将添加一个error.html模板。如果你正在使用FreeMarker，那你将添加一个error.ftl模板。通常，你需要的只是一个名称为error的View，和/或一个处理/error路径的@Controller。除非你替换了一些默认配置，否则你将在你的ApplicationContext中找到一个BeanNameViewResolver，所以一个id为error的@Bean可能是完成该操作的一个简单方式。详情参考ErrorMvcAutoConfiguration。
查看Error Handling章节，了解下如何将处理器（handlers）注册到servlet容器中。

