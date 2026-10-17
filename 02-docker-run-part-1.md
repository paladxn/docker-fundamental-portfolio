# Lab 2.2 — Docker Run (Part 1)

**Course:** Docker Fundamental (Adinusa)
**Topic:** Docker Run — search image, foreground/background, name, port mapping, inspect, logs, stats, top, stop
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Mencari image dari **Docker Hub** dan **Harbor Registry**
- Menjalankan container dalam mode **foreground** dan **background**
- Memberi **nama** container dan mengelola lifecycle-nya
- Melihat informasi container dengan **`inspect`, `logs`, `stats`, `top`**
- Melakukan **port mapping** host ↔ container (fixed & random)
- Memverifikasi container berjalan dengan **`curl`**

---

## 📘 Konsep Dasar

### Perintah Utama

| Perintah | Fungsi |
|---|---|
| `docker search` | Cari image di Docker Hub |
| `skopeo list-tags` | List tag image di registry (Harbor) |
| `docker run` | Buat + jalankan container dari image |
| `docker ps` | List container yang sedang running |
| `docker inspect` | Detail container dalam format JSON |
| `docker logs` | Lihat stdout/stderr container |
| `docker stats` | Resource usage real-time |
| `docker top` | Lihat proses di dalam container |
| `docker stop` | Hentikan container |
| `docker port` | Lihat port mapping |
| `docker images` | List image lokal |

### Registry Penting

| Registry | Keterangan |
|---|---|
| Docker Hub (`docker.io`) | Registry publik default |
| Harbor (`registry.adinusa.id`) | Private registry enterprise Adinusa |

### Konsep `docker run`

`docker run` = kombinasi 3 aksi dalam satu perintah:

```
docker create  →  docker start  →  (docker attach opsional)
```

### Opsi Penting `docker run`

| Opsi | Fungsi |
|---|---|
| `-d` | Jalan di background (detached) |
| `-it` | Interactive + TTY |
| `--name` | Beri nama container |
| `-p host:container` | Publish port ke host |
| `-P` | Publish semua port EXPOSE ke random host |
| `-v` | Mount volume |
| `-e` | Set environment variable |
| `--rm` | Auto-hapus saat container exit |

### Format Port Mapping

```
-p <HOST_PORT>:<CONTAINER_PORT>
   ↑             ↑
port di host    port di dalam container
```

| Variasi | Arti |
|---|---|
| `-p 8080:80` | Akses `localhost:8080` → container port 80 |
| `-p 127.0.0.1:8080:80` | Hanya bisa dari localhost host |
| `-p 8080:80/udp` | Protokol UDP |
| `-p 80` | Host port random |

### Foreground vs Background

| Mode | Blocking | Cocok Untuk |
|---|---|---|
| Foreground (tanpa `-d`) | ✅ Ya | Debug, lihat output real-time |
| Background (`-d`) | ❌ Tidak | Service, produksi |

### Container Lifecycle

```
Created → Running → Paused → Exited → Removed
```

> ⚠️ `docker stop` **tidak sama** dengan `docker rm` — stop hanya menghentikan container tanpa menghapusnya.

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start do-002-2
```

Semua perintah dieksekusi di dalam node **`pod-[username]-node01`**.

---

## 🧪 Practice 01 — Redis Container

### Soal

1. Cari image **redis** dari DockerHub & Harbor.
2. Jalankan container redis dalam mode **foreground**.
3. Jalankan container redis dalam mode **background**.
4. Beri nama container **redis1**.
5. Display running container.
6. Display all docker container.
7. Display container description **redis1**.
8. Display content of the logs in container **redis1**.
9. Display live stream resource used in container **redis1**.
10. Display running process in container **redis1**.
11. Shutdown container **redis1**.

### ✅ Solusi

```bash
# 1. Search image redis
# From DockerHub
$ docker search redis

# From Harbor
$ skopeo list-tags docker://registry.adinusa.id/btacademy/redis

# 2. Running container foreground
$ docker run registry.adinusa.id/btacademy/redis
# Tekan CTRL + c untuk quit

# 3. Running container in background
$ docker run -d registry.adinusa.id/btacademy/redis

# 4. Giving the container a name
$ docker run -d --name redis1 registry.adinusa.id/btacademy/redis

# 5. Display running container
$ docker ps
$ docker container ls

# 6. Display all docker container
$ docker ps -a
$ docker container ls -a

# 7. Display container description
$ docker inspect redis1

# 8. Display content of the logs
$ docker logs redis1

# 9. Display live stream resource
$ docker stats redis1

# 10. Display running process
$ docker top redis1

# 11. Shutdown the container
$ docker stop redis1
```

### ⚠️ Urutan Sangat Penting

```
1. docker search      → cari image
2. skopeo list-tags   → cek tag di Harbor
3. docker run         → foreground (blocking)
4. docker run -d      → background
5. --name redis1      → beri nama ←
6. inspect/logs/stats → debug & monitoring
7. docker stop        → shutdown
```

**Kenapa?** Container tanpa `--name` akan dapat nama random seperti `happy_einstein` — susah di-reference waktu mau `inspect`, `logs`, atau `stop`. Selalu beri nama **sebelum** masuk ke tahap debug.

### 🔍 Penjelasan Opsi Penting

**`docker search redis`**
- Mencari image publik di Docker Hub.
- Output kolom: **NAME**, **DESCRIPTION**, **STARS**, **OFFICIAL** (`[OK]` = resmi dari Docker Inc.), **AUTOMATED** (`[OK]` = build otomatis dari GitHub).

**`skopeo list-tags docker://...`**
- Menampilkan **semua tag** yang tersedia di registry.
- `docker://` = protokol URI untuk Docker Registry HTTP API v2.
- `btacademy` = namespace, `redis` = nama repository.
- Lebih ringan dari `docker pull` karena hanya query metadata tag, **tidak download layer**.

**`docker run <img>` (tanpa `-d`)**
- Jalan di **foreground** — terminal "dipakai" sampai container exit.
- Log output muncul real-time.
- Stop dengan `CTRL + c` (kirim SIGINT).

**`docker run -d <img>`**
- Flag `-d` / `--detach` → jalan di **background**.
- Prompt langsung kembali, output hanya container ID panjang.
- Cek status dengan `docker ps`.

**`docker run -d --name redis1 <img>`**
- `--name redis1` memberi nama container agar mudah di-reference.

**`docker inspect redis1`**
- Detail lengkap container dalam format JSON — Config, State, NetworkSettings, Mounts, Image, Env, Cmd.
- Field penting: `.State.Status`, `.State.Pid`, `.NetworkSettings.IPAddress`, `.Config.Image`, `.Config.Env`.
- Filter pakai Go template: `docker inspect -f '{{.State.Status}}' redis1`.

**`docker logs redis1`**
- Menampilkan **stdout + stderr** proses utama container.
- Opsi: `-f` (follow real-time), `--tail 100`, `--since 10m`, `-t` (timestamp).

**`docker stats redis1`**
- Menampilkan **CPU %, memory, network I/O, disk I/O** real-time (refresh 1 detik).
- Kolom: **CPU %**, **MEM USAGE / LIMIT**, **MEM %**, **NET I/O**, **BLOCK I/O**, **PIDS**.
- Keluar: `Ctrl + C`. Sekali snapshot: `--no-stream`.

**`docker top redis1`**
- Menampilkan **daftar proses** di dalam container — mirip `ps aux` di host tapi dibatasi container tersebut.
- PID berbeda dengan host karena container punya **PID namespace** sendiri.

**`docker stop redis1`**
- Mengirim **SIGTERM** → tunggu 10 detik → **SIGKILL** kalau belum stop.
- Grace period ubah dengan `docker stop -t 30 redis1`.
- Alternatif keras: `docker kill redis1` (langsung SIGKILL).

---

## 🧪 Practice 02 — NGINX Container + Port Mapping

### Soal

1. Cari image **nginx** dari DockerHub & Harbor.
2. Jalankan container **nginx1** dengan port mapping `80:80`.
3. Display description container **nginx1**.
4. Jalankan container **nginx2** dengan `-p 80` (random host port).
5. Test browsing ke nginx1 & nginx2.
6. Display container (activate/exit).
7. Display image docker.

### ✅ Solusi

```bash
# 1. Search image nginx
# From DockerHub
$ docker search nginx

# From Harbor
$ skopeo list-tags docker://registry.adinusa.id/btacademy/nginx

# 2. Running image nginx + expose to host port
$ docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest

# 3. Display description nginx1
$ docker inspect nginx1

# 4. Running image nginx + declare container port (random host port)
$ docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest

# 5. Test browsing
$ curl localhost:$(docker port nginx2 80 | cut -d : -f 2)

# 6. Display container (activate/exit)
$ docker ps -a
$ docker container ls -a

# 7. Display image docker
$ docker images
```

### ⚠️ Urutan Sangat Penting

```
1. docker search           → cari image
2. skopeo list-tags        → cek tag di Harbor
3. --name nginx1 -p 80:80  → fixed port
4. inspect                 → cek konfigurasi
5. --name nginx2 -p 80     → random host port ←
6. docker port             → cek host port yang dipilih Docker
7. curl                    → verifikasi akses
8. docker images           → cek image lokal
```

**Kenapa?** Tanpa `--name`, container dapat nama random → susah di-reference. Tanpa `-p`, container tidak bisa diakses dari host. Urutan `HOST:CONTAINER` di `-p` **tidak boleh kebalik** — kalau kebalik, mapping jadi salah.

### 🔍 Penjelasan Opsi Penting

**`docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest`**
- `-d` = background.
- `--name nginx1` = nama container `nginx1`.
- `-p 80:80` = port mapping `<host>:<container>` — host port 80 → container port 80.
- `registry.adinusa.id/btacademy/nginx` = image dari Harbor Adinusa.
- `:latest` = tag versi terbaru.

**`docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest`**
- `-p 80` = hanya sebut **port container** → Docker pilih host port **random** yang masih kosong.
- Keuntungan: bisa jalankan banyak container tanpa port conflict.
- Kerugian: host port tidak predictable — harus dicek dengan `docker port`.

**`docker inspect nginx1`**
- Detail JSON container. Fokus ke bagian network:
  - `NetworkSettings.Ports` → mapping port aktif.
  - `NetworkSettings.IPAddress` → IP internal container.
  - `Mounts` → volume/bind mount kalau ada.
  - `Config.Env` → environment variable.
- Filter: `docker inspect -f '{{.NetworkSettings.Ports}}' nginx1`.

**`docker port nginx2`**
- Menampilkan mapping port container ke host.
- `docker port nginx2 80` untuk spesifik port 80 container.
- Contoh output: `80/tcp -> 0.0.0.0:32768`.

**`curl localhost:$(docker port nginx2 80 | cut -d : -f 2)`**
- `docker port nginx2 80` → output `0.0.0.0:32768`.
- `cut -d : -f 2` → ambil setelah `:` → `32768`.
- `$(...)` → command substitution → jadikan argumen.
- Hasil akhir: `curl localhost:32768` → HTTP GET ke port tersebut.

**`docker images`**
- Menampilkan image lokal. Kolom: **REPOSITORY**, **TAG**, **IMAGE ID**, **CREATED**, **SIZE**.
- Filter: `docker images nginx` atau `docker images --format "{{.Repository}}:{{.Tag}} ({{.Size}})"`.

---

## 🔍 Verifikasi

### 1. Cek container running

```bash
$ docker ps
```

Output yang diharapkan:

| CONTAINER | STATUS | PORTS |
|---|---|---|
| nginx1 | Up | 0.0.0.0:80->80/tcp |
| nginx2 | Up | 0.0.0.0:32768->80/tcp (random) |

### 2. Cek semua container (termasuk exited)

```bash
$ docker ps -a
```

Output yang diharapkan:

| CONTAINER | STATUS | PORTS |
|---|---|---|
| nginx1 | Up | 0.0.0.0:80->80/tcp |
| nginx2 | Up | 0.0.0.0:32768->80/tcp |
| redis1 | Exited (0) | - |

### 3. Cek image lokal

```bash
$ docker images
```

Output yang diharapkan: image `registry.adinusa.id/btacademy/redis` dan `registry.adinusa.id/btacademy/nginx` muncul di daftar.

### 4. Test akses nginx1 & nginx2

```bash
$ curl localhost:80
$ curl localhost:$(docker port nginx2 80 | cut -d : -f 2)
```

Expected: HTML default NGINX (`Welcome to nginx!`).

### 5. Cek status redis1

```bash
$ docker ps -a | grep redis1
```

Expected: muncul dengan status `Exited (0)`.

### 6. Grading

```bash
$ nusactl grade do-002-2
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Nyata yang Dialami

- **Container foreground "nyangkut" tidak bisa keluar:**

  **Penyebab:** Lupa bahwa `docker run` tanpa `-d` bersifat blocking.

  **Solusi:** Tekan `CTRL + c` untuk kirim SIGINT. Kalau tidak responsif, buka terminal baru dan `docker stop <name>`.

- **Port 80 sudah dipakai container lain:**

  **Penyebab:** Ada container/web server lain yang pakai port 80.

  **Gejala:**
  ```
  Error response from daemon: driver failed programming external connectivity:
  Bind for 0.0.0.0:80 failed: port is already allocated
  ```

  **Solusi:**
  ```bash
  $ docker ps --filter "publish=80"
  $ docker stop nginx1 && docker rm nginx1
  $ docker run -d --name nginx1 -p 8080:80 nginx:alpine
  ```

- **`curl localhost:$(docker port nginx2 80 | cut -d : -f 2)` error "No such file":**

  **Penyebab:** `docker port nginx2 80` belum ada outputnya — container belum siap atau tidak running.

  **Solusi:** Cek `docker ps` dulu, pastikan nginx2 statusnya Up.

- **Container langsung exit setelah `docker run -d`:**

  **Penyebab:** Image tidak punya long-running command, atau command langsung selesai.

  **Gejala:** `docker ps` kosong, `docker ps -a` menunjukkan `Exited (0)`.

  **Solusi:** Cek `docker logs <name>` untuk lihat output. Pastikan image yang dipakai memang untuk service.

### Kesalahan Umum Lainnya

- **Lupa flag `-d`:** Container jalan di foreground → terminal blocking. Selalu pakai `-d` untuk service.

- **Salah urutan port mapping:** Format adalah `HOST:CONTAINER`, bukan `CONTAINER:HOST`. `-p 80:8080` artinya host 80 → container 8080.

- **Lupa `--name`:** Container dapat nama random → susah di-reference. Selalu beri `--name`.

- **Hapus container yang masih running:** Harus `docker stop` dulu atau `docker rm -f`.

- **`docker inspect` output terlalu panjang:** Pakai `-f` filter atau pipe ke `grep`/`less`.

- **Salah image tag:** `nginx:latest` ≠ `nginx:1.25-alpine` — perilaku dan ukuran bisa beda. Selalu pin versi di produksi.

- **Log grading gagal di case tertentu:**
  ```bash
  $ cat /var/log/nusactl/labs/do-002-2.log
  ```

---

## 💡 Catatan & Insight Pribadi

### Foreground vs Background — Kapan Pakai Apa?

| Skenario | Mode |
|---|---|
| Debug aplikasi baru | Foreground (lihat output langsung) |
| Service produksi (web, DB) | Background (`-d`) |
| One-shot task (migrasi DB) | Foreground atau `--rm` |
| Container dengan logging ke file | Background |

**Rule of thumb:** Kalau butuh interaksi atau lihat output real-time → foreground. Kalau service yang jalan lama → background.

### Port Mapping — Analogi Sederhana

Bayangkan container itu rumah kos. Port mapping seperti **nomor pintu depan**:

- Container punya **nomor internal** (port 80, 3000, 5432) yang hanya diketahui "penghuni" (container lain).
- Host punya **nomor publik** (80, 8080) yang dilihat "tamu dari luar" (browser, curl).
- `-p 8080:80` artinya: "Tamu ketuk pintu 8080, saya antar ke pintu 80 dalam rumah."

Tanpa `-p`, container **invisible** dari host — hanya bisa diakses container lain di network yang sama.

### Docker Hub vs Harbor — Kapan Pakai Mana?

| Aspek | Docker Hub | Harbor |
|---|---|---|
| Akses | Publik | Private (internal) |
| Rate limit | Ada (100/6 jam anon) | Tidak ada |
| Kualitas | Bervariasi (official aman) | Terkurasi internal |
| Use case | Belajar, public image | Produksi perusahaan |

**Best practice:** Untuk produksi perusahaan, gunakan **private registry** seperti Harbor. Untuk learning/personal project, Docker Hub sudah cukup.

### `docker inspect` — Field yang Sering Dicek

| Field | Kapan Dicek |
|---|---|
| `.State.Status` | Cek container running/exited |
| `.State.Pid` | Cek PID di host |
| `.NetworkSettings.IPAddress` | Cek IP internal container |
| `.NetworkSettings.Ports` | Cek port mapping |
| `.Config.Env` | Cek env variable |
| `.Mounts` | Cek volume/bind mount |

Gunakan `-f` Go template untuk ambil field tertentu tanpa output JSON panjang.

### `docker stop` vs `docker kill` vs `docker rm`

| Perintah | Efek |
|---|---|
| `docker stop` | SIGTERM → 10s → SIGKILL (graceful) |
| `docker kill` | SIGKILL langsung (paksa) |
| `docker rm` | Hapus container (harus stop dulu) |
| `docker rm -f` | Force remove (running → hapus) |

**Rekomendasi:** Selalu `docker stop` dulu untuk beri kesempatan aplikasi shutdown graceful.

### `docker ps` vs `docker ps -a`

| Perintah | Isi |
|---|---|
| `docker ps` | Hanya container **running** |
| `docker ps -a` | **Semua** container (termasuk `Exited`) |

Container yang sudah `stop` tetap ada — konfigurasi & datanya masih bisa diinspect atau di-restart. Untuk benar-benar hapus, gunakan `docker rm`.

### Real-World Use Cases

- **Debug aplikasi baru:** `docker run -it --rm alpine sh` → masuk ke shell, coba command, exit → otomatis hapus.
- **Web server portabel:** `docker run -d --name web -p 8080:80 nginx` → langsung bisa akses dari browser.
- **Testing multiple version:** `docker run -d -P nginx:1.25` & `docker run -d -P nginx:1.24` → bandingkan perilaku.
- **Quick database:** `docker run -d --name db -p 5432:5432 postgres:15` → DB langsung ready.

### Pelajaran Kunci dari Lab Ini

1. **`docker run` = create + start** — satu perintah untuk 2 aksi.
2. **`-d` wajib** untuk service agar terminal tidak blocking.
3. **`--name` wajib** agar mudah di-reference.
4. **`-p HOST:CONTAINER`** — urutan penting, jangan kebalik.
5. **`-p CONTAINER` saja** → Docker pilih host port random.
6. **`docker port`** untuk cek port yang dipilih Docker.
7. **`docker inspect`** untuk detail lengkap, pakai `-f` untuk filter.
8. **`docker logs`** untuk debug output aplikasi.
9. **`docker stats`** untuk monitoring resource real-time.
10. **`docker top`** untuk lihat proses di dalam container.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `nusactl login` | Login ke akun Adinusa |
| `nusactl start do-002-2` | Start & preconfigure environment lab |
| `docker search <img>` | Cari image di Docker Hub |
| `skopeo list-tags docker://...` | List tag image di Harbor tanpa pull |
| `docker run <img>` | Jalankan container (foreground) |
| `docker run -d <img>` | Jalankan container (background) |
| `docker run -d --name X <img>` | + beri nama container |
| `docker run -d -p H:C <img>` | + port mapping host:container |
| `docker run -d -p C <img>` | + random host port |
| `docker ps` | List container running |
| `docker ps -a` | List semua container (termasuk exited) |
| `docker container ls [-a]` | Alias modern dari `docker ps` |
| `docker inspect <c>` | Detail container (JSON) |
| `docker inspect -f '...' <c>` | Filter field spesifik |
| `docker logs <c>` | Lihat stdout/stderr container |
| `docker stats <c>` | Resource usage real-time |
| `docker top <c>` | Proses di dalam container |
| `docker stop <c>` | Stop container (SIGTERM → SIGKILL) |
| `docker port <c> [port]` | Lihat port mapping |
| `docker images` | List image lokal |

**File & konsep penting:**

| Konsep | Keterangan |
|---|---|
| Image | Template read-only |
| Container | Instance running dari image |
| Registry | Tempat simpan image (Docker Hub / Harbor) |
| Port mapping | Host:Container |
| Foreground | Blocking, untuk debug |
| Background | Non-blocking (`-d`), untuk service |

**Urutan perintah lengkap (final):**

```bash
# ===== Practice 01: Redis =====
docker search redis
skopeo list-tags docker://registry.adinusa.id/btacademy/redis

docker run registry.adinusa.id/btacademy/redis
docker run -d registry.adinusa.id/btacademy/redis
docker run -d --name redis1 registry.adinusa.id/btacademy/redis

docker ps
docker ps -a
docker inspect redis1
docker logs redis1
docker stats redis1
docker top redis1
docker stop redis1

# ===== Practice 02: NGINX =====
docker search nginx
skopeo list-tags docker://registry.adinusa.id/btacademy/nginx

docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest
docker inspect nginx1
docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest

curl localhost:80
curl localhost:$(docker port nginx2 80 | cut -d : -f 2)

docker ps -a
docker images
```

**Verifikasi cepat:**

```bash
docker ps                    # harus muncul nginx1 & nginx2
docker ps -a                 # harus muncul redis1 (Exited), nginx1 & nginx2 (Up)

curl localhost:80            # Welcome to nginx!
curl localhost:$(docker port nginx2 80 | cut -d : -f 2)  # Welcome to nginx!

docker images | grep btacademy

nusactl grade do-002-2
```

---

## 📌 Kesimpulan

Lab 2.2 ini menguji kemampuan dasar **menjalankan dan mengelola container Docker** — dari mencari image di registry, menjalankan dalam berbagai mode, sampai inspect dan debug. Yang diuji:

- **`docker search`** & **`skopeo list-tags`** untuk discovery image.
- **`docker run`** dengan berbagai opsi (`-d`, `--name`, `-p`).
- **`docker ps`** & **`docker ps -a`** untuk melihat state container.
- **`docker inspect`** untuk detail konfigurasi.
- **`docker logs`**, **`docker stats`**, **`docker top`** untuk debug & monitoring.
- **`docker stop`** untuk shutdown graceful.
- **`docker port`** untuk cek mapping port.

Kemampuan ini adalah **fondasi dari semua operasi Docker selanjutnya** — hampir setiap hari berurusan dengan `docker run`, `docker ps`, dan `docker logs`. Yang paling berharga dari lab ini:

1. **Pelajaran foreground vs background** — pilih mode yang tepat sesuai use case.
2. **Pelajaran port mapping** — `HOST:CONTAINER`, bukan sebaliknya.
3. **Pelajaran `--name`** — selalu beri nama container agar mudah di-reference.

Ketiga pelajaran ini **sangat berharga** untuk workflow Docker sehari-hari — detail kecil seperti urutan port mapping atau lupa `-d` bisa bikin workflow kacau.

**Motto Docker:** *Container yang rapi = workflow yang cepat.*

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.