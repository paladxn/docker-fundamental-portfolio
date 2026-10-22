# 🐳 DOCKER FUNDAMENTAL — COMPLETE NOTES

**Course:** Docker Fundamental (Adinusa)
**Author:** [@paladxn](https://github.com/paladxn)
**Status:** 🚀 In Progress / Completed

> Dokumen ini adalah rangkuman komprehensif (Satu File untuk Semua) dari seluruh kurikulum Docker Fundamental Adinusa. Dibuat sebagai *self-reminder*, portofolio, dan panduan praktis.

---

# 1. Introduction of Container
**Konsep:** Container adalah cara mengemas aplikasi beserta dependensinya dalam satu unit standar (Standardized Black Box). 
- **Perbandingan VM vs Container:** VM memakan banyak resource karena membawa *Full OS*. Container hanya membawa *library* & *binary* aplikasi, berbagi *Kernel* dengan Host OS. (Ukuran VM: GB, Container: MB).
- **Eksekusi:** Container berjalan sebagai proses terisolasi di Host OS menggunakan fitur Linux Kernel: `Namespaces` (isolasi) dan `Cgroups` (limit resource).

# 2. Introduction to Docker
**Konsep:** Docker adalah platform open-source untuk *Build, Ship, and Run* aplikasi dalam container.
- **Arsitektur Docker:** `Docker Client` (CLI) ➡️ `Docker Daemon` (`dockerd`) ➡️ `Container Runtime` (`containerd`) ➡️ `runc`.
- **Registry:** Tempat menyimpan image. Default-nya adalah Docker Hub (Public). Bisa juga Private Registry (seperti Harbor).
- **Instalasi (Ubuntu):** 
  *(Lihat Lab 1 - Instalasi Docker Engine via repository resmi).*
  ```bash
  # Cek versi & info setelah install
  docker version
  docker info
  # Tambah user ke group docker agar tidak perlu sudo
  sudo usermod -aG docker $USER
  ```

# 3. Managing Docker Container
**Konsep:** Container memiliki *Lifecycle*: `Create` ➡️ `Start` ➡️ `Running` ➡️ `Paused` ➡️ `Exited` ➡️ `Removed`.
- **Run Container:** `docker run -d --name <nama> -p 80:80 <image>`
- **Eksekusi di dalam:** `docker exec -it <nama> bash` (membuka shell baru tanpa mematikan container).
- **Monitoring:** `docker logs`, `docker stats`, `docker top`, `docker inspect`.
- **Resource Limit:** `--memory 256m`, `--cpu-shares 512`, `--cap-drop net_raw` (mencabut hak akses jaringan).
- **Stop vs Kill:** `stop` (graceful SIGTERM), `kill` (paksa SIGKILL).

# 4. Creating Custom Docker Container Image
**Konsep:** Image terdiri dari tumpukan *Layer* (read-only). Saat container jalan, ada *Writable Layer* di paling atas. Jika container dihapus, writable layer hilang.
- **Dockerfile:** Resep untuk membuat image.
  - `FROM`: Base image.
  - `RUN`: Eksekusi saat *build* (install package).
  - `COPY` / `ADD`: Salin file dari Host ke Image.
  - `WORKDIR`: Set direktori kerja.
  - `CMD` / `ENTRYPOINT`: Perintah saat container *run*.
- **Best Practice:** Gabungkan perintah `RUN` dengan `&&` agar tidak menumpuk layer. Gunakan base image yang ringan (seperti Alpine).
- **Registry Push:**
  ```bash
  docker tag <image-lokal> <username-dockerhub>/<repo>:<tag>
  docker push <username-dockerhub>/<repo>:<tag>
  ```

# 5. Docker Compose
**Konsep:** Alat untuk mendefinisikan dan menjalankan aplikasi Docker *multi-container*. Cukup tulis satu file `docker-compose.yml` (atau `compose.yaml`), lalu jalankan satu perintah.
- **Struktur File:**
  ```yaml
  version: '3.8'
  services:
    web:
      image: nginx:alpine
      ports:
        - "8080:80"
      depends_on:
        - db
    db:
      image: mysql:5.7
      environment:
        MYSQL_ROOT_PASSWORD: rahasia
      volumes:
        - db_data:/var/lib/mysql
  volumes:
    db_data:
  ```
- **Perintah Utama:**
  - `docker compose up -d`: Jalankan semua service di background.
  - `docker compose down`: Hentikan dan hapus container.
  - `docker compose logs -f`: Lihat log semua service.
  - `docker compose ps`: Cek status service.

# 6. Docker Continuous Integration (CI)
**Konsep:** Integrasi Docker ke dalam *pipeline* CI/CD (seperti GitHub Actions, GitLab CI, Jenkins).
- **Alur CI:** Push code ke Git ➡️ CI Server mendeteksi ➡️ Build Docker Image ➡️ Test Image ➡️ Push ke Registry ➡️ Deploy ke Server.
- **GitHub Actions Example:**
  ```yaml
  name: Docker Build and Push
  on: [push]
  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
        - name: Login to Docker Hub
          uses: docker/login-action@v2
          with:
            username: ${{ secrets.DOCKER_USER }}
            password: ${{ secrets.DOCKER_PASS }}
        - name: Build and Push
          uses: docker/build-push-action@v4
          with:
            push: true
            tags: user/app:latest
  ```

# 7. Docker Swarm & Portainer
**Konsep:**
- **Docker Swarm:** Alat *orchestration* bawaan Docker (mirip Kubernetes tapi lebih sederhana). Menggabungkan banyak Docker Host menjadi satu *Cluster*.
  - `docker swarm init`: Inisiasi node Master.
  - `docker service create`: Deploy aplikasi di Swarm.
- **Portainer:** GUI (Graphical User Interface) berbasis web untuk mengelola Docker (container, image, volume, network) tanpa harus mengetik CLI.
  - Install: `docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer-ce`

# 8. Logging Driver
**Konsep:** Docker secara default menyimpan log container dalam file JSON di host (`json-file`). Kita bisa mengubah *driver* agar log dikirim ke sistem terpusat (seperti `syslog`, `fluentd`, `journald`, `awslogs`).
- **Cek driver saat ini:** `docker info | grep "Logging Driver"`
- **Jalankan dengan driver spesifik:**
  ```bash
  docker run --log-driver=syslog --log-opt syslog-address=udp://1.2.3.4:1111 nginx
  ```
- **Batasi ukuran log (Best Practice):**
  Di `/etc/docker/daemon.json`:
  ```json
  {
    "log-driver": "json-file",
    "log-opts": { "max-size": "10m", "max-file": "3" }
  }
  ```

# 9. Health Check
**Konsep:** Docker hanya tahu container *running* atau *stopped*. Docker tidak tahu apakah aplikasi di dalamnya *hang* atau error 500. Health Check mengecek secara berkala apakah aplikasi benar-benar sehat.
- **Di Dockerfile:**
  ```dockerfile
  HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost/ || exit 1
  ```
- **Di Docker Compose:**
  ```yaml
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost"]
    interval: 1m30s
    timeout: 10s
    retries: 3
  ```
- **Cek status:** `docker ps` (kolom STATUS akan menampilkan `(healthy)` atau `(unhealthy)`).

# 10. Security
**Konsep:** Mengamankan container dari serangan.
- **Gunakan Non-Root User:** Di Dockerfile, tambahkan `USER appuser` (jangan biarkan aplikasi berjalan sebagai root).
- **Read-Only Filesystem:** `docker run --read-only <image>`.
- **Cap Drop:** Mencabut hak akses kernel tertentu. `docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE`.
- **Secrets Management:** Jangan simpan password di Env Variable/Dockerfile. Gunakan Docker Secrets (khusus Swarm) atau mount file rahasia secara aman.
- **Scanning:** Gunakan alat seperti `Trivy` atau `docker scan` untuk mencari celah keamanan (CVE) di dalam image.

# 11. Storage Driver
**Konsep:** Driver yang mengatur bagaimana *Layer* image dan *Writable Layer* container disimpan di filesystem Host.
- **Macam-macam:** `overlay2` (default & tercepat di Linux modern), `aufs` (lama), `btrfs`, `zfs`, `devicemapper`.
- **Cek driver:** `docker info | grep "Storage Driver"`.
- **Kenapa penting?** Jika storage driver bermasalah atau penuh, Docker tidak bisa membuat container baru atau memodifikasi file.

# 12. Logging and Error Handling
**Konsep:** Cara menangani error saat container mati.
- **Exit Codes:**
  - `0`: Sukses.
  - `1`: Error aplikasi.
  - `137`: OOM (Out Of Memory) dibunuh oleh kernel / SIGKILL.
  - `143`: Terminasi SIGTERM (biasanya karena `docker stop`).
- **Restart Policy:**
  ```bash
  docker run --restart=unless-stopped nginx
  # Opsi: no (default), on-failure, always, unless-stopped
  ```
- **Error Handling:** Selalu gunakan `docker logs <container>` dan `docker inspect <container> | grep -i error` untuk debugging.

# 13. Lab Challenge
**Kumpulan Soal & Solusi Lab:**
1. **Quiz Web Statis (Nginx):**
   Menggunakan `COPY` untuk memindahkan file HTML ke `/usr/share/nginx/html` di dalam image Nginx.
2. **Volume SSHFS (Remote Storage):**
   Menginstall plugin `vieux/sshfs`, melakukan `disable` -> `set sshkey` -> `enable` -> buat volume -> mount ke container.
3. **Troubleshooting `i/o timeout` Registry:**
   Dilakukan dengan melakukan `docker pull` manual terlebih dahulu atau restart Daemon (`systemctl restart docker`).
4. **Error 404 `apt-get` saat Build:**
   Mengubah base image di Dockerfile (misal dari `ubuntu:16.04` ke `ubuntu:22.04` atau `node:18.13.0` ke `node:18`).

---
*Disclaimer: Catatan ini adalah rangkuman pribadi berdasarkan modul dan lab dari Adinusa Docker Fundamental.*
