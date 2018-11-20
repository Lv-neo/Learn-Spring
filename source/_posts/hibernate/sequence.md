---
title:  Hibernate-sequence doesn't exist
tag: java
---
<!-- toc -->
#  Hibernate-sequence doesn't exist
 学过hibernate的都知道hibernate支持n种主键生成策略。但有一种比较诡异，就是sequence。众所周知，oracle 表的主键生成策略是没有自增(identity或者autoIncrement)的，他是通过 sequence 来实现的。而mysql正好与oracle相反，mysql支持自增，恰好不支持 sequence。如果想要在mysql上应用sequence主键生成策略那就悲剧了。 
 
其实最简单的办法就是用hibernate实体设置

```
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
```

网上很多描述主要关于这一点存在两种情况，一种是没设置对id
一种是设置了id但是使用了错误的hibernate.xml配置

