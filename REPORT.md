# Laporan Assessment - Aldi

---

## 1. Troubleshooting
- **Masalah:**  
  Service Nginx tidak berjalan, muncul error 502 Bad Gateway dan 403 Forbidden.

- **Solusi:**  
  1. Cek status Nginx: `sudo systemctl status nginx`  
  2. Periksa log error: `cat /var/log/nginx/error.log`  
  3. Perbaiki konfigurasi default di `/etc/nginx/sites-enabled/default`  
  4. Pastikan file index ada di `/var/www/html`  
  5. Restart Nginx: `sudo systemctl restart nginx`

- **Bukti:**  
  - `evidence/images/error awal.png`  
  - `evidence/images/Error 502 Bukti salah arsitektur (proxy tanpa backend).png`  
  - `evidence/images/rror 403 ukti masalah file index.png`  
  - `evidence/images/buti perbaikan config.png`  
  - `evidence/images/Bukti isi directory .png`  
  - `evidence/images/buti berhasil Welcome to nginx!.png`

---

## 2. Security Hardening
- **Masalah:**  
  Server tidak aman, port terbuka sembarangan, SSH masih pakai password.

- **Solusi:**  
  1. Setup UFW: allow port 6622, 80, 8080  
     ```bash
     sudo ufw reset
     sudo ufw default deny incoming
     sudo ufw default allow outgoing
     sudo ufw allow 6622/tcp
     sudo ufw allow 80/tcp
     sudo ufw allow 8080/tcp
     sudo ufw enable
     ```
  2. Buat user baru `aldi` dan tambahkan ke group sudo  
     ```bash
     sudo adduser aldi
     sudo usermod -aG sudo aldi
     ```
  3. Setup SSH Key Authentication & disable password auth  
     ```bash
     ssh-keygen -t ed25519 -C "aldi@103.30.144.48"
     mkdir -p ~/.ssh
     cp /home/deploy/.ssh/id_ed25519 ~/.ssh/
     cp /home/deploy/.ssh/id_ed25519.pub ~/.ssh/
     chmod 700 ~/.ssh
     chmod 600 ~/.ssh/id_ed25519
     chmod 644 ~/.ssh/id_ed25519.pub
     cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
     chmod 600 ~/.ssh/authorized_keys
     chown -R aldi:aldi ~/.ssh
     ```
     - Nonaktifkan password login di `/etc/ssh/sshd_config`:
       ```
       PasswordAuthentication no
       ```
     - Restart SSH: `sudo systemctl restart ssh`

- **Bukti:**  
  - `evidence/images/adduser_aldi.png`  
  - `evidence/images/User baru & sudo.png`  
  - `evidence/images/Permission folder ~.ssh.png`  
  - `evidence/images/SSH key-only login.png`  
  - `evidence/images/ufw_before.png`  
  - `evidence/images/ufw_status.png`  

---

## 3. Docker
- **Masalah:**  
  Aplikasi web di `/opt/app-test/` harus dijalankan di container.

- **Solusi:**  
  1. Buat Dockerfile berbasis `nginx:alpine`  
     ```dockerfile
     FROM nginx:alpine
     COPY . /usr/share/nginx/html
     EXPOSE 8080
     CMD ["nginx", "-g", "daemon off;"]
     ```
  2. Build image & jalankan container:  
     ```bash
     docker build -t app-test /opt/app-test/
     docker run -d -p 8080:80 app-test
     ```

- **Bukti:**  
  - `evidence/images/status dockersebelum dan sesudah di start.png`
  - `evidence/images/bukti setup docker tanpa sudo.png`
  - `evidence/images/bukti docker & run berhasil.png`
  - `evidence/images/bukti Verifikasi konten aplikasi.png`
## Soal 4: Backup Log Nginx

Untuk soal ini, dibuat script `backup.sh` yang melakukan backup log Nginx ke folder `evidence/logs/`. Berikut bukti hasil backup:

- File backup yang berhasil dibuat: `log-backup-2026-04-01.tar.gz`
- Screenshot isi folder logs:

![Soal 4: Backup Log Nginx](evidence/images/soal4_ls_logs_2026-04-01.png)

## 5. Architecture Design (High Traffic Scenario)

**Kasus:**  
Aplikasi statis yang dijalankan di Docker Nginx tiba-tiba viral. CPU server mentok 100% dan database (yang akan dipasang) mulai lambat.

### 5.1 Identifikasi Masalah
- **Server tunggal:** Semua trafik web dan database ditangani satu server → bottleneck CPU & I/O.  
- **Database:** Jika database ditambahkan di server yang sama, I/O akan makin lambat.  
- **Scalability:** Arsitektur saat ini tidak scalable. Semua request masuk ke satu titik → single point of failure.

### 5.2 Rancangan Arsitektur High Traffic

#### a. Load Balancer
- **Fungsi:** Mendistribusikan request ke beberapa server Nginx/Docker container.  
- **Teknologi:** Nginx/HAProxy atau Cloud Load Balancer (AWS ELB, GCP LB).  
- **Benefit:** Mencegah overload pada satu server, meningkatkan availability.

#### b. Horizontal Scaling (Web Server)
- Jalankan beberapa container Docker Nginx di server berbeda.  
- **Strategi:** Docker Swarm atau Kubernetes untuk orchestrasi dan auto-scaling.  
- **Benefit:** Trafik tinggi bisa ditangani lebih banyak container → CPU tidak mentok.

#### c. Database Scaling
- **Masalah:** Database lambat → harus dipisah dari web server.  
- **Solusi:**  
  - Database Server terpisah (dedicated VM/container).  
  - Scaling:  
    - **Vertical:** Upgrade CPU/RAM.  
    - **Horizontal:** Replikasi read-only (Master-Slave) untuk query baca.  
  - Gunakan caching (Redis/Memcached) untuk mengurangi load DB.

#### d. Caching
- **Fungsi:** Menyimpan response statis atau query populer agar web server & DB tidak overload.  
- **Teknologi:** Redis, Varnish, CDN (Cloudflare/Akamai) untuk distribusi global.

#### e. Auto-scaling & Monitoring
- Monitoring untuk deteksi CPU/memory overload (Prometheus + Grafana).  
- Auto-scale container/web server saat load naik → elastis terhadap trafik viral.
### 5.3 Diagram Arsitektur
+-----------------+
             |  Load Balancer  |
             +--------+--------+
                      |
      +---------------+---------------+
      |                               |

+------------------+ +------------------+
| Nginx Docker VM1 | | Nginx Docker VM2 |
+------------------+ +------------------+
| |
+---------------+---------------+
|
+-----------------+
| Database Server |
| (MySQL/Postgres)|
+-----------------+
|
+-----------------+
| Cache (Redis)|
+-----------------+


**Perbaikan utama:**  
- Flow traffic jelas dari atas ke bawah.  
- Horizontal scaling terlihat rapi, garis menyambung dengan benar.  
- Tidak ada garis dobel atau spasi kosong yang bikin layout pecah.  
- Bisa langsung dipaste di `REPORT.md` tanpa pecah di GitHub.  

Kalau mau, aku bisa buat versi **lebih compact** tapi tetap rapi, sehingga diagramnya lebih kecil dan mudah dilihat di layar sempit. Mau aku buatkan versi itu juga?
