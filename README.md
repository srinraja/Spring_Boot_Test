# Spring_Boot_Test
To manage review comments

Review comments:
@Transactional is added because the method writes to the database via bookRepository.save(book).
It ensures:
1. Atomicity If anything fails mid-way (e.g. DB constraint violation during save()), the entire operation is rolled back automatically — no partial/corrupt data is persisted.
2. Data Consistency Without @Transactional, the save() runs in auto-commit mode — meaning every DB operation commits immediately with no rollback safety net.
3. Exception Safety If a RuntimeException is thrown after save() (unlikely here but possible in more complex flows), the transaction rolls back ensuring nothing is persisted incorrectly.

What is @Component annotation ?
The @Component annotation in Spring is a class-level stereotype annotation used to mark a Java class as a Spring-managed component (or bean). This allows the Spring IoC (Inversion of Control) container to automatically detect, create an instance of, and manage the lifecycle of that class during application startup, provided that component scanning is enabled.

Automatic Detection: The annotated class is automatically detected by Spring's classpath scanning mechanism when the @ComponentScan annotation is used (which is included by default in Spring Boot's @SpringBootApplication).

Dependency Injection: Once a class is registered as a bean in the Spring container, it becomes eligible for dependency injection into other components using mechanisms like @Autowired.

Default Bean Name: By default, the bean name will be the class name with the first letter lowercased (e.g., a class named UserService would have a bean name of userService). A custom name can be specified within the annotation, such as @Component("myCustomName")

Single Responsibility Principle:
Fetch
Validate
Map
Save

Fail Fast principle:
Detect and reject invalid conditions as early as possible — before doing any expensive operations like external API calls. 


Unit tests 
verify your logic is correct in isolation — fast, focused, no dependencies.

Integration tests 
verify your layers work together correctly — slower, real context, catches wiring issues.

Refactored UT, IT test cases separately with Happy path, Edge cases


Annotations learned
@Component

@Builder - implements the Builder design pattern automatically — generating a static builder() method, setter-style chained methods for each field, and a final .build() method that constructs the object. Avoids boilerplate and makes object creation readable.

@Service
@Controller
@ Repository

@Transactional

@SpringBootTest
@Mock
@BeforeAll
@AfterAll
@BeforeEach

@Autowired

Concepts:

1. What is @SpringBootApplication?
     Combination of 3 annotations,
@SpringBootConfiguration - marks the configuration class 
@ EnableAutoConfiguration - enables spring boot auto-config
 @ ComponentScan - scans the current package for beans


2. What is dependency injection and why constructor injection is preferred?

  Lombok is a library that plugs into your build process. When you compile your code, Lombok sees that annotation and automatically generates a constructor for you that includes all fields marked as final.

@RequiredArgsConstructor -  Auto generates the constructor during compilation

Lombok generates the constructor for your final fields during compilation.
Spring detects this single constructor when starting the application.
Spring looks for beans of type BookService and GoogleBookService and "injects" them through that generated constructor.

BookController → handles HTTP only (no business logic)
BookService → orchestrates the flow only
BookMapper → mapping logic only
GoogleBookService → Google API communication only
BookRepository → DB access only
GlobalExceptionHandler → error handling only
  


Three-layer error handling strategy:
1. Fail Fast validation (BookService)  
└── null/blank googleId → 400
└── duplicate ID → 409

2. Specific exception catching (fetchBookFromGoogle)
 └── 404 from Google → GoogleBookRetrievalException → 400  
└── other HTTP errors → preserve upstream status code
└── unexpected errors → 500

3. Centralized handling (GlobalExceptionHandler)  
└── @RestControllerAdvice catches domain exceptions
└── Returns consistent JSON error payload



To follow best practices, the codebase was restructured into the following distinct layers:
Controller: Handles HTTP routing and requests (
BookController)
Service: Orchestrates business logic and transactional boundaries (BookService)
Repository: Manages local data persistence (BookRepository)
Model: Contains domain entity definitions (Book)
Client: Houses external API consumers (GoogleBookService)


Our Project (Local): We expose the endpoint GET /google. This is our "internal" route that acts as a proxy or gatekeeper.

Google API (Upstream): They expose the endpoint GET /volumes.
When a user calls our API at http://localhost:8080/google?q=java, this is what happens behind the scenes:
Our Controller receives the request on /google.
Our Service (GoogleBookService) creates a new request.
The Request travels to the Google API's /volumes endpoint.
Google sends the data back to us.
Our Controller sends that data back to the user.
So, while /volumes is the real source of the data, it is not a route that exists directly on our server. Our users don't see /volumes; they only see /google and /books.
This is a common pattern in API development called "Abstraction"—it allows us to change how we call Google (or even swap Google for a different provider) without forcing the people using our API to change their code.

Commands to run project:
$env:JAVA_HOME = "D:\OneDrive - Prodapt Solutions Private Limited\C_drive\.jdks\ms-21.0.8"; .\mvnw.cmd verify

$env:JAVA_HOME = "D:\OneDrive - Prodapt Solutions Private Limited\C_drive\.jdks\ms-21.0.8"; .\mvnw.cmd spring-boot:run

Action
Method
URL
Notes
Search Google
GET
/google?q=effective+java
Returns raw Google API search results.
View Details
GET
/google/ka2VUBqHiWkC
Returns details for a specific Google volume ID.
Add to Collection
POST
/books/ka2VUBqHiWkC
Persists the book into your local DB. Returns 201 Created.
List Personal Books
GET
/books
Returns all books currently saved in your collection.


Test Scenarios to try:
Success: POST a valid ID (like ka2VUBqHiWkC), then GET /books to see it there.
Conflict: Try to POST the same ID twice. You should receive a 409 Conflict with a clear error message.
Validation: Try POST /books/%20 (a space). You should receive a 400 Bad Request.
Not Found: Try POST /books/this_id_does_not_exist. You should receive a 400 Bad Request (mapped from Google's 404).

Yes, it is! ka2VUBqHiWkC is the real, unique volume ID for "Effective Java" by Joshua Bloch (the 2nd Edition).
When you call POST /books/ka2VUBqHiWkC in your app, the following happens:
Your code calls the Google Books API at: https://www.googleapis.com/books/v1/volumes/ka2VUBqHiWkC
It retrieves the title, author, and page count from Google's live database.
It then saves that data into your local H2 database.
Other real IDs you can try for testing:
lRtdEAAAQBAJ: Spring in Action, Sixth Edition.
71B7vAEACAAJ: Clean Code by Robert C. Martin.
_i6bDeoCQzsC: Java: A Beginner's Guide.
Try adding any of these, and then check GET /books to see your growing collection