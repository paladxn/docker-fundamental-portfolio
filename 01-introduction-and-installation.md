# Docker Fundamental — Teori & Praktikum Lengkap

**Course:** Docker Fundamental (Adinusa)
**Topic:** Pengenalan Docker, Arsitektur, Instalasi, Persiapan Environment, dan Docker Run (Part 1)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan modul ini, saya mampu:

- Memahami konsep dasar **containerization** dan perbedaannya dengan Virtual Machine (VM)
- Mengenal ekosistem, produk, dan alternatif dari Docker (Podman, LXC, LXD)
- Menyiapkan environment lab menggunakan **VirtualBox** (NAT & Host-Only Network)
- Melakukan **instalasi Docker Engine** secara manual via repository resmi
- Menjalankan dan mengelola container dasar (Redis & Nginx) menggunakan **Docker CLI**

---

## 📘 Bagian 1: Konsep Dasar Container & Docker

### 1.1 Apa itu Container?
Di masa lalu, developer menyerahkan aplikasi ke tim ops, lalu tim ops harus menginstalnya di server. Masalah muncul ketika banyak aplikasi dengan dependensi (library, framework, versi) yang berbeda harus berjalan di satu server yang sama.

**Virtual Machine (VM)** sempat menjadi solusi, namun VM sangat berat karena membawa *full operating system* di dalamnya.

**Container** hadir sebagai solusi ringan. Container mengemas aplikasi beserta dependensinya dalam satu "kotak hitam" standar (standardized black box). Karena berbagi kernel OS host, container jauh lebih kecil, cepat, dan efisien dibanding VM. Inilah yang mempopulerkan semboyan **"Build, ship, and run anywhere"**.

### 1.2 Perbandingan Host, VM, dan Container
| Fitur | Dedicated Server (Host) | Virtual Machine (VM) | Container (Docker) |
|---|---|---|---|
| **Isolasi** | Tidak ada | Hardware-level (Hypervisor) | OS-level (Namespaces & cgroups) |
| **OS** | 1 OS Host | Full OS (Guest OS) per VM | Share OS Kernel Host |
| **Ukuran** | Sangat Besar | Puluhan GB | Puluhan MB |
| **Boot Time** | Lambat | Menit | Detik (Instan) |

### 1.3 Ekosistem Container: Docker vs Alternatifnya
- **Docker**: Platform open-source paling populer untuk build, ship, dan run aplikasi. Terdiri dari Docker CE (Community Edition - Gratis) dan Docker EE (Enterprise Edition - Berbayar).
- **Podman**: Alternatif *daemonless* dan *rootless* yang kompatibel dengan CLI Docker (bisa di-alias `alias docker=podman`). Sangat bagus untuk keamanan di Linux.
- **LXC (Linux Containers)**: Teknologi container tingkat OS (System Container), terasa seperti menggunakan VM ringan.
- **LXD**: Manajer VM dan system container generasi baru yang dibangun di atas LXC (dikembangkan oleh Canonical/Ubuntu).

### 1.4 Model Lisensi Docker (Pricing)
Docker memiliki beberapa tingkatan langganan (Subscription):
- **Docker Personal ($0)**: Gratis untuk individu, edukasi, open-source, dan bisnis kecil.
- **Docker Pro ($9/bln)**: Untuk profesional individu (fitur debugging & cloud build).
- **Docker Team ($15/bln)**: Untuk tim kecil (kolaborasi, RBAC, audit log).
- **Docker Business ($24/bln)**: Untuk enterprise (SSO, image access management, isolasi container yang lebih ketat).

---

## 🔧 Bagian 2: Persiapan Lab Environment (Lab 1.1)

Lab ini menggunakan **VirtualBox** untuk membuat 2 VM (Docker-Node01 & Docker-Node02) dengan OS Ubuntu 22.04. 

### 2.1 Spesifikasi VM
| VM | vCPU | RAM | NAT Network | Host-Only Network | Internal Network |
|---|---|---|---|---|---|
| Docker-Node01 | 1 | 1GB | 192.168.0.11/24 | 10.10.10.11/24 | 10.7.7.10/24 |
| Docker-Node02 | 1 | 1GB | 192.168.0.12/24 | 10.10.10.12/24 | 10.7.7.20/24 |

### 2.2 Setup VirtualBox
1. **NAT Network**: Buka *Tools > Network Manager > NAT Networks*. Buat network baru dengan IP `192.168.0.0/24`.
2. **Host-Only Network**: Buka *Tools > Network Manager > Host-Only Networks*. Set IP `10.10.10.1`, netmask `255.255.255.0`, dan *Disable DHCP*.
3. **Import VM**: File > Import Appliance > Pilih file `BTA-Server.ova`. Ubah nama menjadi `Docker-Node01`.
4. **Clone VM**: Klik kanan `Docker-Node01` > Clone > Beri nama `Docker-Node02` > Pilih **Full Clone**.

### 2.3 Konfigurasi OS (Dilakukan di kedua Node)
Login ke VM dengan user `student` dan password `Adinusa2023`.
1. Edit network config: `sudo nano /etc/netplan/50-cloud-init.yaml` (sesuaikan dengan tabel IP di atas).
2. Apply network: `sudo netplan apply`.
3. Ubah hostname: `sudo hostnamectl set-hostname pod-[username]-node01` (atau node02).
4. Daftarkan host di `/etc/hosts` agar bisa saling ping by name.
5. **Setup SSH Key** (agar bisa ssh tanpa password):
   ```bash
   ssh-keygen -t rsa
   ssh-copy-id student@pod-[username]-node01
   ssh-copy-id student@pod-[username]-node02
   ```
6. Verifikasi passwordless SSH: `ssh student@pod-[username]-node01 "whoami; hostname"`.

---

## 🧪 Bagian 3: Lab 1 — Instalasi Docker

### Soal
Install Docker pada node, tambahkan user ke group docker, dan jalankan test image `hello-world`.

### ✅ Solusi

```bash
# 1. Persiapan (Login & Start Lab)
$ nusactl login
$ nusactl start do-002-1

# 2. Add Docker's official GPG key:
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl -y
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc

# 3. Add the repository to Apt sources:
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update

# 4. Install Docker packages
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 5. Display docker version & info
$ docker version
$ docker info

# 6. Add user into docker group (agar tidak perlu sudo)
$ sudo usermod -aG docker $USER
$ sudo chmod 666 /var/run/docker.sock

# 7. Test installation
$ docker run registry.adinusa.id/btacademy/hello-world

# 8. Display downloaded image & all containers
$ docker image ls
$ docker container ls -a
```

### 🔍 Penjelasan Opsi Penting
- `usermod -aG docker $USER`: Memasukkan user saat ini ke grup `docker` agar bisa menjalankan perintah docker tanpa `sudo`. Wajib logout/login atau jalankan `newgrp docker` setelahnya.
- `chmod 666 /var/run/docker.sock`: Memberikan izin baca/tulis ke socket Docker (shortcut di lab, namun secara best practice menggunakan grup docker lebih disarankan).

---

## 🧪 Bagian 4: Lab 2.2 — Docker Run (Part 1)

### Soal
1. Cari image **redis** & **nginx** dari DockerHub dan Harbor.
2. Jalankan container dengan mode foreground, background, dan beri nama.
3. Lakukan port mapping pada Nginx (`80:80` dan random port).
4. Lakukan inspeksi (`inspect`, `logs`, `stats`, `top`).

### ✅ Solusi

```bash
# Persiapan Lab 2.2
$ nusactl login
$ nusactl start do-002-2

# ===== PRACTICE 01: Redis =====
# 1. Search image
$ docker search redis
$ skopeo list-tags docker://registry.adinusa.id/btacademy/redis

# 2. Run container
# Foreground (Tekan Ctrl+C untuk stop)
$ docker run registry.adinusa.id/btacademy/redis

# Background
$ docker run -d registry.adinusa.id/btacademy/redis

# Dengan nama 'redis1'
$ docker run -d --name redis1 registry.adinusa.id/btacademy/redis

# 3. Display container
$ docker ps
$ docker container ls
$ docker ps -a
$ docker container ls -a

# 4. Inspect & Monitor
$ docker inspect redis1
$ docker logs redis1
$ docker stats redis1
$ docker top redis1

# 5. Shutdown
$ docker stop redis1


# ===== PRACTICE 02: NGINX =====
# 1. Search image
$ docker search nginx
$ skopeo list-tags docker://registry.adinusa.id/btacademy/nginx

# 2. Run Nginx (Port 80:80)
$ docker run -d --name nginx1 -p 80:80 registry.adinusa.id/btacademy/nginx:latest
$ docker inspect nginx1

# 3. Run Nginx (Random Host Port)
$ docker run -d --name nginx2 -p 80 registry.adinusa.id/btacademy/nginx:latest

# 4. Test Browsing (Port nginx2 dicari otomatis)
$ curl localhost:$(docker port nginx2 80 | cut -d : -f 2)

# 5. Display container & images
$ docker ps -a
$ docker images
```

### ⚠️ Urutan Sangat Penting
- **`-d` vs Tanpa `-d`**: Tanpa `-d`, terminal akan terblokir (foreground) hingga ditekan `Ctrl+C`. Gunakan `-d` untuk service yang berjalan lama.
- **Port Mapping**: Format penulisan adalah `HOST:CONTAINER`. Jika `-p 80:80`, host port 80 diteruskan ke container port 80. Jika hanya `-p 80`, Docker memilihkan port acak untuk host.
- **Nama Container**: Selalu gunakan `--name` agar mudah di-*inspect* atau di-*stop* tanpa harus menyalin *Container ID* yang panjang.

---

## 🔍 Verifikasi

### 1. Verifikasi Instalasi (Lab 1)
Pastikan perintah `docker run` bisa dieksekusi **tanpa** `sudo`.
```bash
$ docker run registry.adinusa.id/btacademy/hello-world
```

### 2. Verifikasi Container (Lab 2.2)
```bash
$ docker ps
```
Pastikan `nginx1` dan `nginx2` berstatus `Up`. Cek `curl localhost:80` dan `curl localhost:<port_nginx2>`, keduanya harus mengembalikan halaman default HTML Nginx.

### 3. Grading
```bash
$ nusactl grade do-002-1
$ nusactl grade do-002-2
```

---

## 💡 Insight & Catatan Pribadi

### Mengapa Docker Sangat Cepat?
Docker tidak memerlukan *boot sequence* OS seperti VM. Docker hanya menjalankan proses (PID 1) di dalam *namespace* yang terisolasi, menggunakan kernel Linux milik host secara langsung. Inilah sebabnya container bisa menyala dalam hitungan milidetik.

### Docker vs Podman
Saat ini banyak enterprise beralih ke **Podman** karena arsitekturnya yang *daemonless* (tidak butuh background service seperti `dockerd`) dan *rootless* (bisa dijalankan oleh user biasa tanpa eskalasi hak akses). Namun, Docker tetap menjadi standar industri karena ekosistem GUI (Docker Desktop) dan kemudahan penggunaannya.

### Port Random (`-p 80`)
Fitur `-p 80` sangat berguna saat kita ingin menjalankan banyak container Nginx tanpa bentrok. Docker akan memilih port host yang belum terpakai. Kita bisa mengeceknya dengan `docker port <nama_container>`.

> ⚠️ **Disclaimer:** Catatan ini merupakan rangkuman pribadi dari modul materi dan lab Adinusa (Docker Fundamental). Materi asli tidak didistribusikan.
