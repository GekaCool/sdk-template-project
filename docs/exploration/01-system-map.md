# Module 1 — System map

## a. My own diagram

```mermaid
flowchart LR
    subgraph HOST["Client"]
        B["Browser"]
        FE["frontend<br/>Vanilla JS + Vite<br/>:5173"]
    end

    subgraph NET["Server-side"]
        P["products-service<br/>PHP 8.4 / Slim + Apache<br/>container :80"]
        U["users-service<br/>Python / FastAPI + uvicorn<br/>container :8000"]
        O["orders-service<br/>Java / Spring Boot<br/>container :8080"]
        DB[("database<br/>PostgreSQL 18<br/>:5432<br/>volume: postgres_data")]
        M["migration-runner<br/>Node + Prisma<br/>runs once, then exits"]
    end

    B -- "1. GET / (HTML + JS)<br/>localhost:5173" --> FE
    FE -- "2. GET /products<br/>localhost:8082 → :80" --> P
    FE -- "3. GET /users<br/>localhost:8000 → :8000" --> U
    FE -- "4. GET /orders<br/>localhost:8083 → :8080" --> O

    P -- "PDO<br/>SELECT * FROM &quot;Product&quot;" --> DB
    U -- "asyncpg<br/>SELECT id,email,… FROM &quot;User&quot;" --> DB
    O -- "JPA findAll()<br/>orders JOIN order_statuses" --> DB
    M -- "migrate deploy + db seed<br/>(creates tables, inserts data)" --> DB
```

## b. One request, traced end to end

Request traced: **`GET /users`**, the users list rendered on page load.

| Hop (from `module-01.md`)                                                                    | `file:line`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend file and line that issues the request                                               | [`frontend/src/main.js:62`](../../frontend/src/main.js#L62) (`DOMContentLoaded` listener)<br/>[`frontend/src/main.js:56`](../../frontend/src/main.js#L56) (`renderUsers(...)`)<br/>[`frontend/src/components/users.js:11`](../../frontend/src/components/users.js#L11) (`` fetchData(`${apiUrl}/users`) ``)<br/>[`frontend/src/api/api.js:8`](../../frontend/src/api/api.js#L8) (`fetch(url)`)                                                                                                                                                                                     |
| The URL and the service that receives it — `http://localhost:8000/users` → **users-service** | [`frontend/src/main.js:8`](../../frontend/src/main.js#L8) (base URL)<br/>[`docker-compose.yml:19`](../../docker-compose.yml#L19) (`VITE_USERS_API_URL`)<br/>[`.env:4`](../../.env#L4) (`USERS_SERVICE_PORT=8000`)<br/>[`docker-compose.yml:52`](../../docker-compose.yml#L52) (port mapping)<br/>                                                                                                                                                                                                                                                                                  |
| The route or controller that matches it                                                      | [`users-service/main.py:35-36`](../../users-service/main.py#L35-L36) (`@app.get("/users")` → `get_users()`)<br/>[`users-service/main.py:13-19`](../../users-service/main.py#L13-L19) (CORS)                                                                                                                                                                                                                                                                                                                                                                                        |
| The query or ORM call that runs, and the table it touches — raw SQL on **`"User"`**          | [`users-service/main.py:26-33`](../../users-service/main.py#L26-L33) (`get_db_connection`)<br/>[`users-service/main.py:23`](../../users-service/main.py#L23) (`USER_COLUMNS`, no `password`)<br/>[`users-service/main.py:39`](../../users-service/main.py#L39) (`conn.fetch(...)`)<br/>[`database/prisma/migrations/20251125234441_init/migration.sql:2`](../../database/prisma/migrations/20251125234441_init/migration.sql#L2) (`CREATE TABLE "User"`)<br/>[`database/prisma/schema.prisma:14`](../../database/prisma/schema.prisma#L14) (`model User`, source of the migration) |
| The frontend function that turns the response into DOM — `renderUsers()`                     | [`frontend/src/api/api.js:12`](../../frontend/src/api/api.js#L12) (`response.json()`)<br/>[`frontend/src/components/users.js:18-29`](../../frontend/src/components/users.js#L18-L29) (render cards)<br/>[`frontend/src/components/users.js:41-45`](../../frontend/src/components/users.js#L41-L45) (`escapeHtml`)<br/>[`frontend/src/components/users.js:30-33`](../../frontend/src/components/users.js#L30-L33) (error path)                                                                                                                                                      |

## c. Environment gotchas

The only problem i faced is port-forwarding
## d. One thing the documentation gets wrong or leaves out

_TODO_
