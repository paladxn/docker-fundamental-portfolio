# 🐳 DOCKER FUNDAMENTAL — COMPLETE NOTES

**Course:** Docker Fundamental (Adinusa)
**Author:** [@paladxn](https://github.com/paladxn)
**Status:** 🚀 Completed (Modul 1 - 9)

> Dokumen ini adalah rangkuman komprehensif (Satu File untuk Semua) dari seluruh kurikulum Docker Fundamental Adinusa. Disusun sesuai urutan *Course Content* asli, dilengkapi dengan praktik lab, penjelasan command (agar paham betul), dan *troubleshooting* nyata.

---

# 📘 MODUL 1: Introduction of Container

### 1.1 Course Introduction & What Are Containers?
Container adalah cara standar untuk mengemas aplikasi beserta dependensinya agar bisa berjalan di mana saja (*Build, Ship, and Run Anywhere*).

### 1.2 Why are containers important?
- Mengatasi masalah "Works on my machine".
- Jauh lebih ringan dibanding Virtual Machine (VM).
- Mempercepat siklus rilis aplikasi (CI/CD).

### 1.3 Host, Virtual Machine and Container
- **VM:** Memiliki OS tamu (Guest OS) lengkap, sangat berat, booting lambat.
- **Container:** Berbagi Kernel dengan Host OS, sangat ringan, booting dalam milidetik.
- **Isolasi:** Container menggunakan Linux Namespaces dan Cgroups.

### 1.4 Lab 1.1: Preparation of Lab Environment
**Task:** Menyiapkan VirtualBox (NAT Network & Host-Only), import VM Ubuntu 22.04, set IP statis via Netplan, dan mengatur *passwordless SSH* antar Node01 dan Node02.

---

# 📘 MODUL 2: Introduction to Docker

### 2.1 Lab 2.1: Installing Docker
**Task:** Install Docker Engine di Ubuntu via repository resmi.

```bash
# Menambahkan GPG Key dan Repository resmi Docker
sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Ijinkan user menjalankan docker tanpa sudo
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock

# Test
docker run registry.adinusa.id/btacademy/hello-world
```
💡 **Catatan Pemahaman:**
- `usermod -aG docker $USER`: Memasukkan user aktif ke dalam grup `docker`. Ini agar kamu tidak perlu mengetik `sudo` terus-menerus setiap menjalankan perintah Docker.
- `chmod 666 /var/run/docker.sock`: Memberikan izin baca-tulis ke *socket* Docker (jalur komunikasi antara terminal CLI dan layanan Docker di *background*).

### 2.2 Lab 2.2: Docker Run - Part 1 (Redis & Nginx)
```bash
# Foreground (Terminal terblokir, tekan Ctrl+C untuk stop)
docker run registry.adinusa.id/btacademy/redis

# Background (Detached)
docker run -d --name redis1 registry.adinusa.id/btacademy/redis

# Port Mapping (Host:Container)
docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest

# Random Port (Docker pilih port host acak)
docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest
docker port nginx2 80
```
💡 **Catatan Pemahaman:**
- `-d` (Detach): Melepas container agar berjalan di *background* sehingga terminal bisa dipakai untuk mengetik perintah lain.
- `-p 80:80`: Formatnya `<Port_Host>:<Port_Container>`. Port 80 di laptop/VM-mu diteruskan ke port 80 di dalam container.
- `-p 80` (tanpa host port): Docker akan memilihkan port acak di host (misal 32768) untuk diteruskan ke port 80 container.

### 2.3 Lab 2.3: Docker Run - Part 2 (Exec & Ephemeral)
```bash
# Masuk ke container (interactive)
docker exec -it nginx2 /bin/bash

# Ephemeral Container (otomatis terhapus saat keluar)
docker run -it --rm --name ubuntu2 registry.adinusa.id/btacademy/ubuntu
```
💡 **Catatan Pemahaman:**
- `exec -it`: `-i` (interactive) dan `-t` (TTY). Membuka "pintu masuk" ke dalam container yang sedang berjalan *tanpa* mematikan aplikasi utamanya.
- `--rm`: *Remove*. Begitu container berhenti (misal kita mengetik `exit`), container langsung dihancurkan agar tidak menumpuk sampah di penyimpanan.

### 2.4 Lab 2.4: Docker Run - Part 3 (Environment, Pause, Limits)
```bash
# Environment Variables & Link
docker run -d --name my-mysql -e MYSQL_ROOT_PASSWORD=RAHASIA -e MYSQL_DATABASE=latihan05 -p 3306:3306 registry.adinusa.id/btacademy/mysql

# Pause (Membekukan proses agar hemat CPU)
docker pause ubuntu1
docker unpause ubuntu1

# Resource Limit
docker run -d --name ch6_mariadb --memory 256m --cpu-shares 1024 -e MYSQL_ROOT_PASSWORD=test registry.adinusa.id/btacademy/mariadb:5.5
```
💡 **Catatan Pemahaman:**
- `-e`: *Environment variable*. Digunakan untuk mengonfigurasi aplikasi di dalam container tanpa harus mengubah kode (misal: set password root MySQL).
- `--memory`: Batas RAM maksimal. Jika aplikasi di dalamnya mencoba memakai RAM melebihi batas, sistem akan mematikannya (OOM Killer).
- `--cpu-shares`: Bobot prioritas CPU (bukan batas maksimal), menentukan siapa yang dapat jatah CPU lebih banyak jika server sedang sibuk.

---

# 📘 MODUL 3: Managing Docker Container

### 3.1 Life Cycle & Registry
- **Public Registry:** Docker Hub (ada limit pull).
- **Private Registry:** Harbor (bebas limit, lebih aman untuk perusahaan).

### 3.2 Lab 3.1: Mount Volume (Local)
```bash
docker create -v /my-volume --name my-volume registry.adinusa.id/btacademy/busybox
docker cp . my-volume:/my-volume
docker run --volumes-from my-volume registry.adinusa.id/btacademy/ubuntu ls /my-volume
```
💡 **Catatan Pemahaman:**
- `docker create`: Membuat container tapi tidak menjalankannya (hanya disiapkan).
- `docker cp`: Menyalin file dari VM Host ke dalam container (bahkan bisa saat container dalam keadaan mati).
- `--volumes-from`: Meminjam volume dari container lain yang sudah ada.

### 3.3 Lab 3.2: Mount Volume with NFS Server
```bash
# Di Node01 (Server)
docker run -itd --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v /data/nfs-storage01:/data -p 2049:2049 registry.adinusa.id/btacademy/nfs-server-alpine:12

# Di Node02 (Client)
sudo mount -v -t nfs4 10.7.7.10:/ /mnt
```
💡 **Catatan Pemahaman:**
- `--privileged`: Memberikan akses *super-admin* (root penuh) ke container. Diperlukan karena NFS Server butuh memodifikasi konfigurasi kernel untuk melayani sharing file.
- `-t nfs4`: Menentukan tipe *filesystem* (NFS versi 4) saat melakukan mount dari remote server.

### 3.4 Lab 3.3: Mount Volume with Read-only Mode
```bash
docker run -d --name=nginx-rwvol -v my-vol:/usr/share/nginx/html -p 4005:80 registry.adinusa.id/btacademy/nginx:latest

# Mount Read-Only (Container tidak bisa menulis/mengubah file)
docker run -d --name=nginx-rovol -v my-vol:/usr/share/nginx/html:ro registry.adinusa.id/btacademy/nginx:latest
```
💡 **Catatan Pemahaman:**
- `:ro` (*Read-Only*): Sangat penting untuk keamanan (Security). Jika web server di-hack, penyerang tidak bisa menaruh file berbahaya (webshell) karena folder web-nya dikunci *Read-Only*.

### 3.5 Lab 3.4: Volume Driver (SSHFS)
```bash
docker plugin install --grant-all-permissions vieux/sshfs
docker plugin disable b8275dd4fb5c
docker plugin set vieux/sshfs sshkey.source=/home/student/.ssh/
docker plugin enable b8275dd4fb5c

docker volume create --driver vieux/sshfs -o sshcmd=student@10.7.7.20:/share -o allow_other sshvolume
```
💡 **Catatan Pemahaman:**
- `docker plugin disable -> set -> enable`: Urutan ini wajib. Kita tidak bisa mengubah konfigurasi (set SSH key) jika plugin sedang aktif.
- `-o sshcmd=...`: Perintah yang dikirim plugin untuk melakukan SSH ke Node02 dan mengambil direktori `/share`.

### 3.6 Lab 3.5 & 3.6: Default Bridge vs Host Network
```bash
# Default Bridge (Ping via nama GAGAL)
docker exec -it alpine1 ping alpine2

# Custom Bridge (Ping via nama BERHASIL)
docker network create --driver bridge bridge1
docker network connect bridge1 alpine1
docker network connect bridge1 alpine2
docker exec -it alpine1 ping alpine2

# Host Network
docker run -itd --network host --name my_nginx registry.adinusa.id/btacademy/nginx
```
💡 **Catatan Pemahaman:**
- `--network host`: Container meminjam jaringan host secara langsung. IP container = IP Host. Flag `-p` tidak berguna di mode ini.

---

# 📘 MODUL 4: Creating Custom Docker Container Image

### 4.1 Teori Dockerfile
- Setiap instruksi (`RUN`, `COPY`) membuat 1 *Layer*.
- **Best Practice:** Taruh file yang jarang berubah di atas (seperti `package.json` atau `requirements.txt`), install dependensi, baru copy source code. Ini memanfaatkan *Layer Caching* agar *build* berikutnya cepat.

### 4.2 Lab 4.1 & 4.2: Exploring Dockerfile
```bash
docker build -t node-latihan01 .
docker run -d --rm --name node-latihan01 -p 8080:8080 node-latihan01

# Push ke Docker Hub
docker tag flask-latihan03 <username_dockerhub>/flask-latihan03:latest
docker push <username_dockerhub>/flask-latihan03:latest
```
💡 **Catatan Pemahaman:**
- `docker build -t <nama> .`: Tanda titik (`.`) di akhir berarti **Build Context**. Docker akan mengirim semua file di folder tersebut ke Docker Daemon untuk dijadikan image.
- `docker tag`: Memberi nama alias. Docker Hub mewajibkan format `<username>/<repo>:<tag>` agar kamu punya hak akses untuk mem-push image tersebut.

---

# 📘 MODUL 5: Docker Compose

### 5.1 Konsep Declarative
Dengan Compose, kita tidak mengetik `docker run` satu per satu. Kita tulis file YAML, dan Docker akan menyesuaikan state sistem dengan file tersebut.

### 5.2 Lab 5.1: WordPress & MySQL
```yaml
services:
  db:
    image: registry.adinusa.id/btacademy/mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
  wordpress:
    depends_on:
      - db
    image: registry.adinusa.id/btacademy/wordpress:latest
    ports:
      - "8000:80"
```
```bash
# Jalankan
docker compose up -d
```
💡 **Catatan Pemahaman:**
- `docker compose` (pakai spasi): Ini adalah Compose V2 (plugin). Jangan gunakan `docker-compose` (pakai strip) karena versi V1 sudah usang dan rentan error.
- `depends_on`: Memerintahkan Docker untuk menyalakan service `db` terlebih dahulu sebelum menyalakan `wordpress`.

---

# 📘 MODUL 6: Docker Continuous Integration (CI)

### 6.1 Alur CI
Developer `git push` -> GitHub (Webhook) -> Jenkins/GitHub Actions -> `docker build` -> `docker push` ke Registry -> `docker run` di Server Produksi.

### 6.2 Automated Build
Docker Hub bisa terintegrasi dengan GitHub. Setiap ada commit baru, Docker Hub otomatis melakukan *build* image tanpa kita perintah manual.

---

# 📘 MODUL 7: Docker Swarm & Portainer

### 7.1 Konsep Swarm
Swarm adalah alat *orchestration* (pengelola cluster) untuk menggabungkan banyak server fisik menjadi satu kesatuan.
- **Manager Node:** Otak cluster (mengatur penjadwalan).
- **Worker Node:** Otot cluster (menjalankan container).
- **Service:** Definisi aplikasi (bisa di-*scale* otomatis).

### 7.2 Lab 7.1: Inisialisasi Swarm & Join Node
```bash
# Di Node01 (Manager)
docker swarm init --advertise-addr 10.7.7.10

# Di Node02 (Worker)
docker swarm join --token <TOKEN> 10.7.7.10:2377

# Di Node01 (Verifikasi)
docker node ls
```
💡 **Catatan Pemahaman:**
- `--advertise-addr`: Memberi tahu node lain alamat IP mana yang harus mereka hubungi untuk berkomunikasi dengan Manager.
- `docker node ls`: Hanya bisa dijalankan di Manager. Berguna untuk melihat apakah Worker sudah terhubung.

### 7.3 Lab 7.2: Deploy Service & Resource Limits
```bash
# Deploy Nginx (dengan constraint jika Node02 bermasalah DNS)
docker service create --name web --replicas 2 --constraint 'node.role == manager' -p 80:80 registry.adinusa.id/btacademy/nginx:latest

# Resource Limit
docker service create --name arijfarid69 --reserve-cpu 1 --limit-cpu 1 --reserve-memory 256mb --limit-memory 128mb registry.adinusa.id/btacademy/httpd:latest
```
💡 **Catatan Pemahaman:**
- `--replicas 2`: Menyalakan 2 container dari image yang sama di dalam cluster.
- `--constraint`: Memaksa Docker hanya menaruh container di node tertentu (berguna saat Node02 error DNS).
- `--limit-cpu` vs `--reserve-cpu`: `limit` adalah batas maksimal, `reserve` adalah jatah minimal yang dijamin.

### 7.4 Lab 7.3: Scale & Rolling Update
```bash
# Ubah jumlah replika menjadi 1
docker service scale web2=1

# Update image ke versi baru (tanpa mematikan service)
docker service update --image sistyo/myweb web2
```
💡 **Catatan Pemahaman:**
- `update --image`: Melakukan *Rolling Update*. Docker akan mematikan container lama satu per satu, dan menggantinya dengan image baru secara perlahan agar web tidak pernah mati (Zero Downtime).

### 7.5 Lab 7.4, 7.5, 7.6: Portainer (GUI Management)
Portainer mempermudah manajemen Docker via browser.
```bash
# Install Portainer Server (di Node01)
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9000:9000 --name portainer --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
💡 **Catatan Pemahaman:**
- `-v /var/run/docker.sock:...`: Ini adalah "kabel" yang menghubungkan Portainer ke Docker Daemon. Tanpa ini, Portainer tidak bisa mengontrol Docker.
- **Setup Token:** Saat pertama buka web Portainer, kita harus mengambil `setup_token` dari log (`docker logs portainer`). Ini fitur keamanan agar server tidak bisa dibajak.
- **Registries:** Menghubungkan Portainer ke Docker Hub wajib menggunakan **Personal Access Token (PAT)**, bukan password biasa.

---

# 📘 MODUL 8: Logging Driver

### 8.1 Konsep Logging Driver
Log sangat penting untuk DevOps (debugging & troubleshooting). Docker memiliki mekanisme bawaan untuk mengelola log.
- **Default:** `json-file` (menyimpan log dalam format JSON di host). Kekurangannya, file log bisa membengkak dan memenuhi *disk* (disk exhaustion) jika tidak diatur.
- **Local Driver:** Sangat direkomendasikan. Melakukan rotasi log secara otomatis dan menggunakan format file yang lebih efisien.
- **Driver Lain:** `syslog`, `journald`, `fluentd`, `awslogs`, dll.

### 8.2 Lab 8.1: Configuring Logging Driver
**Task:** Mengatur batas maksimal ukuran log agar tidak memenuhi penyimpanan VM.

**1. Konfigurasi Global (Semua Container)**
Buat/edit file daemon.json:
```bash
sudo vim /etc/docker/daemon.json
```
Isi dengan konfigurasi berikut:
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "100"
  }
}
```
Restart Docker agar konfigurasi terbaca:
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
💡 **Catatan Pemahaman:**
- `max-size: "10m"`: File log akan dirotasi (dipotong) jika ukurannya mencapai 10 Megabyte.
- `max-file: "100"`: Menyimpan maksimal 100 file log lama. Jika lebih, file paling lama akan dihapus.
- `daemon-reload`: Memberitahu systemd (manajer service Linux) bahwa ada perubahan konfigurasi, agar Docker bisa di-restart dengan bersih.

**2. Konfigurasi Spesifik Container**
Kita juga bisa mengatur log hanya untuk container tertentu saat menjalankannya:
```bash
docker run --name alpine --log-driver json-file --log-opt max-size=10m registry.adinusa.id/btacademy/alpine echo hello world
```
💡 **Catatan Pemahaman:**
- `--log-driver`: Menentukan driver khusus untuk container ini, mengabaikan settingan global di `daemon.json`.
- `--log-opt`: Menentukan opsi spesifik.

**3. Cek File Log Secara Fisik**
Log tidak hanya bisa dilihat dengan `docker logs`, tapi tersimpan sebagai file di VM Host.
```bash
# Mendapatkan ID Container secara otomatis
sudo cat /var/lib/docker/containers/$(docker ps --no-trunc -a | grep 'alpine' | awk '{print $1}')/$(docker ps --no-trunc -a | grep 'alpine' | awk '{print $1}')-json.log | jq .
```
💡 **Catatan Pemahaman:**
- `/var/lib/docker/containers/...`: Ini adalah lokasi asli (di dalam VM Host) tempat Docker menyimpan log container.
- `$(docker ps -a...)`: Perintah untuk mengambil ID container secara otomatis agar kita tidak perlu mengetik ID yang panjang secara manual.
- `jq .`: Perintah untuk merapikan (format *pretty-print*) output JSON agar mudah dibaca oleh mata manusia.

---

# 📘 MODUL 9: Health Check

### 9.1 Konsep Health Check
Docker hanya tahu apakah container *running* atau *stopped*. Docker tidak tahu apakah aplikasi di dalamnya benar-benar sehat (misal: web server hang atau error 500). 
Instruksi `HEALTHCHECK` memberi tahu Docker bagaimana cara mengetes container untuk memastikan aplikasinya masih berfungsi.

- **Status:** `starting` (baru nyala), `healthy` (tes lolos), `unhealthy` (tes gagal terus-menerus).
- **Opsi:**
  - `--interval`: Jeda antar tes (default 30s).
  - `--timeout`: Batas waktu maksimal satu kali tes (default 30s).
  - `--retries`: Berapa kali gagal berturut-turut sebelum dinyatakan `unhealthy`.
  - `--start-period`: Waktu tunggu di awal agar aplikasi siap (gagal di masa ini tidak dihitung).

### 9.2 Lab 9.1: Health Check (Practice 01 - HTTP)
**Task:** Membuat web server yang bisa berubah status dari `healthy` ke `unhealthy`.

**1. Buat Dockerfile**
```dockerfile
FROM registry.adinusa.id/btacademy/docker-http-server:health
HEALTHCHECK --interval=1s --retries=3 \
  CMD curl --fail http://localhost:80/ || exit 1
```
💡 **Catatan Pemahaman:**
- `curl --fail`: Opsi `-f` (fail) membuat curl mengembalikan *exit code* 1 jika server merespons dengan kode error HTTP (seperti 404 atau 500). `|| exit 1` memastikan container ditandai `unhealthy`.

**2. Build & Run**
```bash
docker build -t http-healthcheck .
docker run -d -p 80:80 --name http-healthcheck http-healthcheck
```

**3. Uji Coba**
```bash
# Awalnya sehat
curl http://localhost/
docker ps # Status: (healthy)

# Rusak aplikasinya
curl http://localhost/unhealthy
docker ps # Status: (unhealthy)
```

### 9.3 Lab 9.1: Health Check (Practice 02 - Node.js)
**Task:** Menguji Health Check pada aplikasi Node.js yang memiliki endpoint untuk mematikan/menyalakan server.
⚠️ **PENTING:** Lakukan praktik ini di **Node01**. Node02 sering mengalami error DNS saat mengunduh image `node`.

**1. Buat `server.js`**
*(Sesuai instruksi lab: Endpoint port 8081 digunakan untuk mematikan/menyalakan server di port 8080).*

**2. Buat Dockerfile**
```dockerfile
FROM registry.adinusa.id/btacademy/node
COPY server.js /
EXPOSE 8080 8081
HEALTHCHECK --interval=5s --timeout=10s --retries=3 CMD curl -sS 127.0.0.1:8080 || exit 1
CMD ["node","server.js"]
```
💡 **Catatan Pemahaman:**
- `curl -sS`: `-s` (silent) tidak menampilkan *progress bar*, `-S` (show-error) menampilkan pesan error jika curl gagal. Sangat rapi untuk di dalam script.

**3. Build & Run (Di Node01)**
```bash
docker build -t node-server .
docker run -d --name nodeserver -p 8080:8080 -p 8081:8081 node-server
```

**4. Uji Coba**
```bash
# Cek status kesehatan (Raw JSON)
docker inspect --format "{{json .State.Health }}" nodeserver | jq .

# Matikan server
curl 127.0.0.1:8081
# Tunggu beberapa detik, cek status jadi (unhealthy)

# Nyalakan server lagi
curl 127.0.0.1:8081
# Status kembali (healthy)
```

---

# 🐛 KUMPULAN TROUBLESHOOTING (ERROR NYATA YANG PERNAH DIALAMI)

1. **Error `404 Not Found` saat `docker build` (apt-get):**
   - *Penyebab:* Base image terlalu tua (repositori paketnya sudah dihapus).
   - *Solusi:* Edit Dockerfile, ganti base image ke versi yang lebih baru (misal `node:18`).

2. **Error `i/o timeout` pada `registry.adinusa.id` / `docker.io`:**
   - *Penyebab:* Masalah DNS lokal (terutama di Node02) atau koneksi internet terputus.
   - *Solusi:* `docker pull <image>` secara manual di Node01, atau restart docker (`sudo systemctl restart docker`). Jika di Swarm, gunakan `--constraint 'node.role == manager'`. **Sebaiknya jalankan lab yang butuh pull image di Node01.**

3. **Error `plugin vieux/sshfs is in use`:**
   - *Penyebab:* Mencoba `disable` plugin saat volume/container masih menggunakannya.
   - *Solusi:* Hapus container & volume dulu, baru `disable` plugin.

4. **Error `Permission denied` saat membuat file di folder Volume:**
   - *Penyebab:* Folder `/var/lib/docker/volumes/` milik `root`.
   - *Solusi:* Gunakan `echo "teks" | sudo tee /path/file`.

5. **Error `Segmentation fault` saat `docker-compose --version`:**
   - *Penyebab:* Binary Compose V1 usang.
   - *Solusi:* Hapus `/usr/bin/docker-compose`, install `docker-compose-plugin`, dan gunakan `docker compose` (pakai spasi).

6. **Error `docker swarm join requires 1 argument`:**
   - *Penyebab:* Lupa mencantumkan IP Manager dan port (2377).
   - *Solusi:* `docker swarm join --token <TOKEN> 10.7.7.10:2377`.

7. **Portainer error `dial tcp 10.7.7.20:9001: connect: connection refused`:**
   - *Penyebab:* Portainer Agent di Node02 belum jalan atau port-nya terblokir.
   - *Solusi:* Pastikan container `portainer_agent` di Node02 statusnya `Up`. Jika pakai Swarm mode, pastikan service `portainer_agent` sudah berjalan di semua node.

8. **Error `denied: requested access to the resource is denied` (Docker Push/Portainer):**
   - *Penyebab:* Nama image tidak sesuai dengan username Docker Hub yang sedang login.
   - *Solusi:* Lakukan `docker tag <image_lama> <username_dockerhub>/<image_baru>` lalu push ulang.

9. **Error `Bind for 0.0.0.0:9001 failed: port is already allocated`:**
   - *Penyebab:* Sisa container dari percobaan Swarm sebelumnya masih nyangkut dan memakan port.
   - *Solusi:* Hapus service (`docker service rm portainer_agent`), lalu hapus semua container (`docker rm -f $(docker ps -aq)`).

10. **Error DNS Timeout saat Build di Lab 9.1 Practice 02:**
    - *Gejala:* `dial tcp: lookup registry.adinusa.id ... i/o timeout` saat `docker build -t node-server .`
    - *Penyebab:* Perintah dieksekusi di Node02 yang bermasalah DNS.
    - *Solusi:* Hentikan eksekusi di Node02. Buka terminal **Node01**, buat ulang folder `hc-latihan02`, file `server.js`, dan `Dockerfile`, lalu jalankan `docker build` dan `docker run` di sana.

---
*Disclaimer: Catatan ini adalah rangkuman pribadi berdasarkan modul dan lab dari Adinusa Docker Fundamental.*
