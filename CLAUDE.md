# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Learning project for JWT-based authentication with Spring Security. Spring Boot 4.0.3, Java 21. Uses Maven wrapper for builds.

## Build & Test Commands

```bash
./mvnw clean install          # Build and run all tests
./mvnw test                   # Run all tests
./mvnw test -Dtest=ClassName  # Run a single test class
./mvnw test -Dtest=ClassName#methodName  # Run a single test method
./mvnw spring-boot:run        # Run the application
```

## Architecture

- **Base package:** `com.github.spring_security_jwt_auth`
- **Framework:** Spring Boot 4.0.3 with Spring Security, Spring WebMVC, Spring Actuator
- **Annotation processing:** Lombok (excluded from final artifact)
- **Config format:** YAML (`application.yaml`)

This is a learning project — JWT authentication infrastructure is not yet implemented. Prioritize clear, understandable code over abstractions.
