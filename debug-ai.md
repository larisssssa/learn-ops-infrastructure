# Step 2: Claude's trace of the update method

## Bug in `student_view.py`: slack handle not saving despite success response

### The bug

`StudentViewSet.update` (lines 77–100):

```python
def update(self, request, pk=None):
    try:
        student = NssUser.objects.get(pk=pk)

        if request.auth.user == student.user or request.auth.user.is_staff:
            if "slack_handle" in request.data:
                student.slack_handle = request.data["slack_handle"]   # line 88
            if "gitub_handle" in request.data:
                student.gitub_handle = request.data["gitub_handle"]   # line 90

            return Response(None, status=status.HTTP_204_NO_CONTENT)  # line 92
```

It sets the attribute on the in-memory Python object but never calls `student.save()`. Without `save()`, Django never issues the `UPDATE` SQL, so nothing persists. The object goes out of scope at the end of the request and the change is discarded.

It still returns `204 No Content` — a success status — regardless. That's the "invisible save": success response, no persistence.

### Compare to working methods in the same file

Every other write path calls `.save()`:

- `latest_assessment.save()` — line 250
- `student_assessment.save()` — line 321
- `student_project.save()` — line 399

`update` is the only mutation path that skips it.

### A second bug (not your immediate symptom, but adjacent)

Lines 89–90:

```python
if "gitub_handle" in request.data:
    student.gitub_handle = request.data["gitub_handle"]
```

`gitub_handle` is missing the `h` — both as the request key and the attribute name. The real field is `github_handle` (see the serializer at line 641, and the raw SQL at line 185). This only affects the GitHub handle field, not the Slack handle, but it means even after fixing `.save()`, that branch would still silently no-op.

### Fix

```python
if request.auth.user == student.user or request.auth.user.is_staff:
    if "slack_handle" in request.data:
        student.slack_handle = request.data["slack_handle"]
    if "github_handle" in request.data:          # fix typo
        student.github_handle = request.data["github_handle"]

    student.save()                                # persist the change

    return Response(None, status=status.HTTP_204_NO_CONTENT)
```

### Takeaway

A 2xx status only means "the view finished without raising an exception" — it says nothing about whether data was written. In Django's ORM, assigning to a model attribute is plain Python; the database only changes on `.save()` (or `.update()`, `.create()`, `bulk_*`, etc.). When you see "says success but nothing changed," check for a missing ORM write call first.

# Step 3: Comparison to debugger findings

Claude's output pointed out the same error as I did in [debug-notes](/learn-ops-infrastructure/debug-notes.md), showing that the error is the missing `.save()` for the student object before returning a response.

# Step 5: Test failure output

```bash
 ~/workspace/lms/learn-ops-infrastructure   main ±  docker compose exec api pytest LearningAPI/tests/test_student_update.py -v

================================== test session starts ===================================
platform linux -- Python 3.11.11, pytest-9.0.3, pluggy-1.6.0 -- /usr/local/bin/python3
cachedir: .pytest_cache
django: version: 5.2.13, settings: LearningPlatform.test_settings (from ini)
rootdir: /app
configfile: pytest.ini
plugins: django-4.12.0, cov-7.1.0
collected 1 item

LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database FAILED [100%]

======================================== FAILURES ========================================
_________ StudentPartialUpdateTests.test_patch_slack_handle_persists_to_database _________
LearningAPI/tests/test_student_update.py:37: in test_patch_slack_handle_persists_to_database
    self.assertEqual(nss_user.slack_handle, "@newhandle")
E   AssertionError: '@old_handle' != '@newhandle'
E   - @old_handle
E   + @newhandle
--------------------------------- Captured stderr setup ----------------------------------
Using existing test database for alias 'default' ('file:memorydb_default?mode=memory&cache=shared')...
================================ short test summary info =================================
FAILED LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database - AssertionError: '@old_handle' != '@newhandle'
- @old_handle
+ @newhandle
=================================== 1 failed in 1.42s ====================================

```

# Step 6: Ask for the fix

The single fix is adding the missing `.save()` call in `student_view.py`, right after the field assignments in `update` (currently lines 87–91):

```python
if "slack_handle" in request.data:
    student.slack_handle = request.data["slack_handle"]
if "gitub_handle" in request.data:
    student.gitub_handle = request.data["gitub_handle"]

student.save()
```

Add `student.save()` on the line right before `return Response(None, status=status.HTTP_204_NO_CONTENT)` (line 92). That's the one line that makes the slack handle change actually persist to the database.

# Step 7: Test Passing Output

```bash
 ~/workspace/lms/learn-ops-infrastructure   main ±  docker compose exec api pytest LearningAPI/tests/test_student_update.py -v

================================== test session starts ===================================
platform linux -- Python 3.11.11, pytest-9.0.3, pluggy-1.6.0 -- /usr/local/bin/python3
cachedir: .pytest_cache
django: version: 5.2.13, settings: LearningPlatform.test_settings (from ini)
rootdir: /app
configfile: pytest.ini
plugins: django-4.12.0, cov-7.1.0
collected 1 item

LearningAPI/tests/test_student_update.py::StudentPartialUpdateTests::test_patch_slack_handle_persists_to_database PASSED [100%]

=================================== 1 passed in 1.40s ====================================

```
