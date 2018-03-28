# Spring基于Annotation的简单介绍

>注解这个话题非常庞大，写这个文章压力也很大，生怕表达不清楚，职能稍做整理，仅供大家参考。

###前言
Spring 自 2.0 版本开始，陆续引入了一些注解用于简化 Spring 的开发。@Repository 注解便属于最先引入的一批，它用于将数据访问层 (DAO 层 ) 的类标识为 Spring Bean。具体只需将该注解标注在 DAO 类上即可。同时，为了让 Spring 能够扫描类路径中的类并识别出 @Repository 注解，需要在 XML 配置文件中启用 Bean 的自动扫描功能，这可以通过以下代码实现自动扫描。

```
<context:component-scan base-package = ""/> 
```

###注解的基本概念和原理
注解（Annotation）提供了一种安全的类似注释的机制，为我们在代码中添加信息提供了一种形式化得方法，使我们可以在稍后某个时刻方便的使用这些数据（通过解析注解来使用这些数据），用来将任何的信息或者元数据与程序元素（类、方法、成员变量等）进行关联。其实就是更加直观更加明了的说明，这些说明信息与程序业务逻辑没有关系，并且是供指定的工具或框架使用的。Annotation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的申明语句中。

Annotation其实是一种接口。通过java的反射机制相关的API来访问Annotation信息。相关类（框架或工具中的类）根据这些信息来决定如何使用该程序元素或改变它们的行为。Java语言解释器在工作时会忽略这些Annotation，因此在JVM中这些Annotation是“不起作用”的，只能通过配套的工具才能对这些Annotation类型的信息进行访问和处理。


