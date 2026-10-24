# 🐳 DOCKER FUNDAMENTAL — COMPLETE NOTES

**Course:** Docker Fundamental (Adinusa)
**Author:** [@paladxn](https://github.com/paladxn)
**Status:** 🚀 Ongoing (Completed: Modul 1 - 6)
# 🐳 DOCKER FUNDAMENTAL — COMPLETE NOTES

**Course:** Docker Fundamental (Adinusa)
**Author:** [@paladxn](https://github.com/paladxn)
**Status:** 🚀 Completed (Modul 1 - 7)

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
- `usermod -aG docker $USER`: Memasukkan user aktif ke dalam grup `docker` agar tidak perlu mengetik `sudo` terus-menerus.
- `chmod 666 /var/run/docker.sock`: Memberikan izin baca-tulis ke *socket* Docker (jalur komunikasi antara CLI dan Daemon).

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
- `-d` (Detach): Melepas container agar berjalan di *background* sehingga terminal bisa dipakai lagi.
- `-p 80:80`: Formatnya `<Port_Host>:<Port_Container>`. Port 80 di laptop/VM diteruskan ke port 80 di dalam container.
- `-p 80` (tanpa host port): Docker akan memilihkan port acak di host (misal 32768) untuk meneruskan ke port 80 container.

### 2.3 Lab 2.3: Docker Run - Part 2 (Exec & Ephemeral)
```bash
# Masuk ke container (interactive)
docker exec -it nginx2 /bin/bash

# Ephemeral Container (otomatis terhapus saat keluar)
docker run -it --rm --name ubuntu2 registry.adinusa.id/btacademy/ubuntu
```
💡 **Catatan Pemahaman:**
- `exec -it`: `-i` (interactive) dan `-t` (TTY). Membuka "pintu" baru ke dalam container yang sedang berjalan *tanpa* mematikan aplikasi utamanya.
- `--rm`: *Remove*. Begitu container berhenti (misal kita mengetik `exit`), container langsung dihancurkan agar tidak menumpuk sampah.

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
- `-e`: *Environment variable*. Digunakan untuk mengonfigurasi aplikasi di dalam container tanpa harus mengubah kode. (Misal: set password root MySQL).
- `--memory`: Batas RAM maksimal. Jika lewat, container akan dibunuh sistem (OOM Killer).
- `--cpu-shares`: Bobot prioritas CPU (bukan batas maksimal), menentukan siapa yang dapat jatah CPU lebih banyak jika server sibuk.

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
- `docker create`: Membuat container tapi tidak menjalankannya.
- `docker cp`: Menyalin file dari VM Host ke dalam container (bahkan saat container mati).
- `--volumes-from`: Meminjam volume dari container lain.

### 3.3 Lab 3.2: Mount Volume with NFS Server
```bash
# Di Node01 (Server)
docker run -itd --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v /data/nfs-storage01:/data -p 2049:2049 registry.adinusa.id/btacademy/nfs-server-alpine:12

# Di Node02 (Client)
sudo mount -v -t nfs4 10.7.7.10:/ /mnt
```
💡 **Catatan Pemahaman:**
- `--privileged`: Memberikan akses *super-admin* (root penuh) ke container. Diperlukan karena NFS Server butuh memodifikasi konfigurasi kernel untuk melayani sharing file.
- `-t nfs4`: Format *filesystem* untuk mount dari remote server.

### 3.4 Lab 3.3: Mount Volume with Read-only Mode
```bash
docker run -d --name=nginx-rwvol -v my-vol:/usr/share/nginx/html -p 4005:80 registry.adinusa.id/btacademy/nginx:latest

# Mount Read-Only (Container tidak bisa menulis/mengubah file)
docker run -d --name=nginx-rovol -v my-vol:/usr/share/nginx/html:ro registry.adinusa.id/btacademy/nginx:latest
```
💡 **Catatan Pemahaman:**
- `:ro` (*Read-Only*): Sangat penting untuk keamanan (Security). Jika web server kita di-hack, penyerang tidak bisa menaruh file berbahaya (webshell) karena folder web-nya dikunci *Read-Only*.

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

# 🐛 KUMPULAN TROUBLESHOOTING (ERROR NYATA YANG PERNAH DIALAMI)

1. **Error `404 Not Found` saat `docker build` (apt-get):**
   - *Penyebab:* Base image terlalu tua (repositori paketnya sudah dihapus).
   - *Solusi:* Edit Dockerfile, ganti base image ke versi yang lebih baru (misal `node:18`).

2. **Error `i/o timeout` pada `registry.adinusa.id` / `docker.io`:**
   - *Penyebab:* Masalah DNS lokal (terutama di Node02) atau koneksi internet terputus.
   - *Solusi:* `docker pull <image>` secara manual di Node01, atau restart docker (`sudo systemctl restart docker`). Jika di Swarm, gunakan `--constraint 'node.role == manager'`.

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

---
*Disclaimer: Catatan ini adalah rangkuman pribadi berdasarkan modul dan lab dari Adinusa Docker Fundamental.*
> Dokumen ini adalah rangkuman komprehensif (Satu File untuk Semua) dari seluruh kurikulum Docker Fundamental Adinusa. Disusun sesuai urutan *Course Content* asli, dilengkapi dengan praktik lab dan *troubleshooting* nyata.

---

# 📘 MODUL 1: Introduction of Container

### 1.1 Course Introduction & What Are Containers?
Container adalah cara standar untuk mengemas aplikasi beserta dependensinya (library, konfigurasi) agar bisa berjalan di mana saja (Build, Ship, and Run Anywhere).

### 1.2 Why are containers important?
- Mengatasi masalah "Works on my machine" (aplikasi jalan di laptop dev, tapi error di server).
- Jauh lebih ringan dibanding Virtual Machine (VM).
- Mempercepat siklus rilis aplikasi (CI/CD).

### 1.3 Example of Containerized Technology
Contoh paling nyata adalah Docker Hub, tempat ribuan image siap pakai (seperti Nginx, MySQL, Redis) bisa di-*pull* dan langsung dijalankan dalam hitungan detik.

### 1.4 Host, Virtual Machine and Container
- **VM:** Memiliki OS tamu (Guest OS) lengkap, sangat berat (ukuran GB), booting lambat.
- **Container:** Berbagi Kernel dengan Host OS, sangat ringan (ukuran MB), booting dalam milidetik.
- **Isolasi:** Container menggunakan Linux Namespaces (untuk isolasi) dan Cgroups (untuk limit resource).

### 1.5 Platform Discussion
Komunitas belajar Docker Adinusa menggunakan Discord untuk berdiskusi terkait kendala lab dan teori.

### 1.6 Lab 1.1: Preparation of Lab Environment
**Task:** Menyiapkan VirtualBox, membuat NAT Network, Host-Only Network, dan meng-*import* VM Ubuntu 22.04 (Docker-Node01 & Docker-Node02).
**Key Step:** Mengatur IP statis via Netplan, mengubah hostname, dan mengatur *passwordless SSH* antar node.

---

# 📘 MODUL 2: Introduction to Docker

### 2.1 Docker Products & Pricing
- **Docker CE (Community Edition):** Gratis, untuk developer individu.
- **Docker EE (Enterprise Edition):** Berbayar, untuk perusahaan (fitur keamanan ekstra).
- **Docker Desktop:** Gratis untuk personal/edukasi, berbayar untuk perusahaan besar (>250 karyawan).

### 2.2 Docker Release Cycle
Docker merilis versi stabil dua kali setahun (contoh: 24.0, 25.0), dengan siklus pemeliharaan (maintenance) selama 7 bulan sebelum versi baru dirilis.

### 2.3 Lab 2.1: Installing Docker
**Task:** Install Docker Engine di Ubuntu via repository resmi.
```bash
# Install prerequisites & GPG Key
sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Tambah repository & install
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Ijinkan user menjalankan docker tanpa sudo
sudo usermod -aG docker $USER
sudo chmod 666 /var/run/docker.sock

# Test
docker run registry.adinusa.id/btacademy/hello-world
```

### 2.4 Lab 2.2: Docker Run - Part 1
**Task:** Menjalankan container Redis & Nginx (Foreground, Background, Port Mapping).
```bash
# Foreground (Blocking terminal)
docker run registry.adinusa.id/btacademy/redis

# Background (Detached)
docker run -d --name redis1 registry.adinusa.id/btacademy/redis

# Port Mapping (Host:Container)
docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest

# Random Port (Docker pilih port host acak)
docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest
docker port nginx2 80  # Cek port acak yang didapat
curl localhost:$(docker port nginx2 80 | cut -d : -f 2)
```

### 2.5 Lab 2.3: Docker Run - Part 2 (Exec & Ephemeral)
**Task:** Masuk ke dalam container, install tools, dan hapus container otomatis.
```bash
# Masuk ke container (interactive)
docker exec -it nginx2 /bin/bash
apt-get update && apt-get install nano -y
nano /usr/share/nginx/html/index.html # Edit file web
exit

# Ephemeral Container (--rm otomatis hapus saat exit)
docker run -it --rm --name ubuntu2 registry.adinusa.id/btacademy/ubuntu
```

### 2.6 Lab 2.4: Docker Run - Part 3 (Environment, Pause, Limits)
**Task:** Konfigurasi environment, link antar container, pause, dan limit CPU/RAM.
```bash
# Run MySQL dengan Environment Variables & Link
docker run -d --name my-mysql -e MYSQL_ROOT_PASSWORD=RAHASIA -e MYSQL_DATABASE=latihan05 -p 3306:3306 registry.adinusa.id/btacademy/mysql
docker run --name my-phpmyadmin -d --link my-mysql:db -p 8090:80 registry.adinusa.id/btacademy/phpmyadmin

# Pause & Unpause (Membekukan proses)
docker pause ubuntu1
docker unpause ubuntu1

# Resource Limit (Batasi RAM & CPU)
docker run -d --name ch6_mariadb --memory 256m --cpu-shares 1024 -e MYSQL_ROOT_PASSWORD=test registry.adinusa.id/btacademy/mariadb:5.5
```

---

# 📘 MODUL 3: Managing Docker Container

### 3.1 & 3.2 Managing the Life Cycle of Containers
- **Create:** `docker create` atau `docker run`
- **Start/Stop:** `docker start`, `docker stop` (SIGTERM), `docker kill` (SIGKILL)
- **Remove:** `docker rm` (hapus container, tapi data di writable layer hilang)
- **Inspect:** `docker inspect` (melihat detail JSON), `docker stats` (melihat RAM/CPU), `docker top` (melihat proses).

### 3.3 Public Registry & Private Registry
- **Public:** Docker Hub (Rate limit 100 pull/6 jam untuk anonymous).
- **Private:** Harbor (Bisa di-*host* sendiri, mendukung *role-based access control* dan *scanning vulnerability*).

### 3.4 Docker Volume & Docker Network (Teori)
- **Volume:** Direktori di host (`/var/lib/docker/volumes/`) yang di-mount ke container agar data persisten.
- **Network:** Jaringan virtual Docker. Default-nya `bridge` (tidak support DNS by name). Harus buat `custom bridge` agar container bisa saling ping berdasarkan nama.

### 3.5 Lab 3.1: Mount Volume (Local)
**Task:** Copy file dari host ke volume container menggunakan `--volumes-from`.
```bash
docker create -v /my-volume --name my-volume registry.adinusa.id/btacademy/busybox
docker cp . my-volume:/my-volume
docker run --volumes-from my-volume registry.adinusa.id/btacademy/ubuntu ls /my-volume
```

### 3.6 Lab 3.2: Mount Volume with NFS Server
**Task:** Membuat container NFS Server dan me-mount-nya dari node lain.
```bash
# Di Node01 (Server NFS)
sudo mkdir -p /data/nfs-storage01/
docker run -itd --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v /data/nfs-storage01:/data -p 2049:2049 registry.adinusa.id/btacademy/nfs-server-alpine:12

# Di Node02 (Client NFS)
sudo apt install nfs-client -y
sudo mount -v -t nfs4 10.7.7.10:/ /mnt
sudo touch /mnt/file1.txt
```

### 3.7 Lab 3.3: Mount Volume with Read-only Mode
**Task:** Mount volume sebagai Read-Only (`:ro`) agar container tidak bisa memodifikasi file.
```bash
docker volume create my-vol
docker run -d --name=nginx-rwvol -v my-vol:/usr/share/nginx/html -p 4005:80 registry.adinusa.id/btacademy/nginx:latest

# Modifikasi dari Host (Gunakan sudo tee jika permission denied)
echo "This is from my-vol" | sudo tee /var/lib/docker/volumes/my-vol/_data/index.html

# Mount Read-Only
docker run -d --name=nginx-rovol -v my-vol:/usr/share/nginx/html:ro registry.adinusa.id/btacademy/nginx:latest
```

### 3.8 Lab 3.4: Volume Driver (SSHFS)
**Task:** Mount direktori remote via SSH menggunakan plugin `vieux/sshfs`.
```bash
# ⚠️ WAJIB DISABLE DULU SEBELUM SET SSH KEY
docker plugin install --grant-all-permissions vieux/sshfs
docker plugin ls # Catat ID (misal: b8275dd4fb5c)
docker plugin disable b8275dd4fb5c
docker plugin set vieux/sshfs sshkey.source=/home/student/.ssh/
docker plugin enable b8275dd4fb5c

# Buat volume & jalankan
docker volume create --driver vieux/sshfs -o sshcmd=student@10.7.7.20:/share -o allow_other sshvolume
docker run -d --name=nginxtest-sshfs -p 8090:80 -v sshvolume:/usr/share/nginx/html registry.adinusa.id/btacademy/nginx:latest
```

### 3.9 Lab 3.5 & 3.6: Default Bridge vs Host Network
**Task:** Membuktikan DNS resolution di custom bridge & perilaku host network.
```bash
# Default Bridge (Ping by name GAGAL)
docker run -dit --name alpine1 registry.adinusa.id/btacademy/alpine ash
docker run -dit --name alpine2 registry.adinusa.id/btacademy/alpine ash
docker exec -it alpine1 ping alpine2 # Gagal!

# Custom Bridge (Ping by name BERHASIL)
docker network create --driver bridge bridge1
docker network connect bridge1 alpine1
docker network connect bridge1 alpine2
docker exec -it alpine1 ping alpine2 # Berhasil!

# Host Network (Port -p Diabaikan, langsung tembus host)
docker run -itd --network host --name my_nginx registry.adinusa.id/btacademy/nginx
```

---

# 📘 MODUL 4: Creating Custom Docker Container Image

### 4.1 Docker Images & Registry (Teori Layer)
Image terdiri dari tumpukan **Layer** (Read-Only). Saat container dibuat, Docker menambahkan **Writable Layer** di paling atas. Jika container dihapus, Writable Layer hilang, tapi Image (Layer di bawahnya) tetap aman dan bisa di-*share* ke container lain.

### 4.2 Manipulating Container Images
- `docker save -o file.tar <image>`: Simpan image ke file fisik (untuk backup/offline).
- `docker load -i file.tar`: Load image dari file.
- `docker tag <image> <user>/<repo>:<tag>`: Beri nama alias untuk push.
- `docker push <user>/<repo>:<tag>`: Upload ke Docker Hub.

### 4.3 Building & Reviewing Dockerfile
**Instruksi Penting:**
- `FROM`: Base image (misal `alpine`).
- `RUN`: Eksekusi saat *build* (install package). Gabungkan dengan `&&` agar irit layer.
- `COPY`: Copy file dari Host ke dalam Image.
- `WORKDIR`: Set direktori kerja.
- `CMD` & `ENTRYPOINT`: Perintah yang dijalankan saat container *start*.

### 4.4 Lab 4.1: Exploring Dockerfile
**Task:** Build image Node.js & Whalesay.
```bash
# ⚠️ PERBAIKAN: Jika apt-get 404, ubah base image (misal dari node:18.13.0 ke node:18)
# ⚠️ PERBAIKAN: Jika registry timeout, lakukan 'docker pull <image>' manual dulu.
docker build -t node-latihan01 .
docker run -d --rm --name node-latihan01 -p 8080:8080 node-latihan01
```

### 4.5 Lab 4.2: Exploring Dockerfile (Flask Apps)
**Task:** Build image Python Flask & Push ke Docker Hub.
```bash
docker build -t flask-latihan03 .
docker tag flask-latihan03 <username_dockerhub>/flask-latihan03:latest
docker push <username_dockerhub>/flask-latihan03:latest
docker run -d -p 5000:5000 --name flask03 <username_dockerhub>/flask-latihan03
```

### 4.6 Lab 4.3: Exploring Dockerfile (Quiz Web Statis)
**Task:** Menyalin file web statis ke dalam image Nginx.
```dockerfile
FROM registry.adinusa.id/btacademy/nginx:latest
COPY mywebsite/ /usr/share/nginx/html/
```
```bash
docker build -t latihan-quiz01-image .
docker run -d --name quiz01 -p 80:80 latihan-quiz01-image
```

---

# 📘 MODUL 5: Docker Compose

### 5.1 Konsep Declarative vs Imperative
- **Imperative (Cara Lama):** Kita memberikan perintah spesifik langkah demi langkah ke Docker (`docker run`, `docker network create`, dll). Jika ada error, kita harus perbaiki manual.
- **Declarative (Cara Compose):** Kita hanya mendefinisikan "Apa yang kita inginkan" (state akhir) di dalam file YAML. Docker akan otomatis mengurus langkah-langkah untuk mencapai state tersebut.

### 5.2 Struktur File `docker-compose.yml`
```yaml
services:
  web:
    image: palopalepalo/web:1.0
    build: web  # Build dari folder 'web' jika image belum ada
    ports:
      - "80:3000"
  db:
    image: postgres:12-alpine
    volumes:
      - pets-data:/var/lib/postgresql/data

volumes:
  pets-data: # Volume harus dideklarasikan di sini
```

### 5.3 Perintah Penting Docker Compose
| Perintah | Fungsi |
|---|---|
| `docker compose up -d` | Jalankan semua service di background. |
| `docker compose ps` | Lihat status container (hanya yang ada di compose file). |
| `docker compose down` | Hentikan & hapus container, network (tapi volume tetap ada). |
| `docker compose down -v` | Hentikan & hapus container, network, **dan volume**. |
| `docker compose -p <nama> up` | Menjalankan dengan nama project kustom (bukan nama folder). |

### 5.4 Scaling Service
Jika kita ingin menjalankan 3 container untuk service `web`:
```bash
docker compose up -d --scale web=3
```
*⚠️ Peringatan:* Jika di `ports` kita menulis `"80:3000"`, scaling akan **gagal** karena port 80 host bentrok. 
**Solusi:** Cukup tulis port container-nya saja (`- "3000"`), maka Docker akan memberikan port host acak untuk setiap instance.

### 5.5 Lab 5.1: Using Docker Compose (WordPress & MySQL)
**Task:** Membuat stack WordPress + MySQL menggunakan Compose.
*⚠️ **CATATAN PENTING:** Gunakan plugin Compose V2 (`docker compose` pakai spasi). Jangan mengunduh binary `docker-compose` manual karena sering `Segmentation fault`.*

```yaml
services:
  db:
    image: registry.adinusa.id/btacademy/mysql:5.7
    volumes:
      - dbdata:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: [username]
      MYSQL_PASSWORD: [password]

  wordpress:
    depends_on:
      - db
    image: registry.adinusa.id/btacademy/wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: [username]
      WORDPRESS_DB_PASSWORD: [password]
volumes:
  dbdata:
```
```bash
# Jalankan (Gunakan Spasi, bukan tanda hubung)
docker compose up -d
docker ps
```

---

# 📘 MODUL 6: Docker Continuous Integration (CI)

### 6.1 Apa itu CI/CD?
**Continuous Integration (CI)** adalah praktik developer menggabungkan kode ke repositori (seperti GitHub) beberapa kali sehari. Setiap *commit* akan memicu *automated build* dan *testing* untuk mendeteksi *bug* lebih awal.

### 6.2 Alur CI dengan Docker (Jenkins/GitHub Actions)
1. Developer `git push` ke GitHub.
2. GitHub mengirim *webhook* ke Jenkins (atau GitHub Actions).
3. Jenkins *pull* repository (termasuk Dockerfile).
4. Jenkins melakukan `docker build`.
5. Jenkins menjalankan container untuk testing.
6. Jika test lolos, image di-`docker push` ke Docker Hub / Private Registry.

### 6.3 Docker Hub Automated Build
Docker Hub bisa terhubung langsung ke repositori GitHub. 
- Setiap kali kamu push kode ke branch tertentu (misal `main`), Docker Hub akan otomatis melakukan build image dan menyimpannya.
- Status build bisa dilihat: `Queued` (Antre), `Building` (Proses), `Success` (Berhasil), `Error` (Gagal).

---

# 🐛 KUMPULAN TROUBLESHOOTING (ERROR YANG PERNAH DIALAMI)

1. **Error `404 Not Found` saat `docker build` (apt-get):**
   - *Penyebab:* Base image terlalu tua (repositori paketnya sudah dihapus).
   - *Solusi:* Edit Dockerfile, ganti base image ke versi yang lebih baru (misal `node:18` atau `ubuntu:22.04`).

2. **Error `i/o timeout` pada `registry.adinusa.id`:**
   - *Penyebab:* Masalah DNS lokal atau salah eksekusi di Node02.
   - *Solusi:* Pastikan eksekusi di Node01. Jika masih timeout, `docker pull <image>` secara manual, atau restart docker (`sudo systemctl restart docker`).

3. **Error `plugin vieux/sshfs is in use`:**
   - *Penyebab:* Mencoba `disable` plugin saat volume/container masih menggunakannya.
   - *Solusi:* Hapus container & volume dulu (`docker rm -f <container>` & `docker volume rm <volume>`), baru `disable` plugin.

4. **Error `Permission denied` saat membuat `index.html` di folder Volume:**
   - *Penyebab:* Folder `/var/lib/docker/volumes/` adalah milik `root`.
   - *Solusi:* Jangan pakai `>` (redirection), pakai `sudo tee`: `echo "teks" | sudo tee /path/index.html`.

5. **`docker inspect` IP Address mengembalikan `null`:**
   - *Penyebab:* Perubahan format JSON di Docker versi baru.
   - *Solusi:* Gunakan `docker inspect <container> | jq -r '.[].NetworkSettings.Networks.bridge.IPAddress'` atau cukup akses via `localhost:<port_mapping>`.

6. **Tampilan masih "Welcome to nginx" setelah build Dockerfile:**
   - *Penyebab:* File HTML tidak ter-copy ke `/usr/share/nginx/html`.
   - *Solusi:* Cek `COPY` di Dockerfile. Pastikan nama file adalah `index.html`.

7. **Error `Segmentation fault` saat `docker-compose --version`:**
   - *Penyebab:* Mengunduh binary Compose V1 (Python) yang tidak cocok dengan OS/arsitektur.
   - *Solusi:* Hapus binary lama (`sudo rm -f /usr/bin/docker-compose`), lalu install plugin V2 (`sudo apt install docker-compose-plugin -y`). Gunakan perintah `docker compose` (pakai spasi).

---
*Disclaimer: Catatan ini adalah rangkuman pribadi berdasarkan modul dan lab dari Adinusa Docker Fundamental. Akan terus di-update seiring berjalannya materi.*
