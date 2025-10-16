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



Perfect — let’s get precise.
In **Rundeck’s database**, job *logs* (the execution output: stdout/stderr from your job steps) can be stored in several different ways depending on your **log storage configuration**.

Here’s the full breakdown 👇

---

## 🧩 Where Job Logs Are Stored in Rundeck DB

### 🧱 Key Tables (if DB log storage is enabled)

| Table                             | Purpose                                                           | Notes                                           |
| --------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------- |
| **`execution`**                   | Metadata for each run (start time, user, status, etc.)            | Links to logs via `id` → `execution_id`         |
| **`rdlog`**                       | Stores raw job output lines (log text)                            | Each execution has one or more log entries here |
| **`execution_log_state`**         | Tracks log storage completion, upload status, and retrieval state | Used by the log storage subsystem               |
| **`base_report` / `exec_report`** | Summary of job outcomes                                           | For reporting; not raw logs                     |

> ⚠️ Note: Depending on Rundeck version (especially after 3.x), `rdlog` might be replaced or supplemented by `execution_output` or external storage references.

---

## 🗄️ Common Configurations

### 1. **Database Log Storage (less common, small setups)**

If you see this in `rundeck-config.properties`:

```properties
rundeck.execution.logs.fileStoragePlugin=db
```

→ Logs are stored **directly in the database**, in the `rdlog` table.

Example query:

```sql
SELECT e.id AS execution_id,
       e.job_id,
       e.user,
       e.status,
       l.log,
       l.completed
FROM execution e
JOIN rdlog l ON e.id = l.execution_id
WHERE e.id = 1234;
```

Here:

* `rdlog.log` → contains the job’s console output (text)
* `completed` → true when Rundeck finished storing the log

---

### 2. **Filesystem Log Storage (default in most setups)**

If you see:

```properties
rundeck.execution.logs.fileStoragePlugin=filesystem
rundeck.execution.logs.fileStorageDir=/var/lib/rundeck/logs
```

→ Logs **are not stored in the DB**, but in the local filesystem.

Typical log path:

```
/var/lib/rundeck/logs/rundeck/<project>/job/<job-id>/log/<execution-id>.rdlog
```

The DB (`execution` table) still keeps a **reference** to the log, but not the content itself.

You can confirm by checking `execution.output_filepath` or `execution.output_file` (depends on Rundeck version):

```sql
SELECT id, output_filepath, status FROM execution WHERE id = 1234;
```

---

### 3. **External (S3, Azure, etc.) Log Storage**

If configured like this:

```properties
rundeck.execution.logs.fileStoragePlugin=s3
```

→ The log files are uploaded to S3 (or another plugin target).
In this case:

* The `execution_log_state` table stores references and flags (`plugin_name`, `state`, `last_updated`)
* No raw log data is in the DB

Example:

```sql
SELECT * FROM execution_log_state WHERE execution_id = 1234;
```

---

## 🧠 Quick Reference Summary

| Table                       | Contains                                    | When Used                         |
| --------------------------- | ------------------------------------------- | --------------------------------- |
| `execution`                 | Job metadata (ID, user, status, timestamps) | Always                            |
| `rdlog`                     | Actual log text (stdout/stderr)             | Only if DB log storage is enabled |
| `execution_log_state`       | Log upload and completion state             | Always, for tracking              |
| `execution.output_filepath` | Log file reference (if filesystem)          | Always if file-based logs         |

---

## ✅ Quick Check Command (to know your mode)

On your Rundeck server:

```bash
grep "rundeck.execution.logs.fileStoragePlugin" /etc/rundeck/rundeck-config.properties
```

* `db` → logs are in the Rundeck DB (`rdlog` table)
* `filesystem` → logs are on disk
* `s3`, `azure`, etc. → logs are external

---

Would you like me to give you a **ready-to-use SQL query** that returns the *latest 10 job runs* including **log content (if stored in DB)** and **user/job name**?
