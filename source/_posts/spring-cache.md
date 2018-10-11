---
title: Spring Framework缓存基于Redis的使用（@CachePut, @Cacheable, @CacheEvict）
cdn: header-off
date: 2018-09-11 11:37:27
header-img: "/img/springboot.jpg"
tags:
	- Java
	- Spring
---
> Spring Framework提供了基于Redis的缓存功能，方便被动的写缓存、读缓存、删除缓存。下面记录一下Spring缓存的配置及使用。

## 缓存配置

为了更定制化地使用Spring缓存，我们将通过创建一个RedisCacheManager的@Bean对Spring缓存进行配置。
``` java
// Spring Boot auto-configures the cache infrastructure as long as caching support is enabled via the @EnableCaching annotation
@EnableCaching
@Configuration
public class RedisConfiguration {
    @Bean
    public RedisCacheManager cacheManager(RedisTemplate<String, String> redisTemplate) {
        RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
        RedisSerializer<String> keyRedisSerializer = new StringRedisSerializer();
        RedisSerializer<Object> valueRedisSerializer = new GenericJackson2JsonRedisSerializer();
        redisTemplate.setKeySerializer(keyRedisSerializer);
        redisTemplate.setValueSerializer(valueRedisSerializer);
        redisTemplate.afterPropertiesSet();
        redisCacheManager.setUsePrefix(true);
        //默认过期时间2小时
        redisCacheManager.setDefaultExpiration(7200);
        return redisCacheManager;
    }
}	
```

## 注解使用

### @CachePut
``` java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    String unless() default "";

    boolean sync() default false;
}
```

### @Cacheable
``` java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Cacheable {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    String unless() default "";

    boolean sync() default false;
}
```

### @CacheEvict
``` java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {
    @AliasFor("cacheNames")
    String[] value() default {};

    @AliasFor("value")
    String[] cacheNames() default {};

    String key() default "";

    String keyGenerator() default "";

    String cacheManager() default "";

    String cacheResolver() default "";

    String condition() default "";

    boolean allEntries() default false;

    boolean beforeInvocation() default false;
}
```