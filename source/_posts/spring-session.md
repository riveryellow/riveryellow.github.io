---
title: Spring Session 1.3.1配置
cdn: header-off
date: 2018-10-26 17:45:03
header-img: "/img/springboot.jpg"
tags:
    - Java
    - Spring
---
### Maven依赖
```xml
        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session-data-redis</artifactId>
            <version>1.3.1.RELEASE</version>
            <type>pom</type>
        </dependency>
        <dependency>
            <groupId>biz.paluch.redis</groupId>
            <artifactId>lettuce</artifactId>
            <version>4.4.6.Final</version>
        </dependency>
```

### Java Configuration
WebInitializer.java
``` java
public class WebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{HttpSessionConfiguration.class};
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[0];
    }

    @Override
    protected String[] getServletMappings() {
        return new String[0];
    }
}
```

SpringSessionInitializer.java
``` java
public class SpringSessionInitializer extends AbstractHttpSessionApplicationInitializer {
    SpringSessionInitializer() {
        super(HttpSessionConfiguration.class);
    }
}
```

HttpSessionConfiguration.java
``` java
@Configuration
@EnableRedisHttpSession(redisNamespace = "token")
public class HttpSessionConfiguration {

    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory("pmusrds02.la.gomo.com", 6379);
    }

    @Bean
    public CookieSerializer cookieSerializer() {
        DefaultCookieSerializer serializer = new DefaultCookieSerializer();
        serializer.setCookieName("JSESSIONID"); // <1>
        serializer.setCookiePath("/"); // <2>
        serializer.setDomainNamePattern("^.+?\\.(\\w+\\.[a-z]+)$"); // <3>
        return serializer;
    }

    @Bean
    public FilterRegistrationBean sessionFilterRegistration() {
        FilterRegistrationBean registration = new FilterRegistrationBean();
        registration.setFilter(new SessionFilter());//添加过滤器
        registration.addUrlPatterns("/*");//设置过滤路径，/*所有路径
        registration.setName("SessionFilter");//设置优先级
        registration.setOrder(1);//设置优先级
        return registration;
    }

    public class SessionFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) {

        }

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) {
            if(((HttpServletRequest) servletRequest).getSession().getAttribute("access_token") != null) {
                Cookie cookie = new Cookie("access_token", ((HttpServletRequest) servletRequest).getSession().getAttribute("access_token").toString());
                ((HttpServletResponse) servletResponse).addCookie(cookie);
            }
        }

        @Override
        public void destroy() {
        }
    }

}
```