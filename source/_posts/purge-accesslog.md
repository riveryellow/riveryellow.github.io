---
title: 基于Spring Boot的Accesslog存储数量控制源码分析
cdn: header-off
date: 2018-12-10 16:58:06
header-img: "/img/springboot.jpg"
tags:
    - Spring
    - Java
---

在项目中，所有的access log都通过logstash导入到ES中进行存储分析，因此不希望在应用服务器中存储大量的access log占用存储空间。
在Spring Boot的默认配置中并没有可以直接对日志数量进行配置的参数，于是在[Github](https://github.com/)上找到一个开源的[spring-boot-starter-purge-accesslog](https://github.com/marcosbarbero/spring-boot-starter-purge-accesslog)，于是借来使用一下并学习一下实现方式。

## Logback的maxHistory
在开发基于Spring Boot的应用时，默认使用Logback作日志功能。通过对logback的配置，可以通过配置appender中TimeBasedRollingPolicy#maxHistory来管理各个类型日志存储数量。
logback-spring.xml
```xml
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>7</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <charset>${CHAR_SET}</charset>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n%ex{short}</pattern>
        </encoder>
    </appender>
```
如上的配置可以保留最近的7个日志文件。

## spring-boot-autoconfigure ServerProperties#Tomcat#Accesslog

Access Log作为服务器级别的日志，在进入到应用容器之前就已经被记录了。在使用Spring Boot之前，我们通过配置Tomcat conf/server.xml来配置access log的属性。使用Spring Boot后，Spring Boot因为集成了Tomcat，提供了ServerProperties.Tomcat.Accesslog对accesslog进行配置。
``` java
        public static class Accesslog {

			/**
			 * Enable access log.
			 */
			private boolean enabled = false;

			/**
			 * Format pattern for access logs.
			 */
			private String pattern = "common";

			/**
			 * Directory in which log files are created. Can be relative to the tomcat
			 * base dir or absolute.
			 */
			private String directory = "logs";

			/**
			 * Log file name prefix.
			 */
			protected String prefix = "access_log";

			/**
			 * Log file name suffix.
			 */
			private String suffix = ".log";

			/**
			 * Enable access log rotation.
			 */
			private boolean rotate = true;

			/**
			 * Defer inclusion of the date stamp in the file name until rotate time.
			 */
			private boolean renameOnRotate;

			/**
			 * Date format to place in log file name.
			 */
			private String fileDateFormat = ".yyyy-MM-dd";

			/**
			 * Set request attributes for IP address, Hostname, protocol and port used for
			 * the request.
			 */
			private boolean requestAttributesEnabled;

			/**
			 * Buffer output such that it is only flushed periodically.
			 */
			private boolean buffered = true;

            @Getter
            @Setter
		}
```
如上，并没有和logback中maxHistory类似的配置参数。

## spring-boot-starter-purge-accesslog

### PurgeProperties
PurgeProperties.java
``` java
@Data
@ConfigurationProperties(PREFIX)
public class PurgeProperties implements InitializingBean {

	/**
	 * The constant PREFIX.
	 */
	public static final String PREFIX = "server.accesslog.purge";
	/**
	 * The constant ALLOWED_UNITS.
	 */
	private static final EnumSet<ChronoUnit> ALLOWED_UNITS = of(SECONDS, MINUTES, HOURS,
			DAYS);

	/**
	 * The Enabled.
	 */
	private boolean enabled;
	/**
	 * The Execute on startup.
	 */
	private boolean executeOnStartup;
	/**
	 * The Execution interval.
	 */
	private long executionInterval = 24;
	/**
	 * The Max history.
	 */
	private long maxHistory = 30;
	/**
	 * The Execution interval unit.
	 */
	private ChronoUnit executionIntervalUnit = HOURS;
	/**
	 * The Max history unit.
	 */
	private ChronoUnit maxHistoryUnit = DAYS;

	/**
	 * After properties set.
	 */
	@Override
	public void afterPropertiesSet() {
		isTrue(this.executionInterval > 0, "'executionInterval' must be greater than 0");
		isTrue(this.maxHistory > 0, "'maxHistory' must be greater than 0");
		isTrue(ALLOWED_UNITS.contains(this.executionIntervalUnit),
				"'executionIntervalUnit' must be one of the following units: SECONDS, MINUTES, HOURS, DAYS");
		isTrue(ALLOWED_UNITS.contains(this.maxHistoryUnit),
				"'maxHistoryUnit' must be one of the following units: SECONDS, MINUTES, HOURS, DAYS");
	}
}
```
### PurgeAccessLogAutoConfiguration
PurgeAccessLogAutoConfiguration.java
```java
@Configuration
@EnableConfigurationProperties(PurgeProperties.class)
@ConditionalOnClass(ServerProperties.class)
@ConditionalOnProperty(prefix = PREFIX, name = "enabled", havingValue = "true")
public class PurgeAccessLogAutoConfiguration {
    /**
     * The type Tomcat purge access log configuration.
     */
    @ConditionalOnClass(AccessLogValve.class)
    @ConditionalOnProperty(name = "server.tomcat.accesslog.enabled", havingValue = "true")
    public static class TomcatPurgeAccessLogConfiguration {

        /**
         * Purge access log customizer embedded servlet container customizer.
         *
         * @param serverProperties the server properties
         * @param purgeProperties  the purge properties
         * @return the embedded servlet container customizer
         */
        @Bean
        public WebServerFactoryCustomizer<TomcatServletWebServerFactory> purgeAccessLogCustomizer(
                final ServerProperties serverProperties,
                final PurgeProperties purgeProperties) {
            return factory -> {
                final Accesslog accesslog = serverProperties.getTomcat().getAccesslog();
                factory.getEngineValves().stream()
                        .filter(valve -> valve instanceof AccessLogValve)
                        .map(valve -> (AccessLogValve) valve).findFirst()
                        .ifPresent(valve -> {
                            final TomcatPurgeAccessLogHolder accessLogHolder = new TomcatPurgeAccessLogHolder(
                                    purgeProperties, Paths.get(accesslog.getDirectory()),
                                    accesslog.getPrefix(), accesslog.getSuffix(), valve);
                            factory.addContextCustomizers(accessLogHolder);
                        });
            };
        }
    }
}

```
### PurgeAccessLogHolder
PurgeAccessLogHolder.java
```java
@RequiredArgsConstructor
public abstract class PurgeAccessLogHolder {

	/**
	 * The Purge properties.
	 */
	private final PurgeProperties purgeProperties;
	/**
	 * The Directory.
	 */
	private final Path directory;
	/**
	 * The Prefix.
	 */
	private final String prefix;
	/**
	 * The Suffix.
	 */
	private final String suffix;
	/**
	 * The Current log file name supplier.
	 */
	private final Supplier<String> currentLogFileNameSupplier;

	/**
	 * Creates a scheduled thread pool and schedules the purge task according to the
	 * properties. If executeOnStartup property is false, then the task is scheduled to
	 * midnight of the next day.
	 */
	protected void attachPurgeTask() {
		long initialDelay = 0;

		if (!this.purgeProperties.isExecuteOnStartup()) {
			final LocalDateTime now = now();
			final LocalDateTime midnight = now.plusDays(1).with(MIDNIGHT);
			initialDelay = MILLIS.between(now, midnight);
		}

		final PurgeTask purgeTask = new PurgeTask(this.purgeProperties, this.directory,
				this.prefix, this.suffix, this.currentLogFileNameSupplier);
		final long executionInterval = this.purgeProperties.getExecutionInterval();
		final String executionIntervalUnit = this.purgeProperties
				.getExecutionIntervalUnit().name();

		ThreadFactory threadFactory = r -> new Thread(r, "access-log-purge-worker");
		ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor(threadFactory);

		executor.scheduleWithFixedDelay(purgeTask,
										initialDelay,
										executionInterval,
										valueOf(executionIntervalUnit));
	}

}
```
