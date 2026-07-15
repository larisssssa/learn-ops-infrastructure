# Step 3: /conventions learn-ops-client

## Linting Tools in `learn-ops-client`

### ESLint (via Create React App / react-scripts)

- **Command:** `npx eslint ~/workspace/lms/learn-ops-client/src` (or, more idiomatically for this project, `npm run build` / `npm start` inside `~/workspace/lms/learn-ops-client` — CRA runs ESLint automatically as part of `react-scripts build`/`start`, since there's no standalone `lint` script defined)
- **Config location:** `~/workspace/lms/learn-ops-client/package.json:39-41`, under the `"eslintConfig"` key:

  ```json
  "eslintConfig": {
    "extends": "react-app"
  }
  ```

  (An identical copy also exists in `package.json.bak:1-4`, an apparent backup file.)

- **What "react-app" enforces:** This extends the `eslint-config-react-app` preset that ships with `react-scripts` (declared as a dependency, `"react-scripts": "^5.0.1"`). It bundles several rule categories:
  - **React-specific correctness** (`eslint-plugin-react`) — e.g. `react/jsx-no-undef` (flags undefined JSX components), `react/jsx-uses-vars`, `react/no-direct-mutation-state`
  - **React Hooks rules** (`eslint-plugin-react-hooks`) — e.g. `react-hooks/rules-of-hooks` (hooks must be called at the top level, not in loops/conditionals), `react-hooks/exhaustive-deps` (warns on missing dependencies in `useEffect`/`useCallback`/etc.)
  - **General JS correctness** (core ESLint + `eslint-plugin-import`) — e.g. `no-undef` (flags references to undeclared variables), `no-unused-vars`, `default-case`, `no-array-constructor`
  - **Accessibility** (`eslint-plugin-jsx-a11y`) — e.g. `jsx-a11y/alt-text`, `jsx-a11y/anchor-is-valid`

- **Rules explicitly disabled in this codebase:** None are disabled at the config level (`package.json` only sets `"extends"`, no `"rules"` overrides). However, the rule `no-undef` is locally suppressed inline via comments in two files, rather than in config:
  - `src/components/cohorts/StudentCardList.js:152` — `/* eslint-disable no-undef */`
  - `src/components/cohorts/StudentCapstoneList.js:55` — `/* eslint-disable no-undef */`

  Without these comments, ESLint would flag any variable/identifier used in that code block that isn't declared, imported, or a recognized global — likely because the file references a JSX component or global not explicitly imported.

  Two additional blanket `eslint-disable-next-line` comments (no specific rule named, so they suppress _all_ rules for that line) appear at:
  - `src/components/utils/HumanDate.js:21`
  - `src/components/dashboard/student/StudentDashboard.js:20`

### Python linters (Pylint)

Not applicable — no `.pylintrc`, no `[pylint]` section in `setup.cfg`, and no `[tool.pylint]` section in `pyproject.toml` exist anywhere in the project. This is a JavaScript/React codebase with no Python code.

---

# Step 4: /conventions learn-ops-api

I have everything needed. Since a raw markdown format was requested previously, I'll continue that formatting.

## Linting Tools in `learn-ops-api`

### Pylint (with pylint-django)

- **Command:** `pylint --rcfile=~/workspace/lms/learn-ops-api/.pylintrc ~/workspace/lms/learn-ops-api` (run from within the project's virtualenv/Pipenv environment, since `pylint` and `pylint-django` are declared as dependencies in the `Pipfile`, not installed globally)
- **Config location:** `~/workspace/lms/learn-ops-api/.pylintrc`

  ```ini
  [FORMAT]
  good-names=i,j,ex,pk

  [MESSAGES CONTROL]
  disable=broad-except,arguments-differ,missing-function-docstring
  ```

- **What it enforces:** Pylint's default rule set covers several categories, scoped here for a Django project (via `pylint-django`, also in the `Pipfile`):
  - **Naming conventions** (`C0103` invalid-name) — variable/argument names must be descriptive by default; the `good-names` setting explicitly whitelists `i`, `j`, `ex`, `pk` as exceptions to this rule (otherwise short names like a loop counter `i` or Django's `pk` would be flagged)
  - **Code smells / design** (`R` category) — e.g. `R0913` too-many-arguments, `R0902` too-many-instance-attributes, `R0201` no-self-use
  - **Errors / correctness** (`E` category) — e.g. `E1101` no-member (flags attribute access that pylint can't statically verify exists — common with Django ORM dynamic fields), `E0307` invalid-str-returned (flags `__str__` methods that don't return a `str`)
  - **Convention/documentation** (`C` category) — e.g. `C0111`/missing docstrings, `C0301` line-too-long

- **Rules explicitly disabled** (config-level, via `disable=` in `.pylintrc`):
  - `broad-except` (`W0703`) — would otherwise flag `except Exception:` (or bare `except:`) clauses that catch overly broad exception types instead of specific ones
  - `arguments-differ` (`W0221`) — would otherwise flag overridden methods (e.g. Django view/serializer methods) whose signature doesn't match the parent class/interface it overrides
  - `missing-function-docstring` (`C0116`) — would otherwise flag every function/method lacking a docstring

- **Additional inline suppressions found in source** (not in config, but worth noting as local overrides):
  - `E0307` (invalid-str-returned) suppressed in `LearningAPI/models/coursework/course.py:10` and `LearningAPI/models/coursework/foundation_exercise.py:18`
  - `E1101` (no-member) suppressed in `LearningAPI/models/coursework/capstone_timeline.py:12`, `LearningAPI/models/people/assessment.py:24,31,32`, and `LearningAPI/views/team_maker_view.py:118` — these are Django ORM relationship/attribute accesses that pylint can't resolve statically
  - `E0307` also suppressed in `LearningAPI/models/people/assessment.py:35`

### ESLint / JavaScript linters

Not applicable — no `package.json`, no `.eslintrc*` file exists anywhere in the project. This is a Django/Python-only codebase (see `manage.py`, `Pipfile`, `.python-version`).

---

# Step 5: /conventions learn-ops-api

Confirmed — no compose files, `Pipfile`/`Pipfile.lock` present, `.pylintrc` present. Same result as before.

# Linting Tools in `learn-ops-api`

## Pylint (with pylint-django)

- **Command:** `pipenv run pylint --rcfile=~/workspace/lms/learn-ops-api/.pylintrc ~/workspace/lms/learn-ops-api`
  - **Evidence for this command form:** No `docker-compose.yml`/`compose.yml` exists at the repo root or in the parent directory, so this is not a Docker Compose project. `Pipfile` and `Pipfile.lock` are present at `~/workspace/lms/learn-ops-api/Pipfile`, and both `pylint` and `pylint-django` are declared as dependencies there — so the tool lives inside the Pipenv-managed virtualenv, not on the global PATH. Hence `pipenv run pylint`, not a bare `pylint` call.
- **Config location:** `~/workspace/lms/learn-ops-api/.pylintrc`

  ```ini
  [FORMAT]
  good-names=i,j,ex,pk

  [MESSAGES CONTROL]
  disable=broad-except,arguments-differ,missing-function-docstring
  ```

- **What it enforces:** Pylint's default rule set, extended by `pylint-django` for Django-aware checks:
  - **Naming conventions** — `C0103` invalid-name flags identifiers that don't match snake_case/naming conventions by default; the `good-names` setting explicitly whitelists `i`, `j`, `ex`, `pk` as exceptions (otherwise a loop counter `i` or Django's `pk` attribute name would be flagged)
  - **Errors / correctness** — `E1101` no-member flags attribute access pylint can't statically verify exists (common with Django ORM dynamic relations/fields); `E0307` invalid-str-returned flags `__str__` methods that don't return a `str`
  - **Design / code smell** — `R0913` too-many-arguments flags functions with an excessive parameter count; `R0902` too-many-instance-attributes flags classes with too many attributes

- **Rules explicitly disabled** (in `.pylintrc`, under `[MESSAGES CONTROL]`), and what they'd otherwise flag:
  - **`broad-except`** (`W0703`) — would flag `except Exception:` or a bare `except:` clause, i.e. code that catches an overly broad exception type instead of a specific one, which can silently swallow unrelated errors (e.g. a `KeyError` masking as if it were the expected `ValueError`).
  - **`arguments-differ`** (`W0221`) — would flag a method override whose signature doesn't match its parent class/interface — e.g. a Django view or serializer method (`def get(self, request, extra_arg):`) that adds/removes parameters compared to the base class method it overrides, which can break callers that rely on the base signature.
  - **`missing-function-docstring`** (`C0116`) — would flag every function or method that has no docstring immediately below its `def` line, regardless of how self-explanatory the function name/body is.

- **Additional inline suppressions found in source** (local overrides, not in config):
  - `E0307` (invalid-str-returned) suppressed in `LearningAPI/models/coursework/course.py:10`, `LearningAPI/models/coursework/foundation_exercise.py:18`, and `LearningAPI/models/people/assessment.py:35`
  - `E1101` (no-member) suppressed in `LearningAPI/models/coursework/capstone_timeline.py:12`, `LearningAPI/models/people/assessment.py:24,31,32`, and `LearningAPI/views/team_maker_view.py:118` — all Django ORM relationship/attribute accesses pylint can't resolve statically

## ESLint / JavaScript linters

Not applicable — no `package.json`, no `.eslintrc*` file exists anywhere in the project. This is a Django/Python-only codebase (see `manage.py`, `Pipfile`, `.python-version`).

---
