---
title:  log4j2 与 spring mvc整合
tag: java
---
<!-- toc -->
#  log4j2 与 spring mvc整合

## maven依赖
我用的log4j.version = 2.6.2
spring.version = 4.3.1.RELEASE

```
 <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-api</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-core</artifactId>
      <version>${log4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.logging.log4j</groupId>
      <artifactId>log4j-web</artifactId>
      <version>${log4j.version}</version>
    </dependency>
```

## 演示代码

```java
private Logger logger = LogManager.getLogger(this.getClass());

ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        logger.info("URL : " + request.getRequestURL().toString());
        logger.info("HTTP_METHOD : " + request.getMethod());
        logger.info("IP : " + request.getRemoteAddr());
        logger.info("CLASS_METHOD : " + pjp.getSignature().getDeclaringTypeName() + "." + pjp.getSignature().getName());
        logger.info("ARGS : " + Arrays.toString(pjp.getArgs()));

```

## 默认配置

在没有任何配置的情况下，log4j2会使用默认配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="error">
            <AppenderRef ref="Console" />
        </Root>
    </Loggers>
</Configuration>
```

该配置只有一个Appender：Console，目标是SYSTEM_OUT，即日志内容，都会打印在eclipse控制台上。Root Logger的级别是error，即：所有error及以上级别的日志才会记录。(注：日志级别顺序为 TRACE < DEBUG < INFO < WARN < ERROR < FATAL )，所以最终只有2日志会输出(error,fatal)

```
1 13:07:56.099 [main] ERROR  - error message
2 13:07:56.100 [main] FATAL  - fatal message
3 Hello World!
```

配置第1行中的status="WARN"，可以去掉，它的含义为是否记录log4j2本身的event信息，默认是OFF，设置成“WARN”指：所有log4j2的event信息中，只有WARN及以上级别的信息才记录，大家可以把它改成TRACE试试(最低级别)，看下输出内容有何变化。

另：配置文件通常命名为log4j2.xml，运行时只要在classpath下能找到即可。


## 文件记录日志

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- log4j2使用说明（create By SeanXiao    ）：
使用方式如下：
private static final Logger logger = LogManager.getLogger(实际类名.class.getName());

2、日志说明：
（1）请根据实际情况配置各项参数
（2）需要注意日志文件备份数和日志文件大小，注意预留目录空间
（3）实际部署的时候backupFilePatch变量需要修改成linux目录
 -->
<configuration status="error" monitorInterval="1800">
    <Properties>
        <Property name="fileName">oauth.mailejifen.com.log</Property>
        <Property name="backupFilePatch">/usr/local/logs/java/</Property>
    </Properties>
    <!--先定义所有的appender-->
    <appenders>
        <!--这个输出控制台的配置-->
        <Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->
            <ThresholdFilter level="trace" onMatch="ACCEPT" onMismatch="DENY" />
            <!--这个都知道是输出日志的格式-->
            <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n" />
        </Console>

        <!--这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFile" fileName="${backupFilePatch}${fileName}"
                     filePattern="${backupFilePatch}$${date:yyyy-MM}/app-%d{yyyyMMddHHmmssSSS}.log.gz">
            <PatternLayout
                    pattern="[%d{yyyy.MM.dd HH:mm:ss}] %-5level %class{36} %L %M - %msg%xEx%n" />

            <!-- 日志文件大小 -->
            <SizeBasedTriggeringPolicy size="20MB" />
            <!-- 最多保留文件数 -->
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </appenders>

    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <loggers>
        <!--建立一个默认的root的logger-->
        <Logger name="com.mailejifen.oauth" level="trace"
                additivity="true">
            <AppenderRef ref="RollingFile" />
        </Logger>
        <Root level="error">
            <AppenderRef ref="Console" />
        </Root>
    </loggers>
</configuration>
```

configuration中的 monitorInterval="1800" 指log4j2每隔1800秒（半小时），自动监控该配置文件是否有变化，如果变化，则自动根据文件内容重新配置（很不错的功能！）

Properties定义了一些属性（可以根据需要自己随便添加），主要是为了后面引用起来方便

RollingFile 即表示以文件方式记录，注意一下filePattern 的设置，它与20行的SizeBasedTriggeringPolicy （表示单个文件最大多少容量）结合在一起，非常有用，以这段配置为例，当单个文件达到10M后，会自动将以前的内容，先创建类似 2014-09（年-月）的目录，然后按 "xxx-年-月-日-序号"命名，打成压缩包（很体贴的设计，即省了空间，又不丢失以前的日志信息）

定义了一个新logger，它的级别是trace ，使用文件方式来记录日志，additivity="true" 这里注意一下，因为下面还有一个root logger，任何其它的logger最终都相当于继承自root logger，所以“com.cnblogs.yjmyzz.App2”这个logger中，如果记录了error及以上级别的日志，除了文件里会记录外，root logger也会生效，即：控制台也会输出一次。如果把additivity="true" 中的true，改成false，root logger就不会再起作用，即只会记录在文件里，控制台无输出。

附录：log4j2官方pdf文档
http://logging.apache.org/log4j/2.x/log4j-users-guide.pdf


