# Boarding House Management System API - Deployment

REST API untuk sistem manajemen kos (boarding house) dengan containerization menggunakan Docker. API ini mendukung autentikasi JWT, manajemen kamar, dan integrasi pembayaran Midtrans (sandbox).

## Prerequisites

Sebelum memulai, pastikan Anda telah menginstal:

| Software | Minimal Versi | Link Download |
|----------|---------------|---------------|
| Docker Desktop | 4.0+ | [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop) |
| Docker Compose | 2.0+ | (Sudah termasuk Docker Desktop) |
| Git | 2.0+ | [git-scm.com](https://git-scm.com/) |

**Untuk pengguna Windows:** Pastikan WSL2 sudah diaktifkan dan Docker Desktop menggunakan backend WSL2.

## Quick Start

### 1. Clone Repository

```bash
git clone https://github.com/ardentiena/boardinghouse-api-deploy.git
cd boardinghouse-api-deploy