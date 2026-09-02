# Alerting and Incident Response

## Overview

Alerting merupakan bagian dari operational layer pada homelab.

Tujuannya bukan hanya menampilkan kondisi infrastructure melalui dashboard, tetapi juga memberikan notification ketika terjadi kondisi yang membutuhkan perhatian operator.

Saat ini alerting menggunakan Grafana Alerting dengan Telegram sebagai notification channel.

---

## Alerting Architecture

Current alerting pipeline:

```text
Linux Host
    |
    v
Node Exporter
    |
    | Metrics
    v
Prometheus
    |
    | Query
    v
Grafana Alerting
    |
    | Notification
    v
Telegram
```

Alerting berjalan secara asynchronous sehingga operator tidak harus terus membuka Grafana untuk mengetahui adanya masalah.

---

## Current Alert

Alert yang saat ini diimplementasikan:

```text
Name: Application CPU High
Metric: CPU utilization
Threshold: > 80%
Evaluation interval: 1 minute
Pending duration: 5 minutes
```

PromQL:

```promql
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

Alert menggunakan average CPU utilization pada host yang dimonitor.

---

## Alert Lifecycle

Alert memiliki lifecycle:

```text
Normal
  |
  | Condition exceeded
  v
Pending
  |
  | Condition remains true
  v
Firing
  |
  | Notification
  v
Telegram
  |
  | Condition returns to normal
  v
Resolved
```

Pending state digunakan untuk menghindari notification akibat spike CPU yang hanya berlangsung sangat singkat.

---

## Notification

Ketika alert memasuki firing state, Grafana mengirimkan notification ke Telegram.

Operator kemudian dapat melakukan investigation berdasarkan kondisi host dan service yang sedang berjalan.

Ketika kondisi kembali normal, Grafana juga mengirimkan resolved notification.

Dengan demikian operator mendapatkan dua informasi:

```text
Incident detected
        +
Incident recovered
```

---

## Alert Testing

Alert pipeline telah diuji menggunakan controlled CPU load.

Tool yang digunakan:

```text
stress-ng
```

Testing dilakukan dengan sengaja meningkatkan CPU utilization pada host.

Flow pengujian:

```text
stress-ng
    |
    v
CPU utilization increases
    |
    v
Node Exporter
    |
    v
Prometheus
    |
    v
Grafana Alert
    |
    v
Telegram notification
```

Setelah CPU load dihentikan dan utilization kembali normal, recovery notification juga diterima.

---

## Incident Response Workflow

Ketika menerima alert, operator mengikuti workflow sederhana:

```text
Alert received
      |
      v
Identify affected host
      |
      v
Check Grafana dashboard
      |
      v
Determine resource/service impact
      |
      v
Investigate root cause
      |
      v
Apply remediation
      |
      v
Verify recovery
      |
      v
Close incident
```

Untuk tahap awal homelab, workflow ini cukup untuk melatih operational troubleshooting.

---

## Example: High CPU Incident

Contoh incident:

```text
Event:
CPU utilization > 80%

Detection:
Grafana Alerting

Notification:
Telegram

Investigation:
Grafana host dashboard

Possible causes:
- runaway process
- excessive workload
- container resource usage
- unexpected background process
```

Operator kemudian dapat memeriksa host menggunakan Linux tools seperti:

```bash
top
htop
ps aux
docker stats
```

Setelah penyebab ditemukan, remediation dilakukan sesuai sumber masalah.

---

## Recovery Verification

Incident tidak dianggap selesai hanya karena alert berubah menjadi resolved.

Operator tetap perlu melakukan verification.

Contoh verification:

```text
CPU utilization
      |
      v
Back to normal
      |
      v
Container status
      |
      v
Application health
      |
      v
No new alerts
```

Prinsipnya:

> Alert resolved ≠ incident automatically understood.

Recovery harus diverifikasi dari perspective service dan infrastructure.

---

## Alert Severity

Saat ini homelab masih memiliki satu alert utama.

Seiring bertambahnya service, alert dapat dikategorikan berdasarkan severity.

Contoh:

| Severity | Example |
|---|---|
| Critical | Host unavailable |
| High | Application unavailable |
| Warning | CPU > 80% |
| Info | Service restarted |

Severity classification membantu operator menentukan prioritas response.

---

## Future Alerts

Alert yang dapat ditambahkan:

### Host

```text
CPU high
Memory high
Disk usage high
Host unavailable
```

### Application

```text
Application unavailable
High HTTP error rate
High response latency
Container unhealthy
```

### Database

```text
Database unavailable
Too many connections
Disk usage high
Backup failure
```

### Infrastructure

```text
VM unavailable
Monitoring target down
Backup job failed
```

---

## Alert Fatigue

Alerting harus digunakan secara selektif.

Terlalu banyak alert dapat menyebabkan **alert fatigue**, yaitu operator menerima terlalu banyak notification sehingga alert penting kehilangan perhatian.

Karena itu setiap alert sebaiknya memiliki:

- meaningful threshold
- appropriate evaluation window
- clear severity
- actionable response
- notification channel yang sesuai

Tujuan alerting bukan menghasilkan notification sebanyak mungkin, tetapi menghasilkan notification yang dapat ditindaklanjuti.

---

## Current State

| Capability | Status |
|---|---|
| Grafana Alerting | Implemented |
| CPU alert | Implemented |
| Telegram notification | Working |
| Firing notification test | Passed |
| Recovery notification test | Passed |
| Incident workflow | Documented |
| Multiple severity levels | Future |
| Application alerts | Future |
| Database alerts | Future |
| Backup failure alerts | Future |

---

## Conclusion

Alerting layer menghubungkan monitoring dengan operational response.

Architecture saat ini:

```text
Metrics
   |
   v
Detection
   |
   v
Alert
   |
   v
Notification
   |
   v
Investigation
   |
   v
Remediation
   |
   v
Verification
```

Dengan adanya alerting dan incident-response workflow, homelab tidak hanya berfungsi sebagai environment untuk menjalankan service, tetapi juga sebagai environment untuk melatih operational reliability dan troubleshooting.

Tahap berikutnya adalah membangun dan mendokumentasikan **backup dan restore**, sehingga infrastructure memiliki mekanisme recovery terhadap kehilangan data.
