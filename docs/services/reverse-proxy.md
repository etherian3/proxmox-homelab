# Reverse Proxy

## Overview

Traefik digunakan sebagai **reverse proxy** pada `application-01`.

Perannya adalah menjadi entry point untuk service yang berjalan di dalam Docker, sehingga client tidak perlu mengakses setiap container secara langsung berdasarkan port masing-masing.

Arsitektur saat ini:

```text
Client
   |
   | http://app.homelab
   v
Traefik
   |
   | Docker network: proxy
   v
homelab-app
   |
   | Docker network: app-network
   v
PostgreSQL
```

Traefik menjadi layer yang berada di depan application container.

---

## Why Reverse Proxy?

Tanpa reverse proxy, client harus mengakses application secara langsung:

```text
Client
   |
   v
192.168.1.111:8000
   |
   v
homelab-app
```

Dengan Traefik:

```text
Client
   |
   v
app.homelab
   |
   v
Traefik
   |
   v
homelab-app:8000
```

Pendekatan ini memberikan beberapa keuntungan:

- hostname yang lebih mudah digunakan
- centralized routing
- application container tidak perlu menjadi entry point utama
- lebih mudah menambahkan service baru
- dapat menjadi lokasi terpusat untuk middleware dan security policy
- mempersiapkan infrastructure untuk TLS dan authentication

---

## Current Deployment

Traefik berjalan pada:

```text
Host: application-01
IP: 192.168.1.111
Runtime: Docker
```

Application berjalan sebagai container terpisah:

```text
homelab-app
Port: 8000
```

Traefik kemudian meneruskan request menuju application container.

---

## Docker Network

Traefik dan application menggunakan Docker network:

```text
proxy
```

Network ini digunakan sebagai jalur komunikasi antara reverse proxy dan service yang ingin diekspos melalui Traefik.

Application juga memiliki network internal:

```text
app-network
```

Sehingga terdapat pemisahan antara:

```text
proxy
  |
  +-- Traefik
  |
  +-- homelab-app


app-network
  |
  +-- homelab-app
  |
  +-- PostgreSQL
```

Application menjadi penghubung antara reverse-proxy layer dan database layer.

PostgreSQL tidak perlu berada pada network `proxy` karena database tidak perlu menerima request langsung dari Traefik.

---

## Request Flow

Ketika user membuka:

```text
http://app.homelab
```

request mengikuti flow:

```text
Browser
   |
   v
app.homelab
   |
   v
192.168.1.111
   |
   v
Traefik
   |
   v
homelab-app:8000
   |
   v
Application
```

Application kemudian berkomunikasi dengan PostgreSQL melalui:

```text
app-network
```

Dengan demikian database tetap berada di belakang application layer.

---

## Hostname Resolution

Saat ini `app.homelab` masih menggunakan local hosts configuration pada client.

Contohnya:

```text
192.168.1.111 app.homelab
```

Ini berarti DNS internal belum diimplementasikan.

Flow resolution saat ini:

```text
Browser
   |
   v
/etc/hosts
   |
   v
192.168.1.111
   |
   v
Traefik
```

Pendekatan ini cukup untuk tahap development dan homelab awal.

Namun untuk environment yang lebih mature, internal DNS akan digunakan sehingga hostname tidak perlu dikonfigurasi secara manual pada setiap client.

---

## Service Isolation

Salah satu tujuan penggunaan reverse proxy adalah menjaga agar application service tidak perlu menjadi public entry point.

Application tetap berjalan pada:

```text
homelab-app:8000
```

sedangkan client berinteraksi melalui:

```text
app.homelab
```

Secara konseptual:

```text
External/Client Layer
        |
        v
    Traefik
        |
        v
 Application Layer
        |
        v
   Database Layer
```

Hal ini membuat architecture lebih mudah dikembangkan ketika service bertambah.

Contohnya:

```text
                    +--> homelab-app
                    |
Client --> Traefik -+--> Grafana
                    |
                    +--> Other Services
```

---

## Configuration Management

Deployment dan konfigurasi infrastructure dikelola menggunakan Ansible.

Ansible bertanggung jawab terhadap konfigurasi host dan deployment service, sedangkan Docker menjalankan container.

Pembagian responsibility:

```text
Terraform
   |
   +--> VM provisioning


Ansible
   |
   +--> OS configuration
   +--> Docker installation
   +--> Application deployment
   +--> Monitoring deployment


Docker
   |
   +--> Container runtime


Traefik
   |
   +--> HTTP routing
```

Dengan pembagian tersebut, setiap layer memiliki responsibility yang jelas.

---

## Operational Validation

Reverse proxy telah diuji dengan application yang berjalan pada `application-01`.

Validation utama:

```text
http://app.homelab
```

berhasil diarahkan oleh Traefik menuju application container.

Hal ini membuktikan bahwa:

- hostname dapat diarahkan ke host
- Traefik menerima request
- routing menuju application berjalan
- application container dapat diakses melalui reverse proxy
- Docker network `proxy` berfungsi sebagai communication layer

---

## Current State

Status reverse proxy saat ini:

| Component | Status |
|---|---|
| Traefik | Working |
| Docker integration | Working |
| `proxy` network | Working |
| Application routing | Working |
| `app.homelab` | Working |
| HTTPS/TLS | Not implemented |
| Internal DNS | Not implemented |

---

## Future Improvements

Beberapa improvement yang direncanakan:

### TLS

Implement HTTPS untuk service yang diekspos melalui Traefik.

Target architecture:

```text
Client
   |
   | HTTPS
   v
Traefik
   |
   v
Application
```

### Internal DNS

Menggantikan konfigurasi `/etc/hosts` dengan centralized internal DNS.

```text
Client
   |
   v
Internal DNS
   |
   v
app.homelab
   |
   v
Traefik
```

### Middleware

Menambahkan middleware untuk kebutuhan seperti:

- authentication
- security headers
- rate limiting
- request control

### Additional Services

Traefik nantinya dapat menjadi entry point untuk service lain:

```text
Traefik
├── app.homelab
├── grafana.homelab
├── prometheus.homelab
└── other services
```

---

## Conclusion

Traefik menyediakan centralized reverse-proxy layer untuk homelab.

Architecture saat ini memisahkan:

```text
Client
   |
   v
Reverse Proxy
   |
   v
Application
   |
   v
Database
```

Pemisahan tersebut membuat infrastructure lebih scalable dan memberikan pengalaman yang lebih mendekati architecture production dibandingkan mengakses setiap container secara langsung melalui port host.

Reverse proxy juga menjadi foundation untuk implementasi berikutnya seperti TLS, internal DNS, authentication, dan centralized service routing.
