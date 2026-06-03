# System Map AI Prompts

## 1. Describe the System
Prompt:
``` Describe the system, response in a raw markdown format```

Response: 
```
# Learning Platform (LMS) — System Overview

This workspace contains **4 separate repositories** that form a Learning Management System for tracking student progress through a coding bootcamp.

---

## Repositories

### 1. `learn-ops-api` — Backend (Django)

A Python/Django REST Framework API managing all core data:

- **People** — students, cohorts, teams, mentors, notes, assessments
- **Coursework** — courses, books, projects, capstones, learning objectives
- **Auth** — GitHub OAuth login
- **Notifications** — email/messaging support
- Backed by **PostgreSQL**

### 2. `learn-ops-client` — Frontend (React)

A React single-page app used by students and instructors in the browser. Communicates with the Django API via REST calls.

### 3. `learn-ops-infrastructure` — Docker Compose Stack

Wires all services together for local development:

| Service | Purpose | Port |
|---|---|---|
| `postgres` | Database | 5432 |
| `api` | Django backend | 8000 |
| `client` | React frontend | 3000 |
| `prometheus` | Metrics scraper | 9090 |
| `grafana` | Metrics dashboard | 3001 |
| `postgres_exporter` | DB metrics | 9187 |

### 4. `service-monarch` — GitHub Issue Migrator (Pyt

A background service that migrates GitHub issues betwprojects. Key patterns:

- **Valkey** (Redis-compatible) as a message broker
- **Sliding window controller** for GitHub API rate limiting
- **Circuit breaker** to fail fast when GitHub is dow
- **Prometheus metrics** + structured logging

### 5. `tutorial-extensions` — Student Content

GitBook-style tutorial extensions in multiple languages (English, Spanish, Japanese, Korean, Farsi).

---

## Architecture

\`\`\`
Browser
  └─▶ React Client :3000
        └─▶ Django API :8000
              └─▶ PostgreSQL :5432

Django API ──▶ Valkey (message broker)
                    └─▶ Monarch service ──▶ GitHub AP

Prometheus :9090 ──▶ Grafana :3001
\`\`\`

---                                                                                                                                     
**In short:** a bootcamp ops platform where instructors track student progress, record assessments, manage cohorts, and automate GitHub project management.
```

## 2. Convert to a Mermaid Diagram
Prompt: 
``` create a system diagram in mermaid format```

Response: 
```
graph TD
    Browser["🌐 Browser"]

    subgraph Frontend
        Client["learn-ops-client\nReact App :3000"]
    end

    subgraph Backend
        API["learn-ops-api\nDjango REST :8000"]
        DB["PostgreSQL :5432"]
    end

    subgraph Messaging
        Valkey["Valkey\nRedis-compatible\nMessage Broker"]
    end

    subgraph Migration
        Monarch["service-monarch\nGitHub Issue Migrator"]
        GitHub["GitHub API"]
    end

    subgraph Observability
        Prometheus["Prometheus :9090"]
        Grafana["Grafana :3001"]
        PGExporter["postgres_exporter :9187"]
    end

    Browser --> Client
    Client --> API
    API --> DB
    API --> Valkey
    Valkey --> Monarch
    Monarch --> GitHub

    DB --> PGExporter
    PGExporter --> Prometheus
    API --> Prometheus
    Monarch --> Prometheus
    Prometheus --> Grafana
    ```

    ---
