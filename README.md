# GreenCarWash Central Configuration Repository

This repository acts as the **Centralized Configuration Server Backing Store** for the **GreenCarWash** microservices platform, powered by **Spring Cloud Config Server**.

## 📌 Architecture Overview

```
                                      [ This GitHub Repository ]
                                  (RaajeshKumaar17/carwash-config-repo)
                                                 │
                                                 ▼
                                     [ Config Server (:8888) ]
                               (Spring Cloud Config Server in Git Mode)
                                                 │
                                                 ▼
                ┌────────────────────────────────┼────────────────────────────────┐
                ▼                                ▼                                ▼
      [ API Gateway (:8080) ]         [ Auth Service (:8081) ]       [ User Service (:8082) ]
                ▼                                ▼                                ▼
    [ Catalog Service (:8083) ]      [ Booking Service (:8084) ]    [ Payment Service (:8085) ]
                ▼                                ▼                                ▼
    [ Notify Service (:8086) ]        [ Review Service (:8087) ]     [ Admin Service (:8088) ]
                ▼                                ▼                                ▼
    [ Support Service (:8089) ]      [ Vehicle Service (:8090) ]    [ Assignment Svc (:8091) ]
                ▼                                ▼                                ▼
   [ Verification Svc (:8092) ]     [ Org B2B Service (:8093) ]    [ Sustainability (:8094) ]
```

---

## 📁 Repository Contents

| Configuration File | Target Microservice | Port | Database (MySQL / H2) | Description |
| :--- | :--- | :--- | :--- | :--- |
| [`application.yml`](./application.yml) | **Global Common** | - | - | Common Spring Boot config: RS256 JWT Keys, RabbitMQ, Redis, Resilience4j Circuit Breakers, Logging & Actuator. |
| [`api-gateway.yml`](./api-gateway.yml) | **API Gateway** | `8080` | Redis Cache | Routing table, CORS deduplication, JWT auth filter, Swagger documentation aggregation, rate limiters. |
| [`auth-service.yml`](./auth-service.yml) | **Auth Service** | `8081` | `auth_db` | RS256 JWT token generation, refresh tokens, user credentials, BCrypt password encoder. |
| [`user-service.yml`](./user-service.yml) | **User Service** | `8082` | `user_db` | Customer profiles, multi-address management, geocoded coordinates, avatar uploads. |
| [`service-catalog-service.yml`](./service-catalog-service.yml) | **Catalog Service** | `8083` | `catalog_db` | Eco wash packages, add-ons (ceramic coating, odor eliminator), discount promo code engine. |
| [`booking-service.yml`](./booking-service.yml) | **Booking Service** | `8084` | `booking_db` | "Wash Now" & scheduled bookings, dynamic weather/demand surge pricing, auto-dispatch triggers. |
| [`payment-service.yml`](./payment-service.yml) | **Payment Service** | `8085` | `payment_db` | Two-phase payments, escrow hold, detailer release upon job verification, idempotency keys. |
| [`notification-service.yml`](./notification-service.yml) | **Notification Service** | `8086` | `notification_db` | RabbitMQ event listeners, push notifications, emails, unread notification counter. |
| [`review-service.yml`](./review-service.yml) | **Review Service** | `8087` | `review_db` | Customer & washer bidirectional ratings (1-5 stars), verified reviews, detailer scorecards. |
| [`admin-service.yml`](./admin-service.yml) | **Admin Service** | `8088` | `admin_db` | Platform-wide audit logs, service monitor, system administration controls. |
| [`support-service.yml`](./support-service.yml) | **Support Service** | `8089` | `support_db` | Helpdesk ticketing system, priority escalation, customer-agent communication threads. |
| [`vehicle-service.yml`](./vehicle-service.yml) | **Vehicle Service** | `8090` | `vehicle_db` | Customer garage, vehicle categories (`SEDAN`, `SUV`, `HATCHBACK`, `TRUCK`), license plates. |
| [`washer-assignment-service.yml`](./washer-assignment-service.yml) | **Washer Assignment** | `8091` | `assignment_db` | Haversine proximity dispatch, washer shifts/radius, Swiggy-style non-GPS milestone order tracker. |
| [`media-verification-service.yml`](./media-verification-service.yml) | **Media Verification** | `8092` | `verification_db` | Pre-wash & post-wash photo uploads, AI cleanliness scoring, fraud prevention audit trail. |
| [`organization-service.yml`](./organization-service.yml) | **Organization Service** | `8093` | `organization_db` | B2B corporate fleet accounts, business facility addresses, recurring automated wash schedules. |
| [`sustainability-service.yml`](./sustainability-service.yml) | **Sustainability Service** | `8094` | `sustainability_db` | Gallons of water saved calculation (vs 45 gal baseline), CO2 offset, washer eco-champion leaderboard. |
| [`discovery-service.yml`](./discovery-service.yml) | **Discovery Service** | `8761` | In-Memory | Netflix Eureka Service Registry configuration and self-preservation settings. |
| [`config-service.yml`](./config-service.yml) | **Config Service** | `8888` | - | Spring Cloud Config Server bootstrapping to connect to this Git repository. |

---

## ⚙️ Active Profiles

Each configuration file supports multi-document YAML with profile activation:

1. **`default` (Local Development)**:
   - Microservices can connect to local in-memory H2 or localhost MySQL.
   - Eureka connects to `http://localhost:8761/eureka/`.
   - Redis connects to `localhost:6379`.
   - RabbitMQ connects to `localhost:5672`.

2. **`docker` (Containerized Production)**:
   - Activated via `SPRING_PROFILES_ACTIVE=docker`.
   - Connects to dedicated MySQL instances with database-per-service isolation (`Raajesh` root password).
   - Eureka connects to `http://discovery-service:8761/eureka/`.
   - Redis connects to `redis:6379`.
   - RabbitMQ connects to `rabbitmq:5672`.

---

## 🚀 How Config Server Uses This Repository

In `config-service/src/main/resources/application.yml`:

```yaml
spring:
  application:
    name: config-service
  cloud:
    config:
      server:
        git:
          uri: https://github.com/RaajeshKumaar17/carwash-config-repo.git
          default-label: main
          clone-on-start: true
```

### Dynamic Configuration Refresh
When a property in this repository is updated and pushed:
1. Trigger a refresh across services:
   ```bash
   curl -X POST http://localhost:8080/actuator/busrefresh
   ```
2. Or refresh an individual service:
   ```bash
   curl -X POST http://localhost:8081/actuator/refresh
   ```
Fields annotated with `@RefreshScope` will instantly adopt the new values without restarting containers.
