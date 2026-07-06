# Debug Notes: The Invisible Save

## 1. Bug report summary

What did the user report? What behavior did you observe when reproducing it?

> The User reported that when updating the slack handle, the handle saved with no issues but the Dashboard UI did not update with the changes.

## 2. What the debugger showed

| Question                                       | Answer                                                              |
| ---------------------------------------------- | ------------------------------------------------------------------- |
| Value of `student.slack_handle` before line 87 | `None`                                                              |
| Value of `student.slack_handle` after line 88  | `changed_handle`                                                    |
| Next line executed after line 90               | line 92, `return Response(None, status=status.HTTP_204_NO_CONTENT)` |

## 3. Root cause

What was the underlying reason the change was not saved? What Django concept does this come back to?
> The response is being returned before saving the changes to the database

## 4. The fix

What line did you add? Where exactly in the method?
> added `student.save()` after updating the values, before return response

## 5. What the test verifies

Describe in plain language what `test_patch_slack_handle_persists_to_database` checks and why that is the right thing to test.
> First it checks if given the correct values to update, it returns the appropriate success response. 

> Then, it gets the updated data from the database, and compares the database slack handle to the handle provided by the user.
