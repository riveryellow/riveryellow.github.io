---
title: 通过PropertyPlaceholderConfigurer配置不同环境配置
date: 2017-09-25 14:46:15
cdn: "header-off"
header-img: "./img/configuration.jpg"
tags:
	- Spring
---
> 项目中，本地调试、测试环境、预发布环境、线上，不同环境有时需要不同的配置文件。原来的项目中，可以通过公共的配置管理服务为不同的环境在线配置不同的配置文件，然后在构建或服务器启动时去获取。
> 现在没有了这样的公共服务，将环境配置硬编码到代码中显然是不现实的。在Spring中，可以通过扩展PropertyPlaceholderConfigurer（bean factory post processor）来实现不同环境读取不同配置。

## 硬编码实现
java代码
``` java
public class UserServiceImpl implements UserService {
    private String couponSystem;
    private String couponLanguage;
	private String jpeaKey;
	...
	...
	public void setCouponSystem(String couponSystem) {
        this.couponSystem = couponSystem;
    }
    public void setCouponLanguage(String couponLanguage) {
        this.couponLanguage = couponLanguage;
    }
    public void setJpeaKey(String jpeaKey) {
        this.jpeaKey = jpeaKey;
    }
}
```
spring配置
``` xml
<bean id="userService" class="com.yihaodian.homePage.service.impl.UserServiceImpl">
	<property name="couponSystem" value="H5"></property>
	<property name="couponLanguage" value="zh-CN"></property>
	<property name="jpeaKey" value="e3b95cec784745"></property>
</bean>
```
通过setter方法，spring将配置注入到类的属性中。但是如果环境变化，需要不同的配置值，就要通过修改代码来实现，非常不方便。

## 扩展PropertyPlaceholderConfigurer实现
http://www.javainterviewpoint.com/spring-propertyplaceholderconfigurer-example/
``` java
public class GlobalPropertyConfigurer extends PropertyPlaceholderConfigurer {

    private static Map<String, String> ctxPropertiesMap;
    /**
     * Logger available to subclasses
     */
    protected final Log logger = LogFactory.getLog(getClass());

    protected boolean localOverride = false;

    private Resource[] locations;

    private PropertiesPersister propertiesPersister = new DefaultPropertiesPersister();

    /**
     * copy
     */
    public void setLocations(Resource[] locations) {
        this.locations = locations;
    }

    /**
     * copy
     */
    public void setLocalOverride(boolean localOverride) {
        this.localOverride = localOverride;
    }

    protected void processProperties(ConfigurableListableBeanFactory beanFactory, Properties props)
            throws BeansException {
        super.processProperties(beanFactory, props);
        //load properties to ctxPropertiesMap
        ctxPropertiesMap = Maps.newHashMap();
        for (Object key : props.keySet()) {
            String keyStr = key.toString();
            String value = props.getProperty(keyStr);
            ctxPropertiesMap.put(keyStr, value);
        }
    }

    protected void loadProperties(Properties props) throws IOException {
        if (locations != null) {
            logger.info("Start loading properties file.....");
            for (Resource location : locations) {
                InputStream is = null;
                try {
                    String filename = location.getFilename();
                    String env = System.getProperty("run.env", "") + ".properties";
                    if (filename.contains(env)) {
                        logger.info("Loading properties file from " + location);
                        is = location.getInputStream();
                        propertiesPersister.load(props, is);
                    }
                } catch (IOException ex) {
                    logger.info("读取配置文件失败.....");
                    ex.printStackTrace();
                    throw ex;
                } finally {
                    if (is != null) {
                        is.close();
                    }
                }
            }
        }

    }

    public static String getValue(String key) {
        return ctxPropertiesMap.get(key);
    }
```


