# Architecture

## Approach

JGMRPWeb is a **modular monolith**. The repository contains the website, APIs, and logical services, but they are parts of one Spring Boot application. They are built, tested, configured, and deployed together. A logical service or subdomain is a code boundary, not a separately hosted microservice.

This keeps related functionality organized without requiring separate repositories, deployments, or service-to-service network calls. Avoid introducing independently deployable services unless there is a concrete need to scale, secure, operate, or release one separately.

## Code organization

Organize code by business capability, with each capability owning its API and implementation. For example:

```text
JGMRPWeb/
├── docs/
├── pom.xml
└── src/
    ├── main/java/com/geomrp/web/
    │   ├── JgmrpWebApplication.java
    │   ├── config/                 # Application-wide Spring configuration
    │   ├── shared/                 # Small, genuinely cross-cutting components
    │   ├── accounts/               # Example capability/module
    │   │   ├── api/                # HTTP controllers and request/response DTOs
    │   │   ├── application/        # Use cases and orchestration
    │   │   ├── domain/             # Capability-specific business rules
    │   │   └── persistence/        # JDBC queries and database mapping
    │   ├── maps/                   # Other capabilities follow the same pattern
    │   └── utilities/
    └── test/java/com/geomrp/web/   # Tests organized to mirror production code
```

The names above illustrate the layout; create capability packages as their features are implemented. Keep each module's business rules and persistence details within that module. Modules should interact through clear application-level interfaces rather than reaching into one another's internals. Keep `shared/` small so it does not become a place for unrelated code.

## APIs and subdomains

APIs belong to the capability that implements them. Keep HTTP controllers and API request/response types in that capability's `api/` package. Use consistent URL and versioning conventions for public APIs (for example, `/api/v1/...`) and keep internal implementation types out of the public API.

Subdomains are routing and presentation boundaries, not deployment boundaries. The single application can serve requests for multiple hostnames and route them to the appropriate pages or API controllers. Hostname routing can be handled by the reverse proxy, Spring configuration, or a combination of both; it does not require a separate repository or application per hostname.

## Database

The application connects to the Oracle database through JDBC. Keep SQL and persistence code inside the capability that owns the related data. Modules may use the same database and application deployment, but should not depend on one another's persistence implementation. Coordinate schema changes with the modules that own the affected data.

## Testing

Add JUnit Jupiter unit tests for new or changed behavior and keep tests alongside the corresponding modules under `src/test/java`. Prefer real collaborators or small fakes; use Mockito only when a dependency is impractical to use directly in a unit test. Unit tests should not require a running web server, network service, or Oracle instance unless the test is explicitly an integration test.
