# System Architecture & Codebase Guide

## Folder Structure Analysis

The project follows a standard modular NestJS architecture.

### Root Directory
*   `src/`: Contains all source code.
*   `docker-compose.yml`: orchestrates the app, db, and redis containers.
*   `Jenkinsfile`: CI/CD definitions.

### Key Modules (`src/`)

#### 1. Core Module (`src/jobs/`)
This is the heart of the application.
*   **`jobs.service.ts`**: The "God Class" of the project. It handles:
    *   **Search Logic**: Constructing and sending ONDC search payloads.
    *   **Weather Logic**: Checking Redis cache -> Fetching External API -> Updating Cache.
    *   **Analytics**: Executing complex raw SQL queries to generate reports.
*   **`jobs.controller.ts`**: The HTTP interface exposing endpoints like `/jobs/search` and `/jobs/weather`.

#### 2. Service Integrations (`src/services/`)
Houses reusable adapters for external systems.
*   **`hasura/hasura.service.ts`**: The Data Access Layer.
    *   Instead of using TypeORM for everything, the app heavily relies on **Hasura GraphQL** mutations/queries executed via HTTP.
    *   *Why?* Likely to leverage Hasura's auto-generated APIs and permission handling.
*   **`proxy/proxy.service.ts`**:
    *   A simple HTTP wrapper (using `axios`) to talk to the **BAP Client**.

#### 3. Database Entities (`src/entity/`)
*   **`response.entity.ts`**:
    *   Defines the `response_cache` table.
    *   This table acts as a **Transaction Log**, storing the raw JSON responses from the ONDC network for auditing and replays.

---

## Critical Execution Flows

### 1. The Search & Cache Loop
A "Search" operation triggers a complex flow:
1.  **API Call**: Controller receives `POST /jobs/search`.
2.  **JobService**: Constructs the ONDC Context (Domain, BAP ID, Timestamp).
3.  **ProxyService**: Forwards payload to `BAP_CLIENT_URL`.
4.  **Async Processing**:
    *   The BAP Client returns results from the network.
    *   `JobsService` parses the raw JSON (extracting Providers, Items, Locations).
    *   `JobsService` calls `HasuraService.insertCacheData()` to save cleaned data into Postgres.

### 2. Weather Data (Redis Strategy)
The weather feature uses a sophisticated caching strategy to save API costs:
1.  **Request**: User asks for weather at `lat, lon`.
2.  **Cache Key**: Service generates key `weather:{lat},{lon}`.
3.  **Check Redis**:
    *   **HIT**: Return data immediately (0ms latency).
    *   **MISS**:
        1.  Call OpenWeatherMap API via `ProxyService`.
        2.  Store result in Redis with `EX` (Expiry) = 3600s (1 hour).
        3.  Return data.

---

## Database Schema (Concept)
While the schema is managed by Hasura, the code implies the presence of these dynamic tables:
*   **`response_cache_dev`** (or variable name): Raw JSON logs.
*   **`job_cache`** (or variable name): Structured job listings.
*   **`seeker_user`**: User profile data.

---

*Next Step: Read `ONBOARDING.md` to set up your environment.*
