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

### 2.4 Profiles

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

## 3. Web development

### 3.1 MVC

### 3.2 Embedded Container

## 4. Working with Data

### 4.1 SQL

### 4.2 NoSQL

## 5. Messaging

### 5.1 JMS

## 6. Testing

## 7. Extending

## 8. Monitoring

## 9. Management endpoints

## 10. Connection options