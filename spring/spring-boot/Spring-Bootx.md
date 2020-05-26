# Spring Boot

## 1. introduce

### 1.1 what is spring boot?

### 1.2 Why Spring Boot?

### 1.3 How to use Spring Boot?

There are two ways to use Spring Boot,

- inherit
- dependency

#### 1.3.1 inherit

Inherit from its parent, <font color="#AA0000">spring-boot-starter-parent</font>,

``` xml
<parent>
	<groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
    <relativePath />
</parent>
```

#### 1.3.2 dependency

Add dependency of <font color="#aa0000">spring-boot-dependencies</font>,

```xml
<dependencyManagement>
	<dependencies>
    	<dependency>
        	<groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.2.RELEASE</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### 1.3.3 starting

``` java
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

``` java
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.run(args);
    }
}
```



## 2. Core Features

### 2.1 Basic configuration

Spring Boot favors <font color="#aa0000">Java-based</font> configuration. It is recommended that your primary source can be a single <font color="#aa0000">@Configuration</font> class. 

#### 2.1.1 Importing additional configuration classes

The <font color="#aa0000">@Import</font> annotation can be used to import additional configuration classes. 

Alternatively, <font color="#aa0000">@ComponentScan</font> can be used to automatically pick up all Spring components, including <font color="#a00">@Configuration</font> classes.

#### 2.1.2 Importing XML configuration

If you absolutely must use XML-based configuration, it is strongly recommended that you still start with a <font color="#a00">@Configuration</font> class. Then use <font color="#a00">@ImportResource</font> annotation to load XML configuration files.

#### 2.1.3 Auto-configuration

Spring Boot auto-configuration attempts to automatically configure your Spring application based on the <u>jar dependencies</u> that you have added.

You need opt-in to auto-configuration by adding the <font color="#0aa">@EnableAutoConfiguration</font> or <font color="#0aa">@SpringBootConfiguration</font> annotations to one of your <font color="#0aa">@Configuration</font> classes. 

> it is recommended that you add one or the other to your primary <font color="#0aa">@Configuration</font> class.

##### 2.1.3.1 Disable specific auto-configuration classes

If you do not want specific auto-configuration to be applied, use the <u>exclude</u> attribute of <font color="#0aa">@EnableAutoConfiguration</font> to disable them.

``` java
@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {}
```

##### 2.1.3.2 Gradually replacing auto-configuration

Auto-configuration is non-invasive. At any point, you can start to define your own configuration to replace specific parts of of the auto-configuration. Use <font color="#0aa">--debug</font> to find out what auto-configuration is currently applied. 

### 2.2 External configuration

You can use properties files, YAML files, environment variables, and command-line arguments to externalize configuration. Property values can be injected directly into your beans by using <font color="#0aa">@Value</font> annotation, accessed through Spring's <font color="#0aa">Environment</font> abstraction, or be <u>bound to structured objects</u> through <font color="#0aa">@ConfigurationProperties</font>.

#### 2.2.1 Command-line arguments

By default, <font color="#0aa">SpringApplication</font> converts any command line option arguments (that is, arguments starting with<font color="#0aa">--</font>, such as <font color="#0aa">--server.port=9000</font>)

If you do not want command line properties to be added to the environment, you can disable them by using <font color="#0aa">SpringApplication.setAddCommandLineProperties(false)</font>.

#### 2.2.2 Application Property files

<font color="#0aa">SpringApplication</font> loads properties from <font color="#0aa">application.properties</font> files in following locations, and adds them to the Spring <font color="#0aa">Environment</font>,

1. A <font color="#0af">/config</font> subdirectory of the current directory

2. The current directory

3. A classpath <font color="#0af">/config</font> package

4. The classpath root

   ordered by precedence (properties higher override those defined in lower locations)

   > <font color="#0af">YAML ('.yml')</font> files are an alternative to '.properties'

<font color="#aa0">spring.config.name</font> environment property is specified to switch the configuration file name, default <u><i>application</i></u>, to another.

<font color="#aa0">spring.config.location</font> environment property is specified to refer to an explicit location. Files specified in <font color="#aa0">spring.config.location</font> are used as-is, with <u>no support</u> for profile-specific variants, and are overridden by any profile-specific properties.

> <font color="#aa0">spring.config.name</font> and <font color="#aa0">spring.config.location</font> are used very early to determine which files have to be loaded. They must be defined as an environment property (typically an OS environment variable, a system property, or a command-line argument)

#### 2.2.3 Profile-specific properties

Profile-specific properties are defined by using the following naming convention: <font color="#aa0">application-{profile}.properties</font>. Profile specific properties are loaded from the same locations as standard <font color="#aa0">application.properties</font>, with profile-specific files <u>always overriding</u> the non-specific ones, whether or not the profile-specific files are inside or outside your packaged jar.

> If you have specified any files in <font color="#aa0">spring.config.location</font>, profile-specific <u>variants</u> of those files are not considered. Use directories in <font color="#aa0">spring.config.location</font> if you want to also use profile-specific properties.

#### 2.2.4 Encrypting Properties

The <font color="#aa0">EnvironmentPostProcessor</font> interface allows you to manipulate the <font color="#aa0">Environment</font> before the application start.

### 2.3 Logging

By default, if you use <b><u>starters</u></b>, Logback is used for logging.

> When you deploy your application to a servlet container or application server, logging performed via the Java Util logging API is not routed into your application's log.

#### 2.3.1 File output

By default, Spring Boot logs only to console, not to log files. <font color="#aa0">logging.file.name</font> or <font color="#aa0">logging.file.path</font> property can be set to achieve the target to write logs to log files.

| logging.file.name | logging.file.path  |                                                              |
| ----------------- | ------------------ | :----------------------------------------------------------- |
| (none)            | (none)             | console by logging                                           |
| specific file     | (none)             | writes to the specific log file. Names can be exact location or relative to current directory |
| (none)            | specific directory | writes <font color="#0aa">spring.log</font> to the specified directory. Names can be an exact location or relative to the current directory |

 <font color="#aa0">logging.file.max-size</font> can be set to <u>limit the file size</u>.

Previously rotated files are archived indefinitely unless the <font color="#aa0">logging.file.max-history</font> property has been set

The <u>total size of log archives</u> can be capped using <font color="#aa0">logging.file.total-size-cap</font>.

To force <u>log archive cleanup</u> on application startup, use the <font color="#aa0">logging.file.clean-history-on-startup</font> property

#### 2.3.2 Log levels

All the supported logging system can have the logger levels set in the Spring <font color="#aa0">Environment</font> by using <font color="#aa0">logging.level.<logger-name>=<level></font> where the <font color="#aa0">level</font> is one of TRACE, DEBUG, INFO, WARN, ERROR, FETAL, or OFF.

``` properties
logging.level.root=INFO
logger.level.org.springframework.web=DEBUG
logger.level.org.hibernate=ERROR
```

#### 2.3.3 Log groups

It's often useful to be able to group related loggers together so that they can all be configured at the same time, by using <font color="#aa0">logging.group.<group-name>=<packages></font>.

```properties
logging.group.tomcat=org.tomcat.catalina, org.apache.coyote
```

once defined, you can change the level for all the loggers in the group with single line,

```properties
logging.level.tomcat=INFO
```

#### 2.3.4 Custom Log Configuration

The various logging systems can be activated by including the appropriate libraries on the classpath and can be further customized by providing a suitable configuration file in the root of the classpath or in a location by the following Spring <font color="#aa0">Environment</font> property: <font color="#aa0">logging.config</font>.

Depending on your logging system, the following files are added,

| Logging System | Customization                                                |
| -------------- | ------------------------------------------------------------ |
| Logback        | logback-spring.xml, logback-spring.groovy, logback.xml or logback.groovy |
| Log4j2         | log4j2-spring.xml or log4j2.xml                              |
| JDK            | logging.properties                                           |

> When possible, it is strongly recommended you use the <font color="#0aa">-spring</font> variants for your logging configuration

### 2.4 Profiles

Spring profiles provides a way to segregate parts of your application configuration and make it be available only in certain environments.

Any <font color="#aa0">@Component</font>, <font color="#aa0">@Configuration</font>, <font color="#aa0">@ConfigurationProperties</font> can be marked with <font color="#aa0">@Profile</font> to limit the when it is loaded.

```java
@Configuration(proxyBeanMethods = false)
@Profile("prod")
public class ProdConfig {
    // ... configuration goes here
}
```

>if <font color="#aa0">@ConfigurationProperties</font> beans are registered via <font color="#aa0">@EnableConfigurationProperties</font> instead of automatic scanning, the <font color="#aa0">@Profile</font> annotation needs to be specified on the <font color="#aa0">@Configuration</font> class that has <font color="#aa0">@EnableConfigurationProperties</font>.

<font color="#aa0">spring.profiles.active</font> can be set to specify which profiles are active.

<font color="#aa0">spring.profiles.include</font> or <font color="#aa0">SpringApplication.setAdditionalProfiles(...)</font> programatically,  can be set to <b><u>add</u></b> profile-specific properties to the active profile rather than replace them.

### 2.5 Internals

#### 2.5.1 Startup Failure

If your application fails to start, registered <font color="#aa0">FailureAnalyzers</font> get a chance to provide a dedicated error message and a concrete action to fix the problem.

> Spring Boot provides numerous <font color="#aa0">FailureAnalyzer</font> implementations, and you can add your own.

#### 2.5.2 Lazy Initialization

<font color="#aa0">SpringApplication</font> allows an application to be initialized <u>lazily</u>. When lazy initialization is enabled, beans are created as they are needed rather than during application startup.

Lazy initialization can reduce the time that it takes your application to start.

A <u>downside</u> of lazy initialization is that it can delay the discovery of a problem with the application.

Lazy initialization can be enabled programatically using the <font color="#aa0">LazyInitialization</font> method on <font color="#aa0">SpringApplicationBuilder</font> or the <font color="#aa0">setLazyInitialization</font> method on <font color="#aa0">SpringApplication</font>.

Alternatively, it can be enabled using the <font color="#aa0">spring.main.lazy-initialization</font> property.

#### 2.5.3 Application Events and Listeners

There are three ways to register listeners,

- using <font color="#aa0">@Bean</font>, if the events it listens on are triggered after the <font color="#aa0">ApplicationContext</font> is created
- register the listeners if events are triggered before <font color="#aa0">ApplicationContext</font> is created, via <font color="#aa0">SpringAppication.addListeners(~)</font> or <font color="#aa0">SpringApplicationBuilder.listeners(~)</font>
- add a <font color="#aa0"><u>META-INF/spring.factories</u></font> file to your project and reference your listeners by using the key as <font color="#aa0">org.springframework.context.ApplicationListener</font>

#### 2.5.4 Web Environment

A <font color="#aa0">SpringApplication</font> attempts to create right type of <font color="#aa0">ApplicationContext</font> on your behalf. The algorithm to determine a <font color="#aa0">WebApplicationType</font> is fairly simple:

- If Spring MVC is present, an <font color="#aa0">AnnotationConfigServletWebServerApplicationContext</font> is used.
- If Spring MVC is not present and Spring <b><u>WebFlux</u></b> is present, then an <font color="#aa0">AnnotationConfigReactiveWebServerApplicationContext</font> is used.
- otherwise, <font color="#aa0">AnnotationConfigApplicationContext</font> is used

> it is often desirable to call <font color="#aa0">setWebApplicationType(WebApplicationType.NONE)</font> when using it in a JUnit test.

#### 2.5.5 Accessing Application Arguments

If you need to access the application arguments that were passed to <font color="#aa0">SpringApplication.run(...)</font>, you can inject a <font color="#aa0">org.springframework.boot.ApplicationArguments</font> bean.

``` java
@Component
public class AppArgus {
    @Autowired
    public AppArgus(final ApplicationArguments args) {
        boolean exists = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if passed in --debug fil1.log then exists = true, files: fil1.log
    }
}
```

#### 2.5.6 Using ApplicationRunner or CommandLineRunner

If you want to run some specific code once the <font color="#aa0">SpringApplication</font> has started, you can implement the <font color="#aa0">ApplicationRunner</font> or <font color="#aa0">CommandLineRunner</font> interfaces. 

Both interfaces work in the same way and offers a single <font color="#aa0">run</font> method, which is called just <u>before</u> <font color="#aa0">SpringApplication.run(...)</font> <u><b>completes</b></u>.

``` java
@Component
public class MyAppRunner implements ApplicationRunner {
    public void run(final ApplicationArguments args) {
        // code runs once the SpringApplication has started, before run() completes.
    }
}
```

#### 2.5.7 Application exit

<font color="#aa0">ExitCodeGenerator</font> will be implemented when a specific exit code is desired.

#### 2.5.8 Admin feature

You can enable admin-related features for the application by specifying the <font color="#aa0">spring.application.admin.enabled</font> property. This exposes the <font color="#aa0">SpringApplicationMXBean</font> on the platform <font color="#aa0">MBeanServer</font>.

### 2.6 Internationalization

Spring Boot supports localized messages so that your application can cater to users of different language preferences.

The base-name of the resource bundle as well as several other attributes can be configured using the <font color="#aa0">spring.messages</font> namespace,

``` properties
spring.messages.basename=messages, config.i18n.messages
spring.messages.fallback-to-system-locale=false
```



## 3. Web development

### 3.1 MVC

### 3.2 Embedded Container

## 4. Working with Data

### 4.1 SQL

### 4.2 NoSQL

#### 4.2.1 Redis

Redis is a <font color="#aa0">cache, <b>message broker</b>, richly-featured key-value</font> store. Spring Boot offers basic auto-configuration for the <font color="#aa0">Lettuce</font> and <font color="#aa0">Jedis</font> client libraries and the abstractions on top of them.

A <font color="#aa0">spring-boot-starter-data-redis</font> "Starter" is introduced for collecting the dependencies in a convenient way. By default, it uses <font color="#aa0">Lettuce</font>.

> a <font color="#aa0">spring-boot-starter-data-redis-reactive</font> "Starter" is provided for consistency with the other stores with reactive support.

##### connecting to Redis

You can inject an auto-configured <font color="#aa0">RedisConnectionFactory</font>, <font color="#aa0">StringRedisTemplate</font>, or vanilla <font color="#aa0">RedisTemplate</font> instance as you would.

``` java
@Component
public class MyBean {
    private StringRedisTemplate stringRedisTemplate;
    
    @Autowired
    public MyBean(final StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }
    
    // code goes here
}
```



## 5. Messaging

### 5.1 JMS

## 6. Testing

## 7. Extending

## 8. Monitoring

## 9. Management endpoints

## 10. Connection options