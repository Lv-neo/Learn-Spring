# 如何在spring boot中加载文件
>今天有同学在做支付验证问题，本地调试spring boot 用idea集成的jar方式工作，准备上到测试环境打包成war后发现，加载到签名key文件无法读取。然后我们来研究下如何使用spring boot 加载文件。

## jar加载
spring boot默认加载文件的路径是 

```
/META-INF/resources/ 
/resources/ 
/static/ 
/public/ 

//源码里有
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {  
        "classpath:/META-INF/resources/", "classpath:/resources/",  
        "classpath:/static/", "classpath:/public/" };
        
//配置里也有
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/ # Locations of static resources. 

所有本地的静态资源都配置在了classpath下面了, 而非在webapp下了 
        
```

我们可以看到spring boot 已经将公共文件夹，静态，资源
都定义好了，一般情况下，按照实际情况分配代码就好了

## war加载



