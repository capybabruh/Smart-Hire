# SmartHire implementation checklist

This checklist turns the [client brief](./SmartHire_Client_Brief.md) into a practical build order. Check an item by changing `- [ ]` to `- [x]` in this file.

> **Recommended approach:** build a **modular monolith** and deliver it in small vertical slices. Start with the shared foundation and backend data/API contract, then connect Angular one feature at a time. Do not build the whole backend before checking it from the frontend.

## How to use this checklist

- [ ] Work through phases in order; do not start optional features until the MVP works.
- [ ] Keep each commit small and use messages such as `feat: add job creation API`.
- [ ] Run the relevant tests before checking an item.
- [ ] Treat every **phase gate** as a pause point: demonstrate the feature before moving on.
- [ ] Never commit passwords, database credentials, JWT secrets, or Google OAuth credentials.

## Target architecture

```text
Browser
  |
Angular application
  |  HTTPS / JSON REST API
Spring Boot modular monolith
  |-- auth and security
  |-- users and profiles
  |-- companies and jobs
  |-- applications and interviews
  |-- dashboards and audit logs
  |
PostgreSQL + file storage abstraction
```

The backend should normally follow this flow:

```text
Controller -> Service -> Repository -> PostgreSQL
                |
              Mapper
DTOs <------------------------------> Entities
```

## 0. Confirm the plan and development environment

- [ ] Read the full client brief once and keep the MVP/out-of-scope sections visible.
- [ ] Install and verify Git with `git --version`.
- [ ] Install one supported JDK and verify it with `java --version`.
- [ ] Decide on the project JDK version and make `pom.xml`, local Java, Docker, and CI use the same version.
- [ ] Verify Maven Wrapper with `./mvnw.cmd --version` on Windows.
- [ ] Install Node.js LTS, npm, and Angular CLI; record the versions in the README.
- [ ] Install Docker Desktop and verify `docker compose version`.
- [ ] Install an API client such as Postman or Bruno.
- [ ] Create a Git repository if this folder is not already inside one.
- [ ] Add/confirm `.gitignore` entries for IDE files, environment files, Angular build output, uploads, and secrets.
- [ ] Decide the repository layout: keep Spring Boot at the root and create Angular in `frontend/`.
- [ ] Fix the project naming convention before adding classes (recommended base package: `com.smarthire`).
- [ ] Add a short root `README.md` with prerequisites and placeholder run commands.

**Phase gate:** a fresh terminal can report the chosen Java, Maven, Node, Angular, and Docker versions.

## 1. Design before implementation

- [ ] Write the three roles: `ROLE_ADMIN`, `ROLE_RECRUITER`, and `ROLE_CANDIDATE`.
- [ ] Write a role-permission matrix for every planned endpoint/page.
- [ ] Draw an ER diagram for `User`, `Role`, `CandidateProfile`, `RecruiterProfile`, `Company`, `Job`, `Skill`, `Application`, `ApplicationHistory`, `Interview`, `RefreshToken`, and `AuditLog`.
- [ ] Mark every one-to-one, one-to-many, many-to-one, and many-to-many relationship.
- [ ] Decide record ownership rules (for example, only a job's company recruiters can edit that job).
- [ ] Define enums for job status, job type, experience level, application status, interview type, interview result, and account status.
- [ ] Define the allowed application transitions: `APPLIED -> SCREENING -> INTERVIEW -> OFFER -> HIRED`, with `REJECTED` and candidate `WITHDRAWN` paths.
- [ ] Decide which fields are required, optional, unique, and indexed.
- [ ] Decide money representation and currency rules for salary values.
- [ ] Decide time rules: store timestamps in UTC and display them in the user's local timezone.
- [ ] Define the first API contract: paths, request DTOs, response DTOs, validation, status codes, pagination, and error format.
- [ ] Decide how CVs and logos are stored behind a `FileStorageService` abstraction (local volume for development; cloud/object storage can come later).
- [ ] Define upload rules: PDF only for CVs, allowed image types for logos, size limits, generated filenames, and authorization to download.
- [ ] Separate **MVP** requirements from optional email, Redis, AI matching, and CI/CD work.

**Phase gate:** the ER diagram, role matrix, workflow, and API contract agree with one another.

## 2. Bootstrap local infrastructure and backend foundation

- [ ] Review `pom.xml` and retain only intentional dependencies.
- [ ] Add the PostgreSQL driver, Spring Data JPA, Web, Security, Validation, OAuth2 Client, and Actuator dependencies.
- [ ] Add a JWT library and SpringDoc OpenAPI dependencies.
- [ ] Add a database migration tool (recommended: Flyway) rather than relying on Hibernate to change production schema.
- [ ] Add test dependencies for JUnit 5, Mockito, MockMvc/Spring Security, and PostgreSQL integration testing (recommended: Testcontainers).
- [ ] Create `compose.yaml` with a PostgreSQL service, named volume, health check, database, and development user.
- [ ] Create local configuration using environment variables with safe development defaults where appropriate.
- [ ] Create profile files such as `application-dev.yml`, `application-test.yml`, and `application-prod.yml`.
- [ ] Configure JPA so migrations own the schema (`ddl-auto=validate` outside disposable tests).
- [ ] Configure UTC serialization and database timestamps.
- [ ] Create packages/modules for `config`, `auth`, `user`, `company`, `job`, `application`, `interview`, `audit`, `common`, and `storage`.
- [ ] Inside each feature, separate controller, service, repository, DTO, mapper, and entity responsibilities.
- [ ] Add common audit fields such as `createdAt`, `updatedAt`, and optimistic locking where concurrent updates matter.
- [ ] Add standard success/page response DTOs if they add value; do not wrap file downloads or every response unnecessarily.
- [ ] Add a structured error response containing timestamp, status, code, message, path, and field validation errors.
- [ ] Implement global exception handling with `@RestControllerAdvice`.
- [ ] Add `/actuator/health` and expose only safe actuator endpoints.
- [ ] Add the first Flyway migration and confirm it runs against a clean PostgreSQL database.
- [ ] Add a backend test profile that never connects to the development database.
- [ ] Run `./mvnw.cmd test` successfully.

**Phase gate:** PostgreSQL starts in Docker, Spring Boot starts without secrets in source control, migrations run, and the health endpoint is healthy.

## 3. Authentication and authorization backend

### Users, roles, and password login

- [ ] Implement `User` and `Role` entities plus repositories and migrations.
- [ ] Normalize email addresses and enforce a database-level unique constraint.
- [ ] Seed the three roles safely and idempotently.
- [ ] Create registration DTO validation (name, email, password length/strength, role rules).
- [ ] Ensure public registration cannot create an admin account.
- [ ] Hash passwords with BCrypt; never log or return password fields.
- [ ] Implement `POST /api/auth/register` for candidate registration.
- [ ] Define a controlled recruiter registration/onboarding rule.
- [ ] Implement `POST /api/auth/login` with generic invalid-credential errors.
- [ ] Add JWT access-token generation with subject, roles, issued time, and short expiry.
- [ ] Add a JWT authentication filter and stateless Spring Security configuration.

### Refresh, logout, and account controls

- [ ] Implement `RefreshToken` persistence using a hashed token value, owner, expiry, and revoked timestamp.
- [ ] Implement refresh-token rotation in `POST /api/auth/refresh`.
- [ ] Implement token revocation in `POST /api/auth/logout`.
- [ ] Reject expired, revoked, malformed, and reused refresh tokens.
- [ ] Decide secure token delivery (recommended: access token in memory and refresh token in a Secure, HttpOnly, SameSite cookie).
- [ ] Configure CSRF deliberately for the chosen cookie/token design.
- [ ] Block authentication for disabled or locked accounts.

### Authorization and Google login

- [ ] Enable method security and protect endpoints with roles plus ownership checks.
- [ ] Configure CORS from an environment-based frontend origin allow-list.
- [ ] Configure Google OAuth2 client ID, secret, scopes, and redirect URI using environment variables.
- [ ] Link Google identity to an existing account only through a safe, documented rule.
- [ ] Create new Google users with the candidate role by default.
- [ ] Implement OAuth2 success/failure handling and a safe redirect back to Angular.
- [ ] Document the exact Google Console setup without recording secret values.
- [ ] Add Swagger bearer-token security configuration.
- [ ] Test registration, login, refresh rotation, logout, locked users, missing tokens, wrong roles, and ownership denial.

**Phase gate:** candidate email/password login and Google login work; revoked tokens fail; candidate, recruiter, and admin test accounts cannot cross role boundaries.

## 4. Angular foundation and authentication UI

- [ ] Generate the Angular app in `frontend/` with routing, strict TypeScript, SCSS, and tests enabled.
- [ ] Add Angular Material and define a small theme/design system.
- [ ] Create `core/`, `shared/`, `features/`, and `layouts/` folders.
- [ ] Add environment configuration for the API base URL (no secrets in Angular files).
- [ ] Create typed API/error/page models matching the backend contract.
- [ ] Create public, candidate, recruiter, and admin layouts with responsive navigation.
- [ ] Add routes for home, jobs, login, register, and unauthorized/not-found pages.
- [ ] Build accessible reactive login and registration forms with inline validation.
- [ ] Implement `AuthService` and the chosen safe token-storage strategy.
- [ ] Add an HTTP interceptor for access tokens and one controlled refresh attempt on `401`.
- [ ] Prevent multiple simultaneous refresh requests from causing a refresh storm.
- [ ] Add auth and role guards for protected routes.
- [ ] Implement logout and clear client authentication state.
- [ ] Complete the Google login redirect/callback flow.
- [ ] Show user-friendly loading, success, and error states.
- [ ] Add component/service tests for forms, guards, and refresh behavior.

**Phase gate:** a user can register, log in both ways, refresh a session, log out, and is redirected away from pages for the wrong role.

## 5. Profiles and company management vertical slice

### Backend

- [ ] Create migrations/entities for `CandidateProfile`, `RecruiterProfile`, `Company`, and `Skill`.
- [ ] Implement candidate profile read/update endpoints with validation.
- [ ] Implement recruiter profile read/update endpoints with validation.
- [ ] Implement company create/read/update endpoints and recruiter-company membership rules.
- [ ] Implement reusable skill lookup/creation without case-sensitive duplicates.
- [ ] Implement authenticated CV PDF upload, replacement, download, and metadata storage.
- [ ] Implement company logo upload using the storage abstraction.
- [ ] Prevent path traversal, executable uploads, oversized files, and unauthorized downloads.
- [ ] Add service and endpoint tests, including ownership tests.

### Frontend

- [ ] Build candidate profile view/edit page including skills and CV status.
- [ ] Build recruiter profile view/edit page.
- [ ] Build company view/edit page for authorized recruiters.
- [ ] Add CV and logo upload controls with type/size guidance and progress/error feedback.

**Phase gate:** candidate and recruiter profiles persist after reload, and protected files can be accessed only by authorized users.

## 6. Job management and public job search vertical slice

### Backend

- [ ] Create the `Job` schema/entity and its company, recruiter, and skill relationships.
- [ ] Add DTOs and mapping so JPA entities are never returned directly.
- [ ] Implement recruiter job creation as `DRAFT`.
- [ ] Implement recruiter job update with company ownership checks.
- [ ] Implement explicit publish and close actions with valid transition rules.
- [ ] Decide whether deletion means hard delete or archive; preserve jobs referenced by applications.
- [ ] Implement public `GET /api/jobs` showing only published, non-expired jobs.
- [ ] Implement public `GET /api/jobs/{id}` with appropriate visibility rules.
- [ ] Add keyword, location, skill, job type, experience level, and salary filters.
- [ ] Add pagination, deterministic sorting, maximum page size, and documented defaults.
- [ ] Add database indexes for common filters after inspecting generated queries.
- [ ] Prevent recruiters from editing another company's jobs.
- [ ] Add unit/integration tests for CRUD, transitions, filters, pagination, and authorization.

### Frontend

- [ ] Build public job list cards, pagination, filters, loading, empty, and error states.
- [ ] Build the public job details page.
- [ ] Keep filters in URL query parameters so searches can be bookmarked/shared.
- [ ] Build recruiter job list and status controls.
- [ ] Build reusable create/edit job reactive forms.
- [ ] Add confirmation for close/archive actions.

**Phase gate:** a recruiter can draft and publish a job; a candidate can find it using filters; unauthorized edits fail on the server.

## 7. Job applications and status history vertical slice

### Backend

- [ ] Create `Application` and `ApplicationHistory` migrations/entities.
- [ ] Add a database unique constraint on candidate plus job to prevent duplicate applications.
- [ ] Implement `POST /api/jobs/{jobId}/apply` with CV selection/upload and cover letter.
- [ ] Reject applications to draft, closed, or expired jobs.
- [ ] Implement candidate `GET /api/applications/my` with pagination.
- [ ] Implement candidate application details with history visible at an appropriate level.
- [ ] Implement candidate withdrawal only when the current status permits it.
- [ ] Implement recruiter applicants-by-job endpoint with status, skill, and date filters.
- [ ] Implement recruiter application details with authorized CV access.
- [ ] Implement status changes using an explicit state-transition service.
- [ ] Record every status change atomically with previous status, new status, actor, time, and optional note.
- [ ] Add recruiter notes with clear rules about whether candidates can see them.
- [ ] Prevent candidate enumeration and cross-company access.
- [ ] Add concurrency protection so two updates cannot silently overwrite each other.
- [ ] Test duplicate apply, ownership, every allowed/forbidden transition, withdrawal, and history creation.

### Frontend

- [ ] Add an apply action/form to job details with duplicate/closed-job feedback.
- [ ] Build candidate “My Applications” list and details/timeline.
- [ ] Add candidate withdrawal with confirmation.
- [ ] Build recruiter applicant list with filters.
- [ ] Build recruiter application/CV review page, notes, and valid next-status actions.

**Phase gate:** a candidate applies once and tracks the result; the correct recruiter moves it through valid stages; every change appears in immutable history.

## 8. Interview management vertical slice

### Backend

- [ ] Create the `Interview` schema/entity and enums for type/result/status.
- [ ] Implement recruiter create, read, update, cancel/delete endpoints.
- [ ] Require an eligible application status before scheduling.
- [ ] Validate future date/time, duration, timezone, URL/location, and assigned recruiter.
- [ ] Prevent obvious overlapping interviews for the candidate and recruiter.
- [ ] Decide whether scheduling automatically moves an application to `INTERVIEW` and enforce it transactionally.
- [ ] Implement interview result recording and optional notes.
- [ ] Implement candidate upcoming/past interview endpoints with only candidate-visible fields.
- [ ] Add ownership, validation, conflict, and endpoint tests.

### Frontend

- [ ] Build recruiter interview create/edit form.
- [ ] Build recruiter interview list/calendar-style view.
- [ ] Build interview result controls.
- [ ] Build candidate interview schedule with local date/time and meeting/location details.

**Phase gate:** a recruiter can schedule and record an interview, and only the relevant candidate can see the appropriate schedule details.

## 9. Admin, audit, and dashboards

### Admin and audit backend

- [ ] Create append-only `AuditLog` storage for security-sensitive and administrative actions.
- [ ] Record actor, action, target type/id, timestamp, outcome, and safe metadata without tokens/passwords.
- [ ] Implement paginated admin user/recruiter/candidate listing and filters.
- [ ] Implement activate/deactivate and lock/unlock actions.
- [ ] Implement controlled role management that prevents accidental removal of the last admin.
- [ ] Implement admin job listing and removal/archive with an audit reason.
- [ ] Implement paginated/filterable audit log endpoint.
- [ ] Test that every admin endpoint rejects non-admin users.

### Statistics backend

- [ ] Implement recruiter metrics: active jobs, applicants, applications by status/date, upcoming interviews, and hires.
- [ ] Scope recruiter statistics to the correct company.
- [ ] Implement admin metrics: users by role, total/active jobs, and total applications.
- [ ] Optimize aggregate queries and avoid loading full tables into memory.

### Frontend

- [ ] Build recruiter dashboard with summary cards, simple charts, and useful empty states.
- [ ] Build admin dashboard.
- [ ] Build admin user management with status and role controls.
- [ ] Build admin job moderation page.
- [ ] Build audit log viewer with pagination and filters.

**Phase gate:** recruiters see only company statistics; admins can manage accounts/jobs and every privileged change is auditable.

## 10. API quality, security hardening, and observability

- [ ] Document every public API with OpenAPI summaries, DTO schemas, validation, response codes, and examples.
- [ ] Verify Swagger UI can authenticate with a JWT in development.
- [ ] Return correct HTTP status codes (`200`, `201`, `204`, `400`, `401`, `403`, `404`, `409`).
- [ ] Make all list endpoints paginated and cap page size.
- [ ] Ensure validation errors are consistent and do not reveal internals.
- [ ] Review every object-by-ID endpoint for broken object-level authorization (IDOR).
- [ ] Review logs and error responses for passwords, tokens, CV contents, and personal data.
- [ ] Configure secure response headers and production HTTPS assumptions.
- [ ] Add rate limiting or documented upstream limits for login, registration, refresh, and uploads.
- [ ] Validate MIME type and file signature, not only filename extension.
- [ ] Add request correlation IDs and structured application logging.
- [ ] Define production-safe health/readiness checks.
- [ ] Review JPA queries for N+1 problems and unbounded collections.
- [ ] Add sensible indexes based on actual search and join queries.
- [ ] Run dependency/security scanning and resolve important findings.

**Phase gate:** the API is fully discoverable in Swagger, common attacks/authorization mistakes have been reviewed, and logs contain no secrets.

## 11. Testing and acceptance

- [ ] Unit-test important services: registration, tokens, job creation, applying, transitions, interviews, and admin actions.
- [ ] Integration-test repositories and migrations with PostgreSQL rather than relying only on an in-memory database.
- [ ] Integration-test critical REST flows with MockMvc and Spring Security.
- [ ] Test validation boundary cases and standardized errors.
- [ ] Test candidate, recruiter, admin, anonymous, disabled, and locked-user permissions.
- [ ] Test CV upload/download security and size/type limits.
- [ ] Test Angular services, guards, interceptors, forms, and important components.
- [ ] Add a small end-to-end happy path: register -> publish job -> apply -> screen -> interview -> offer -> hire.
- [ ] Add negative end-to-end paths: duplicate apply, wrong company, invalid transition, expired job, and revoked refresh token.
- [ ] Create repeatable seed/demo data without real personal information.
- [ ] Run all tests from a clean checkout.
- [ ] Manually complete the candidate journey from the brief.
- [ ] Manually complete the recruiter journey from the brief.
- [ ] Manually complete the administrator journey from the brief.

**Phase gate:** automated tests pass and all three manual user journeys meet the client brief.

## 12. Docker, documentation, and release

- [ ] Add a multi-stage backend `Dockerfile` running as a non-root user.
- [ ] Add a multi-stage Angular `Dockerfile` and production web-server configuration.
- [ ] Extend `compose.yaml` with `frontend`, `backend`, and `postgres` services.
- [ ] Add health checks and startup dependencies without relying only on fixed delays.
- [ ] Persist database data and development uploads in named volumes.
- [ ] Add `.dockerignore` files and ensure secrets are passed at runtime.
- [ ] Confirm `docker compose up --build` starts the entire clean system.
- [ ] Confirm Angular calls the backend correctly through the chosen proxy/domain setup.
- [ ] Confirm OAuth redirect URLs and CORS work in the container setup.
- [ ] Document prerequisites, environment variables, setup, migrations, tests, demo accounts, and troubleshooting in `README.md`.
- [ ] Add architecture, ER, authentication-flow, and deployment diagrams.
- [ ] Add screenshots of the main candidate, recruiter, and admin flows.
- [ ] Add an API/Postman collection or link clearly to Swagger.
- [ ] Document backup/restore basics for PostgreSQL and uploaded files.
- [ ] Perform a final clean-machine or clean-VM setup test using only the README.
- [ ] Tag an MVP release after the final acceptance checklist passes.

**Phase gate / MVP done:** a new developer can clone the repository, provide documented environment variables, run one Docker Compose command, and complete all required user journeys.

## 13. Client-brief final acceptance checklist

- [ ] Email/password registration and login work securely.
- [ ] Google OAuth2 login works.
- [ ] JWT access and rotating/revocable refresh tokens work.
- [ ] Candidate, recruiter, and admin authorization is enforced by the backend.
- [ ] Candidate and recruiter profiles work.
- [ ] Company management works.
- [ ] Recruiter job create/edit/publish/close works.
- [ ] Public job search, filters, sorting, and pagination work.
- [ ] Candidate CV upload, application, tracking, and withdrawal work.
- [ ] Duplicate applications are prevented.
- [ ] Recruiter filtering, notes, status workflow, and application history work.
- [ ] Interview scheduling, visibility, updates, and results work.
- [ ] Recruiter and admin dashboards show correct scoped statistics.
- [ ] Admin user/job management and audit logs work.
- [ ] Validation, standard errors, CORS, file security, and ownership checks work.
- [ ] Swagger/OpenAPI is accurate and usable.
- [ ] Important business logic and critical API flows have passing tests.
- [ ] Angular integrates successfully with Spring Boot.
- [ ] PostgreSQL schema is managed by repeatable migrations.
- [ ] The full application runs with Docker Compose.
- [ ] README setup instructions, architecture, and screenshots are complete.

## 14. Optional work only after MVP

- [ ] Add email notifications using an outbox/retry-friendly design.
- [ ] Add Redis only after measuring a real caching/rate-limit need.
- [ ] Add privacy-aware AI CV matching as advice, never an automatic hiring decision.
- [ ] Add GitHub Actions for backend tests, frontend tests/build, and container builds.
- [ ] Deploy to a staging environment with HTTPS, backups, monitoring, and secret management.
- [ ] Run accessibility, performance, and browser compatibility audits.

## Suggested first working session

- [ ] Choose and align the Java/Spring Boot/Angular/Node versions.
- [ ] Initialize/verify Git and create the root README.
- [ ] Create the ER diagram and role-permission matrix.
- [ ] Add PostgreSQL to `compose.yaml` and connect Spring Boot.
- [ ] Add Flyway and create the first `users`/`roles` migration.
- [ ] Make the health endpoint and first database integration test pass.

When that session is complete, continue with **Phase 3: Authentication and authorization backend**.
