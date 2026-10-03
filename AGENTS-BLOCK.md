## Agent Register (required for every task)

Before starting:
1. Sync .register/ from shared storage (see Sync below).
2. Read .register/tasks/index. Find this task by ID or title.
   - Found: open it. Not found: create tasks/T-<YYYYMMDD>-<initials>-<n>-<slug>.md
     and add one index line. <n> is the next number for that date and initials.
   - Claim it: set claimed_by to yourself and claimed_at to now (UTC).
   - If someone else holds a claim updated in the last 4 hours, stop and ask.
     If the claim is older, you may take it over; say so in the memory file.
3. Read memory/<task id>.md and every mistakes/ entry with status: active whose
   tags match this task. Follow their "Rule for next time".

While working:
- Never edit another task's files. Never create a second task for the same work;
  add follow-ups as children of the existing task.
- Save outputs under artifacts/<task id>/.
- At each milestone, update memory/<task id>.md and the task's updated date.

When finished:
4. Update the task file: status, artifacts, mistakes. Clear claimed_by and claimed_at.
5. Finish memory/<task id>.md: what was done, decisions, files touched, next step,
   and commit: <current HEAD>.
6. If anything went wrong, add mistakes/M-<YYYYMMDD>-<initials>-<n>-<slug>.md
   with status: active, linked to this task.
7. Update only this task's lines in each index, then sync back.

Initials: <your initials, e.g. sz>
Sync: <command for your storage, e.g. rclone sync .register remote:project/.register>
