# Docker Fundamental — Custom Docker Images & Dockerfile

**Course:** Docker Fundamental (Adinusa)
**Topic:** Docker Images, Registry, Dockerfile (FROM, RUN, COPY, CMD), Push to Docker Hub
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan modul ini, saya mampu:

- Memahami konsep **Image Layer**, **Writable Layer**, dan efisiensi penyimpanan Docker.
- Membedakan **Docker Hub** (Public) dan **Docker Registry** (Private).
- Melakukan manipulasi image (`save`, `load`, `tag`, `push`).
- Menulis **Dockerfile** dengan instruksi lengkap (`FROM`, `RUN`, `COPY`, `WORKDIR`, `ENTRYPOINT`, `CMD`).
- Membangun image kustom (Node.js, Whalesay, Flask, Web Statis) dan mempublikasikannya ke Docker Hub.

---

## 📘 Konsep Dasar

### 1. Docker Images & Layered Filesystem
Image Docker bukanlah satu file besar, melainkan tumpukan **layer** (lapisan) yang bersifat *immutable* (tidak bisa diubah). 
- **Base Layer**: Lapisan paling bawah (misal: Alpine Linux, Ubuntu).
- **Layer Tambahan**: Setiap instruksi `RUN` atau `COPY` di Dockerfile akan membuat layer baru yang hanya berisi *delta* (perubahan) dari layer di bawahnya.
- **Writable Layer**: Saat container dijalankan, Docker menambahkan satu lapisan *Read-Write* di paling atas. Jika container dihapus, lapisan ini hilang, tapi layer *image* di bawahnya tetap aman (bisa di-*share* ke container lain).

### 2. Docker Registry vs Docker Hub
- **Docker Hub**: Registry publik default. Memiliki batasan *rate limit* (100 pull/6 jam untuk anonim).
- **Docker Registry**: Aplikasi open-source yang bisa di-*host* sendiri (on-premise) untuk menyimpan image secara privat tanpa batasan rate limit.

### 3. Manipulasi Image
- `docker save` & `docker load`: Menyimpan image ke file `.tar` dan memuatnya kembali (berguna untuk transfer offline).
- `docker tag`: Memberi nama/alias baru pada image (wajib sebelum push).
- `docker push`: Mengunggah image ke registry.
- `docker rmi $(docker images -q)`: Menghapus semua image yang tidak terpakai oleh container.

### 4. Instruksi Dockerfile
| Instruksi | Fungsi |
|---|---|
| `FROM` | Menentukan base image (contoh: `alpine:latest`). |
| `LABEL` | Menambahkan metadata (maintainer, deskripsi). |
| `RUN` | Menjalankan perintah saat *build* (install package). |
| `COPY` | Menyalin file dari host ke dalam image. |
| `ADD` | Mirip COPY, tapi bisa mengekstrak `.tar` atau download URL. |
| `WORKDIR` | Menentukan direktori kerja di dalam container. |
| `EXPOSE` | Dokumentasi port yang akan dibuka (tidak mem-bind port ke host). |
| `ENV` | Set environment variable. |
| `ENTRYPOINT` | Perintah utama yang dieksekusi saat container jalan. |
| `CMD` | Argumen default untuk ENTRYPOINT, atau perintah default jika ENTRYPOINT kosong. |

---

## 🔧 Persiapan Lab

```bash
$ nusactl login
$ nusactl start do-004-1  # Untuk Lab 4.1
$ nusactl start do-004-2  # Untuk Lab 4.2
$ nusactl start do-004-3  # Untuk Lab 4.3
```
*(⚠️ **PENTING:** Eksekusi di dalam node `pod-[username]-node01`)*

---

## 🧪 Lab 4.1 — Exploring Dockerfile (Node & Whalesay)

### Soal
1. Clone repo `docker-node-hello`, build image `node-latihan01`, jalankan di port 8080.
2. Buat Dockerfile `whalesay` (install `fortunes`), build image `docker-whale`, jalankan.

### ✅ Solusi & Perbaikan Error

```bash
# ===== Practice 01: Node.js =====
$ cd $HOME
$ git clone https://github.com/spkane/docker-node-hello.git --config core.autocrlf=input latihan01
$ cd latihan01

# ⚠️ PERBAIKAN: Edit Dockerfile (ubah FROM node:18.13.0 menjadi node:18 agar tidak 404)
$ nano Dockerfile

$ docker build -t node-latihan01 .
$ docker run -d --rm --name node-latihan01 -p 8080:8080 node-latihan01
$ curl localhost:8080

# ===== Practice 02: Whalesay =====
$ cd $HOME && mkdir latihan02 && cd latihan02
$ vim Dockerfile
```

Isi `Dockerfile` Whalesay:
```dockerfile
FROM registry.adinusa.id/btacademy/whalesay:latest
RUN apt-get update && apt-get install -y fortunes
CMD /usr/games/fortune -a | cowsay
```

```bash
# ⚠️ PERBAIKAN: Jika timeout, pull manual dulu
$ docker pull registry.adinusa.id/btacademy/whalesay:latest
$ docker build -t docker-whale .
$ docker run docker-whale
```

---

## 🧪 Lab 4.2 — Exploring Dockerfile (Flask Apps & Push Docker Hub)

### Soal
1. Login ke Docker Hub.
2. Buat aplikasi Flask (`app.py`), `requirements.txt`, dan `Dockerfile`.
3. Build image `flask-latihan03`, tag dengan username Docker Hub, lalu push.
4. Jalankan container di port 5000.

### ✅ Solusi & Eksekusi

```bash
# 1. Login ke Docker Hub (Gunakan Access Token jika 2FA aktif)
$ docker login -u <username_dockerhub>

# 2. Setup Aplikasi Flask
$ cd $HOME && mkdir latihan03 && cd latihan03
$ vim app.py
```

Isi `app.py`:
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hey, we have Flask in a Docker container!'

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

Isi `requirements.txt`:
```text
Flask==0.10.1
Werkzeug==1.0.0
Jinja2==2.8.1
MarkupSafe==1.1.1
itsdangerous==1.1.0
```

Isi `Dockerfile`:
```dockerfile
FROM registry.adinusa.id/btacademy/ubuntu:16.04
RUN mkdir /app
RUN apt-get update -y && \
    apt-get install python-pip python-dev -y

COPY ./requirements.txt /app
COPY ./app.py /app

WORKDIR /app
RUN pip install -r requirements.txt

ENTRYPOINT ["python"]
CMD ["app.py"]
```

```bash
# 3. Build & Test
$ docker build -t flask-latihan03 .

# 4. Tag & Push ke Docker Hub (Ganti <username_dockerhub>)
$ docker tag flask-latihan03 <username_dockerhub>/flask-latihan03:latest
$ docker push <username_dockerhub>/flask-latihan03:latest

# 5. Jalankan Container
$ docker run -d -p 5000:5000 --name flask03 <username_dockerhub>/flask-latihan03
$ curl localhost:5000
```

---

## 🧪 Lab 4.3 — Exploring Dockerfile (Quiz Web Statis)

### Soal
1. Buat direktori `latihan-quiz01`, clone repo `bta-adinusa/mywebsite`.
2. Buat Dockerfile menggunakan base image Nginx.
3. Build image `latihan-quiz01-image`, jalankan dengan nama `quiz01` di port 80.
4. Pastikan tampilan bukan halaman default Nginx.

### ✅ Solusi & Eksekusi

```bash
# 1. Clone Repo
$ cd $HOME && mkdir latihan-quiz01 && cd latihan-quiz01
$ git clone https://github.com/bta-adinusa/mywebsite.git

# 2. Buat Dockerfile
$ vim Dockerfile
```

Isi `Dockerfile`:
```dockerfile
FROM registry.adinusa.id/btacademy/nginx:latest
COPY mywebsite/ /usr/share/nginx/html/
```

```bash
# 3. Build Image
$ docker build -t latihan-quiz01-image .

# 4. Jalankan Container
$ docker run -d --name quiz01 -p 80:80 latihan-quiz01-image

# 5. Verifikasi
$ curl localhost:80
# (Pastikan yang muncul adalah HTML dari mywebsite, bukan "Welcome to nginx!")
```

---

## 🔍 Verifikasi

### 1. Cek Image
```bash
$ docker images
```
Pastikan image `node-latihan01`, `docker-whale`, `flask-latihan03`, `latihan-quiz01-image`, dan `<username_dockerhub>/flask-latihan03` ada.

### 2. Cek Container
```bash
$ docker ps
$ curl localhost:8080   # Node.js
$ curl localhost:5000   # Flask
$ curl localhost:80     # Web Statis Quiz
```

### 3. Grading
```bash
$ nusactl grade do-004-1
$ nusactl grade do-004-2
$ nusactl grade do-004-3
```

---

## 🐛 Troubleshooting & Kesalahan

### Masalah 1: `i/o timeout` pada registry (Network Timeout)
**Gejala:** 
```text
ERROR: failed to resolve source metadata for registry.adinusa.id/...: dial tcp ... i/o timeout
```
**Penyebab:** Koneksi terputus atau DNS node bermasalah.
**Solusi:** Lakukan `docker pull <nama_image>` secara manual terlebih dahulu. Jika masih gagal, restart service Docker (`sudo systemctl restart docker`) atau pastikan kamu berada di Node01.

### Masalah 2: Error `404 Not Found` saat `apt-get install`
**Gejala:** 
```text
E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```
**Penyebab:** Base image `node:18.13.0` atau `ubuntu:16.04` sudah berumur dan repositori paketnya tidak tersedia lagi.
**Solusi:** Ubah base image di Dockerfile ke versi yang lebih baru (misal `FROM node:18` atau `FROM ubuntu:22.04`). Jalankan build dengan `--no-cache` jika perlu.

### Masalah 3: `Could not open lock file /var/lib/dpkg/lock-frontend`
**Gejala:** Error permission denied saat menjalankan `apt install` di terminal.
**Penyebab:** Mencoba install package di VM Host tanpa `sudo`, atau salah mengira harus menginstall package di Host.
**Solusi:** Jangan install package di Host. Cukup deklarasikan `RUN apt-get install ...` di dalam Dockerfile. Docker akan mengeksekusinya di dalam container secara terisolasi.

### Masalah 4: `port is already allocated`
**Gejala:** Container gagal jalan karena port host sudah dipakai.
**Penyebab:** Port (misal 80 atau 5000) sedang digunakan oleh container lain.
**Solusi:** Hentikan container lain (`docker ps` -> `docker stop <id>`), atau ganti port host di `docker run` (misal `-p 8081:80`).

---

## 💡 Catatan & Insight Pribadi

### Efisiensi Layer (Layer Caching)
Docker sangat pintar meng-*cache* layer. Jika kita menulis Dockerfile seperti ini:
```dockerfile
COPY . /app
RUN pip install -r /app/requirements.txt
```
Setiap kali kita mengubah 1 baris kode di `app.py`, Docker akan menganggap layer `COPY` berubah, sehingga instruksi `RUN pip install` akan dijalankan ulang (memakan waktu).
**Best Practice:** Copy `requirements.txt` dulu, install, baru copy seluruh kode.
```dockerfile
COPY requirements.txt /app
RUN pip install -r /app/requirements.txt
COPY . /app
```

### Perbedaan `COPY` dan `ADD`
Meskipun mirip, `COPY` lebih disarankan untuk menyalin file lokal. `ADD` sebaiknya digunakan hanya jika kita ingin mengekstrak file `.tar` secara otomatis atau mengunduh dari URL.

### Memahami Build Context
Tanda titik `.` di akhir `docker build -t nama .` adalah **Build Context**. Artinya, Docker Daemon akan menerima seluruh file di direktori tersebut (termasuk file `.git`, `node_modules`, dll) sebelum build dimulai. Selalu gunakan `.dockerignore` untuk mengecualikan file besar agar build lebih cepat.

### Pelajaran Kunci
1. **Jangan install di Host**, install di Dockerfile (`RUN`).
2. **Layer Caching** bisa mempercepat build, tapi harus diatur urutannya.
3. **`docker tag`** wajib dilakukan sebelum `docker push` ke registry.
4. **`COPY`** adalah jembatan penyalur file dari host ke dalam image.

---

## 🧠 Perintah yang Dikuasai

| Perintah | Fungsi |
|---|---|
| `docker build -t <nama> .` | Build image dari Dockerfile |
| `docker tag <image> <user>/<repo>:tag` | Memberi alias/tag untuk registry |
| `docker push <user>/<repo>:tag` | Upload image ke Docker Hub |
| `docker save -o <file.tar> <image>` | Simpan image ke file tar (offline) |
| `docker load -i <file.tar>` | Load image dari file tar |
| `docker login -u <user>` | Login ke Docker Hub |
| `docker rmi $(docker images -q)` | Hapus semua image lokal (jika tidak dipakai) |

**Urutan perintah lengkap (final):**

```bash
# ===== Lab 4.1 =====
cd $HOME
git clone https://github.com/spkane/docker-node-hello.git --config core.autocrlf=input latihan01
cd latihan01
# (Edit Dockerfile: FROM node:18)
docker build -t node-latihan01 .
docker run -d --rm --name node-latihan01 -p 8080:8080 node-latihan01
curl localhost:8080

# ===== Lab 4.2 =====
docker login -u <username>
cd $HOME && mkdir latihan03 && cd latihan03
# (Buat app.py, requirements.txt, Dockerfile)
docker build -t flask-latihan03 .
docker tag flask-latihan03 <username>/flask-latihan03:latest
docker push <username>/flask-latihan03:latest
docker run -d -p 5000:5000 --name flask03 <username>/flask-latihan03
curl localhost:5000

# ===== Lab 4.3 =====
cd $HOME && mkdir latihan-quiz01 && cd latihan-quiz01
git clone https://github.com/bta-adinusa/mywebsite.git
# (Buat Dockerfile: FROM nginx + COPY)
docker build -t latihan-quiz01-image .
docker run -d --name quiz01 -p 80:80 latihan-quiz01-image
curl localhost:80
```

---

## 📌 Kesimpulan

Modul ini adalah inti dari pengembangan aplikasi dengan Docker. Kita tidak hanya belajar menggunakan image yang sudah ada, tetapi juga membangun image kita sendiri (Custom Image) menggunakan **Dockerfile**. 

Kemampuan untuk merakit layer, mengatur *Build Context*, melakukan *troubleshooting* pada dependensi yang usang, hingga mempublikasikan hasil karya ke **Docker Hub** adalah keterampilan yang sangat dicari di industri *DevOps* dan *Software Engineering*.

**Motto Docker:** *Rakit sekali, jalankan di mana saja.*

> ⚠️ **Disclaimer:** Catatan ini ditulis berdasarkan eksekusi lab dan pemahaman pribadi dari modul Adinusa. Materi asli tidak didistribusikan.
