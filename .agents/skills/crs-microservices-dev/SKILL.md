---
name: crs-microservices-dev
description: >-
  Provides workflow guidelines, architecture reference, port mappings, database-per-service rules,
  and API specifications for developing, running, and testing the Course Registration System (CRS) microservices architecture.
---

# CRS Microservices Development Guide

This skill provides comprehensive instructions for developing, building, and operating the Course Registration System (CRS) microservices stack.

## Service Architecture & Port Mapping

| Service | Port | Database | Primary Responsibility | Gateway Prefix |
| :--- | :--- | :--- | :--- | :--- |
| **api-gateway** | 8080 | (No DB) | Single entry point, routing, auth check, CORS | `/` |
| **auth-service** | 8081 | `auth_db` | User & Student management, login, JWT issuance | `/api/auth` |
| **course-service** | 8082 | `course_db` | Course management, search, pagination, available seats | `/api/courses` |
| **registration-service** | 8083 | `registration_db` | Course enrollment/registration, seat reservation | `/api/registrations` |
| **crs-frontend** | 3000 / 5173 | N/A | React/Vite/Next.js Web User Interface | N/A |

## Core Principles & Rules

1. **Database per Service**:
   - Each microservice owns its database exclusively.
   - Microservices MUST NOT access another service's database directly.
   - Cross-service data requests must be made via internal REST APIs (e.g., Spring `WebClient` or `RestTemplate` / OpenFeign).

2. **Internal API Communication**:
   - `registration-service` references courses via `courseId` (primitive type, no DB foreign key).
   - Seat updates occur via internal APIs:
     - `PATCH /internal/courses/{id}/reserve-seat`
     - `PATCH /internal/courses/{id}/release-seat`
   - Internal endpoints MUST NOT be exposed through `api-gateway`.

3. **Authentication & Authorization**:
   - `auth-service` generates JWT tokens upon successful login (`POST /auth/login`).
   - `api-gateway` validates incoming requests and forwards claims (`X-User-Id`, `X-User-Role`) downstream.
   - Roles: `STUDENT`, `ADMIN`.

## API Specification Overview

### `auth-service` (Port 8081)
- `POST /auth/login` - Public login endpoint, returns JWT
- `POST /auth/register` - Public account registration

### `course-service` (Port 8082)
- `GET /courses` - List courses (search, pagination) - Public
- `GET /courses/{id}` - Get course details - Public
- `POST /courses` - Create course - Requires `ADMIN` role
- `PUT /courses/{id}` - Update course - Requires `ADMIN` role
- `DELETE /courses/{id}` - Delete course - Requires `ADMIN` role
- `PATCH /internal/courses/{id}/reserve-seat` - Internal (seat decrement)
- `PATCH /internal/courses/{id}/release-seat` - Internal (seat increment)

### `registration-service` (Port 8083)
- `POST /registrations` - Register for course - Requires `STUDENT` role
- `GET /registrations/my` - Get current student's registrations - Requires `STUDENT` role
- `DELETE /registrations/{id}` - Cancel registration - Requires `STUDENT` or `ADMIN` role

## Verification Checklist

When implementing new endpoints or features:
- Ensure proper DTO mappings and error handling (`@RestControllerAdvice`).
- Verify database migrations (Flyway / Liquibase) or `schema.sql` update.
- Verify Spring Boot service compiles: `mvn clean test` or `./gradlew test`.
- Verify API Gateway routes match planned path patterns.
