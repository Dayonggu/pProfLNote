# Spring boot
* following
  * https://www.tutorialspoint.com/spring_boot/spring_boot_bootstrapping.htm
    * tbc: batch process 
## Quick start
* to built a `springboot` based microservice, you need start with *main* application classes following annotations
  * `@SpringBootApplication` or
  * `@EnableAutoConfiguration, @ComponentScan and @SpringBootConfiguration`: MD used the Scone annotation which use the first two from here
* in `pom.xml`, you need specify the *start class*
  * `<start-class>com.YourComp.YourApplication</start-class>`
  * it maybe encapsulated by implementation your application based on though  * like *SCONE*
  * eventually, the main class is invoked or passed to a `SpringApplication` app, and then call `app.run()`
  * which would read the `application.property`
* Way of packaging
  * `<packaging>war</packaging>` or `jar`

## Dependency injection, application property  and auto-config
* The `@ComponentScan` annotation is used to find beans and the corresponding injected with `@Autowired` annotation
* Configuration
  * `application.property` defined the config, and you could have different version of it for different profile, eg
    * `application-dev.properties`
    * `application-prod.properties`
    * when run, you could choose one by  `--spring.profiles.active=dev`
  * `@Value` annotation would give you change to read the value of a property
    * `@Value("${spring.application.conf.item}")`
    * `spring.application.conf.item` would be the name of one config line in application.property

## WebService concepts
* To build a RESTful webservice from `Springboot`
* You could have `<artifactId>spring-boot-starter-web</artifactId>` in pom.xml (or from the implementation you based on)
* *Controller*
  * `@RestController` annotation is used to define the RESTful web services.
  * in the class, you could have `@RequestMapping(value = "/products")` to define the code for given Endpoint
    * eg: `@RequestMapping(value = "/products/{id}", method = RequestMethod.PUT)`
  * other related annotations:
    * @RequestBody, @RequestParam
* Exception handling
  * handle globally: `@ControllerAdvice` annotation to the handler class, or
  * make specific handler for a given type of exception
    * `@ExceptionHandler(value = ProductNotfoundException.class)`
* *Interceptor*
  * work as a kind of plugin/hook/modifier in the request process life cycle :
    * Before sending the request to the controller
    * Before sending the response to the client
    * Methods: preHandle() postHandle() afterCompletion()
  * need annotation `@Component class`
  * Register to Interceptor *AppConfig*
* *filter* : works like the *Interceptor*: `public void doFilter(req, resp, FilterChain)`
* *Service Components* : annotation `@Service`:
  * are used to write business logic in a different layer, separated from `@RestController` class file.
  * write detailed implementations, would be called with in the *Controller* class (avoid making the controller a giant one, better abstraction)
* *Cross-Origin Resource Sharing* (CORS)
  * like a 8080 service would like to access resource in 9090 port
  * `@CrossOrigin(origins = "http://localhost:9090")` in controller method
  * or in a configuration `@Bean` explicitly do for global : `registry.addMapping("/products").allowedOrigins("http://localhost:9090");`
* *Service Registration*
  * *Eureka Server*:
    * Eureka Server is an application that holds the information about all client-service applications.
    * Every Micro service will register into the Eureka server and Eureka server knows all the client applications running on each port and IP address.
    * Eureka Server is also known as *Discovery Server*.
    * comes with the bundle of *Spring Cloud*
    * `@EnableEurekaServer  public class EurekaserverApplication`
* *Zuul Server* : Or *Edge Server*
  * a gateway application that handles all the requests and does the *dynamic routing* of microservice applications
  * For Example, /api/user is mapped to the user service and /api/products is mapped to the product service
  * `@EnableZuulProxy` : add it to main springboot application , make your Spring Boot application act as a Zuul Proxy server.
* *Config server*
  * `@EnableConfigServer` makes your Spring Boot application act as a Configuration Server.
  * add the below configuration to your properties file and replace the application.properties file into bootstrap.properties file.
  ~~~
  server.port = 8888
spring.cloud.config.server.native.searchLocations=file:///C:/configprop/
SPRING_PROFILES_ACTIVE=native
  ~~~
  * Configuration Server runs on the Tomcat port *8888* and application configuration properties are loaded from native search locations.
  * Now, in file:///C:/configprop/, place your client application - *application.properties file*. For example, your client application name is *config-client*, then rename your *application.properties* file as *config-client.properties* and place the properties file on the path file:///C:/configprop/.Actuator
* *Actuator* : provides *secured endpoints* for *monitoring and managing* your Spring Boot application
  * such as extend the `HealthIndicator` in `org.springframework.boot.actuate.health`
* *Admin Server* : `<artifactId>spring-boot-admin-server</artifactId>`
  * provide Admin UI to manage each microservices `@EnableAdminServer`
  * *Admin client* : `<artifactId>spring-boot-admin-starter-client</artifactId>`
* *Batch process*
  * `@EnableBatchProcessing`

## scheduling
* `@EnableScheduling` for application
* `@Scheduled(cron = "0 * 9 * * ?")`  over a  cron job function
  * fixed delay: `@Scheduled(fixedDelay = 1000, initialDelay = 1000)`
  * some other flexibiliteis provided  

## Related system
* *Docker* file :
  * create a file with the name Dockerfile under the directories src/main/docker
  * in pom add dependency: `<artifactId>docker-maven-plugin</artifactId>`
  * `mvn package docker:build`
* *Swagger* : generate the REST API documents for RESTful web services
  * `<artifactId>springfox-swagger2</artifactId>`
  * `@EnableSwagger2` and  create Docket Bean to configure Swagger2 for your Spring Boot application.
  * you will get *swagger* document: `http://localhost:8080/swagger-ui.html`
* *Zipkin* tracing :
  * Spring cloud *Sleuth* logs , in format of `[application-name,traceid,spanid,zipkin-export]`
    * `Spanid` = Span Id is printed along with Trace Id. Span Id is different every request and response calling one service to another service.
    * if `zipkin-export` is `true`, log would be export to a *Zipkin server*
    * *Zipkin server*
      * add dep: `<artifactId>zipkin-server</artifactId>`
      * in its property file: `server.port = 9411`
      * Add the `@EnableZipkinServer` annotation in your main Spring Boot application class
      * Then you got Zipkin service at: `http://localhost:9411/zipkin/`
        * finer: `http://localhost:9411/zipkin/traces/{traceid}/`
* *Flyway* DB : `flyway-core`
  *  evolve your Database schema easily and reliably
  * Add properties:
  ~~~
  flyway.url = jdbc:mysql://localhost:3306/mysql
flyway.schemas = USERSERVICE
flyway.user = root
flyway.password = root
  ~~~
  * create  SQL file(s) under the *src/main/resources/db/migration* directory
* *Hystrix*: `<artifactId>spring-cloud-starter-hystrix</artifactId>`
  * *isolates* the points of access between the services,
  * stops *cascading failures* across them and
  * provides the fallback options.
  * add `@EnableHystrix` to you main spring boot application
  * command and properties:
  ~~~
  @HystrixCommand(fallbackMethod = "fallback_hello",
   commandProperties = {
   @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1000")
})
  ~~~


* End   
## Appendix
### Terms
* `graddle`: a new generation of *building* tool, (like `ant`, `maven`)
  * use code like  build.gradle file, instead of xml configurations
* `POJO` : plain Old Jave Object
  * Define purely *api/lib/framework independent* *data* for your service
  * not extend/impl from any given class/interface (not including serializable)
  * Access field by getter/setter, with default ctor
