# Developer Onboarding Guide

Welcome to the **OAN-Seeker-Service**! This guide will help you set up your environment and avoid common pitfalls.

## Prerequisites
*   **Node.js**: v18 or higher recommended.
*   **Docker**: For running Redis and potentially the DB locally.
*   **Access Credentials**: You need the `HASURA_GRAPHQL_ADMIN_SECRET` and `BAP_CLIENT_URL` from the team lead.

## Installation

1.  **Clone the repository**:****
    ```bash
    git clone <repo-url>
    cd OAN-Seeker-Service
    ```

2.  **Install Dependencies**:
    ```bash
    npm install
    ```

3.  **Environment Setup**:
    Create a `.env` file in the root directory. Use this template:
    ```env
    PORT=3000
    
    # Hasura / Database
    HASURA_URL=http://localhost:8080/v1/graphql
    HASURA_GRAPHQL_ADMIN_SECRET=<ask-team-lead>
    DB_TYPE=postgres
    DB_HOST=localhost
    DB_PORT=5432
    DB_USERNAME=hasura
    DB_PASSWORD=hasura
    DB_NAME=hasura_db
    
    # Table Names (CRITICAL: Must match Hasura Schema)
    CACHE_DB="scheme_cache_data"
    RESPONSE_CACHE_DB="response_cache_dev"
    JOBS_SEEKER_DEV="seeker_user"
    JOBS_ORDER_DEV="order_dev"
    JOBS_TELEMETRY_DB="telemetry_dev"

    # Redis
    REDIS_HOST=localhost
    REDIS_PORT=6379

    # External APIs
    API_KEY=<WEATHERMAP_API_KEY>
    GEO_URL=http://api.openweathermap.org/geo/1.0/direct

    # ONDC Network Config
    DOMAIN="schemes:oan"
    BAP_CLIENT_URL="<ask-team-lead>"
    BAP_ID="<your-bap-id>"
    BAP_URI="<your-bap-uri>"
    BPP_ID="<target-bpp-id>"
    BPP_URI="<target-bpp-url>"
    ```

## Running the Application

1.  **Start Dependencies (Optional)**:
    If you don't have local instances, use Docker:
    ```bash
    docker-compose up -d
    ```

2.  **Start the App (Dev Mode)**:
    ```bash
    npm run start:dev
    ```
    The server should start on port `3000`.

## Common Pitfalls & Tech Debt Awareness

### 1. Dynamic Table Names
**Pay Attention**: The caching logic allows table names to be defined in `.env` (e.g., `CACHE_DB`).
*   **Risk**: If you change the `.env` variable without migrating the database, queries in `hasura.service.ts` will fail silently or throw 500s.
*   **Check**: Always verify `CACHE_DB` matches your actual Hasura table name.

### 2. SQL Injection Risks
The `jobs.service.ts` file contains methods like `generateQuery` that manually concatenate strings for SQL queries.
*   **Risk**: Be extremely careful when modifying filters or input handling here.
*   **Guideline**: Do not pass raw user input directly into these string templates if possible.

### 3. The "God" Service
`JobsService` is very large.
*   **Guideline**: If you are adding new features (e.g., a new analytics report), try to avoid adding it to `JobsService`. Consider creating a new `AnalyticsService` instead.

### 4. Telemetry
Telemetry is handled via raw inserts into `JOBS_TELEMETRY_DB`. Ensure this table exists in your local Hasura instance, or analytics calls will fail.

## Troubleshooting

### Error: `/bin/sh: 1: Syntax error: "(" unexpected`
If you see this error when running `npm run start`, it is likely because your project path contains **parentheses** (e.g., `Projects(COSS)`).
*   **Cause**: The Linux shell in WSL interprets `(` and `)` as special commands.
*   **Fix**: Rename your parent folder to remove special characters (e.g., rename `Projects(COSS)` to `Projects-COSS`).
