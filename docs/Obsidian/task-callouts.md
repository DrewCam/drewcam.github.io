# Task callouts

I was asked to share my task callouts, so here they are. I use these to manage my tasks in Obsidian. I tag my tasks with `#task` and use the following callouts to manage them.

plugins
- Dataview
- tasks

```markdown
---
>[!Failure]- **⚠️ Overdue**
>```tasks
>not done
>happens before today
>sort by due
>hide recurrence rule
>```
---
>[!Success]- **Today**
>```tasks
>not done
>happens today
>sort by due
>sort by scheduled
>sort by done
>hide recurrence rule
>```
---
>[!info]- **This Week**
>```tasks
>not done
>happens after yesterday
>happens before in 1 week
>sort by due
>sort by scheduled
>group by happens
>hide recurrence rule
>```
---
>[!Caution]- **Next Week** 
>```tasks
>not done
>happens after today
>happens before in 1 week
>sort by urgency
>group by due
>hide recurrence rule
>hide task count
>```
---
>[!NOTE]- **All Uncompleted Tasks**
>```tasks
>not done
>path does not include 900
>group by happens
>```
---
>[!error]- **Needs scheduling**
>```tasks
>(not done) AND ((no due date) AND (no scheduled date)) AND NOT (path includes Supporting Files)
>sort by path
>hide recurrence rule 
>```
---
>[!error]- **Invalid Data**
>```tasks
>(done date is invalid) OR (due date is invalid) OR (scheduled date is invalid) OR (start date is invalid)
>
># Optionally, uncomment this line and exclude your templates location
># path does not include _templates
>
>group by path
>```
---
>[!Example]- **All Completed Tasks**
>```dataview
>TASK
>where completed & due != null
>
>```
```
