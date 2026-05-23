# Java REST API Demo

A throwaway REST API built as part of learning Java enterprise development with Spring Boot. This project was scaffolded using [Spring Initializr](https://start.spring.io) and serves as a proof of concept before building a full production application.

---

## Why This Was Built

This project was built as a learning exercise to understand the full request/response cycle of a Spring Boot REST API before moving on to a real production project. The goal was to get hands on experience with Java, Spring Boot, JPA, and JWT authentication in a low stakes environment where nothing needed to be perfect.

---

## What It Does

A simple product API with full CRUD operations. Public endpoints allow anyone to read product data. Write and delete operations are protected and require a valid JWT token, mirroring the pattern used in real enterprise applications.

### Endpoints

| Method | URL | Auth Required | Description |
|---|---|---|---|
| GET | /api/products | No | Get all products |
| GET | /api/products/{id} | No | Get product by ID |
| GET | /api/products/search?name= | No | Search products by name |
| POST | /api/products | Yes | Create a product |
| DELETE | /api/products/{id} | Yes | Delete a product |

---

## Stack

| Layer | Choice |
|---|---|
| Language | Java 21 |
| Build Tool | Maven |
| Framework | Spring Boot 3.5.14 |
| Database | H2 (in-memory) |
| ORM | JPA / Hibernate |
| Authentication | JWT via jjwt 0.12.6 |
| Boilerplate Reduction | Lombok |
| API Testing | Bruno / curl |

---

## How Spring Boot Is Used

Spring Boot manages the entire application lifecycle. Rather than manually constructing and wiring objects together, Spring's IoC container reads the annotated classes at startup and handles dependency injection automatically. Controllers, services, and repositories are each annotated with their respective roles (`@RestController`, `@Service`, `@Repository`) and Spring wires them together without any manual instantiation.

The embedded Tomcat server starts automatically on port 8080 with no external server configuration required.

---

## How JWT Authentication Works

Authentication is handled via JSON Web Tokens. When the admin hits the login endpoint with valid credentials, Spring Security validates them against an `InMemoryUserDetailsManager` configured in `SecurityConfig.java` with a hardcoded admin user. A signed JWT token is then returned. Every subsequent request to a protected endpoint must include this token in the Authorization header as `Bearer <token>`.

On each request, a custom `JwtFilter` intercepts it before it reaches any controller, extracts the token from the header, and validates the signature using a secret key. If valid, Spring Security marks the request as authenticated and allows it through. If missing or invalid, a 403 is returned. The token is signed using HMAC-SHA384 so it cannot be forged without the secret key.

---

## How JPA / Hibernate Is Used

The `Product` entity maps directly to a database table via JPA annotations. `ProductRepository` extends `JpaRepository` which gives full CRUD operations for free with no SQL written manually. Spring Data JPA reads method names like `findByNameContainingIgnoreCase` and generates the appropriate query automatically at startup.

H2 is used as an in-memory database for simplicity — the schema is created automatically by Hibernate when the app starts and dropped when it stops. No database setup is required to run this project.

---

## Testing

All endpoints were tested manually using [Bruno](https://www.usebruno.com), an open source API testing desktop application, and curl which is built into Windows 10 (version 1803 and later), Windows 11, Mac, and Linux. Both were used to send HTTP requests, inspect responses, manage the JWT token in the Authorization header, and verify that protected endpoints correctly rejected unauthenticated requests with a 403.

---

## Full Test Cycle Using curl

Run these commands in order to verify the entire API is working. Replace `<token>` with the token returned from the login command.

Note: Steps 2 onwards use Windows Command Prompt syntax with escaped quotes. On Mac / Linux replace `\"` with `'` and wrap the body in single quotes instead.

**1. Start the server**

Windows:

    mvnw.cmd spring-boot:run

Mac / Linux:

    ./mvnw spring-boot:run

**2. Get a token**

    curl -X POST http://localhost:8080/api/auth/login -H "Content-Type: application/json" -d "{\"username\":\"admin\",\"password\":\"password\"}"

**3. Create a product**

    curl -X POST http://localhost:8080/api/products -H "Content-Type: application/json" -H "Authorization: Bearer <token>" -d "{\"name\":\"Heineken\",\"description\":\"Dutch lager\",\"price\":9.99}"

**4. Get all products**

    curl http://localhost:8080/api/products

**5. Get product by ID**

    curl http://localhost:8080/api/products/1

**6. Search by name**

    curl "http://localhost:8080/api/products/search?name=hein"

**7. Delete product**

    curl -X DELETE http://localhost:8080/api/products/1 -H "Authorization: Bearer <token>"

**8. Confirm deletion**

    curl http://localhost:8080/api/products

Expected results in order — token JSON, product with id, array with one product, single product, search result, empty response, empty array.

---

## H2 Database Console

The H2 database console is available at http://localhost:8080/h2-console while the server is running.

Use the following to connect:

    JDBC URL:  jdbc:h2:mem:testdb
    Username:  sa
    Password:  (leave blank)
