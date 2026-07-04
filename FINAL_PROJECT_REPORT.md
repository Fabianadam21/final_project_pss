# Final Project Learning Management System (LMS)

## Identitas Mahasiswa

- **Nama:** Fabian Adam Maheswara
- **NIM:** A11.2023.15336
- **Kelas:** A11.4618
- **Repository:** https://github.com/Fabianadam21/final_project_pss.git

---

## Deskripsi Proyek

Simple LMS adalah sistem manajemen pembelajaran berbasis web yang dibangun dengan Django REST Framework dan teknologi modern seperti:
- **Celery** untuk background task processing
- **Redis** untuk caching
- **MongoDB** untuk activity logging
- **RabbitMQ** sebagai message broker

Sistem mendukung tiga peran pengguna (Admin, Instructor, Student) untuk mengelola course, enrollment, tracking progress, dan generate sertifikat serta laporan secara otomatis. Semua proses berat berjalan secara asynchronous tanpa mengganggu pengalaman pengguna.

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

Proyek ini mengimplementasikan arsitektur asynchronous processing menggunakan Celery dan RabbitMQ untuk menangani tugas-tugas berat di background. Tujuannya adalah meningkatkan responsivitas aplikasi agar pengguna tidak menunggu saat menjalankan proses seperti pengiriman email, pembuatan sertifikat, dan ekspor laporan.

### 1. Email Notification Async

Fitur ini mengirimkan notifikasi email secara asynchronous saat pengguna mendaftar ke course. Ketika pengguna mengakses `/api/enrollments-async`:
1. Sistem mencatat enrollment dan segera mengembalikan response
2. T2. Generate Certificate/Report Async

Fitur ini menangani pembuatan sertifikat PDF dan laporan CSV secara asynchronous:

**Generate Certificate:**
- Pengguna menyelesaikan course via `/api/courses/{id}/complete-async`
- Celery Worker memproses pembuatan PDF di background menggunakan ReportLab
- File PDF disimpan di `media/certificates/`

**Export Report:**
- Ad3. Scheduled Task - Celery Beat

Fitur ini menjalankan task secara periodik menggunakan Celery Beat:

**Task yang Dijadwalkan:**
1. `update_course_statistics` - Setiap jam: menghitung jumlah member dan update statistik course
2. `send_daily_report` - Tengah malam (00:00): mengirim laporan harian ke admin via email

**Konfigurasi (celery.py):**
- Schedule per jam: `crontab(minute=0, hour='*/1')`
- Schedule harian: `crontab(hour=0, minute=0)`

Task berjalan otomatis tanpa intervensi
- Implementasi menggunakan task `generate_certificate` dan `export_course_report`

Proses berat tidak menghambat pengguna lain mengakses aplikasi

Impl4. Task Status Endpoint

Endpoint `/api/tasks/{task_id}` memungkinkan pengguna untuk memonitor status task asynchronous. Informasi yang tersedia:
- **Status**: PENDING, STARTED, SUCCESS, atau FAILURE
- **Pesan status**: Deskripsi kondisi task
- **Hasil task**: Data output jika sudah selesai
- **Waktu penyelesaian**: Kapan task selesai dijalankan

Implementasi menggunakan `AsyncResult` dari Celery. Sangat berguna untuk memonitor task berdurasi lama

Fitur ini menangani pembuatan sertifikat PDF dan laporan CSV secara asynchronous. Pada proses generate certificate, pengguna menyelesaikan course melalui endpoint /api/courses/{id}/complete-async, kemudian Celery Worker langsung memproses pembuatan sertifikat PDF di latar belakang. File PDF yang dihasilkan disimpan di folder media/certificates/. Untuk ekspor laporan, admin atau teacher dapat mengakses endpoint /api/courses/{id}/export-async untuk menghasilkan laporan CSV yang juga diproses secara asynchronous dan disimpan di folder media/reports/. Implementasi ini menggunakan task generate_certificate dengan library reportlab untuk generate PDF, serta task export_course_report dengan library csv untuk generate laporan. Kedua task ini memastikan bahwa proses berat tidak menghambat pengguna lain.

### SCHEDULED TASK - CELERY BEAT

Fitu5. Flower Monitoring

Flower menyediakan dashboard real-time untuk memonitor Celery. Dapat diakses di `http://localhost:5555` dan menampilkan:
- **Status Worker**: Daftar worker yang sedang online
- **Daftar Task**: Task yang telah dijalankan beserta statusnya (SUCCESS, FAILURE, PENDING)
- **Informasi Broker**: Koneksi RabbitMQ dan message queue

Flower berjalan sebagai container terpisah dan terhubung langsung dengan Celery Worker dan RabbitMQ. Memudahkan deteksi kegagalan pada task Celery

Fitur ini memungkinkan pengguna untuk mengecek status tugas asynchronous yang sedang berjalan. Pengguna dapat mengakses endpoint /api/tasks/{task_id} untuk melihat informasi lengkap tentang suatu task, termasuk status saat ini (PENDING, STARTED, SUCCESS, atau FAILURE), pesan status yang informatif, hasil tugas jika sudah selesai, dan waktu penyelesaian jika task telah sukses. Implementasi ini menggunakan AsyncResult dari Celery untuk mengambil informasi task berdasarkan task_id yang diberikan. Endpoint ini sangat berguna bagi pengguna untuk memantau progres tugas yang memakan waktu lama, seperti pembuatan sertifikat atau ekspor laporan.
Integrasi Sistem

Semua fitur bekerja secara terintegrasi:
- **Celery** = Task queue untuk eksekusi background
- **RabbitMQ** = Message broker mengantarkan tugas dari Django ke Celery Worker
- **Flower** = Dashboard monitoring kesehatan sistem
- **Docker** = Container orchestration untuk scalability

Arsitektur ini memastikan aplikasi tetap responsif meskipun sedang memproses tugas-tugas berat,
Fitur ini menyediakan antarmuka monitoring untuk memantau aktivitas Celery secara real-time. Flower dapat diakses melalui http://localhost:5555 dan menampilkan informasi penting seperti status worker yang sedang online, daftar task yang telah dijalankan beserta statusnya (SUCCESS, FAILURE, atau PENDING), serta informasi broker RabbitMQ yang terhubung. Dengan Flower, pengembang dapat dengan mudah memantau kesehatan sistem dan mendeteksi jika terjadi kegagalan pada task-task Celery. Flower berjalan sebagai container terpisah di Docker Compose dan terhubung langsung dengan Celery Worker dan RabbitMQ.

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

### Kendala

1. **File tidak sinkron antara container dan host**
   - Celery worker tidak memiliki volume mount untuk media folder
   - Path relatif menyebabkan file tersimpan di root container

2. **Error MEDIA_ROOT tidak terdefinisi**
   - Variable MEDIA_ROOT tidak diakses di dalam Celery task
   - Celery Worker tidak membaca variable global dari luar task

3. **Endpoint async tidak mengembalikan task_id**
   - User tidak bisa mengecek status task yang sedang berjalan
   - Respons hanya berisi pesan sukses saja

Proyek Simple LMS berhasil mengintegrasikan teknologi modern untuk membangun sistem learning management yang scalable dan responsif:

**Teknologi yang digunakan:**
- Django REST Framework (backend API)
- PostgreSQL (database utama)
- JWT Authentication (autentikasi tiga role)
- Celery + RabbitMQ (async task processing)
- Redis (caching layer)
- MongoDB (activity logging)
- Docker (containerization)
- Flower & Django Silk (monitoring & profiling)

**Pencapaian:**
- Semua fitur async berhasil diimplementasikan
- Task scheduling berjalan otomatis
- Real-time monitoring dan profiling tersedia
- API documentation lengkap via Swagger
- Berbagai kendala teknis berhasil diatasi

Proyek ini menjadi fondasi kuat untuk pengembangan sistem modern dan memberikan wawasan mendalam tentang arsitektur microservices dan praktik best practices dalam pengembangan aplikasi skala besar

1. **Sinkronisasi file dengan host:**
   - Menambahkan volume mount `./media:/app/media` pada service celery-worker di docker-compose.yml
   - Mengubah path penyimpanan menggunakan `settings.MEDIA_ROOT`

2. **Mengakses MEDIA_ROOT dari task:**
   - Mengganti `MEDIA_ROOT` dengan `settings.MEDIA_ROOT` di dalam task export_course_report
   - Restart Celery Worker agar kode terbaru terbaca

3. **Mengembalikan task_id:**
   - Menyimpan hasil task.delay() sebelum mengembalikan response
   - User sekarang bisa mengecek status melalui endpoint `/api/tasks/{task_id}`

## Kesimpulan

Final project Simple LMS ini memberikan pengalaman berharga dalam memahami implementasi Pemrograman Sistem Skala Besar secara langsung. Proyek ini berhasil mengintegrasikan teknologi modern seperti Docker dengan 8 service, PostgreSQL sebagai database utama, JWT untuk autentikasi tiga role, serta Celery dan RabbitMQ untuk asynchronous processing pada pengiriman email, pembuatan sertifikat PDF, dan ekspor laporan CSV. Celery Beat menjalankan task periodik update statistik, dan tersedia endpoint task status untuk memantau proses async. Redis digunakan untuk cache, MongoDB untuk logging aktivitas, serta Flower dan Django Silk untuk monitoring dan profiling. Dokumentasi API tersedia via Swagger/OpenAPI. Sepanjang pengerjaan, berbagai kendala teknis berhasil diatasi, memperdalam pemahaman tentang Django REST Framework dan arsitektur microservices. Proyek ini menjadi fondasi kuat untuk pengembangan sistem modern ke depannya.





