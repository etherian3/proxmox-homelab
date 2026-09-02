# Monitoring

## Overview

Monitoring digunakan untuk memberikan visibility terhadap kondisi infrastructure dan application host.

Monitoring stack saat ini berjalan pada `application-01` menggunakan Docker.

Komponen utama:

```text id="8l3m4k"
Node Exporter
      |
      | metrics
      v
 Prometheus
      |
      | PromQL
      v
  Grafana
      |
      | Alerting
      v
  Telegram
```

Monitoring berfokus pada infrastructure metrics terlebih dahulu, khususnya resource utilization pada `application-01`.

---

## Monitoring Components

### Node Exporter

Node Exporter digunakan untuk mengumpulkan system-level metrics dari Linux host.

Metrics yang tersedia antara lain:

- CPU utilization
- memory utilization
- filesystem utilization
- network traffic
- host availability

Node Exporter berjalan sebagai Docker container pada `application-01`.

---

### Prometheus

Prometheus digunakan sebagai metrics collection dan time-series database.

Prometheus melakukan scraping terhadap Node Exporter:

```text id="e8p4hj"
application-01
     |
     v
Node Exporter
     |
     | metrics
     v
Prometheus
```

Prometheus kemudian menyediakan data tersebut untuk query dan alerting.

---

### Grafana

Grafana digunakan sebagai visualization dan alerting layer.

Grafana mengambil metrics dari Prometheus menggunakan Prometheus sebagai data source.

```text id="7a6g0r"
Prometheus
     |
     | PromQL
     v
Grafana
     |
     +--> Dashboard
     |
     +--> Alerting
```

Grafana dashboard digunakan untuk melihat kondisi host secara real-time maupun berdasarkan historical metrics.

---

## Host Dashboard

Dashboard monitoring yang dibuat mencakup beberapa metric utama.

### CPU Usage

CPU utilization dihitung menggunakan:

```promql id="cvq3ab"
100 - (avg by(instance) (
  rate(node_cpu_seconds_total{mode="idle"}[5m])
) * 100)
```

Query tersebut menghitung persentase CPU yang sedang digunakan berdasarkan idle CPU time.

---

### Memory Usage

Memory utilization:

```promql id="8u7y0c"
100 * (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
```

Metric ini digunakan untuk melihat persentase memory yang sedang digunakan.

---

### Disk Usage

Filesystem utilization:

```promql id="yv9g7x"
100 * (
  1 -
  node_filesystem_avail_bytes{mountpoint="/",fstype!~"tmpfs|overlay"}
  /
  node_filesystem_size_bytes{mountpoint="/",fstype!~"tmpfs|overlay"}
)
```

Metric difokuskan pada filesystem utama host.

---

### Network Receive

Incoming network traffic:

```promql id="l8v0mc"
rate(node_network_receive_bytes_total{device!="lo"}[5m])
```

---

### Network Transmit

Outgoing network traffic:

```promql id="m7b2n4"
rate(node_network_transmit_bytes_total{device!="lo"}[5m])
```

---

### Host Availability

Host availability dipantau menggunakan:

```promql id="4x7m1p"
up{job="node"}
```

Nilai:

```text id="5apqz4"
1 = target reachable
0 = target unavailable
```

---

## Alerting

Monitoring tidak hanya digunakan untuk visualization.

Grafana Alerting digunakan untuk mendeteksi kondisi abnormal dan mengirimkan notification.

Alert yang saat ini dibuat:

```text id="k3xv5h"
Name: Application CPU High
Condition: CPU > 80%
Evaluation: every 1 minute
Pending duration: 5 minutes
```

Query alert:

```promql id="g6r4pk"
100 - (
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  ) * 100
)
```

Alert akan masuk ke firing state ketika CPU utilization melewati threshold yang telah ditentukan selama periode evaluasi.

---

## Notification Pipeline

Notification menggunakan Telegram sebagai notification channel.

Architecture:

```text id="w6j8zq"
Node Exporter
      |
      v
Prometheus
      |
      v
Grafana Alerting
      |
      v
Telegram
```

Dengan demikian administrator tidak perlu terus-menerus membuka dashboard untuk mengetahui adanya resource issue.

---

## Alert Testing

Alert tidak hanya dikonfigurasi tetapi juga diuji menggunakan controlled CPU load.

Testing workflow:

```text id="q7d4wm"
stress-ng
   |
   | CPU load
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
Telegram
```

CPU load dibuat secara sengaja menggunakan `stress-ng`.

Setelah CPU melewati threshold, alert masuk ke firing state dan notification dikirim melalui Telegram.

Ketika CPU kembali normal, alert kemudian masuk ke resolved state dan recovery notification diterima.

Testing tersebut membuktikan end-to-end alerting pipeline.

---

## Monitoring Deployment

Monitoring stack dikelola menggunakan Ansible.

Role monitoring berada pada:

```text id="e2w1d9"
homelab-ansible/
└── roles/
    └── monitoring/
```

Deployment menggunakan Docker sehingga monitoring components memiliki lifecycle yang terpisah dari host operating system.

Konsep deployment:

```text id="m9l4q1"
Ansible
   |
   v
Docker
   |
   +--> Prometheus
   +--> Grafana
   +--> Node Exporter
```

---

## Operational Workflow

Monitoring digunakan sebagai bagian dari operational workflow:

```text id="w0g5tj"
Infrastructure
      |
      v
Metrics Collection
      |
      v
Visualization
      |
      v
Alert Detection
      |
      v
Notification
      |
      v
Operator Response
```

Hal ini mengubah monitoring dari sekadar dashboard menjadi bagian dari operational feedback loop.

---

## Current State

| Component | Status |
|---|---|
| Node Exporter | Working |
| Prometheus | Working |
| Grafana | Working |
| Prometheus data source | Working |
| Host dashboard | Implemented |
| CPU monitoring | Implemented |
| Memory monitoring | Implemented |
| Disk monitoring | Implemented |
| Network monitoring | Implemented |
| Host availability | Implemented |
| CPU alert | Implemented |
| Telegram notification | Working |
| Alert firing test | Passed |
| Alert recovery test | Passed |
| Application-specific metrics | Future |

---

## Current Monitoring Scope

Monitoring saat ini terutama berfokus pada host infrastructure.

```text id="h7d9c3"
application-01
├── CPU
├── Memory
├── Disk
├── Network
└── Availability
```

Application-level observability belum menjadi fokus utama pada tahap ini.

Pengembangan berikutnya dapat menambahkan:

- application metrics
- HTTP request metrics
- response latency
- error rate
- container health
- PostgreSQL metrics
- service-specific alerts

---

## Future Improvements

### Application Metrics

Mengumpulkan metrics dari `homelab-app` secara langsung.

### Container Monitoring

Menambahkan visibility terhadap resource utilization setiap container.

### PostgreSQL Monitoring

Menambahkan database-specific metrics.

### Additional Alerts

Contoh alert yang dapat ditambahkan:

```text id="9b4h1w"
High CPU
High Memory
Low Disk Space
Host Down
Application Down
High HTTP Error Rate
High Request Latency
Database Connection Issues
```

### Centralized Observability

Dalam tahap berikutnya monitoring dapat dikembangkan menjadi centralized observability stack untuk seluruh infrastructure.

---

## Conclusion

Monitoring layer menyediakan visibility dan alerting untuk infrastructure homelab.

Stack saat ini:

```text id="b9q3fk"
Node Exporter
      |
      v
 Prometheus
      |
      v
  Grafana
      |
      v
  Telegram
```

Dashboard telah dibuat untuk CPU, memory, disk, network, dan host availability.

CPU alert juga telah diuji menggunakan controlled load dan terbukti dapat mengirimkan notification ketika threshold terlampaui serta notification ketika kondisi kembali normal.

Monitoring layer saat ini menjadi dasar untuk operational visibility dan dapat dikembangkan menuju application-level observability.
