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

## 2. Spring Cloud Netflix

### 2.1 Service Discovery: Eureka Server

#### 2.1.1 How to include

To include Eureka Server in your project, use the starter with a group ID of <font color="#AA0000">org.springframework.cloud</font> and an artifact ID of <font color="#aa0000">spring-cloud-starter-netflix-eureak-server</font>. 

> <font color="red">Warning:</font> if your project use Thymeleaf as it template engine, the Freemarker templates of the eureka server may not be loaded correctly. In this case, it is necessary to configure the template loader manually: 

``` yaml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```

#### 2.1.2 How to run

The following example shows how to run Eureka Server:

``` java
@SpringBootApplication
@EnableEurekaServer
public class Application {
    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class)
            .web(true).run(args);
    }
}
```

#### 2.1.3 High availability, Zones & Regions

The Eureka Server does not have a back-end store, but the service instances in the registry all have to send heartbeats to keep their registration up to date.

Clients also have an in-memory cache of Eureka registration (so they do <i><b>Not</b></i> have go to the registry for every request to a service)

By default, every Eureka Server is also a Eureka client and requires (at least one) service URL to locate a peer.

#### 2.1.4 Standalone mode

The combination of the two caches (client and server) and the heartbeats make a standalone Eureka Server fairly resilient to failure, as long as there is some sort of monitor or elastic runtime keeping it alive. In standalone mode, some configurations are switched off.

``` yaml
eureka:
  instance:
    hostname: localhost
   client:
     registerWithEureka: false
     fetchRegistry: false
     serviceUrl:
       defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### 2.1.5 When to prefer IP address

Set <font color="#aa0000">eureka.instance.preferIpAddress</font> to <font color="#aa0000">true</font> and, when the application registers with eureka, it uses IP address rather than host-name.

#### 2.1.6 Securing Eureka Server

You can secure your Eureka server simply by adding Spring Security to your server's classpath via <font color="#aa0000">spring-boot-starter-security</font>. By default when Spring Security is on the classpath it will require that a valid CSRF token be sent via every request to the app. You can disable it by setting,

``` java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatches("/eureka/**");
        super.configure(http);
    }
}
```

### 2.2 Service Discovery: Eureka Client

#### 2.2.1 How to include

To include the Eureka Client in your project, use the starter with an artifact ID of <font color="#aa0000">spring-cloud-starter-netflix-eureka-client</font>.

#### 2.2.2 Register with Eureka

When a client registers with Eureka, it provides meta-data about itself - such as host, port, health indicator URL, home page, and other details.

Eureka receives heartbeat messages from each instance belonging to a service. If the heartbeat fails over a configurable timetable, the instance is normally removed from the registry.

To disable the Eureka Discovery Client, you can either set <font color="#aa0000">eureka.client.enabled</font> or <font color="#aa0000">spring.cloud.discovery.enabled</font> to <font color="#aa0000">false</font>. 

#### 2.2.3 Authenticating with the Eureka Server

HTTP basic authentication is automatically added to your eureka client if one of the <font color="#aa0000">eureka.client.serviceUrl.defaultZone</font> URLs has credentials embedded in it (curl style, as follows: username:password@localhost:8761/eureka).

For more complex needs, create a <font color="#aa0000">@Bean</font> of type <font color="#aa0000">DiscoveryClientOptionArgs</font> and inject <font color="#aa0000">ClientFilter</font> instance to it.

> Because of a limitation in Eureka, it is not possible to support per-server basic auth credentials, so only the first that are found is used.

#### 2.2.4 Status & Health page

The status page and health indicators for a Eureka instance default to <font color="#aa0000">/info</font> and <font color="#aa0000">/health</font> respectively, which are the default locations of the useful endpoints in a Spring Boot Actuator Application. You can change the settings,

``` yam
eureka:
  instance:
    status-page-url-path: ${server.servletPath}/info
    health-page-url-path: ${server.servletPath}/health
```

#### 2.2.5 Registering a Secure Application

If your app wants to be contacted over HTTPS, you can set two flags in the <font color="#aa0000">EurekaInstanceConfig</font>:

- <font color="#aa0000">eureka.instance.[nonSecurePortEnabled]=[false]</font>
- <font color="#aa0000">eureka.instance.securePortEnabled=true</font>