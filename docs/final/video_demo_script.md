# PANDUAN DAN NASKAH REKAMAN VIDEO DEMONSTRASI PURPLE TEAM LOOP
## Mata Kuliah TEK1314 — Keamanan Siber (PBL Fase 3)
### Smart Cow Monitoring System — Replika Keamanan Backend IoT & RTSP Streaming

---

**Nomor Dokumen:** VDOC-PBL-2026-KEL06  
**Tanggal Revisi:** 26 November 2026 (Sinkronisasi Fitur Loose/Harden & Live Stream)  
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

## 2. Persiapan Sakelar Simulasi Purple Team (Target Node)

Untuk mempermudah perekaman transisi dari **Kondisi Sebelum Mitigasi (Vulnerable)** ke **Kondisi Setelah Mitigasi (Hardened)** secara live, Blue Team telah menyediakan 2 skrip sakelar otomatis di direktori `/home/kel6/` pada Target Node:

1. **`./loose_target.sh` (Kondisi Sebelum Mitigasi):**
   * Mengizinkan *Password Authentication* pada SSH.
   * Membuka port RTSP 554 untuk Kali Linux (`192.168.6.10`).
2. **`./harden_target.sh` (Kondisi Setelah Mitigasi):**
   * Mengunci SSH hanya untuk *Public Key Authentication* (menolak password).
   * Memblokir akses port 554 dari Kali Linux via firewall UFW.
3. **`smartcow-stream.service` (Layanan Streaming Latar Belakang):**
   * Daemon `ffmpeg` otomatis berjalan di latar belakang menyiarkan video sintetis CCTV kandang sapi ke MediaMTX di `rtsp://192.168.6.20:554/live/cow1`.

---

## 3. Struktur Storyboard & Naskah Narasi (Scene-by-Scene)

### 🎬 Scene 0: Pembukaan & Pembuktian Identitas IP (00:00 – 00:45)
* **Tampilan Visual:** Tampilkan jendela VirtualBox dan terminal ketiga mesin virtual berdampingan.
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
  * Target Server menjalankan: `./loose_target.sh`
  * Kali Linux menjalankan pemindaian port:
    ```bash
    nmap -sV -p 22,80,554 192.168.6.20
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario pertama, ATK-01: Network Reconnaissance. Penyerang menjalankan pemindaian port Nmap ke Target Node.*  
  > *(Sebelum mitigasi)*: Seluruh port layanan—port 22 SSH, port 80 Web, dan port 554 RTSP—berstatus OPEN tanpa firewall.*  
  > *Di Security Onion, Zeek mencatat lonjakan TCP SYN pada conn.log dan memicu Notice pemindaian port.*  
  > *Mitigasi Blue Team mengaktifkan firewall UFW dengan kebijakan default deny incoming dan menyaring port 554.*  
  > *(Retest)*: Saat Red Team mengulang Nmap setelah hardening, port 554 kini berstatus **FILTERED**. Penyerang kehilangan akses pemetaan port streaming."*

---

### 🎬 Scene 2: ATK-02 — SQLite Database Exposure via Web (01:40 – 02:35)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Kali Linux menjalankan uji request HTTP GET:
    ```bash
    curl -I http://192.168.6.20/smartcow.db
    curl -I http://192.168.6.20/local/data/smartcow.db
    ```
  * Target Server memverifikasi lokasi file lokal:
    ```bash
    ls -l /home/kel6/SmartCow/local/data/smartcow.db
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario kedua, ATK-02: Information Disclosure pada basis data SQLite.*  
  > *(Sebelum mitigasi)*: File basis data smartcow.db tersimpan di folder web publik, sehingga penyerang dapat mengunduh seluruh data sensor dan hash password.*  
  > *Mitigasi Blue Team: Kami menerapkan konsep Out-of-Web-Root dengan memindahkan file database ke direktori privat /home/kel6/SmartCow/local/data/ dan menghapus rute static file sharing.*  
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
* **Naskah Narasi (Voice-Over):**
  > *"Skenario ketiga, ATK-03: Web Credential Stuffing.*  
  > *(Sebelum mitigasi)*: Penyerang melancarkan kamus login tanpa batasan kecepatan, dan server terus merespons HTTP 401 tanpa memblokir.*  
  > *Di Security Onion Kibana, terlihat lonjakan ratusan log request login gagal dari IP 192.168.6.10.*  
  > *Mitigasi Blue Team: Kami mengintegrasikan middleware express-rate-limit dengan kuota maksimum 10 percobaan per menit.*  
  > *(Retest)*: Perhatikan terminal Kali! Percobaan 1 sampai 10 mendapat respon 401, dan tepat pada percobaan ke-11 dan ke-12, server langsung mengembalikan kode **HTTP 429 Too Many Requests**. Serangan brute force berhasil dilumpuhkan."*

---

### 🎬 Scene 4: ATK-04 — SSH Password Brute-Force (03:45 – 04:50)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Kibana Discover rsyslog $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * *Sebelum:* Jalankan `./loose_target.sh` di Target. Dari Kali ketik `ssh kel6@192.168.6.20` $\rightarrow$ muncul prompt `Password: `.
  * *Mitigasi:* Jalankan `./harden_target.sh` di Target.
  * *Retest:* Dari Kali ketik `ssh kel6@192.168.6.20` $\rightarrow$ langsung ditolak `Permission denied (publickey)`.
  * Target Server menampilkan status Fail2ban:
    ```bash
    fail2ban-client status sshd
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario keempat, ATK-04: SSH Dictionary Attack.*  
  > *(Sebelum mitigasi)*: Target mengizinkan login berbasis password, sehingga prompt password muncul dan rentan ditebak secara masif.*  
  > *Security Onion merekam lonjakan log rsyslog 'Failed password' secara real-time via UDP 514.*  
  > *Mitigasi Blue Team: Kami menerapkan pertahanan berlapis: mematikan PasswordAuthentication, menonaktifkan root login, mewajibkan SSH Key, dan mengaktifkan Fail2ban.*  
  > *(Retest)*: Saat penyerang mencoba login SSH dari Kali, server langsung menolak koneksi pada tahap handshake: **Permission denied (publickey)**. Penyerang tidak diberi kesempatan menebak kata sandi."*

---

### 🎬 Scene 5: ATK-05 — RTSP Video Stream Hijacking (04:50 – 05:45)
* **Tampilan Visual:** Terminal Kali Linux $\rightarrow$ Terminal Target Server.
* **Langkah Demonstrasi:**
  * Target Server memverifikasi proses streaming aktif:
    ```bash
    systemctl status smartcow-stream
    ```
  * *Sebelum:* Saat `./loose_target.sh` aktif, tes dari Kali:
    ```bash
    nc -zv 192.168.6.20 554
    ```
    *(Hasil: `554 (rtsp) open`)*
  * *Mitigasi:* Jalankan `./harden_target.sh` di Target.
  * *Retest:* Tes ulang dari Kali:
    ```bash
    nc -zv -w 3 192.168.6.20 554
    ```
    *(Hasil: `Connection timed out`)*
* **Naskah Narasi (Voice-Over):**
  > *"Skenario kelima, ATK-05: Unauthorized Video Feed Interception.*  
  > *(Sebelum mitigasi)*: MediaMTX membuka port RTSP 554 secara terbuka sehingga feed video kamera kandang sapi dapat disedot bebas dari jaringan.*  
  > *Mitigasi Blue Team: Kami mengaktifkan streaming internal background ffmpeg dan menerapkan aturan firewall UFW agar port 554 hanya diizinkan untuk Monitoring Node 192.168.6.30.*  
  > *(Retest)*: Saat Kali Linux mencoba mengakses port RTSP 554, koneksi mengalami drop paket: **Connection timed out**. Privasi rekaman kandang sapi terlindungi."*

---

### 🎬 Scene 6: ATK-06 — Config File Tampering & Permission 640 (05:45 – 06:40)
* **Tampilan Visual:** Terminal Target Server (`target-kel6`).
* **Langkah Demonstrasi:**
  * Target Server mengecek kepemilikan dan izin berkas:
    ```bash
    ls -l /opt/kel6-target/app.js /opt/mediamtx/mediamtx.yml /home/kel6/SmartCow/local/data/smartcow.db
    ```
* **Naskah Narasi (Voice-Over):**
  > *"Skenario keenam, ATK-06: Privilege Abuse & Configuration Tampering.*  
  > *(Sebelum mitigasi)*: Berkas app.js, mediamtx.yml, dan database memiliki hak akses longgar 644/666 yang memungkinkan proses atau pengguna lokal lain membaca atau mengubah isinya.*  
  > *Mitigasi Blue Team: Kami menerapkan prinsip Least Privilege dengan menyetel permission menjadi **640** dan kepemilikan ketat kel6:kel6.*  
  > *(Retest)*: Seluruh berkas kritis kini berstatus `-rw-r-----`. Pengguna lain maupun proses tidak berizin langsung diblokir sistem dengan pesan **Permission denied** jika mencoba mengubah konfigurasi."*

---

### 🎬 Scene 7: Kesimpulan & Penutup (06:40 – 07:15)
* **Tampilan Visual:** Tampilkan slide presentasi atau file `README.md` pada repositori GitHub.
* **Naskah Narasi (Voice-Over):**
  > *"Kesimpulannya, melalui siklus Purple Team Loop dari ATK-01 hingga ATK-06, seluruh serangan berhasil dimitigasi dengan tingkat keberhasilan verifikasi 100% Pass.*  
  > *Seluruh bukti tangkapan layar, slide presentasi, dan Incident Report lengkap berstandar NIST SP 800-61r2 telah kami unggah ke repositori GitHub TEK1314-2026-Kel6-KelasA.*  
  > *Sekian demonstrasi dari Kelompok 6 Kelas A. Terima kasih atas perhatian Bapak/Ibu dosen penguji."*

---

## 4. Command Cheat Sheet (Panduan Salin-Tempel Perintah)

### 4.1 Perintah di Target Node (Ubuntu Server `192.168.6.20`)

```bash
# === 1. Sakelar Kendurkan (Sebelum Mitigasi) ===
./loose_target.sh

# === 2. Sakelar Kunci (Setelah Mitigasi / Hardening) ===
./harden_target.sh

# === 3. Cek Status Layanan Video Stream FFmpeg ===
systemctl status smartcow-stream

# === 4. Cek Bukti Fail2ban SSH (ATK-04) ===
fail2ban-client status sshd

# === 5. Cek Bukti Izin Berkas Kritis 640 (ATK-06 & ATK-02) ===
ls -l /opt/kel6-target/app.js /opt/mediamtx/mediamtx.yml /home/kel6/SmartCow/local/data/smartcow.db*
```

### 4.2 Perintah di Attacker Node (Kali Linux `192.168.6.10`)

```bash
# === 1. Cek Identitas IP Attacker ===
ip a show eth0

# === 2. Pengujian ATK-01 (Port Scan) ===
nmap -sV -p 22,80,554 192.168.6.20

# === 3. Pengujian ATK-02 (Database Exposure) ===
curl -I http://192.168.6.20/smartcow.db
curl -I http://192.168.6.20/local/data/smartcow.db

# === 4. Pengujian ATK-03 (Web Brute Force - 12x Request) ===
for i in {1..12}; do curl -s -o /dev/null -w "Req $i: HTTP %{http_code}\n" -X POST http://192.168.6.20/api/login -H "Content-Type: application/json" -d '{"username":"admin","password":"123"}'; done

# === 5. Pengujian ATK-04 (SSH Password Login) ===
ssh kel6@192.168.6.20

# === 6. Pengujian ATK-05 (RTSP Video Access) ===
nc -zv -w 3 192.168.6.20 554
```

### 4.3 Tampilan di Monitoring Node (Security Onion `192.168.6.30`)
* Buka browser di Security Onion $\rightarrow$ buka **Kibana Discover**.
* Tampilkan **Zeek Hunting $\rightarrow$ Connections** untuk membuktikan rekaman Nmap dan koneksi web.
* Cari `note: "Scan::Port_Scan"` di pencarian untuk menunjukkan *Notice* resmi deteksi pemindaian port.
* Cari `sshd` pada log syslog untuk menunjukkan lonjakan *Failed password* saat brute force berlangsung.
