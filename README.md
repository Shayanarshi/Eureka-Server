# Eureka Server – Spring Boot Microservices

## 1. Project Overview

Eureka Server is a service registry used in a Spring Boot microservices architecture. It allows microservices to register themselves and discover other services without hardcoding their hostnames and ports.

This project provides service discovery for:

- **CustomerApp** – Customer management and profile APIs
- **FriendApp** – Friend/contact management APIs
- **PlanApp** – Plan management APIs

## 2. Architecture

```text
             ┌─────────────────────┐
             │    Eureka Server    │
             │   Port: 8761        │
             └──────────┬──────────┘
                        │
               Service Registration
                        │
          ┌─────────────┼─────────────┐
          │             │             │
          ▼             ▼             ▼
    CustomerApp     FriendApp      PlanApp
     Port 8080       Port 8081      Port 8082
          │             ▲             ▲
          └─────────────┴─────────────┘
             Service-to-Service Calls
```

*Ports are examples; use the ports configured in your applications.*

## 3. Technologies Used

- Java 21
- Spring Boot
- Spring Cloud Netflix Eureka
- Maven
- Spring Cloud dependency management

## 4. Maven Dependency

Add the Eureka Server dependency to the Eureka Server project's `pom.xml`:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

Configure Spring Cloud dependency management:

```xml
<properties>
    <java.version>21</java.version>
    <spring-cloud.version>2025.1.4</spring-cloud.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

Ensure the selected Spring Cloud release is compatible with your Spring Boot version.

## 5. Enable Eureka Server

Add `@EnableEurekaServer` to the main application class:

```java
package in.ashokit.eureka;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(
            EurekaServerApplication.class, args);
    }
}
```

Adjust the package name to match your project.

## 6. application.yml Configuration

Create or update `src/main/resources/application.yml`:

```yaml
server:
  port: 8761

spring:
  application:
    name: EUREKASERVER

eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

The Eureka Server does not need to register itself as a client or fetch a registry in this standalone setup.

## 7. Configure Eureka Clients

Add the following dependency to CustomerApp, FriendApp, and PlanApp:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

Each client should have a unique application name and point to the Eureka Server.

Example for FriendApp:

```yaml
spring:
  application:
    name: FRIENDAPP

eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
```

Use `CUSTOMERAPP` and `PLANAPP` as the respective application names in the other services. Preserve each application's existing database and other configuration settings.

## 8. How to Run the Project

Start the applications in this order:

1. Start **Eureka Server**.
2. Start **PlanApp**.
3. Start **FriendApp**.
4. Start **CustomerApp**.

Open the Eureka dashboard:

http://localhost:8761

Once the client applications register successfully, their service names should appear on the dashboard.

## 9. Service Discovery

Instead of hardcoding a URL such as:

```text
http://localhost:8082/plans/1
```

CustomerApp can use a service name:

```text
http://PLANAPP/plans/1
```

This requires the HTTP client to be configured for service discovery and load balancing, such as a `@LoadBalanced` `RestTemplate` or another appropriately configured client. A Eureka Client dependency alone does not make every HTTP client resolve service names automatically.

## 10. Benefits

- Centralized service registry
- Dynamic service discovery
- Less dependence on hardcoded service URLs
- Support for multiple instances of a service
- Easier service communication as the application grows

## 11. Troubleshooting

- **Services are not visible:** Verify the Eureka URL and application names.
- **Connection refused:** Start Eureka Server first and confirm port `8761` is available.
- **Service name cannot be resolved:** Check that the calling HTTP client supports Eureka-based service discovery and load balancing.
- **Dependency errors:** Verify Spring Boot and Spring Cloud release compatibility.
- **Database errors:** Check each application's database configuration separately.

## 12. References

- [Spring Cloud Netflix Documentation](https://docs.spring.io/spring-cloud-netflix/reference/)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)

