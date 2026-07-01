# Data Model AI Prompts

## 1. Find the Database Connection Details

    I need to connect to the database for this system using pgAdmin.
    Find the host, port, database name, username, and password.
    pgAdmin itself running as a Docker container, connect pgAdmin to the DB's network

```
pgadmin is now attached to both bridge and learningplatform networks, so it can resolve database and reach the DB container directly.

In pgAdmin, register a new server with:

┌────────────────┬──────────────────┐
│     Field      │      Value       │
├────────────────┼──────────────────┤
│ Host           │ database         │
├────────────────┼──────────────────┤
│ Port           │ 5432             │
├────────────────┼──────────────────┤
│ Maintenance DB │ learningplatform │
├────────────────┼──────────────────┤
│ Username       │ learnops         │
├────────────────┼──────────────────┤
│ Password       │ learnops123      │
└────────────────┴──────────────────┘
```

## 2. Identify the Database Type

    Identify the database engine this application uses. provide location of that information in the project files

```
    Database engine: PostgreSQL

Location: LearningPlatform/settings.py:197, in the DATABASES['default']['ENGINE'] setting:

'ENGINE': 'django.db.backends.postgresql_psycopg2'

This is Django's standard PostgreSQL backend, using the psycopg2 driver under the hood. It's corroborated by two other places in the project you've already seen:

- The postgres:16 Docker image running as learning-platform-db (confirmed earlier via docker inspect/docker ps).
- .env lines 7-11 (LEARN_OPS_HOST, PORT, DB, USER, PASSWORD), which feed directly into this same DATABASES dict via os.getenv(...) calls on lines 198-202.
```

## 3. Map the ORM to the Database

```
1. ORM and connection config

ORM: Django's built-in ORM (this is a Django REST Framework project).

Connection config location: LearningPlatform/settings.py:195-204

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': os.getenv("LEARN_OPS_DB"),
        'USER': os.getenv("LEARN_OPS_USER"),
        'PASSWORD': os.getenv("LEARN_OPS_PASSWORD"),
        'HOST': os.getenv("LEARN_OPS_HOST"),
        'PORT': os.getenv("LEARN_OPS_PORT"),
    }
}

ENGINE value: django.db.backends.postgresql_psycopg2 — Django's PostgreSQL backend (uses the psycopg2 driver). The connection values themselves are pulled from environment variables at runtime, sourced from .env.

2. Model fields vs. SQL columns — Book

Model defined in LearningAPI/models/coursework/book.py:4-9. I described the live table (LearningAPI_book) directly in Postgres to get the ground tm the Python:

┌──────────────┬─────────────┬───────────────────────────────────────────────┐
│ Python field │ SQL column  │   SQL type   │                            Notes                            │
│    (model)   │             │                                               │
├──────────────┼─────────────┼──────────────┼─────────────────────────────────────────────────────────────┤
│ id           │ id          │ integer  efault as identity                   │
│ (implicit)   │             │              │                                                             │
├──────────────┼─────────────┼───────────────────────────────────────────────┤
│ name         │ name        │ character    │ From CharField(max_length=75)                               │
│              │             │ varying(7                                     │
├──────────────┼─────────────┼──────────────┼─────────────────────────────────────────────────────────────┤
│              │             │          ends _id to the field name for the   │
│ course       │ course_id   │ integer      │ actual column, and adds an index                            │
│              │             │          e_id_b8c297d0) + FK constraint to    │
│              │             │              │ LearningAPI_course(id)                                      │
├──────────────┼─────────────┼───────────────────────────────────────────────┤
│ description  │ description │ text         │ From TextField(default='')                                  │
├──────────────┼─────────────┼───────────────────────────────────────────────┤
│ index        │ index       │ integer      │ From IntegerField(default=0)                                │
└──────────────┴─────────────┴───────────────────────────────────────────────┘

Note projects and has_assessment in the thods (book.py:11-17) — they computevalues from related tables at access time and have no corresponding column in LearningAPI_book.

3. book.save() in create() — actual generated SQL

book_view.py:15-34 builds an unsaved Book(), sets three fields plus the course FK, then calls book.save() at
line 30.

I turned on Django's query logging and rgainst the real database. The ORMgenerated:

INSERT INTO "LearningAPI_book" ("name", "course_id", "description", "index")
VALUES ('test name', 1, 'test desc', 1)
RETURNING "LearningAPI_book"."id"

Why this shape specifically:
- Since book is a brand-new instance witws this must be an INSERT, not an UPDATE(it doesn't do a SELECT first to check — it infers from whether pk is None).
- book.course = course (an assigned modejango unwraps the FK object into course_id in the column list.                                                                                        - RETURNING ... "id" is Postgres-specifid-trip to fetch the new auto-generated PK, Django asks Postgres to hand it back in the same statement, then sets book.id in Python from that returned value — which is why serializer.data on ct id immediately after save().
```

## 4. Generate a Database Diagram

```
Generate an entity-relationship diagram of the database as a Mermaid erDiagram.
  Look at all the model files in LearningAPI/models/.
  Include every table, every field, and every relationship between tables.
  Output only the Mermaid code block.
```

```
This is a large task — every model file across the whole LearningAPI/models/ tree. Let me survey the directory structure first, then read all model files.

  Searched for 1 pattern

44 model files. I'll dispatch parallel Explore agents to read them in batches by directory and extract structured field/relationship data, then I'll assemble the Mermaid diagram myself.

⏺ 3 background agents launched (↓ to manage)
Extract model fields from coursework models
Extract model fields from people models
Extract model fields from skill models and tag.py

Three research agents are running in parallel over the coursework, people, and skill/tag model directories. I'll wait for them to finish before assembling the diagram.
Invalid tool parameters
Invalid tool parameters
Invalid tool parameters

⏺ Agent "Extract model fields from coursework models" finished · 17s

Coursework agent is done. Waiting on the other two (people, skill/tag).

⏺ Agent "Extract model fields from skill models and tag.py" finished · 19s

Two of three done. Still waiting on the people models agent — the largest set (21 files).
Message queued for delivery to a8a290e242038aedd at its next tool round.

⏺ Agent "Extract model fields from people models" finished · 38s

All three batches are in. Now I'll assemble the complete Mermaid erDiagram, combining fields, PKs, FKs, and relationship cardinalities across all 44 models.

⏺ Write(/private/tmp/claude-501/-Users-lf-workspace-lms-learn-ops-api/62d837bd-c731-4a75-95c8-c5166fa4e008/scratchpad/erd.md)
Wrote 1 lines to ../../../../../private/tmp/claude-501/-Users-lf-workspace-lms-learn-ops-api/62d837bd-c731-4a75-95c8-c5166fa4e008/scratchpad/erd.md
scratch
```

## 5. Find Relationship Examples

```
❯ Find one example each of a one-to-one, one-to-many, and many-to-many relationship
  in the Django models in LearningAPI/models/.
  For each example give me the file path and the name of the field that defines the relationship.

Based on the model extraction already done, here's one clean example of each relationship type:

One-to-One
- File: LearningAPI/models/people/cohort_info.py
- Field: cohort = models.OneToOneField("Cohort", on_delete=models.CASCADE, related_name="info")
- Each Cohort has exactly one CohortInfo record (and vice versa) — Django's OneToOneField enforces uniqueness on the FK column, unlike a regular ForeignKey.

One-to-Many
- File: LearningAPI/models/coursework/book.py
- Field: course = models.ForeignKey("Course", on_delete=models.CASCADE, related_name="books")
- One Course can have many Books, but each Book belongs to exactly one Course. This is the plain ForeignKey — the most common relationship type in this codebase.

Many-to-Many
- File: LearningAPI/models/people/assessment.py
- Field: objectives = models.ManyToManyField("LearningWeight", through="AssessmentWeight")
- An Assessment can target many LearningWeights, and a LearningWeight can apply to many Assessments. It uses an explicit through model (AssessmentWeight, defined in LearningAPI/models/skill/assessment_weight.py) rather than Django's auto-generated join table — likely because AssessmentWeight needs its own identity/fields beyond just the two FKs.
```
