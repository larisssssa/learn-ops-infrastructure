# learn-ops-api: Service Exploration

## 1. Top-level folders in `learn-ops-api`

| Folder           | Why does this folder need to exist?                                              |
| ---------------- | -------------------------------------------------------------------------------- |
| config           | config for deployment                                                            |
| LearningAPI      | main app - contains fixtures, migrations, models, serializers, and views modules |
| LearningPlatform | project configurations - contains setting, urls, and wsgi modules.               |
| LogViewer        | separate app for viewing logs                                                    |

## 2. Folders inside `LearningAPI`

| Folder      | What responsibility does it own and why?                    |
| ----------- | ----------------------------------------------------------- |
| fixtures    | data to be seeded into the database                         |
| migrations  | tracks changes to the database                              |
| models      | database structure                                          |
| serializers | formatter for the models, creates JSON output               |
| views       | API endpoints, handles HTTP requests and returns a response |

## 3. What is the Pipfile?

    The Pipfile is the dependency file for a Python project. It lists packages and the required versions.

## 4. Key packages in the Pipfile

| Package             | What functionality does it provide and why?                                                   |
| ------------------- | --------------------------------------------------------------------------------------------- |
| django              | python framework for creating web applications                                                |
| djangorestframework | Turns django into a framework for building JSON APIs. Adds serializers, ViewSets, and routers |
| django-allauth      | authenticantion package - handles login, registration, email verification, pw reset, Oauth    |

## 5. What does `decorators.py` do?

    decorators.py has two methods "is_instructor" and "is_staff" which checks for the current users role and whether the user is allowed to perform the CRUD action.

## 6. What is a serializer, and what serializers are defined here?

    Serializers are the translator for the app. Serializers can take python data and translate it into JSON format for the frontend to use, and also accepts JSON from API requests and changes them into python data for the database.

## 7. Models and what they represent

| Model      | Real-world thing it represents                                            |
| ---------- | ------------------------------------------------------------------------- |
| coursework | book, capstone, course, exercises, learning objective, project, proposals |
| people     | cohort, assessments, users, groups, student notes and info                |
| skill      | learning records, assessment weighting, core skills                       |

## 8. Views vs. viewsets

| Type    | Example class | File path                                       |
| ------- | ------------- | ----------------------------------------------- |
| View    | GithubLogin   | learn-ops-api/LearningAPI/views/github_login.py |
| ViewSet | BookViewSet   | learn-ops-api/LearningAPI/views/book_view.py    |

## 9. Serializers paired with their models

| Serializer        | Model  | Link |
| ----------------- | ------ | ---- |
| user_serializer   | User   |      |
| cohort_serializer | Cohort |      |

## 10. What replaces the Templates and why?

    The Templates are replaced by the views and models. This django app does not render HTML so the models provides the data and the views handles the API requests. That data is sent to the front end which handles the HTML rendering.
