
# System Map (AI)

## 1. System Diagram
``` mermaid

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

## 1. System Diagram (Sample prompt) (ascii format)

┌─────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    BROWSER / USER                                           │
└─────────────────────────────────────────────────────────────────────────────────────────────┘
                                           │
                                     HTTP :3000
                                           │
                                           ▼
                          ┌────────────────────────────────┐
                          │     learn-ops-client           │
                          │         React                  │
                          │         :3000                  │
                          └────────────────────────────────┘
                                           │
                                   HTTP REST :8000
                                           │
                                           ▼
                          ┌────────────────────────────────┐
                          │      learn-ops-api             │
                          │   Django REST Framework        │
                          │         :8000                  │
                          └────────────────────────────────┘
              │            │            │            │            │
       SQL :5432    PUBLISH :6379  HTTPS :443   HTTPS :443   HTTPS :443
              │            │            │            │            │
              ▼            ▼            │            │            │
  ┌──────────────┐  ┌────────────┐     │            │            │
  │  PostgreSQL  │  │   Valkey   │     │            │            │
  │     :5432    │  │   :6379    │     │            │            │
  └──────────────┘  └────────────┘     │            │            │
         │                │            ▼            ▼            ▼
  SQL :5432        SUBSCRIBE :6379  ┌──────────┐ ┌──────────┐ ┌──────────────┐
         │                │         │ GitHub   │ │  Slack   │ │  GitHub      │
         ▼                ▼         │   API    │ │   API    │ │  OAuth       │
  ┌─────────────┐  ┌────────────┐   │  :443    │ │  :443    │ │  :443        │
  │  postgres   │  │  service-  │   └──────────┘ └──────────┘ └──────────────┘
  │  _exporter  │  │  monarch   │       ▲              ▲
  │    :9187    │  │  :8080     │       │              │
  └─────────────┘  └────────────┘  HTTPS :443    HTTPS :443
         │              │  │
  HTTP :9187     PUBLISH :6379
         │              │
         │              ▼
         │         ┌────────────┐
         │         │   Valkey   │
         │         │   :6379    │
         │         │  (logs +   │
         │         │   state)   │
         │         └────────────┘
         │
         ▼
  ┌──────────────┐
  │  Prometheus  │◀── HTTP scrape :8000 ── learn-ops-api
  │    :9090     │◀── HTTP scrape :9187 ── postgres_exporter
  └──────────────┘
         │
   HTTP :9090
         │
         ▼
  ┌──────────────┐
  │   Grafana    │
  │    :3001     │
  └──────────────┘


  ## 1. System Diagram (sample prompt) (mermaid)

  ``` mermaid
graph TD
    Browser["Browser / User"]

    Client["learn-ops-client\nReact · :3000"]
    API["learn-ops-api\nDjango REST Framework · :8000"]
    DB["PostgreSQL · :5432"]
    Valkey["Valkey · :6379"]
    Monarch["service-monarch\nPython · :8080"]
    PGExporter["postgres_exporter · :9187"]
    Prometheus["Prometheus · :9090"]
    Grafana["Grafana · :3001"]
    GitHubAPI["GitHub API · :443"]
    GitHubOAuth["GitHub OAuth · :443"]
    SlackAPI["Slack API · :443"]

    Browser -->|"HTTP :3000"| Client
    Client -->|"HTTP REST :8000"| API

    API -->|"SQL :5432"| DB
    API -->|"PUBLISH :6379"| Valkey
    API -->|"HTTPS REST :443"| GitHubAPI
    API -->|"HTTPS OAuth :443"| GitHubOAuth
    API -->|"HTTPS REST :443"| SlackAPI

    Monarch -->|"SUBSCRIBE :6379"| Valkey
    Monarch -->|"HTTPS REST :443"| GitHubAPI
    Monarch -->|"HTTPS REST :443"| SlackAPI

    DB -->|"SQL :5432"| PGExporter

    Prometheus -->|"HTTP scrape :8000"| API
    Prometheus -->|"HTTP scrape :9187"| PGExporter
    Prometheus -->|"HTTP scrape :8080"| Monarch

    Grafana -->|"HTTP :9090"| Prometheus
  ```