# Attack Plan — Smart Cow Monitoring System

> **PBL Keamanan Siber | Kelompok 6**  
> **Fokus:** Pengujian keamanan website monitoring dan RTSP streaming pada lingkungan lab terkendali.  
> **Status:** Rencana pengujian (*planned*) — hasil serangan belum diklaim berhasil atau gagal.

## 1. Gambaran Umum

Smart Cow Monitoring System merupakan proyek pemantauan kondisi kandang berbasis IoT dan Computer Vision. Dalam PBL Keamanan Siber, pengujian **dibatasi pada infrastruktur website monitoring dan layanan RTSP**, bukan ESP32, sensor, model machine learning, atau aktuator fisik.

Pengujian dilakukan dengan pendekatan **Red Team vs Blue Team** untuk mengidentifikasi permukaan serangan, memverifikasi konfigurasi hardening, serta mengumpulkan bukti sebelum dan sesudah penerapan kontrol keamanan.

### Tujuan

1. Memetakan layanan yang dapat dijangkau dari jaringan pengujian.
2. Mengevaluasi autentikasi, otorisasi, pembatasan login, dan sesi pada website monitoring.
3. Memastikan berkas database SQLite tidak terekspos melalui HTTP.
4. Menguji pembatasan akses dan autentikasi streaming RTSP melalui MediaMTX.
5. Mendokumentasikan hasil *before vs after hardening* dan log keamanan yang relevan.

## 2. Scope dan Lingkungan Pengujian

| Parameter | Keterangan |
|---|---|
| Target IP | `192.168.6.20` |
| Hostname | `target-kel6` |
| Sistem operasi | Ubuntu Server 26.04.1 LTS |
| Website monitoring | Node.js + SQLite |
| Layanan web | HTTP `80` (sesuai konfigurasi awal; HTTPS `443` jika diaktifkan) |
| Streaming | MediaMTX / RTSP `554` |
| Red Team | Kali Linux (host/IP disesuaikan dengan lab) |
| Blue Team | Target Ubuntu Server + Security Onion |
| Security Onion | `192.168.6.30` **(konfirmasi IP dan perannya)** |
| Metode | Pengujian terkontrol, verifikasi log, dan perbandingan sebelum–sesudah hardening |

**Dalam scope:** web route/API monitoring, halaman login, sesi login, paparan file SQLite dari web, konektivitas RTSP, autentikasi RTSP, serta aturan firewall terkait.

**Di luar scope:** pengambilalihan ESP32, manipulasi data sensor nyata, kontrol pompa/solenoid, serangan DoS/DDoS, phishing, malware, eksploitasi pihak ketiga, dan serangan terhadap CCTV produksi yang tidak menjadi bagian lab.

### Aturan pelaksanaan

- Jalankan hanya pada host dan akun yang memang diizinkan oleh tim/dosen.
- Gunakan **akun uji, video dummy**, dan snapshot/backup VM bila tersedia.
- Batasi intensitas request; jangan menggunakan brute-force masif atau mengganggu layanan produksi.
- Jangan menyimpan password, token, cookie sesi, atau rekaman pribadi dalam repository publik.
- Untuk pengujian kondisi *sebelum hardening*, gunakan snapshot lab khusus; **jangan menurunkan keamanan server aktif** demi menghasilkan bukti.
- Catat waktu pengujian, IP sumber, konfigurasi yang relevan, hasil aktual, dan bukti. Screenshot target menggunakan **UTC**.

## 3. Attack Scenario Matrix

| ID | Skenario | Target | Metode / Tools | Tujuan | Expected Defense |
|---|---|---|---|---|---|
| **ATK-01** | Port Scanning | Ubuntu Server | Nmap | Mengidentifikasi port/layanan yang bisa dijangkau | UFW memblokir koneksi yang tidak diizinkan |
| **ATK-02** | Unauthorized Web Access | Website / API monitoring | Browser, `curl`, Burp Suite | Menguji akses endpoint sensitif tanpa autentikasi | `401`, `403`, atau redirect login; tidak ada data sensitif bocor |
| **ATK-03** | Login Brute-Force Simulation | Login website | Burp Suite / request uji terbatas | Memverifikasi pembatasan percobaan login | HTTP `429 Too Many Requests` sesuai kebijakan rate limit |
| **ATK-04** | SSH Password Brute-Force | Layanan OpenSSH :22 | OpenSSH Client, Hydra | Menguji keamanan akses remote management SSH terhadap tebakan password | Password authentication dinonaktifkan (`PasswordAuthentication no`); hanya public key yang diterima (`Permission denied (publickey)`) |
| **ATK-05** | Database File Exposure | SQLite melalui HTTP | Browser, `curl` | Memastikan file DB tidak dapat diunduh dari web | `403` / `404`, file DB tidak terekspos |
| **ATK-06** | Unauthorized RTSP Access | MediaMTX / RTSP | VLC, `ffprobe` | Menguji akses stream anonim dan pembatasan IP | IP tak diizinkan diblokir; klien tanpa izin RTSP ditolak |

> **Catatan:** *Expected defense* adalah kriteria keberhasilan, **bukan hasil pengujian yang telah diverifikasi**. Dokumentasikan hasil aktual di bagian [Hasil Pengujian](#6-hasil-pengujian).

## 4. Detail Skenario Pengujian

### ATK-01 — Port Scanning

- **Tujuan:** Mengidentifikasi permukaan layanan yang dapat dijangkau dari mesin Red Team.
- **Target:** `192.168.6.20`.
- **Tools:** Nmap.
- **Langkah uji:**
  1. Pastikan alamat target dan IP Red Team sesuai cakupan lab.
  2. Dari Kali Linux, jalankan pemindaian terbatas pada port yang menjadi bagian arsitektur:

     ```bash
     nmap -sT -Pn -p 22,80,443,554 --reason 192.168.6.20
     ```

  3. Bandingkan status port dengan kebijakan UFW dan layanan yang memang diharapkan aktif.
- **Expected defense:** Hanya port yang diizinkan dari IP Red Team yang dapat dijangkau.
- **Bukti:** Hasil Nmap, `sudo ufw status numbered`, dan daftar listener (`sudo ss -tulpen`).

### ATK-02 — Unauthorized Web Access

- **Tujuan:** Memastikan endpoint monitoring yang sensitif mensyaratkan autentikasi/otorisasi.
- **Tools:** Browser, `curl`, Burp Suite.
- **Langkah uji:**
  1. Tentukan endpoint monitoring yang benar-benar ada (contoh placeholder: `/ENDPOINT_MONITORING`).
  2. Akses endpoint tanpa cookie/login:

     ```bash
     curl -i http://192.168.6.20/ENDPOINT_MONITORING
     ```

  3. Bandingkan dengan akses menggunakan akun uji yang sah.
  4. Pastikan respons unauthenticated tidak berisi data monitoring sensitif.
- **Expected defense:** Respons `401`, `403`, atau redirect ke login untuk resource yang dilindungi.
- **Bukti:** Request/response HTTP tanpa login dan setelah login (sensor data/token sensitif disamarkan).

### ATK-03 — Login Brute-Force Simulation

- **Tujuan:** Memvalidasi kebijakan maksimal **10 percobaan login per menit per IP**.
- **Tools:** Browser, Burp Suite, atau request HTTP uji terbatas.
- **Langkah uji:**
  1. Gunakan **akun pengujian milik tim** dan lingkungan lab.
  2. Lakukan hingga **11 percobaan** dengan password pengujian yang salah, secara berurutan dalam satu jendela rate limit.
  3. Amati HTTP status tiap percobaan dan log aplikasi.
  4. Tunggu jendela rate limit berakhir sebelum pengujian ulang.
- **Expected defense:** Request melebihi ambang batas menerima `429 Too Many Requests`.
- **Bukti:** Riwayat 11 percobaan, status `429`, dan log login/rate-limit. Pastikan rate limiter tidak sudah terpicu **sebelum** uji dimulai.

### ATK-04 — SSH Password Brute-Force & Remote Access Control

- **Tujuan:** Memvalidasi penegakan autentikasi asimetris berbasis kunci publik dan penonaktifan total autentikasi kata sandi pada layanan remote OpenSSH port 22.
- **Target:** `192.168.6.20:22` (OpenSSH Server).
- **Tools:** OpenSSH Client (`ssh`), Hydra, Nmap.
- **Langkah uji:**
  1. Dari Kali Linux (`192.168.6.10`), coba lakukan koneksi SSH ke akun `kel6` dan `root` dengan memaksa metode autentikasi password:
     ```bash
     ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no kel6@192.168.6.20
     ```
  2. Amati apakah server meminta prompt password atau menolak koneksi secara langsung.
  3. Uji login menggunakan akun `root` untuk memastikan direct root login diblokir (`PermitRootLogin no`).
- **Expected defense:** Koneksi ditolak seketika oleh daemon SSH dengan pesan `Permission denied (publickey)`; prompt password tidak pernah ditampilkan; upaya brute-force kata sandi gagal secara struktural.
- **Bukti:** Pesan penolakan terminal Kali Linux, file konfigurasi `/etc/ssh/sshd_config.d/00-hardening.conf`, dan log auth di `/var/log/auth.log` serta Kibana Security Onion.

### ATK-05 — Database File Exposure

- **Tujuan:** Mengonfirmasi SQLite tidak dipublikasikan melalui web root atau static file serving.
- **Tools:** Browser, `curl`.
- **Langkah uji:**
  1. Blue Team menginformasikan **path URL pengujian yang aman** berdasarkan struktur aplikasi, tanpa mempublikasikan path host yang sensitif.
  2. Lakukan request GET terhadap path web yang berpotensi mengekspose database pengujian.
  3. Periksa apakah server mengirim file database atau justru menolaknya.
  4. Verifikasi permission dan lokasi DB pada target bersama Blue Team.
- **Expected defense:** Database tidak dapat diunduh (`403`/`404` atau respons aman yang setara).
- **Bukti:** HTTP response, file permissions, dan konfigurasi static serving terkait.

### ATK-06 — Unauthorized RTSP Access

- **Tujuan:** Memvalidasi kontrol akses network **dan** autentikasi RTSP sebagai dua lapis terpisah.
- **Tools:** VLC, FFprobe, MediaMTX logs.
- **Langkah uji:**
  1. Gunakan video dummy dan path stream lab yang disepakati (`NAMA_STREAM` sebagai placeholder).
  2. Dari klien Red Team yang **tidak diizinkan firewall**, uji koneksi RTSP; ekspektasinya koneksi terblokir.
  3. Dari **klien pengujian yang diizinkan firewall**, coba akses stream **tanpa kredensial**:

     ```bash
     ffprobe -v error -rtsp_transport tcp \
       rtsp://192.168.6.20:554/NAMA_STREAM
     ```

  4. Bandingkan dengan akses dari klien yang memiliki izin dan kredensial **read-only** yang sah.
  5. Periksa log MediaMTX dan aturan UFW.
- **Expected defense:** IP yang diblokir firewall tidak dapat terkoneksi; klien tanpa kredensial yang bisa menjangkau server tetap ditolak oleh autentikasi RTSP; klien berizin dapat membaca stream.
- **Bukti:** Keluaran `ffprobe`, respons akses dari dua kelas IP, log MediaMTX, dan aturan UFW.

**Perhatian pada `192.168.6.30`:** Checklist hardening mencatat port RTSP `554` hanya boleh diakses dari `192.168.6.30`, yang juga disebut sebagai IP **Security Onion**. Konfirmasi kembali peran host ini. Bila ia hanya collector/monitoring trafik, **bukan** server CV/reader RTSP, aturan firewall perlu ditinjau. Monitoring jaringan dengan SPAN/TAP tidak mengharuskan Security Onion diberi izin membaca RTSP secara langsung.

## 5. Tahapan Red Team vs Blue Team

| Fase | Red Team | Blue Team | Output |
|---|---|---|---|
| **1. Persiapan** | Menetapkan IP sumber, akun uji, endpoint, dan stream dummy | Menyiapkan snapshot, logging, serta dokumentasi konfigurasi | Scope dan baseline |
| **2. Baseline Test** | Menjalankan skenario di snapshot sebelum hardening (jika tersedia) | Mengumpulkan konfigurasi sebelum | Bukti `sebelum-*` |
| **3. Hardening** | Tidak melakukan perubahan pada target | Menerapkan konfigurasi firewall, web, sesi, dan RTSP | Catatan konfigurasi |
| **4. Retest** | Mengulang pengujian terkontrol | Memantau status kontrol | Bukti `sesudah-*` |
| **5. Evaluasi** | Membandingkan perilaku sistem | Mengkorelasikan log dan alert | Laporan hasil dan temuan |

> Tidak semua percobaan akan menghasilkan alert Security Onion secara otomatis. Pastikan sumber log dan visibilitas trafik (misalnya SPAN/TAP atau pengaturan jaringan virtual) tersedia; gunakan log aplikasi/MediaMTX untuk melengkapi bukti.

## 6. Hasil Pengujian

Seluruh skenario pengujian telah dieksekusi secara empiris dalam siklus Purple Team Loop (Minggu 9 s/d Minggu 11). Hasil pengujian aktual membuktikan seluruh kontrol mitigasi Blue Team berfungsi efektif menggagalkan serangan (*Status: Pass*).

| ID | Expected Defense | Hasil Aktual | Status | Referensi Bukti |
|---|---|---|---|---|
| **ATK-01** | Port tidak diizinkan terblokir | Port 554 RTSP berstatus `filtered` (drop) dari Kali Linux | **Pass** | [Bukti ATK-01](evidence/ATK-01/sesudah-port-scan.png) |
| **ATK-02** | Endpoint dilindungi autentikasi | Akses tanpa cookie menghasilkan HTTP 401 Unauthorized | **Pass** | [Bukti ATK-02](evidence/ATK-02/sesudah-unauthorized.png) |
| **ATK-03** | Login melebihi batas mendapat `429` | Percobaan ke-11 dan ke-12 menghasilkan HTTP 429 Too Many Requests | **Pass** | [Bukti ATK-03](evidence/ATK-03/sesudah-ratelimit-429.png) |
| **ATK-04** | Password ditolak (kunci publik wajib) | Login password ditolak: `Permission denied (publickey)` | **Pass** | [Bukti ATK-04](evidence/ATK-04/sesudah-ssh-keyonly.png) |
| **ATK-05** | File SQLite tidak terekspos | Request HTTP GET `smartcow.db` merespons HTTP 404 Not Found | **Pass** | [Bukti ATK-05](evidence/ATK-05/sesudah-db-404.png) |
| **ATK-06** | Akses RTSP tak sah ditolak | Probing dan pemutaran RTSP Kali Linux mengalami `Connection timed out` | **Pass** | [Bukti ATK-06](evidence/ATK-06/sesudah-rtsp-filtered.png) |

**Ringkasan Status Akhir:** 6 Skenario Teruji | **6 Pass (100% Berhasil Dimonitor & Dimitigasi)** | 0 Fail | 0 Partial.

### Dokumentasi Lengkap Verifikasi Hasil Pengujian Empiris

```text
Attack ID        : ATK-01
Tanggal (UTC)    : 2026-11-25 04:19
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : Target Node dalam segmen terisolasi siber-kel6 (192.168.6.0/24)
Langkah uji      : nmap -sV -p 22,80,554 192.168.6.20
Expected result  : Port 554 RTSP tersaring firewall dan berstatus FILTERED
Actual result    : Port 554/tcp filtered rtsp (drop paket UFW), port 22 & 80 open
Status           : Pass
Evidence         : evidence/ATK-01/sesudah-port-scan.png
Catatan          : Zeek pada Security Onion merekam lonjakan TCP SYN pada conn.log

Attack ID        : ATK-02
Tanggal (UTC)    : 2026-11-25 04:20
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : Layanan web Node.js port 80 aktif
Langkah uji      : curl -i http://192.168.6.20/api/me (tanpa session cookie)
Expected result  : Request ditolak dengan kode respon HTTP 401 Unauthorized
Actual result    : Server merespons HTTP/1.1 401 Unauthorized ({"error":"Belum masuk."})
Status           : Pass
Evidence         : evidence/ATK-02/sesudah-unauthorized.png
Catatan          : Endpoint data terlindungi autentikasi session cookie

Attack ID        : ATK-03
Tanggal (UTC)    : 2026-11-25 04:21
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : Endpoint /api/login aktif dengan express-rate-limit
Langkah uji      : Loop 12 request POST berturut-turut dengan kredensial salah
Expected result  : Request melebihi 10 kali per menit diblokir dengan HTTP 429
Actual result    : Request ke-11 dan ke-12 mengembalikan HTTP/1.1 429 Too Many Requests
Status           : Pass
Evidence         : evidence/ATK-03/sesudah-ratelimit-429.png
Catatan          : Serangan kamus login otomatis berhasil dilumpuhkan

Attack ID        : ATK-04
Tanggal (UTC)    : 2026-11-25 04:22
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : OpenSSH port 22 dikonfigurasi PasswordAuthentication no
Langkah uji      : ssh kel6@192.168.6.20 (tanpa kunci publik sah)
Expected result  : Koneksi ditolak pada handshake tanpa prompt password
Actual result    : Ditanggapi seketika: Permission denied (publickey)
Status           : Pass
Evidence         : evidence/ATK-04/sesudah-ssh-keyonly.png
Catatan          : Target mencatat failed preauth di syslog dan Fail2ban aktif

Attack ID        : ATK-05
Tanggal (UTC)    : 2026-11-25 04:23
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : Database SQLite dipindahkan ke direktori privat local/data/
Langkah uji      : curl -I http://192.168.6.20/smartcow.db
Expected result  : File database tidak dapat diunduh via web publik (HTTP 404)
Actual result    : Server merespons HTTP/1.1 404 Not Found
Status           : Pass
Evidence         : evidence/ATK-05/sesudah-db-404.png
Catatan          : File database aman dengan permission 640 di luar web root

Attack ID        : ATK-06
Tanggal (UTC)    : 2026-11-25 04:24
Red Team IP      : 192.168.6.10
Target IP        : 192.168.6.20
Preconditions    : Port 554 di-filter UFW hanya untuk Security Onion (192.168.6.30)
Langkah uji      : nc -zv -w 3 192.168.6.20 554 && ffplay rtsp://192.168.6.20:554/live/cow1
Expected result  : Koneksi probing dan video feed dari Kali Linux ditolak
Actual result    : Connection to 192.168.6.20 554 failed: Connection timed out
Status           : Pass
Evidence         : evidence/ATK-06/sesudah-rtsp-filtered.png
Catatan          : Siaran CCTV aman dari intersepsi liar penyerang
```

## 7. Referensi dan Keterkaitan dengan Hardening

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) — panduan pengujian autentikasi, otorisasi, dan session management.
- [MediaMTX Documentation](https://mediamtx.org/docs/) — dokumentasi RTSP, autentikasi, dan konfigurasi MediaMTX.
- [Nmap Reference Guide](https://nmap.org/book/man.html) — interpretasi status port dan metode scanning.
- [Security Onion Documentation](https://docs.securityonion.net/) — monitoring jaringan, logs, dan deteksi.

Checklist hardening Blue Team tetap menjadi dokumen terpisah; attack plan ini berfokus pada cara **memverifikasi kontrol** melalui skenario pengujian yang aman dan terukur.
