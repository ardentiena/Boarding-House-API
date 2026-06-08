# Boarding House Management System API

> **⚠️ STATUS: PENELITIAN / EKSPERIMEN**
>
> Repository ini adalah bagian dari penelitian matakuliah metode penelitian dengan judul:
> *"Penerapan Containerization untuk Portabilitas Deployment: Studi Kasus REST API Sistem Manajemen Kos"*
>
> Tujuan dari eksperimen ini adalah membuktikan bahwa teknologi containerization (Docker) dapat membuat aplikasi dapat di-deploy secara portabel di berbagai lingkungan (Windows, Linux) tanpa modifikasi kode, hanya dengan mengandalkan environment variable.
>
> **Karakteristik eksperimen:**
> - Image yang digunakan telah di-push ke Docker Hub
> - Konfigurasi dipisahkan melalui file `.env`
> - Deployment memerlukan koneksi internet (untuk pull image)
> - **Tidak perlu build source code** — langsung pull dan run
> - Payment gateway menggunakan **Midtrans Sandbox** (testing, bukan produksi)
---
> **⚠️ STATUS: RESEARCH / EXPERIMENT**
>
> This repository is part of a research methods course project titled:
> *"Implementing Containerization for Deployment Portability: A Case Study of a Boarding House Management System REST API"*
>
> The goal of this experiment is to demonstrate that containerization technology (Docker) can make applications deployable portably across different environments (Windows, Linux) without any code modifications, relying solely on environment variables.
>
> **Experiment characteristics:**
> - Images have been pushed to Docker Hub
> - Configuration is separated via a `.env` file
> - Deployment requires an internet connection (to pull images)
> - **No need to build source code** — just pull and run
> - Payment gateway uses **Midtrans Sandbox** (testing only, not production)
---

## Daftar Isi

- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Pengujian Endpoint](#pengujian-endpoint)
- [Perintah Berguna](#perintah-berguna)
- [Struktur Aplikasi](#struktur-aplikasi)
- [Antisipasi Error](#antisipasi-error)
- [Menghentikan Aplikasi](#menghentikan-aplikasi)
- [Troubleshooting](#troubleshooting)
- [Lisensi](#lisensi)
- [Referensi](#referensi)

---

## Prerequisites

Sebelum memulai, pastikan Anda telah menginstal:

| Software | Minimal Versi | Link Download |
|----------|---------------|---------------|
| Docker Desktop | 4.0+ | [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) |
| Docker Compose | 2.0+ | (Sudah termasuk Docker Desktop) |
| Git | 2.0+ | [git-scm.com](https://git-scm.com/) |

> **Untuk pengguna Windows:** Pastikan WSL2 sudah diaktifkan dan Docker Desktop menggunakan backend WSL2.

---

## Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/ardentiena/boardinghouse-api-deploy.git
cd boardinghouse-api-deploy
```

### 2. Buat File Environment

```bash
cp .env.example .env
```

### 3. Konfigurasi Environment Variable

Edit file `.env` dengan text editor (nano, vim, notepad, dsb):

```bash
nano .env
```

**Variabel yang wajib diisi:**

| Variabel | Keterangan |
|----------|------------|
| `DB_PASSWORD` | Password database PostgreSQL |
| `JWT_SECRET` | String minimal 32 karakter untuk signing JWT |
| `MIDTRANS_SERVER_KEY` | Server key dari dashboard Midtrans Sandbox |
| `MIDTRANS_CLIENT_KEY` | Client key dari dashboard Midtrans Sandbox |
| `ADMIN_PASSWORD` | Password untuk akun admin default |

**Variabel opsional (dapat dibiarkan default):**

| Variabel | Keterangan |
|----------|------------|
| `DB_NAME` | Nama database |
| `DB_USER` | Username database |
| `ADMIN_USERNAME` | Username admin default |
| `NGINX_PORT` | Port yang diekspos keluar (default: `8000`) |

### 4. Jalankan Aplikasi

```bash
docker-compose up -d
```

> Image akan diunduh dari Docker Hub (sekitar 400MB). Tunggu hingga semua container berstatus `up` atau `healthy`.

### 5. Verifikasi Deployment

```bash
# Cek status semua container
docker-compose ps

# Test health endpoint (8000 adalah default, bisa diganti di .env)
curl http://localhost:8000/health
```

**Response yang diharapkan:**

```json
{"status":"UP","message":"Boarding House API is running"}
```

---

## Pengujian Endpoint

### 1. Login untuk Mendapatkan Token JWT

```bash
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"[ADMIN_PASSWORD]"}'
```

> Sesuaikan nilai `username` dan `password` dengan nilai `ADMIN_USERNAME` dan `ADMIN_PASSWORD` di `.env`.

**Response yang diharapkan (ini contoh):**

```json
{"token":"iJIUzI1NiJ9.eyJzdWIiOiJhZG1..."}
```

### 2. Mendapatkan Data Kamar

```bash
# Simpan token ke variabel (ini contoh)
TOKEN="iJIUzI1NiJ9.eyJzdWIiOiJhZG1..."

# Request ke endpoint rooms
curl -X GET http://localhost:8000/api/rooms \
  -H "Authorization: Bearer $TOKEN"
```

**Response yang diharapkan:**

```json
[
  {
    "id": 1,
    "roomNumber": "A101",
    "pricePerMonth": 1500000.00,
    "isAvailable": true,
    "description": "Standard room with AC"
  },
  {
    "id": 2,
    "roomNumber": "A102",
    "pricePerMonth": 1500000.00,
    "isAvailable": true,
    "description": "Standard room with fan"
  },
  {
    "id": 3,
    "roomNumber": "B201",
    "pricePerMonth": 1750000.00,
    "isAvailable": true,
    "description": "Deluxe room with balcony"
  }
]
```

### 3. Membuat Kamar Baru

```bash
curl -X POST http://localhost:8000/api/rooms \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roomNumber": "C301",
    "pricePerMonth": 2000000,
    "description": "Executive suite with balcony"
  }'
```

### 4. Melakukan Pembayaran

```bash
curl -X POST http://localhost:8000/api/payments/charge \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "roomNumber": "A101",
    "amount": 150000,
    "customerName": "John Doe",
    "email": "john@example.com"
  }'
```

---

## Perintah Berguna

| Perintah | Fungsi |
|----------|--------|
| `docker-compose up -d` | Menjalankan semua container di background |
| `docker-compose down` | Menghentikan semua container |
| `docker-compose down -v` | Menghentikan dan menghapus volume (data database ikut hilang) |
| `docker-compose ps` | Melihat status container |
| `docker-compose logs -f` | Melihat log secara realtime |
| `docker-compose logs web-api` | Melihat log dari API saja |
| `docker-compose restart web-api` | Restart API saja |

---

## Struktur Aplikasi

```
┌─────────────────────────────────────────────────────────────┐
│                     Docker Network                           │
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Nginx      │    │  Spring Boot │    │  PostgreSQL  │  │
│  │   Port 8000  │───▶│   API        │───▶│  Database    │  │
│  │  (Reverse    │    │   Port 8080  │    │  Port 5432   │  │
│  │   Proxy)     │    │              │    │              │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Antisipasi Error

### Error 1: Port Already Allocated

**Gejala:**
```
Error response from daemon: Bind for 0.0.0.0:5432 failed: port is already allocated
```

**Penyebab:** Port 5432 (PostgreSQL), 8080 (API), atau 8000 (Nginx) sudah digunakan aplikasi lain.

**Solusi:**
- Hentikan aplikasi yang menggunakan port tersebut, atau
- Ubah port di file `.env` (untuk Nginx) atau di `docker-compose.yml`

---

### Error 2: Database Connection Failed

**Gejala:**
```
org.postgresql.util.PSQLException: Connection refused
```

**Penyebab:** Database belum siap saat API mencoba koneksi.

**Solusi:** Tunggu 10–15 detik, lalu restart API:

```bash
docker-compose restart web-api
```

---

### Error 3: JWT Secret Not Set

**Gejala:**
```
JWT_SECRET is not set in .env
```

**Penyebab:** `JWT_SECRET` tidak diisi di file `.env`.

**Solusi:** Isi `JWT_SECRET` di `.env` dengan string minimal 32 karakter.

---

### Error 4: Container Name Conflict

**Gejala:**
```
Conflict. The container name "/boardinghouse-api-web-api" is already in use
```

**Penyebab:** Container dengan nama yang sama sudah ada (misalnya dari project lama).

**Solusi:** Hapus container lama:

```bash
docker rm -f boardinghouse-api-web-api boardinghouse-api-db-1 boardinghouse-nginx
docker-compose up -d
```

---

### Error 5: Permission Denied (Linux/WSL)

**Gejala:**
```
permission denied while trying to connect to the Docker daemon socket
```

**Penyebab:** User tidak memiliki akses ke Docker.

**Solusi:**

```bash
sudo usermod -aG docker $USER
# Logout dan login kembali
```

---

### Error 6: Disk Space Full

**Gejala:**
```
no space left on device
```

**Solusi:** Bersihkan Docker cache:

```bash
docker system prune -a -f
docker builder prune -a -f
```

---

### Error 7: Image Tidak Ditemukan

**Gejala:**
```
Pull access denied, repository does not exist
```

**Penyebab:** Image di Docker Hub tidak tersedia atau koneksi internet bermasalah.

**Solusi:**

```bash
# Cek koneksi internet
ping google.com

# Cek apakah image tersedia
docker pull ardentiena/boardinghouse-api:latest
docker pull ardentiena/boardinghouse-nginx:latest
```

---

## Menghentikan Aplikasi

Untuk menghentikan semua container:

```bash
docker-compose down
```

Untuk menghentikan dan menghapus semua data (database reset):

```bash
docker-compose down -v
```

---

## Troubleshooting

Jika mengalami masalah, jalankan perintah berikut untuk mendapatkan informasi lebih detail:

```bash
# Lihat semua log
docker-compose logs

# Lihat log spesifik service
docker-compose logs web-api
docker-compose logs nginx
docker-compose logs boardinghouse-api-db

# Cek apakah container berjalan
docker ps -a | grep boardinghouse
```

---

## Keterbatasan Eksperimen (Future Work)
Eksperimen ini memiliki beberapa keterbatasan yang dapat/akan dikembangkan lebih lanjut:
- *Role-Base Access Controll (RBAC)*: belum diimplementasikan
- *Activation key untuk tenant*: belum diimplementasikan
- *Spring Scheduler untuk pembuatan tagihan otomatis*: belum diimplementasikan
- *Swagger UI untuk dokumentasi API*: belum diimplementasikan
- *Midtrans Production Mode*: saat ini hanya menggunakan sandbox

---

## Lisensi

Tugas Metode Penelitian — Universitas Kristen Satya Wacana (UKSW)

---

## Referensi

- [Docker Documentation](https://docs.docker.com/)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Midtrans Documentation](https://docs.midtrans.com/)
- [JWT Documentation](https://www.jwt.io/)