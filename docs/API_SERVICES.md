# API Services & Usage Guide

This document details the REST API services provided by the **OAN-Seeker-Service**.

## Base URL
Unless configured otherwise, the service runs on:
`http://localhost:3000`

---

## 1. Jobs & Content Services
**Base Endpoint**: `/content`

These endpoints form the core business logic for searching jobs, caching data, and analytics.

### A. Data Retrieval
| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **POST** | `/content/search` | Search for cached jobs/content in the local Hasura ID. |
| **POST** | `/content/searchWeather` | Get weather data. Checks Redis cache first, then calls OpenWeatherMap. |
| **POST** | `/content/select` | Select a specific job item (ONDC flow). |
| **POST** | `/content/responseSearch` | Search through raw transaction logs in `response_cache`. |

#### Example: Weather Search
**Request**: `POST /content/searchWeather`
```json
{
  "location": "Bangalore"
}
```
**Response**: JSON object with weather details (temperature, humidity, etc.).

### B. Filter Helpers
These endpoints provide metadata for frontend dropdowns/filters.
| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **GET** | `/content/getState` | Get list of available states. |
| **GET** | `/content/getCity?state=Karnataka` | Get cities for a specific state. |
| **GET** | `/content/getTitle` | Get unique job titles. |
| **GET** | `/content/getSkills` | Get list of skills. |

### C. System Triggers
| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **POST** | `/content/create` | **Manual Trigger**: Initiates a fresh crawl/search on the ONDC network via the BAP Client. |
| **POST** | `/content/analytics` | customized analytics query (logic in `JobsService`). |

---

## 2. ONDC Network Proxy
**Base Endpoint**: `/` (Root)

These endpoints directly forward requests to the **BAP Client** (The external ONDC gateway). Use these when you need to perform raw ONDC transactions.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **POST** | `/search` | Trigger ONDC `search` API. |
| **POST** | `/select` | Trigger ONDC `select` API. |
| **POST** | `/init` | Trigger ONDC `init` API. |
| **POST** | `/confirm` | Trigger ONDC `confirm` API. |
| **POST** | `/status` | Check status of an order. |
| **POST** | `/update` | Update an order. |

**Note**: These methods (in `AppController`) rely on the `ProxyService` to forward the exact JSON body you send to the BAP Client defined in the `BAP_CLIENT_URL` env variable.

---

## 3. User & Order Management
**Base Endpoint**: `/user`

Manages the local "Seeker" user profiles and Order records.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **POST** | `/user/create` | Create a new user profile. |
| **GET** | `/user/:id` | Get user details by ID. |
| **POST** | `/user/createOrder` | Link a user to a specific Content/Job ID (Creation of an "Order"). |
| **GET** | `/user/searchOrder/:OrderId` | Find an order by its unique Order ID. |

---

## 4. Location Utility
**Base Endpoint**: `/location`

Provides static location data (States/Districts), likely populated from a local seed or database.

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| **GET** | `/location/states?lang=en` | Get list of states in a specific language. |
| **GET** | `/location/districts/:state_id` | Get districts for a given state ID. |

---

## How to Run & Test

1.  **Start the Server**:
    ```bash
    npm run start:dev
    ```
2.  **Using Postman**:
    *   Create a new Collection.
    *   Add a Request for `POST http://localhost:3000/content/create`.
    *   Send the request. You should see logs in your terminal indicating a network call to the BAP Client.
3.  **Using Curl (Weather Test)**:
    ```bash
    curl -X POST http://localhost:3000/content/searchWeather \
    -H "Content-Type: application/json" \
    -d '{"location": "Delhi"}'
    ```
