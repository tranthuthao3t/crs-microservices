---
name: spring-boot-microservice
description: >-
  Provides best practices, package structure, REST API conventions, database configuration,
  and error handling patterns for creating and extending Spring Boot microservices.
---

# Spring Boot Microservice Standard Guide

Use this skill when initializing or refactoring Spring Boot microservices, building RESTful controllers, setting up JPA repositories, or handling global exceptions.

## Recommended Package Structure

```text
com.crs.<servicename>/
├── config/             # Security, Web, Swagger/OpenAPI, Beans configuration
├── controller/         # REST Controllers (@RestController, @RequestMapping)
├── dto/                # Request and Response DTOs
│   ├── request/        # LoginRequest, CreateCourseRequest, etc.
│   └── response/       # AuthResponse, CourseResponse, ApiResponse
├── entity/             # JPA Entities (@Entity, @Table)
├── exception/          # Custom exceptions & GlobalExceptionHandler (@RestControllerAdvice)
├── repository/         # Spring Data JPA Repositories (@Repository)
├── service/            # Business logic interfaces & implementations
│   └── impl/
└── mapper/             # Entity <-> DTO converters (MapStruct or manual)
```

## Standard API Response Envelope

Wrap API responses in a consistent wrapper:

```java
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;
    private Instant timestamp;

    public static <T> ApiResponse<T> success(T data, String message) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setSuccess(true);
        response.setMessage(message);
        response.setData(data);
        response.setTimestamp(Instant.now());
        return response;
    }

    public static <T> ApiResponse<T> error(String message) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setSuccess(false);
        response.setMessage(message);
        response.setTimestamp(Instant.now());
        return response;
    }
}
```

## Global Exception Handling

Use `@RestControllerAdvice` to capture errors cleanly without exposing raw stack traces to users:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ApiResponse.error(ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Map<String, String>>> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        ApiResponse<Map<String, String>> response = ApiResponse.error("Validation failed");
        response.setData(errors);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
}
```

## Verification & Testing

1. Validate Maven build:
   `mvn clean verify`
2. Validate Gradle build:
   `./gradlew check`
3. Run unit tests for individual services before opening pull requests.
