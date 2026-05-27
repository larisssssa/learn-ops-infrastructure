# Tech Stack (AI)

## 1. Run Questions

<!-- ### 1a. Config Files

| Config File | Location | Config Value | What it's for | How it's used |
|---|---|---|---|---|
| | | | | |
| | | | | |
| | | | | | -->

### 1a. Config Files

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
| `.env` | `learn-ops-client/kend API base URL` | Prefixedon every HTTP call the React app makes to the API |
| `.env` | `learn-ops-client/ment name string` | Used toconditionally enable dev vs production UI behavior |                                 | `.env` | `learn-ops-client/ile watcher mode flag` |Enables hot-reload when running inside a Docker container |                          | `.env` | `learn-ops-client/urce map output toggle | Whenfalse, strips source maps from production builds |                                   | `.env` | `learn-ops-infrastatabase name | Passed to thePostgreSQL container to create the database on first start |                         | `.env` | `learn-ops-infrast Database username | Createsthe DB user that Django and the exporter connect with |                              | `.env` | `learn-ops-infrastD` | Database password |Secures the PostgreSQL container |                                                   | `.env` | `learn-ops-infrast` | Prometheus exporter DSN |Tells postgres_exporter how to connect to PostgreSQL to scrape metrics |             | `docker-compose.yml` | `leatworks` (learningplatform) |Shared Docker network | Lets all containers reach each other by hostname |           | `docker-compose.yml` | `lealumes` | Persistent storagemounts | Keeps database data and source code alive between container restarts |      | `docker-compose.yml` | `leaalthcheck` | DB readinesscheck | Prevents the API from starting before PostgreSQL is fully ready |            | `prometheus.yml` | `learn-o_interval` | Global metricpolling rate | Defines how often Prometheus collects metrics from all targets |      | `prometheus.yml` | `learn-ome` (django) | Django metricsscrape job | Points Prometheus at the Django API's /metrics endpoint |               | `prometheus.yml` | `learn-ome` (postgresql) | PostgreSQLmetrics scrape job | Points Prometheus at the postgres_exporter metrics endpoint |   | `.env` | `service-monarch/` Access Token | AuthorizesMonarch to read and write issues via the GitHub API |                                | `.env` | `service-monarch/`ker hostname | Tells Monarchwhere to subscribe and listen for migration tasks |                                  | `.env` | `service-monarch/`k incoming webhook URL | Sends migration success/failure notifications to a Slack channel |                        | `.env` | `service-monarch/`token | Alternate auth methodfor Slack API calls |                                                                | `settings.py` | `service-moHUB_API_URL` | GitHub REST API base URL | Base URL prepended to every GitHub API request Monarch makes |           | `settings.py` | `service-moHUB_RATE_LIMIT_PAUSE` | Ratelimit back-off delay | Pauses Monarch when GitHub signals it is being rate-limited | | `settings.py` | `service-moO_MIGRATION_PAUSE` | Delaybetween migrations | Throttles the speed of issue copying to avoid overwhelming the  GitHub API |
| `settings.py` | `service-monarch/service/config/` | `PROMETHEUS_PORT` | Metrics ser port | Starts a small HTTP sPrometheus metrics |
| `book.json` | `tutorial-extensions/` | `gitbook` | GitBook version pin | Specifies which GitBook CLI version to te |
| `book.json` | `tutorial-extensions/` | `plugins` | Plugin list array | Loads extras like GitHub links, sidebar ad
| `book.json` | `tutorial-extensions/` | `pluginsConfig` | Per-plugin config block | Configures each plugin — e.g.anguages |

### 1b. How to Start It
#### The Entry Point
All commands are run from **`~/workspace/lms/learn-ops-infrastructure/`**.


#### First-Time Setup

| Command | What it runs | What it does |
|---|---|---|
| `make setup` | `./scripts/setup.sh` | Full interactive wizard — clones repos, collects credentials, writes `.env` files, creates Docker network, starts the stack, and runs GitHub OAuth |
| `make doctor` | `./scripts/setup.sh --doctor` | Checks your environment only (Docker, Git, Python, ports) — makes no changes |

**`make setup` vs `make doctor`** — `doctor` is a dry run. It checks that all the prerequisites are in place and reports what's healthy or broken, but stops before touching anything. Run `doctor` first if you're unsure your machine is ready.


#### Day-to-Day Start Commands

| Command | Underlying command | What it does |
|---|---|---|
| `make up` | `docker compose up --build -d` | Starts **all** services and rebuilds images if code has changed |
| `make up-api` | `docker compose up --build -d api` | Starts **only the Django API** (and its dependency, the database) |
| `make up-client-api` | `docker compose up --build -d api client` | Starts the **API + React client**, skips Prometheus/Grafana |

**How they differ:**
- `make up` — the normal daily command. Starts every service defined in `docker-compose.yml`: database, API, client, Prometheus, Grafana, and postgres_exporter
- `make up-api` — lightweight option when you only need the backend (e.g., running API tests, no browser work)
- `make up-client-api` — middle ground: full app (frontend + backend) without the monitoring stack

All three use `--build`, which means Docker will detect any code changes and rebuild the image before starting. The `-d` flag runs everything in the background (detached).


#### Stop & Inspect Commands

| Command | Underlying command | What it does |
|---|---|---|
| `make down` | `docker compose down` | Stops and removes all running containers — **data is preserved** |
| `make reset` | `docker compose down -v --remove-orphans` | Stops containers and **deletes all volumes** (wipes the database) |
| `make restart` | `down` then `up --build -d` | Full stop + rebuild + start in one step |
| `make logs` | `docker compose logs -f` | Streams live logs from all containers to your terminal |
| `make ps` | `docker compose ps` | Shows the current status (running/stopped) of each container |

**`make down` vs `make reset`** — this is the most important difference:
- `make down` is safe to use daily. It stops the containers but keeps the database volume (`learning_platform_data`) intact, so your data is there when you start again
- `make reset` is destructive. The `-v` flag deletes all Docker volumes, which means the database is wiped and will be re-seeded from fixtures on the next `make up`

#### Teardown (Remove Everything)

| Command | What it runs | What it does |
|---|---|---|
| `make teardown` | `./scripts/teardown.sh` | Stops containers, uninstalls Docker, deletes `.env` files and all cloned repos. **Irreversible** — requires typing `teardown` to confirm |

This is only for fully leaving the project. It removes Docker itself from your machine, deletes all local repos (except `learn-ops-infrastructure`), and walks you through revoking your GitHub token.


#### Start-Up Order (What Happens Under the Hood)

When you run `make up`, Docker enforces this order automatically based on `depends_on` in `docker-compose.yml`:

1. database (PostgreSQL)        ← starts first
 ↓ health check passes
2. api (Django)                 ← waits for DB to be healthy
3. client (React)               ← starts in parallel with api
4. prometheus                   ← waits for api
5. grafana                      ← waits for prometheus
6. postgres_exporter            ← waits for database

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