	⁃	# 构建Spring Framework

### gradle 构建

* 安装jdk，设置JAVA_HOME
* 安装[gradle](http://www.gradle.org/downloads)
* gradle init 初始化
* 设置如下依赖

```gradle
dependencies {
    compile 'org.springframework:spring-context:4.3.1.RELEASE'
}
```

### maven 构建

* 安装jdk，设置JAVA_HOME
* 安装[maven](http://maven.apache.org/download.cgi) 设置M2_HOME
* mvn 初始化

```
mvn archetype:generate -DgroupId=com.mailejifen.oauth -DartifactId=oauth -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false

```

* mvn 增加Spring Framework依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>4.3.1.RELEASE</version>
    </dependency>
</dependencies>
```


