# 🏆 Milestone 1 — Foundation & Infrastructure Complete

**Date:** August 25, 2026  
**Status:** ✅ Complete

---

## What Was Built

Three microservices forming the foundation of the SocialApp platform:

```
SocialApp/
├── discovery-server/     ← Service Registry (Eureka)
├── api-gateway/          ← Single Entry Point (Spring Cloud Gateway)
└── user-service/         ← User Management (First Business Service)
```

---

## 1. Discovery Server (Eureka)

**Port:** `8761`  
**Purpose:** Service registry — allows microservices to find each other dynamically.

### Why?
In a microservices architecture, services need to communicate with each other. Instead of hardcoding URLs like `http://localhost:8080`, services **register** themselves with Eureka. When one service needs to call another, it asks Eureka: *"Where is USER-SERVICE?"* — and Eureka responds with the current address. This enables:
- Dynamic scaling (multiple instances of a service)
- No hardcoded URLs
- Automatic detection when a service goes down

### What it does:
- Runs a web dashboard at `http://localhost:8761`
- Accepts registrations from all microservices
- Provides service lookup for the API Gateway

### Tech: 
`Spring Cloud Netflix Eureka Server`

---

## 2. API Gateway (Spring Cloud Gateway)

**Port:** `8000`  
**Purpose:** Single entry point for all client requests.

### Why?
Without a gateway, the frontend would need to know the URL and port of every microservice. The gateway solves this by providing **one URL** (`localhost:8000`) that routes requests to the correct service:

```
Client → localhost:8000/api/user/**  → routed to USER-SERVICE (8080)
Client → localhost:8000/api/post/**  → routed to POST-SERVICE (future)
```

### What it does:
- **Routes requests** to the correct microservice via Eureka (`lb://USER-SERVICE`)
- **JWT Authentication** — validates Bearer tokens on protected endpoints
- **Open endpoints** — `/api/user/register` and `/api/user/login` pass through without authentication
- **Adds `X-Auth-User` header** — after validating the token, it extracts the username and forwards it to downstream services so they know who's making the request

### Key Files:
| File | Purpose |
|------|---------|
| `JwtAuthenticationFilter.java` | Gateway filter that validates JWT tokens |
| `JwtUtil.java` | Token parsing and validation using shared secret |
| `RouteValidator.java` | Defines which endpoints are public (no auth needed) |

### Tech:
`Spring Cloud Gateway (WebFlux/Reactive)`, `Eureka Client`, `jjwt`

---

## 3. User Service

**Port:** `8080`  
**Database:** `socialapp_users` (PostgreSQL)  
**Purpose:** Handles user registration, login, profile management.

### Why?
The first business service — every social app needs users. It's also the **JWT issuer** — the only service that generates tokens (on login), which the Gateway then validates on subsequent requests.

### API Endpoints:
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/user/register` | Create new account | ❌ Public |
| POST | `/api/user/login` | Login, returns JWT token | ❌ Public |
| GET | `/api/user/{username}` | Get user profile | ✅ Required |
| PATCH | `/api/user/{username}/update` | Update user profile | ✅ Required |

### Key Files:
| File | Purpose |
|------|---------|
| `User.java` | JPA entity mapped to `users` table |
| `UserController.java` | REST endpoints |
| `UserService.java` | Business logic (register, login, update) |
| `UserRepository.java` | Database queries (Spring Data JPA) |
| `JwtService.java` | Generates JWT tokens on login |
| `SecurityConfig.java` | All requests permitted (Gateway handles auth) |
| `PasswordConfig.java` | BCrypt password encoder bean |
| `GlobalExceptionHandler.java` | Maps exceptions to proper HTTP status codes |

### DTOs:
| DTO | Used For |
|-----|----------|
| `LoginRequest` | Login input (username + password) |
| `LoginResponse` | Login output (JWT token + user info) |
| `UserResponse` | User profile output (no password exposed) |
| `UpdateUser` | Partial update input |
| `ErrorResponse` | Error output (status + message + timestamp) |

### Custom Exceptions:
| Exception | HTTP Status | When |
|-----------|-------------|------|
| `UserNotFoundException` | 404 | User not found by username |
| `UserNameAlreadyExistException` | 409 | Username taken (register/update) |
| `EmailAlreadyExistException` | 409 | Email already in use (register/update) |

### Tech:
`Spring Boot 4.1.1`, `Spring Security`, `Spring Data JPA`, `PostgreSQL`, `Lombok`, `jjwt`, `Eureka Client`

---

## How It All Works Together

```
1. Client sends POST /api/user/login to Gateway (port 8000)
   → Gateway sees it's a public endpoint → passes through to User Service
   → User Service validates credentials → returns JWT token

2. Client sends GET /api/user/nitin with "Authorization: Bearer <token>" to Gateway
   → Gateway's JwtAuthenticationFilter:
      a. Checks it's NOT a public endpoint
      b. Extracts token from Authorization header
      c. Validates signature using shared secret
      d. Extracts username from token
      e. Adds "X-Auth-User: nitin" header
      f. Forwards request to User Service
   → User Service returns user profile

3. Gateway looks up USER-SERVICE in Eureka (not hardcoded URL)
   → Eureka returns the current address (localhost:8080)
   → Gateway forwards the request
```

---

## Tech Stack Summary

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Java | 21 |
| Framework | Spring Boot | 4.1.1 |
| Cloud | Spring Cloud | 2025.1.3 |
| Gateway | Spring Cloud Gateway (WebFlux) | — |
| Service Discovery | Netflix Eureka | — |
| Authentication | JWT (jjwt) | 0.12.6 |
| Database | PostgreSQL | 14+ |
| ORM | Hibernate / Spring Data JPA | 7.4 |
| Security | Spring Security | 7.1 |
| Build | Maven | 3.9 |

---

## What's Next (Phase 3)

**Post Service** — the next business service:
- Post, Like, Comment entities with own PostgreSQL database
- CRUD APIs for creating posts, liking, commenting
- Inter-service communication (reads `X-Auth-User` header to know who's making the request)
- Registers with Eureka, routes via Gateway

---

*Milestone 1 establishes the core infrastructure pattern. Every future service (Post, Friendship, Messaging) follows the same pattern: create Spring Boot app → register with Eureka → add Gateway route → read `X-Auth-User` header.*

