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
  2. Buat user baru  
  3. Setup SSH Key Authentication & disable password auth

- **Bukti:**  
  (tambah screenshot nanti setelah implementasi)

---

## 3. Docker
- **Masalah:**  
  Aplikasi web di `/opt/app-test/` harus dijalankan di container.

- **Solusi:**  
  1. Buat Dockerfile berbasis `nginx:alpine`  
  2. Build image & jalankan container di port 8080

- **Bukti:**  
  (tambah screenshot container running)

---

## 4. Automation
- **Masalah:**  
  Perlu backup log Nginx secara otomatis.

- **Solusi:**  
  1. Script backup `/var/log/nginx/` ke `/tmp/backup-logs/` setiap jam 2 pagi  
  2. Hapus file >7 hari otomatis (pakai cron job)

- **Bukti:**  
  (tambah screenshot cron job & script output)

---

## 5. Architecture Design
- **Desain untuk 100.000 user/hari:**  
  - Load Balancer di depan Nginx  
  - Cluster Nginx / Web server  
  - Database terdistribusi / sharding  
  - CDN untuk konten statis  
  - Auto-scaling server jika traffic tinggi

- **Bukti / Diagram:**  
  (tambahkan diagram arsitektur jika ada)
