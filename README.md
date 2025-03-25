# Building a Simple Hospital Appointment Service

**Objective:** To develop a foundational backend service using Java Spring Boot, simulating core functionalities of a hospital/provider system. This assignment will cover essential backend concepts like API design, data persistence, error handling, documentation, logging, containerization, and asynchronous task execution.

**Tech Stack:**

*   **Programming Language:** Java (Version 17 or higher recommended)
*   **Framework:** Spring Boot (Latest Stable Version)
*   **Database:** SQLite or LiteDB (Choose one - SQLite is recommended for simplicity)
*   **Build Tool:** Maven or Gradle (Maven is recommended for beginners)
*   **Documentation:** Swagger/OpenAPI (SpringDoc library)
*   **Logging:** OpenTelemetry (Simple logging to file)
*   **Containerization:** Docker

**Assignment Structure:**

This assignment is divided into progressive steps, each building upon the previous one.

**Step 1: Project Setup and Basic Patient API (CRUD Operations)**

**Task:**

1.  **Set up a new Spring Boot project:**
    *   Use Spring Initializr ([https://start.spring.io/](https://start.spring.io/)) to generate a new Maven or Gradle project.
    *   Include dependencies: `Spring Web`, `Spring Data JPA` (if using SQLite), or relevant LiteDB dependency, and `H2 Database` (for initial testing, can switch to SQLite later).
    *   Import the project into your preferred IDE (IntelliJ IDEA, Eclipse, VS Code).

2.  **Define the `Patient` Entity:**
    *   Create a Java class `Patient` under `src/main/java/<your_package>/model`.
    *   Include fields: `patientId` (Long, primary key, auto-generated), `firstName` (String), `lastName` (String), `dateOfBirth` (LocalDate), `contactNumber` (String).
    *   Annotate `Patient` class with `@Entity` and `patientId` with `@Id` and `@GeneratedValue(strategy = GenerationType.IDENTITY)`.

    *Hint:*
    ```java
    import jakarta.persistence.*;
    import java.time.LocalDate;

    @Entity
    public class Patient {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long patientId;
        private String firstName;
        private String lastName;
        private LocalDate dateOfBirth;
        private String contactNumber;

        // ... Getters and Setters, Constructors ...
    }
    ```

3.  **Create a `PatientRepository`:**
    *   Create an interface `PatientRepository` extending `JpaRepository<Patient, Long>` under `src/main/java/<your_package>/repository`.
    *   This will provide basic CRUD operations for `Patient`.

    *Hint:*
    ```java
    import org.springframework.data.jpa.repository.JpaRepository;
    import org.springframework.stereotype.Repository;

    @Repository
    public interface PatientRepository extends JpaRepository<Patient, Long> {
    }
    ```

4.  **Implement `PatientService`:**
    *   Create a `PatientService` class under `src/main/java/<your_package>/service`.
    *   Implement methods for:
        *   `createPatient(Patient patient)`:  Saves a new patient using `PatientRepository`.
        *   `getPatientById(Long patientId)`: Retrieves a patient by ID. Handle `Optional` appropriately and return a 404 status if not found.
        *   `updatePatient(Long patientId, Patient updatedPatient)`: Updates an existing patient. Handle 404 if patient not found.
        *   `deletePatient(Long patientId)`: Deletes a patient by ID. Handle 404 if patient not found.
        *   `getAllPatients()`: Retrieves all patients.

    *Hint:*
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.stereotype.Service;
    import java.util.List;
    import java.util.Optional;

    @Service
    public class PatientService {

        @Autowired
        private PatientRepository patientRepository;

        public Patient createPatient(Patient patient) {
            return patientRepository.save(patient);
        }

        public Optional<Patient> getPatientById(Long patientId) {
            return patientRepository.findById(patientId);
        }
        // ... other methods ...
    }
    ```

5.  **Create `PatientController`:**
    *   Create a `PatientController` class under `src/main/java/<your_package>/controller`.
    *   Implement REST endpoints using `@RestController` and `@RequestMapping("/api/patients")`.
    *   Implement endpoints for:
        *   `POST /api/patients`: Create a new patient (use `@PostMapping`).
        *   `GET /api/patients/{patientId}`: Get patient by ID (use `@GetMapping("/{patientId}")`).
        *   `PUT /api/patients/{patientId}`: Update patient (use `@PutMapping("/{patientId}")`).
        *   `DELETE /api/patients/{patientId}`: Delete patient (use `@DeleteMapping("/{patientId}")`).
        *   `GET /api/patients`: Get all patients (use `@GetMapping`).
    *   Use `@RequestBody` for request bodies and `@PathVariable` for path parameters.
    *   Return appropriate HTTP status codes using `ResponseEntity`.

    *Hint:*
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;
    import java.util.List;
    import java.util.Optional;

    @RestController
    @RequestMapping("/api/patients")
    public class PatientController {

        @Autowired
        private PatientService patientService;

        @PostMapping
        public ResponseEntity<Patient> createPatient(@RequestBody Patient patient) {
            Patient createdPatient = patientService.createPatient(patient);
            return new ResponseEntity<>(createdPatient, HttpStatus.CREATED);
        }

        @GetMapping("/{patientId}")
        public ResponseEntity<Patient> getPatientById(@PathVariable Long patientId) {
            Optional<Patient> patient = patientService.getPatientById(patientId);
            return patient.map(p -> new ResponseEntity<>(p, HttpStatus.OK))
                           .orElseGet(() -> new ResponseEntity<>(HttpStatus.NOT_FOUND));
        }
        // ... other endpoints ...
    }
    ```

6.  **Test your API:**
    *   Run the Spring Boot application.
    *   Use a tool like Postman, Insomnia, or `curl` to test the API endpoints. Verify CRUD operations are working correctly and status codes are appropriate.

**Step 2: Implement Appointment Entity and Relationships**

**Task:**

1.  **Define the `Appointment` Entity:**
    *   Create a Java class `Appointment` under `src/main/java/<your_package>/model`.
    *   Include fields: `appointmentId` (Long, primary key, auto-generated), `appointmentDateTime` (LocalDateTime), `reasonForVisit` (String), `status` (String - e.g., "SCHEDULED", "COMPLETED", "CANCELLED").
    *   Establish a relationship with `Patient`:  One Patient can have many Appointments. Use `@ManyToOne` annotation in `Appointment` and `@JoinColumn(name = "patient_id")` to link it to the `Patient` entity. Add `@JsonBackReference` on the Patient side to avoid infinite recursion during serialization.

    *Hint:*
    ```java
    import jakarta.persistence.*;
    import java.time.LocalDateTime;
    import com.fasterxml.jackson.annotation.JsonBackReference;

    @Entity
    public class Appointment {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long appointmentId;
        private LocalDateTime appointmentDateTime;
        private String reasonForVisit;
        private String status;

        @ManyToOne
        @JoinColumn(name = "patient_id")
        @JsonBackReference // To prevent infinite recursion during serialization
        private Patient patient;

        // ... Getters and Setters, Constructors ...
    }
    ```
    *   In the `Patient` entity, add a collection of appointments using `@OneToMany(mappedBy = "patient", cascade = CascadeType.ALL)` and `@JsonManagedReference`.

    *Hint:* In `Patient.java`:
    ```java
    import jakarta.persistence.*;
    import java.time.LocalDate;
    import java.util.List;
    import com.fasterxml.jackson.annotation.JsonManagedReference;

    @Entity
    public class Patient {
        // ... existing fields ...

        @OneToMany(mappedBy = "patient", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
        @JsonManagedReference
        private List<Appointment> appointments;

        // ... Getters and Setters, Constructors ...
    }
    ```

2.  **Create `AppointmentRepository`, `AppointmentService`, and `AppointmentController`:**
    *   Similar to `Patient`, create `AppointmentRepository` (extending `JpaRepository`), `AppointmentService`, and `AppointmentController`.
    *   Implement CRUD operations for `Appointment`. Ensure you handle the relationship with `Patient` appropriately, especially when creating and retrieving appointments.

    *Hint:* When creating an appointment, you'll need to associate it with an existing patient. In your `AppointmentController` (POST endpoint), you might accept `patientId` in the request body or path to link the appointment to a specific patient.

3.  **Test Appointment APIs:**
    *   Use Postman/Insomnia/`curl` to test the Appointment APIs, including creating appointments associated with patients, retrieving appointments, updating, and deleting.

**Step 3: Input Validation and Error Handling**

**Task:**

1.  **Implement Input Validation:**
    *   Use Bean Validation annotations (`jakarta.validation.constraints`) in your `Patient` and `Appointment` entities. Examples: `@NotBlank`, `@NotNull`, `@PastOrPresent`, `@Size`, `@Email`, `@Pattern`.
    *   In your controllers, use `@Valid` annotation before `@RequestBody` parameters to trigger validation.

    *Hint:*
    ```java
    import jakarta.validation.constraints.NotBlank;
    import jakarta.validation.constraints.Size;
    // ... other imports

    @Entity
    public class Patient {
        // ... existing fields ...
        @NotBlank(message = "First name cannot be blank")
        @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
        private String firstName;
        // ... other fields with validations ...
    }
    ```

2.  **Implement Global Exception Handling:**
    *   Create a global exception handler class using `@ControllerAdvice` and `@ExceptionHandler`.
    *   Handle `MethodArgumentNotValidException` to return a user-friendly error response (e.g., a list of validation errors) with HTTP status code 400 (Bad Request).
    *   Handle `NoSuchElementException` or custom exceptions for resource not found scenarios and return 404 (Not Found).
    *   Handle general exceptions and return 500 (Internal Server Error) for unexpected issues.

    *Hint:*
    ```java
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.MethodArgumentNotValidException;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import java.util.HashMap;
    import java.util.Map;

    @ControllerAdvice
    public class GlobalExceptionHandler {

        @ExceptionHandler(MethodArgumentNotValidException.class)
        public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex) {
            Map<String, String> errors = new HashMap<>();
            ex.getBindingResult().getFieldErrors().forEach(error ->
                    errors.put(error.getField(), error.getDefaultMessage())
            );
            return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
        }

        // ... other exception handlers ...
    }
    ```

3.  **Test Error Handling:**
    *   Send invalid requests (e.g., missing required fields, invalid data formats) to your APIs and verify that you get appropriate error responses with 400 status codes.
    *   Try to access non-existent resources (e.g., patient ID that doesn't exist) and verify 404 responses.

**Step 4:  Correct HTTP Status Codes**

**Task:**

1.  **Review and Refine Status Codes:**
    *   Ensure you are using appropriate HTTP status codes throughout your controllers.
    *   **200 OK:** For successful GET, PUT, PATCH, DELETE requests.
    *   **201 Created:** For successful POST requests (resource creation).
    *   **204 No Content:** For successful DELETE requests where no response body is needed.
    *   **400 Bad Request:** For validation errors or invalid input.
    *   **404 Not Found:** When a resource is not found.
    *   **500 Internal Server Error:** For unexpected server errors (handled by exception handler).

    *Hint:* Refer to HTTP status code documentation (e.g., MDN Web Docs) to ensure you're using them semantically correctly. Spring `ResponseEntity` is your friend for setting status codes explicitly.

2.  **Test Status Codes:**
    *   Test each API endpoint scenario (success, validation failure, resource not found, etc.) and confirm the returned HTTP status codes are correct.

**Step 5: Swagger Documentation (OpenAPI)**

**Task:**

1.  **Add Swagger/OpenAPI Documentation:**
    *   Add SpringDoc OpenAPI library dependency to your `pom.xml` or `build.gradle`.
    *   Add `@OpenAPIDefinition` and `@Info` annotations to your main application class to configure basic API information (title, description, version).
    *   Use Swagger annotations (e.g., `@Operation`, `@ApiResponse`, `@Schema`) in your controllers to further document your API endpoints, request/response bodies, and parameters.

    *Hint (Maven Dependency - Add to `<dependencies>` in `pom.xml`):*
    ```xml
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
        <version>2.x.x</version>  <!-- Use the latest version -->
    </dependency>
    ```
    *Hint (Main Application Class Annotation):*
    ```java
    import io.swagger.v3.oas.annotations.OpenAPIDefinition;
    import io.swagger.v3.oas.annotations.info.Info;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;

    @SpringBootApplication
    @OpenAPIDefinition(
            info = @Info(title = "Hospital Appointment Service API", version = "v1", description = "API for managing hospital appointments and patients")
    )
    public class HospitalAppointmentServiceApplication {
        public static void main(String[] args) {
            SpringApplication.run(HospitalAppointmentServiceApplication.class, args);
        }
    }
    ```
    *Hint (Controller Endpoint Annotation):*
    ```java
    import io.swagger.v3.oas.annotations.Operation;
    import io.swagger.v3.oas.annotations.responses.ApiResponse;
    import io.swagger.v3.oas.annotations.media.Content;
    import io.swagger.v3.oas.annotations.media.Schema;

    @RestController
    @RequestMapping("/api/patients")
    public class PatientController {
        // ...

        @Operation(summary = "Create a new patient",
                   responses = {
                           @ApiResponse(responseCode = "201", description = "Patient created successfully",
                                   content = @Content(schema = @Schema(implementation = Patient.class))),
                           @ApiResponse(responseCode = "400", description = "Invalid input")
                   })
        @PostMapping
        public ResponseEntity<Patient> createPatient(@Valid @RequestBody Patient patient) {
            // ...
        }
        // ...
    }
    ```

2.  **Access Swagger UI:**
    *   Run your application and access the Swagger UI in your browser (usually at `/swagger-ui.html` or `/v3/api-docs/swagger-ui/index.html`).
    *   Verify that your API documentation is correctly generated and reflects your endpoints, request/response models, and descriptions.

**Step 6: OpenTelemetry Logging (File)**

**Task:**

1.  **Add OpenTelemetry Logging Dependencies:**
    *   Add OpenTelemetry and Simple Logging Facade for Java (SLF4j) dependencies to your `pom.xml` or `build.gradle`. You can use a simple logging backend like `logback-classic`.

    *Hint (Maven Dependencies - Add to `<dependencies>` in `pom.xml`):*
    ```xml
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-api</artifactId>
        <version>1.35.0</version> <!-- Use latest version -->
    </dependency>
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-logging</artifactId>
        <version>1.35.0</version> <!-- Use latest version -->
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.9</version> <!-- Use latest version -->
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version> <!-- Use latest version -->
    </dependency>
    ```

2.  **Configure Logging:**
    *   Configure your logging framework (e.g., Logback) to write logs to a file (e.g., `application.log`) in your project directory or a specified location.
    *   Use SLF4j `Logger` in your services and controllers to log important events:
        *   Log entry and exit of service methods.
        *   Log exceptions and errors.
        *   Log significant data changes (e.g., patient creation, appointment updates).

    *Hint (Example Logging in Service):*
    ```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.stereotype.Service;
    import java.util.Optional;

    @Service
    public class PatientService {

        private static final Logger logger = LoggerFactory.getLogger(PatientService.class);

        // ...

        public Patient createPatient(Patient patient) {
            logger.info("Creating new patient: {}", patient.getFirstName() + " " + patient.getLastName());
            Patient createdPatient = patientRepository.save(patient);
            logger.info("Patient created successfully with ID: {}", createdPatient.getPatientId());
            return createdPatient;
        }

        public Optional<Patient> getPatientById(Long patientId) {
            logger.debug("Fetching patient with ID: {}", patientId);
            Optional<Patient> patient = patientRepository.findById(patientId);
            if (patient.isEmpty()) {
                logger.warn("Patient not found with ID: {}", patientId);
            }
            return patient;
        }
        // ...
    }
    ```

3.  **Verify Logs:**
    *   Run your application and interact with your APIs.
    *   Check if logs are being written to the configured log file.
    *   Review the logs to see if they contain useful information about application execution, errors, and events.

**Step 7: Dockerization**

**Task:**

1.  **Create a `Dockerfile`:**
    *   Create a file named `Dockerfile` in the root directory of your project.
    *   Define instructions to build a Docker image for your application:
        *   Use a base image (e.g., `openjdk:17-jdk-slim`).
        *   Copy your Maven/Gradle build files (`pom.xml` or `build.gradle.kts`) and `src` directory into the image.
        *   Build your Spring Boot application using Maven/Gradle within the image.
        *   Use `EXPOSE 8080` to expose the application port.
        *   Use `CMD` to define the command to run your application when the container starts (e.g., `java -jar <your-app>.jar`).

    *Hint (Example `Dockerfile` - Maven Project):*
    ```dockerfile
    FROM openjdk:17-jdk-slim

    WORKDIR /app

    COPY pom.xml .
    COPY src ./src

    RUN ./mvnw clean install -DskipTests  # Or 'mvnw clean package -DskipTests'

    FROM openjdk:17-jre-slim
    WORKDIR /app
    COPY --from=0 /app/target/*.jar app.jar

    EXPOSE 8080

    CMD ["java", "-jar", "app.jar"]
    ```

2.  **Build Docker Image:**
    *   Open a terminal in your project root directory.
    *   Run the command: `docker build -t hospital-appointment-service .` (replace `hospital-appointment-service` with your desired image name).

3.  **Run Docker Container:**
    *   Run the Docker image using: `docker run -p 8080:8080 hospital-appointment-service`
    *   Access your API through `http://localhost:8080/swagger-ui.html` (or your API endpoints) and verify it's working within the Docker container.

**Step 8: Fire and Forget Request (Asynchronous Task)**

**Task:**

1.  **Implement an Asynchronous Service Method:**
    *   Create a new service (e.g., `NotificationService`) or add a method to an existing service that simulates a long-running task (e.g., sending a welcome email or appointment confirmation).
    *   Annotate this method with `@Async`.
    *   Enable asynchronous processing in your main application class using `@EnableAsync`.

    *Hint (Example `NotificationService`):*
    ```java
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.scheduling.annotation.Async;
    import org.springframework.stereotype.Service;

    @Service
    public class NotificationService {

        private static final Logger logger = LoggerFactory.getLogger(NotificationService.class);

        @Async
        public void sendAppointmentConfirmation(Appointment appointment) {
            logger.info("Starting asynchronous task: Sending appointment confirmation for appointment ID: {}", appointment.getAppointmentId());
            // Simulate a long-running task (e.g., email sending, external API call)
            try {
                Thread.sleep(5000); // Simulate 5 seconds delay
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                logger.error("Task interrupted: ", e);
            }
            logger.info("Asynchronous task completed: Appointment confirmation sent for appointment ID: {}", appointment.getAppointmentId());
        }
    }
    ```
    *Hint (Enable Async in Main Application Class):*
    ```java
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.scheduling.annotation.EnableAsync;

    @SpringBootApplication
    @EnableAsync // Enable asynchronous processing
    public class HospitalAppointmentServiceApplication {
        // ...
    }
    ```

2.  **Call Asynchronous Method from Controller:**
    *   In your `AppointmentController` (or relevant controller), after successfully creating an appointment, inject `NotificationService` and call the `@Async` method (`sendAppointmentConfirmation`).

    *Hint (Calling Async Method in Controller):*
    ```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.*;
    import jakarta.validation.Valid;

    @RestController
    @RequestMapping("/api/appointments")
    public class AppointmentController {

        @Autowired
        private AppointmentService appointmentService;
        @Autowired
        private NotificationService notificationService; // Inject NotificationService

        @PostMapping
        public ResponseEntity<Appointment> createAppointment(@Valid @RequestBody Appointment appointment) {
            Appointment createdAppointment = appointmentService.createAppointment(appointment);
            notificationService.sendAppointmentConfirmation(createdAppointment); // Fire and forget
            return new ResponseEntity<>(createdAppointment, HttpStatus.CREATED);
        }
        // ...
    }
    ```

3.  **Verify Asynchronous Execution:**
    *   Run your application.
    *   Create a new appointment using the API.
    *   Observe the logs. You should see that the API request returns quickly (201 Created), and the log messages from `sendAppointmentConfirmation` method are printed asynchronously *after* the API response has been sent back.

***
