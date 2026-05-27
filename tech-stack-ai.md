# Tech Stack (AI)

## 1. Run Questions

### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| | | | | |
| | | | | |
| | | | | |

## Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| `.env` | `learn-ops-api/` | `LEARNING_GITHUB_CALLBACK` | GitHub OAuth redirect URL | Tells GitHub where to send the user after they authorize login |
| `.env` | `learn-ops-api/` | `LEARN_OPS_CLIENT_ID` | GitHub OAuth App client ID | Identifies this app to GitHub when starting the OAuth flow |
| `.env` | `learn-ops-api/` | `LEARN_OPS_SECRET_KEY` | GitHub OAuth App client secret | Verifies the token exchange with GitHub after login |
| `.env` | `learn-ops-api/` | `LEARN_OPS_DJANGO_SECRET_KEY` | Django cryptographic secret | Signs sessions, CSRF tokens, and cookies |
| `.env` | `learn-ops-api/` | `DEBUG` | Django debug mode toggle | Enables detailed error pages and dev tooling |
| `.env` | `learn-ops-api/` | `LEARN_OPS_DB` | PostgreSQL database name | Tells Django which database to connect to |
| `.env` | `learn-ops-api/` | `LEARN_OPS_USER` | PostgreSQL username | Authenticates Django's connection to the database |
| `.env` | `learn-ops-api/` | `LEARN_OPS_PASSWORD` | PostgreSQL password | Secures the database connection |
| `.env` | `learn-ops-api/` | `VALKEY_HOST` | Valkey broker hostname | Points Django to the message queue server |
| `.env` | `learn-ops-api/` | `GITHUB_TOKEN` | GitHub Personal Access Token | Authorizes direct GitHub API calls from the backend |
| `.env` | `learn-ops-api/` |t auth token | Allows the APIto post messages to Slack channels |
| `settings.py` | `learn-ops-RET_KEY` | Django mastersecret key | Read from `LEARN_OPS_DJANGO_SECRET_KEY` env var at startup |
| `settings.py` | `learn-ops-OWED_HOSTS` | Accepted request hostnames | Blocks requests from unrecognized domains |
| `settings.py` | `learn-ops-S_ORIGIN_WHITELIST` | AllowedCORS origins | Restricts which frontends can call the API |
| `settings.py` | `learn-ops-ABASES` | Full DB connectionconfig block | Assembled from `LEARN_OPS_*` env vars to connect to PostgreSQL |
| `settings.py` | `learn-ops-KEY_CONFIG` | Valkeyconnection block | Assembled from `VALKEY_*` env vars for the message queue client |
| `settings.py` | `learn-ops-GING` | Log level and handlers | Controls how structured logs are written and at what verbosity |
| `settings.py` | `learn-ops-TALLED_APPS` | RegisteredDjango apps list | Tells Django which features, APIs, and modules to activate |
| `.env` | `learn-ops-client/kend API base URL | Prefixedon every HTTP call the React app makes to the API |
| `.env` | `learn-ops-client/ment name string | Used toconditionally enable dev vs production UI behavior |                                 | `.env` | `learn-ops-client/ile watcher mode flag |Enables hot-reload when running inside a Docker container |                          | `.env` | `learn-ops-client/urce map output toggle | Whenfalse, strips source maps from production builds |                                   | `.env` | `learn-ops-infrastatabase name | Passed to thePostgreSQL container to create the database on first start |                         | `.env` | `learn-ops-infrast Database username | Createsthe DB user that Django and the exporter connect with |                              | `.env` | `learn-ops-infrastD` | Database password |Secures the PostgreSQL container |                                                   | `.env` | `learn-ops-infrast` | Prometheus exporter DSN |Tells postgres_exporter how to connect to PostgreSQL to scrape metrics |             | `docker-compose.yml` | `leatworks` (learningplatform) |Shared Docker network | Lets all containers reach each other by hostname |           | `docker-compose.yml` | `lealumes` | Persistent storagemounts | Keeps database data and source code alive between container restarts |      | `docker-compose.yml` | `leaalthcheck` | DB readinesscheck | Prevents the API from starting before PostgreSQL is fully ready |            | `prometheus.yml` | `learn-o_interval` | Global metricpolling rate | Defines how often Prometheus collects metrics from all targets |      | `prometheus.yml` | `learn-ome` (django) | Django metricsscrape job | Points Prometheus at the Django API's /metrics endpoint |               | `prometheus.yml` | `learn-ome` (postgresql) | PostgreSQLmetrics scrape job | Points Prometheus at the postgres_exporter metrics endpoint |   | `.env` | `service-monarch/` Access Token | AuthorizesMonarch to read and write issues via the GitHub API |                                | `.env` | `service-monarch/`ker hostname | Tells Monarchwhere to subscribe and listen for migration tasks |                                  | `.env` | `service-monarch/`k incoming webhook URL | Sends migration success/failure notifications to a Slack channel |                        | `.env` | `service-monarch/`token | Alternate auth methodfor Slack API calls |                                                                | `settings.py` | `service-moHUB_API_URL` | GitHub REST API base URL | Base URL prepended to every GitHub API request Monarch makes |           | `settings.py` | `service-moHUB_RATE_LIMIT_PAUSE` | Ratelimit back-off delay | Pauses Monarch when GitHub signals it is being rate-limited | | `settings.py` | `service-moO_MIGRATION_PAUSE` | Delaybetween migrations | Throttles the speed of issue copying to avoid overwhelming the  GitHub API |
| `settings.py` | `service-monarch/service/config/` | `PROMETHEUS_PORT` | Metrics ser port | Starts a small HTTP sPrometheus metrics |
| `book.json` | `tutorial-extensions/` | `gitbook` | GitBook version pin | Specifies which GitBook CLI version to te |
| `book.json` | `tutorial-extensions/` | `plugins` | Plugin list array | Loads extras like GitHub links, sidebar ad
| `book.json` | `tutorial-extensions/` | `pluginsConfig` | Per-plugin config block | Configures each plugin — e.g.anguages |

### 1b. How to Start It

### 1c. Where to Access It

| Service | Port | URL |
|---|---|---|
| | | |
| | | |
| | | |

### 1d. Service Dependencies

| Service | Depends On | Why |
|---|---|---|
| | | |
| | | |
| | | |

### 1e. Main Entry Points

| Service | Startup File | Routes / URL Config File |
|---|---|---|
| | | |
| | | |
| | | |

## 2. Services

| Service Name | Tech Stack (including version) | Purpose |
|---|---|---|
| | | |

## 3. System Overview