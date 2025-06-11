If you're using **OpenShift on-premises**, the approach to securing your PostgreSQL backups and enabling WAL encryption is similar, but with some adjustments for on-premise infrastructure limitations (e.g., no cloud-native storage or KMS services). Here's how to ensure:

- ✅ **PostgreSQL WAL Encryption**
- ✅ **Encrypted Database Backups**
- ✅ **Secure Storage of Backup Data**

---

## 🔐 1. Enable WAL Encryption in PostgreSQL

Starting from **PostgreSQL v16**, native support for WAL encryption is available.

### ✅ Steps:
1. **Ensure PostgreSQL version ≥ 16**
   - Use an image like `registry.developers.crunchydata.com/crunchydata/crunchy-postgres:ubi8-16.2-0` if using Crunchy Operator.
   - Or build your own container image if not using an operator.

2. **Edit `postgresql.conf`:**
   ```conf
   wal_level = replica
   wal_encryption = aes256
   ```

3. **Reload PostgreSQL:**
   ```bash
   pg_ctl reload
   ```
   In OpenShift:
   ```bash
   oc exec -it <postgres-pod> -- pg_ctl reload
   ```

4. **Verify:**
   ```sql
   SHOW wal_encryption;
   ```

✅ This ensures that all WAL segments are encrypted before being written to disk.

---

## 📦 2. Encrypted Database Backups on On-Prem OpenShift

Since you don’t have access to cloud providers like AWS or Azure, here are options tailored for on-prem environments.

---

### 🔁 A. Logical Backup (`pg_dump`) with Encryption

#### ✅ Example using `pg_dump + openssl`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: postgres-backup
spec:
  template:
    spec:
      containers:
        - name: postgres
          image: postgres:16
          env:
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
          command:
            - sh
            - -c
            - |
              mkdir -p /backup
              pg_dump -h <host> -U <user> -Fc <dbname> | \
              openssl aes-256-cbc -salt -k "mysecretpassword" > /backup/dbname.dump.enc
              # Optionally copy to a secure volume or external server
      volumes:
        - name: backup-volume
          persistentVolumeClaim:
            claimName: backup-pvc
      restartPolicy: OnFailure
```

#### 🔒 Notes:
- Store the encryption password securely via Kubernetes Secrets.
- Mount a **secure PVC** (with encrypted filesystem) for storing backups.

---

### 💾 B. File System Snapshot with Encrypted Volumes

You can use **Velero with Restic** to back up PVs even on-prem.

#### 🔧 Setup Steps:

1. **Install Velero + Restic on OpenShift**
   - Follow [Velero Install Guide](https://velero.io/docs/v1.13/basic-install/)

2. **Annotate PostgreSQL Pod to enable Restic:**
   ```yaml
   metadata:
     annotations:
       backup.velero.io/backup-volumes: data-volume
   ```

3. **Create Backup Schedule:**
   ```bash
   velero schedule create daily-backup --schedule="@daily"
   ```

#### 🔒 Encrypting Persistent Volumes:

Use **LUKS** or **file system-level encryption** on the underlying nodes or storage class.

- If using NFS, consider mounting with **encrypted filesystems (e.g., eCryptfs or LUKS)**.
- Alternatively, encrypt at the application level (i.e., inside the backup process).

---

### 🔄 C. Continuous Archiving (WAL Streaming + PITR)

For point-in-time recovery, archive WAL logs to a local or remote secure location.

#### ✅ Configuration:
In `postgresql.conf`:
```conf
wal_level = replica
archive_mode = on
archive_command = 'gzip -c %p > /mnt/wal_archive/%f.gz'
```

#### 🔒 Secure WAL Archive Location:
- Mount `/mnt/wal_archive` as an **encrypted volume** (e.g., LUKS).
- Automate rotation and offsite transfer (e.g., rsync over SSH or sftp).
- Rotate keys regularly if using manual encryption.

---

## 🔒 3. Infrastructure-Level Encryption (On-Prem)

Since you're on-prem, you must handle encryption manually or via your infrastructure.

### Options:

| Layer | Solution |
|-------|----------|
| **Storage** | Use LUKS-encrypted disks or file systems |
| **PVCs** | Use CSI drivers with encryption support (e.g., OCS with encryption enabled) |
| **Network** | Enforce TLS for all DB connections |
| **Secrets** | Use Kubernetes Secrets or integrate with HashiCorp Vault |
| **Backups** | Always encrypt backup files using AES before saving |

---

## ✅ 4. Best Practices for On-Prem OpenShift

| Area | Recommendation |
|------|----------------|
| 🔑 Secret Management | Use Kubernetes Secrets or Vault |
| 🛡️ Access Control | RBAC + network policies to isolate Postgres |
| 🗂️ Backup Retention | Keep multiple versions and test restores |
| 📜 Audit Logs | Enable logging and forward to internal SIEM or log store |
| 🔄 Automation | Use CronJobs or Operators for scheduled tasks |
| 🧪 Testing | Regularly test restore process from backup |

---

## 🧱 Optional: Use Crunchy Data Postgres Operator

Crunchy offers enterprise-grade Postgres management on OpenShift and supports:

- Encrypted backups
- WAL archiving
- Integrated TLS
- Automated jobs

[Crunchy Postgres for Kubernetes Docs](https://docs.crunchydata.com/postgres-operator/latest/)

---

## 📌 Summary

| Goal | Implementation |
|------|----------------|
| WAL Encryption | PostgreSQL v16+, `wal_encryption = aes256` |
| Encrypted Backups | `pg_dump + openssl`, encrypted PVCs |
| Backup Strategy | Logical dumps, Velero/Restic snapshots, WAL archiving |
| Secure Storage | LUKS, encrypted filesystems, secure PVCs |
| Automation | Kubernetes Jobs/CronJobs or Crunchy Operator |

---

Would you like me to provide a full working example using `CronJob + pg_dump + openssl` or help you configure **Velero with Restic** on your on-prem OpenShift cluster?
