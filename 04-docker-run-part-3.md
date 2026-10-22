# Lab 2.4 — Docker Run (Part 3)

**Course:** Docker Fundamental (Adinusa)
**Topic:** Docker Run — environment variables, linking, pause/unpause, resource limits, Linux capabilities
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menjalankan container database dengan konfigurasi **environment variables** (`-e`)
- Menghubungkan container menggunakan **container linking** (`--link`)
- Mengelola status container dengan **`pause`** dan **`unpause`**
- Membatasi resource container (**`--memory`**, **`--cpu-shares`**)
- Meningkatkan keamanan container dengan **`--cap-drop`**
- Melakukan monitoring resource dan log pada container yang kompleks

---

## 📘 Konsep Dasar

### Environment Variables (`-e`)

Container seringkali membutuhkan konfigurasi awal, seperti password database atau nama database. Image seperti `mysql`, `mariadb`, dan `wordpress` dirancang untuk membaca variabel environment saat pertama kali dijalankan.

| Env Var | Fungsi |
|---|---|
| `MYSQL_ROOT_PASSWORD` | Set password untuk user root database |
| `MYSQL_DATABASE` | Otomatis membuat database dengan nama ini |
| `WORDPRESS_DB_PASSWORD` | Password database yang dipakai WordPress |

### Container Linking (`--link`)

> ⚠️ **Catatan:** `--link` adalah fitur *legacy* Docker. Di production modern, disarankan menggunakan **Docker Network**. Namun, di lab ini kita menggunakan `--link` sesuai instruksi.

Format: `--link <nama_container_target>:<alias_hostname>`
Contoh: `--link my-mysql:db` → container target (`my-mysql`) bisa diakses dengan hostname `db` dari dalam container yang me-link.

### Pause vs Stop

| Perintah | Efek | Status di `docker ps` |
|---|---|---|
| `docker pause` | Membekukan (*freeze*) proses menggunakan `cgroup freezer`. | `Up X (Paused)` |
| `docker unpause` | Mencairkan proses yang dibekukan. | `Up X` |
| `docker stop` | Mengirim sinyal `SIGTERM`, mematikan proses utama. | `Exited (0)` |

### Resource Limits

| Flag | Fungsi |
|---|---|
| `--memory` | Batas maksimal RAM (misal: `256m`, `1g`). Jika lewat batas, container akan OOM (Out Of Memory) dan mati. |
| `--cpu-shares` | Bobot prioritas CPU (default: `1024`). Ini *bukan* batas maksimal, melainkan bobot relatif jika terjadi *resource contention*. |

### Linux Capabilities (`--cap-drop`)

Docker secara default memberikan sekumpulan *capabilities* Linux ke container (misal: `NET_RAW` untuk membuat paket raw/ICMP). Flag `--cap-drop` digunakan untuk mencabut hak akses tertentu demi keamanan (*principle of least privilege*).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start do-002-4
```

Semua perintah dieksekusi di dalam node **`pod-[username]-node01`**.

---

## 🧪 Practice 05 — MySQL & phpMyAdmin (Environment & Link)

### Soal

1. Jalankan container **mysql** dengan parameter tambahan (env vars & port).
2. Pull image **phpmyadmin** dari Harbor.
3. Jalankan **phpmyadmin** dan hubungkan dengan container mysql.
4. Test browsing phpMyAdmin.

### ✅ Solusi

```bash
# 1. Running mysql container with additional parameters
$ docker run -d --name my-mysql -e MYSQL_ROOT_PASSWORD=RAHASIA -e MYSQL_DATABASE=latihan05 -p 3306:3306 \
  registry.adinusa.id/btacademy/mysql

# 2. Pull image phpmyadmin
$ docker pull registry.adinusa.id/btacademy/phpmyadmin:latest

# 3. Running phpmyadmin container and connect it with mysql container
$ docker run --name my-phpmyadmin -d --link my-mysql:db -p 8090:80 \
  registry.adinusa.id/btacademy/phpmyadmin

# 4. Test browsing
# Buka di browser: http://10.10.10.11:8090
# Login dengan user: root, password: RAHASIA
```

### ⚠️ Urutan Sangat Penting

```
1. docker run mysql        → buat DB & set password
2. docker pull phpmyadmin  → pastikan image GUI ada
3. docker run phpmyadmin   → hubungkan via --link
4. Browser test            → login pakai root & RAHASIA
```

**Kenapa?** `--link my-mysql:db` hanya bisa bekerja jika container target (`my-mysql`) **sudah running** terlebih dahulu. Hostname `db` yang dipakai phpMyAdmin (di belakang layar) akan resolve ke IP container `my-mysql`.

### 🔍 Penjelasan Opsi Penting

**`-e MYSQL_ROOT_PASSWORD=RAHASIA -e MYSQL_DATABASE=latihan05`**
- `-e` (*environment*) menyuntikkan variabel ke dalam container.
- Saat container MySQL pertama kali *start*, script entrypoint akan membaca variabel ini untuk mengkonfigurasi database secara otomatis.

**`--link my-mysql:db`**
- Menghubungkan container `my-phpmyadmin` ke `my-mysql`.
- Memberikan alias `db`. Artinya, dari dalam phpMyAdmin, kita bisa ping/connect ke host `db` seolah-olah itu nama domain.

**`-p 8090:80`**
- Port 80 milik container phpMyAdmin (web interface) diekspos ke port 8090 di host.

---

## 🧪 Practice 06 — Pause & Unpause Container

### Soal

1. Run ubuntu containers dengan nama **ubuntu1** & **ubuntu2**.
2. Display list container.
3. Pause container ubuntu1 & ubuntu2.
4. Cek status di list container (harus paused).
5. Cek resource usage saat paused.
6. Unpause container ubuntu1.

### ✅ Solusi

```bash
# 1. Run ubuntu containers with names ubuntu1 & ubuntu2
# Perhatikan flag -dit (detach, interactive, tty)
$ docker run -dit --name ubuntu1 registry.adinusa.id/btacademy/ubuntu
$ docker run -dit --name ubuntu2 registry.adinusa.id/btacademy/ubuntu

# 2. Display list container
$ docker ps

# 3. Pause container ubuntu
$ docker pause ubuntu1
$ docker pause ubuntu2

# 4. Check in list container if the container status is paused
$ docker ps

# 5. Check resource usage when the ubuntu container is paused
$ docker stats --no-stream ubuntu1
$ docker stats --no-stream ubuntu2

# 6. Unpause container ubuntu1
$ docker unpause ubuntu1
```

### ⚠️ Urutan Sangat Penting

```
1. docker run -dit   → pastikan container Ubuntu tetap hidup (running)
2. docker pause      → bekukan
3. docker stats      → lihat resource drop ke 0
4. docker unpause    → cairkan kembali
```

**Kenapa pakai `-dit`?** Jika hanya `docker run -d`, container Ubuntu akan langsung `Exited` karena tidak ada proses *foreground* yang berjalan. Flag `-it` (interactive TTY) membuat bash tetap terbuka, dan `-d` menjalankannya di background.

### 🔍 Penjelasan Opsi Penting

**`docker pause <c>`**
- Menggunakan fitur kernel Linux `cgroup freezer` untuk menghentikan semua proses di dalam container secara instan (seperti `SIGSTOP`).

**`docker stats --no-stream`**
- `--no-stream` memaksa `docker stats` untuk hanya menampilkan satu kali output lalu keluar (tidak me-refresh terus menerus seperti mode *watch*). Saat di-pause, CPU dan MEM akan menunjukkan angka `0`.

---

## 🧪 Practice 07 — Resource Limits & Capabilities (WordPress + MariaDB)

### Soal

1. Buat container database (**mariadb**) dengan spesifikasi terbatas.
2. Buat container **wordpress** dan hubungkan ke database.
3. Cek logs, running process, dan resource.
4. Test browsing WordPress.

### ✅ Solusi

```bash
# 1. Create database containers with limited specifications
$ docker container run -d --name ch6_mariadb --memory 256m --cpu-shares 1024 --cap-drop net_raw \
  -e MYSQL_ROOT_PASSWORD=test registry.adinusa.id/btacademy/mariadb:5.5

# 2. Create wordpress container and connect it to the database
$ docker container run -d -p 80:80 -P --name ch6_wordpress --memory 512m --cpu-shares 512 \
  --cap-drop net_raw --link ch6_mariadb:mysql -e WORDPRESS_DB_PASSWORD=test \
  registry.adinusa.id/btacademy/wordpress:5.0.0-php7.2-apache

# 3. Check logs, running process, and resource
$ docker logs ch6_mariadb
$ docker logs ch6_wordpress
$ docker top ch6_mariadb
$ docker top ch6_wordpress
$ docker stats --no-stream ch6_mariadb
$ docker stats --no-stream ch6_wordpress

# 4. Test browsing
# Buka di browser: http://10.10.10.11
# Selesaikan proses instalasi WordPress
```

### ⚠️ Urutan Sangat Penting

```
1. docker run mariadb    → setup DB (batasi RAM 256m)
2. docker run wordpress  → setup App (batasi RAM 512m) + link ke DB
3. docker logs           → pastikan tidak ada error koneksi
4. Browser test          → install WordPress
```

**Kenapa?** Sama seperti Practice 05, container WordPress membutuhkan database. `--link ch6_mariadb:mysql` memastikan WordPress bisa menemukan MariaDB menggunakan hostname `mysql`.

### 🔍 Penjelasan Opsi Penting

**`--memory 256m` & `--memory 512m`**
- Membatasi RAM. Jika WordPress atau MariaDB mencoba menggunakan RAM melebihi batas ini, kernel akan membunuh proses tersebut (OOM Killer).

**`--cpu-shares 1024` vs `--cpu-shares 512`**
- Ini adalah *relative weight*. Jika host memiliki beban CPU tinggi, MariaDB akan mendapat porsi CPU **dua kali lebih besar** dibanding WordPress (karena 1024 berbanding 512).

**`--cap-drop net_raw`**
- Mencabut hak akses `NET_RAW`. Artinya, proses di dalam container tidak bisa membuat *raw sockets* (seperti yang dibutuhkan untuk `ping` atau manipulasi paket jaringan tingkat rendah). Ini memitigasi risiko eskalasi hak akses dari dalam container.

**`-p 80:80 -P`**
- `-p 80:80` memetakan port web WordPress ke port 80 host.
- `-P` (*Publish all*) akan mengekspos semua port yang di-`EXPOSE` oleh image WordPress (jika ada port lain) ke port random host.

---

## 🔍 Verifikasi

### 1. Cek phpMyAdmin
Buka browser ke `http://10.10.10.11:8090`. Pastikan halaman login phpMyAdmin muncul. Coba login dengan user `root` dan password `RAHASIA`.

### 2. Cek status pause Ubuntu
```bash
$ docker ps
```
Output: `ubuntu2` harus memiliki status `Up X (Paused)`. `ubuntu1` harusnya sudah `Up X` (karena sudah di-unpause).

### 3. Cek limit resource WordPress
```bash
$ docker inspect ch6_wordpress | grep -i memory
$ docker inspect ch6_wordpress | grep -i cpu
```
Output: Memastikan limit `512000000` (512MB) dan `CpuShares: 512` terpasang.

### 4. Cek WordPress
Buka browser ke `http://10.10.10.11`. Pilih bahasa **English**, isi form instalasi (judul web, user admin, dll), lalu klik **Install WordPress**. Jika berhasil, artinya koneksi ke `ch6_mariadb` berjalan lancar.

### 5. Grading
```bash
$ nusactl grade do-002-4
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Nyata yang Dialami

- **phpMyAdmin error "Cannot log in to the MySQL server":**

  **Penyebab:** Container `my-mysql` belum siap menerima koneksi, atau `--link` gagal.

  **Solusi:** Cek `docker logs my-mysql`. Jika masih proses inisialisasi, tunggu 10-20 detik lalu refresh browser. Pastikan juga container `my-mysql` statusnya `Up`.

- **WordPress error "Error establishing a database connection":**

  **Penyebab:** Sama seperti di atas, atau parameter `-e WORDPRESS_DB_PASSWORD=test` tidak cocok dengan password MariaDB (`-e MYSQL_ROOT_PASSWORD=test`).

  **Solusi:** Pastikan password sama persis. Cek log dengan `docker logs ch6_wordpress`.

- **Container Ubuntu langsung mati (Exited):**

  **Penyebab:** Lupa menggunakan flag `-it` (atau `-dit`).

  **Solusi:** Hapus container yang mati (`docker rm ubuntu1 ubuntu2`) dan jalankan ulang dengan `docker run -dit --name ubuntu1 ...`.

- **Container OOM (Out Of Memory) Killed:**

  **Penyebab:** Limit RAM (`--memory`) terlalu kecil untuk aplikasi (misal WordPress butuh lebih dari 512MB saat proses instalasi berat).

  **Solusi:** Cek dengan `docker inspect <c> | grep OOMKilled`. Jika `true`, naikkan limit RAM di perintah `docker run`.

### Kesalahan Umum Lainnya

- **`--link` tidak bekerja lintas network:** `--link` hanya bekerja pada *default bridge network*. Jika menggunakan custom network, gunakan nama container langsung sebagai hostname.
- **Mengedit port tapi bentrok:** Jika port `8090` atau `80` sudah dipakai, ganti dengan port lain (misal `-p 8091:80`).
- **Mengira `--cpu-shares` adalah limit:** `--cpu-shares` bukanlah *hard limit* (seperti `--cpus 0.5`), melainkan bobot prioritas.

---

## 💡 Catatan & Insight Pribadi

### Legacy `--link` vs Modern `Docker Network`

Meskipun lab ini menggunakan `--link`, di dunia nyata (production), `--link` sudah **deprecated** sejak Docker 1.9. Pendekatan modern adalah menggunakan **Custom Bridge Network**:

```bash
# Modern approach (Best Practice)
docker network create mynet
docker run -d --name db --network mynet -e MYSQL_ROOT_PASSWORD=RAHASIA mysql
docker run -d --name phpmyadmin --network mynet -p 8090:80 phpmyadmin
# phpMyAdmin bisa langsung connect ke host "db" karena DNS internal Docker network
```

### Kapan Pakai `pause`?

- **`pause`** (Freeze): Saat kita ingin menghentikan sementara proses agar tidak memakan CPU, tapi kita tidak ingin kehilangan *state* di memori. Cocok untuk *maintenance* singkat.
- **`stop`** (Graceful Shutdown): Saat kita benar-benar ingin mematikan aplikasi, menyimpan data ke disk, dan membebaskan semua resource.

### Mengapa `--cap-drop` Penting?

Secara default, Docker memberikan sekitar 14 Linux Capabilities. Fitur `--cap-drop` memungkinkan kita menerapkan *Least Privilege*. Men-drop `net_raw` mencegah container melakukan serangan *ping flood* atau memanipulasi *raw packets* (sering menjadi celah keamanan).

### Mengapa WordPress Butuh 2 Container?

Ini adalah pola **Microservices / Separation of Concerns**. 
- Container `ch6_mariadb` khusus menangani data (stateful).
- Container `ch6_wordpress` khusus menangani logika web (stateless).
Keuntungan: Kita bisa *scale* WordPress tanpa harus menduplikasi database. Jika WordPress crash, database tetap aman.

### Pelajaran Kunci dari Lab Ini

1. **`-e`** adalah cara utama untuk konfigurasi awal aplikasi di container.
2. **`--link <target>:<alias>`** menyuntikkan entri ke `/etc/hosts` container sumber.
3. **`pause`** = membekukan container secara instan tanpa mematikan proses.
4. **`--memory`** = limit memori (OOM Killer siap mengeksekusi).
5. **`--cpu-shares`** = bobot prioritas CPU (bukan batas mutlak).
6. **`--cap-drop`** = mencabut hak istimewa kernel untuk keamanan.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `docker run -e KEY=VAL` | Inject environment variable ke container |
| `docker run --link <target>:<alias>` | Hubungkan container (legacy) |
| `docker pause <c>` | Bekukan proses container |
| `docker unpause <c>` | Cairkan proses container |
| `docker run --memory <limit>` | Batasi RAM maksimal |
| `docker run --cpu-shares <weight>` | Set bobot prioritas CPU |
| `docker run --cap-drop <cap>` | Cabut Linux capability |
| `docker run -P` | Publish semua exposed ports ke random host |
| `docker stats --no-stream` | Snapshot resource sekali lihat |

**File & konsep penting:**

| Konsep | Keterangan |
|---|---|
| Environment Variables | Cara pass config tanpa hardcode di image |
| Container Link | Legacy DNS resolution via `/etc/hosts` |
| cgroup freezer | Teknologi kernel di balik `docker pause` |
| OOM Killer | Mekanisme kernel membunuh proses jika RAM penuh |
| Capabilities | Hak istimewa Linux tingkat granular |

**Urutan perintah lengkap (final):**

```bash
# ===== Practice 05: MySQL & phpMyAdmin =====
docker run -d --name my-mysql -e MYSQL_ROOT_PASSWORD=RAHASIA -e MYSQL_DATABASE=latihan05 -p 3306:3306 registry.adinusa.id/btacademy/mysql

docker pull registry.adinusa.id/btacademy/phpmyadmin:latest
docker run --name my-phpmyadmin -d --link my-mysql:db -p 8090:80 registry.adinusa.id/btacademy/phpmyadmin

# ===== Practice 06: Pause/Unpause =====
docker run -dit --name ubuntu1 registry.adinusa.id/btacademy/ubuntu
docker run -dit --name ubuntu2 registry.adinusa.id/btacademy/ubuntu

docker pause ubuntu1
docker pause ubuntu2
docker ps
docker stats --no-stream ubuntu1
docker unpause ubuntu1

# ===== Practice 07: Resource Limits =====
docker container run -d --name ch6_mariadb --memory 256m --cpu-shares 1024 --cap-drop net_raw -e MYSQL_ROOT_PASSWORD=test registry.adinusa.id/btacademy/mariadb:5.5

docker container run -d -p 80:80 -P --name ch6_wordpress --memory 512m --cpu-shares 512 --cap-drop net_raw --link ch6_mariadb:mysql -e WORDPRESS_DB_PASSWORD=test registry.adinusa.id/btacademy/wordpress:5.0.0-php7.2-apache

docker logs ch6_wordpress
docker top ch6_wordpress
docker stats --no-stream ch6_wordpress
```

---

## 📌 Kesimpulan

Lab 2.4 ini melangkah lebih jauh ke ranah **manajemen resource dan konektivitas antar container**. Kita tidak hanya menjalankan satu container, tapi merangkai beberapa container agar saling terhubung (DB & Web App), membatasinya, dan mengelola statusnya.

Yang diuji:
- **Environment Variables (`-e`)** untuk konfigurasi otomatis.
- **`--link`** untuk resolusi DNS antar container (legacy).
- **`pause` / `unpause`** untuk state *freezing*.
- **`--memory` & `--cpu-shares`** untuk manajemen resource (QoS).
- **`--cap-drop`** untuk *hardening* keamanan container.

Dua pelajaran paling berharga:
1. **Resource Limit itu Wajib di Production** — Tanpa `--memory`, satu container bisa "memakan" seluruh RAM host dan membuat container lain mati (efek *noisy neighbor*).
2. **Prinsip Least Privilege (`--cap-drop`)** — Container yang tidak butuh akses jaringan tingkat rendah sebaiknya dicabut haknya. Keamanan dimulai dari pembatasan sekecil mungkin.

**Motto Docker:** *Beri ruang yang cukup, tapi jangan biarkan dia mengambil alih rumah.*

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
