# SocialApp — Microservices Social Media Platform

A learning project: microservices-based social app with posts, follows, and messaging.  
Built with **Java 21, Spring Boot 4, Spring Cloud**, and deployed on **AWS**.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Microservices](#microservices)
- [Data Model](#data-model)
- [API Contracts](#api-contracts)
- [Development Phases](#development-phases)
- [Design Patterns Used](#design-patterns-used)
- [Deployment Plan](#deployment-plan)
- [Out of Scope (v1)](#out-of-scope-v1)

---

## 🧭 Project Overview

A social media application (inspired by Instagram/Facebook) built as a **backend-first** learning project. The backend uses a microservices architecture covering API Gateway, service discovery, JWT authentication, inter-service communication, and containerized deployment on AWS.

### V1 Features
- ✅ User sign up, login, and profile management
- ✅ Image posts with captions
- ✅ Like and comment on posts
- ✅ Follow / unfollow users
- ✅ One-to-one text messaging

---

## 🏗 Architecture

```
                         ┌─────────────────┐
                         │    Frontend      │
                         │   (AI-assisted)  │
                         └────────┬─────────┘
                                  │
                         ┌────────▼─────────┐
                         │   API Gateway     │
                         │ (Spring Cloud GW) │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    │             │              │
           ┌────────▼──┐  ┌──────▼─────┐  ┌─────▼──────────┐
           │   Eureka   │  │  Config    │  │  Auth (JWT)    │
           │  (Service  │  │  Server    │  │  via User Svc  │
           │ Discovery) │  │ (optional) │  │                │
           └────────────┘  └────────────┘  └────────────────┘
                    │
     ┌──────────────┼──────────────┬──────────────┐
     │              │              │              │
┌────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐  ┌────▼───────┐
│  User    │  │   Post    │  │ Friendship│  │ Messaging  │
│ Service  │  │  Service  │  │  Service  │  │  Service   │
│          │  │           │  │           │  │            │
│ [Postgres]  │ [Postgres]│  │ [Postgres]│  │ [Postgres] │
└──────────┘  └───────────┘  └───────────┘  └────────────┘
```

---

## 🛠 Tech Stack

| Layer              | Technology                                  |
|--------------------|---------------------------------------------|
| Language           | Java 21                                     |
| Framework          | Spring Boot 4.1                             |
| API Gateway        | Spring Cloud Gateway                        |
| Service Discovery  | Netflix Eureka                              |
| Authentication     | JWT (JSON Web Tokens)                       |
| Database           | PostgreSQL (per-service)                    |
| ORM                | Spring Data JPA / Hibernate                 |
| Async Messaging    | Kafka / RabbitMQ                            |
| Resilience         | Resilience4j (circuit breaker, retry)       |
| Containerization   | Docker + Docker Compose                     |
| Cloud Deployment   | AWS (ECS/EKS, RDS, ECR)                    |
| Observability      | Spring Boot Actuator, Micrometer, Zipkin    |
| Build Tool         | Maven                                       |
| Frontend           | Built separately with AI assistance         |

---

## 📦 Microservices

### 1. User Service ✅
**Owns:** `User` entity  
**Responsibilities:** Sign up, login (JWT token generation), profile management, password hashing  
**Port:** `8080`  
**Database:** `socialapp_users`

### 2. Post Service 🔲
**Owns:** `Post`, `Like`, `Comment` entities  
**Responsibilities:** Create/delete posts, image upload, like/unlike, add/delete comments  
**Port:** `8081`  
**Database:** `socialapp_posts`

### 3. Friendship Service 🔲
**Owns:** `Follow` entity  
**Responsibilities:** Follow/unfollow users, get followers list, get following list  
**Port:** `8082`  
**Database:** `socialapp_friendships`

### 4. Messaging Service 🔲
**Owns:** `Message` entity  
**Responsibilities:** Send/receive messages, conversation history, mark as read  
**Port:** `8083`  
**Database:** `socialapp_messages`

### 5. Discovery Server (Eureka) 🔲
**Responsibilities:** Service registration and discovery  
**Port:** `8761`

### 6. API Gateway 🔲
**Responsibilities:** Request routing, JWT validation, rate limiting, load balancing  
**Port:** `8000`

---

## 📊 Data Model

### User (User Service)
| Field    | Type    | Constraints               |
|----------|---------|---------------------------|
| id       | Long    | PK, auto-generated        |
| username | String  | Unique, not null, max 50  |
| name     | String  | Not null, max 100         |
| email    | String  | Unique, not null, max 150 |
| password | String  | Not null, max 255 (hashed)|
| bio      | String  | Optional, max 500         |

### Post (Post Service)
| Field        | Type      | Constraints        |
|--------------|-----------|-------------------|
| id           | Long      | PK, auto-generated|
| userId       | Long      | Not null (FK ref)  |
| image        | String    | URL/path to image  |
| caption      | String    | Optional text      |
| likeCount    | Integer   | Default 0          |
| commentCount | Integer   | Default 0          |
| timestamp    | LocalDateTime | Auto-set        |

### Like (Post Service)
| Field     | Type      | Constraints         |
|-----------|-----------|---------------------|
| id        | Long      | PK, auto-generated  |
| postId    | Long      | FK to Post           |
| userId    | Long      | FK ref to User       |
| timestamp | LocalDateTime | Auto-set         |

### Comment (Post Service)
| Field     | Type      | Constraints         |
|-----------|-----------|---------------------|
| id        | Long      | PK, auto-generated  |
| postId    | Long      | FK to Post           |
| userId    | Long      | FK ref to User       |
| text      | String    | Not null             |
| timestamp | LocalDateTime | Auto-set         |

### Follow (Friendship Service)
| Field      | Type      | Constraints         |
|------------|-----------|---------------------|
| id         | Long      | PK, auto-generated  |
| followerId | Long      | FK ref to User       |
| followeeId | Long      | FK ref to User       |
| timestamp  | LocalDateTime | Auto-set         |

### Message (Messaging Service)
| Field      | Type      | Constraints         |
|------------|-----------|---------------------|
| id         | Long      | PK, auto-generated  |
| senderId   | Long      | FK ref to User       |
| receiverId | Long      | FK ref to User       |
| content    | String    | Not null             |
| timestamp  | LocalDateTime | Auto-set         |
| isRead     | Boolean   | Default false        |

---

## 🔌 API Contracts

### User Service — `/api/user`
| Method | Endpoint              | Description         | Auth Required |
|--------|-----------------------|---------------------|---------------|
| POST   | `/api/user/register`  | Register new user   | No            |
| POST   | `/api/user/login`     | Login, returns JWT  | No            |
| GET    | `/api/user/{username}`| Get user profile    | Yes           |

### Post Service — `/api/post`
| Method | Endpoint                        | Description            | Auth Required |
|--------|---------------------------------|------------------------|---------------|
| POST   | `/api/post`                     | Create a new post      | Yes           |
| GET    | `/api/post/{id}`                | Get a post by ID       | Yes           |
| GET    | `/api/post/user/{userId}`       | Get all posts by user  | Yes           |
| DELETE | `/api/post/{id}`                | Delete a post          | Yes           |
| POST   | `/api/post/{id}/like`           | Like a post            | Yes           |
| DELETE | `/api/post/{id}/like`           | Unlike a post          | Yes           |
| POST   | `/api/post/{id}/comment`        | Add a comment          | Yes           |
| GET    | `/api/post/{id}/comments`       | Get comments on a post | Yes           |
| DELETE | `/api/post/comment/{commentId}` | Delete a comment       | Yes           |

### Friendship Service — `/api/friendship`
| Method | Endpoint                             | Description           | Auth Required |
|--------|--------------------------------------|-----------------------|---------------|
| POST   | `/api/friendship/follow/{userId}`    | Follow a user         | Yes           |
| DELETE | `/api/friendship/unfollow/{userId}`  | Unfollow a user       | Yes           |
| GET    | `/api/friendship/followers/{userId}` | Get followers list    | Yes           |
| GET    | `/api/friendship/following/{userId}` | Get following list    | Yes           |

### Messaging Service — `/api/message`
| Method | Endpoint                              | Description              | Auth Required |
|--------|---------------------------------------|--------------------------|---------------|
| POST   | `/api/message/send`                   | Send a message           | Yes           |
| GET    | `/api/message/conversation/{userId}`  | Get conversation history | Yes           |
| PUT    | `/api/message/{id}/read`              | Mark message as read     | Yes           |

---

## 🚀 Development Phases

### Phase 1 — User Service ✅
- [x] Project setup (Spring Boot 4, Maven, PostgreSQL)
- [x] User entity, repository, service, controller
- [x] Register, login, get profile endpoints
- [x] Password hashing with BCrypt
- [x] Custom exception handling (404, 409)
- [x] JWT token generation on login
- [x] JWT validation filter for protected endpoints
- [ ] Dockerfile

### Phase 2 — Infrastructure ✅
- [x] **Eureka Server** — service discovery server
- [x] **API Gateway** — Spring Cloud Gateway with route config
- [x] Register User Service with Eureka
- [x] Gateway routes to User Service via Eureka
- [ ] JWT validation at Gateway level
- [ ] Dockerfiles for Eureka & Gateway

### Phase 3 — Post Service
- [ ] New Spring Boot project with its own PostgreSQL database
- [ ] Post, Like, Comment entities and CRUD APIs
- [ ] Inter-service communication (validate userId via User Service)
- [ ] Register with Eureka, route via Gateway
- [ ] Dockerfile

### Phase 4 — Friendship Service
- [ ] New Spring Boot project with its own PostgreSQL database
- [ ] Follow entity and follow/unfollow APIs
- [ ] Get followers/following lists
- [ ] Register with Eureka, route via Gateway
- [ ] Dockerfile

### Phase 5 — Messaging Service
- [ ] New Spring Boot project with its own PostgreSQL database
- [ ] Message entity and send/receive APIs
- [ ] Conversation history endpoint
- [ ] Consider WebSocket for real-time messaging
- [ ] Register with Eureka, route via Gateway
- [ ] Dockerfile

### Phase 6 — Async Communication & Resilience
- [ ] Set up Kafka or RabbitMQ
- [ ] Event-driven notifications (e.g., new follower, new like)
- [ ] Resilience4j circuit breakers on inter-service calls
- [ ] Retry and fallback mechanisms

### Phase 7 — Observability
- [ ] Spring Boot Actuator health/metrics endpoints
- [ ] Centralized logging
- [ ] Distributed tracing with Zipkin/Micrometer
- [ ] Monitoring dashboards

### Phase 8 — Containerization & Deployment
- [ ] Docker Compose for local multi-service setup
- [ ] Push images to AWS ECR
- [ ] Deploy PostgreSQL databases on AWS RDS
- [ ] Deploy services on AWS ECS (Fargate) or EKS
- [ ] Configure networking, security groups, load balancers
- [ ] Set up CI/CD pipeline

### Phase 9 — Frontend Integration
- [ ] Build frontend with AI assistance
- [ ] Connect to API Gateway
- [ ] End-to-end testing

---

## 🧩 Design Patterns Used

| Pattern               | Where Used                                    |
|-----------------------|-----------------------------------------------|
| **API Gateway**       | Single entry point for all client requests    |
| **Service Discovery** | Eureka for dynamic service lookup             |
| **Circuit Breaker**   | Resilience4j on inter-service calls           |
| **Repository**        | Spring Data JPA for data access               |
| **DTO**               | Separate request/response objects from entities|
| **Builder**           | Lombok `@Builder` for response construction   |
| **Observer**          | Event-driven notifications (new post, follow) |
| **Saga**              | Distributed transaction management            |

---

## 🚫 Out of Scope (v1)

- Email verification / password reset
- Group messaging
- Follow approval flow
- Video uploads
- Feed ranking algorithm
- Push notifications
- Admin panel
- OAuth / social login

---

## 📁 Project Structure

```
SocialApp/
├── README.md
├── SocialAppPlan
├── user-service/          ← ✅ Built
├── post-service/          ← 🔲 Phase 3
├── friendship-service/    ← 🔲 Phase 4
├── messaging-service/     ← 🔲 Phase 5
├── discovery-server/      ← 🔲 Phase 2
├── api-gateway/           ← 🔲 Phase 2
└── docker-compose.yml     ← 🔲 Phase 8
```

---

## 🏃 How to Run (Current)

```bash
# Ensure Java 21 and PostgreSQL are running
# Create database
psql -U postgres -c "CREATE DATABASE socialapp_users;"

# Run User Service
cd user-service
./mvnw spring-boot:run

# Test endpoints
# Register: POST http://localhost:8080/api/user/register
# Login:    POST http://localhost:8080/api/user/login
# Profile:  GET  http://localhost:8080/api/user/{username}
```

---

*Built as a learning project to master microservices architecture, Spring Cloud, and AWS deployment.*

