# Spring Boot
## 1. Using Spring Boot
### 1.1 Build Systems
two ways via using Maven,
* Inheriting the Starter parent
* Using Spring boot without parent POM

#### Inheriting the Starter parent
To configure your project to inherite from the <font size='2' color='blue'>spring-boot-starter-parent</font>, set as the followings,
``` xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.3-RELEASE</version>
</parent>
```

> <b>Note</b>
> You should need to specify only the Spring boot version number in this dependency

#### Using Spring boot without parent POM
If you wanna have your own corporate parent, you can still keep the benefit of the dependency management by using a <font size='3' color='blue'>scope=import</font> dependency.
``` xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-dependencies</artifactId>
      <version>2.1.3-RELEASE</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

> <b>Note</b>
> To override individual dependencies specified in <font size='2' color='blue'>sprint-boot-dependencies</font>, you need to add an entry in the dependencyManagement of your project <strong>before</strong> the <font size='2'>spring-boot-dependencies</font>.

#### 1.1.1 Starters
Starters are a set of convenient dependency descriptors that you can include in your application. You get a one-stop shop for all the Spring and related technologies that you need without having to hunt through sample code and copy-paste loads of dependency descriptors.
> What's in the name
> An <strong>official</strong> starters follow a similiar naming pattern; <font size='3' color='blue'>spring-boot-starter-*</font>, where * is a particular type of application.
> A third-party starter project called thirdpartyproject would typically be named thirdpartyproject-spring-boot-starter.

### 1.2 Structuring your code
#### 1.2.1 Locating the main application class
generally recommend that you locate your main application class in a root package above other classes.
``` java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }
}
```

### 1.3 Configuration classes
Spring Boot favors Java-based configuration. Although it is possible to use <font color='gray' size='3'>SpringApplication</font> with XML sources, we generally recommend that your primary source be a single <font color='gray' size='3'>@Configuration</font> class.
> <strong>Tips</strong>
> If possible, always try to use the equivalent Java-based configuration. Searching for <font color='gray' size='3'>Enable*</font> annotations can be a good starting point.

* Using <font color='gray' size='3'>@Import</font> annotation to import additional configuration classes, alernatively using <font color='gray' size='3'>@ComponentScan</font> to automatically import configuration classes.
* Using <font color='gray' size='3'>@ImportResource</font> annotation to load XML configuration files.

### 1.4 Auto-Configuration
Spring Boot auto-configuration attempts to automatically configure your Spring application based on the jar dependencies that you have added.

you need to opt-in to auto-configuration by adding <font size='3' color='gray'>@EnableAutoConfiguration</font> or <font size='3' color='gray'>@SpringBootApplication</font> annotations to one of your <font size='3' color='gray'>@Configuration</font> classes.

#### 1.4.1 Disabling Specific Auto-Configuration Classes
if you find that specific auto-configuration classes that you do not want are being applied, you can use exclude attribute of <font color='#cccccc' size='3'>@EnableAutoConfiguration</font>.

``` java
@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```
Finally, you can also control the list of auto-configuration classes to exclude by using the <font color='gray' size='2'>spring.autoconfigure.exclude</font> property.

> <strong>Tip</strong>
> you can define exclusions both at the annotation level and by using the property.

### 1.5 Using the @SpringBootApplication Annotation
A single @SpringBootApplication annotation can be used to enable those three features, they are,
* <font color='blue'>@EnableAutoConfiguration</font>: enable Spring Boot's auto-configuration mechnism
* <font color='blue'>@ComponentScan</font>: enable @Component scan on the package where the application is located
* <font color='blue'>@Configuration</font>: allow to register extra beans in the context or import additional configuration classes.

> <strong>Note</strong>
> @SpringBootApplication also provides aliases to customize the attributes of @EnableAutoConfiguration and @ComponentScan

### 1.5 Developer Tools

``` text
Restart VS Reload

The restart technology provided by the Spring Boot works by using two classloaders.
Classes that do not change are loaded into a base classloader. Classes that you are actively developing are loaded into a restart classloader.
```

##### Exclude Resources
Certain resources do not necessarily need to trigger a restart when they are changed. if you want to customize these exclusions, you can use the <font size='2' color='gray'>spring.devtools.restart.exclude</font> property.
``` properties
spring.devtools.restart.exclude=static/**,public/**
```

> <strong>Tip</strong>
> If you want to keep those defaults and <i>add</i> additional exclusions, use the spring.devtools.restart.additional-exclude property instead.

##### Disabling Restart
If you do not want to use the restart feature, you can disable it by using the <font color='gray' size='2'>spring.devtools.restart.enabled</font> property in your application.properties (<font color='blue' size='2'> doing so still initializes the restart classloader, but it does not watch for file changes</font>).

If you need to <i>completely</i> disable restart support, you need to set the <font size='2' color='gray'>spring.devtools.restart.enabled</font> System property to false before calling SpringApplication.run(...).

If you work on multi-module project, and not every module is imported into your IDE, you may need to customize things. To do so, you can create a META-INF/spring-devtools.properties file. <strong>Prefixed with</strong>
* restart.exclude: items that should be pushed down into the <i>base</i> classloader
* restart.include: itmes that should be pushed down into the <i>restart</i> classloader

For example:
``` properties
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
restart.exclude.companycommonlibs=/mycorp-myproj-[\\w-]+\.jar
```

## 2. Spring Boot features
### 2.1 SpringApplication
#### 2.1.1 Application Events and Listeners
In addition to the usual Spring Framework events, such as <font color='blue' size='3'>ContextRefreshEvent</font>, a SpringApplication sends some additional application events.
> <strong>Note</strong>
> Some events are actually triggered before the <font color='gray' size='3'>ApplicationContext</font> is created. So you cannot register a listener on those as a @Bean. You can register them with the <font color='gray' size='3'>SpringApplication.addListeners(...)</font> method or the <font color='gray' size='3'>SpringApplicationBuilder.listeners(...)</font> method.
> If you want them registered automatically, added a META-INF/spring.factories by using the <font size='2' color='gray'>org.springframework.context.ApplicationListener</font> key, as followings,
> ``` properties
> org.springframework.context.ApplicationListener=com.example.MyListener
> ```

Application events are sent in the following order, as your application runs:
1. An <font size='2' color='blue'>ApplicationStartingEvent</font> is sent at the start of a run but before any processing, except for the registration of listeners and initializers.
2. An <font size='2' color='blue'>ApplicationEnvironmentPreparedEvent</font> is sent when the <font size='2' color='blue'>Environment</font> to be used in the context is known but before the context is created.
3. An <font size='2' color='blue'>ApplicationPreparedEvent</font> is sent just before the refresh is started but after bean definitions have been loaded.
4. An <font size='2' color='blue'>ApplicationStartedEvent</font> is sent after the context has been refreshed but before any application and command-line runners have been called.
5. An <font size='2' color='blue'>ApplicationReadyEvent</font> is sent after the any application and command-line runners have been called. it indicates the application is ready to service the request.
6. An <font size='2' color='blue'>ApplicationFailedEvent</font> is sent if there is an exception on startup.

#### 2.1.2 Web Environment
The algorithm used to determine a <font color='gray' size='2'>WebApplicationType</font> is fairly simple:
* if Spring MVC presents, an <font color='blue' size='2'>AnnotationConfigServletWebServerApplicationContext</font> is used.
* if Spring WebFlux presents, an <font color='blue' size='2'>AnnotationConfigReactiveWebServerApplicationContext</font> is used.
* otherwise, <font size='2' color='blue'>AnnotationConfigApplicationContext</font> is used.

#### 2.1.3 Accessing Application Arguments
If you need to access the application arguments that were passed to <font color='gray'>SpringApplication.run(...)</font>, you can inject a <font size='2'> org.springframework.boot.ApplicationArguments</font> bean. The <font size='2'>ApplicationArguments</font> interface provides access to both the raw <font color='gray'>String[]</font> arguments as well as parsed <font color='gray'>option</font> and <font color='gray'>non-option</font> arguments.

``` java
@Component
public class MyBean {
  @AutoWired
  public MyBean(final ApplicationArguments args) {
    boolean debug = args.containsOption("debug");
    List<String> files = args.getNonOptionArgs();
    // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
  }
}
```

#### 2.1.4 Using ApplicationRunner or CommandLineRunner
If you want to run some specific code once the SpringApplication has started, you can implement the <font size='2'>ApplicationRunner</font> or <font size='2'>CommandLineRunner</font> interfaces.

``` java
@Component
public class MyBean implements CommandLineRunner {
  public void run(String... args) {
    // do something here
  }
}
```

#### 2.1.5 Admin Features
It is possible to enable admin-related features for the application by specifying the <font size='2'>spring.application.admin.enabled</font> property. This exposes the <font color='blue' size='3'>SpringApplicationAdminMXBean</font> on the platform MBeanServer.

> <strong>Caution</strong>
> Take care when enabling this feature, as the MBean exposes a method to shutdown the application.

### 2.2 Externalized Configuration
SpringBoot uses a very particular <font color='gray'>PropertySource</font> order that is designed to allow sensible overriding of values. Properties are considered in the following order:
1. devtools global setting properties on your home directory (~/.spring-boot-devtools.properties when devtools activate)
2. @TestPropertySource annotations on your tests
3. <font color='gray'>properties</font> attribute on your tests. Available on @SpringBootTest and the test annotations for testing particular slice of your application.
4. command line Arguments
5. Properties from <font size='3' color='gray'>SPRING_APPLICATION_JSON</font> (inline JSON embeded in an environment variable or system property)
6. <font color='gray'>ServletConfig</font> init parameters
7. <font color='gray'>ServletContext</font> init parameters
8. JNDI attribtues from java:comp/env
9. Java System properties (System.getProperties())
10. OS environment variables
11. A <font color='grey'>RandomValuePropertySource</font> that has properties only in random.*
12. Profile specific application properties outside of your packaged jar
13. Profile specific application properties inside of your packaged jar
14. Application properties outside of your packaged jar
15. Application properties inside of your packaged jar
16. <font color='grey'>@PropertySource</font> annotation on your <font color='grey'>@Configuration</font> classes
17. Default properties (specified by setting SpringApplication.setDefaultProperties)

> <strong>Tip</strong>
> The <font size='3' color='grey'>SPRING_APPLICATION_JSON</font> properties can be supplied on the command line with an environment variable.
> ``` bash
> $ SPRING_APPLICATION_JSON='{"name":"test"}' java -jar myapp.jar
> ```
>
> you can also supply the JSON as <font color='grey' size='2'>spring.application.json</font> in a System property, as followings,
> ``` bash
> $ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
> ```
>
> also can by using a command line argument
> ``` bash
> $ java -jar myapp.jar --spring.application.json='{"name":"test"}'
> ```

#### 2.2.1 Application Property Files
<font color='grey'>SpringApplication</font> loads properties from <font color='grey'>application.properties</font> file in the following locations and adds them to the Spring <font color='grey'>Environment</font>
1. A /config subdirectory of the current directory
2. The current directory
3. A classpath /config package
4. the classpath root

> <strong>Note</strong>
> If you do not like <font color='grey'>application.properties</font> as the configuration file name, you can switch to another file name by specifying a <font color='grey'>spring.config.name</font> environment property. you can also refer to an explicit location by using the <font color='grey'>spring.config.location</font> environment property (which is a comma-separated list of directory location or file paths).
> ``` bash
> $ java -jar myapp.jar --spring.config.location=classpath:/default.properties
> ```

#### 2.2.2 Placeholders in Properties
you can refer back to previously defined values

``` properties
app.name=MyApp
app.description=${app.name} is a SpringBoot application
```

#### 2.2.3 Using YAML instead of Properties
<font color='blue'>YAML</font> is a superset of JSON and, as such, is a convenient format for specifying hierarchical configuration data.
> <strong>Note</strong>
> If you use <i>Starters</i>, SnakeYAML is automatically provided by <font color='grey' size='2'>spring-boot-starter</font>

#### 2.2.4 Type-safe Configuration Properties
Using the <font color='grey'>@Value("${property}")</font> annotation to inject configuration properties.
Spring Boot provides an alternative method of working with properties that lets strongly typed beans govern and validate the configuration of your application.

To avoid injecting other beans from the context, the <font color='grey'>@ConfigurationProperties</font> is recommended only deal with the environment.
``` java
@Component
@ConfigurationProperties(prefix = 'acme')
public class AcmeProperties {
  // defines property acme.name
  private String name;

  // getters and setters
}
```

##### Relaxed Binding
SpringBoot uses some relaxed rules for binding <font color='grey'>Environment</font> properties to <font color='grey'>@ConfigurationProperties</font> beans, so there does not need to be an exact match between the <font color='grey'>Environment</font> property name and the bean property name.

When binding to <font color='grey'>Map</font> properties, if the <font color='grey'>key</font> contains anything other than lowercase alpha-numeric characters or -, you need to use the bracket notation so that the original value is preserved.

``` YAML
acme:
 map:
  "[/key1]": value1
  "[/key2]": value2
  /key3: value3
```
results in /key1, /key2 and <font color='blue' size='2'>key3</font> as a key in this Map.

##### @ConfigurationProperties vs. @Value
the comparison as followings,
| Feature          | @ConfigurationProperties  |   @Value  |
|:-----------------|:-------------------------:|:---------:|
| Relaxed Binding  | Yes                       | No        |
|Meta-data support | Yes                       | No        |
|spEL Expression   | No                        | No        |

### 2.3 Profiles
Spring Profiles provide a way to segregate parts of your application configuration and make it be Available only in certain environments. Any <font color='grey'>@Component</font> or <font color='grey'>@Configuration</font> can be marked with <font color='grey'>@Profile</font> to limit when it is loaded.

``` java
@Configuration
@Profile("production")
public class ProdConfiguration {
  // configuration goes here
}
```

As well, can specify <font color='grey'>spring.profiles.active Environment</font> property to specify which profile are active.
``` properties
spring.profiles.active=prod,hsqldb
```

Instead replace the profiles, you can use <font color='grey'>spring.profiles.include</font> property to <b>add</b> additional profiles to current active profile.

``` YAML
spring
 profiles: prod
   include:
     - proddb
     - prodmq
```

### 2.4 Logging
By default, Spring Boot logs only to console and does not write log files. If you want to write log files in addition to the console output, you need to set a <font color='grey'>logging.file</font> or <font color='grey'>logging.path</font> properties.

| logging.file   | logging.path    | Example        | Description  |
|:----------|:--------|:------|:-----------------------------------|
| None  | None | | Console by logging
| Specific file  | None      | my.log | writes to the specified log file. Names can be an exact location or relative to the current directory |
| None | Specific directory | /var/log | Writes <font color='blue'>spring.log</font> to the specified directory. Names can be an exact location or relative to current directory |

> <strong>Note</strong>
> The logging system is initialized early in the application lifecycle. Consequently, logging properties are not found in property file loaded through <font color='grey'>@PropertySource</font> annotations.

##### Logger Groups
SpringBoot allows you to define logging groups in your Spring <font color='grey'>Environment</font>.

``` Properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
logging.level.tomcat=TRACE
```

### 2.5 JSON
SpringBoot provides integration with three JSON mapping libraries,
* Gson
* Jackson
* JSON-B

Jackson is preferred and default library

### 2.6 Developing Web Application
##### Spring MVC Auto-configuration
The auto-configuration adds the following features on the top of Spring's defaults,
* inclusion of <font color='grey'>ContentNegotiatingViewResolver</font> and <font color='grey'>BeanNameViewResolver</font> beans.
* support for serving static resources, including support for WebJars.
* Automicatic registration of <font color='grey'>Converter, GenericConverter</font> and <font color='grey'>Formatter</font> beans.
* Support for <font color='grey'>HttpMessageConverters</font>
* Automatic registration of <font color='grey'>MessageCodeResolver</font>
* static <font color='grey'>static.html</font> support
* Custom <font color='grey'>Favicon</font> support
* Automatic use of <font color='grey'> ConfigurableWebBindingInitializer</font> bean

if you want to keep Spring Boot MVC features and you want to add additional <font color='blue'>MVC configuration</font> (interceptors, formatters, view controllers and other features), you add your own <font color='grey'>@Configuration</font>class of type <font color='grey'>WebMvcConfigurer</font> but <strong>without</strong> <font color='grey'>@EnableMvc</font>.
If customization of <font color='grey'>RequestMappingHandlerMapping, RequestMappingHandlerAdapter</font>, or <font color='grey'>ExceptionHandlerExceptionResolver</font> needed, you can declare a <font color='grey'>WebMvcRegistrationsAdapter</font> instance to provide such components.
To take a complete control of Spring MVC, add your own <font color='grey'>@Configuration</font> annotated with <font color='grey'>@EnableMvc</font>

##### HttpMessageConverters
Spring MVC uses the <font color='grey'>HttpMessageConverter</font> interface to convert HTTP requests and responses. By default, strings are encoded in <font color='blue'>UTF-8</font>
``` Java
@Configuration
public class MyConfiguration {
  @Bean
  public HttpMessageConverters customConverters() {
    HttpMessageConverter<?> addition = ...
    HttpMessageConverter<?> another = ...
    return new HttpMessageConverters(addition, another);
  }
}
```
##### Custom JSON serializer and deserializer
Custom serializers are usually registered with Jackson through a module, but Spring Boot provides an alternative <font color='grey'>@JsonComponent</font> annotation that makes it easier to directly register Spring Beans.
``` Java
@JsonComponent
public class Example {
  public static class Serializer extends JacksonSerializer<SomeObject> {
    // code goes here
  }

  public static class Deserializer extends JacksonDeserializer<SomeObject> {
    // code goes here
  }
}
```
##### Static Content
By default, Spring Boot serves static content from a directory called <font color='grey'>/static</font>(or <font color='grey'>/public</font> or <font color='grey'>/resources</font> or <font color='grey'>/META-INF/resources</font>) in the classpath or from the root of the <font color='grey'>ServletContext</font>
By default, resources are mapped on <font color='grey'>/**</font>, but you can tune it with <font color='grey'>spring.mvc.static-path-pattern</font> property.

``` properties
spring.mvc.static-path-pattern=/resource/**
```
you can also customize the static resource locations by using the <font color='grey'>spring.resources.static-locations</font>property (replacing the default values with a list of directory locations)
``` Properties
spring.resources.static-locations=/static/**
```

In addition to the "standard" static resource locations mentioned earlier, a special case is made for <font color='blue'>Webjars content</font>.

To use version agnositic URLs for WebJars, add the <font color='grey'>webjars-locator-core</font> dependency.

``` html
<script src='/webjars/jquery/jquery.min.js'></script>
```

To use cache busting, the following configuration configures a cache busting solution for all static resources, effectively adding a content hash, such as <link href="/css/spring-2a2d59.css" />, in URLS:

``` properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.path=/**
```

### Appendix
#### Change from version 1.x to 2.x, using undertow will cause exception
> <strong>note</strong>
> 1. guarantee that undertow-core is introduced in classpath
> 2. do configure the server factory
> ``` Java
> @Bean
> public UndertowServletWebServerFactory serverFactory() {
>        return new UndertowServletWebServerFactory();
> }
> ```
