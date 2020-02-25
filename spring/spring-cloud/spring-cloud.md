# Spring Cloud

## 1. Cloud Native Applications

<u>Cloud Native</u> is a style of application development that encourages easy adoption of best practices in the areas of continuous <font color='red'>delivery</font> and <font color='red'>value-driven</font> development. 

Two extra libraries are provided by <u>Spring Cloud</u>, which is built upon <font color='light blue'> Spring Boot</font> 

- Spring Cloud context: provides utilities and special services for the <font color='red'>ApplicationContext</font> of a Spring Cloud Application (bootstrap context, encryption, refresh scope, and environment endpoints)
- Spring Cloud Commons: a set of abstractions and common classes used in different Spring Cloud implementations

### 1.1 Spring Cloud Context: Application Context Services

#### 1.1.1 The Bootstrap Application Context

A Spring Cloud application operates by creating a "bootstrap" context, which is a parent context for the main application. This context is responsible for loading configuration properties from the external sources and for decrypting properties in the local external configuration files.

By default, bootstrap properties (properties that loaded during the bootstrap phase) are added with high precedence, so they **CANNOT** be *overridden* by local configuration.

Bootstrap phase can be completely disabled by setting <font color='red'>spring.cloud.bootstrap.enabled</font> (in system properties) like following,

``` properties
spring.cloud.bootstrap.enabled = false
```

#### 1.1.2 Application Context Hierarchies

If you build an application context from <font color='red'>SpringApplication</font> or <font color='red'>SpringApplicationBuilder</font>, the Bootstrap context is added as a parent to that context. The additional property sources are:

- "bootstrap": if any <font color='red'>PropertyPlaceHolders</font> are found in the bootstrap context and if they have non-empty properties, an optional <font color='red'>CompositePropertySource</font> appears with high priority
- "applicationConfig: [classpath:bootstrap.yml]": if a <font color='red'>bootstrap.yml</font> (or <font color='red'>.properties</font>) presented. They have lower precedence than the <font color='red'>application.yml</font> (or <font color='red'>.properties</font>) and any other property sources that are added to the child as a normal part of process of creating a Spring Boot Application.

#### 1.1.3 Changing the Location of Bootstrap Properties

You can specify the <font color='red'>bootstrap.yml</font> (or <font color='red'>.properties</font>) location by setting <font color='red'>spring.cloud.bootstrap.name</font> or <font color='red'>spring.cloud.bootstrap.locaton</font>. 

#### 1.1.4 Overriding the Values of Remote Properties

The property sources that are added to your application by the bootstrap context are often "remote" (for example, from Spring Cloud Config Server). By default, they cannot be overridden locally. If you want to override, the remote property source has to grant it permission by setting <font color='red'>spring.cloud.config.allowOverride=true</font> (does not work locally). Under that circumstances, two other factors should be paid attention to,

- <font color='red'>spring.cloud.config.overrideNone=true</font>: Override from any local property source
- <font color='red'>spring.cloud.config.overrideSystemProperties=false</font>: only system properties, command line arguments, and environment variables should override the remote settings

#### 1.1.5 Customizing the Bootstrap Configuration

The bootstrap context can be set to do anything you like by adding entries to <font color='red'>/META-INF/spring.factories</font> under a key named <font color='red'>org.springframework.cloud.bootstrap.BootstrapConfiguration</font>. This holds a comma-separated list of Spring <font color='red'>@Configuration</font> classes that are used to create the context.

> when adding custom <font color='red'>BootstrapConfiguration</font>, be careful that the classes you add are not <font color='red'>@ComponentScanned</font> by mistake into your "main" application context, where they might not be needed. Use a separate package name for boot configuration classes and make sure that name is not already covered by your <font color='red'>@ComponentScan</font> or <font color='red'>@SpringBootApplication</font> annotated configuration classes.

#### 1.1.6 Customizing the Bootstrap Property Sources

The default property source for external configuration added by the bootstrap process is the Spring Cloud Server, but you can add additional sources by adding beans of type <font color='red'>PropertySourceLocator</font> to the bootstrap context (through <font color='red'>spring.factores</font>). For example,

``` java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {
    @Override
    public PropertySource<?> locate(Environment env) {
        return new MapPropertySource("customProperty", Collections.<String, Object>singltonMap("property.from.sample.custom.source", "work as intended"));
    }
}
```

you added a <font color='red'>/META-INF/spring.factories</font> containing the following setting,

``` properties
org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```

#### 1.1.7 Logging Configuration

If you use Spring Boot to configure log settings, you should place this configuration in <font color='red'>bootstrap.[yml | properties]</font> if you would like it to apply to all events.

#### 1.1.8 Environment Changes

The application listens for an <font color='red'>EnvironmentChangeEvent</font> and reacts to the change in a couple of standard ways. When an <font color='red'>EnvironmentChangeEvent</font> is observed, it has a list of key values that have changed, and the application uses those to,

- Re-bind any <font color='red'>@ConfigurationProperties</font> beans in the context
- Set the logger levels for any properties in <font color='red'>logging.level.*</font> 