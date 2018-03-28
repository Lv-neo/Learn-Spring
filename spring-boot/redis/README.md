# spring-boot使用redis

###导入jar

```
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

###配置

```
# REDIS (RedisProperties)
#spring.redis.database=
spring.redis.host=
spring.redis.password=
spring.redis.port=6379 
spring.redis.pool.max-idle=100 
spring.redis.pool.min-idle=1
spring.redis.pool.max-active=1000
spring.redis.pool.max-wait=-1
server.port=8081
```

#编写cache管理类

```
package com.example;
import java.lang.reflect.Method;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.interceptor.KeyGenerator;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
    /**
     * 缓存管理（注解用）
     * @author Administrator
     *
     */
    @Configuration
    @EnableCaching
    public class CacheService extends CachingConfigurerSupport {
            
        /**
     * 生成key的策略
     *
     * @return
     */
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }
    
    /**
     * 管理缓存
     *
     * @param redisTemplate
     * @return
     */
    @SuppressWarnings("rawtypes")
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate) {
        RedisCacheManager rcm = new RedisCacheManager(redisTemplate);
        //设置缓存过期时间
        // rcm.setDefaultExpiration(60);//秒
        //设置value的过期时间
        Map<String,Long> map=new HashMap();
        map.put("test",60L);
        rcm.setExpires(map);
        return rcm;
    }
    
    /**
     * RedisTemplate配置
     * @param factory
     * @return
     */
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate template = new StringRedisTemplate(factory);
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}

```

###在方法前面加上缓存

```
RequestMapping("/")
@Cacheable(value="test")
public String getSessionId(HttpSession session){
redisUtil.set("123", "测试");
System.out.println("进入了方法");
String string= redisUtil.get("123").toString();
return string;
}
```

###附录
@Cacheable参数
value  指明缓存将被存到什么地方。
key     Spring默认使用被@Cacheable注解的方法的签名来作为key
condition = "#age < 25" 数将指明方法的返回结果是否被缓存。

转载自https://my.oschina.net/wangnian/blog/661389


