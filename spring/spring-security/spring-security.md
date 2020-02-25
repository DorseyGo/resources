# Spring Security

## 1. Architecture & Implementation

### 1.1 Technical Overview

#### 1.1.1 Core Components

**SecurityContextHolder, SecurityContext and Authentication Objects**

The most fundamental object is <u>SecurityContextHolder</u>. This is where we store details of the present security context of the application, which includes details of the principal currently using the application.

Three ways that SecurityContextHolder is using to store these details,

1. <font color='blue'>THREADLOCAL</font>: by default, using this in this way is quite safe if care is taken to clear the thread after the present principal's request is proceed.
2. <font color='blue'>GLOBAL</font>: for a standalone application
3. <font color='blue'>INHERITABLETHREADLOCAL</font>: might have threads spawned by the secure thread also assume the same security identity.

**Obtaining information about the current user**

Inside the <u>SecurityContextHolder</u> we store the details of the principal currently interact with the application. Spring Security uses the <u>Authentication</u> object to represent this information.

```java
Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
String username = null;
if (principal instanceof UserDetails) {
    username = ((UserDetails) principal).getUsername();
} else {
    username = principal.toString();
}
```

**The UserDetailsService**

There is a special interface called <u>UserDetailsService</u>. The only method on this interface accepts a <u>String</u>-based username argument and returns a <u>UserDetails</u>.

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException
```

On successful authentication, <u>UserDetails</u> is used to build the <u>Authentication</u> object that is stored in <u>SecurityContextHolder</u>.

> There is some confusion about <font color='red'>UserDetailsService</font>. It is purely a DAO for user data and performs *no further function* other than to supply that the data to other components within the framework. 
>
> ----
>
> In particular, it does not authenticate the user, which is done by the <font color='red'>**AuthenticationManager**</font>. In many cases, it makes more sense to <font color='blue'>implement</font> <font color='red'>**AuthenticationProvider**</font> directly if you require a custom authentication process.

**GrantedAuthority**

A <u>GrantedAuthority</u> is, not surprising, an authority that is granted to the principal. Such authorities are usually "roles", such as <u>ROLE_ADMINISTRATOR</u>. These roles are later on configured for web authorization, method authorization and domain object authorization. <u>GrantedAuthority</u> objects are usually loaded by <u>UserDetailsService</u>.

Usually <u>GrantedAuthority</u> objects are <font color='red'>**application-wide**</font> permissions. They are **NOT** specific to a given domain object (eg. not represent the permission to <font color='#AA0000'>Employee</font> object number 54 ). Spring Security is expressly designed to handle this common requirement, but you'd instead use the <font color='red'>**project's domain object security capabilities**</font> for this purpose.

#### 1.1.2 Authentication

**What is authentication in Spring Security**

Let's consider a standard authentication scenario,

1. A user is prompted to login with username and password
2. The system (successfully) verifies that the password is correct for the username
3. The context information for that user is obtained (roles and so on)
4. A security context is established
5. The user proceeds, potentially to perform some operation which is potentially protected by an access control mechanism which checks the required permissions for the operation against the current security context information

the authentication proceeds in following way,

1. the username and password are obtained and combined into an instance of <font color='#AA0000'>UsernameAndPasswordAuthenticationToken</font>.
2. the token is passed to an instance of <font color='#AA0000'>AuthenticationManager</font> for validation
3. the <font color='#AA0000'>AuthenticationManager</font> returns a fully populated <font color='#AA0000'>Authentication</font> instance on successful authentication
4. the security context is established by calling <font color='#AA0000'>SecurityContextHolder.getContext().setAuthentication(...)</font>, passing in the returned authentication object.

#### 1.1.3 Authentication in a Web Application

<font color='#AA0000'>ExceptionTranslationFilter</font> and <font color='#AA0000'>AuthenticationEntryPoint</font> and an "authentication mechanism" are responsible for calling the <font color='#AA0000'>**AuthenticationManager**</font>.

**ExceptionTranslationFilter**

<font color='#AA000'>ExceptionTranslationFilter</font> is a Spring Security filter that has responsibility for detecting any Spring Security exceptions that are thrown. Such exceptions will generally be thrown by an <font color='#AA0000'>AbstractSecurityInterceptor</font>, which is the main provider of authorization services.

#### 1.1.4 Access-Control (Authorization) in Spring Security

The main interface responsible for making accessing-control decisions in Spring Security is the <font color='#AA0000'>AccessDecisionManager</font>. It has a <font color='#AA0000'>decide</font> method which takes an <font color='#AA0000'>Authentication</font> object representing the principal requesting access.

**Configuration Attributes**

A "Configuration Attribute" can be thought of as a String that has special meaning to the classes used by <u><font color='#AA0000'>AbstractSecurityInterceptor</font></u>. Configuration attributes will be entered as annotations on secured methods or as access attributes on secured URLs. Strictly speaking though, they are just attributes and intepretation is dependent on the <u><font color='#AA0000'>AccessDecisionManager</font></u> implementation.

**RunAsManager**

Assuming <u><font color='#AA0000'>AccessDecisionManager</font></u> decides to allow the request, the  <u><font color='#AA0000'>AbstractSecurityInterceptor</font></u> will normally just proceed the request. On rare occasions users may want to **replace** the <font color='#AA0000'>Authentication</font> inside the <font color='#AA0000'>SecurityContext</font> with a different <font color='#AA0000'>Authentication</font>, which is handled by <u><font color='#AA0000'>RunAsManager</font></u>. 

### 1.2 Core Services

#### 1.2.1 AuthenticationManager

The default implementation of <u>AuthenticationManager</u> in Spring Security is called <u><font color='#AA0000'>ProviderManager</font></u> and rather than handling the authentication request itself, it delegates to a list of configured <u><font color='#AA0000'>AuthenticationProvider</font></u>s, each of which is queried in turn to see if it can perform the authentication. 

**Erasing Credentials on Successful Authentication**

By default, the <u><font color='#AA0000'>ProviderManager</font></u> will attempt to clear any sensitive credentials information from the <font color='#AA0000'>Authentication</font> object which is returned by a successful authentication request. This prevents information like passwords being retained longer than necessary.

If you don't want to clear any sensitive credentials information, two solutions are provided obviously,

1. make a copy of the object first, either in the cache implementation or in the <u><font color='#AA0000'>AuthentitionProvider</font></u> which creates the returned <u>Authentication</u> object.
2. disable the <u>eraseCredentialsAfterAuthentication</u> property on <u><font color='#AA0000'>ProviderManager</font></u>

### 2. Authentication

#### 2.1 Password Encoding

Spring Security's <u><font color='#AA0000'>PasswordEncoder</font></u> interface is used to perform a one way transformation of a password to allow the password to be stored securely. Typically, <u><font color='#AA0000'>PasswordEncoder</font></u> is used for storing a password that needs to be compared to a user provided password at the time of authentication.

#### 2.2 DelegatingPasswordEncoder

Three real world problems,

- there are many applications using old password encodings that cannot easily migrate
- the best practise for password storage will change again
- As a framework Spring Security cannot make breaking changes frequently

under this circumstance, <u><font color='#AA0000'>DelegatingPasswordEncoder</font></u> is introduced to fix these problems.

```java
String idForEncode = "bcrypt";
Map<String, PasswordEncoder> encoders = Maps.newHashMap();
encoders.put("noop", NoOpPasswordEncoder.getInstance());
encoders.put("pbkdf2", new PbkdfPasswordEncoder());
encoders.put("scrypt", new SCryptPasswordEncoder());
encoders.put("sha256", new StandardPasswordEncoder());

PasswordEncoder encoder = new DelegatingPasswordEncoder(idForEncode, encoders);
```

**Password Encoding**

The <u><font color='#AA0000'>idForEncode</font></u> passed into the constructor determins which <u><font color='#AA0000'>PasswordEncoder</font></u> will be used for encoding password. 

**Password Matching**

Matching is done based upon the <u><font color='#AAAA00'>{id}</font></u> to the <u><font color='#AA0000'>PasswordEncoder</font></u> provided in the constructor. By default, the result of invoking <u><font color='#AAAA00'>matches(CharSequence, String)</font></u> with a password and an <u><font color='#AAAA00'>id</font></u> that is not mapped will result in <u><font color='#AAAA00'>IllegalArgumentException</font></u>.

#### 2.3 Session Management

HTTP session related functionality is handled by a combination of the <u><font color='#AA0000'>SessionManagementFilter</font></u> and the <u><font color='#AA0000'>SessionAuthenticationStrategy</font></u> interface, which the filter delegates to. Typical usage includes session-fixation protection attack prevention, detection of session timeouts and restrictions on how many sessions an authenticated user may have open concurrently.

##### 2.3.1 Concurrent Session Control

If you wish to place constraints on a single user's ability to login to your application, Spring Security supports this out of box with the following simple additions. First, in <u><font color='#AAAA00'>web.xml</font></u>, 

``` xml
<listener>
	<listener-class>
org.springframework.security.web.session.HttpSessionEventPublisher
	</listener-class>
</listener>
```

then add followings

``` XML
<http>
	<session-management>
    	<concurrency-control max-sessions="1" />
    </session-management>
</http>
```

This will prevent a user from logging in multiple times - a second login will cause the first to be invalidated.

##### 2.3.2 Session Fixation Attack Protection

Session Fixation attacks are a potential risk where it is possible for a malicious attacker to create a session by accessing a site, then persuade another user to login with the same session. There are four options for <u><font color='#AAAA00'>session-fixation-protection</font></u>,

