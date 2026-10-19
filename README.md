# SensorFlow — IoT Measurement Management Platform

A full-stack web application for managing, exploring, and visualizing IoT sensor measurements. SensorFlow combines an Angular dashboard with a Spring Boot REST API and PostgreSQL storage, providing time-series charts, filtering, and role-based administration.

**Tech stack:** Java 21 · Spring Boot 3.5.6 · Angular 20 · PostgreSQL 16 · Docker Compose · Nginx

## Features

### Measurement dashboard

![Measurement dashboard](./documentation/dahsboard.png)

Explore measurements across sensor series in a responsive interface:

- Interactive time-series charts powered by Chart.js
- Preset date ranges (7 and 30 days) and custom date filters
- Select which measurement series to display
- Paginated, sortable measurement table synchronized with the charts
- Print-friendly data export

### Series management

![Series management](./documentation/admin_series.png)

Administrators can create, update, and delete measurement series (such as temperature, humidity, or pressure), configure valid value ranges, and assign chart colors. Deleting a series also deletes its associated measurements.

### Measurement management

![Measurement management](./documentation/admin_measurements.png)

Administrators can browse, create, edit, and delete individual measurements, including historical entries. Input validation enforces the constraints defined for each series.

### User management

![User management](./documentation/admin_users.png)

Administrators can manage accounts and assign `USER` or `ADMIN` access levels. Standard users have read-only access to measurement data; administrators can also manage series, measurements, and users.

### Authentication and account security

- JWT-based authentication for protected API operations
- Login form with client-side validation and session persistence
- Route guards and role-based API authorization
- Password changes requiring the current password, new-password validation, and confirmation
- BCrypt password hashing

## Architecture

The application uses a three-tier architecture with separate frontend, backend, and database services.

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     Client      │    │    Frontend     │    │     Backend     │    │    Database     │
│                 │    │                 │    │                 │    │                 │
│   Browser       │    │   Angular 20    │    │  Spring Boot 3  │    │ PostgreSQL 16   │
│   Mobile        │<-->│   Material UI   │<-->│  Java 21        │<-->│                 │
│   API           │    │   Chart.js      │    │  JWT Security   │    │ Measurement     │
│                 │    │   Nginx         │    │  Swagger API    │    │ Series          │
│                 │    │                 │    │                 │    │ Users           │
└─────────────────┘    └─────────────────┘    └─────────────────┘    └─────────────────┘
                              Port 80              Port 8080              Port 5432
```

Docker Compose runs the frontend, backend, and database as separate containers. PostgreSQL uses persistent volumes, and Nginx serves the frontend and proxies requests to the API. The application has also been deployed on an Ubuntu 24.04 Azure VM (2 vCPU, 4 GB RAM); deployment availability is not guaranteed.

### Frontend structure

The Angular application uses standalone components organized by feature:

```text
src/app/
├── features/
│   ├── dashboard/
│   │   ├── components/
│   │   │   ├── chart/
│   │   │   ├── data-table/
│   │   │   └── filter-bar/
│   │   └── dashboard.component.ts
│   ├── admin/
│   │   ├── components/
│   │   │   ├── series-form/
│   │   │   ├── series-table/
│   │   │   ├── measurement-form/
│   │   │   ├── measurement-table/
│   │   │   ├── user-form/
│   │   │   └── user-table/
│   │   └── admin.component.ts
│   ├── auth/
│   └── change-password/
├── services/
├── guards/
├── interceptors/
└── shared/
```

### Backend structure

The Spring Boot application separates HTTP handling, business logic, persistence, and security:

```text
src/main/java/com/zaiuz/mesurement/backend/  # Existing Java package path (not yet renamed)
├── controllers/
│   ├── AuthController.java
│   ├── AdminController.java
│   ├── MeasurementController.java
│   └── SeriesController.java
├── domain/
│   ├── User.java
│   ├── Series.java
│   ├── Measurement.java
│   └── dto/
├── services/
│   ├── UserService.java
│   ├── SeriesService.java
│   └── MeasurementService.java
├── repositories/
│   ├── UserRepository.java
│   ├── SeriesRepository.java
│   └── MeasurementRepository.java
└── security/
    ├── SecurityConfig.java
    ├── JwtService.java
    └── JwtAuthFilter.java
```

Spring Data JPA handles persistence, while Spring Security and a custom JWT filter authenticate requests and enforce role-based access.

## Technology stack

| Layer | Technologies |
| --- | --- |
| Backend | Java 21, Spring Boot 3.5.6, Spring Data JPA, Spring Security, Gradle |
| API | REST, SpringDoc OpenAPI / Swagger UI, Bean Validation |
| Backend utilities | Lombok, ModelMapper |
| Frontend | Angular 20.3.0, Angular Material 20.2.9, TypeScript, SCSS |
| Charts | Chart.js 4.5.1, ng2-charts |
| Authentication | JWT, BCrypt, Angular HTTP interceptors and route guards |
| Database | PostgreSQL 16 |
| Deployment | Docker, Docker Compose, Nginx, Ubuntu / Azure VM |

## Data model

The core schema contains three entities:

| Entity | Main fields | Purpose |
| --- | --- | --- |
| `users` | `id` (UUID), `username`, `password`, `role` | Accounts and access roles |
| `series` | `id` (UUID), `name`, `min_value`, `max_value`, `color` | Measurement categories and display settings |
| `measurements` | `id` (UUID), `series_id` (UUID), `value`, `timestamp` | Timestamped readings belonging to a series |

A series has many measurements (`series` 1:N `measurements`). The entities also include audit fields such as `created_by`, `created_at`, and `updated_at`.

Sample data includes temperature (-30 °C to 60 °C) and humidity (0% to 100%) series, with seven days of readings at six-hour intervals.

## Getting started

### Requirements

- Docker and Docker Compose (recommended)
- For development without Docker: Java 21, Node.js 18+, and PostgreSQL

### Run with Docker Compose

The clone URL below uses the repository’s existing name until it is renamed on GitHub. The local checkout is named `sensorflow`.

```bash
git clone https://github.com/BacaSystem/ZAIUZ.git sensorflow
cd sensorflow/ops
docker compose up --build -d
```

Once the services have started:

| Service | Local URL |
| --- | --- |
| Web application | http://localhost:3000 |
| Backend API | http://localhost:8080 |
| Swagger UI | http://localhost:8080/swagger-ui.html |

The project includes seeded demo accounts for local evaluation. **Check the seed/configuration in your checkout for the current credentials and replace all default passwords before exposing the application to a network.** Do not use demo credentials in a public deployment.

## REST API

Interactive API documentation is available in [Swagger UI](http://localhost:8080/swagger-ui.html) when the backend is running.

### Authentication

| Method | Endpoint | Description | Authentication |
| --- | --- | --- | --- |
| `POST` | `/auth/login` | Sign in | No |
| `POST` | `/auth/register` | Register an account | No |
| `POST` | `/auth/logout` | Sign out | No |
| `GET` | `/auth/me` | Get current user | Yes |
| `POST` | `/auth/change-password` | Update password | Yes |

### Read-only measurement API

| Method | Endpoint | Description | Authentication |
| --- | --- | --- | --- |
| `GET` | `/api/series` | List measurement series | No |
| `GET` | `/api/series/{id}` | Get a series | No |
| `GET` | `/api/measurement` | Query measurements | No |

### Administrator API

All endpoints below require the administrator role.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/admin/series` | Create a series |
| `PUT` | `/api/admin/series/{id}` | Update a series |
| `DELETE` | `/api/admin/series/{id}` | Delete a series |
| `POST` | `/api/admin/measurements` | Create a measurement |
| `PUT` | `/api/admin/measurements/{id}` | Update a measurement |
| `DELETE` | `/api/admin/measurements/{id}` | Delete a measurement |
| `GET` | `/api/admin/users` | List users |
| `POST` | `/api/admin/users` | Create a user |
| `PUT` | `/api/admin/users/{id}` | Update a user |
| `DELETE` | `/api/admin/users/{id}` | Delete a user |

List endpoints support pagination and sorting using `page`, `size`, and `sort`. Measurement queries also support `seriesIds`, `from`, and `to` filters. Requests are validated and CORS is configured for the frontend origin.

## Authorization

Protected API requests use a signed JWT supplied in the `Authorization` header:

```http
Authorization: Bearer <token>
```

![JWT authentication flow](./documentation/jwt.png)

`USER` accounts can view measurements and series. `ADMIN` accounts can additionally manage measurements, series, and users. Authorization is enforced by the backend, not just frontend route guards.

## Contributing and support

For questions, bugs, or improvement suggestions, please open a GitHub issue. Before reporting a problem, check the API documentation and deployment configuration.

## License

Licensed under the MIT License. See [LICENSE](./LICENSE) for details.