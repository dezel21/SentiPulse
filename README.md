# <p align="center"> 📊 SentiPulse (`senti-pulse`)</p>

<p align="center">
  <img src="https://img.shields.io/badge/JHipster-v9.1.0-F58120?style=for-the-badge&logo=jhipster&logoColor=white" alt="JHipster">
  <img src="https://img.shields.io/badge/Spring_Boot-3.5.14-6DB33F?style=for-the-badge&logo=spring-boot&logoColor=white" alt="Spring Boot">
  <img src="https://img.shields.io/badge/Angular-21.2.14-DD0031?style=for-the-badge&logo=angular&logoColor=white" alt="Angular">
  <img src="https://img.shields.io/badge/MySQL-8%2B-4479A1?style=for-the-badge&logo=mysql&logoColor=white" alt="MySQL">
</p>

## 📝 Gambaran Umum
**SentiPulse** adalah platform analisis sentimen pasar saham dan aset kripto berbasis *full-stack monolith reaktif*. Proyek ini dirancang khusus untuk memenuhi standar tugas akhir kelompok mata kuliah **Keamanan Jaringan (Semester 4), Program Studi S1 Sistem Informasi, Universitas Pembangunan Nasional "Veteran" Jakarta**. 

Fokus utama proyek ini adalah menerapkan cetak biru keamanan hibrida pada lapisan atas model OSI (Layer 5, 6, dan 7) menggunakan kombinasi teknologi tangguh dari **Apache Software Foundation** untuk melindungi data finansial yang sensitif.

---

## 🔒 Fitur Utama Keamanan (Layer 5-6-7 OSI)

### 🔑 Layer 5 — Session Layer
* **Centralized Identity Store:** Autentikasi disinkronisasikan secara terpusat melalui direktori LDAP berbasis **Apache Fortress (OpenLDAP)** pada port 389.
* **Stateless JWT Session:** Manajemen sesi menggunakan token JWT (HS512) dengan validitas standar 24 jam dan ekstensi *Remember Me* hingga 30 hari.

### 🛡️ Layer 6 — Presentation Layer
* **TLS Termination:** Mengubah transmisi data *plaintext* menjadi *ciphertext* terenkripsi aman melalui HTTPS pada port 9443 menggunakan **Apache APISIX**.
* **Data Integrity:** Validasi format objek data JSON di level *gateway* sebelum dieksekusi oleh *backend*.

### 🚀 Layer 7 — Application Layer
* **API Gateway & Rate Limiting:** Pembatasan kuota pemanggilan rute via plugin `limit-req` di **Apache APISIX** untuk menangkal *bot scraper* dan serangan *brute force*[cite: 1].
* **Distributed Tracing:** Pengawasan jejak transaksi API non-blocking secara visual menggunakan **Apache SkyWalking**.
* **Availability Monitoring:** Sistem deteksi *health-check* infrastruktur otomatis setiap 60 detik menggunakan **Apache HertzBeat**.

---

## 💻 Arsitektur & Tech Stack

### Backend & Database
* **Core Framework:** Spring Boot 3.5.14 & Spring WebFlux (Reactive Stack)
* **Reactive Stream:** Project Reactor & Micrometer Context Propagation
* **Data Access:** Spring Data R2DBC (Non-blocking MySQL driver)
* **Migration:** Liquibase Database Schema Management

### Frontend UI
* Angular 21.2.14, Bootstrap 5.3.8, & RxJS 7.8.2

### Perangkat Jaringan & Observabilitas
* Apache APISIX v3.10.0, Apache Fortress (OpenLDAP v1.5.0), Apache SkyWalking v9.5.0, Apache HertzBeat v1.6.0, dan etcd v3.5.0.

---

## 🗺️ Arsitektur Jaringan Zero-Trust

```text
       [ CLIENT / USER BROWSER ] 
                   │
                   ▼ (Koneksi Terenkripsi Port 9443 / HTTPS)  <-- [LAYER 6: TLS Termination]
   ┌─────────────────────────────────────────────────────────┐
   │                  APACHE APISIX (Gateway)                │
   │  - Validasi Sesi & JWT Token     <-- [LAYER 5]          │
   │  - Plugin Rate Limiting (Anti-BotScraper) <-- [LAYER 7] │
   └───────────┬─────────────────────────────────────────────┘
               │
               ├───────► [ APACHE FORTRESS ] (Verifikasi Otorisasi Direktori LDAP) <-- [LAYER 5/7]
               │
               ▼ (Jaringan Jembatan Internal / Docker Bridge Network)
   ┌─────────────────────────────────────────────────────────┐
   │                    ZONA LAYANAN JHIPSTER                │
   │  ├── sentiment-service (Port 8081) ◄── [SkyWalking Agent]│
   │  └── analyst-service   (Port 8082) ◄── [SkyWalking Agent]│
   └───────────────────────────┬─────────────────────────────┘
                               │
                               ├───────► [ APACHE SKYWALKING OAP ] (Distributed Tracing Log)
                               │
                               └───────► [ APACHE HERTZBEAT ] (Health Check & Alerting)
```
---

## 🚀 Panduan Instalasi & Simulasi

### 🛠️ Prasyarat Perangkat
* Docker & Docker Compose aktif
* Java 21/25 & Node.js minimum v24

### 1. Kloning Repositori
```bash
git clone [https://github.com/username-kamu/sentimen-pasar-jhipster.git](https://github.com/username-kamu/sentimen-pasar-jhipster.git)
cd sentimen-pasar-jhipster
```
2. Nyalakan Kontainer Ekosistem Keamanan via Docker
```Bash
docker compose up -d
```
3. Periksa Status Kontainer
```Bash
docker compose ps
```
Pastikan seluruh layanan (APISIX, Fortress, SkyWalking, HertzBeat, JHipster application) berstatus Up / Running.
---

## 🧪 Skenario Uji (Bahan Laporan Bab 5)
Jalankan perintah pengujian keamanan berikut melalui Terminal/Postman untuk memverifikasi efektivitas benteng jaringan:

A. Uji Coba Layer 5 — Sesi Ditolak (Unauthorized)
Mencoba menembak API sentimen tanpa melampirkan JWT token:

```Bash
curl -i http://localhost:9080/api/authenticate
```
Ekspektasi Respon: HTTP/1.1 401 Unauthorized dengan header Server: APISIX/3.10.0.
B. Uji Coba Layer 7 — Broken Access Control (Forbidden)
Mencoba mengakses fitur administratif internal menggunakan peran level rendah:

```Bash
curl -i -X POST http://localhost:9080/api/admin/users -H "Role: ROLE_USER"
```
Ekspektasi Respon: HTTP/1.1 403 Forbidden diblokir oleh kebijakan hak akses.

C. Uji Coba Monitoring Dashboard
Apache SkyWalking (Tracing UI): 
Buka ```http://localhost:8081``` untuk melihat peta topologi transaksi data dan audit log trace
Apache HertzBeat (Availability UI): 
Buka ```http://localhost:1157``` untuk memantau status kesehatan server secara real-time
