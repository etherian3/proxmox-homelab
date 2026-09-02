# Backup and Restore

## Overview

Backup and restore digunakan untuk melindungi persistent data dari kehilangan data, corruption, kesalahan konfigurasi, atau kegagalan service.

Database utama pada homelab menggunakan PostgreSQL yang berjalan sebagai Docker container.

Target operational workflow:

```text id="b7h2v4"
PostgreSQL
    |
    v
pg_dump
    |
    v
Backup File
    |
    v
Backup Storage
    |
    v
Restore Test
    |
    v
Verified Backup
```

Backup dianggap valid setelah proses restore berhasil dilakukan dan data dapat diverifikasi.

---

## Current Database

Database service berjalan pada:

```text
Host: application-01
Container: homelab-postgres
Image: postgres:16-alpine
Port: 5432
```

Database digunakan oleh:

```text
homelab-app
```

Communication antara application dan database dilakukan melalui Docker network internal.

---

## Backup Strategy

Backup PostgreSQL menggunakan logical backup melalui `pg_dump`.

Konsep:

```text id="h0gq9p"
PostgreSQL
     |
     | pg_dump
     v
SQL Backup
```

Logical backup dipilih karena:

- mudah dipindahkan
- mudah di-restore
- dapat diverifikasi
- tidak bergantung pada Docker container lifecycle
- cocok untuk database kecil dan homelab

---

## Backup Scope

Backup mencakup database application.

Target backup:

```text id="0c7t9k"
Database
├── Tables
├── Data
├── Schema
└── Database objects
```

Backup tidak dimaksudkan sebagai pengganti seluruh host backup.

Host-level backup dan disaster recovery akan dibahas secara terpisah.

---

## Backup Storage

Backup disimpan di storage yang terpisah dari container database.

Target structure:

```text id="5j3m7c"
/opt/backups/
└── postgres/
    ├── backup-YYYY-MM-DD_HH-MM-SS.sql
    └── ...
```

Directory backup berada di luar PostgreSQL container sehingga lifecycle database container tidak menentukan lifecycle backup.

---

## Backup Workflow

Operational workflow:

```text id="m4j8k2"
Scheduled/Manual Backup
        |
        v
Check PostgreSQL
        |
        v
Run pg_dump
        |
        v
Create backup file
        |
        v
Verify file
        |
        v
Report success/failure
```

Backup process harus gagal apabila `pg_dump` gagal.

Hal ini penting agar backup automation tidak memberikan false positive.

---

## Backup Verification

Minimal verification dilakukan dengan memastikan:

1. PostgreSQL dapat diakses.
2. `pg_dump` selesai tanpa error.
3. Backup file berhasil dibuat.
4. Backup file memiliki ukuran yang masuk akal.
5. Backup dapat digunakan untuk restore.

Contoh:

```bash
ls -lh /opt/backups/postgres/
```

---

## Restore Strategy

Restore dilakukan ke database terpisah untuk menghindari overwrite database production/current application database.

Architecture:

```text id="8j4nq1"
Existing PostgreSQL
       |
       | backup
       v
   backup.sql
       |
       | restore
       v
Test PostgreSQL
       |
       v
Data Verification
```

Dengan pendekatan ini, restore test tidak mengganggu application yang sedang berjalan.

---

## Restore Validation

Restore dianggap berhasil apabila:

```text id="n5x1q7"
Backup file
    |
    v
Restore
    |
    v
Test database
    |
    v
Connect successfully
    |
    v
Expected tables exist
    |
    v
Expected data exists
```

Restore testing lebih penting daripada hanya memastikan file backup tersedia.

---

## Backup Retention

Backup tidak boleh disimpan tanpa batas.

Future retention policy dapat menggunakan pendekatan:

```text id="e2r9v5"
Daily
  |
  +--> Keep recent backups

Weekly
  |
  +--> Keep longer-term backups

Old backups
  |
  +--> Remove according to retention policy
```

Retention policy akan disesuaikan dengan storage capacity dan kebutuhan recovery.

---

## Automation

Backup nantinya dapat dijalankan secara scheduled menggunakan:

```text id="5x9q8k"
Cron
   |
   v
Backup Script
   |
   v
pg_dump
```

Ansible dapat digunakan untuk deploy:

- backup directory
- backup script
- cron configuration
- permissions
- retention configuration

Dengan demikian backup menjadi bagian dari infrastructure configuration.

---

## Failure Handling

Backup job harus dapat membedakan:

```text id="w4m2p9"
SUCCESS
  |
  +--> Backup created
  +--> File verified


FAILURE
  |
  +--> pg_dump failed
  +--> Backup file missing
  +--> Backup file invalid
```

Failure harus menghasilkan error yang dapat dideteksi oleh operator atau monitoring system.

---

## Disaster Recovery Relationship

Backup merupakan salah satu komponen disaster recovery.

Relationship:

```text id="q2w8x6"
Backup
  |
  v
Restore
  |
  v
Recovery
```

Backup tanpa restore procedure yang teruji belum memberikan jaminan recovery.

Karena itu restore testing menjadi bagian dari operational workflow.

---

## Current State

| Capability | Status |
|---|---|
| PostgreSQL backup strategy | Defined |
| Logical backup (`pg_dump`) | To be implemented |
| Dedicated backup directory | To be implemented |
| Automated backup | To be implemented |
| Backup retention | To be implemented |
| Restore test | To be implemented |
| Backup monitoring | Future |
| Remote backup storage | Future |
| Disaster recovery procedure | Next stage |

---

## Target State

Setelah implementation selesai:

```text id="y6q4p1"
PostgreSQL
    |
    v
Automated pg_dump
    |
    v
Local Backup Storage
    |
    +--> Retention
    |
    +--> Monitoring
    |
    v
Periodic Restore Test
```

Future development dapat menambahkan remote/off-host backup storage untuk menghindari single-host failure.

---

## Conclusion

Backup and restore merupakan bagian penting dari operational reliability.

Infrastructure tidak dianggap memiliki backup hanya karena terdapat file hasil `pg_dump`.

Backup harus:

1. dibuat secara konsisten,
2. disimpan dengan benar,
3. memiliki retention policy,
4. dapat dipulihkan,
5. dan restore-nya dapat diverifikasi.

Target akhir adalah memiliki backup workflow yang automated, observable, dan dapat digunakan sebagai foundation untuk disaster recovery.
