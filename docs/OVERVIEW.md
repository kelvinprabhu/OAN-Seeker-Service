# Project Overview: OAN-Seeker-Service

## Introduction
**OAN-Seeker-Service** is a specialized middleware application designed for the ONDC (Open Network for Digital Commerce) ecosystem. It functions as a "Seeker" node, responsible for discovering, aggregating, and serving data from the decentralized network to frontend client applications.

## Core Problem
In a decentralized network like ONDC, data (such as job listings or weather forecasts) is fragmented across multiple "Provider" nodes. A client application (like a mobile app) cannot efficiently query every provider in real-time. 

**This service solves that by:**
1.  **Aggregating**: It polls the network or sends broadcast searches via a BAP (Buyer App Protocol) Client.
2.  **Caching**: It stores the results in a local high-performance database (PostgreSQL/Hasura) and memory cache (Redis).
3.  **Serving**: It provides a unified, fast API for frontend clients to access this aggregated data.

## Technology Stack
The project is built on a modern, scalable stack:

| Component | Technology | Purpose |
| h | h | h |
| **Framework** | **NestJS** | The core Node.js framework (TypeScript). |
| **Language** | **TypeScript** | Type usage for reliability. |
| **Primary DB** | **PostgreSQL** | Persistent storage for job data and transaction logs. |
| **Data Access** | **TypeORM** | ORM for database interactions. |
| **GraphQL** | **Hasura** | Manages database schemas and allows flexible data querying. |
| **Caching** | **Redis** | High-speed cache for ephemeral data (e.g., Weather). |
| **Protocol** | **ONDC / HTTP** | Communication standard with the external BAP network. |
| **External API** | **OpenWeatherMap** | Source for weather data augmentation. |

## Mental Model: How It Runs
Imagine the service as a **Smart Proxy**:

1.  **Frontend Request**: A user asks for "Jobs in Bangalore".
2.  **Service Action**: 
    *   The service constructs a standardized ONDC search payload.
    *   It forwards this to the **BAP Client** (an external gateway to the ONDC network).
3.  **Network Response**: 
    *   The network returns raw data from multiple Employment Providers.
    *   The **Seeker Service** intercepts these responses.
4.  **Processing & Storage**:
    *   It parses the complex JSON structure.
    *   It saves the structured data into **Hasura** (Postgres).
    *   It updates the **Redis** cache if needed.
5.  **Analytics**: It logs the transaction for business intelligence (e.g., "How many female candidates searched for jobs?").

---
*Next Step: Read `ARCHITECTURE.md` to understand the code structure.*
