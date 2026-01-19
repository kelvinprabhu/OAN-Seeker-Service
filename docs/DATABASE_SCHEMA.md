# Database Schema Guide

This document outlines the recommended PostgreSQL database schema for the OAN Seeker Service. 
The application relies on Hasura for GraphQL APIs, but the underlying tables must be created in PostgreSQL.

## Table Summary

| Table Name       | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `response_cache` | Acts as a transaction log, storing raw ONDC JSON responses.                 |
| `job_cache`      | Stores structured, searchable job/item data extracted from ONDC responses.  |
| `seeker_user`    | Manages user profiles for job seekers (or general users).                   |
| `job_order`      | Tracks orders/applications made by users, linking them to jobs.             |
| `job_telemetry`  | Stores analytics events and logs.                                           |

---

## 1. Response Cache (`response_cache`)
Stores all raw responses from the BAP Client for auditing and debugging.

```sql
CREATE TABLE response_cache (
    id SERIAL PRIMARY KEY,
    action TEXT,
    transaction_id TEXT,
    response JSONB, -- JSONB is faster for querying than JSON
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Index for faster lookups
CREATE INDEX idx_response_cache_transaction_id ON response_cache(transaction_id);
CREATE INDEX idx_response_cache_action ON response_cache(action);
```

## 2. Job Cache (`job_cache`)
Stores processed items (Jobs, Courses, Scholarships) for the actual application UI to query.

```sql
CREATE TABLE job_cache (
    id SERIAL PRIMARY KEY,
    unique_id TEXT UNIQUE NOT NULL,
    
    -- ONDC Routing Info
    bpp_id TEXT,
    bpp_uri TEXT,
    provider_id TEXT,
    provider_name TEXT,
    
    -- Item Details
    item_id TEXT,
    title TEXT,
    short_desc TEXT,
    long_desc TEXT,
    
    -- Media
    image TEXT,
    media TEXT,
    mimetype TEXT,
    
    -- JSON/Array Fields
    locations JSONB,      -- or TEXT[] if preferred, but JSONB is more flexible
    categories JSONB,
    fulfillments JSONB,
    tags JSONB,           -- Important: Stores robust attributes like Salary, Duration
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for search performance
CREATE INDEX idx_job_cache_unique_id ON job_cache(unique_id);
CREATE INDEX idx_job_cache_title ON job_cache(title);
CREATE INDEX idx_job_cache_provider ON job_cache(provider_name);
```

## 3. Seeker User (`seeker_user`)
Stores user profile information.

```sql
CREATE TABLE seeker_user (
    id SERIAL PRIMARY KEY,
    user_id INT,          -- Optional: External reference ID if using another auth provider
    name TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    phone TEXT,
    brief_profile TEXT,   -- Replaces 'brief' if needed
    gender TEXT,
    age TEXT,             -- or INT if strictly numeric
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_seeker_user_email ON seeker_user(email);
```

## 4. Job Order (`job_order`)
Links a user to a specific job/item they have applied for or interacted with.

```sql
CREATE TABLE job_order (
    id SERIAL PRIMARY KEY,
    order_id TEXT UNIQUE NOT NULL,
    
    -- Relationships
    seeker_id INT REFERENCES seeker_user(id) ON DELETE CASCADE,
    content_id TEXT, -- Matches job_cache.unique_id? (Ideally should be FK if types match)
    
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_job_order_seeker_id ON job_order(seeker_id);
CREATE INDEX idx_job_order_order_id ON job_order(order_id);
```

**Note**: To enforce referential integrity between `job_order` and `job_cache`, ensure `content_id` matches the type and values of `job_cache.unique_id`.

## 5. Telemetry (`job_telemetry`)
Stores analytics data from the frontend/app.

```sql
CREATE TABLE job_telemetry (
    id TEXT PRIMARY KEY, -- or SERIAL if auto-increment is preferred
    ver TEXT,
    events JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Environment Variables Mapping

Ensure your `.env` file matches the table names created above:

```bash
RESPONSE_CACHE_DB=response_cache
CACHE_DB=job_cache
JOBS_SEEKER_DEV=seeker_user
JOBS_ORDER_DEV=job_order
JOBS_TELEMETRY_DB=job_telemetry
```

## Auto-Update Timestamp Function (Optional)
To automatically update `updated_at` columns:

```sql
CREATE OR REPLACE FUNCTION update_modified_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = now();
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_response_cache_modtime BEFORE UPDATE ON response_cache FOR EACH ROW EXECUTE PROCEDURE update_modified_column();
CREATE TRIGGER update_job_cache_modtime BEFORE UPDATE ON job_cache FOR EACH ROW EXECUTE PROCEDURE update_modified_column();
CREATE TRIGGER update_seeker_user_modtime BEFORE UPDATE ON seeker_user FOR EACH ROW EXECUTE PROCEDURE update_modified_column();
CREATE TRIGGER update_job_order_modtime BEFORE UPDATE ON job_order FOR EACH ROW EXECUTE PROCEDURE update_modified_column();
```
