# Spring Cloud

## Cloud Native Applications

<u>Cloud Native</u> is a style of application development that encourages easy adoption of best practices in the areas of continuous <font color='red'>delivery</font> and <font color='red'>value-driven</font> development. 

Two extra libraries are provided by <u>Spring Cloud</u>, which is built upon <font color='light blue'> Spring Boot</font> 

- Spring Cloud context: provides utilities and special services for the <font color='red'>ApplicationContext</font> of a Spring Cloud Application (bootstrap context, encryption, refresh scope, and environment endpoints)
- Spring Cloud Commons: a set of abstractions and common classes used in different Spring Cloud implementations

### Spring Cloud Context: Application Context Services

#### The Bootstrap Application Context

A Spring Cloud application operates by creating a "bootstrap" context, which is a parent context for the main application. This context is responsible for loading configuration properties from the external sources and for decrypting properties in the local external configuration files.

By default, bootstrap properties (properties that loaded during the bootstrap phase) are added with high precedence, so they **CANNOT** be *overridden* by local configuration.

Bootstrap phase can be completely disabled by setting <font color='red'>spring.cloud.bootstrap.enabled</font> (in system properties) like following,

``` properties
spring.cloud.bootstrap.enabled = false
```

