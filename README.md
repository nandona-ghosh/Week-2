# StockPilot Product Catalog API

A small Spring Boot REST API for the product catalogue proposed in the Week 1 StockPilot design. It supports creating, listing, reading, replacing and deleting products. This Week 2 version intentionally focuses on basic CRUD: authentication, stock movements, archive semantics and PostgreSQL from the broader design are future work. Do not expose it publicly as-is.

## Requirements and setup

- Java 17 or newer (a JDK), and internet access on the first build for Maven and dependencies.
- Unzip the project, open a terminal in the `stockpilot-api` folder, and run:

```sh
# macOS / Linux
./mvnw spring-boot:run
# Windows PowerShell
.\mvnw.cmd spring-boot:run
```

The API starts at `http://localhost:8080`. If necessary make the Unix wrapper executable with `chmod +x mvnw`. The app uses an H2 file database in `./data/`; data persists between restarts. To reset the local data, stop the application and remove that directory. The initial database is empty. No credentials are needed for the API.

Run the automated endpoint tests with `./mvnw test` (or `.\mvnw.cmd test` on Windows). Tests use a separate in-memory database. Spring Boot 3.5.16, Spring Web, Spring Data JPA, Bean Validation, H2 and JUnit 5 are used.

## Product fields

| Field | Rules |
| --- | --- |
| `id` | Server-generated numeric identifier, returned in responses |
| `sku` | Required, nonblank, unique, at most 64 characters; case-sensitive |
| `name` | Required, nonblank, at most 200 characters |
| `unit` | Required, nonblank, at most 32 characters |
| `reorderPoint` | Required whole number, zero or greater |

String fields are trimmed before storage. PUT replaces every writable field, so include all four. IDs are generated and may differ from the examples below.

## Endpoints

| Method | Path | Result |
| --- | --- | --- |
| POST | `/api/v1/products` | 201 Created, product JSON and `Location` header |
| GET | `/api/v1/products` | 200 OK, array of products ordered by id |
| GET | `/api/v1/products/{id}` | 200 OK, or 404 if absent |
| PUT | `/api/v1/products/{id}` | 200 OK with updated product, or 404 if absent |
| DELETE | `/api/v1/products/{id}` | 204 No Content, or 404 if absent |

## Try it

These commands use `curl`; on Windows PowerShell use `curl.exe`. Run them in order, replacing `1` with the ID returned by the create request if needed.

```sh
curl -i -X POST http://localhost:8080/api/v1/products -H "Content-Type: application/json" -d '{"sku":"PEN-01","name":"Blue pen","unit":"piece","reorderPoint":5}'
curl -i http://localhost:8080/api/v1/products
curl -i http://localhost:8080/api/v1/products/1
curl -i -X PUT http://localhost:8080/api/v1/products/1 -H "Content-Type: application/json" -d '{"sku":"PEN-01","name":"Black pen","unit":"piece","reorderPoint":8}'
curl -i -X DELETE http://localhost:8080/api/v1/products/1
```

Example response to create (the Location host and ID depend on the request):

```http
HTTP/1.1 201 Created
Location: http://localhost:8080/api/v1/products/1
Content-Type: application/json

{"id":1,"sku":"PEN-01","name":"Blue pen","unit":"piece","reorderPoint":5}
```

A list returns an array of these objects; PUT returns the updated object. DELETE returns an empty body. Invalid fields return 400 with a JSON error such as `{"code":"VALIDATION_ERROR","message":"Please check the supplied fields","details":{"sku":"must not be blank"}}`. Malformed JSON returns 400, missing products return 404, and duplicate SKUs return 409. Error messages contain no stack trace. See the tests for more examples.

## Project layout

`src/main/java` contains the controller, service, entity, repository and error handler; `src/main/resources` contains local configuration. `src/test` contains endpoint integration tests exercising the real Spring MVC, service and database layers. The H2 schema is managed by Hibernate for this exercise; use versioned migrations, access control, a production database and deployment configuration before a real deployment.
