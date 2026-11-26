# PANDUAN DAN NASKAH REKAMAN VIDEO DEMONSTRASI PURPLE TEAM LOOP
## Mata Kuliah TEK1314 — Keamanan Siber (PBL Fase 3)
### Smart Cow Monitoring System — Replika Keamanan Backend IoT & RTSP Streaming

---

**Nomor Dokumen:** VDOC-PBL-2026-KEL06  
**Tanggal Revisi:** 26 November 2026 (Penambahan Prosedur Konfigurasi Manual & Sinkronisasi Live Stream)  
**Klasifikasi:** Internal / Restricted (PBL Mata Kuliah TEK1314)  
**Institusi:** Program Studi D4 Teknologi Rekayasa Komputer, Sekolah Vokasi, Institut Pertanian Bogor  
**Mata Kuliah:** TEK1314 — Keamanan Siber (Kelas A)  
**Kelompok Pelaksana:** Kelompok 6 — Kelas A  

| Nama Anggota | NIM | Peran dalam Demo |
|---|---|---|
| **Hodelia Victory Dorola Marbun** | J0404241100 | Lead Coordinator / Narator Pembuka & Penutup |
| **Muhammad Kevin Almer** | J0404241163 | Red Team Specialist (Operator Kali Linux) |
| **Rizki Ramadhani** | J0404241144 | Blue Team Specialist (Operator Target Server) |
| **Yuwandri Alfarizi** | J0404241010 | Blue Team Specialist (Operator SIEM Security Onion) |

---

## 1. Parameter Topologi Lab & Identitas Mesin Virtual

Sesuai ketentuan Bagian VI Panduan Final PBL, rekaman video wajib menampilkan nama host dan alamat IP masing-masing mesin virtual untuk membuktikan orisinalitas lab kelompok:

* **Attacker Node (`attacker-kel6`):** `192.168.6.10` (Kali Linux 2026.3 AMD64)
* **Target Node (`target-kel6`):** `192.168.6.20` (Ubuntu Server 26.04.1 LTS)
* **Monitoring SIEM Node (`monitor-kel6`):** `192.168.6.30` (CyberOps Security Onion 16.04)
* **Segmen Internal Lab:** VirtualBox Internal Network `siber-kel6` (`192.168.6.0/24`)

---

## 2. Prosedur Sakelar Simulasi Purple Team (Target Node)

Untuk membuktikan efektivitas mitigasi secara langsung (*live demonstration*), Blue Team menyediakan dua pilihan metode untuk mengubah status Target dari **Kondisi Sebelum Mitigasi (Vulnerable / Loose)** menjadi **Kondisi Setelah Mitigasi (Hardened / Secure)** pada skenario SSH dan RTSP.

### 2.1 Pilihan Metode: Manual vs Skrip Otomatis
* **Metode Manual (Sangat Direkomendasikan saat Perekaman):** Operator Target mengedit langsung berkas konfigurasi SSH dan menjalankan perintah firewall UFW di terminal. Hal ini memberikan transparansi penuh kepada dosen penguji bahwa konfigurasi sistem benar-benar diubah dan dipahami oleh mahasiswa.
* **Metode Skrip Cepat:** Menggunakan skrip sakelar otomatis `./loose_target.sh` dan `./harden_target.sh` yang telah disediakan di direktori `/home/kel6/`.

---

### 2.2 Prosedur Konfigurasi Manual (Terminal Target Server)

#### A. Mengendurkan Konfigurasi (Kondisi Sebelum Mitigasi — Loose)
Jalankan langkah-langkah berikut di terminal Target Server (`target-kel6`):

1. **Kendurkan Autentikasi SSH (Izinkan Login Password):**
   Buka file konfigurasi drop-in SSH hardening:
   ```bash
   sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
   ```
   Ubah baris:
   ```ini
   PasswordAuthentication no
   KbdInteractiveAuthentication no
   ```
   Menjadi:
   ```ini
   PasswordAuthentication yes
   KbdInteractiveAuthentication yes
   ```
   *(Simpan berkas: tekan `Ctrl + O`, lalu `Enter`, lalu keluar dengan `Ctrl + X`)*.  
   Muat ulang layanan SSH:
   ```bash
   sudo systemctl reload ssh
   ```
   *(Atau satu baris perintah cepat)*:
   ```bash
   sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/00-hardening.conf
   sudo sed -i 's/KbdInteractiveAuthentication no/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config.d/00-hardening.conf
   sudo systemctl reload ssh
   ```

2. **Buka Akses Port RTSP 554 ke Penyerang (Kali Linux):**
   ```bash
   sudo ufw allow from 192.168.6.10 to any port 554 proto tcp
   ```
   Verifikasi status aturan UFW:
   ```bash
   sudo ufw status verbose
   ```

---

<div style="page-break-before: always;"></div>

#### B. Mengunci Kembali Konfigurasi (Kondisi Setelah Mitigasi — Hardened)
Jalankan langkah-langkah berikut di terminal Target Server (`target-kel6`):

1. **Kunci Kembali Autentikasi SSH (Hanya Kunci Publik, Tolak Password):**
   Buka file konfigurasi:
   ```bash
   sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
   ```
   Kembalikan baris menjadi:
   ```ini
   PasswordAuthentication no
   KbdInteractiveAuthentication no
   ```
   Muat ulang layanan SSH:
   ```bash
   sudo systemctl reload ssh
   ```
   *(Atau satu baris perintah cepat)*:
   ```bash
   sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config.d/00-hardening.conf
   sudo sed -i 's/KbdInteractiveAuthentication yes/KbdInteractiveAuthentication no/' /etc/ssh/sshd_config.d/00-hardening.conf
   sudo systemctl reload ssh
   ```

2. **Tutup Akses Port RTSP 554 dari Penyerang (Kali Linux):**
   ```bash
   sudo ufw delete allow from 192.168.6.10 to any port 554 proto tcp
   ```
   Verifikasi bahwa port 554 kini hanya terbuka untuk IP Monitoring Security Onion (`192.168.6.30`) atau ditolak secara default:
   ```bash
   sudo ufw status verbose
   ```

---

### 2.3 Prosedur Alternatif: Skrip Sakelar Cepat
Jika waktu presentasi terbatas, gunakan skrip siap pakai di direktori `/home/kel6/`:

* **Mengendurkan Target (Sebelum Mitigasi):**  
  `sudo /home/kel6/loose_target.sh`
* **Mengunci Kembali Target (Setelah Mitigasi):**  
  `sudo /home/kel6/harden_target.sh`

---

### 2.4 Status Layanan Latar Belakang Video Streaming (FFmpeg)
Layanan video sintetis CCTV kandang sapi berjalan otomatis sebagai *daemon systemd* di Target Server.

Perintah verifikasi status layanan:
```bash
systemctl status smartcow-stream
```
Titik siar RTSP aktif di: `rtsp://192.168.6.20:554/live/cow1` (didistribusikan secara internal oleh MediaMTX).

<div style="page-break-before: always;"></div>

## 3. Struktur Storyboard & Naskah Narasi (Scene-by-Scene)

### 🎬 Scene 0: Pembukaan & Pembuktian Identitas IP (00:00 – 00:45)
* **Tampilan Visual:** Tampilkan jendela terminal ketiga mesin virtual berdampingan.
* **Perintah yang Dijalankan:**
  * Di Kali Linux: `ip a show eth0`
  * Di Ubuntu Target: `ip a show enp0s3`
  * Di Security Onion: `ip a show eth0`
* **Naskah Narasi (Voice-Over):**
  > *"Halo, selamat pagi/siang Bapak/Ibu dosen penguji dan rekan-rekan. Kami dari Kelompok 6 Kelas A mempresentasikan video demonstrasi Purple Team Loop dan penanganan insiden siber pada replika Smart Cow Monitoring System.*  
  > *Sebagai bukti orisinalitas infrastruktur lab kelompok kami: di sebelah kiri adalah Attacker Node Kali Linux dengan IP 192.168.6.10; di tengah adalah Target Node Ubuntu Server dengan IP 192.168.6.20; dan di kanan adalah Monitoring Node Security Onion dengan IP 192.168.6.30."*

---

### 🎬 Scene 1: ATK-01 — Port Scanning & Reconnaissance (00:45 – 01:40)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Kibana Security Onion $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Target Server memastikan port 554 terbuka untuk simulasi:
    * *Metode Manual:* `sudo ufw allow from 192.168.6.10 to any port 554 proto tcp`
    * *Metode Skrip:* `./loose_target.sh`
  * Kali Linux menjalankan pemindaian port:
    ```bash
    nmap -sV -p 22,80,554 192.168.6.20
    ```
  * Tunjukkan hasil: port 22, 80, dan 554 berstatus **OPEN**.
  * Di Security Onion, tunjukkan log koneksi Zeek dan notice deteksi scan.
  * Terapkan mitigasi hardening di Target (hapus izin port 554):
    * *Metode Manual:* `sudo ufw delete allow from 192.168.6.10 to any port 554 proto tcp`
    * *Metode Skrip:* `./harden_target.sh`
  * Kali Linux mengulang nmap: port 554 kini **FILTERED**.
* **Naskah Narasi (Voice-Over):**
  > *"Skenario pertama, ATK-01: Network Reconnaissance. Penyerang menjalankan pemindaian port Nmap ke Target Node.*  
  > *(Sebelum mitigasi)*: Seluruh port layanan—port 22 SSH, port 80 Web, dan port 554 RTSP—berstatus OPEN tanpa firewall.*  
  > *Di Security Onion, Zeek mencatat lonjakan TCP SYN pada conn.log dan memicu alert pemindaian port.*  
  > *Mitigasi Blue Team: Kami mengaktifkan firewall UFW dengan menyaring akses port 554 khusus untuk IP monitoring.*  
  > *(Retest)*: Saat Red Team mengulang Nmap setelah mitigasi, port 554 kini berstatus **FILTERED**. Penyerang kehilangan akses pemetaan port streaming."*

---

### 🎬 Scene 2: ATK-02 — SQLite Database Exposure via Web (01:40 – 02:35)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Kali Linux menjalankan uji request HTTP GET:
    ```bash
    curl -I http://192.168.6.20/smartcow.db
    curl -I http://192.168.6.20/local/data/smartcow.db
    ```
  * Target Server memverifikasi lokasi berkas lokal yang aman:
    ```bash
    ls -l /home/kel6/SmartCow/local/data/smartcow.db
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario kedua, ATK-02: Information Disclosure pada basis data SQLite.*  
  > *(Sebelum mitigasi)*: Berkas basis data smartcow.db tersimpan di folder web publik, sehingga penyerang dapat mengunduh seluruh data sensor dan hash password.*  
  > *Mitigasi Blue Team: Kami menerapkan konsep Out-of-Web-Root dengan memindahkan berkas database ke direktori privat /home/kel6/SmartCow/local/data/ dan menghapus rute static file sharing.*  
  > *(Retest)*: Saat Red Team mencoba request ulang via curl, server mengembalikan respon **HTTP/1.1 404 Not Found**. Basis data terselamatkan dari kebocoran."*

---

### 🎬 Scene 3: ATK-03 — Web Login Brute-Force & Rate Limiting (02:35 – 03:45)
*(Siklus demo utama Purple Team Loop yang wajib ditunjukkan secara mendalam)*
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Kibana Discover Security Onion $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Kali Linux melancarkan loop request autentikasi:
    ```bash
    for i in {1..12}; do curl -s -o /dev/null -w "Percobaan $i: HTTP %{http_code}\n" -X POST http://192.168.6.20/api/login -H "Content-Type: application/json" -d '{"username":"admin","password":"wrongpassword"}'; done
    ```
  * Tunjukkan bahwa percobaan ke-1 s/d 10 merespons `401`, dan percobaan ke-11 & 12 langsung merespons `HTTP 429 Too Many Requests`.
  * Di Security Onion Kibana, tunjukkan rekaman log HTTP POST yang dibatasi.
* **Naskah Narasi (Voice-Over):**
  > *"Skenario ketiga, ATK-03: Web Credential Stuffing.*  
  > *(Sebelum mitigasi)*: Penyerang melancarkan kamus login tanpa batasan kecepatan, dan server terus merespons HTTP 401 tanpa memblokir.*  
  > *Di Security Onion Kibana, terlihat lonjakan request login dari IP 192.168.6.10.*  
  > *Mitigasi Blue Team: Kami mengintegrasikan middleware express-rate-limit dengan kuota maksimum 10 percobaan per menit.*  
  > *(Retest)*: Perhatikan terminal Kali! Percobaan 1 sampai 10 mendapat respon 401, dan tepat pada percobaan ke-11 dan ke-12, server langsung mengembalikan kode **HTTP 429 Too Many Requests**. Serangan brute force berhasil dilumpuhkan."*

---

### 🎬 Scene 4: ATK-04 — SSH Password Brute-Force (03:45 – 04:50)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Terminal Target Server $\rightarrow$ Kibana Discover rsyslog.
* **Langkah Demonstrasi:**
  * **Kondisi Sebelum Mitigasi (Loose):**
    * *Metode Manual di Target:*
      ```bash
      sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
      # Ubah: PasswordAuthentication yes | KbdInteractiveAuthentication yes
      sudo systemctl reload ssh
      ```
      *(Atau jalankan `sudo /home/kel6/loose_target.sh`)*
    * *Pengujian Red Team dari Kali:*
      ```bash
      ssh kel6@192.168.6.20
      ```
      *(Hasil: Muncul prompt `kel6@192.168.6.20's password:` — rentan terhadap brute force)*.
  * **Penerapan Mitigasi Blue Team (Hardened):**
    * *Metode Manual di Target:*
      ```bash
      sudo nano /etc/ssh/sshd_config.d/00-hardening.conf
      # Kembalikan: PasswordAuthentication no | KbdInteractiveAuthentication no
      sudo systemctl reload ssh
      ```
      *(Atau jalankan `sudo /home/kel6/harden_target.sh`)*
  * **Retest Red Team:**
    * *Pengujian Ulang dari Kali:*
      ```bash
      ssh kel6@192.168.6.20
      ```
      *(Hasil: Langsung ditolak `Permission denied (publickey)` tanpa ada prompt password)*.
  * **Verifikasi Status Fail2ban di Target:**
    ```bash
    fail2ban-client status sshd
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario keempat, ATK-04: SSH Dictionary Attack.*  
  > *(Sebelum mitigasi)*: Target mengizinkan login berbasis password, sehingga prompt kata sandi muncul dan rentan ditebak secara masif.*  
  > *Security Onion merekam lonjakan log rsyslog 'Failed password' secara real-time via UDP 514.*  
  > *Mitigasi Blue Team: Kami menerapkan pertahanan berlapis: mengunci PasswordAuthentication ke 'no', menonaktifkan root login, mewajibkan kunci publik Ed25519, dan mengaktifkan Fail2ban.*  
  > *(Retest)*: Saat penyerang mencoba login SSH dari Kali, server langsung menolak koneksi pada tahap inisiasi: **Permission denied (publickey)**. Penyerang sama sekali tidak diberi kesempatan menebak kata sandi."*

---

### 🎬 Scene 5: ATK-05 — RTSP Video Stream Hijacking (04:50 – 05:45)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Target Server memverifikasi proses video streaming berjalan aktif di background:
    ```bash
    systemctl status smartcow-stream
    ```
  * **Kondisi Sebelum Mitigasi (Loose):**
    * *Metode Manual di Target:*
      ```bash
      sudo ufw allow from 192.168.6.10 to any port 554 proto tcp
      ```
      *(Atau jalankan `sudo /home/kel6/loose_target.sh`)*
    * *Pengujian Red Team dari Kali (Port Probe & Pemutaran Live Feed):*
      ```bash
      # 1. Cek keterbukaan port RTSP:
      nc -zv 192.168.6.20 554
      # 2. Putar langsung live video feed di jendela GUI (ffplay):
      ffplay -rtsp_transport tcp -autoexit -t 5 rtsp://192.168.6.20:554/live/cow1
      # (Atau ambil snapshot bukti kebocoran frame CCTV):
      ffmpeg -y -rtsp_transport tcp -i rtsp://192.168.6.20:554/live/cow1 -vframes 1 /tmp/cctv_leaked.jpg
      ```
      *(Hasil: Port 554 `open` dan jendela pemutar video menampilkan siaran video CCTV kandang sapi secara real-time!)*
  * **Penerapan Mitigasi Blue Team (Hardened):**
    * *Metode Manual di Target:*
      ```bash
      sudo ufw delete allow from 192.168.6.10 to any port 554 proto tcp
      ```
      *(Atau jalankan `sudo /home/kel6/harden_target.sh`)*
  * **Retest Red Team:**
    * *Pengujian Ulang dari Kali:*
      ```bash
      # Uji ulang koneksi port dan pemutaran video:
      nc -zv -w 3 192.168.6.20 554
      ffplay -rtsp_transport tcp -autoexit -t 3 rtsp://192.168.6.20:554/live/cow1
      ```
      *(Hasil: `Connection timed out` — paket koneksi di-drop oleh firewall UFW, pemutaran video feed gagal total!)*.
* **Naskah Narasi (Voice-Over):**
  > *"Skenario kelima, ATK-05: Unauthorized Video Feed Interception.*  
  > *(Sebelum mitigasi)*: Port RTSP 554 terbuka sehingga feed video kamera kandang sapi dapat disedot secara ilegal dari jaringan penyerang.*  
  > *Mitigasi Blue Team: Kami membatasi akses streaming pada level kernel packet filtering menggunakan UFW. Port 554 hanya diizinkan untuk node monitoring resmi, sementara paket dari node penyerang langsung di-drop.*  
  > *(Retest)*: Saat Kali Linux mencoba probing ulang port RTSP 554, koneksi mengalami drop: **Connection timed out**. Privasi rekaman kandang sapi terlindungi sepenuhnya."*

---

### 🎬 Scene 6: ATK-06 — Config File Tampering & Permission 640 (05:45 – 06:40)
* **Tampilan Visual:** Terminal Target Server (`target-kel6`).
* **Langkah Demonstrasi:**
  * Target Server mengecek kepemilikan dan izin berkas:
    ```bash
    ls -l /opt/kel6-target/app.js /opt/mediamtx/mediamtx.yml /home/kel6/SmartCow/local/data/smartcow.db
    ```
  * Buktikan izin berkas adalah `-rw-r-----` (640) dengan kepemilikan `kel6:kel6`.
* **Naskah Narasi (Voice-Over):**
  > *"Skenario keenam, ATK-06: Privilege Abuse & Configuration Tampering.*  
  > *(Sebelum mitigasi)*: Berkas app.js, mediamtx.yml, dan database memiliki hak akses longgar yang memungkinkan pengguna lokal lain membaca atau memodifikasi kredensial.*  
  > *Mitigasi Blue Team: Kami menerapkan prinsip Least Privilege dengan menyetel permission berkas menjadi **640** dan kepemilikan ketat akun layanan kel6:kel6.*  
  > *(Retest)*: Seluruh berkas kritis kini berstatus `-rw-r-----`. Pengguna lain tanpa hak istimewa langsung diblokir sistem dengan pesan **Permission denied** jika mencoba mengubah konfigurasi."*

---

### 🎬 Scene 7: Kesimpulan & Penutup (06:40 – 07:15)
* **Tampilan Visual:** Tampilkan slide presentasi atau file `README.md` pada repositori GitHub kelompok.
* **Naskah Narasi (Voice-Over):**
  > *"Kesimpulannya, melalui siklus Purple Team Loop dari ATK-01 hingga ATK-06, seluruh serangan berhasil dimitigasi dengan tingkat keberhasilan verifikasi 100% Pass.*  
  > *Seluruh bukti tangkapan layar, slide presentasi, dan Incident Report lengkap berstandar NIST SP 800-61r2 telah kami unggah ke repositori GitHub TEK1314-2026-Kel6-KelasA.*  
  > *Sekian demonstrasi dari Kelompok 6 Kelas A. Terima kasih atas perhatian Bapak/Ibu dosen penguji."*

<div style="page-break-before: always;"></div>

## 4. Command Cheat Sheet (Panduan Lengkap Salin-Tempel Perintah)

### 4.1 Target Node (Ubuntu Server `192.168.6.20`)

#### 4.1.1 Opsi Manual (Konfigurasi Terminal Langsung)
```bash
# === A. Mengendurkan Konfigurasi (Sebelum Mitigasi - Loose) ===
# 1. SSH: Izinkan Password Login
sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config.d/00-hardening.conf
sudo sed -i 's/KbdInteractiveAuthentication no/KbdInteractiveAuthentication yes/' /etc/ssh/sshd_config.d/00-hardening.conf
sudo systemctl reload ssh

# 2. RTSP: Buka Port 554 ke Kali Linux (192.168.6.10)
sudo ufw allow from 192.168.6.10 to any port 554 proto tcp
sudo ufw status verbose

# === B. Mengunci Kembali Konfigurasi (Setelah Mitigasi - Hardened) ===
# 1. SSH: Kunci Hanya Kunci Publik (Tolak Password)
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config.d/00-hardening.conf
sudo sed -i 's/KbdInteractiveAuthentication yes/KbdInteractiveAuthentication no/' /etc/ssh/sshd_config.d/00-hardening.conf
sudo systemctl reload ssh

# 2. RTSP: Tutup Port 554 dari Kali Linux
sudo ufw delete allow from 192.168.6.10 to any port 554 proto tcp
sudo ufw status verbose
```

#### 4.1.2 Opsi Skrip Cepat (Otomatis)
```bash
# Kendurkan Target (Sebelum Mitigasi)
sudo /home/kel6/loose_target.sh

# Kunci Kembali Target (Setelah Mitigasi)
sudo /home/kel6/harden_target.sh
```

#### 4.1.3 Pemeriksaan Status Layanan & Bukti Keamanan
```bash
# 1. Cek Layanan Video Streaming FFmpeg
systemctl status smartcow-stream

# 2. Cek Status Fail2ban SSH (ATK-04)
fail2ban-client status sshd

# 3. Cek Status Efektif SSH Daemon
sshd -T | grep -E "passwordauthentication|kbdinteractiveauthentication"

# 4. Cek Izin Berkas Kritis 640 (ATK-02 & ATK-06)
ls -l /opt/kel6-target/app.js /opt/mediamtx/mediamtx.yml /home/kel6/SmartCow/local/data/smartcow.db*
```

---

### 4.2 Attacker Node (Kali Linux `192.168.6.10`)

```bash
# === 1. Pembuktian Identitas IP Attacker ===
ip a show eth0

# === 2. Pengujian ATK-01 (Port Scan Reconnaissance) ===
nmap -sV -p 22,80,554 192.168.6.20

# === 3. Pengujian ATK-02 (SQLite Database Exposure via HTTP) ===
curl -I http://192.168.6.20/smartcow.db
curl -I http://192.168.6.20/local/data/smartcow.db

# === 4. Pengujian ATK-03 (Web Brute Force - 12x Request Rate Limiting) ===
for i in {1..12}; do curl -s -o /dev/null -w "Percobaan $i: HTTP %{http_code}\n" -X POST http://192.168.6.20/api/login -H "Content-Type: application/json" -d '{"username":"admin","password":"wrongpassword"}'; done

# === 5. Pengujian ATK-04 (SSH Password Login) ===
# Sebelum mitigasi: Muncul prompt password
# Setelah mitigasi: Permission denied (publickey)
ssh kel6@192.168.6.20

# === 6. Pengujian ATK-05 (RTSP Video Streaming Access & Live Playback) ===
# Sebelum mitigasi: open & video live memutar feed CCTV di jendela Kali
nc -zv 192.168.6.20 554
ffplay -rtsp_transport tcp -autoexit -t 5 rtsp://192.168.6.20:554/live/cow1
# (Atau snapshot bukti ke file gambar):
ffmpeg -y -rtsp_transport tcp -i rtsp://192.168.6.20:554/live/cow1 -vframes 1 /tmp/cctv_leaked.jpg

# Setelah mitigasi: Connection timed out (drop paket UFW)
nc -zv -w 3 192.168.6.20 554
ffplay -rtsp_transport tcp -autoexit -t 3 rtsp://192.168.6.20:554/live/cow1
```

---

### 4.3 Monitoring SIEM Node (Security Onion `192.168.6.30`)

* **Akses Antarmuka:** Buka Chromium Browser di Security Onion $\rightarrow$ buka antarmuka **Kibana Discover**.
* **Filter Zeek Hunting:** Pilih index pattern `*:*` atau `logstash-*` untuk meninjau log koneksi Nmap (`conn.log`).
* **Pencarian Notice Alert:** Masukkan kueri filter `note: "Scan::Port_Scan"` untuk menunjukkan bukti resmi sistem deteksi intrusi Zeek.
* **Pencarian Syslog SSH:** Masukkan kueri filter `program: "sshd"` atau `Failed password` untuk menunjukkan pembuktian rekaman upaya brute force SSH secara terpusat via UDP 514.

<div style="page-break-before: always;"></div>

## 5. Tips Perekaman & Manajemen Waktu (Timeboxing Video 5–8 Menit)

1. **Tata Letak Jendela (Screen Layout):**
   * Rekomendasi: Gunakan perekam layar (OBS Studio / SimpleScreenRecorder) pada resolusi 1080p.
   * Bagi layar menjadi 2 atau 3 bagian: Terminal Kali Linux di kiri, Terminal Target di tengah, dan Browser Kibana di kanan.
2. **Kesiapan Layanan:**
   * Pastikan `smartcow-stream.service` sudah aktif (`active (running)`) sebelum menekan tombol rekam.
   * Pastikan tab Kibana Discover di Security Onion sudah terbuka dan siap di-refresh.
3. **Eksekusi Alur Purple Team:**
   * Tunjukkan **Fail/Vulnerable state** terlebih dahulu $\rightarrow$ jelaskan mengapa bahaya.
   * Tunjukkan **Langkah Mitigasi** (edit manual atau skrip) $\rightarrow$ jelaskan konsep pertahanan.
   * Tunjukkan **Pass/Hardened state (Retest)** $\rightarrow$ konfirmasikan keberhasilan blokir.
4. **Artikulasi Suara:**
   * Bacakan naskah narasi dengan tempo jelas dan tegas. Gunakan mikrofon yang jernih tanpa kebisingan latar belakang.
