Great question — this gets into the internal schema of Rundeck’s database.
Yes — **you can distinguish “user jobs”** (manually created or triggered by users) from **system or scheduled executions** by looking at a few key fields in the Rundeck DB — mainly in the `execution` and `scheduled_execution` tables.

Let’s go step by step 👇

---

## 🧩 Key Tables

| Table                 | Purpose                                                                  |
| --------------------- | ------------------------------------------------------------------------ |
| `scheduled_execution` | Stores **job definitions** (name, project, owner, schedule, etc.)        |
| `execution`           | Stores **each job run**, including who ran it, when, how, and the result |
| `auth_token`, `user`  | Store user and API token info (if you need to trace who triggered it)    |

---

## ⚙️ Distinguishing “User Jobs” in the Database

### 1. Check if the execution is linked to a defined job

In the `execution` table:

| Column                         | Meaning                                                                                   |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| `scheduled_execution_id`       | FK to the `scheduled_execution` table. Non-null if this run corresponds to a defined job. |
| `user`                         | The Rundeck username that triggered the job.                                              |
| `server_nodeuuid`              | Rundeck server UUID that ran the job.                                                     |
| `date_started`, `status`, etc. | Execution metadata.                                                                       |

If `scheduled_execution_id` is **NULL**, that means the execution was **an ad-hoc command** (e.g., a user ran a one-off command via the Rundeck UI or API).

✅ **Example query:**

```sql
SELECT id, user, scheduled_execution_id, status, date_started
FROM execution
ORDER BY date_started DESC;
```

Interpretation:

* `scheduled_execution_id` **not null** → job run (could be user-triggered or scheduled)
* `scheduled_execution_id` **null** → **ad-hoc command**

---

### 2. Determine if a job run was **user-triggered** or **scheduled**

To distinguish *manual* vs *scheduled* runs, look at the `execution_type` or `date_scheduled` fields (depending on Rundeck version).

| Column           | Meaning                                             |
| ---------------- | --------------------------------------------------- |
| `execution_type` | `'user'`, `'scheduled'`, `'retry'`, `'adhoc'`, etc. |
| `date_scheduled` | Non-null if triggered by a schedule (cron).         |

✅ **Example query:**

```sql
SELECT id, user, execution_type, date_scheduled, status
FROM execution
ORDER BY date_started DESC;
```

Typical values for `execution_type`:

* `user` → manually run via UI or API
* `scheduled` → triggered by Rundeck’s internal scheduler
* `adhoc` → direct command (not part of a saved job)
* `retry` → user retried a previous execution

---

### 3. See who created the job (for job definitions)

In `scheduled_execution`:

| Column       | Meaning                                                  |
| ------------ | -------------------------------------------------------- |
| `id`         | Job ID (matches `scheduled_execution_id` in `execution`) |
| `job_name`   | Name of the job                                          |
| `project`    | Project name                                             |
| `user`       | The user who created or last modified the job            |
| `group_path` | Folder/group path in the Rundeck UI                      |

✅ Example query:

```sql
SELECT id, job_name, project, user, group_path
FROM scheduled_execution
ORDER BY date_created DESC;
```

---

## 🧠 Putting It All Together

Here’s a join to see recent executions **with their job name and who ran them**:

```sql
SELECT
  e.id AS execution_id,
  e.user AS executed_by,
  e.execution_type,
  e.status,
  e.date_started,
  se.job_name,
  se.project
FROM execution e
LEFT JOIN scheduled_execution se
  ON e.scheduled_execution_id = se.id
ORDER BY e.date_started DESC
LIMIT 20;
```

---

## ✅ Summary

| Type                               | How to Identify                                                                    |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| **User-triggered job**             | `execution.execution_type = 'user'`                                                |
| **Scheduled job**                  | `execution.execution_type = 'scheduled'` or `execution.date_scheduled IS NOT NULL` |
| **Ad-hoc command**                 | `execution.scheduled_execution_id IS NULL`                                         |
| **Job definition (owned by user)** | `scheduled_execution.user = '<username>'`                                          |

---

Would you like me to show a **full SQL query** that lists all user-triggered job runs (including project, job name, user, and start time)? That’s a common one for reporting or dashboards.
