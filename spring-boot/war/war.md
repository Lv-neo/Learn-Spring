# spring-boot 传统部署方式

## war方式
### 修改你的Application
产生一个war包，应该提供一个SpringBootServletInitializer子类并覆盖它的configure 方法
这样让spring framework Servlet 3.0 支持通过你的配置和应用程序产生一个Servlet容器。

>The first step in producing a deployable war file is to provide a SpringBootServletInitializer subclass and override its configure method. This makes use of Spring Framework’s Servlet 3.0 support and allows you to configure your application when it’s launched by the servlet container. Typically, you update your application’s main class to extend SpringBootServletInitializer:

上面的意思就是让你的Application入口文件继承SpringBootServletInitializer
然后重写下configure 

代码如下

```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}

```

### maven or gradle 配置
#### maven
要先依赖spring-boot-starter-parent 
然后加上

```java
<packaging>war</packaging>
```
#### gradle

```java
apply plugin: 'war'
```

### 去掉tomcat容器
#### maven
```
<dependencies>
    <!-- … -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <!-- … -->
</dependencies>

```
#### gradle
```
dependencies {
    // …
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
    // …
}
```


### 附录
http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-traditional-deployment



