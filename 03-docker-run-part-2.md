# Lab 2.3 — Docker Run (Part 2)

**Course:** Docker Fundamental (Adinusa)
**Topic:** Docker Run — exec, package install, start, logs, stats, top, ephemeral container (--rm)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan lab ini, saya mampu:

- Menjalankan container dengan **port mapping** yang berbeda-beda
- Melakukan **akses interaktif** ke dalam container menggunakan `docker exec`
- Melakukan **modifikasi file di dalam container** (install package, edit file, restart service)
- Mengelola lifecycle container dengan `docker start`
- Memahami konsep **ephemeral container** menggunakan flag `--rm`
- Melakukan monitoring container dengan `logs`, `stats`, dan `top`

---

## 📘 Konsep Dasar

### Perintah Utama

| Perintah | Fungsi |
|---|---|
| `docker run -it` | Jalankan container interaktif + TTY |
| `docker exec -it` | Masuk ke dalam container yang sedang running |
| `docker start` | Jalankan container yang sudah ada (status Exited) |
| `docker logs` | Lihat stdout/stderr container |
| `docker stats` | Resource usage real-time |
| `docker top` | Lihat proses di dalam container |
| `docker pull` | Download image dari registry |

### Container Ephemeral (`--rm`)

Flag `--rm` pada `docker run` akan **otomatis menghapus** container dari daftar (`docker ps -a`) segera setelah container tersebut exit/berhenti. Sangat berguna untuk *one-shot task* atau *temporary testing* agar tidak menumpuk sampah container.

### Akses Interaktif: `exec` vs `attach`

| Perintah | Karakteristik |
|---|---|
| `docker exec -it <c> bash` | Membuka *sesi baru* di dalam container. Keluar dari sesi ini tidak mematikan container. |
| `docker attach <c>` | Menempel ke *sesi utama* container. Menekan `Ctrl+C` bisa mematikan container. |

### Modifikasi di Dalam Container

Container memiliki *writable layer*. Jika kita menginstall package atau mengedit file di dalam container menggunakan `exec`, perubahan tersebut akan tersimpan selama container tersebut belum dihapus (`docker rm`).

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start do-002-3
```

Semua perintah dieksekusi di dalam node **`pod-[username]-node01`**.

---

## 🧪 Practice 03 — NGINX Custom Page & Container Inspection

### Soal

1. Cari image **nginx** dari DockerHub & Harbor.
2. Jalankan **nginx1** expose port `8080`.
3. Inspect container **nginx1**.
4. Jalankan **nginx2** expose port `8081`.
5. Display container (activate/exit).
6. Check output `curl` di kedua port.
7. Akses container **nginx2** (`exec`).
8. Update & install editor (nano) di dalam container.
9. Edit `index.html` menjadi "hello, username" dan pindahkan ke default directory nginx.
10. Restart service nginx di dalam container.
11. Rerun (start) container **nginx2**.
12. Display container, test `curl` lagi.
13. Display inspect, logs, stats, top untuk `nginx1` & `nginx2`.

### ✅ Solusi

```bash
# 1. Search image nginx
$ docker search nginx
$ skopeo list-tags docker://registry.adinusa.id/btacademy/nginx

# 2. Running nginx1 expose port 8080
$ docker run -d --name nginx1 -p 8080:80 registry.adinusa.id/btacademy/nginx:latest

# 3. Inspect nginx1
$ docker inspect nginx1

# 4. Running nginx2 expose port 8081
$ docker run -d --name nginx2 -p 8081:80 registry.adinusa.id/btacademy/nginx:latest

# 5. Display container
$ docker ps -a
$ docker container ls -a

# 6. Check nginx output
$ curl localhost:8080
$ curl localhost:8081

# 7. Accessing the container
$ docker exec -it nginx2 /bin/bash

# ===== (Mulai eksekusi di DALAM container nginx2) =====
# 8. Update and install editor
$ apt-get update -y && apt-get install nano -y

# 9. Edit index.html
$ nano index.html
# Isi dengan teks: hello, username
# Simpan (Ctrl+O, Enter, Ctrl+X)

# Move index.html to default nginx document root
$ mv index.html /usr/share/nginx/html

# 10. Restart service nginx
$ service nginx restart

# Keluar dari container
$ exit
# =====================================================

# 11. Rerun the container (jika sempat mati)
$ docker start nginx2

# 12. Display container & Check output lagi
$ docker ps
$ curl localhost:8080
$ curl localhost:8081

# 13. Inspect, logs, stats, top
$ docker inspect nginx1
$ docker inspect nginx2
$ docker logs nginx1
$ docker logs nginx2
$ docker stats nginx1
$ docker stats nginx2
$ docker top nginx1
$ docker top nginx2
```

### ⚠️ Urutan Sangat Penting

```
1. docker exec -it nginx2 /bin/bash   → masuk ke container
2. apt-get update & install           → siapkan tools
3. nano & mv                          → ubah web content
4. service nginx restart              → apply perubahan
5. exit                               → keluar container
6. curl localhost:8081                → verifikasi hasil
```

**Kenapa?** `docker exec` membuat sesi *shell baru* di dalam container tanpa menghentikan proses utama (NGINX). Jika kita menggunakan `docker attach`, kita akan menempel pada proses NGINX dan tidak bisa menjalankan command bash seperti `apt-get`. Pastikan keluar dengan `exit` (bukan `Ctrl+C`) agar container tetap running.

### 🔍 Penjelasan Opsi Penting

**`docker exec -it nginx2 /bin/bash`**
- Menjalankan shell bash di dalam container `nginx2` yang sedang running.
- `-i` (interactive) = jaga STDIN tetap terbuka.
- `-t` (TTY) = alokasikan pseudo-TTY (kayak terminal beneran).

**`apt-get update -y && apt-get install nano -y`**
- `-y` = otomatis jawab "yes" pada prompt instalasi.
- `&&` = jalankan perintah kedua jika perintah pertama sukses.

**`service nginx restart`**
- Me-restart daemon NGINX di dalam container agar membaca file `index.html` yang baru.

**`docker start nginx2`**
- Jika container sempat berhenti (Exited), perintah ini menyalakannya kembali tanpa membuat container baru (beda dengan `docker run`).

---

## 🧪 Practice 04 — Ubuntu Container & Ephemeral Mode

### Soal

1. Cari image **ubuntu** dari DockerHub & Harbor.
2. Pull image ubuntu dari Harbor.
3. Running ubuntu container dan akses console (beri nama **ubuntu1**). Lalu keluar.
4. Display container.
5. Run ubuntu container dengan flag **delete when exiting** (beri nama **ubuntu2**). Lalu keluar.
6. Display container.

### ✅ Solusi

```bash
# 1. Search image ubuntu
$ docker search ubuntu
$ skopeo list-tags docker://registry.adinusa.id/btacademy/ubuntu

# 2. Pull image ubuntu
$ docker pull registry.adinusa.id/btacademy/ubuntu

# 3. Running ubuntu container & access console
$ docker run -it --name ubuntu1 registry.adinusa.id/btacademy/ubuntu
# Keluar dengan mengetik: exit (atau Ctrl+D)
$ docker ps -a

# 4. Run ubuntu container & delete it when exiting (--rm)
$ docker run -it --rm --name ubuntu2 registry.adinusa.id/btacademy/ubuntu
# Keluar dengan mengetik: exit (atau Ctrl+D)

# 5. Display container (ubuntu2 tidak akan muncul)
$ docker ps -a
```

### ⚠️ Urutan Sangat Penting

```
1. docker pull         → pastikan image ada di lokal
2. docker run -it      → masuk ke console
3. exit                → keluar (container jadi Exited)
4. docker run -it --rm → masuk console mode ephemeral
5. exit                → keluar (container langsung DIHAPUS)
```

**Kenapa?** `ubuntu1` akan berstatus `Exited (0)` di `docker ps -a`. Sedangkan `ubuntu2` karena menggunakan flag `--rm`, saat kita mengetik `exit`, container langsung dihancurkan dan **tidak akan muncul** di `docker ps -a`. Ini membuktikan konsep *ephemeral container*.

### 🔍 Penjelasan Opsi Penting

**`docker run -it --name ubuntu1 <img>`**
- Menjalankan container interaktif dengan nama `ubuntu1`.
- Tanpa command tambahan (seperti `bash`), Docker akan menjalankan default command dari image Ubuntu, yaitu `bash`.

**`docker run -it --rm --name ubuntu2 <img>`**
- `--rm` = *Remove*. Menghapus container secara otomatis saat proses utamanya selesai/exit.
- Sangat berguna untuk *testing* agar tidak perlu repot menjalankan `docker rm` secara manual.

---

## 🔍 Verifikasi

### 1. Cek web NGINX (Practice 03)

```bash
$ curl localhost:8080
$ curl localhost:8081
```
Output untuk `8080` harusnya masih default NGINX (`Welcome to nginx!`), sedangkan `8081` harusnya menampilkan teks **`hello, username`**.

### 2. Cek status container NGINX

```bash
$ docker ps
```
Output: `nginx1` dan `nginx2` berstatus `Up`.

### 3. Cek status container Ubuntu (Practice 04)

```bash
$ docker ps -a | grep ubuntu
```
Output: Hanya `ubuntu1` yang muncul dengan status `Exited (0)`. `ubuntu2` **tidak ada** karena sudah dihapus oleh flag `--rm`.

### 4. Grading

```bash
$ nusactl grade do-002-3
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah Nyata yang Dialami

- **`nano` atau `mv` tidak ditemukan (command not found):**

  **Penyebab:** Image `nginx:alpine` atau `nginx:latest` dari Adinusa mungkin tidak memiliki `nano` secara bawaan.

  **Solusi:** Pastikan sudah menjalankan `apt-get update && apt-get install nano -y` sebelum memanggil `nano`. Jika masih gagal, gunakan `vi` atau `echo "hello, username" > index.html`.

- **Mengedit file tapi web tidak berubah:**

  **Penyebab:** File disimpan di direktori yang salah, atau service NGINX belum di-restart.

  **Solusi:** Pastikan file dipindah ke `/usr/share/nginx/html` dan jalankan `service nginx restart` di dalam container.

- **Container mati setelah `docker exec`:**

  **Penyebab:** Keluar dari container menggunakan `Ctrl+C`, atau menjalankan `kill` pada PID 1.

  **Solusi:** Selalu gunakan perintah `exit` untuk keluar dari sesi `exec` atau `bash`.

- **`docker run` ubuntu langsung exit:**

  **Penyebab:** Lupa menggunakan flag `-it`. Tanpa `-it`, Ubuntu tidak punya proses berjalan di background sehingga langsung exit.

  **Solusi:** Selalu gunakan `docker run -it` untuk container OS murni seperti Ubuntu/Alpine.

### Kesalahan Umum Lainnya

- **`docker exec` ke container yang mati:** `exec` hanya bisa dilakukan pada container yang statusnya `Up`. Jika mati, gunakan `docker start` dulu.
- **Port conflict:** Jika port `8080` sudah dipakai, gunakan port lain seperti `-p 8090:80`.
- **Bingung `docker run` vs `docker start`:** `run` membuat container baru dari image. `start` hanya menyalakan container lama yang sudah ada.

---

## 💡 Catatan & Insight Pribadi

### Konsep Writable Layer

Saat kita menginstall `nano` dan mengedit `index.html` di dalam container `nginx2`, perubahan itu **tidak merusak image aslinya**. Perubahan tersebut tersimpan di **writable layer** milik container `nginx2`. Jika kita menghapus container `nginx2` dan menjalankan image `registry.adinusa.id/btacademy/nginx` lagi, isinya akan kembali kosong/default.

### Urutan Eksekusi di Dalam vs Luar Container

Penting untuk membedakan di mana kita mengetik perintah:

- `docker exec -it nginx2 /bin/bash` → Dieksekusi di **Host**.
- `apt-get update`, `nano`, `mv`, `service nginx restart` → Dieksekusi di **Dalam Container**.
- `exit` → Dieksekusi di **Dalam Container** (untuk kembali ke Host).
- `docker ps -a`, `curl localhost:8081` → Dieksekusi di **Host**.

### Ephemeral Container (`--rm`)

Container yang berjalan dengan `--rm` tidak akan meninggalkan sisa di *disk* setelah berhenti. Sangat berguna untuk:

- Menjalankan script *one-off* (misal: `docker run --rm alpine ping google.com`).
- Melakukan *testing* sementara tanpa mengotori `docker ps -a`.
- Menjalankan *build process*.

### Real-World Use Cases

- **Debugging aplikasi web:** `docker exec -it <web-container> bash` → masuk, cek log, cek environment, coba curl internal.
- **Migrasi database:** `docker run --rm -v dbdata:/data postgres:15 pg_dump ...` → container langsung hilang setelah selesai.
- **Mengubah konfigurasi sementara:** Edit file di dalam container via `exec` (meskipun *best practice*-nya adalah *rebuild image*).

### Pelajaran Kunci dari Lab Ini

1. **`docker exec`** = masuk ke container tanpa mematikan proses utama.
2. **`apt-get`** = install package di dalam container (hanya di writable layer).
3. **`service nginx restart`** = apply perubahan file web.
4. **`exit`** = keluar dengan aman, bukan `Ctrl+C`.
5. **`--rm`** = konsep ephemeral container, auto-hapus saat exit.
6. **`-it`** = wajib untuk container yang butuh interaksi shell.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `docker run -d --name X -p H:C <img>` | Jalankan container background + port mapping |
| `docker run -it --name X <img>` | Jalankan container interaktif + nama |
| `docker run -it --rm --name X <img>` | Interaktif + auto-hapus saat exit |
| `docker exec -it <c> /bin/bash` | Masuk ke shell container yang running |
| `docker start <c>` | Nyalakan container yang sudah Exited |
| `docker inspect <c>` | Detail container (JSON) |
| `docker logs <c>` | Lihat log output container |
| `docker stats <c>` | Monitoring resource (CPU/RAM) |
| `docker top <c>` | Lihat proses di dalam container |
| `apt-get update && apt-get install <pkg>` | Install software di dalam container |
| `service nginx restart` | Restart web server di dalam container |

**File & konsep penting:**

| Konsep | Keterangan |
|---|---|
| Writable Layer | Lapisan yang bisa diubah di atas image (hilang jika container dihapus) |
| Ephemeral | Container yang tidak permanen (`--rm`) |
| PID 1 | Proses utama container (jika mati, container mati) |
| `-it` | Interactive + TTY (untuk masuk ke shell) |

**Urutan perintah lengkap (final):**

```bash
# ===== Practice 03: NGINX =====
docker run -d --name nginx1 -p 8080:80 registry.adinusa.id/btacademy/nginx:latest
docker run -d --name nginx2 -p 8081:80 registry.adinusa.id/btacademy/nginx:latest

# Masuk ke nginx2
docker exec -it nginx2 /bin/bash
apt-get update -y && apt-get install nano -y
nano index.html
# (ketik: hello, username) simpan & keluar
mv index.html /usr/share/nginx/html
service nginx restart
exit

# Verifikasi
curl localhost:8080
curl localhost:8081

# Monitoring
docker ps -a
docker inspect nginx1
docker logs nginx2
docker stats nginx1
docker top nginx2

# ===== Practice 04: Ubuntu =====
docker pull registry.adinusa.id/btacademy/ubuntu

# Normal container
docker run -it --name ubuntu1 registry.adinusa.id/btacademy/ubuntu
exit
docker ps -a  # ubuntu1 muncul (Exited)

# Ephemeral container
docker run -it --rm --name ubuntu2 registry.adinusa.id/btacademy/ubuntu
exit
docker ps -a  # ubuntu2 tidak muncul
```

---

## 📌 Kesimpulan

Lab 2.3 ini menguji kemampuan **interaksi lanjutan** dengan container Docker — bukan hanya menjalankan, tapi masuk ke dalamnya, memodifikasi state, dan memahami siklus hidup interaktif.

Yang diuji:
- **`docker exec`** untuk masuk ke dalam container tanpa menghentikan service.
- **Package management di dalam container** (`apt-get`).
- **Modifikasi file & restart service** di dalam container.
- **`docker start`** untuk menyalakan ulang container yang mati.
- **Ephemeral container** (`--rm`) untuk use-case sementara.

Dua pelajaran paling berharga dari lab ini:
1. **Writable layer** — perubahan di `exec` itu sementara dan hanya hidup selama container-nya belum dihapus. (Kalau mau permanen, ubah di `Dockerfile`).
2. **`--rm` itu powerful** — untuk *testing* dan *one-off task*, ini menyelamatkan kita dari penumpukan container `Exited` yang memakan disk.

**Motto Docker:** *Masuk, ubah, uji, dan bersihkan tanpa jejak.*

> ⚠️ **Disclaimer:** Catatan ini ditulis ulang berdasarkan pemahaman pribadi dari lab Adinusa. Materi asli tidak didistribusikan.
