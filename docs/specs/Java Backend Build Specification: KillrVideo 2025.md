
**Java Backend Build Specification: KillrVideo 2025 (Revised for Astra DB Data API)**

**1. Core Technologies & Versions:**

*   **Java Version:** Java 21 LTS.
*   **Framework:** Spring Boot 3.3.x (or latest stable 3.x version).
*   **Database Interaction:** Astra DB Data API via `astra-sdk-java`.
*   **Target Database:** Astra DB (Serverless).
*   **Build Tool:** Maven 3.9.x.

**2. Project Setup & Dependencies (Maven `pom.xml`):**

*   **Spring Boot Starter Parent:**
    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version> <!-- Or latest stable 3.3.x -->
        <relativePath/>
    </parent>
    ```
*   **Core Spring Boot Starters:**
    *   `spring-boot-starter-web`
    *   `spring-boot-starter-security`
    *   `spring-boot-starter-validation`
    *   `spring-boot-starter-actuator`
    *   `spring-boot-starter-test`
*   **Astra DB Data API Client:**
    *       It appears the most direct client for the Data API is `com.datastax.astra:astra-db-java`.
    Let's use this. We need to confirm the latest version from Maven Central. As of my last update, a version like `1.x.x` or `2.0.0-PREVIEW` might be available. Let's assume `1.2.0` for now as a placeholder for a recent stable version.
    ```xml
    <dependency>
        <groupId>com.datastax.astra</groupId>
        <artifactId>astra-db-java</artifactId>
        <version>1.2.0</version> <!-- Replace with the actual latest stable version -->
    </dependency>
    ```
    *   There's also an `astra-spring-boot-3x-starter` and `astra-spring-boot-3x-autoconfigure` which might simplify setup.
        *   **Decision Point/Clarification Needed:** Should we use the `astra-spring-boot-3x-starter`? This could be beneficial for auto-configuration. The documentation at the provided link (`dataapiclient.html`) shows direct instantiation of `DataAPIClient`. For now, I will proceed with the direct use of `astra-db-java` and manual Spring bean configuration, but the starter is worth investigating for simplification.
*   **Security - JWT:**
    *   `io.jsonwebtoken:jjwt-api`, `io.jsonwebtoken:jjwt-impl`, `io.jsonwebtoken:jjwt-jackson`.
*   **Utilities:**
    *   `org.projectlombok:lombok`.
*   **API Documentation (OpenAPI):**
    *   `org.springdoc:springdoc-openapi-starter-webmvc-ui`.
*   **JSON Processing:**
    *   Jackson (comes with `spring-boot-starter-web`, `astra-db-java` will also use Jackson).
*   **HTTP Client (for YouTube, Embeddings Service):**
    *   Spring Boot's `WebClient`.

**3. Configuration (`application.yml` or `application.properties`):**

*   **Server Configuration:**
    *   `server.port`
    *   `server.servlet.context-path: /api/v1`
*   **Spring Security & JWT:**
    *   `killrvideo.jwt.secret`
    *   `killrvideo.jwt.expiration-ms`
*   **Astra DB Data API Connection (managed via environment variables):**
    *   `ASTRA_DB_API_ENDPOINT`: The API endpoint for your Astra DB (e.g., `https://<database_id>-<region>.apps.astra.datastax.com`).
    *   `ASTRA_DB_APPLICATION_TOKEN`: The application token for authentication.
    *   `ASTRA_DB_NAMESPACE` (or `KEYSPACE`): The default namespace/keyspace to use.
*   **External Services URLs:**
    *   `killrvideo.external.youtube-api.url`
    *   `killrvideo.external.embeddings-service.url`
    *   `killrvideo.external.sentiment-service.url`
*   **Async Processing:**
    *   `spring.task.execution.*`
*   **Springdoc OpenAPI:**
    *   `springdoc.api-docs.path: /openapi`
    *   `springdoc.swagger-ui.path: /swagger-ui.html`

**4. Project Structure (Maven Standard Layout):**

```
killrvideo-java/
├── pom.xml
└── src/
    ├── main/
    │   ├── java/
    │   │   └── com/killrvideo/
    │   │       ├── KillrVideoApplication.java
    │   │       ├── config/                     // Spring configurations (Security, AstraDBClientConfig, Async, OpenAPI)
    │   │       ├── controller/                 // REST API Controllers
    │   │       ├── dto/                        // Data Transfer Objects & domain objects (POJOs for collections)
    │   │       ├── enums/                      // Application-specific enumerations
    │   │       ├── exception/                  // Custom exceptions and global exception handler
    │   │       ├── security/                   // JWT utilities, UserDetailsService implementation
    │   │       ├── service/                    // Business logic services
    │   │       │   └── impl/
    │   │       ├── dao/                        // Data Access Objects using AstraDB Data API Client
    │   │       └── utils/
    │   └── resources/
    │       ├── application.yml
    │       └── ...
    └── test/
        └── java/
            └── com/killrvideo/
                └── // ...
```
*   **Note the change:** No `entity/` or `repository/` in the Spring Data Cassandra sense. POJOs will exist in `dto/` or a dedicated `model/` or `domain/` package representing the documents in collections. Data access logic is now in `dao/` (Data Access Objects) or directly within services if simple.

**5. Key Implementation Details & Considerations:**

*   **Astra DB Data API Client Configuration:**
    *   Create a Spring configuration class (e.g., `AstraDBConfig.java`) to instantiate and provide the `DataAPIClient` (from `astra-db-java`) as a Spring bean.
        ```java
        // In AstraDBConfig.java
        @Configuration
        public class AstraDBConfig {
            @Value("${ASTRA_DB_API_ENDPOINT}")
            private String apiEndpoint;

            @Value("${ASTRA_DB_APPLICATION_TOKEN}")
            private String applicationToken;

            @Bean
            public DataAPIClient dataAPIClient() {
                return new DataAPIClient(applicationToken); // Simpler constructor if token contains endpoint info or if using default Astra endpoint
                                                            // Or: new DataAPIClient(apiEndpoint, applicationToken); - check SDK specifics
            }

            @Bean
            public Database killrVideoDatabase(DataAPIClient dataAPIClient, @Value("${ASTRA_DB_NAMESPACE}") String namespace) {
                // The SDK might require specifying the database ID in the endpoint or via another method.
                // The following is a conceptual way to get a Database object.
                // Refer to the latest astra-db-java documentation for exact usage.
                // E.g. dataAPIClient.getDatabase(apiEndpoint, namespace); or similar.
                // Let's assume for now the client is general and Database object is obtained with endpoint + namespace
                return dataAPIClient.getDatabase(apiEndpoint, namespace);
            }
        }
        ```
    *   DAOs or services will then `@Autowire` the `Database` bean (or `DataAPIClient`) to interact with collections.
*   **Data Model & DAO Layer:**
    *   Define POJOs (Plain Old Java Objects) to represent the structure of your documents in Astra DB collections (e.g., `User.java`, `Video.java`, `Comment.java`). These POJOs will be serialized/deserialized to/from JSON by Jackson, which the `astra-db-java` client uses.
    *   Create DAO classes (e.g., `UserDao.java`, `VideoDao.java`) that encapsulate the logic for interacting with Astra DB collections using the `Database` object from the `astra-db-java` SDK.
    *   Example `VideoDao` (conceptual):
        ```java
        @Repository // Or @Component
        public class VideoDao {
            private final Collection<Video> videoCollection; // Video is your POJO

            @Autowired
            public VideoDao(Database killrVideoDatabase) {
                // Assuming Video.class is your POJO for the video documents
                this.videoCollection = killrVideoDatabase.getCollection("videos", Video.class);
            }

            public Video findById(String videoId) {
                return videoCollection.findById(videoId);
            }

            public Video save(Video video) {
                // The SDK might return the saved document or an ID, adjust accordingly
                videoCollection.insertOne(video);
                return video;
            }

            // Methods for findLatest, findByTag, findByUser, search (including vector search)
            public FindIterable<Video> findLatest(int limit, int skip) {
                // Example: Assuming sort options are available
                return videoCollection.find(new FindOptions().sort("uploadedAt", -1).limit(limit).skip(skip));
            }

            public FindIterable<Video> findByTag(String tag, int limit, int skip) {
                // Using a filter expression. Syntax depends on the SDK.
                // E.g. Filters.eq("tags", tag)
                return videoCollection.find(Filters.eq("tags", tag), new FindOptions().limit(limit).skip(skip));
            }

            public FindIterable<Video> findByVector(float[] vector, int limit) {
                // Example of a vector search call. Syntax is SDK-specific.
                return videoCollection.find(
                    null, // No filter, or additional filters
                    new FindOptions().sort("$vector", vector).limit(limit) // Or specific vectorSearch method
                );
                // Or: videoCollection.findVector(vector, limit, options...);
            }
            // ... other CRUD and query methods
        }
        ```
*   **Vector Fields in POJOs:** Your `Video.java` POJO will need a field to hold the vector embeddings, e.g., `private float[] vector;`. Ensure Jackson can serialize/deserialize this correctly with the Data API.
*   **Querying:**
    *   Utilize the methods provided by the `astra-db-java` client's `Collection` interface: `insertOne`, `insertMany`, `findById`, `find`, `updateOne`, `updateMany`, `deleteOne`, `deleteMany`, etc.
    *   Use `Filter` objects or JSON-based query documents for filtering, as supported by the SDK.
    *   **Vector Search:** The `astra-db-java` SDK provides methods to perform vector similarity searches (e.g., `find().sort("$vector", vectorEmbedding)` or a dedicated `findVector()` method). This will be crucial for "Related Videos" and "For You" recommendations.
*   **Asynchronous Video Processing & Webhooks:** The logic remains similar:
    *   `POST /videos` initiates.
    *   `@Async` service calls external YouTube/Embeddings services.
    *   **Webhook Reception:** A controller endpoint (`POST /webhooks/video-processed`) will receive notifications from the external processing services. This endpoint's handler will use the appropriate DAO to update the video document in Astra DB.
*   **Sentiment Analysis:** Similar to async processing – call external service, then use DAO to store comment with sentiment.
*   **Soft Deletes:** Implement using a boolean flag (e.g., `isDeleted: true`) in your POJOs/documents. DAO methods will need to filter based on this flag for regular queries and provide methods for moderators to see/restore.
*   **Authentication, Authorization, Error Handling, NFRs, Testing, Build/Deployment:** These sections remain largely the same as in the previous spec, with the main difference being the data access layer tests will now mock/use the `astra-db-java` client instead of Spring Data Cassandra. Integration tests could use a real (dev) Astra DB instance via the Data API.

**6. Key Questions Revisited with Data API Context:**

1.  **Astra Spring Boot Starter:** Is `com.datastax.astra:astra-spring-boot-3x-starter` preferred for auto-configuration of the `DataAPIClient` / `Database` beans? (This could simplify `AstraDBConfig.java`).
2.  **Vector Embedding Storage & Querying:** Confirm the exact field name (e.g., `$vector` or user-defined) and method (`sort` option vs. dedicated `findVector` method) for storing and querying vector embeddings with the chosen `astra-db-java` version.
3.  **Pagination with Data API:** How is pagination (skip/limit, total count) best handled with `find` operations in `astra-db-java`? The API spec requires total counts for paginated responses. The `FindIterable` might need to be processed to get a list and then count, or the API might offer a way to get counts separately.
4.  **Complex Queries/Transactions:**
    *   How are more complex queries (e.g., multiple `OR` conditions, aggregations if needed beyond simple counts) constructed?
    *   The Data API is generally document-oriented and might not have multi-document transaction support in the traditional RDBMS sense. Operations are typically atomic at the document level. This should be fine for KillrVideo's needs.

This revised build specification aligns with the use of Astra DB's Data API, offering a more modern, HTTP-centric approach to database interaction. It emphasizes POJO-based document modeling and direct use of the `astra-db-java` SDK.