# Spring Framework Essentials
- (Docs)[https://docs.spring.io/spring-framework/reference/]

## Service locator pattern vs. IoC container
- Service Locator (SL)
  - Central registry of services.
  - Clients pull dependencies by calling the locator (e.g., ServiceLocator.getService("foo")).
  - Dependencies are hidden in code → harder to see and test.
  - Creates coupling to the locator.
  - Considered an anti-pattern in modern design if overused.

- Inversion of Control (IoC) / Dependency Injection (DI)
  - Container holds a registry of services.
  - Dependencies are pushed into classes automatically (via constructor, setter, or field).
  - Dependencies are explicit → easy to read, maintain, and test.
  - Decouples classes from the container.
  - Preferred in modern frameworks (e.g., Spring, Guice).

- Key Difference
  - SL: “Class asks for what it needs.”
  - IoC/DI: “Container provides what the class needs.”

---

- DI is useful when testing (mocking dependencies)
- First release of spring came out at 2003, on 2004 interface 21 company was the developer.
- 2009 it gor aquired by VMware.
- POJO: A simple Java object that is not bound by any special restriction or framework. It typically just contains fields, constructors, and methods.
- **JavaBean:** A special type of POJO that follows specific conventions:
  1. Must have a no-argument constructor
  2. Private fields
  3. Public getters and setters
  4. Should be serializable

## Spring configuration class [M2]
- Configuration classes is annotated with `@Configuration`
- Configuration `@Bean`s would be eagerly loaded in the correct dependency order.
- By default, the name of the method is the name of the bean. (can be overwritten in the @Bean(name=) annotation call)
- Later the Bean is accessible using `ApplicationContext::getBean()` in 3 forms.
  ![img](imgs/retreving_a_bean.png)
- The Application context is the dependency injection container.
- The application context can be created in any environment. (e.g., Standalone app, web app, JUnit test, etc.)
  - Using `SpringApplication.run(ApplicationConfig.class): ApplicationContext`
- Other configuration classes can be imported to the root config class with `@Import`.
  ![img](imgs/multiple_config_classes.png)
- Dependencies can also be injected in config classes constructors using `@Autowired`.
  ![img](imgs/autowired_config_constructor.png)
- Avoid tramp data:
  ![img](imgs/tramp_data.png)

### Common Bean Scopes
  - Singleton (default): so they should be thread safe. either by:
    - Use Stateless or Immutable beans.
    - Use synchronized (harder)
  - Prototype: A new instance is created every time bean is resolved.
  - Session (Web only): A new instance is created once per user session.
  - Request (Web only): A new instance is created once per request.
  ```java
  @Bean
  @Scope("singleton") // Redundant, since it's the default
  // Same as: @Scope(scopeName="singleton")
  public Action deviceAction() {
    //
  }
  ```
  - Web Socket scope (Spring 4)
  - Refresh scope (Spring cloud)
  - Thread scope (Defined but not registered by default)
  - Custom scopes (rarely)
    - You define a factory for creating bean instances
    - Register to define a custom scope name

### DI Summary
- The code is handed with what is needs to work
- Promotes programming to interface
- Improves testability.
- Allows centralized control over object lifecycle.

## More on Configuration [M3]
### Use External Properties to control Configuration [M3E1]
- `Environment` bean represents loaded properties from runtime environment.
- Properties derived from various sources, in this order:
  - JVM System Properties - `System.getProperty()` (Automatically populated)
  - System environment variables - `System.getenv()` (Automatically populated)
  - Java Properties Files (Manually populated)
- `@PropertySource("path_to_tile.properties")` contributes additional properties.
  - Available resource prefixes: `classpath:`, `file:`, `http:`
  ![img](imgs/property_source.png)
- To retrieve properties either:
  - use `Environment::getProperty(String key)`
  - or use `@Value("${ key }")` as argument annotation.
  ![img](imgs/@value.png)

### Profiles [M3E2]

![img](imgs/profiles.png)

- Beans can be grouped into profiles.
- Profiles can represent:
  - Environments: dev, test, production
  - implementation: jpa, jdbc
  - deployment platform: on-premise, cloud
- Beans included/excluded based on profile membership.
- Defining profiles
  - `@Profile("name")` annotation, at configuration class or method level.
  - `@Profile("!name")` annotation is used to mark all other profiles but `name`.
- Ways to activate profiles (must be activated at run-time)
  - System properties via command-line: `-Dspring.profiles.active=embedded,jpa`
  - System property programmatically:
    ```java
    System.setProperty("spring.profiles.active", "embedded,jpa");
    SpringApplication.run(AppConfig.class); // Before this line!
    ```
  - Integration test only: `@ActiveProfiles` annotation
- `@Profile` annotation can control which `@PropertySource` are included in the environment if mentioned right after.
![img](imgs/profiles_and_properysource.png)

### Spring Expression Language (SpEL) [M3E3]
- Inspired by expression language used in Spring WebFlow
- Based on Unified Expression Language used by JSP and JSF.
- Pluggable/Expandable by other Spring-based frameworks (Spring Security or others).
- `@Value("#{ systemProperties['user.region'] }")`: `#{}` is used instead of `${}`
  ![img](imgs/spel1.png)
- Can be used to access spring beans
  ![img](imgs/spel2.png)
- Can access properties via the environment:
```java
@Value("${daily.limit}")
int maxTransfersPerDay; // Auto conversion, string -> int
// These are equivalent
@Value("#{environment['daily.limit']}")
int maxTransfersPerDay; // Auto conversion, string -> int
```
- But:
```java
@Value("#{new Integer(environment['daily.limit']) * 2}") // OK
@Value("#{new java.net.URL(environment['home.page']).host}") // OK
@Value("${daily.limit * 2}") // NOT OK!
```
- Fallback values
  - `@Value("${daily.limit : 10000})`
  - `@Value("#{environment['daily.limit'] ?: 10000}")` -> `?:` is Elvis operator.

#### SpEL Summary
- EL attributes can be:
  - Spring beans
  - Implicit references
    - Spring's `environment`, `systemProperties`, `systemEnvironment` available by default.
    - Others depending on context. (Spring Security, Spring WebFlow, Spring Batch, Spring Integration)
- SpEL allows to create custom functions and references

## Component Scanning [M4]
### Annotation-based configuration (Implicit Configuration)
- So far we only had explicit configuration (Configuration class and @Bean)
- Annotating a POJO with `@Component` marks it for being bind to the IoC container.
- In this case Bean id/name is derived from the classname (camelCased).
- Then, `@ComponentScan("package.address")` annotation is used in a configuration class to find and bind `@Component`s.
![img](imgs/component_scan.png)

#### Usage of `@Autowired`
- Constructor-injection: (Recommended practice): Optional if only one constructor exists
- Method-injection
- Field-injection: Even when the field is private, it works. But [it's not recommended](https://odrotbohm.de/2013/11/why-field-injection-is-evil/).

⚠️ In all cases: A unique bean of the required type must exist!
- By default if no such bean exists in the container it will result in exception at run-time.
- To prevent such exception we can use `@Autowired(required=false)` to mark it as optional.
- Another way to mark optional dependencies is to use JDK8 `Optional<T>` like so:
![img](imgs/optional_dependency.png)

#### Constructor vs. Setter dependency injection
| Constructors                          | Setters                               |
|---------------------------------------|---------------------------------------|
| Mandatory dependencies                | Circular dependencies possible        |
| Dependencies can be immutable         | Dependencies are mutable              |
| Concise (pass several params at once) | Could be verbose for several params   |
|                                       | Inherited automatically               |
- Generally constructor injection is preferred.
- Consistency in the project team is important.

#### Autowiring and Disambiguation
- using `@Qualifier("bean name")`.
![img](imgs/qualifier.png)
- Variable name can be used to resolve the correct bean!
- Autowired resolution rules
  1. Look for unique bean of required type
  2. (If found multiple) Use @Qualifier if supplied
  3. (If not found) Try to find matching bean by variable name!
- Common strategy: avoid using @Qualifier when possible. Usually rare to have 2 beans of the same type in ApplicationContext.

#### Delayed bean initialization 
- Beans are normally created on startup when the application context is created. (Because we want to fail fast)
- `@Lazy` beans created first time they get used (Careful! often misused)
  - When required for dependency injection
  - By `ApplicaitonContext::getBean()`
- Useful if bean's dependencies not available at startup.

#### Annotations syntax vs. Java Config
![img](imgs/annotation_vs_java_config.png)
- For the classes we do not own (infrastructure) we only can use Java Config.

#### Best practices [M4E2]
* Spring by default prioritizes no-arg constructor.
- Component Scanning
  - Really bad: `@ComponentScan("com", "org")`
  - Still bad: `@ComponentScan("com")`
  - OK: `@ComponnetScan("com.bank.app")`
  - Optimized: `@ComponentScan({"com.bank.app.repository", "com.bank.app.service", "com.bank.app.controller"})`

### Life cycle annotations  and `@PreDestroy` [M4E3]
- `@PostConstruct`: is called at startup after all dependencies are initialized.
- `@PreDestroy`: called at shutdown prior to destroying the bean instance.
  - Only is called when the application shuts down normally, not if the process dies of is killed.
  - Useful for releasing resources & cleaning up.
  - Not called for prototype beans.
  - Runs on `ConfigurableApplicationContext::close()"`
  - Spring app on startup will automatically register a JVM shutdown hook (`System.exit()`/`Runtime.exit()`).
- These methods can have **any visibility** but must take **no parameters** and only **return void**.
- These are not Spring annotations!
  - Defined by JSR-250, part of JDK6.
  - Located at `javax.annotation` package.
  - Supported by Spring, and Java EE (Jakarta E).
- Alternatively: `@Bean(initMethod="populateCache", destroyMethod="flushCache)`

### Stereotype annotations [M4E4]
- Annotations that are themselves annotated with `@Component`.
- These annotations are also picked up when ComponentScanning.
- `@Component` is a meta-annotation. it can be used to annotate other annotations.

![img](imgs/stereotypes.png)

## Inside the Spring container [M5]
* The content of this chapter is a simplified view of spring's inner workings.
- Spring bean lifecycle:
  1. Initialization
     - A. Load & Process Bean Definitions
       - The `@Configuration` classes, xml files, and `@Component` classes are scanned.
       - Bean Definitions added to **BeanFactory** each indexed under its id and type.
       - Special **BeanFactoryPostProcessor** beans invoked, which can modify the definition of any bean.
     - B. Perform Bean Creation
  2. Usage
     - Beans are available for use in the application.
  3. Destruction (Application context closed)
     - Beans are released for Garbage Collection.

---
### 1. Initialization phase

![img](imgs/bean_initialization_steps.png)

- `ApplicationContext` is a `BeanFactory`.

- `BeanFactoryPostProcessor`
  - Is a functional interface.
    ```java
    public interface BeanFactoryPostProcessor {
        void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory);
    }
    ```
  - Is a Spring extension point
  - Allows you to modify bean definitions before any beans are actually instantiated.
  - `PropertySourcesPlaceholderConfigurer` (Spring 4.3+) is an example of it that resolves `@Value("${}")` placeholder values.
  - It needs to run before any beans are created so use of **static** `@Bean` method is recommended.
  - It's an internal bean invoked by spring (not your code).

- The Initializer extension point
  - Special case of a bean post-processing
  - Causes initialization methods to be called (`@PostConstruct`, `init-method`, etc.)
  - Internally Spring uses several initializer BPPs. (`CommonAnnotationBeanPostProcessor` enables JSR-250 annotations like `@PostConstruct`, `@Resource`, etc.)

- `BeanPostProcessor` Extension point
  - Can modify bean **instances** in any way.
  - Will run against every bean.
  - Can modify a bean before and/or after Initialization.
  ```java
  public interface BeanPostProcessor {
      Object postProcessBeforeInitialization(Object bean, String beanName);
      Object postProcessAfterInitialization(Object bean, String beanName);
      // Object in method args -> Original bean
      // Object in method return -> Post-processed bean
      // If the bean is not retuned, it will be lost!
  }
  ```
  - The beanPostProcessor classes can be a normal bean in spring, annotated with `@Component` or `@Bean` in config class.
  ![img](imgs/bean_configuration_lifecycle.png)


### 2. Usage phase [M5E3]
- When retrieving a bean, two cases may occur:
  1. The retrieved bean is just a bean: Nothing new!
  2. The retrieved bean is a **Proxy** to the original bean!
- The proxy is created during initialization phase by a `BeanPostProcessor`.
- The BPP wraps your bean in a "proxy" adding behavior to it transparently.
- Example: Transactions
  ![img](imgs/bean_proxy_transactions.png)

#### Kinds of proxies
- **JDK Proxy**
  - Also called dynamic proxies
  - API is built into the JDK
  - Requirements: Java Interface(s)
  - All interfaces proxied
- **CGLib Proxy**
  - Not built into JDK
  - Included in Spring jars
  - Used when interface is not available
  - Cannot be applied to final classes or methods
  - Spring Boot will always go with CGLib Proxies. (JDK Proxy can be used in vanilla Spring)
  ![img](imgs/jdk_vs_cglib_proxy.png)

### Destruction phase
- All beans are cleaned up
  - Any registered `@PreDestroy` methods are invoked.
  - Beans released for the Garbage Collector to destroy.
- Happens when we close the context. (`ConfigurableApplicationContext::close()`)
- Also happens when any bean goes out of scope (except Prototype scoped beans).
- Only happens when app is shutdown gracefully, not if killed or crashed.

### More on Bean Creation [M5E4]
- Find/create the bean dependencies is a **recursive** process.
- You can force dependency order by `@DependsOn("beanName")` both with annotation-based config method and java config.
```java
@Component
@DependsOn("accountService")
class TransferService {
  // ...
}
```
- In this example `TransferService` will be created right after `AccountService`.

#### Determining Bean Name & Type

| Definition                     | Name                                                               | Type                    |
|--------------------------------|--------------------------------------------------------------------|-------------------------|
| **1. Java config**             | From `@Bean` name/value attribute<br>Or from method name           | From method return type |
| **2. Annotation-based Config** | From annotation value attribute<br>Or derived from class name      | From annotated class    |

- **1** typically returns an *interface*
- **2** provides *actual* implementation class

#### Why won't this work?

![img](imgs/wtww_1.png)

- **Solution 1:** Use `BankTransferServiceImpl` as return type instead of `TransferService` in the second `@Bean`; giving more info to the container.
- **Solution 2:** Create a composite interface and use it as return type of second `@Bean` like so:
![img](imgs/wtww_2.png)
- Can determine `BankTransferService` extends both `TransferService` and `BankService`.

#### Best practices of defining Spring Beans
  - Aim to be "sufficiently expressive"
    - Return interfaces except where multiple interfaces exists and they are needed for dependency injection.
    - Coding to interface is good practice.
  - **Warning:** Even if you return implementation types:
    - Still use interfaces when injecting dependencies.
    - Injecting implementation types is brittle: may fail if the bean is proxied or a different implementation returned.

## Introducing Aspect Oriented Programming (AOP) [M6]