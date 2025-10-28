# Spring Boot
## Feature introduction
- The "Opinionated" view of spring platform and third party libraries.
  - But still customizable.
- handles low-level, predictable set-up for us.
- Provides a range of non-functional features
  - Embedded servers (App server)
  - metrics (actuator)
  - health check
  - externalized configuration
  - containerization
- Rescue us from dependency hell (compatibility problems)
  - Parent or Starters
  - Fine-grained dependency management is still possible
    - Can exclude dependencies
    - Explicitly define dependencies
- Spring boot parent POM
  - Defines versions of key dependencies
    - Uses a `dependencyManagement` section internally
    - Through `spring-boot-dependencies` as parent
  - Defines maven plugins
  - Sets up Java version
- `spring-boot-starter` pulls ~18 JARs!
  ![img](imgs/starter_dependencies.png)
- Many starters are available
  - spring-boot-starter-jdbc
  - spring-boot-starter-data-jpa
  - spring-boot-starter-web
  - spring-boot-starter-batch
  - [etc...](https://docs.spring.io/spring-boot/reference/using/build-systems.html#using.build-systems.starters)
- Auto-configuration enabled by `@EnableAutoConfiguration` on a config class.
  - Spring Boot automatically creates beans it thinks you need based on some conditions!
  - Examples of it
  ![img](imgs/auto_config_example.png)
- Very common to use `@SpringBootConfiguration`, `@ComponentScan` & `@EnableAutoConfiguration` together.
  - `@SpringBootConfiguration` simply extends `Configuration`.
  - `SpringBootApplication` combines all three.
![img](imgs/spring_boot_application_annotation.png)