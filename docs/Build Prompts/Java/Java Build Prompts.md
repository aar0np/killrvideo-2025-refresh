# KillrVideo Java Backend LLM Prompts (Astra DB Data API)

This document provides a series Vaughn-Vernon of prompts for a code-generation LLM to build the KillrVideo Java backend, targeting the Astra DB Data API as specified in "Java Backend Build Specification: KillrVideo 2025.md". Each prompt represents an incremental step.

## Phase 1: Project Foundation & Astra DB Configuration

**Context:** This phase establishes the basic project structure, dependencies, and initial configuration for Spring Boot and the Astra DB Java SDK.

---

### Prompt 1.1: Initialize Maven Project & Spring Boot Parent
```text
Create a new Maven project.
File: `pom.xml`

<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.killrvideo</groupId>
    <artifactId>killrvideo-java-service</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>killrvideo-java-service</name>
    <description>KillrVideo Java Backend Service with Astra DB Data API</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <java.version>21</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- Dependencies will be added in subsequent prompts -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```
This sets up the basic Maven structure, Spring Boot parent, and Java version.

---

### Prompt 1.2: Add Core Spring Boot & Utility Dependencies
```text
In `pom.xml`, add the following dependencies inside the `<dependencies>` section:
1.  `spring-boot-starter-web`: For building web applications, including RESTful APIs.
2.  `spring-boot-starter-validation`: For input validation using annotations.
3.  `spring-boot-starter-actuator`: For production-ready features like health checks and metrics.
4.  `org.projectlombok:lombok`: To reduce boilerplate code. Ensure the `optional>true</optional>` and annotation processor setup for Lombok if standard in your environment, or just the dependency.
    ```xml
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    ```
5.  `spring-boot-starter-test`: For testing Spring Boot applications (scope: `test`).
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    ```
Ensure these are added within the existing `<dependencies>` tags.
```

---

### Prompt 1.3: Create Main Application Class
```text
Create the main application class `KillrVideoApplication.java` in the package `com.killrvideo`.
This class should be annotated with `@SpringBootApplication`.

File: `src/main/java/com/killrvideo/KillrVideoApplication.java`
```java
package com.killrvideo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class KillrVideoApplication {

    public static void main(String[] args) {
        SpringApplication.run(KillrVideoApplication.class, args);
    }

}
```
This class serves as the entry point for the Spring Boot application.
```

---

### Prompt 1.4: Basic Application Configuration
```text
Create the main application configuration file `application.yml` in `src/main/resources/`.
Configure the server port to `8080` (or another suitable default like `8081` if `8080` is commonly used) and the servlet context path to `/api/v1`.

File: `src/main/resources/application.yml`
```yaml
server:
  port: 8080 # Consider 8081 if 8080 is often occupied
  servlet:
    context-path: /api/v1

spring:
  application:
    name: killrvideo-java-service

# Further configurations will be added later
```
This sets up basic server properties.
```

---

### Prompt 1.5: Simple Health Check REST Controller
```text
Create a package `com.killrvideo.controller`.
Inside this package, create a `HealthController.java`.
This controller should have a public GET mapping endpoint at `/health` that returns a simple string: "Service is up and running!".

File: `src/main/java/com/killrvideo/controller/HealthController.java`
```java
package com.killrvideo.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/health") // Mapped at the class level, relative to context path
public class HealthController {

    @GetMapping
    public String checkHealth() {
        return "Service is up and running!";
    }
}
```
This controller provides a basic endpoint to verify the application is running.
```

---

### Prompt 1.6: Add Astra DB SDK Dependency
```text
In `pom.xml`, add the dependency for the Astra DB Java SDK: `com.datastax.astra:astra-db-java`.
Use version `1.2.0` as a placeholder. (Note: The user should verify and update to the latest stable version from Maven Central if `1.2.0` is outdated or a preview).

Add this to the `<dependencies>` section:
```xml
<dependency>
    <groupId>com.datastax.astra</groupId>
    <artifactId>astra-db-java</artifactId>
    <version>1.2.0</version> <!-- Verify and use latest stable version -->
</dependency>
```
```

---

### Prompt 1.7: Astra DB Configuration Beans
```text
Create a package `com.killrvideo.config`.
Inside this package, create `AstraDBConfig.java`.
This class should be annotated with `@Configuration`.

1.  Inject the following properties from environment variables using `@Value`:
    *   `ASTRA_DB_API_ENDPOINT` (e.g., `https://<database_id>-<region>.apps.astra.datastax.com`)
    *   `ASTRA_DB_APPLICATION_TOKEN` (the application token)
    *   `ASTRA_DB_NAMESPACE` (the keyspace/namespace to use)

2.  Create a Spring bean method for `DataAPIClient` from `com.datastax.oss.driver.api.core.DataAPIClient`.
    *   This bean should be named `dataAPIClient`.
    *   The specification notes: `return new DataAPIClient(applicationToken); // Simpler constructor if token contains endpoint info or if using default Astra endpoint OR: new DataAPIClient(apiEndpoint, applicationToken); - check SDK specifics`.
    *   For now, let's assume the token might be an "Database Admin Token" which might not have the API endpoint embedded.
    *   **Action:** Use `return new DataAPIClient(applicationToken);` for now. If the SDK requires the endpoint explicitly for this constructor or if a different constructor is more appropriate for general tokens, this might need adjustment. The spec shows `new DataAPIClient(applicationToken)` in the example bean.

3.  Create a Spring bean method for `Database` from `com.datastax.oss.driver.api.core.Database`.
    *   This bean should be named `killrVideoDatabase`.
    *   It should take `DataAPIClient dataAPIClient`, `@Value("${ASTRA_DB_API_ENDPOINT}") String apiEndpoint`, and `@Value("${ASTRA_DB_NAMESPACE}") String namespace` as parameters.
    *   It should return `dataAPIClient.getDatabase(apiEndpoint, namespace)`. (Note: The spec example is `dataAPIClient.getDatabase(apiEndpoint, namespace)`. Double-check the `astra-db-java` SDK's `DataAPIClient` for the exact method to obtain a `Database` instance, as it could also be `dataAPIClient.getDatabase(namespace)` if the client is initialized with the endpoint, or if the token implies the endpoint). The current approach matches the specification's example.

File: `src/main/java/com/killrvideo/config/AstraDBConfig.java`
```java
package com.killrvideo.config;

import com.datastax.oss.driver.api.core.DataAPIClient;
import com.datastax.oss.driver.api.core.Database;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AstraDBConfig {

    @Value("${ASTRA_DB_API_ENDPOINT}")
    private String apiEndpoint;

    @Value("${ASTRA_DB_APPLICATION_TOKEN}")
    private String applicationToken;

    @Value("${ASTRA_DB_NAMESPACE}")
    private String namespace;

    @Bean
    public DataAPIClient dataAPIClient() {
        // This constructor assumes the token is a generic one not embedding the endpoint.
        // If using a token that includes endpoint info, or if the SDK offers a simpler
        // way to initialize with an endpoint for the client itself, adjust as needed.
        // The primary goal is to make the DataAPIClient available.
        // The current spec example for the bean is: new DataAPIClient(applicationToken)
        // However, the getDatabase call below uses the apiEndpoint.
        // Let's ensure the client is aware of the endpoint if it's not in the token.
        // A common pattern is new DataAPIClient(apiEndpoint, applicationToken);
        // Given the Database bean below needs apiEndpoint, it might be better to initialize client with it.
        // For now, sticking to the spec's direct DataAPIClient bean example:
        // return new DataAPIClient(applicationToken);
        // Let's refine this: The SDK's DataAPIClient typically just needs the token if it's a C* an Astra Token.
        // The endpoint is then passed when getting a specific database.
        // Let's use the simpler client constructor if the token itself is enough for authentication,
        // and the endpoint is specified when getting the database.
        // If the Astra token is like "AstraCS:...", it contains credentials.
        // DataAPIClient client = new DataAPIClient("AstraCS:TOKEN_HERE"); is typical for client init.
        // Database db = client.getDatabase("API_ENDPOINT_HERE", "KEYSPACE_HERE");
        // So, the current bean structure seems correct.
        return new DataAPIClient(applicationToken);
    }

    @Bean
    public Database killrVideoDatabase(DataAPIClient dataAPIClient) {
        // The spec mentions: dataAPIClient.getDatabase(apiEndpoint, namespace);
        // This assumes the DataAPIClient can connect to *any* db, and we specify which one here.
        return dataAPIClient.getDatabase(apiEndpoint, namespace);
    }
}
```
This sets up the necessary beans to interact with Astra DB using the Data API.
Ensure `com.datastax.oss.driver.api.core.DataAPIClient` and `com.datastax.oss.driver.api.core.Database` are the correct imports based on `astra-db-java` SDK (they might be under `com.datastax.astra.sdk...`).
*Correction based on `astra-db-java` common usage:*
The classes are typically `com.datastax.astra.client.DataAPIClient` and `com.datastax.astra.client.Database`.
Please use these imports:
`import com.datastax.astra.client.DataAPIClient;`
`import com.datastax.astra.client.Database;`
The constructor `new DataAPIClient(applicationToken)` is correct if `applicationToken` is the full Astra token string (e.g., "AstraCS:...").
The `getDatabase` method is often `dataAPIClient.getDatabase(namespace)` if the API endpoint is inferred or set globally, OR `dataAPIClient.getDatabase(apiEndpoint, namespace)`. The specification uses the latter.
The prompt needs to be updated to reflect these correct class names. I'll assume the LLM can infer standard library paths for `astra-db-java` or I'll correct it in the actual prompt content below.
The `DataAPIClient` and `Database` classes are indeed in `com.datastax.astra.client`. I will ensure the generated code uses these.
```

---

### Prompt 1.8: Initial POJOs for Core Entities
```text
Create a package `com.killrvideo.dto`. This package will hold Data Transfer Objects and domain objects (POJOs for collections).

1.  **User POJO:**
    Create `User.java` in `com.killrvideo.dto`.
    Fields:
    *   `userId (String)`: Should be the primary identifier, consider annotating with `@JsonProperty("user_id")` or ensure field name matches collection.
    *   `firstName (String)`: `@JsonProperty("first_name")`
    *   `lastName (String)`: `@JsonProperty("last_name")`
    *   `email (String)`
    *   `hashedPassword (String)`: `@JsonProperty("hashed_password")` (won't be returned in all DTOs)
    *   `createdAt (java.time.Instant)`: `@JsonProperty("created_at")`
    Use Lombok's `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`.

2.  **Video POJO:**
    Create `Video.java` in `com.killrvideo.dto`.
    Fields:
    *   `videoId (String)`: `@JsonProperty("video_id")`
    *   `userId (String)`: `@JsonProperty("user_id")`
    *   `name (String)`
    *   `description (String)`
    *   `tags (java.util.Set<String>)`
    *   `location (String)`: (e.g., YouTube ID or URL)
    *   `previewImageLocation (String)`: `@JsonProperty("preview_image_location")`
    *   `addedDate (java.time.Instant)`: `@JsonProperty("added_date")`
    *   `vector (float[])`: For vector embeddings. This will be populated later.
    Use Lombok's `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`.

3.  **Comment POJO:**
    Create `Comment.java` in `com.killrvideo.dto`.
    Fields:
    *   `commentId (String)`: `@JsonProperty("comment_id")`
    *   `videoId (String)`: `@JsonProperty("video_id")`
    *   `userId (String)`: `@JsonProperty("user_id")`
    *   `comment (String)`
    *   `timestamp (java.time.Instant)`
    Use Lombok's `@Data`, `@NoArgsConstructor`, `@AllArgsConstructor`.

For all POJOs, ensure Jackson annotations (`@JsonProperty`) are used if the Java field names differ from the desired JSON property names / Astra DB field names. The specification implies these POJOs map to collection documents.

File: `src/main/java/com/killrvideo/dto/User.java`
```java
package com.killrvideo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.Instant;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @JsonProperty("user_id")
    private String userId;
    @JsonProperty("first_name")
    private String firstName;
    @JsonProperty("last_name")
    private String lastName;
    private String email;
    @JsonProperty("hashed_password")
    private String hashedPassword; // Be careful with exposing this
    @JsonProperty("created_at")
    private Instant createdAt;
}
```

File: `src/main/java/com/killrvideo/dto/Video.java`
```java
package com.killrvideo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.Instant;
import java.util.Set;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Video {
    @JsonProperty("video_id")
    private String videoId;
    @JsonProperty("user_id")
    private String userId;
    private String name;
    private String description;
    private Set<String> tags;
    private String location; // e.g., YouTube ID or internal ref
    @JsonProperty("preview_image_location")
    private String previewImageLocation;
    @JsonProperty("added_date")
    private Instant addedDate;
    private float[] vector; // For vector embeddings
}
```

File: `src/main/java/com/killrvideo/dto/Comment.java`
```java
package com.killrvideo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.Instant;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Comment {
    @JsonProperty("comment_id")
    private String commentId;
    @JsonProperty("video_id")
    private String videoId;
    @JsonProperty("user_id")
    private String userId;
    private String comment;
    private Instant timestamp;
}
```
These POJOs represent the data structures for our main entities.
```

---
## Phase 2: User Management & Authentication

**Context:** This phase focuses on setting up user registration, login, and securing endpoints using JWT. It builds upon the Astra DB configuration and User POJO from Phase 1.

---

### Prompt 2.1: Add Security Dependencies (Spring Security & JWT)
```text
In `pom.xml`, add the following dependencies for Spring Security and JWT handling:
1.  `spring-boot-starter-security`: Core Spring Security dependency.
2.  `io.jsonwebtoken:jjwt-api`: JWT API.
3.  `io.jsonwebtoken:jjwt-impl`: JWT Implementation.
4.  `io.jsonwebtoken:jjwt-jackson`: JWT Jackson support for JSON serialization/deserialization.

Specify appropriate versions for `jjwt` (e.g., `0.11.5` or latest stable).
Example for `jjwt` (versions may vary, check Maven Central for latest stable `0.12.x` or similar):
```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version> <!-- Check for latest stable version -->
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version> <!-- Check for latest stable version -->
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version> <!-- Check for latest stable version -->
    <scope>runtime</scope>
</dependency>
```
```

---

### Prompt 2.2: Configure JWT Properties in `application.yml`
```text
In `src/main/resources/application.yml`, add configuration for JWT:
```yaml
killrvideo:
  jwt:
    secret: YourSuperSecretKeyForJWTGenerationThatIsLongAndSecure12345 # Replace with a strong, environment-specific secret
    expiration-ms: 3600000 # 1 hour
```
These properties will be used by the JWT provider. **Note:** The secret key should ideally be externalized via environment variables in a production setup.
```

---

### Prompt 2.3: Create `UserDao` for Astra DB Operations
```text
Create a package `com.killrvideo.dao`.
Inside this package, create `UserDao.java`. This class will handle data access for `User` entities using the Astra DB Data API.

1.  Annotate the class with `@Repository`.
2.  Inject the `Database` bean (named `killrVideoDatabase`) configured in `AstraDBConfig.java`.
3.  In the constructor, get the "users" collection: `this.userCollection = killrVideoDatabase.getCollection("users", User.class);`. Ensure `User.class` is the POJO created earlier.
4.  Implement the following methods:
    *   `public Optional<User> findById(String userId)`: Uses `userCollection.findById(userId)`. (Note: `findById` in the SDK returns `T`, so wrap it with `Optional.ofNullable`).
    *   `public Optional<User> findByEmail(String email)`: Uses `userCollection.findOne(Filters.eq("email", email))`. (The SDK's `findOne` returns an `Optional<T>`).
    *   `public User save(User user)`: Uses `userCollection.insertOne(user)` and returns the saved user. The `insertOne` method in the SDK might return `InsertOneResult` or `void`. If it's `void` or `InsertOneResult`, the method should still return the input `user` object as it typically contains the ID generated client-side or is passed in. For Astra Data API, IDs are usually client-generated (e.g., UUIDs). Assume `insertOne` doesn't return the entity, so return the passed `user`. *Correction:* The Data API `insertOne` often returns an `InsertOneResult`. The entity itself is what you passed in. It's good practice to ensure the `userId` is set before calling `save`.
    *   `public boolean existsByEmail(String email)`: Implement by calling `findByEmail(email).isPresent()`.

File: `src/main/java/com/killrvideo/dao/UserDao.java`
```java
package com.killrvideo.dao;

import com.datastax.astra.client.Database;
import com.datastax.astra.client.Collection;
import com.datastax.astra.client.model.Filters;
import com.killrvideo.dto.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class UserDao {

    private final Collection<User> userCollection;

    @Autowired
    public UserDao(Database killrVideoDatabase) {
        // Assuming User.class is your POJO for the user documents
        this.userCollection = killrVideoDatabase.getCollection("users", User.class);
    }

    public Optional<User> findById(String userId) {
        return Optional.ofNullable(userCollection.findById(userId));
    }

    public Optional<User> findByEmail(String email) {
        // findOne method already returns Optional<User>
        return userCollection.findOne(Filters.eq("email", email));
    }

    public User save(User user) {
        // insertOne typically returns an InsertOneResult or void.
        // The user object passed in is saved.
        // Ensure userId is populated before this call if it's client-generated.
        userCollection.insertOne(user);
        return user; // Return the entity that was passed, now persisted.
    }

    public boolean existsByEmail(String email) {
        return findByEmail(email).isPresent();
    }
}
```
This DAO provides methods to interact with the `users` collection in Astra DB.
```

---

### Prompt 2.4: Create Custom `UserDetails` and `UserDetailsService`
```text
Create a package `com.killrvideo.security`.

1.  **`UserDetailsImpl.java`**:
    Create `UserDetailsImpl.java` implementing `org.springframework.security.core.userdetails.UserDetails`.
    *   It should store `userId (String)`, `username (String)` (which will be the email), `password (String)`, and `authorities (Collection<? extends GrantedAuthority>)`.
    *   Implement all methods from `UserDetails`. For simplicity, `getAuthorities()` can return an empty list or a default "ROLE_USER". `isAccountNonExpired`, `isAccountNonLocked`, `isCredentialsNonExpired`, `isEnabled` can all return `true`.
    *   Add a static build method: `public static UserDetailsImpl build(User user)` that takes your `User` POJO and maps it to `UserDetailsImpl`. Use email as username.

2.  **`UserDetailsServiceImpl.java`**:
    Create `UserDetailsServiceImpl.java` implementing `org.springframework.security.core.userdetails.UserDetailsService`.
    *   Annotate with `@Service`.
    *   Inject `UserDao`.
    *   Implement `loadUserByUsername(String username)` (where username is email). This method should:
        *   Call `userDao.findByEmail(username)`.
        *   If user not found, throw `UsernameNotFoundException`.
        *   If found, create and return `UserDetailsImpl.build(user)`.


File: `src/main/java/com/killrvideo/security/UserDetailsImpl.java`
```java
package com.killrvideo.security;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.killrvideo.dto.User;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

public class UserDetailsImpl implements UserDetails {
    private static final long serialVersionUID = 1L;

    private String id;
    private String username; // email
    @JsonIgnore
    private String password;
    private Collection<? extends GrantedAuthority> authorities;

    public UserDetailsImpl(String id, String email, String password, Collection<? extends GrantedAuthority> authorities) {
        this.id = id;
        this.username = email;
        this.password = password;
        this.authorities = authorities;
    }

    public static UserDetailsImpl build(User user) {
        // For now, all users get ROLE_USER. This can be expanded later.
        List<GrantedAuthority> authorities = Collections.singletonList(new SimpleGrantedAuthority("ROLE_USER"));

        return new UserDetailsImpl(
                user.getUserId(),
                user.getEmail(),
                user.getHashedPassword(),
                authorities);
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    public String getId() {
        return id;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        UserDetailsImpl user = (UserDetailsImpl) o;
        return Objects.equals(id, user.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

File: `src/main/java/com/killrvideo/security/UserDetailsServiceImpl.java`
```java
package com.killrvideo.security;

import com.killrvideo.dao.UserDao;
import com.killrvideo.dto.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional; // Optional if no complex transactions

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    UserDao userDao;

    @Override
    @Transactional // Optional, good practice for service methods involving DB access
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        User user = userDao.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException("User Not Found with email: " + email));

        return UserDetailsImpl.build(user);
    }
}
```
```

---

### Prompt 2.5: Create `JwtUtils` for Token Generation and Validation
```text
In the `com.killrvideo.security` package, create `JwtUtils.java`.
This class will be responsible for generating, parsing, and validating JWTs.

1.  Annotate with `@Component`.
2.  Inject JWT secret and expiration from `application.yml` using `@Value("${killrvideo.jwt.secret}")` and `@Value("${killrvideo.jwt.expiration-ms}")`.
3.  Implement `public String generateJwtToken(Authentication authentication)`:
    *   Get `UserDetailsImpl` from `authentication.getPrincipal()`.
    *   Build a JWT with the username (email) as subject, issued at, expiration date, and sign with HS512 algorithm and the secret key. Use `io.jsonwebtoken.Jwts`.
4.  Implement `public String generateTokenFromUsername(String username)`:
    *   Similar to above, but takes username directly. Useful for registration response.
5.  Implement `public String getEmailFromJwtToken(String token)`:
    *   Parse the token and get the subject (email).
6.  Implement `public boolean validateJwtToken(String authToken)`:
    *   Try to parse the token. Catch exceptions (`SignatureException`, `MalformedJwtException`, `ExpiredJwtException`, `UnsupportedJwtException`, `IllegalArgumentException`) and log them. Return `true` if valid, `false` otherwise.
    *   Make sure to use `Jwts.parserBuilder().setSigningKey(key()).build().parseClaimsJws(token)`.
    *   `key()` helper method: `private Key key() { return Keys.hmacShaKeyFor(Decoders.BASE64.decode(jwtSecret)); }`

File: `src/main/java/com/killrvideo/security/JwtUtils.java`
```java
package com.killrvideo.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.io.Decoders;
import io.jsonwebtoken.security.Keys;
import io.jsonwebtoken.security.SignatureException; // Correct import for SignatureException
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;

@Component
public class JwtUtils {
    private static final Logger logger = LoggerFactory.getLogger(JwtUtils.class);

    @Value("${killrvideo.jwt.secret}")
    private String jwtSecret;

    @Value("${killrvideo.jwt.expiration-ms}")
    private int jwtExpirationMs;

    public String generateJwtToken(Authentication authentication) {
        UserDetailsImpl userPrincipal = (UserDetailsImpl) authentication.getPrincipal();
        return generateTokenFromUsername(userPrincipal.getUsername());
    }

    public String generateTokenFromUsername(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date((new Date()).getTime() + jwtExpirationMs))
                .signWith(key(), SignatureAlgorithm.HS512)
                .compact();
    }

    private Key key() {
        return Keys.hmacShaKeyFor(Decoders.BASE64.decode(jwtSecret));
    }

    public String getEmailFromJwtToken(String token) {
        return Jwts.parserBuilder().setSigningKey(key()).build()
                   .parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parserBuilder().setSigningKey(key()).build().parseClaimsJws(authToken);
            return true;
        } catch (SignatureException e) {
            logger.error("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            logger.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            logger.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            logger.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            logger.error("JWT claims string is empty: {}", e.getMessage());
        }
        return false;
    }
}
```
```

---

### Prompt 2.6: Create `AuthTokenFilter` for Processing JWTs in Requests
```text
In the `com.killrvideo.security` package, create `AuthTokenFilter.java` extending `org.springframework.web.filter.OncePerRequestFilter`.

1.  Inject `JwtUtils` and `UserDetailsServiceImpl`.
2.  Override `doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)`:
    *   Parse JWT from the `Authorization` header (if present and starts with "Bearer ").
    *   If JWT is valid:
        *   Get username (email) from token.
        *   Load `UserDetails` using `UserDetailsServiceImpl`.
        *   Create `UsernamePasswordAuthenticationToken` and set it in `SecurityContextHolder`.
    *   Call `filterChain.doFilter(request, response)`.
3.  Helper method `private String parseJwt(HttpServletRequest request)`:
    *   Extracts token from "Authorization: Bearer <token>" header.

File: `src/main/java/com/killrvideo/security/AuthTokenFilter.java`
```java
package com.killrvideo.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

public class AuthTokenFilter extends OncePerRequestFilter {
    @Autowired
    private JwtUtils jwtUtils;

    @Autowired
    private UserDetailsServiceImpl userDetailsService;

    private static final Logger logger = LoggerFactory.getLogger(AuthTokenFilter.class);

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String jwt = parseJwt(request);
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                String email = jwtUtils.getEmailFromJwtToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserByUsername(email);
                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(
                                userDetails,
                                null,
                                userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set user authentication: {}", e.getMessage());
        }

        filterChain.doFilter(request, response);
    }

    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");

        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);
        }

        return null;
    }
}
```
**Note:** This filter needs to be registered in the Spring Security configuration.
```

---

### Prompt 2.7: Create `AuthEntryPointJwt` for Handling Unauthorized Errors
```text
In the `com.killrvideo.security` package, create `AuthEntryPointJwt.java` implementing `org.springframework.security.web.AuthenticationEntryPoint`.

1.  Annotate with `@Component`.
2.  Override `commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException)`:
    *   Log the error.
    *   Send an HTTP 401 Unauthorized error. Set content type to `application/json`.
    *   Optionally, write a JSON error message to the response body (e.g., `{"error": "Unauthorized", "message": "authException.getMessage()"}`). Use Jackson's `ObjectMapper` for this.

File: `src/main/java/com/killrvideo/security/AuthEntryPointJwt.java`
```java
package com.killrvideo.security;

import com.fasterxml.jackson.databind.ObjectMapper;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

@Component
public class AuthEntryPointJwt implements AuthenticationEntryPoint {

    private static final Logger logger = LoggerFactory.getLogger(AuthEntryPointJwt.class);

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException)
            throws IOException, ServletException {
        logger.error("Unauthorized error: {}", authException.getMessage());

        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);

        final Map<String, Object> body = new HashMap<>();
        body.put("status", HttpServletResponse.SC_UNAUTHORIZED);
        body.put("error", "Unauthorized");
        body.put("message", authException.getMessage());
        body.put("path", request.getServletPath());

        final ObjectMapper mapper = new ObjectMapper();
        mapper.writeValue(response.getOutputStream(), body);
    }
}
```
```

---

### Prompt 2.8: Configure Spring Security (`WebSecurityConfig.java`)
```text
In `com.killrvideo.config` package (or a new `com.killrvideo.security.config` package), create `WebSecurityConfig.java`.

1.  Annotate with `@Configuration`, `@EnableWebSecurity`, and `@EnableMethodSecurity` (for method-level security like `@PreAuthorize` if needed later).
2.  Inject `UserDetailsServiceImpl`, `AuthEntryPointJwt`.
3.  Define a bean for `AuthTokenFilter`: `public AuthTokenFilter authenticationJwtTokenFilter() { return new AuthTokenFilter(); }`.
4.  Define a bean for `PasswordEncoder`: `public PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }`.
5.  Define a bean for `DaoAuthenticationProvider`:
    *   Set `userDetailsService` and `passwordEncoder`.
6.  Define a bean for `AuthenticationManager`: `public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception { return authConfig.getAuthenticationManager(); }`.
7.  Define the main `SecurityFilterChain` bean:
    *   Disable CSRF and CORS (can be configured properly later if needed: `csrf(AbstractHttpConfigurer::disable)`).
    *   Configure exception handling to use `AuthEntryPointJwt`.
    *   Set session management to stateless: `sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))`.
    *   Authorize HTTP requests:
        *   Permit all for `/api/v1/auth/**`, `/api/v1/health`, `/openapi/**`, `/swagger-ui/**`, `/swagger-ui.html`. (Adjust OpenAPI paths if they differ).
        *   Any other request should be authenticated.
    *   Register `AuthTokenFilter` before `UsernamePasswordAuthenticationFilter`.
    *   Register the `DaoAuthenticationProvider`.

File: `src/main/java/com/killrvideo/config/WebSecurityConfig.java`
```java
package com.killrvideo.config; // Or com.killrvideo.security.config

import com.killrvideo.security.AuthEntryPointJwt;
import com.killrvideo.security.AuthTokenFilter;
import com.killrvideo.security.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractHttpConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity // for @PreAuthorize, etc.
public class WebSecurityConfig {

    @Autowired
    UserDetailsServiceImpl userDetailsService;

    @Autowired
    private AuthEntryPointJwt unauthorizedHandler;

    @Bean
    public AuthTokenFilter authenticationJwtTokenFilter() {
        return new AuthTokenFilter();
    }

    @Bean
    public DaoAuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
            .exceptionHandling(exception -> exception.authenticationEntryPoint(unauthorizedHandler))
            .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth ->
                auth.requestMatchers("/api/v1/auth/**").permitAll()
                    .requestMatchers("/api/v1/health").permitAll()
                    // Permit OpenAPI/Swagger UI paths if/when added
                    .requestMatchers("/openapi/**", "/swagger-ui/**", "/swagger-ui.html", "/v3/api-docs/**").permitAll()
                    .anyRequest().authenticated()
            );

        http.authenticationProvider(authenticationProvider());
        http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```
This configuration secures the application, defining public and protected routes.
```

---

### Prompt 2.9: Create Authentication DTOs
```text
In the `com.killrvideo.dto` package (or a new sub-package like `com.killrvideo.dto.auth`), create DTOs for authentication:

1.  **`LoginRequest.java`**:
    Fields: `email (String)`, `password (String)`.
    Add `@NotBlank` validation for both.

2.  **`SignupRequest.java`**:
    Fields: `firstName (String)`, `lastName (String)`, `email (String)`, `password (String)`.
    Add `@NotBlank` for all, `@Size` for password (e.g., min 6, max 40), `@Email` for email.

3.  **`JwtResponse.java`**:
    Fields: `token (String)`, `id (String)` (user ID), `email (String)`. Optionally `type (String)` (default "Bearer").

File: `src/main/java/com/killrvideo/dto/LoginRequest.java`
```java
package com.killrvideo.dto; // or com.killrvideo.dto.auth

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import lombok.Data;

@Data
public class LoginRequest {
    @NotBlank
    @Email
    private String email;

    @NotBlank
    private String password;
}
```

File: `src/main/java/com/killrvideo/dto/SignupRequest.java`
```java
package com.killrvideo.dto; // or com.killrvideo.dto.auth

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

@Data
public class SignupRequest {
    @NotBlank
    @Size(min = 1, max = 50)
    private String firstName;

    @NotBlank
    @Size(min = 1, max = 50)
    private String lastName;

    @NotBlank
    @Size(max = 50)
    @Email
    private String email;

    @NotBlank
    @Size(min = 6, max = 120) // Adjusted max size for hashed passwords if stored directly
    private String password;
}
```

File: `src/main/java/com/killrvideo/dto/JwtResponse.java`
```java
package com.killrvideo.dto; // or com.killrvideo.dto.auth

import lombok.Data;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;

@Data
@RequiredArgsConstructor // Creates constructor for final fields
public class JwtResponse {
    @NonNull
    private String token;
    private String type = "Bearer";
    @NonNull
    private String id;
    @NonNull
    private String email;

    // Optional: Add roles if you plan to use them
    // private List<String> roles;
}
```
```

---

### Prompt 2.10: Create `AuthController` for Registration and Login
```text
In `com.killrvideo.controller`, create `AuthController.java`.

1.  Annotate with `@RestController` and `@RequestMapping("/api/v1/auth")`. (Note: `/api/v1` is context path, so just `@RequestMapping("/auth")` here).
2.  Inject `AuthenticationManager`, `UserDao`, `PasswordEncoder`, `JwtUtils`.
3.  **`POST /signup` endpoint**:
    *   Takes `@Valid @RequestBody SignupRequest signUpRequest`.
    *   Check if email already exists using `userDao.existsByEmail()`. If so, return `ResponseEntity.badRequest().body("Error: Email is already in use!")`.
    *   Create a new `User` object:
        *   Generate `userId` (e.g., `UUID.randomUUID().toString()`).
        *   Set fields from `signUpRequest`, encode password using `passwordEncoder`.
        *   Set `createdAt` to `Instant.now()`.
    *   Save user using `userDao.save(user)`.
    *   Return `ResponseEntity.ok("User registered successfully!")`.
4.  **`POST /signin` endpoint**:
    *   Takes `@Valid @RequestBody LoginRequest loginRequest`.
    *   Authenticate user: `Authentication authentication = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(loginRequest.getEmail(), loginRequest.getPassword()));`.
    *   Set authentication in `SecurityContextHolder`.
    *   Generate JWT using `jwtUtils.generateJwtToken(authentication)`.
    *   Get `UserDetailsImpl` from authentication principal.
    *   Return `ResponseEntity.ok(new JwtResponse(jwt, userDetails.getId(), userDetails.getUsername()))`.

File: `src/main/java/com/killrvideo/controller/AuthController.java`
```java
package com.killrvideo.controller;

import com.killrvideo.dao.UserDao;
import com.killrvideo.dto.JwtResponse;
import com.killrvideo.dto.LoginRequest;
import com.killrvideo.dto.SignupRequest;
import com.killrvideo.dto.User;
import com.killrvideo.security.JwtUtils;
import com.killrvideo.security.UserDetailsImpl;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.util.UUID;

@CrossOrigin(origins = "*", maxAge = 3600) // Optional: Configure CORS if needed
@RestController
@RequestMapping("/auth") // Relative to /api/v1 context path
public class AuthController {

    @Autowired
    AuthenticationManager authenticationManager;

    @Autowired
    UserDao userDao;

    @Autowired
    PasswordEncoder encoder;

    @Autowired
    JwtUtils jwtUtils;

    @PostMapping("/signin")
    public ResponseEntity<?> authenticateUser(@Valid @RequestBody LoginRequest loginRequest) {
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getEmail(), loginRequest.getPassword()));

        SecurityContextHolder.getContext().setAuthentication(authentication);
        String jwt = jwtUtils.generateJwtToken(authentication);

        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        // List<String> roles = userDetails.getAuthorities().stream()
        // .map(item -> item.getAuthority())
        // .collect(Collectors.toList());

        return ResponseEntity.ok(new JwtResponse(jwt,
                                                 userDetails.getId(),
                                                 userDetails.getUsername()));
    }

    @PostMapping("/signup")
    public ResponseEntity<?> registerUser(@Valid @RequestBody SignupRequest signUpRequest) {
        if (userDao.existsByEmail(signUpRequest.getEmail())) {
            return ResponseEntity
                    .badRequest()
                    .body("Error: Email is already in use!");
        }

        // Create new user's account
        User user = new User();
        user.setUserId(UUID.randomUUID().toString());
        user.setFirstName(signUpRequest.getFirstName());
        user.setLastName(signUpRequest.getLastName());
        user.setEmail(signUpRequest.getEmail());
        user.setHashedPassword(encoder.encode(signUpRequest.getPassword()));
        user.setCreatedAt(Instant.now());

        userDao.save(user);

        return ResponseEntity.ok("User registered successfully!");
    }
}
```
This controller handles user sign-up and sign-in, issuing JWTs upon successful authentication. The `@CrossOrigin` annotation is added as a common requirement for web UIs; it can be refined later.
The path `/api/v1/auth/**` should be permitted in `WebSecurityConfig` if it's intended to be public or secure it appropriately.
```

---
## Phase 3: Video Management Core

**Context:** This phase implements the core functionalities for managing videos, including creating DAOs, controllers, and DTOs for video operations. It assumes user authentication is in place.

---

### Prompt 3.1: Create `VideoDao` for Astra DB Video Operations
```text
Modify `com.killrvideo.dao.VideoDao.java`.

1.  Annotate with `@Repository`.
2.  Inject the `Database` bean (`killrVideoDatabase`).
3.  In the constructor, get the "videos" collection: `this.videoCollection = killrVideoDatabase.getCollection("videos", Video.class);`.
4.  Implement methods:
    *   `public Video save(Video video)`: Uses `videoCollection.insertOne(video)`. Ensure `videoId` is set (e.g., UUID) before saving if not already present. Return the input `video`.
    *   `public Optional<Video> findById(String videoId)`: Uses `videoCollection.findById(videoId)` and wraps with `Optional.ofNullable()`.
    *   `public FindIterable<Video> findLatest(int limit)`:
        *   Uses `videoCollection.find(new FindOptions().sort("added_date", -1).limit(limit))`. Sort by `added_date` descending. The field name "added_date" must match the `@JsonProperty` in `Video.java` or the actual field name in the DB.
    *   `public FindIterable<Video> findByUserId(String userId, int limit)`:
        *   Uses `videoCollection.find(Filters.eq("user_id", userId), new FindOptions().sort("added_date", -1).limit(limit))`.
    *   `public void update(Video video)`: Uses `videoCollection.replaceOne(Filters.eq("video_id", video.getVideoId()), video)`. (Or `updateOne` if partial updates are preferred, but `replaceOne` is simpler for full object updates).

Note: `FindIterable<T>` is from the Astra SDK. Controllers will need to convert this to `List<T>`.
The sort field `added_date` should match the JSON property name if Jackson field naming strategy is not default.

File: `src/main/java/com/killrvideo/dao/VideoDao.java`
```java
package com.killrvideo.dao;

import com.datastax.astra.client.Collection;
import com.datastax.astra.client.Database;
import com.datastax.astra.client.model.Filters;
import com.datastax.astra.client.model.FindIterable;
import com.datastax.astra.client.model.FindOptions;
import com.killrvideo.dto.Video;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class VideoDao {

    private final Collection<Video> videoCollection;

    @Autowired
    public VideoDao(Database killrVideoDatabase) {
        this.videoCollection = killrVideoDatabase.getCollection("videos", Video.class);
    }

    public Video save(Video video) {
        // Ensure videoId is set before saving, typically a UUID.
        // This might be done in the service layer or controller before calling DAO.
        videoCollection.insertOne(video);
        return video;
    }

    public Optional<Video> findById(String videoId) {
        return Optional.ofNullable(videoCollection.findById(videoId));
    }

    /**
     * Finds the latest videos.
     * @param limit Max number of videos to return.
     * @return FindIterable of videos.
     */
    public FindIterable<Video> findLatest(int limit) {
        // Assuming "added_date" is the correct field name for sorting in the database.
        // It should match the @JsonProperty or actual field name in Astra.
        return videoCollection.find(new FindOptions().sort("added_date", -1).limit(limit));
    }

    /**
     * Finds videos by user ID.
     * @param userId The ID of the user.
     * @param limit Max number of videos to return.
     * @return FindIterable of videos.
     */
    public FindIterable<Video> findByUserId(String userId, int limit) {
        // Assuming "user_id" is the correct field name for filtering.
        return videoCollection.find(Filters.eq("user_id", userId), new FindOptions().sort("added_date", -1).limit(limit));
    }

    /**
     * Updates an existing video document by replacing it.
     * @param video The video object with updated information. videoId must be present.
     */
    public void update(Video video) {
        // replaceOne requires a filter to identify the document and the replacement document.
        // Ensure video.getVideoId() is not null.
        if (video.getVideoId() == null) {
            throw new IllegalArgumentException("Video ID cannot be null for update.");
        }
        videoCollection.replaceOne(Filters.eq("video_id", video.getVideoId()), video);
        // For partial updates, investigate videoCollection.updateOne() with update operators.
    }

    // findByTag and findByVector will be added later.
}
```
This DAO handles database operations for videos.
```

---

### Prompt 3.2: Create Video DTOs
```text
In `com.killrvideo.dto` (or a sub-package like `com.killrvideo.dto.video`), create DTOs for video operations:

1.  **`SubmitVideoRequest.java`**:
    Fields: `name (String)`, `description (String)`, `tags (Set<String>)`, `location (String)` (e.g. YouTube video ID or a URL where the video can be found/retrieved from).
    Add `@NotBlank` to `name` and `location`. Tags can be optional or have size constraints.

2.  **`VideoResponse.java`**: (Could also reuse `Video.java` POJO if it's suitable for responses, but a dedicated DTO gives more control).
    Fields: `videoId (String)`, `userId (String)`, `name (String)`, `description (String)`, `tags (Set<String>)`, `location (String)`, `previewImageLocation (String)`, `addedDate (Instant)`.
    (Optionally add user details like `userFirstName`, `userLastName` if denormalization is desired, or fetch separately).

For now, `Video.java` can serve as `VideoResponse`. If specific fields need to be excluded or added for responses, a dedicated `VideoResponse` DTO can be created later. The vector field in `Video.java` should ideally not be sent in general responses unless specifically requested.

File: `src/main/java/com/killrvideo/dto/SubmitVideoRequest.java`
```java
package com.killrvideo.dto; // or com.killrvideo.dto.video

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;
import java.util.Set;

@Data
public class SubmitVideoRequest {
    @NotBlank
    @Size(max = 100)
    private String name;

    @Size(max = 1000)
    private String description;

    private Set<String> tags; // Consider validation for tag format/count

    @NotBlank
    @Size(max = 255) // Assuming location is a URL or ID
    private String location; // e.g., YouTube ID or reference
}
```
This DTO is used for submitting new video metadata.
```

---

### Prompt 3.3: Create `VideoController`
```text
In `com.killrvideo.controller`, create `VideoController.java`.

1.  Annotate with `@RestController` and `@RequestMapping("/videos")`. (Relative to `/api/v1`).
2.  Inject `VideoDao`.
3.  **`POST /` endpoint (Submit Video Metadata)**:
    *   Takes `@Valid @RequestBody SubmitVideoRequest submitVideoRequest` and `Authentication authentication`.
    *   Get `UserDetailsImpl` from `authentication.getPrincipal()` to retrieve the `userId`.
    *   Create a new `Video` object:
        *   `videoId = UUID.randomUUID().toString()`.
        *   `userId` from authenticated user.
        *   Populate `name`, `description`, `tags`, `location` from `submitVideoRequest`.
        *   `addedDate = Instant.now()`.
        *   `previewImageLocation` can be null or set to a default initially.
        *   `vector` field will be null initially (populated by async process later).
    *   Save using `videoDao.save(video)`.
    *   Return `ResponseEntity.status(HttpStatus.CREATED).body(savedVideo)`.
4.  **`GET /{videoId}` endpoint (Get Video Details)**:
    *   Takes `@PathVariable String videoId`.
    *   Fetch video using `videoDao.findById(videoId)`.
    *   If not found, return `ResponseEntity.notFound().build()`.
    *   If found, return `ResponseEntity.ok(video)`.
5.  **`GET /latest` endpoint (Get Latest Videos)**:
    *   Takes `@RequestParam(defaultValue = "10") int limit`.
    *   Fetch videos using `videoDao.findLatest(limit)`.
    *   Convert `FindIterable<Video>` to `List<Video>`: `videos.all()`.
    *   Return `ResponseEntity.ok(videoList)`.
6.  **`GET /user/{userId}` endpoint (Get Videos by User)**:
    *   Takes `@PathVariable String userId`, `@RequestParam(defaultValue = "10") int limit`.
    *   Fetch videos using `videoDao.findByUserId(userId, limit)`.
    *   Convert to `List<Video>` and return `ResponseEntity.ok(videoList)`.

Remember to handle `FindIterable` conversion to `List`.

File: `src/main/java/com/killrvideo/controller/VideoController.java`
```java
package com.killrvideo.controller;

import com.killrvideo.dao.VideoDao;
import com.killrvideo.dto.SubmitVideoRequest;
import com.killrvideo.dto.Video; // Assuming Video POJO is used for response
import com.killrvideo.security.UserDetailsImpl;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.Authentication;
import org.springframework.security.access.prepost.PreAuthorize; // For endpoint protection
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/videos") // Relative to /api/v1 context path
public class VideoController {

    @Autowired
    private VideoDao videoDao;

    @PostMapping
    @PreAuthorize("isAuthenticated()") // Ensure user is logged in
    public ResponseEntity<Video> submitVideo(@Valid @RequestBody SubmitVideoRequest submitVideoRequest, Authentication authentication) {
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        String userId = userDetails.getId();

        Video video = new Video();
        video.setVideoId(UUID.randomUUID().toString());
        video.setUserId(userId);
        video.setName(submitVideoRequest.getName());
        video.setDescription(submitVideoRequest.getDescription());
        video.setTags(submitVideoRequest.getTags());
        video.setLocation(submitVideoRequest.getLocation());
        video.setAddedDate(Instant.now());
        // previewImageLocation and vector will be set later by async processes
        video.setPreviewImageLocation(null); // Placeholder
        video.setVector(null); // Placeholder

        Video savedVideo = videoDao.save(video);
        return ResponseEntity.status(HttpStatus.CREATED).body(savedVideo);
    }

    @GetMapping("/{videoId}")
    public ResponseEntity<Video> getVideoById(@PathVariable String videoId) {
        return videoDao.findById(videoId)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @GetMapping("/latest")
    public ResponseEntity<List<Video>> getLatestVideos(@RequestParam(defaultValue = "10") int limit) {
        if (limit <= 0 || limit > 100) { // Basic validation for limit
            limit = 10;
        }
        List<Video> videos = videoDao.findLatest(limit).all();
        return ResponseEntity.ok(videos);
    }

    @GetMapping("/user/{userId}")
    public ResponseEntity<List<Video>> getVideosByUser(@PathVariable String userId, @RequestParam(defaultValue = "10") int limit) {
        if (limit <= 0 || limit > 100) {
            limit = 10;
        }
        List<Video> videos = videoDao.findByUserId(userId, limit).all();
        return ResponseEntity.ok(videos);
    }
}
```
This controller handles video submission and retrieval. `@PreAuthorize("isAuthenticated()")` is added to the submit endpoint.
```

---
## Phase 4: Comment Management Core

**Context:** This phase implements features for users to comment on videos. It involves creating a DAO, DTOs, and controller endpoints for comments.

---

### Prompt 4.1: Create `CommentDao` for Astra DB Comment Operations
```text
In `com.killrvideo.dao`, create `CommentDao.java`.

1.  Annotate with `@Repository`.
2.  Inject the `Database` bean (`killrVideoDatabase`).
3.  In constructor, get "comments" collection: `this.commentCollection = killrVideoDatabase.getCollection("comments", Comment.class);`.
4.  Implement methods:
    *   `public Comment save(Comment comment)`: Uses `commentCollection.insertOne(comment)`. Ensure `commentId` is set (e.g., UUID). Return the input `comment`.
    *   `public FindIterable<Comment> findByVideoId(String videoId, int limit)`:
        *   Uses `commentCollection.find(Filters.eq("video_id", videoId), new FindOptions().sort("timestamp", -1).limit(limit))`. Sort by `timestamp` descending.
    *   `public Optional<Comment> findById(String commentId)`: Uses `commentCollection.findById(commentId)` and wraps with `Optional.ofNullable()`.
    *   `public void delete(String commentId)`: Uses `commentCollection.deleteOne(Filters.eq("comment_id", commentId))`.

File: `src/main/java/com/killrvideo/dao/CommentDao.java`
```java
package com.killrvideo.dao;

import com.datastax.astra.client.Collection;
import com.datastax.astra.client.Database;
import com.datastax.astra.client.model.Filters;
import com.datastax.astra.client.model.FindIterable;
import com.datastax.astra.client.model.FindOptions;
import com.killrvideo.dto.Comment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public class CommentDao {

    private final Collection<Comment> commentCollection;

    @Autowired
    public CommentDao(Database killrVideoDatabase) {
        this.commentCollection = killrVideoDatabase.getCollection("comments", Comment.class);
    }

    public Comment save(Comment comment) {
        // Ensure commentId is populated (e.g., UUID) before saving.
        // This is typically done in the service or controller.
        commentCollection.insertOne(comment);
        return comment;
    }

    /**
     * Finds comments for a given video ID, sorted by most recent.
     * @param videoId The ID of the video.
     * @param limit Max number of comments to return.
     * @return FindIterable of comments.
     */
    public FindIterable<Comment> findByVideoId(String videoId, int limit) {
        // Assuming "video_id" and "timestamp" are correct field names.
        return commentCollection.find(
                Filters.eq("video_id", videoId),
                new FindOptions().sort("timestamp", -1).limit(limit)
        );
    }

    public Optional<Comment> findById(String commentId) {
        return Optional.ofNullable(commentCollection.findById(commentId));
    }

    /**
     * Deletes a comment by its ID.
     * @param commentId The ID of the comment to delete.
     */
    public void delete(String commentId) {
        // Ensure commentId is not null.
        if (commentId == null) {
            throw new IllegalArgumentException("Comment ID cannot be null for delete.");
        }
        commentCollection.deleteOne(Filters.eq("comment_id", commentId));
    }
}
```
```

---

### Prompt 4.2: Create Comment DTOs
```text
In `com.killrvideo.dto` (or `com.killrvideo.dto.comment`), create DTOs:

1.  **`SubmitCommentRequest.java`**:
    Fields: `videoId (String)`, `commentText (String)`.
    Add `@NotBlank` for both. `videoId` should be a valid UUID or identifier format.

The `Comment.java` POJO can be used for responses. If specific fields need to be different for responses, a `CommentResponse.java` can be created.

File: `src/main/java/com/killrvideo/dto/SubmitCommentRequest.java`
```java
package com.killrvideo.dto; // or com.killrvideo.dto.comment

import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Data;

@Data
public class SubmitCommentRequest {
    @NotBlank
    // Potentially add validation for videoId format (e.g., @Pattern for UUID)
    private String videoId;

    @NotBlank
    @Size(min = 1, max = 1000)
    private String commentText;
}
```
```

---

### Prompt 4.3: Create `CommentController`
```text
In `com.killrvideo.controller`, create `CommentController.java`.

1.  Annotate with `@RestController` and `@RequestMapping("/comments")`.
2.  Inject `CommentDao` and `VideoDao` (to check if video exists).
3.  **`POST /` endpoint (Submit Comment)**:
    *   Takes `@Valid @RequestBody SubmitCommentRequest submitCommentRequest` and `Authentication authentication`.
    *   Protected with `@PreAuthorize("isAuthenticated()")`.
    *   Get `userId` from `UserDetailsImpl`.
    *   **Validate `videoId`**: Check if `videoDao.findById(submitCommentRequest.getVideoId())` exists. If not, return `ResponseEntity.badRequest().body("Video not found")`.
    *   Create a new `Comment` object:
        *   `commentId = UUID.randomUUID().toString()`.
        *   `videoId` from request.
        *   `userId` from authenticated user.
        *   `comment` text from request.
        *   `timestamp = Instant.now()`.
    *   Save using `commentDao.save(comment)`.
    *   Return `ResponseEntity.status(HttpStatus.CREATED).body(savedComment)`.
4.  **`GET /video/{videoId}` endpoint (Get Comments for Video)**:
    *   Takes `@PathVariable String videoId`, `@RequestParam(defaultValue = "20") int limit`.
    *   Fetch comments using `commentDao.findByVideoId(videoId, limit)`.
    *   Convert `FindIterable<Comment>` to `List<Comment>` (`comments.all()`).
    *   Return `ResponseEntity.ok(commentList)`.
5.  **`DELETE /{commentId}` endpoint (Delete Comment)**:
    *   Takes `@PathVariable String commentId` and `Authentication authentication`.
    *   Protected with `@PreAuthorize("isAuthenticated()")`.
    *   Fetch the comment using `commentDao.findById(commentId)`. If not found, return `ResponseEntity.notFound().build()`.
    *   Verify that the authenticated user is the owner of the comment: `comment.getUserId().equals(userDetails.getId())`.
    *   If not authorized, return `ResponseEntity.status(HttpStatus.FORBIDDEN).body("You are not authorized to delete this comment.")`.
    *   If authorized, call `commentDao.delete(commentId)`.
    *   Return `ResponseEntity.noContent().build()`.

File: `src/main/java/com/killrvideo/controller/CommentController.java`
```java
package com.killrvideo.controller;

import com.killrvideo.dao.CommentDao;
import com.killrvideo.dao.VideoDao;
import com.killrvideo.dto.Comment; // Assuming Comment POJO is used for response
import com.killrvideo.dto.SubmitCommentRequest;
import com.killrvideo.security.UserDetailsImpl;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/comments") // Relative to /api/v1 context path
public class CommentController {

    @Autowired
    private CommentDao commentDao;

    @Autowired
    private VideoDao videoDao; // To verify video existence

    @PostMapping
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<?> submitComment(@Valid @RequestBody SubmitCommentRequest submitCommentRequest, Authentication authentication) {
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        String userId = userDetails.getId();

        if (!videoDao.findById(submitCommentRequest.getVideoId()).isPresent()) {
            return ResponseEntity.badRequest().body("Error: Video with ID " + submitCommentRequest.getVideoId() + " not found.");
        }

        Comment comment = new Comment();
        comment.setCommentId(UUID.randomUUID().toString());
        comment.setVideoId(submitCommentRequest.getVideoId());
        comment.setUserId(userId);
        comment.setComment(submitCommentRequest.getCommentText());
        comment.setTimestamp(Instant.now());
        // Sentiment will be set by async service

        Comment savedComment = commentDao.save(comment);

        // Trigger asynchronous sentiment analysis
        sentimentService.analyzeSentiment(savedComment.getCommentId());

        return ResponseEntity.status(HttpStatus.CREATED).body(savedComment);
    }

    @GetMapping("/video/{videoId}")
    public ResponseEntity<List<Comment>> getCommentsForVideo(
            @PathVariable String videoId,
            @RequestParam(defaultValue = "20") int limit) {
        if (limit <= 0 || limit > 100) { // Basic validation
            limit = 20;
        }
        // Optionally check if video exists first
        if (!videoDao.findById(videoId).isPresent()) {
             return ResponseEntity.notFound().build(); // Or return empty list
        }
        List<Comment> comments = commentDao.findByVideoId(videoId, limit).all();
        return ResponseEntity.ok(comments);
    }

    @DeleteMapping("/{commentId}")
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<?> deleteComment(@PathVariable String commentId, Authentication authentication) {
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        String currentUserId = userDetails.getId();

        return commentDao.findById(commentId).map(comment -> {
            // Check if the current user is the owner of the comment
            // Add admin role check here if needed: || userDetails.getAuthorities().stream().anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"))
            if (!comment.getUserId().equals(currentUserId)) {
                return ResponseEntity.status(HttpStatus.FORBIDDEN).body("Error: You are not authorized to delete this comment.");
            }
            commentDao.delete(commentId);
            return ResponseEntity.noContent().build();
        }).orElse(ResponseEntity.notFound().build());
    }
}
```
```

---
## Phase 5: Advanced Features - Search & Async Processing

**Context:** This phase introduces vector search for related videos, asynchronous processing for video ingestion (simulating external calls), and search by tags.

---

### Prompt 5.1: Implement Vector Search in `VideoDao`
```text
Modify `com.killrvideo.dao.VideoDao.java`.

1.  Add a new method:
    `public FindIterable<Video> findByVector(float[] vector, int limit)`
    *   This method should perform a vector similarity search. The exact syntax depends on the `astra-db-java` SDK version.
    *   The specification suggests: `videoCollection.find(null, new FindOptions().sort("$vector", vector).limit(limit))`
    *   Or it might be a dedicated method like `videoCollection.findVector(vector, limit, options...)`.
    *   For now, implement using the `sort("$vector", vector)` approach. The field name `"$vector"` is a common convention in Astra DB for the vector field used in an ANN index. Ensure the `Video.java` POJO has `float[] vector;` and the collection `videos` is indexed for vector search on this field.
    *   The `Filter` argument can be `null` if no other filters are applied, or you can pass `Filters.empty()` if the SDK requires a non-null filter.

Example addition to `VideoDao.java`:
```java
    // In VideoDao.java

    /**
     * Finds videos based on vector similarity.
     * Assumes the collection is indexed for vector search on a field (e.g., mapped from 'vector' POJO field, often named '$vector' in queries).
     * @param vector The query vector.
     * @param limit Max number of similar videos to return.
     * @return FindIterable of similar videos.
     */
    public FindIterable<Video> findByVector(float[] vector, int limit) {
        if (vector == null) {
            throw new IllegalArgumentException("Query vector cannot be null.");
        }
        // The sort key "$vector" is a placeholder for the actual vector field used by Astra for ANN search.
        // This field must be properly configured and indexed in Astra DB.
        // The POJO has a 'vector' field; ensure it's mapped correctly and indexed.
        return videoCollection.find(
            Filters.empty(), // Or null if allowed and no other filters needed
            new FindOptions().sort("$vector", vector).limit(limit)
        );
    }
```
This method enables finding videos similar to a given vector embedding.
The actual field name for sorting by vector (`$vector`) must match the one defined in the Astra DB collection's vector index.
```

---

### Prompt 5.2: Add "Related Videos" Endpoint to `VideoController`
```text
Modify `com.killrvideo.controller.VideoController.java`.

1.  Inject `VideoProcessingService`.
2.  In the `POST /` endpoint (submit video metadata), after successfully saving the initial video metadata with `videoDao.save(video)`:
    *   Call `videoProcessingService.processVideoSubmission(savedVideo.getVideoId(), savedVideo.getLocation())`.

Updated `submitVideo` method in `VideoController.java`:
```java
    // In VideoController.java
    // ... (other imports)
    import com.killrvideo.service.VideoProcessingService; // Add this import
    // ...

    @Autowired // Add this
    private VideoProcessingService videoProcessingService;

    @PostMapping
    @PreAuthorize("isAuthenticated()")
    public ResponseEntity<Video> submitVideo(@Valid @RequestBody SubmitVideoRequest submitVideoRequest, Authentication authentication) {
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        String userId = userDetails.getId();

        Video video = new Video();
        video.setVideoId(UUID.randomUUID().toString());
        video.setUserId(userId);
        video.setName(submitVideoRequest.getName());
        video.setDescription(submitVideoRequest.getDescription());
        video.setTags(submitVideoRequest.getTags());
        video.setLocation(submitVideoRequest.getLocation());
        video.setAddedDate(Instant.now());
        video.setPreviewImageLocation(null); // Will be set by async service
        video.setVector(null); // Will be set by async service

        Video savedVideo = videoDao.save(video);

        // Trigger asynchronous processing
        videoProcessingService.processVideoSubmission(savedVideo.getVideoId(), savedVideo.getLocation());

        return ResponseEntity.status(HttpStatus.CREATED).body(savedVideo);
    }
```
Now, submitting a video will also kick off the asynchronous processing task.
```

---

### Prompt 5.3: (Conceptual) Webhook for External Processing Updates
```text
Create a conceptual webhook endpoint in a new `WebhookController.java` or within `VideoController.java`.
This is for when an *actual* external service (not simulated) completes processing and calls back.

1.  Endpoint: `POST /webhooks/video-processed` (or a more specific path).
2.  It would take a DTO representing the callback payload (e.g., `VideoProcessingResultDto` with `videoId`, `status`, `newPreviewUrl`, `generatedVectorData`).
3.  The handler would find the video by `videoId`, update its fields (status, preview URL, vector), and save it using `videoDao.update()`.
4.  This endpoint might need to be secured (e.g., with a shared secret API key in the header).

**For this LLM generation task, create a simple placeholder for this endpoint without complex security or DTO, just to illustrate the structure. Assume it's called by the (simulated) async process for now, though typically it would be external.**

File: `src/main/java/com/killrvideo/controller/WebhookController.java` (New file)
```java
package com.killrvideo.controller;

import com.killrvideo.dao.VideoDao;
import com.killrvideo.dto.Video;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map; // For simple payload
import java.util.Optional;

@RestController
@RequestMapping("/webhooks") // Relative to /api/v1
public class WebhookController {

    private static final Logger logger = LoggerFactory.getLogger(WebhookController.class);

    @Autowired
    private VideoDao videoDao;

    // This is a simplified conceptual DTO for the webhook payload
    // In a real scenario, this would be a well-defined class.
    public static class VideoProcessingWebhookPayload {
        public String videoId;
        public String status; // e.g., "COMPLETED", "FAILED"
        public String previewImageUrl;
        public float[] vector; // Or path to vector data
        // any other relevant data from processing
    }

    @PostMapping("/video-processed")
    public ResponseEntity<String> handleVideoProcessedWebhook(@RequestBody VideoProcessingWebhookPayload payload) {
        logger.info("Received video processing webhook for videoId: {}", payload.videoId);

        if (payload.videoId == null) {
            return ResponseEntity.badRequest().body("Error: videoId is missing in webhook payload.");
        }

        Optional<Video> sourceVideoOpt = videoDao.findById(payload.videoId);
        if (!sourceVideoOpt.isPresent() || sourceVideoOpt.get().getVector() == null || sourceVideoOpt.get().getVector().length == 0) {
            // If source video not found, or has no vector, return empty list or appropriate response
            return ResponseEntity.ok(Collections.emptyList());
        }

        Video sourceVideo = sourceVideoOpt.get();
        List<Video> relatedVideos = videoDao.findByVector(sourceVideo.getVector(), limit + 1) // Fetch one extra in case source is included
                                         .all()
                                         .stream()
                                         .filter(video -> !video.getVideoId().equals(sourceVideo.getVideoId())) // Exclude source video itself
                                         .limit(limit) // Ensure limit is respected after filtering
                                         .collect(Collectors.toList());

        return ResponseEntity.ok(relatedVideos);
    }
}
```
This endpoint would be called by an external service upon completing video processing.
For the current simulation, the `VideoProcessingService` updates the DB directly. This webhook is for a more realistic scenario.
Remember to permit `/api/v1/webhooks/**` in `WebSecurityConfig` if it's intended to be public or secure it appropriately.
```text
// Add to WebSecurityConfig permitAll list:
.requestMatchers("/api/v1/webhooks/**").permitAll()
```

---

### Prompt 5.4: Implement Search Videos by Tag in `VideoDao` and `VideoController`
```text
1.  **Modify `VideoDao.java`**:
    *   Add method `public FindIterable<Video> findByTag(String tag, int limit)`:
        *   Uses `videoCollection.find(Filters.eq("tags", tag), new FindOptions().sort("added_date", -1).limit(limit))`.
        *   This assumes `tags` is a `Set<String>` in the POJO and stored as an array/list in Astra DB, allowing direct equality match on an element.

2.  **Modify `VideoController.java`**:
    *   Add endpoint `GET /tag/{tag}`:
        *   Takes `@PathVariable String tag`, `@RequestParam(defaultValue = "10") int limit`.
        *   Calls `videoDao.findByTag(tag, limit)`.
        *   Convert `FindIterable` to `List` and return.

Example addition to `VideoDao.java`:
```java
    // In VideoDao.java

    /**
     * Finds videos by a specific tag.
     * Assumes 'tags' field in Astra is an array/list where direct equality match can find if the tag exists.
     * @param tag The tag to search for.
     * @param limit Max number of videos to return.
     * @return FindIterable of videos.
     */
    public FindIterable<Video> findByTag(String tag, int limit) {
        if (tag == null || tag.trim().isEmpty()) {
            // Return empty iterable or throw exception for invalid tag
            return videoCollection.find(Filters.eq("tags", "NON_EXISTENT_TAG_SO_EMPTY_RESULT"), new FindOptions().limit(0));
        }
        // This query assumes the 'tags' field is an array and the DB supports element matching.
        // For Astra Data API, if 'tags' is a Set<String> / List<String> in POJO,
        // Filters.eq("tags", tag) might work if the underlying engine treats it as "array contains".
        // Otherwise, a different filter like Filters.in("tags", tag) or specific array operator might be needed.
        // Let's proceed with Filters.eq as per spec's conceptual example.
        return videoCollection.find(
            Filters.eq("tags", tag), // Matches if the 'tags' array contains the exact tag string
            new FindOptions().sort("added_date", -1).limit(limit)
        );
    }
```

Example addition to `VideoController.java`:
```java
    // In VideoController.java

    @GetMapping("/tag/{tag}")
    public ResponseEntity<List<Video>> getVideosByTag(
            @PathVariable String tag,
            @RequestParam(defaultValue = "10") int limit) {

        if (limit <= 0 || limit > 50) {
            limit = 10;
        }
        if (tag == null || tag.trim().isEmpty()) {
            return ResponseEntity.badRequest().body(Collections.emptyList()); // Or some error message
        }

        List<Video> videos = videoDao.findByTag(tag.trim(), limit).all();
        return ResponseEntity.ok(videos);
    }
```
```

---

### Prompt 5.5: (Conceptual) Async Sentiment Analysis for Comments
```text
1.  **Create `SentimentService.java` in `com.killrvideo.service`**:
    *   Annotate with `@Service`.
    *   Inject `CommentDao`.
    *   Create an `@Async("taskExecutor")` method:
        *   `public void analyzeSentiment(String commentId)`:
        *   Log start.
        *   Simulate calling an external sentiment analysis service: `Thread.sleep(1000);`. Generate a dummy sentiment (e.g., "POSITIVE", "NEGATIVE", "NEUTRAL", or a score).
        *   Fetch the `Comment` by `commentId`.
        *   If found, add a new field to `Comment.java` POJO like `private String sentiment;` (or `private double sentimentScore;`).
        *   Update the comment object with the dummy sentiment.
        *   Save using `commentDao.update(comment)` (this method needs to be added to `CommentDao`, similar to `VideoDao.update`, using `replaceOne`).
        *   Log completion.

2.  **Modify `Comment.java` POJO**: Add `private String sentiment;` with `@JsonProperty("sentiment")` (optional).

3.  **Modify `CommentDao.java`: Add `public void update(Comment comment)` method.
    ```java
    // In CommentDao.java
    public void update(Comment comment) {
        if (comment.getCommentId() == null) {
            throw new IllegalArgumentException("Comment ID cannot be null for update.");
        }
        commentCollection.replaceOne(Filters.eq("comment_id", comment.getCommentId()), comment);
    }
    ```

4.  **Modify `CommentController.java`:
    *   Inject `SentimentService`.
    *   In the `POST /` endpoint (submit comment), after successfully saving the comment, call `sentimentService.analyzeSentiment(savedComment.getCommentId())`.

This is a conceptual outline. The implementation for the LLM should include the POJO change, DAO update method, the new service, and the controller modification.

File: `src/main/java/com/killrvideo/dto/Comment.java` (Add sentiment field)
```diff
package com.killrvideo.dto;

import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.Instant;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Comment {
    @JsonProperty("comment_id")
    private String commentId;
    @JsonProperty("video_id")
    private String videoId;
    @JsonProperty("user_id")
    private String userId;
    private String comment;
    private Instant timestamp;
    @JsonProperty("sentiment") // Optional, if different from field name
    private String sentiment; // e.g., "POSITIVE", "NEGATIVE", "NEUTRAL"
}
```

File: `src/main/java/com/killrvideo/service/SentimentService.java` (New file)
```java
package com.killrvideo.service;

import com.killrvideo.dao.CommentDao;
import com.killrvideo.dto.Comment;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.Optional;
import java.util.Random;

@Service
public class SentimentService {

    private static final Logger logger = LoggerFactory.getLogger(SentimentService.class);
    private static final Random random = new Random();

    @Autowired
    private CommentDao commentDao;

    @Async("taskExecutor")
    public void analyzeSentiment(String commentId) {
        logger.info("Starting sentiment analysis for comment ID: {}", commentId);
        try {
            // Simulate calling external sentiment analysis service
            Thread.sleep(1000); // 1-second delay
            String dummySentiment = SENTIMENTS[random.nextInt(SENTIMENTS.length)];

            Optional<Comment> commentOptional = commentDao.findById(commentId);
            if (commentOptional.isPresent()) {
                Comment comment = commentOptional.get();
                comment.setSentiment(dummySentiment);
                commentDao.update(comment); // Assumes update method exists in CommentDao
                logger.info("Sentiment analysis complete for comment ID: {}. Sentiment: {}", commentId, dummySentiment);
            } else {
                logger.warn("Comment ID: {} not found for sentiment update.", commentId);
            }
        } catch (InterruptedException e) {
            logger.error("Sentiment analysis for comment {} was interrupted.", commentId, e);
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            logger.error("Error during sentiment analysis for comment {}: {}", commentId, e.getMessage(), e);
        }
    }
}
```
```

---
## Phase 6: API Polish & Production Readiness

**Context:** This phase focuses on improving robustness, maintainability, and observability of the API.

---

### Prompt 6.1: Global Exception Handling
```text
Create a package `com.killrvideo.exception`.

1.  **Custom Exceptions**:
    *   `ResourceNotFoundException.java` extending `RuntimeException`. (Constructor takes a message).
    *   Optionally, `BadRequestException.java`, `UnauthorizedException.java` if more specific handling is needed beyond Spring Security's.

2.  **`GlobalExceptionHandler.java` (or `RestExceptionHandler.java`)**:
    *   Annotate with `@ControllerAdvice`.
    *   Method for `ResourceNotFoundException`: `@ExceptionHandler(ResourceNotFoundException.class)` returns `ResponseEntity<ErrorResponseDto>` with HTTP 404.
    *   Method for `MethodArgumentNotValidException` (from `@Valid`): returns `ResponseEntity<ErrorResponseDto>` with HTTP 400, extracting field errors.
    *   Generic fallback `ExceptionHandler(Exception.class)`: returns `ResponseEntity<ErrorResponseDto>` with HTTP 500.
    *   Create a simple `ErrorResponseDto.java` (fields: `timestamp`, `status`, `error`, `message`, `path`, optional `List<String> details` for validation errors).

File: `src/main/java/com/killrvideo/exception/ResourceNotFoundException.java`
```java
package com.killrvideo.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s not found with %s : '%s'", resourceName, fieldName, fieldValue));
    }
}
```

File: `src/main/java/com/killrvideo/dto/ErrorResponseDto.java` (Can be in `com.killrvideo.dto` or `com.killrvideo.exception`)
```java
package com.killrvideo.dto; // Or com.killrvideo.exception

import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ErrorResponseDto {
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd hh:mm:ss")
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private List<String> details; // For validation errors

    public ErrorResponseDto(LocalDateTime timestamp, int status, String error, String message, String path) {
        this.timestamp = timestamp;
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }
}
```

File: `src/main/java/com/killrvideo/exception/GlobalExceptionHandler.java`
```java
package com.killrvideo.exception;

import com.killrvideo.dto.ErrorResponseDto;
import jakarta.servlet.http.HttpServletRequest;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;

@ControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponseDto> handleResourceNotFoundException(ResourceNotFoundException ex, HttpServletRequest request) {
        logger.warn("Resource not found: {} (Path: {})", ex.getMessage(), request.getRequestURI());
        ErrorResponseDto errorResponse = new ErrorResponseDto(
                LocalDateTime.now(),
                HttpStatus.NOT_FOUND.value(),
                HttpStatus.NOT_FOUND.getReasonPhrase(),
                ex.getMessage(),
                request.getRequestURI()
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponseDto> handleValidationExceptions(MethodArgumentNotValidException ex, HttpServletRequest request) {
        logger.warn("Validation error: {} (Path: {})", ex.getMessage(), request.getRequestURI());
        List<String> details = ex.getBindingResult()
                                 .getFieldErrors()
                                 .stream()
                                 .map(error -> error.getField() + ": " + error.getDefaultMessage())
                                 .collect(Collectors.toList());

        ErrorResponseDto errorResponse = new ErrorResponseDto(
                LocalDateTime.now(),
                HttpStatus.BAD_REQUEST.value(),
                "Validation Failed",
                "Input validation failed. See details.",
                request.getRequestURI(),
                details
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }

    // Handle cases like bad request from invalid UUID format for @PathVariable
    @ExceptionHandler(org.springframework.web.method.annotation.MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponseDto> handleMethodArgumentTypeMismatch(org.springframework.web.method.annotation.MethodArgumentTypeMismatchException ex, HttpServletRequest request) {
        logger.warn("Method argument type mismatch: {} (Path: {})", ex.getMessage(), request.getRequestURI());
        String message = String.format("The parameter '%s' of value '%s' could not be converted to type '%s'",
                                       ex.getName(), ex.getValue(), ex.getRequiredType().getSimpleName());
        ErrorResponseDto errorResponse = new ErrorResponseDto(
                LocalDateTime.now(),
                HttpStatus.BAD_REQUEST.value(),
                HttpStatus.BAD_REQUEST.getReasonPhrase(),
                message,
                request.getRequestURI()
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.BAD_REQUEST);
    }


    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponseDto> handleGlobalException(Exception ex, HttpServletRequest request) {
        logger.error("Unhandled exception: {} (Path: {})", ex.getMessage(), request.getRequestURI(), ex); // Log stack trace for unexpected
        ErrorResponseDto errorResponse = new ErrorResponseDto(
                LocalDateTime.now(),
                HttpStatus.INTERNAL_SERVER_ERROR.value(),
                HttpStatus.INTERNAL_SERVER_ERROR.getReasonPhrase(),
                "An unexpected error occurred. Please try again later.", // Generic message for users
                request.getRequestURI()
        );
        return new ResponseEntity<>(errorResponse, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```
Controllers should be updated to throw `ResourceNotFoundException` where appropriate (e.g., `videoDao.findById(videoId).orElseThrow(() -> new ResourceNotFoundException("Video", "id", videoId))`).
```

---

### Prompt 6.2: Integrate SpringDoc for OpenAPI Documentation
```text
1.  **Add Dependency to `pom.xml`**:
    `org.springdoc:springdoc-openapi-starter-webmvc-ui` (use latest version, e.g., `2.5.0`).
    ```xml
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.5.0</version> <!-- Check for latest stable version -->
    </dependency>
    ```

2.  **Configure in `application.yml`**:
    ```yaml
    springdoc:
      api-docs:
        path: /openapi # Default is /v3/api-docs
      swagger-ui:
        path: /swagger-ui.html # Default path
      default-consumes-media-type: application/json
      default-produces-media-type: application/json
      # Optional: Grouping, Info, Security Scheme configuration
      # info:
      #   title: KillrVideo API
      #   version: v1
      #   description: API for KillrVideo application
    ```

3.  **Annotate Controllers and DTOs**:
    *   Use `@Operation(summary = "...")`, `@Parameter(description = "...")`, `@ApiResponse(...)` on controller methods.
    *   Use `@Schema(description = "...")` on DTOs and their fields.
    *   Ensure `WebSecurityConfig` permits access to `/openapi/**` and `/swagger-ui/**`, `/swagger-ui.html`, and `/v3/api-docs/**`. (Already added in Prompt 2.8).

This setup will automatically generate OpenAPI 3 documentation and provide a Swagger UI. No specific code generation for annotations is requested in this prompt, just setup. The LLM should have already incorporated this in DTO and Controller generation.
```

---

### Prompt 6.3: Configure Actuator Endpoints
```text
The `spring-boot-starter-actuator` dependency is already added.

1.  **Expose Endpoints in `application.yml`**:
    By default, only `/health` and `/info` might be exposed over HTTP. Expose more if needed (e.g., `metrics`, `env`, `loggers`).
    ```yaml
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics,env # Add more as needed, e.g. prometheus, loggers
      endpoint:
        health:
          show-details: when_authorized # or 'always' for more details, 'never'
          # Probes enabled for Kubernetes by default with Spring Boot 2.3+
          probes:
            enabled: true
    ```
This provides useful operational endpoints. `/api/v1/actuator/*` would be the path due to context-path, unless management port is different.
The `WebSecurityConfig` should be checked to ensure actuator endpoints are appropriately secured or permitted. Typically, they might be on a separate management port or secured differently. For simplicity now, if they are under `/api/v1/actuator`, they'd be caught by `anyRequest().authenticated()`.
If a separate management port is not used, ensure `/api/v1/actuator/**` is handled by security config if specific access is needed.
```

---

### Prompt 6.4: Implement Soft Deletes (Conceptual for Video)
```text
This is a conceptual change, primarily for DAOs and POJOs, focusing on the Video entity as an example.

1.  **Modify `Video.java` POJO**:
    *   Add `private boolean deleted = false;` with `@JsonProperty("deleted")`.

2.  **Modify `VideoDao.java`**:
    *   **`save` and `update`**: No change needed as they operate on the whole document.
    *   **`findById`**: Modify to `videoCollection.findOne(Filters.and(Filters.eq("video_id", videoId), Filters.eq("deleted", false)))`.
    *   **`findLatest`, `findByUserId`, `findByTag`, `findByVector`**: Add `Filters.eq("deleted", false)` to the `Filters` criteria (using `Filters.and(...)` if other filters exist).
    *   Add `public void softDelete(String videoId)`: Fetches the video, sets `deleted = true`, then calls `update(video)`.
    *   Add `public void restore(String videoId)`: Fetches the video (even if deleted, so a new find method ignoring `deleted` flag might be needed for admin), sets `deleted = false`, then calls `update(video)`.

This makes deletions non-destructive. Apply similar logic to other DAOs/POJOs (`User`, `Comment`) if soft delete is required for them.

Example change for `findById` in `VideoDao.java`:
```java
    // In VideoDao.java
    // (after adding 'deleted' field to Video.java)
    public Optional<Video> findById(String videoId) {
        // Now filters out soft-deleted videos
        return videoCollection.findOne(
            Filters.and(
                Filters.eq("video_id", videoId),
                Filters.eq("deleted", false) // Or Filters.ne("deleted", true)
            )
        );
    }

    // Example for findLatest
    public FindIterable<Video> findLatest(int limit) {
        return videoCollection.find(
            Filters.eq("deleted", false), // Filter for non-deleted videos
            new FindOptions().sort("added_date", -1).limit(limit)
        );
    }
    // Similar adjustments for other find methods.

    public void softDelete(String videoId) {
        Optional<Video> videoOpt = findById(videoId); // This findById already checks for not deleted
        if (videoOpt.isPresent()) {
            Video video = videoOpt.get();
            video.setDeleted(true);
            // video.setDeletedAt(Instant.now()); // Optional: add a deletedAt timestamp
            update(video);
        } else {
            // Optionally throw ResourceNotFound or log
            logger.warn("Video not found or already deleted (soft): {}", videoId);
        }
    }
```
Add `deleted` field to `Video.java` (as `private boolean deleted = false;`).
The find methods need careful adjustment to include `Filters.eq("deleted", false)`. For `findByVector`, it would be `Filters.and(Filters.eq("deleted", false), existingFilterOrNull)`.
The `ResourceNotFoundException` can be used more extensively in DAOs or services.
```

---

### Prompt 6.5: Input Validation
```text
The `spring-boot-starter-validation` dependency and basic `@Valid`, `@NotBlank`, `@Size`, `@Email` annotations are already used in DTOs and Controller method parameters (e.g., `SignupRequest`, `LoginRequest`, `SubmitVideoRequest`, `SubmitCommentRequest`).

This step is a reminder to:
1.  Ensure `@Valid` is present on `@RequestBody` parameters in all controllers where DTOs have validation annotations.
2.  Review all DTOs and add appropriate validation annotations (e.g., `@NotNull`, `@Min`, `@Max`, `@Pattern`) to all fields where constraints apply.
3.  The `GlobalExceptionHandler` already handles `MethodArgumentNotValidException`.

No new code is generated by this prompt directly, but it emphasizes a best practice to be applied throughout the codebase. The LLM should have already incorporated this in DTO and Controller generation.
```

---

This concludes the initial set of prompts. Further enhancements like detailed testing, specific external service integrations, more complex query patterns, or advanced security measures can be built upon this foundation.

</rewritten_file> 