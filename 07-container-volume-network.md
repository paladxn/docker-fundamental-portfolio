# Docker Fundamental — Container Management, Volume & Network

**Course:** Docker Fundamental (Adinusa)
**Topic:** Lifecycle Container, Public/Private Registry, Docker Volume (Local, NFS, SSHFS), Docker Network (Bridge, Host)
**Status:** ✅ Completed

---

## 🎯 Tujuan Pembelajaran

Setelah menyelesaikan modul ini, saya mampu:

- Mengelola *lifecycle* container dari pembuatan hingga penghapusan (`run`, `ps`, `exec`, `stop`, `rm`).
- Memahami perbedaan **Public Registry** (Docker Hub) dan **Private Registry** (Harbor).
- Menerapkan **Docker Volume** untuk persistensi data (Local Volume, NFS, SSHFS).
- Melakukan *backup* dan *restore* volume.
- Memahami dan mengonfigurasi **Docker Network** (Default Bridge, Custom Bridge, Host).

---

## 📘 Bagian 1: Teori Container Lifecycle & Registry

### 1.1 Docker Client Verbs (Lifecycle)
Docker menyediakan berbagai perintah untuk mengubah state (status) container:
- `docker run`: Membuat container baru dari image dan menjalankannya.
- `docker exec`: Menjalankan perintah tambahan di dalam container yang **sedang berjalan**.
- `docker stop`: Menghentikan container secara *graceful* (mengirim SIGTERM, lalu SIGKILL jika timeout).
- `docker kill`: Menghentikan container secara paksa (mengirim SIGKILL).
- `docker restart`: Menghentikan lalu menyalakan ulang container.
- `docker rm`: Menghapus container (state dan filesystem-nya hilang).

**Pentingnya ID dan Nama:**
Setiap container memiliki `CONTAINER ID` (hexadecimal unik) dan `NAMES` (bisa random atau ditentukan dengan `--name`). Nama harus unik, bahkan container yang sudah mati (stopped) tetap memegang namanya sampai dihapus (`docker rm`).

### 1.2 Public vs Private Registry
- **Public Registry (Docker Hub)**: Tempat menyimpan image publik. Memiliki **Rate Limit** (100 pulls/6 jam untuk anonim, 200 untuk login gratis, 5000 untuk paid).
- **Private Registry**: Tempat menyimpan image secara privat (internal perusahaan). Contoh: **Harbor** (open-source, mendukung RBAC, scanning vulnerability, dan signing image).

---

## 📘 Bagian 2: Teori Docker Volume

### 2.1 Tipe Mount
1. **Volume**: Dikelola oleh Docker, disimpan di `/var/lib/docker/volumes/`. Best practice untuk data produksi.
2. **Bind Mount**: Menghubungkan direktori host secara langsung ke container. Cocok untuk *development*.
3. **tmpfs**: Disimpan di RAM (memori). Hilang saat container mati. Cocok untuk data sensitif/cache.

### 2.2 Volume Driver
Docker memungkinkan penggunaan *driver* eksternal untuk menyimpan volume di luar host, seperti:
- **NFS**: Network File System (share direktori antar Linux server).
- **CIFS/Samba**: Share direktori dari Windows.
- **SSHFS**: Mount direktori remote via SSH (menggunakan plugin `vieux/sshfs`).

---

## 📘 Bagian 3: Teori Docker Network

Docker Network menggunakan *driver* yang bisa di-plug:
1. **bridge** (Default): Jaringan internal di satu host. Container bisa saling komunikasi via IP, tapi **tidak bisa** ping berdasarkan nama (DNS) kecuali menggunakan *custom bridge*.
2. **host**: Container berbagi *network namespace* dengan host. Container tidak mendapat IP sendiri, dan *port mapping* (`-p`) akan diabaikan.
3. **overlay**: Menghubungkan banyak host Docker (Swarm mode).
4. **macvlan**: Memberikan MAC address fisik ke container.
5. **none**: Menonaktifkan jaringan.

---

## 🧪 Lab 3.1 — Mount Volume (Local & `--volumes-from`)

### Soal
1. Buat direktori kerja dan 10 file kosong.
2. Buat container (tanpa menjalankannya) dengan volume `/my-volume`.
3. Copy file dari host ke dalam volume container.
4. Jalankan container baru yang menggunakan volume tersebut (`--volumes-from`).

### ✅ Solusi

```bash
# 1. Create working directory
$ cd $HOME
$ mkdir -p latihan/latihan01-volume
$ cd latihan/latihan01-volume

# 2. Create 10 empty files
$ for i in {1..10}; do touch file-$i; done

# 3. Create container and volume without running
$ sudo docker create -v /my-volume --name my-volume registry.adinusa.id/btacademy/busybox

# 4. Copy file ke dalam container
$ sudo docker cp . my-volume:/my-volume

# 5. Running container baru dan mount volume dari container lama
$ docker run --volumes-from my-volume registry.adinusa.id/btacademy/ubuntu ls /my-volume
```

---

## 🧪 Lab 3.2 — Mount Volume with NFS Server

### Soal
1. Setup NFS Server menggunakan container di Node01.
2. Dari Node02, install `nfs-client` dan mount direktori Node01.
3. Buat file di Node02, lalu verifikasi di Node01.

### ✅ Solusi

**Eksekusi di Node01:**
```bash
# 1. Create working directory
$ sudo mkdir -p /data/nfs-storage01/

# 2. Create the NFS Server with docker
$ docker run -itd --privileged --restart unless-stopped -e SHARED_DIRECTORY=/data -v /data/nfs-storage01:/data -p 2049:2049 registry.adinusa.id/btacademy/nfs-server-alpine:12
```

**Eksekusi di Node02:**
```bash
# 3. SSH ke Node02
$ ssh 10.7.7.20

# 4. Install NFS Client
$ sudo apt install nfs-client -y

# 5. Mount direktori Node01 ke /mnt di Node02
$ sudo mount -v -t nfs4 10.7.7.10:/ /mnt

# 6. Cek mounting
$ df -h

# 7. Buat file test
$ sudo touch /mnt/file1.txt
$ sudo touch /mnt/file2.txt
$ exit
```

**Eksekusi kembali di Node01:**
```bash
# 8. Verifikasi bahwa file dari Node02 masuk ke direktori Node01
$ ls /data/nfs-storage01/
# Output: file1.txt file2.txt
```

---

## 🧪 Lab 3.3 — Mount Volume with Read-only Mode

### Soal
Buat volume, mount secara RW (Read-Write) lalu RO (Read-Only), dan inspeksi `Mounts`.

### ✅ Solusi

```bash
# 1. Create docker volume
$ docker volume create my-vol

# 2. Display list docker volume
$ docker volume ls

# 3. Display docker volume details
$ docker volume inspect my-vol

# 4. Running container RW
$ docker run -d --name=nginx-rwvol -v my-vol:/usr/share/nginx/html -p 4005:80 registry.adinusa.id/btacademy/nginx:latest

# 5. Test browsing (Bisa via localhost karena ada port mapping)
$ curl http://localhost:4005

# 6. Ubah isi volume dari Host
$ echo "This is from my-vol source directory" | sudo tee /var/lib/docker/volumes/my-vol/_data/index.html

# 7. Test browsing ulang (Akan berubah)
$ curl http://localhost:4005

# 8. Running container RO
$ docker run -d --name=nginx-rovol -v my-vol:/usr/share/nginx/html:ro registry.adinusa.id/btacademy/nginx:latest

# 9. Inspect container RO (Pastikan "RW": false)
$ docker inspect nginx-rovol | jq '.[].Mounts'
```

---

## 🧪 Lab 3.4 — Volume Driver (SSHFS)

### Soal
Gunakan plugin `vieux/sshfs` untuk mount direktori dari Node02 ke container di Node01.

### ✅ Solusi

**Eksekusi di Node02:**
```bash
$ sudo mkdir /share
$ sudo chmod 777 /share
$ echo "Hello Adinusa!" > /share/index.html
```

**Eksekusi di Node01:**
*(Pastikan Node01 bisa `ssh student@10.7.7.20` tanpa password)*

```bash
# Bersihkan sisa error sebelumnya (jika ada)
$ docker rm -f nginxtest-sshfs
$ docker volume rm sshvolume

# 1. Install plugin & konfigurasi SSH Key
$ docker plugin install --grant-all-permissions vieux/sshfs
$ docker plugin ls
# (Ganti <ID> dengan ID asli dari output, misal: b8275dd4fb5c)
$ docker plugin disable <ID>
$ docker plugin set vieux/sshfs sshkey.source=/home/student/.ssh/
$ docker plugin enable <ID>

# 2. Create volume dengan driver sshfs
$ docker volume create --driver vieux/sshfs -o sshcmd=student@10.7.7.20:/share -o allow_other sshvolume

# 3. Running container
$ docker run -d --name=nginxtest-sshfs -p 8090:80 -v sshvolume:/usr/share/nginx/html registry.adinusa.id/btacademy/nginx:latest

# 4. Test browsing
$ curl http://localhost:8090
# Output: Hello Adinusa!
```

---

## 🧪 Lab 3.5 — Default & Custom Bridge Network

### Soal
Buktikan bahwa di *default bridge network*, container tidak bisa saling ping menggunakan nama, melainkan harus menggunakan IP. Lalu buat custom bridge untuk memperbaikinya.

### ✅ Solusi

```bash
# 1. Display list of networks
$ sudo docker network ls

# 2. Run 2 alpine container
$ docker run -dit --name alpine1 registry.adinusa.id/btacademy/alpine ash
$ docker run -dit --name alpine2 registry.adinusa.id/btacademy/alpine ash
$ docker container ls

# 3. Cek IP Address alpine2
$ docker inspect alpine2 | jq -r '.[].NetworkSettings.Networks.bridge.IPAddress'
# (Misal: 172.17.0.3)

# 4. Masuk ke alpine1
$ docker exec -it alpine1 sh

# 5. Ping internet (Berhasil)
$ ping -c 3 8.8.8.8

# 6. Ping IP alpine2 (Berhasil)
$ ping -c 3 172.17.0.3

# 7. Ping nama alpine2 (GAGAL)
$ ping -c 3 alpine2
$ exit  # (Ctrl+D)

# 8. Buat custom bridge network dan hubungkan ke kedua container
$ docker network create --driver bridge bridge1
$ docker network connect bridge1 alpine1
$ docker network connect bridge1 alpine2

# 9. Kembali ke alpine1 dan ping by name
$ docker exec -it alpine1 sh
$ ping -c 3 alpine2
# Output: Berhasil! (Karena custom bridge mendukung DNS resolution)
$ exit
```

---

## 🧪 Lab 3.6 — Host Network

### Soal
Jalankan Nginx menggunakan network host. Pastikan port mapping tidak berfungsi dan akses langsung via `localhost`.

### ✅ Solusi

```bash
# 1. Run container with host network
$ docker run -itd --network host --name my_nginx registry.adinusa.id/btacademy/nginx

# 2. Test browsing
$ curl http://localhost

# 3. Verification IP Address and bound port 80
$ ip add
$ sudo netstat -tulpn | grep :80
```

### ⚠️ Catatan Penting Lab 3.6
Saat menggunakan `--network host`:
- Container **tidak** memiliki IP sendiri (berbagi IP host).
- Flag `-p` atau `-P` akan **diabaikan** (menghasilkan warning).
- Container langsung menempati port host (misal Nginx menempati port 80 host). Jika ada aplikasi lain di host yang memakai port 80, akan terjadi konflik.

---

## 🐛 Troubleshooting Umum

1. **Plugin "is in use"**: Jika `docker plugin disable` gagal, pastikan tidak ada container atau volume yang sedang menggunakan plugin tersebut (`docker rm -f` dan `docker volume rm`).
2. **Ping by name gagal**: Ini adalah perilaku normal di *default bridge network* Docker. Selalu gunakan **Custom Bridge Network** jika butuh DNS resolution internal.
3. **Mount NFS gagal / RPC Error**: Pastikan container NFS Server berjalan dengan flag `--privileged`, dan pastikan firewall/iptables tidak memblokir port `2049`.
4. **`jq: command not found`**: Install `jq` di host (misal: `sudo apt install jq -y`) untuk mem-parsing output JSON dari `docker inspect`.

---

## 📌 Kesimpulan Modul

Modul ini menggabungkan seluruh aspek penyimpanan (Volume) dan jaringan (Network) di Docker. 
- **Volume** memastikan data tetap ada meski container dihancurkan. Kita bisa menyimpannya secara lokal, di server NFS terpusat, atau di server remote via SSHFS.
- **Network** memungkinkan container berkomunikasi. Memahami perbedaan *Default Bridge*, *Custom Bridge* (DNS resolution), dan *Host Network* (tanpa isolasi) sangat krusial untuk membangun arsitektur aplikasi yang tepat.

**Motto Docker:** *Data yang aman dan koneksi yang tepat adalah kunci skalabilitas.*

> ⚠️ **Disclaimer:** Catatan ini ditulis berdasarkan eksekusi lab dan pemahaman pribadi dari modul Adinusa. Materi asli tidak didistribusikan.
