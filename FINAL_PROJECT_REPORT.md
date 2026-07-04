# Final Project Learning Management System (LMS)

## Identitas Mahasiswa

- **Nama:** Fabian Adam Maheswara
- **NIM:** A11.2023.15336
- **Kelas:** A11.4618
- **Repository:** https://github.com/Fabianadam21/final_project_pss.git

---

## Deskripsi Proyek

Project yang saya kembangkan merupakan sebuah Learning Management System (LMS) berbasis web menggunakan Django REST Framework. Sistem ini dirancang agar proses pembelajaran dapat dilakukan secara terstruktur melalui pengelolaan course, materi, progress belajar, hingga penerbitan sertifikat.

Selain menyediakan fitur utama LMS, project ini juga menerapkan beberapa teknologi pendukung seperti Docker, PostgreSQL, Redis, MongoDB, RabbitMQ, dan Celery. Integrasi teknologi tersebut bertujuan meningkatkan performa aplikasi, terutama dalam menangani proses yang membutuhkan waktu lama agar tidak menghambat pengguna ketika mengakses sistem.

Dengan adanya mekanisme asynchronous processing, aplikasi tetap responsif walaupun sedang menjalankan proses berat seperti pembuatan sertifikat ataupun pengiriman email notifikasi.

---

## Fitur Dasar yang Diimplementasikan

- Dijalankan dengan Docker Compose
- Database PostgreSQL dengan migration sukses
- Authentication JWT
- Role-based access (Admin, Instructor, Student)
- Endpoint untuk course, lesson, enrollment, dan progress tracking
- Dokumentasi README dengan akun demo dan endpoint utama
- Swagger/OpenAPI documentation
- Struktur project rapi, konfigurasi sensitif tidak di-hardcode

---

## Fitur Tambahan yang Diimplementasikan

**Async Processing & Notification** (Total: 65 poin)

| No | Fitur | Implementasi | Poin | Status |
|----|-------|--------------|------|--------|
| 1 | Email Notification Async | Pengiriman email asynchronous melalui Celery | 12 | Selesai |
| 2 | Generate Certificate/Report Async | Pembuatan sertifikat PDF dan laporan CSV sebagai background task | 18 | Selesai |
| 3 | Scheduled Task (Celery Beat) | Task periodik untuk update statistik dan laporan harian | 15 | Selesai |
| 4 | Task Status Endpoint | Endpoint untuk monitoring status task asynchronous | 12 | Selesai |
| 5 | Flower Monitoring | Real-time monitoring Celery worker dan task | 8 | Selesai |

---

## Penjelasan Implementasi

Proyek ini menerapkan arsitektur asynchronous untuk memindahkan proses berat ke background. Hal ini menjaga responsivitas API saat pengguna melakukan aktivitas yang memerlukan waktu lebih lama.

### 1. Email Notification Async

Pada endpoint `/api/enrollments-async`:
- Enrollment langsung disimpan ke database.
- Task Celery `send_enrollment_email` dipicu untuk mengirim email di background.
- Response API kembali cepat tanpa menunggu proses email selesai.

### 2. Generate Certificate/Report Async

Proses pembuatan file dipisahkan dari request utama:
- `/api/courses/{id}/complete-async` memicu task `generate_certificate`.
- `/api/courses/{id}/export-async` memicu task `export_course_report`.
- Sertifikat PDF dibuat dengan ReportLab dan disimpan di `media/certificates/`.
- Laporan CSV dibuat dengan modul `csv` dan disimpan di `media/reports/`.
- Volume `./media:/app/media` ditambahkan pada service Celery agar file tersedia di host.
- Task menggunakan `settings.MEDIA_ROOT` untuk memastikan path file konsisten.

### 3. Scheduled Task - Celery Beat

Saya menambahkan scheduler periodik di `lms/celery.py`:
- `update_course_statistics` dijalankan setiap jam.
- `send_daily_report` dijalankan setiap hari pukul 00:00.
- Ini memastikan statistik dan laporan harian selalu terbarui.

### 4. Task Status Endpoint

Endpoint `/api/tasks/{task_id}` memungkinkan pengguna untuk memonitor status task asynchronous. Informasi yang tersedia:
- **Status**: PENDING, STARTED, SUCCESS, atau FAILURE
- **Pesan status**: Deskripsi kondisi task
- **Hasil task**: Data output jika sudah selesai
- **Waktu penyelesaian**: Kapan task selesai dijalankan

Implementasi menggunakan `AsyncResult` dari Celery. Sangat berguna untuk memonitor task berdurasi lama.

### 5. Flower Monitoring

Flower dipasang sebagai service terpisah di Docker Compose.
- Akses: `http://localhost:5555`
- Menampilkan status worker, status task, dan antrean RabbitMQ.
- Membantu mendeteksi task gagal dan memonitor kinerja worker.

## Integrasi Sistem

Alur kerja sistem:
- Django API menerima request pengguna.
- Task asynchronous dikirim ke RabbitMQ.
- Worker Celery memproses task di background.
- Hasil disimpan ke database atau folder media.
- Flower memberikan monitoring runtime.

Arsitektur ini memastikan aplikasi tetap responsif meskipun sedang memproses tugas-tugas berat.

### Kesimpulan

Semua fitur di atas bekerja secara terintegrasi untuk menciptakan sistem asynchronous processing yang handal. Celery bertindak sebagai task queue yang menangani eksekusi tugas di latar belakang, RabbitMQ berfungsi sebagai message broker yang mengantarkan tugas dari Django ke Celery Worker, dan Flower menyediakan antarmuka monitoring untuk memudahkan pengawasan. Seluruh sistem ini dikemas dalam container Docker sehingga mudah dijalankan dan diskalakan. Pendekatan asynchronous ini memastikan bahwa aplikasi tetap responsif meskipun sedang memproses tugas-tugas berat, sehingga memberikan pengalaman pengguna yang lebih baik.

---

## Cara Menjalankan Project

### 1. Clone Repository

```bash
git clone https://github.com/Fabianadam21/final_project_pss.git
```

### 2. Jalankan Docker Compose

Pertama kali (build image baru):
```bash
docker compose up -d --build
```

Jika sudah ada:
```bash
docker compose up -d
```
![alt text](image-1.png)

### 3. Pastikan Container Berjalan

```bash
docker ps
```

![alt text](image-2.png)

### 4. Buat migration dahulu

```bash
docker compose exec app python manage.py makemigrations
```

![alt text](image-3.png)

### 5. Jalankan migration

```bash
docker compose exec app python manage.py migrate
```

![alt text](image-4.png)

### 6. Seed Data

```bash
docker compose exec app python manage.py seed_data
```

![alt text](image-5.png)

### 7. Menghentikan Project

```bash
docker compose stop
```

![alt text](image-6.png)

---

### Admin Panel

```text
http://localhost:8000/admin/
```

![alt text](image-7.png)

### Django Silk (Query Profiling)

```text
http://localhost:8000/api/docs
```

![alt text](image-9.png)

Flower :

```text
http://localhost:5555/silk/
```

![alt text](image-8.png)

### Swagger/OpenAPI

```text
http://localhost:8000/api/docs
```

![alt text](image-9.png)

### RabbitMQ Management(Celery Monitoring)Q :

```text
http://localhost:15672/
```

![alt text](image-11.png)

---

## Akun Demo

---

## Endpoint Penting

```http
AUTHENTICATION
  [Login]       POST   /api/auth/login

ASYNC TASKS (Celery)
  [Enroll Async]         POST   /api/enrollments-async
  [Complete Course]      POST   /api/courses/{id}/complete-async
  [Export Report]        POST   /api/courses/{id}/export-async
  [Update Stats]         POST   /api/admin/update-stats
  [Task Status]          GET    /api/tasks/{task_id}

MONITORING
  [Flower]               GET    http://localhost:5555
  [RabbitMQ]             GET    http://localhost:15672

```

---

## Bukti Pengujian

### Login API

![alt text](image-12.png)
![alt text](image-13.png)
![alt text](image-14.png)

### Email Notification Async

![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-17.png)
![alt text](image-18.png)

### Generate Certificate/Report Async

![alt text](image-20.png)
![alt text](image-19.png)
![alt text](image-21.png)
![alt text](image-22.png)

![alt text](image-23.png)
![alt text](image-24.png)
![alt text](image-25.png)

### Scheduled Task

![alt text](image-26.png)
![alt text](image-27.png)
![alt text](image-28.png)

### Task Status Endpoint

![alt text](image-29.png)

### Flower Monitoring

![alt text](image-30.png)

### RabbitMQ Management

![alt text](image-31.png)

## Kendala dan Solusi yang Dihadapi

Selama pengerjaan, saya menghadapi tiga masalah utama yang saya selesaikan secara bertahap. Semua solusi berikut dicoba dan diverifikasi secara langsung di dalam lingkungan Docker.

### Kendala 1: File media tidak tersedia di host

- Masalah: Celery worker membuat file sertifikat dan laporan, tetapi hasilnya hanya tersimpan di dalam container.
- Analisis: Service Celery tidak memiliki shared volume `media` yang sama dengan web app.
- Solusi: Tambahkan volume mount `./media:/app/media` di service Celery pada `docker-compose.yml`.
- Hasil: File PDF dan CSV sekarang dapat diakses langsung dari host, sehingga data output tidak hilang saat container direstart.

### Kendala 2: Path MEDIA_ROOT tidak konsisten

- Masalah: Task asynchronous mengakses `MEDIA_ROOT` secara langsung dan menghasilkan error nama variabel tidak ditemukan.
- Analisis: Celery worker tidak mewarisi variabel lingkungan dari Django settings secara otomatis.
- Solusi: Perbaiki task agar menggunakan `settings.MEDIA_ROOT` dan import `from django.conf import settings`.
- Hasil: Pembuatan file menjadi stabil, dan folder output selalu mengarah ke lokasi media yang benar.

### Kendala 3: Tidak ada pemantauan task async oleh client

- Masalah: Endpoint async hanya mengembalikan pesan sukses tanpa `task_id`.
- Analisis: Tanpa ID task, pengguna tidak bisa memeriksa status pekerjaan background.
- Solusi: Simpan objek task dari `task.delay()` dan kembalikan `task.id` dalam response.
- Hasil: Endpoint `/api/tasks/{task_id}` bisa digunakan untuk tracking progress, menampilkan status, dan menangani error.

## Kesimpulan

Melalui project ini saya memperoleh pengalaman dalam membangun aplikasi berbasis Django yang menerapkan konsep asynchronous processing. Integrasi Celery, RabbitMQ, Redis, MongoDB, PostgreSQL, dan Docker menunjukkan bagaimana beberapa layanan dapat bekerja bersama untuk menghasilkan sistem yang lebih efisien dan mudah dikembangkan.

Fitur seperti pengiriman email, pembuatan sertifikat, ekspor laporan, penjadwalan task, serta monitoring menggunakan Flower berhasil diimplementasikan dan berjalan sesuai kebutuhan. Pengalaman ini juga memberikan pemahaman yang lebih baik mengenai penerapan arsitektur backend modern, khususnya pada aplikasi berskala menengah hingga besar.
