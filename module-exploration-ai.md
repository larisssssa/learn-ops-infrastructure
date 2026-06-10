# learn-ops-api: AI-Assisted Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder              | Why does this folder need to exist?                                           |
| ------------------- | ----------------------------------------------------------------------------- |
| `LearningAPI/`      | Core Django app — models, views, serializers, migrations, tests, and fixtures |
| `LearningPlatform/` | Django project config — settings, top-level URL routing, and WSGI entry point |
| `LogViewer/`        | Small Django app for viewing server logs via a web interface                  |
| `config/`           | Infrastructure files — deployment config and nginx configs for API and client |
| `static/`           | Source static assets (CSS, JS, images)                                        |
| `staticfiles/`      | Production-ready static assets collected by Django's `collectstatic`          |
| `templates/`        | HTML templates, primarily for the Django admin interface                      |
| `.github/`          | GitHub Actions workflows for CI/CD                                            |

## 2. Folders inside `LearningAPI`

| Folder         | Responsibility                                                                                                      | Why it exists                                                                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `models/`      | Defines the database tables and the relationships between them (e.g. Student, Cohort, Assessment)                   | Django needs Python classes to represent database structure — each class maps to a table and its attributes map to columns                        |
| `views/`       | Handles incoming HTTP requests and returns responses — this is where API endpoint logic lives                       | Separates request/response handling from data definitions; each view function or class decides what data to fetch, validate, and return           |
| `serializers/` | Converts model instances to JSON (for responses) and validates/parses JSON back into model instances (for requests) | APIs communicate in JSON, but Django works with Python objects internally — serializers are the translation layer between the two                 |
| `migrations/`  | Tracks every change ever made to the database schema as a sequence of versioned files                               | Allows the database structure to evolve safely over time without manually writing SQL; any developer can replay migrations to get the same schema |
| `tests/`       | Contains automated tests that verify the app behaves correctly                                                      | Catches regressions when code changes — tests run against models, views, and serializers to confirm the API contract is not broken                |
| `fixtures/`    | Stores pre-defined seed data (JSON/YAML files) that can be loaded into the database                                 | Useful for populating a fresh development database or setting up a known state before running tests                                               |

## 3. What is the Pipfile?

`Pipfile` is the dependency manifest for this Python project — it's the equivalent of a `package.json` in Node. It's managed by **Pipenv** and declares three things:

**Where to download packages from** (`[[source]]`) — PyPI, the standard Python package registry.

**What packages the app needs to run** (`[packages]`) — the production dependencies:

- `django` — the web framework the whole API is built on
- `djangorestframework` — adds REST API tools on top of Django (serializers, viewsets, etc.)
- `django-cors-headers` — allows the frontend (a different origin) to make requests to this API
- `gunicorn` — the production web server that runs the Django app
- `psycopg2-binary` — the driver that lets Django talk to PostgreSQL
- `dj-rest-auth` + `django-allauth` — handle user authentication (login, registration, tokens)
- `valkey` — a Redis-compatible cache/queue client
- `structlog`, `django-structlog`, `opensearch-py`, `python-logstash` — structured logging and log shipping
- `django-prometheus` — exposes app metrics for monitoring

**What packages are only needed during development** (`[dev-packages]`):

- `pytest`, `pytest-django`, `pytest-cov` — for running and measuring test coverage
- `pylint`, `pylint-django` — code linting
- `debugpy` — allows attaching a debugger (e.g. VS Code) to the running app

**The required Python version** (`[requires]`) — Python 3.11.11 exactly.

**Shortcut scripts** (`[scripts]`) — `pipenv run migrate` is a convenience alias for `python3 manage.py migrate`.

## 4. Key packages

| Package               | What it does                                                                                                                                        | How this project uses it                                                                                                                                                                                     | Why it's needed                                                                                                                                                                       |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `django`              | The core web framework — handles HTTP routing, the ORM (database queries via Python), the admin panel, migrations, and user/session management      | Every part of the app is built on it: `models/` uses Django's ORM to define tables, `views/` uses `HttpResponse` and `authenticate()`, `manage.py` runs migrations and the dev server                        | Without Django there is no app — it is the foundation everything else sits on                                                                                                         |
| `djangorestframework` | Adds REST API tooling on top of Django: serializers (model ↔ JSON), `ViewSet` classes, token-based auth, and response helpers                       | All serializers in `serializers/` extend `rest_framework.serializers`, all API views use `rest_framework` imports, and `Token.objects` (used in `auth.py` to issue login tokens) comes from this package     | Django alone handles HTML pages; `djangorestframework` is what makes it a JSON API that a frontend can consume                                                                        |
| `django-allauth`      | Handles third-party OAuth login flows (GitHub, Google, etc.) — manages the OAuth handshake, creates social accounts, and links them to Django users | Registered in `settings.py` with `allauth.socialaccount.providers.github`; used in `github_login.py` — `GithubLogin` extends `SocialLoginView` and `GitHubOAuth2Adapter` to handle the GitHub OAuth callback | Students and instructors log in with their GitHub accounts instead of a username/password, so the app delegates the entire OAuth flow to allauth rather than implementing it manually |

## 5. What does `decorators.py` do?

**What is a decorator?**

A decorator is a function that _wraps_ another function to add behavior before or after it runs — without changing the original function's code. Think of it like a security checkpoint at a door: the checkpoint runs first, and only if you pass does it let you through to the actual room.

In Python, you apply a decorator with the `@` symbol placed directly above a function definition.

---

**What this file defines**

Both decorators in this file serve the same purpose — **role-based access control**. They check whether the logged-in user belongs to a specific group before allowing the view function to execute.

`is_instructor()` — checks if the user is in the `Instructors` group.
`is_staff()` — checks if the
If the check fails, they immerized` response and the realview function never runs.

---

**How the layering works**  
Each decorator is actually three nested functions:  
is_instructor() ← you call this to create the decorator └── decorator(func) ← receirapped
└── \_\_wrapper(request, ...) ← runs at request time; does the check  
This three-layer pattern exists because `is_instructor()` is designed to be called with `@is_instructor()` (with parents to be passed in the future if needed.

---

**How it's used in the views**

In `student_view.py`, `@method_decorator(is_instructor())` is placed above specific
actions like `destroy` and `p

```python
@method_decorator(is_instructor())
def destroy(self, request, pk
    # only runs if the user is an instructor

method_decorator is a Django utility that adapts a function-style decorator to work on
class-based view methods.

The result: only instructors  projects. Any other user gets a 401 back immediately.
```

## 6. What is a serializer, and how does it fit the request/response cycle?

**What is a serializer?**

A serializer is a translator. It converts data in two directions:

- **Outbound (response):** Python model instance → JSON the frontend can read
- **Inbound (request):** JSON the frontend sent → validated Python data ready to save to the database

Without a serializer, you would have to manually convert every database object into a dictionary and then into JSON on every response — and manually validate every field on every request.

---

**Where it fits in the request/response cycle**

HTTP Request (JSON)
↓
View
↓
Serializer.is_valid() ← validates incoming data, rejects bad input early
↓
Model.save() ← writes to the database
↓
Serializer(instance) ← c to JSON
↓
HTTP Response (JSON)

---

\*\*A concrete example from thi

In `serializers/nssuser_seria (a database row) gets
converted into a JSON object a frontend can consume. The view does not build the
JSON manually — it hands the zer and returns the result:

```python
serializer = StudentSerializer(student, context={'request': request})
return Response(serializer.dae ready-to-send JSON

For an inbound POST request t

serializer = StudentSerializethe incoming JSON
if serializer.is_valid():                           # validate it
    serializer.save()        e to the database

---
Why this separation matters

The view should only decide what to do (fetch this student, save this record).
The serializer owns how data
Keeping those two concerns separate means you can change your API's JSON shape
without touching view logic, hout touching
serialization.
```

## 7. One model and what it represents

**What is a Django model?**

A Django model is a Python class that maps directly to a database table. Each attribute on the class becomes a column in that table. Django reads these class definitions and handles all the SQL — you never write `CREATE TABLE` or `SELECT *` manually. You just define the shape of your data in Python, run migrations, and Django does the rest.

Example of what a model class translates to:

```python
class StudentAssessment(models.Model):
    student = models.ForeignKey(NssUser, ...)   # → a foreign key column pointing to the users table
    assessment = models.ForeignKey(Assessment, ...)  # → a foreign key column pointing to assessments table
    status = models.ForeignKey(StudentAssessmentStatus, ...)
    url = models.CharField(max_length=512)      # → a text column
    date_created = models.DateField(auto_now=True)  # → a date column, auto-set on save
```

---

**A concrete model: `StudentAssessment`**

**What real-world thing it represents**

A `StudentAssessment` is the record of _one student being assigned one specific assessment_. In the real world, an instructor hands a student a coding project or quiz to complete. This model is that act of assignment — it tracks who got it, what they got, what the current status is, which instructor assigned it, and where the student submitted their work.

**Why the API needs to track it**

Without this model the platform has no memory of what work has been given to whom. Specifically it needs to exist because:

- **Status tracking** — a student's submission goes through states (e.g. `In Progress`, `Submitted`, `Reviewed`). The `status` foreign key points to `StudentAssessmentStatus`, which holds those state labels. The frontend needs to know where in the pipeline a student's work is.
- **Instructor accountability** — the `instructor` field records _who_ assigned the work, so the platform knows which instructor is responsible for reviewing it.
- **Submission URL** — the `url` field is where the student links their completed work (e.g. a GitHub repo). Without storing this, the instructor has nowhere to go to review it.
- **Preventing duplicate assignments** — the `unique_together = (('student', 'assessment'),)` constraint at the bottom enforces that the same student cannot be assigned the same assessment twice, which prevents data errors.

**How the three models relate**

```
Assessment          ← defines the assessment itself (name, type, source URL)
    ↑
StudentAssessment   ← joins a student to an assessment, with status + submission URL
    ↑
StudentAssessmentStatus  ← a lookup table of possible statuses (e.g. "Submitted", "Reviewed")
```

`Assessment` is the template. `StudentAssessment` is the instance of that template assigned to a real person.

## 8. Views vs. viewsets

| Type                            | Example class / function | File path                                     | When to use it                                                                                                                                                                                                                  |
| ------------------------------- | ------------------------ | --------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Plain view** (function-based) | `login_user`             | `LearningAPI/views/auth.py`                   | When the endpoint has custom logic that doesn't map to CRUD — e.g. authenticating a user, triggering an action, handling OAuth. You write every `if request.method` branch yourself, giving you full control over what happens. |
| **ViewSet**                     | `StudentNoteTypeViewSet` | `LearningAPI/views/student_note_type_view.py` | When the endpoint is standard CRUD on a model. Extending `ModelViewSet` and declaring a `queryset` and `serializer_class` gives you GET, POST, PUT, PATCH, and DELETE for free — no method bodies needed.                       |

## 9. What replaces templates and why?

**Django's MTV pattern and what changes in a REST API**

In a traditional Django app the three layers are:

- **Model** — the database shape
- **Template** — HTML files that render data for a browser
- **View** — the logic that fetches data and passes it to a template

In a REST API the browser is replaced by a **separate frontend application**
(a React, Vue, or mobile app). That frontend already knows how to render its
own HTML — it does not need the server to produce it.

---

**What takes the template's role here**

The **serializer** replaces the template.

| Traditional Django                                                  | This project                                                            |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| View fetches data → passes it to a Template → Template renders HTML | View fetches data → passes it to a Serializer → Serializer renders JSON |

Instead of producing a finished HTML page, the server produces a structured
JSON response. The frontend consumes that JSON and decides how to display it.

---

**Why that makes sense for a REST API**

| Reason                           | Explanation                                                                                                                                  |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| **Separation of concerns**       | The API owns the data. The frontend owns the UI. Neither is coupled to the other's implementation.                                           |
| **One backend, many clients**    | The same JSON endpoint can serve a React web app, a mobile app, or a third-party integration — none of which could consume an HTML template. |
| **Frontend can change freely**   | The frontend team can redesign the entire UI without touching the API, because the contract between them is JSON, not HTML.                  |
| **Templates would be redundant** | If the server rendered HTML, the frontend framework would have nothing to do — you would be building two rendering layers for the same data. |
