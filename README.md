# spring-security-csrf-android-demo

This project contains demo codes on how to communicate an android project with a spring boot application that has spring security and CSRF enabled.

# Features

* Simple spring boot web application with spring security and CSRF enabled
* Java client to authenticate and communicate with the spring boot web application
* Android client to authenticate and communicate with the spring boot web application

# Spring Security

### spring-boot-application with spring security and CSRF enabled

To use this project create a database named spring_boot_slingshot in your mysql database (make sure it is running at localhost:3306)

```sql
CREATE DATABASE spring_boot_slingshot CHARACTER SET utf8 COLLATE utf8_unicode_ci;
```

Note that the default username and password for the mysql is configured to 

* username: root
* password: chen0469

If your mysql or mariadb does not use these configuration, please change the settings in src/resources/config/application-default.properties

For the spring security configuration, the CSRF is enabled. The configuration in the spring-boot-application as follows:

```java
http
.authorizeRequests()
.antMatchers("/js/client/**").hasAnyRole("USER", "ADMIN")
.antMatchers("/js/admin/**").hasAnyRole("ADMIN")
.antMatchers("/admin/**").hasAnyRole("ADMIN")
.antMatchers("/erp/login-api-json").permitAll()
.antMatchers("/html/**").hasAnyRole("USER", "ADMIN")
.antMatchers("/js/commons/**").permitAll()
.antMatchers("/css/**").permitAll()
.antMatchers("/jslib/**").permitAll()
.antMatchers("/webjars/**").permitAll()
.antMatchers("/bundle/**").permitAll()
.antMatchers("/locales").permitAll()
.antMatchers("/locales/**").permitAll()
.anyRequest().authenticated()
.and()
.formLogin()
.loginPage("/login")
.defaultSuccessUrl("/home")
.successHandler(authenticationSuccessHandler)
.permitAll()
.and()
.logout()
.permitAll()
.and()
.csrf()
.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
```

which can be found in the com.github.chen0040.bootslingshot.configs.WebSecurityConfig

### principle behind restful login to spring-boot-application

The web login api can be found in the com.github.chen0040.bootslingshot.controllers.WebApiController. which consists
 of GET and POST api for the same url "/erp/login-api-json".
 
Any client which wants to authenticate with the spring security in spring-boot-application can first call

GET: http://localhost:8080/erp/login-api-json

this will return a json object containing a valid csrf token YOUR_CSRF_TOKEN, the same client can then post to the same url:

POST: http://localhost:8080/erp/login-api-json

with the following headers:

* _csrf: YOUR_CSRF_TOKEN
* Cookie: XSRF-TOKEN=YOUR_CSRF_TOKEN
* X-XSRF-TOKEN: YOUR_CSRF_TOKEN 

If login is successful, you can find the response json object has authenticated set to true.
By examining the Set-Cookie header of the POST response, you should be able to extract the JSESSIONID=YOUR_SESSION_ID.

Now after login is successful, you can access the spring security protected api by adding the following in the header:

* _csrf: YOUR_CSRF_TOKEN
* Cookie: XSRF-TOKEN=YOUR_CSRF_TOKEN;JSESSIONID=YOUR_SESSION_ID
* X-XSRF-TOKEN: YOUR_CSRF_TOKEN 


# Usage

### Spring Server

Run the "./make.ps1" (windows environment) and "./make.sh" (unix environment). which will compile and stores the built
jars in the "bin" folder.

* spring-boot-application: the spring boot application that has csrf-enabled spring security configuration

```bash
java -jar bin/spring-boot-application.jar
```

This will start the spring-boot-application that is at http://localhost:8080

The application can be authenticated using any one of the accounts below:

ADMIN:

* username: admin
* password: admin

DEMO:

* username: demo
* password: demo

In the following instructions, http://localhost:8080/users/get-account is an url that requires authentication.

### Java Client

The following are the excerpt from spring-boot-java-client unit test to show how to login to the spring-boot-application:

```java
SpringBootClient client = new SpringBootClient();
SpringIdentity identity = client.login("http://localhost:8080/erp/login-api-json", "admin", "admin");

System.out.println(JSON.toJSONString(identity, SerializerFeature.PrettyFormat));
System.out.println(client.getSecured("http://localhost:8080/users/get-account"));
```

### Android Client

The following are the excerpt from spring-boot-android-client unit test to show how to login to the spring-boot-application:

```bash
SpringBootClient client = new SpringBootClient();

SpringIdentity identity = client.login("http://localhost:8080/erp/login-api-json", "admin", "admin");

System.out.println(JSON.toJSONString(identity, SerializerFeature.PrettyFormat));
System.out.println(client.getSecured("http://localhost:8080/users/get-account"));
```













