# Trace Notes (AI): assessments (learn-ops-api)

### Request path table from Claude

| Layer      | File                                                                | Class / Function                                                       | What it does                                                                                                                                       |
| ---------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| UI dialog  | `learn-ops-client/src/components/people/StudentAssessmentForm.js`   | `StudentAssessmentForm`                                                | Dialog where an instructor reviews a student's assessment; on save calls `PUT /students/{id}/assess`                                               |
| API helper | `learn-ops-client/src/components/assessments/AssessmentProvider.js` | `getStudentAssessments(studentId)`                                     | Calls `GET /assessments?studentId={id}` via `fetchIt`, stores results in `studentAssessments` context state                                        |
| URL router | `learn-ops-api/LearningPlatform/urls.py:26`                         | `router.register(r'assessments', StudentAssessmentView, 'assessment')` | Registers the DRF DefaultRouter route; maps `/assessments` and `/assessments/<pk>` to the viewset                                                  |
| View       | `learn-ops-api/LearningAPI/views/student_assessment.py:38`          | `StudentAssessmentView(ViewSet)`                                       | Handles list (filtered by `studentId`), retrieve, create (instructor-only), and destroy; enforces `StudentAssessmentPermission`                    |
| Serializer | `learn-ops-api/LearningAPI/views/student_assessment.py:168`         | `StudentAssessmentSerializer`                                          | Serializes `StudentAssessment` with nested `AssessmentSerializer` (id, name, objectives) and computed `status` and `instructor_username` fields    |
| DB         | `learn-ops-api/LearningAPI/models/people/student_assessment.py`     | `StudentAssessment`                                                    | ORM model linking `NssUser` (student) → `Assessment` → `StudentAssessmentStatus`; queried with `filter(student=student).order_by('-date_created')` |
| UI refresh | `learn-ops-client/src/components/people/StudentDetails.js`          | `useEffect` → `getStudentAssessments`                                  | Re-fetches assessments when `activeStudent` changes, repopulating the context and re-rendering the assessment list                                 |

### Sequence Diagram

```mermaid
sequenceDiagram
    participant UI as StudentDetails (React)
    participant Form as StudentAssessmentForm
    participant Provider as AssessmentProvider
    participant Fetch as fetchIt
    participant View as StudentAssessmentView
    participant Serializer as StudentAssessmentSerializer
    participant DB as PostgreSQL (StudentAssessment)

    UI->>Provider: getStudentAssessments(studentId)
    Provider->>Fetch: fetchIt("/assessments?studentId={id}", {method: "GET"})
    Fetch->>View: GET /assessments?studentId={id}
    View->>DB: StudentAssessment.objects.filter(student=student).order_by('-date_created')
    DB-->>View: QuerySet of StudentAssessment rows
    View->>Serializer: serialize(queryset, many=True)
    Serializer-->>View: JSON array with nested assessment + status
    View-->>Fetch: 200 OK + JSON
    Fetch-->>Provider: parsed JSON array
    Provider-->>UI: setStudentAssessments(data)

    UI->>Form: open review dialog (passes studentAssessments)
    Form->>Provider: changeStatus(assessmentId, statusId)
    Provider->>Fetch: fetchIt("/assessments/{id}", {method: "PUT", body: {status: statusId}})
    Fetch->>View: PUT /assessments/{id}
    View->>DB: StudentAssessment.objects.get(pk=pk) → save status
    DB-->>View: updated StudentAssessment
    View->>Serializer: serialize(updated_assessment)
    Serializer-->>View: JSON object
    View-->>Fetch: 200 OK + JSON
    Fetch-->>Provider: parsed JSON
    Provider-->>UI: re-fetch / update studentAssessments state
```
