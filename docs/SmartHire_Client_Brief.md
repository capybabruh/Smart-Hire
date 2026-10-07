# SmartHire – Client Brief

## 1. Project Overview

**Project Name:** SmartHire  
**Project Type:** Full-stack Recruitment & Applicant Tracking System  
**Primary Goal:** Build a modern recruitment platform that helps companies manage job postings, candidate applications, interviews, and hiring workflows in one centralized system.

SmartHire is designed for three main user groups:

- **Candidates** who search and apply for jobs.
- **Recruiters** who manage job posts and candidate pipelines.
- **Administrators** who manage users, permissions, and system activity.

The project should demonstrate practical full-stack software engineering skills using **Java Spring Boot** for the backend and **Angular** for the frontend.

---

## 2. Business Background

Many small and medium-sized companies still manage recruitment using spreadsheets, email, and separate communication tools. This makes it difficult to:

- Track candidate status.
- Manage multiple job openings.
- Coordinate interview schedules.
- Maintain candidate information.
- Control access between recruiters and administrators.
- Review recruitment performance.

SmartHire aims to solve these problems by providing a centralized Applicant Tracking System (ATS).

---

## 3. Project Objectives

The system should allow users to:

1. Register and authenticate securely.
2. Sign in using Google OAuth2.
3. Manage user roles and permissions.
4. Create and manage job postings.
5. Search and filter available jobs.
6. Apply for jobs and upload CV files.
7. Track application progress.
8. Manage candidates through recruitment stages.
9. Schedule and manage interviews.
10. Provide recruitment statistics through dashboards.
11. Secure APIs using JWT and Spring Security.
12. Provide API documentation using Swagger/OpenAPI.
13. Run the application using Docker.

---

## 4. Target Users

### 4.1 Candidate

A candidate is a user looking for job opportunities.

Candidate should be able to:

- Register an account.
- Login using email/password.
- Login using Google OAuth2.
- Create and update personal profile.
- Upload a CV in PDF format.
- Browse available jobs.
- Search jobs by keyword.
- Filter jobs by location, skill, salary range, and job type.
- View job details.
- Apply for a job.
- View submitted applications.
- Track current application status.
- View interview schedules.
- Withdraw an application before final processing.

---

### 4.2 Recruiter

A recruiter represents a company and manages recruitment activities.

Recruiter should be able to:

- Login securely.
- Manage recruiter profile.
- Create job postings.
- Edit job postings.
- Publish or close job postings.
- View applicants for each job.
- Review candidate profiles and CVs.
- Update candidate application status.
- Add recruiter notes.
- Schedule interviews.
- Update interview results.
- Filter candidates by status, skills, and application date.
- View recruitment statistics.

---

### 4.3 Administrator

An administrator manages the overall platform.

Administrator should be able to:

- View all users.
- View recruiters and candidates.
- Activate or deactivate accounts.
- Lock or unlock accounts.
- Manage roles and permissions.
- View all job postings.
- Remove inappropriate job postings.
- View system audit logs.
- View high-level system statistics.

---

## 5. Core Recruitment Workflow

Candidate application statuses should follow a defined workflow:

```text
APPLIED
   ↓
SCREENING
   ↓
INTERVIEW
   ↓
OFFER
   ↓
HIRED
```

An application may be moved to:

```text
REJECTED
```

at appropriate stages.

Optional additional status:

```text
WITHDRAWN
```

for candidates who withdraw their application.

---

## 6. Functional Requirements

## 6.1 Authentication

The application must support:

- Email/password registration.
- Email/password login.
- Password hashing using BCrypt.
- JWT access token.
- Refresh token.
- Secure logout.
- Refresh token revocation.
- Google OAuth2 login.
- Role-based access control.

Recommended roles:

```text
ROLE_ADMIN
ROLE_RECRUITER
ROLE_CANDIDATE
```

---

## 6.2 Authorization

The backend must restrict endpoints based on user role.

Examples:

```text
Candidate:
GET  /api/jobs
POST /api/jobs/{jobId}/apply
GET  /api/applications/my

Recruiter:
POST  /api/recruiter/jobs
PUT   /api/recruiter/jobs/{id}
PATCH /api/applications/{id}/status

Admin:
GET   /api/admin/users
PATCH /api/admin/users/{id}/status
GET   /api/admin/audit-logs
```

Spring Security should be used for authorization.

Example:

```java
@PreAuthorize("hasRole('RECRUITER')")
```

---

## 6.3 User Profile Management

### Candidate Profile

Suggested information:

- Full name
- Email
- Phone number
- Address
- Date of birth
- Biography
- Education
- Years of experience
- Skills
- LinkedIn URL
- GitHub URL
- CV file

### Recruiter Profile

Suggested information:

- Full name
- Job title
- Company
- Phone number
- Profile image

---

## 6.4 Company Management

A company should contain:

- Company name
- Description
- Website
- Industry
- Company size
- Address
- Logo

A company can have multiple recruiters and multiple job postings.

---

## 6.5 Job Management

Each job should contain:

- Job title
- Job description
- Requirements
- Responsibilities
- Location
- Salary minimum
- Salary maximum
- Job type
- Experience level
- Required skills
- Application deadline
- Created date
- Published date
- Status

Suggested job statuses:

```text
DRAFT
PUBLISHED
CLOSED
```

Suggested job types:

```text
FULL_TIME
PART_TIME
INTERNSHIP
CONTRACT
REMOTE
```

Recruiters should be able to create, update, publish, and close jobs.

---

## 6.6 Job Search

Candidates should be able to search and filter jobs.

Supported filters should include:

- Keyword
- Location
- Skill
- Job type
- Experience level
- Salary range

Pagination should be implemented.

Example:

```http
GET /api/jobs?page=0&size=10
```

Optional filters:

```http
GET /api/jobs?keyword=java&location=Ho%20Chi%20Minh
```

---

## 6.7 Job Application

Candidates should be able to submit applications.

An application should contain:

- Candidate
- Job
- CV
- Cover letter
- Application date
- Current status

The same candidate should not be allowed to apply for the same job multiple times unless business rules explicitly allow it.

---

## 6.8 Application History

Every important application status change should be recorded.

Example:

```text
Candidate applied
        ↓
Recruiter moved candidate to SCREENING
        ↓
Recruiter scheduled INTERVIEW
        ↓
Recruiter moved candidate to OFFER
```

Application history should store:

- Previous status
- New status
- Changed by
- Timestamp
- Optional note

---

## 6.9 Interview Management

Recruiters should be able to schedule interviews.

Interview information should include:

- Candidate
- Job
- Recruiter
- Interview date
- Start time
- Interview type
- Location or meeting URL
- Interview notes
- Interview result

Suggested interview types:

```text
ONLINE
ONSITE
PHONE
```

---

## 6.10 Dashboard

### Recruiter Dashboard

Suggested metrics:

- Total active jobs
- Total applicants
- Applicants by status
- Upcoming interviews
- Hired candidates
- Applications received by date

### Admin Dashboard

Suggested metrics:

- Total users
- Total recruiters
- Total candidates
- Total jobs
- Active jobs
- Total applications

---

## 7. Database Requirements

Recommended database:

```text
PostgreSQL
```

Suggested core entities:

```text
User
Role
CandidateProfile
RecruiterProfile
Company
Job
Skill
Application
ApplicationHistory
Interview
RefreshToken
AuditLog
```

---

## 8. Suggested Database Relationships

```text
User
 ├── CandidateProfile
 ├── RecruiterProfile
 └── Roles

Company
 ├── Recruiters
 └── Jobs

Job
 ├── Skills
 └── Applications

Application
 ├── Candidate
 ├── Job
 ├── ApplicationHistory
 └── Interview
```

Relationships should demonstrate:

- One-to-One
- One-to-Many
- Many-to-One
- Many-to-Many

Spring Data JPA and Hibernate should be used for persistence.

---

## 9. Backend Requirements

### Technology

```text
Java 21+
Spring Boot
Spring Web
Spring Security
Spring Data JPA
Hibernate
Spring Validation
Spring OAuth2 Client
JWT
Maven
PostgreSQL
```

### Recommended Backend Architecture

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

Recommended package structure:

```text
com.smarthire
│
├── config
├── controller
├── dto
├── entity
├── exception
├── mapper
├── repository
├── security
├── service
└── util
```

Controllers should not contain business logic.

Business logic should primarily be implemented in the service layer.

---

## 10. REST API Requirements

The backend should follow REST conventions.

Examples:

### Authentication

```http
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
GET  /oauth2/authorization/google
```

### Jobs

```http
GET    /api/jobs
GET    /api/jobs/{id}
POST   /api/jobs
PUT    /api/jobs/{id}
DELETE /api/jobs/{id}
```

### Applications

```http
POST  /api/jobs/{jobId}/apply
GET   /api/applications/my
GET   /api/applications/{id}
PATCH /api/applications/{id}/status
```

### Interviews

```http
POST   /api/interviews
GET    /api/interviews/{id}
PUT    /api/interviews/{id}
DELETE /api/interviews/{id}
```

Proper HTTP status codes should be returned.

Examples:

```text
200 OK
201 Created
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
500 Internal Server Error
```

---

## 11. API Response Standard

Recommended successful response format:

```json
{
  "success": true,
  "message": "Job created successfully",
  "data": {}
}
```

Recommended error response:

```json
{
  "success": false,
  "message": "Job not found",
  "timestamp": "2026-10-07T12:00:00"
}
```

A global exception handler should be implemented using:

```java
@RestControllerAdvice
```

---

## 12. Validation

Backend validation should be implemented using Jakarta Validation.

Examples:

```java
@NotBlank
@Email
@Size
@Min
@Max
@Pattern
```

Example DTO:

```java
public class RegisterRequest {

    @NotBlank
    private String fullName;

    @Email
    @NotBlank
    private String email;

    @Size(min = 8)
    private String password;
}
```

---

## 13. Frontend Requirements

### Technology

```text
Angular
TypeScript
Angular Router
Angular HttpClient
Angular Reactive Forms
Angular Material
```

Suggested Angular structure:

```text
src/app
│
├── core
│   ├── guards
│   ├── interceptors
│   └── services
│
├── shared
│   ├── components
│   └── models
│
├── features
│   ├── auth
│   ├── jobs
│   ├── candidate
│   ├── recruiter
│   └── admin
│
└── layouts
```

---

## 14. Angular Security Requirements

The frontend should implement:

- Authentication service.
- JWT interceptor.
- Auth guard.
- Role guard.
- Automatic token attachment.
- Refresh token handling.
- Logout when refresh token is no longer valid.

Example request:

```http
Authorization: Bearer <access_token>
```

The frontend must not be considered a security boundary. Authorization must always be enforced by the Spring Boot backend.

---

## 15. Suggested Pages

### Public

```text
Home
Job List
Job Details
Login
Register
```

### Candidate

```text
Candidate Dashboard
My Profile
My Applications
Interview Schedule
```

### Recruiter

```text
Recruiter Dashboard
Job Management
Create Job
Edit Job
Candidate List
Application Details
Interview Management
```

### Admin

```text
Admin Dashboard
User Management
Job Management
Audit Logs
```

---

## 16. Security Requirements

The application should implement:

- Spring Security.
- JWT authentication.
- Short-lived access tokens.
- Refresh tokens.
- Refresh token revocation.
- BCrypt password hashing.
- OAuth2 Google login.
- Role-based authorization.
- Input validation.
- CORS configuration.
- Secure error handling.
- Protection against unauthorized resource access.

Sensitive information must not be stored in source code.

Environment variables should be used for:

```text
Database credentials
JWT secret
OAuth2 client ID
OAuth2 client secret
```

---

## 17. Swagger / OpenAPI

The backend should expose interactive API documentation.

Recommended tool:

```text
SpringDoc OpenAPI
```

Swagger should allow developers to:

- View endpoints.
- View request payloads.
- View response schemas.
- Test APIs.
- Authenticate using JWT.

---

## 18. Testing Requirements

Minimum recommended backend tests:

### Unit Tests

Use:

```text
JUnit 5
Mockito
```

Test important service methods.

Examples:

- User registration.
- Job creation.
- Job application.
- Application status update.

### Integration Tests

Recommended:

```text
Spring Boot Test
MockMvc
```

Test important REST endpoints.

---

## 19. Docker Requirements

The application should be containerized.

Recommended Docker Compose services:

```text
frontend
backend
postgres
```

Example architecture:

```text
Angular
   │
   ↓
Spring Boot
   │
   ↓
PostgreSQL
```

The entire project should be startable with a command similar to:

```bash
docker compose up -d
```

---

## 20. Version Control

The project should use Git and GitHub.

Recommended branches:

```text
main
develop
feature/authentication
feature/job-management
feature/application-management
feature/angular-login
```

Recommended workflow:

```text
feature branch
      ↓
develop
      ↓
main
```

Commit messages should clearly describe changes.

Examples:

```text
feat: implement JWT authentication
feat: add job creation API
fix: prevent duplicate job applications
test: add application service tests
docs: update API documentation
```

---

## 21. Non-Functional Requirements

### Performance

- Common API requests should respond quickly under normal development load.
- Database queries should avoid unnecessary repeated queries.
- Pagination should be used for large lists.
- Database indexes should be considered for frequently searched columns.

### Maintainability

- Follow layered architecture.
- Use DTOs instead of directly exposing JPA entities.
- Avoid duplicated code.
- Keep controllers lightweight.
- Centralize exception handling.

### Scalability

The first version should use a modular monolith architecture.

Microservices are not required for the MVP.

The architecture should still allow future services such as notification or AI matching to be extracted later.

---

## 22. Optional Advanced Features

These features are outside the MVP and can be implemented after the core system is stable.

### Email Notification

Send emails when:

- Application is submitted.
- Application status changes.
- Interview is scheduled.
- Candidate receives an offer.

Possible technology:

```text
Spring Mail
```

---

### Redis

Possible use cases:

- Cache frequently accessed jobs.
- Store temporary authentication data.
- Rate limiting.
- Session-related data.

---

### AI CV Matching

Future extension:

SmartHire can include an AI-assisted candidate ranking feature.

Example:

```text
Java Backend Developer

Candidate A    87%
Candidate B    82%
Candidate C    74%
```

Possible matching factors:

- Required skills
- Candidate skills
- Experience
- Education
- Job description
- Previous projects

Recommended architecture:

```text
Angular
   │
   ↓
Spring Boot
   │
   ├── PostgreSQL
   │
   └── AI Matching Service
          │
          ↓
        Python
```

AI matching should be treated as a recommendation tool rather than an automatic hiring decision system.

---

## 23. CI/CD – Optional

Possible CI/CD workflow:

```text
Developer pushes code
        ↓
GitHub Actions
        ↓
Build backend
        ↓
Run tests
        ↓
Build frontend
        ↓
Build Docker image
        ↓
Deploy
```

This feature is recommended after the core application is completed.

---

## 24. MVP Scope

The minimum version required for the project should include:

- User registration.
- Login.
- JWT authentication.
- Refresh token.
- Candidate and recruiter roles.
- Google OAuth2 login.
- Candidate profile.
- Recruiter profile.
- Company management.
- Job CRUD.
- Job search.
- Job application.
- Application status workflow.
- Interview management.
- Angular frontend.
- PostgreSQL database.
- Swagger API documentation.
- Docker setup.

---

## 25. Out of Scope for MVP

The following features are not required in the first release:

- Kubernetes.
- Kafka.
- Full microservice architecture.
- Elasticsearch.
- Real-time chat.
- Payment integration.
- Native mobile application.
- Advanced recommendation engine.

These may be added later if the MVP is completed successfully.

---

## 26. Definition of Done

The project can be considered portfolio-ready when:

- Backend APIs work correctly.
- Angular frontend communicates successfully with Spring Boot.
- JWT authentication works.
- Google OAuth2 login works.
- Role-based authorization works.
- Candidate can apply for jobs.
- Recruiter can manage jobs and applications.
- Application workflow works.
- PostgreSQL relationships are correctly implemented.
- Validation and exception handling are implemented.
- API documentation is available.
- Main business logic has tests.
- Application runs using Docker.
- Repository contains setup instructions.
- README contains screenshots and architecture information.
- Application is deployed or can be run locally using documented steps.

---

## 27. Recommended Technology Stack

### Backend

```text
Java 21+
Spring Boot
Spring Web
Spring Security
Spring Data JPA
Hibernate
Spring Validation
Spring OAuth2 Client
JWT
Maven
JUnit 5
Mockito
```

### Frontend

```text
Angular
TypeScript
Angular Router
Angular HttpClient
Angular Material
```

### Database

```text
PostgreSQL
```

### Development & DevOps

```text
Git
GitHub
Docker
Docker Compose
Postman
Swagger / OpenAPI
GitHub Actions (optional)
```

---

## 28. Portfolio Value

SmartHire is intended to demonstrate practical knowledge of:

```text
Java Core
Object-Oriented Programming
Spring Boot
REST API Design
Spring Security
JWT
OAuth2
Role-Based Access Control
JPA / Hibernate
SQL
PostgreSQL
Angular
TypeScript
Frontend–Backend Integration
Docker
Git
Testing
Software Architecture
```

The project should focus on depth and correct implementation instead of adding unnecessary technologies.

---

## 29. Suggested Resume Description

**SmartHire – Recruitment & Applicant Tracking System**

Developed a full-stack recruitment platform using Java Spring Boot, Angular, and PostgreSQL. Implemented RESTful APIs, JWT authentication, Google OAuth2 login, refresh tokens, and role-based authorization for Admin, Recruiter, and Candidate users. Built job posting, candidate application tracking, interview scheduling, and recruitment workflow features using Spring Data JPA and Hibernate. Containerized the application with Docker and documented APIs using Swagger/OpenAPI.

**Tech Stack:** Java, Spring Boot, Spring Security, JWT, OAuth2, Angular, TypeScript, PostgreSQL, Hibernate, Docker, REST API, Swagger.

---

## 30. Final Product Vision

The final SmartHire system should provide the following experience:

```text
Candidate
   ↓
Search Jobs
   ↓
Apply
   ↓
Track Application
   ↓
Attend Interview
   ↓
Receive Offer
```

while recruiters manage the same workflow through:

```text
Recruiter
   ↓
Create Job
   ↓
Review Applicants
   ↓
Screen Candidates
   ↓
Schedule Interviews
   ↓
Offer / Hire
```

SmartHire should primarily demonstrate a realistic, secure, maintainable full-stack application suitable for a Java Backend or Full-stack Fresher portfolio.
