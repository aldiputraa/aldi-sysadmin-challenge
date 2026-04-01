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
## 4. Automation
- **Masalah:**  
  Perlu backup log Nginx secara otomatis.

- **Solusi:**  
  1. Script backup `/var/log/nginx/` ke `/tmp/backup-logs/` setiap jam 2 pagi  
     ```bash
     #!/bin/bash
     BACKUP_DIR="/tmp/backup-logs/$(date +%F)"
     mkdir -p "$BACKUP_DIR"
     cp -r /var/log/nginx/* "$BACKUP_DIR"
     find /tmp/backup-logs/* -type d -mtime +7 -exec rm -rf {} \;
     ```
  2. Tambahkan cron job:
     ```
     0 2 * * * /path/to/backup_script.sh
     ```

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
