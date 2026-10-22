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
| **ATK-04** | Session Hijacking Simulation | Session aplikasi web | Browser DevTools, Burp Suite | Menguji pemakaian ulang sesi setelah logout/expired | Sesi lama tidak berlaku; akses ditolak |
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

### ATK-04 — Session Hijacking Simulation

- **Tujuan:** Memverifikasi bahwa sesi lama tidak dapat digunakan kembali setelah logout atau expiry.
- **Tools:** Browser DevTools, Burp Suite.
- **Langkah uji:**
  1. Login dengan akun pengujian sah pada browser di lab.
  2. Catat identitas sesi **secara lokal** (jangan unggah nilainya ke GitHub).
  3. Logout dari aplikasi.
  4. Uji ulang akses endpoint yang dilindungi menggunakan sesi lama dalam konteks pengujian sendiri.
  5. Ulangi secara terpisah untuk pengujian masa kedaluwarsa sesi jika memungkinkan.
- **Expected defense:** Sesi yang sudah logout/expired ditolak; atribut cookie `HttpOnly`, `SameSite`, dan `Secure` (saat HTTPS) sesuai desain.
- **Bukti:** Respons endpoint sebelum logout, sesudah logout, dan atribut cookie **tanpa memperlihatkan nilai rahasia**.

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

Isi kolom **Hasil Aktual** dan **Status** setelah pengujian dilakukan. Jangan mengisi berdasarkan asumsi.

| ID | Expected Defense | Hasil Aktual | Status | Referensi Bukti |
|---|---|---|---|---|
| ATK-01 | Port tidak diizinkan terblokir | _Belum diuji_ | Planned | _Tambahkan path evidence_ |
| ATK-02 | Endpoint dilindungi autentikasi | _Belum diuji_ | Planned | _Tambahkan path evidence_ |
| ATK-03 | Login melebihi batas mendapat `429` | _Belum diuji_ | Planned | _Tambahkan path evidence_ |
| ATK-04 | Session lama ditolak | _Belum diuji_ | Planned | _Tambahkan path evidence_ |
| ATK-05 | File SQLite tidak terekspos | _Belum diuji_ | Planned | _Tambahkan path evidence_ |
| ATK-06 | Akses RTSP tak sah ditolak | _Belum diuji_ | Planned | _Tambahkan path evidence_ |

**Pilihan status:** `Planned`, `Pass`, `Fail`, `Partial`, atau `Not Tested`. Status `Pass` berarti pertahanan bekerja sesuai kriteria, **bukan serangan berhasil**.

### Format dokumentasi tiap skenario

```text
Attack ID        : ATK-XX
Tanggal (UTC)    : YYYY-MM-DD HH:MM
Red Team IP      : xxx.xxx.xxx.xxx
Target IP        : 192.168.6.20
Preconditions    : akun uji / endpoint / stream dummy
Langkah uji      : ...
Expected result  : ...
Actual result    : ...
Status           : Pass / Fail / Partial / Not Tested
Evidence         : evidence/...
Catatan          : ...
```

### Struktur evidence yang disarankan

```text
.
├── ATTACK_PLAN.md
└── evidence/
    ├── README.md
    ├── ATK-01/
    │   ├── sebelum-port-scan.png
    │   └── sesudah-port-scan.png
    ├── ATK-02/
    ├── ATK-03/
    ├── ATK-04/
    ├── ATK-05/
    └── ATK-06/
```

Folder dan nama file di atas adalah **contoh struktur**, bukan klaim bahwa screenshot sudah tersedia. Sebelum diunggah ke GitHub publik, lakukan redaksi IP privat yang tidak perlu dipublikasikan, cookie/token/password, identitas akun, dan informasi CCTV nonpublik.

## 7. Referensi dan Keterkaitan dengan Hardening

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) — panduan pengujian autentikasi, otorisasi, dan session management.
- [MediaMTX Documentation](https://mediamtx.org/docs/) — dokumentasi RTSP, autentikasi, dan konfigurasi MediaMTX.
- [Nmap Reference Guide](https://nmap.org/book/man.html) — interpretasi status port dan metode scanning.
- [Security Onion Documentation](https://docs.securityonion.net/) — monitoring jaringan, logs, dan deteksi.

Checklist hardening Blue Team tetap menjadi dokumen terpisah; attack plan ini berfokus pada cara **memverifikasi kontrol** melalui skenario pengujian yang aman dan terukur.
