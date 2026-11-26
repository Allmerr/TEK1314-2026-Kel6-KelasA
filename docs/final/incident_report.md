# INCIDENT REPORT FINAL
## Penanganan Insiden Keamanan Siber Berdasarkan NIST SP 800-61 Rev. 2
### Sistem Monitoring Kandang Sapi Berbasis IoT (Smart Cow Monitoring System)

---

**Nomor Dokumen:** IR-PBL-2026-KEL06  
**Tanggal Publikasi:** 26 November 2026  
**Klasifikasi:** Internal / Restricted (PBL Mata Kuliah TEK1314)  
**Institusi:** Program Studi D4 Teknologi Rekayasa Komputer, Sekolah Vokasi, Institut Pertanian Bogor  
**Mata Kuliah:** TEK1314 — Keamanan Siber (Kelas A)  
**Kelompok Pelaksana:** Kelompok 6  

| Nama Anggota | NIM | Peran Tim |
|---|---|---|
| **Hodelia Victory Dorola Marbun** | J0404241100 | Incident Lead / Project Coordinator |
| **Muhammad Kevin Almer** | J0404241163 | Red Team Specialist (Attacker) |
| **Rizki Ramadhani** | J0404241144 | Blue Team Specialist (Defender / Forensics) |
| **Yuwandri Alfarizi** | J0404241010 | Blue Team Specialist (Network Security & SIEM) |

---

## 1. Ringkasan Eksekutif (Executive Summary)

Sepanjang siklus praktikum Proyek *Problem-Based Learning* (PBL) Keamanan Siber Fase 1 hingga Fase 3, Kelompok 6 merancang, menguji, dan memitigasi serangkaian vektor serangan siber terhadap infrastruktur backend replika **Smart Cow Monitoring System**. Sistem acuan merupakan solusi pemantauan kondisi ternak berbasis *Internet of Things* (IoT) dan *Computer Vision* yang mengumpulkan telemetri sensor lingkungan (suhu, kelembapan) serta *feed* video pengawasan kandang secara *real-time*.

Dalam pengujian laboratorium tertutup pada segmen jaringan virtual `192.168.6.0/24`, tim menyimulasikan enam skenario serangan agresif yang mencakup pengintaian aktif jaringan (*port scanning*), paparan basis data lokal (*SQLite information disclosure*), eksploitasi brute-force autentikasi web (*credential stuffing*), serangan kamus pada protokol manajemen (*SSH brute-force*), pembajakan *stream* kamera IP (*unauthorized RTSP video feed abuse*), serta manipulasi integritas konfigurasi (*system tampering*).

Hasil pengujian awal (*baseline*) sebelum penerapan kontrol menunjukkan kerentanan kritikal: seluruh port aplikasi terbuka tanpa penyaringan *firewall*, ketiadaan pembatasan laju (*rate limiting*) pada API login, keterpaparan *stream* RTSP publik tanpa autentikasi, serta izin akses berkas basis data yang terlalu longgar. Seluruh anomali ini berhasil dideteksi dan dicatat secara *real-time* oleh ekosistem pemantau **Security Onion** (`192.168.6.30`) melalui kombinasi Zeek Network Security Monitoring, Suricata/Snort NIDS, dan integrasi Syslog terpusat.

Mengacu kerangka kerja penanganan insiden **NIST SP 800-61 Rev. 2**, Blue Team menerapkan strategi pertahanan berlapis (*defense-in-depth*): aktivasi *packet filtering* granular via UFW, isolasi rute basis data, pembatasan laju login (10 req/menit via *express-rate-limit*), penegakan autentikasi kunci publik asimetris pada SSH, eliminasi 7 protokol MediaMTX yang tidak digunakan, serta penguncian izin berkas ke hak minimal `640`. Pada tahap verifikasi pengujian ulang oleh Red Team (*retest*), seluruh serangan berhasil digagalkan (status: **100% Pass**), memastikan ketersediaan (*availability*), kerahasiaan (*confidentiality*), dan integritas (*integrity*) data replika Smart Cow tetap terjaga utuh.

---

## 2. Deklarasi Tech Stack & Arsitektur Lab

### 2.1 Pemetaan Sistem IoT Acuan ke Target Node Replika

Sesuai prinsip kehati-hatian praktikum, simulasi serangan tidak dijalankan pada perangkat keras fisik mitra di lapangan, melainkan pada replika lingkungan *backend* laboratorium (*Target Node*):

| Komponen Arsitektur | Sistem IoT Acuan (Mitra) | Target Node Lab (Replika) | Rasionalisasi & Peran Pengujian |
|---|---|---|---|
| **Sistem Operasi** | Ubuntu Server LTS pada Edge Gateway Kandang | Ubuntu Server 26.04.1 LTS (64-bit) | Replikasi kontrol kernel, layanan systemd, dan manajemen paket. |
| **Basis Data** | SQLite lokal (`smartcow.db`) pada storage gateway | SQLite (`node:sqlite`), storage lokal | Penyimpanan riwayat sensor suhu/kelembapan dan kredensial admin. |
| **Protokol Telemetri** | HTTP REST / JSON dari mikrokontroler ESP32 | HTTP REST Port `80` (Node.js Express) | Menerima data telemetri berkala kandang sapi. |
| **Protokol Kamera** | RTSP H.264 dari IP Camera Kandang | MediaMTX RTSP Server Port `554` | Meneruskan *video feed* langsung pemantauan sapi perah. |
| **Dashboard Pengguna** | Web Admin Dashboard responsif | Web Admin Dashboard Node.js Port `80` | Antarmuka monitoring peternak & kontrol ventilasi kandang. |
| **Akses Manajemen** | SSH Administrasi Jarak Jauh | OpenSSH Server Port `22` | Akses teknisi untuk pemeliharaan gateway. |
| **Pencatatan Keamanan** | Syslog terpusat / Security Event Collector | Rsyslog UDP 514 ke Security Onion | Pengiriman audit log sistem dan otentikasi secara *live*. |

### 2.2 Topologi Jaringan Lab

Topologi jaringan dibangun di atas platform VirtualBox menggunakan mekanisme *Internal Network* terisolasi:

```
                                  +---------------------------------------+
                                  |            Jaringan Virtual           |
                                  |         (Internal: siber-kel6)        |
                                  |            192.168.6.0/24             |
                                  +-------------------+-------------------+
                                                      |
                 +------------------------------------+------------------------------------+
                 |                                    |                                    |
                 v                                    v                                    v
       +--------------------+               +--------------------+               +--------------------+
       |   Attacker Node    |               |    Target Node     |               |  Monitoring Node   |
       |  (attacker-kel6)   |               |   (target-kel6)    |               |   (monitor-kel6)   |
       |    Kali Linux      |               |   Ubuntu Server    |               |   Security Onion   |
       |    192.168.6.10    |               |    192.168.6.20    |               |    192.168.6.30    |
       +--------------------+               +--------------------+               +--------------------+
       | - Nmap / Hydra     |               | - Node.js Web :80  |               | - Zeek NIDS        |
       | - Curl / Scripts   |               | - MediaMTX :554    |               | - Snort NIDS       |
       | - Burp Suite       |               | - OpenSSH :22      |               | - Syslog-ng / ELK  |
       | - Promisc: Deny    |               | - Rsyslog Forward  |               | - Promisc: All     |
       +--------------------+               +--------------------+               +--------------------+
```

* **Attacker Node (`192.168.6.10`):** Berperan sebagai Red Team, mengeksekusi serangan terarah dari luar perimeter yang tidak terpercaya.
* **Target Node (`192.168.6.20`):** Berperan sebagai korban yang merepresentasikan gateway IoT kandang sapi.
* **Monitoring Node (`192.168.6.30`):** Berperan sebagai Blue Team SIEM/NIDS, dilengkapi dua antarmuka: `eth0` (SPAN/Sniffing mode promiskuus) dan `eth1` (Management & Syslog ingestion receiver).

---

## 3. Fase Preparation (Persiapan & Baseline Hardening)

Mengacu panduan NIST SP 800-61r2, fase persiapan bertujuan mencegah insiden dengan mengurangi permukaan serangan (*attack surface*) ke batas minimum yang diperlukan untuk operasi legal. Blue Team mengelompokkan kontrol dasar ke dalam tiga kategori (A, B, dan C) sesuai dokumen [`hardening_checklist.md`](../blueteam/hardening_checklist.md):

### 3.1 Kategori A — Access & Authentication
1. **Penonaktifan SSH Root Login:** Mengubah parameter `PermitRootLogin no` pada konfigurasi OpenSSH untuk mencegah pembajakan akun administratif tertinggi secara langsung via jaringan.
2. **Penegakan Autentikasi Asimetris (Key-Based Only):** Menonaktifkan autentikasi berbasis kata sandi (`PasswordAuthentication no`) dan mewajibkan kunci publik kriptografis (`PubkeyAuthentication yes`).
3. **Pembatasan Batas Percobaan Autentikasi:** Menerapkan `MaxAuthTries 3` guna memutus koneksi secara paksa jika terjadi percobaan login gagal berulang dalam satu sesi.
4. **Whitelist Pengguna SSH:** Membatasi akun yang diizinkan mengakses shell hanya untuk pengguna sah `kel6` (`AllowUsers kel6`).
5. **Penonaktifan X11 Forwarding:** Mematikan `X11Forwarding no` untuk mereduksi potensi *X11 display hijacking* dan *credential interception*.
6. **Integrasi Intrusion Prevention (Fail2ban):** Mengonfigurasi `jail.local` untuk memantau berkas `/var/log/auth.log`, secara dinamis memblokir alamat IP yang gagal login lebih dari ambang batas toleransi.

### 3.2 Kategori B — Service & Attack Surface
1. **Penerapan Default Deny Firewall:** Mengaktifkan *Uncomplicated Firewall* (UFW) dengan kebijakan penolakan bawaan untuk seluruh paket masuk (`ufw default deny incoming`).
2. **Segmentasi Akses Port Aplikasi:** Membatasi pembukaan port HTTP (`80`), SSH (`22`), dan NTP (`123/udp`) hanya untuk segmen internal lab yang diizinkan (`192.168.6.0/24`).
3. **Isolasi Layanan RTSP Port 554:** Membatasi akses video streaming port `554` secara eksklusif ke IP Monitoring Node (`192.168.6.30`), menolak seluruh entitas jaringan luar lainnya.
4. **Eliminasi Protokol Streaming Tak Digunakan:** Mengonfigurasi `mediamtx.yml` untuk menonaktifkan protokol RTMP, HLS, WebRTC, SRT, Web API, metrics, dan playback internal (`rtmp: no`, `hls: no`, dll.) guna menyusutkan port mendengarkan (*listening sockets*).
5. **Patch Management & Update Sistem:** Melakukan pembaruan kernel dan pustaka dependensi keamanan berkala (`apt update && apt upgrade`).

### 3.3 Kategori C — Data & Application Protection
1. **Proteksi Izin Berkas Konfigurasi:** Mengubah permission berkas kritis `mediamtx.yml` dan `app.js` menjadi `640` (`-rw-r-----`) dengan kepemilikan terbatas `kel6:kel6`.
2. **Isolasi Berkas Basis Data SQLite:** Mengunci berkas `smartcow.db`, `smartcow.db-wal`, dan `smartcow.db-shm` ke izin ketat `640` guna mencegah pembacaan data oleh proses unprivileged.
3. **Penyaringan Rute Web & Proteksi Path Traversal:** Mengonfigurasi rute Express.js untuk menolak pembacaan langsung berkas biner `.db` dan direktori induk (`../`), selalu mengembalikan kode respon `404 Not Found`.
4. **Penguatan Hashing Kredensial:** Mengonversi algoritma penyimpanan kata sandi dashboard web menggunakan fungsi derivasi kunci `scrypt` ber-salt kriptografis dengan pembatasan masa berlaku sesi maksimal 12 jam.
5. **Aplikasi Rate Limiting Login:** Memasang middleware `express-rate-limit` pada endpoint autentikasi web, membatasi request maksimal 10 kali per menit per alamat IP.

---

## 4. Fase Detection & Analysis (Deteksi & Analisis Insiden)

Pada fase ini, Red Team melancarkan enam skenario serangan agresif. Tim mencatat metode deteksi, melakukan taksonomi insiden berdasarkan standar **NIST SP 800-61r2**, serta melaksanakan analisis akar masalah (*Root Cause Analysis*):

### Matriks Taksonomi Insiden (NIST SP 800-61r2)

| Attack ID | Skenario Serangan | Kategori Insiden NIST | Dampak Fungsional | Dampak Informasi | Tingkat Keparahan |
|---|---|---|---|---|---|
| **ATK-01** | Full Port Scanning | *Scans / Probes / Reconnaissance* | Rendah (Peningkatan beban pemrosesan paket) | Tidak Ada (Tidak ada kebocoran data) | **Low** |
| **ATK-02** | SQLite Exposure & Traversal | *Unauthorized Access (Information Leak)* | Sedang (Integritas struktur data terancam) | **Kritikal** (Potensi kebocoran riwayat sensor & user hash) | **High** |
| **ATK-03** | Web Login Brute-Force | *Unauthorized Access (Brute-Force)* | Sedang (Beban CPU & potensi akun admin terbajak) | Sedang (Potensi kompromi akun admin) | **High** |
| **ATK-04** | SSH Credential Stuffing | *Unauthorized Access (Service Brute-Force)* | **Tinggi** (Risiko remote root shell takeover) | **Kritikal** (Potensi kompromi total sistem operasi) | **Critical** |
| **ATK-05** | Unauthorized RTSP Interception | *Unauthorized Access (Eavesdropping)* | Rendah (Konsumsi bandwidth video gateway) | **Tinggi** (Privasi fisik kandang sapi terpapar live) | **Medium** |
| **ATK-06** | Configuration Tampering | *System Tampering / Privilege Escalation* | **Tinggi** (Malicious reconfiguration risiko permanen) | Sedang (Paparan konfigurasi internal) | **High** |

---

### Analisis Mendalam per Skenario Serangan

#### Skenario ATK-01: Port Scanning & Surface Reconnaissance
* **Mekanisme Serangan:** Red Team mengeksekusi `nmap -sV -p- 192.168.6.20` untuk menginventarisasi seluruh port TCP/UDP yang aktif.
* **Visibilitas Deteksi (Security Onion):** Zeek NIDS mencatat ribuan baris log koneksi dengan status `conn_state: S0` (koneksi SYN tanpa balasan ACK) dan `REJ` dari IP `192.168.6.10` pada berkas `conn.log`. Snort menghasilkan alert signature klasifikasi *Attempted Information Leak / Portscan*.
* **Root Cause Analysis:** Firewall internal dinonaktifkan secara default pada instalasi awal sistem operasi, membiarkan port mendengarkan (*listening sockets*) merespon secara bebas ke seluruh host pada subnet yang sama.

#### Skenario ATK-02: Unauthorized SQLite Database Exposure
* **Mekanisme Serangan:** Red Team mengirimkan serangkaian request HTTP GET langsung ke endpoint basis data (`/smartcow.db`, `/data/smartcow.db`, `../smartcow.db`).
* **Visibilitas Deteksi (Security Onion):** Web server mencatat URI request berbahaya ke dalam access log lokal yang diteruskan melalui Rsyslog ke Security Onion (`logstash-syslog-*`). Zeek `http.log` memetakan request mencurigakan berulang ke berkas dengan ekstensi `.db`.
* **Root Cause Analysis:** Framework aplikasi Node.js Express sebelumnya menyajikan berkas statis dari direktori kerja aplikasi tanpa filtering whitelist ekstensi, sehingga berkas basis data SQLite yang disimpan di direktori lokal terpetakan ke ruang publik HTTP.

#### Skenario ATK-03: Web Admin Login Brute-Force Simulation
* **Mekanisme Serangan:** Penyerang mengirimkan 12+ request HTTP POST berurutan dalam hitungan detik ke endpoint `/api/login` menggunakan kamus kredensial umum (*admin/password*, *admin/123456*).
* **Visibilitas Deteksi (Security Onion):** Terpantau lonjakan frekuensi HTTP POST dalam rentang waktu singkat dari alamat IP penyerang pada Zeek `http.log` dan log otentikasi aplikasi di Elasticsearch.
* **Root Cause Analysis:** Ketiadaan mekanisme *rate-limiting* di sisi server web yang membatasi ambang batas request per satuan waktu dari satu alamat IP asal, memungkinkan serangan kamus otomatis berkecepatan tinggi tanpa hambatan.

#### Skenario ATK-04: SSH Brute-Force & Password Spraying
* **Mekanisme Serangan:** Penyerang menjalankan perkakas otomatisasi Hydra dan koneksi SSH interaktif untuk menebak kata sandi pengguna `root` dan pengguna layanan lainnya.
* **Visibilitas Deteksi (Security Onion):** Syslog Target Node mengirimkan pesan kegagalan otentikasi bertubi-tubi: `Failed password for invalid user ... from 192.168.6.10 port ... ssh2`. Zeek menandai anomali frekuensi handshake SSH yang terputus tidak wajar.
* **Root Cause Analysis:** Konfigurasi default OpenSSH mengizinkan autentikasi berbasis kata sandi (`PasswordAuthentication yes`) dan mengizinkan login pengguna root (`PermitRootLogin yes`) tanpa batas percobaan sesi yang ketat.

#### Skenario ATK-05: Unauthorized RTSP Video Feed Access
* **Mekanisme Serangan:** Red Team mencoba menginspeksi dan menarik aliran video dari port multimedia menggunakan utilitas `ffprobe -rtsp_transport tcp rtsp://192.168.6.20:554/live/stream` serta memindai port layanan MediaMTX (1935, 8888, 8889, 9997).
* **Visibilitas Deteksi (Security Onion):** Zeek mendeteksi inisiasi sesi protokol RTSP pada port non-HTTP dan aliran data media berbasis TCP.
* **Root Cause Analysis:** Konfigurasi bawaan MediaMTX mengaktifkan seluruh *protocol bindings* (RTMP, HLS, WebRTC, API kontrol) yang sebenarnya tidak diperlukan oleh klien pemantau kandang, serta tidak menerapkan filter IP pada level layer transpor.

#### Skenario ATK-06: Local File Tampering & Permission Abuse
* **Mekanisme Serangan:** Red Team melakukan enumerasi lokal terhadap izin berkas konfigurasi kritis (`mediamtx.yml`, `app.js`) untuk mencari celah penulisan oleh pengguna non-root.
* **Visibilitas Deteksi (Security Onion):** Audit izin sistem operasi memperlihatkan hak akses `644` dan `666` yang memicu temuan kelemahan kepatuhan (*compliance deficiency*).
* **Root Cause Analysis:** Pembuatan berkas aplikasi dilakukan di bawah *umask* default tanpa penerapan kebijakan pembatasan hak akses berbasis prinsip *least privilege*.

---

## 5. Fase Containment, Eradication & Recovery

Mengacu siklus Tahap 4 Purple Team Loop, Blue Team segera menerapkan langkah penahanan, pemusnahan kerentanan, dan pemulihan sistem secara terkoordinasi:

```
+----------------------------------------------------------------------------------------------------+
|                                    ALUR REMEDIASI BLUE TEAM                                        |
+----------------------------------------------------------------------------------------------------+
|  [Containment / Penahanan]                                                                         |
|  - Mengaktifkan UFW default deny untuk menghentikan seluruh probing liar seketika.                |
|  - Menutup port-port MediaMTX yang tidak digunakan (RTMP, WebRTC, HLS, API).                       |
+----------------------------------------------------------------------------------------------------+
                                                  │
                                                  ▼
+----------------------------------------------------------------------------------------------------+
|  [Eradication / Pemusnahan Akar Celah]                                                             |
|  - SSH: Menghapus autentikasi password, mewajibkan kunci publik, blokir root login.                |
|  - Web: Memasang middleware express-rate-limit (10 req/menit), enkripsi scrypt, cookie 12 jam.     |
|  - DB & Files: Membatasi permission ke 640, memblokir seluruh rute URL database -> 404.           |
|  - RTSP: Mengunci port 554 hanya untuk IP terpercaya (192.168.6.30).                               |
+----------------------------------------------------------------------------------------------------+
                                                  │
                                                  ▼
+----------------------------------------------------------------------------------------------------+
|  [Recovery / Pemulihan & Normalisasi]                                                              |
|  - Restart layanan systemd (ssh, mediamtx, smartcow-web, ufw).                                     |
|  - Verifikasi integritas rsyslog ke Security Onion (UDP 514) & konfirmasi health Elasticsearch.     |
+----------------------------------------------------------------------------------------------------+
```

### Rincian Kontrol Teknis yang Diterapkan:
1. **Firewall Ruleset (UFW):**
   ```bash
   sudo ufw default deny incoming
   sudo ufw default allow outgoing
   sudo ufw allow from 192.168.6.0/24 to any port 22 proto tcp
   sudo ufw allow from 192.168.6.0/24 to any port 80 proto tcp
   sudo ufw allow from 192.168.6.0/24 to any port 123 proto udp
   sudo ufw allow from 192.168.6.30 to any port 554 proto tcp
   sudo ufw enable
   ```
2. **OpenSSH Server Hardening (`/etc/ssh/sshd_config.d/hardening.conf`):**
   ```text
   PermitRootLogin no
   PasswordAuthentication no
   PubkeyAuthentication yes
   MaxAuthTries 3
   AllowUsers kel6
   X11Forwarding no
   ```
3. **Application Layer Rate-Limiter (`app.js`):**
   ```javascript
   const rateLimit = require('express-rate-limit');
   const loginLimiter = rateLimit({
       windowMs: 1 * 60 * 1000, // 1 menit
       max: 10, // Maksimal 10 request per IP
       message: { error: 'Terlalu banyak percobaan login. Silakan coba lagi nanti.' },
       statusCode: 429
   });
   app.use('/api/login', loginLimiter);
   ```
4. **MediaMTX Protocol Minimization (`mediamtx.yml`):**
   ```yaml
   rtspAddress: :554
   rtmp: no
   hls: no
   webrtc: no
   srt: no
   api: no
   metrics: no
   ```
5. **Filesystem DAC Permission Tightening:**
   ```bash
   chmod 640 /opt/kel6-target/app.js /opt/mediamtx/mediamtx.yml
   chmod 640 /home/kel6/SmartCow/local/data/smartcow.db*
   chown -R kel6:kel6 /opt/kel6-target/ /home/kel6/SmartCow/local/data/
   ```

---

## 6. Fase Verifikasi (Pengujian Ulang & Validasi Empiris)

Sesuai Tahap 5 Purple Team Loop, Red Team mengulangi seluruh serangan menggunakan perkakas yang sama dari Kali Linux (`192.168.6.10`) untuk memastikan bahwa kontrol pertahanan berfungsi secara efektif:

### Matriks Validasi Empiris Sebelum vs Sesudah Mitigasi

| Attack ID | Parameter Uji | Respon Sebelum Mitigasi | Respon Setelah Mitigasi | Status Verifikasi | Referensi Bukti Screenshot |
|---|---|---|---|---|---|
| **ATK-01** | Pemindaian Nmap Port 554 | `554/tcp open rtsp` | `554/tcp filtered rtsp` | **Pass** | [sesudah-5-pindai-nmap.png](../blueteam/evidence/sesudah-5-pindai-nmap.png) |
| **ATK-02** | HTTP GET `/smartcow.db` | Data biner terbaca / terekspos | `HTTP/1.1 404 Not Found` | **Pass** | [sesudah-18b-db-404.png](../blueteam/evidence/sesudah-18b-db-404.png) |
| **ATK-03** | Brute-force Login ke-11 | `HTTP/1.1 200 OK` (Diproses terus) | `HTTP/1.1 429 Too Many Requests` | **Pass** | [sesudah-19b-ratelimit.png](../blueteam/evidence/sesudah-19b-ratelimit.png) |
| **ATK-04** | SSH Password Attempt | Password prompt muncul / login root | `Permission denied (publickey)` | **Pass** | [sesudah-6-login-ditolak.png](../blueteam/evidence/sesudah-6-login-ditolak.png) |
| **ATK-05** | Akses RTSP dari Penyerang | Stream video berhasil ditarik | `Connection timed out` (Firewall Drop) | **Pass** | [sesudah-4b-port-listen.png](../blueteam/evidence/sesudah-4b-port-listen.png) |
| **ATK-06** | Izin Berkas Konfigurasi | Izin publik `666` / `644` | Terkunci ketat `-rw-r-----` (`640`) | **Pass** | [sesudah-7a-izin-berkas.png](../blueteam/evidence/sesudah-7a-izin-berkas.png) |

---

## 7. Post-Incident Activity & Lessons Learned

### 7.1 Evaluasi & Pembelajaran Tim
1. **Pentingnya Visibilitas Monitoring Terpusat:** Pengujian membuktikan bahwa konfigurasi keamanan tidak dapat diverifikasi tanpa pemantauan log yang aktif. Deteksi dini serangan brute-force dan port scanning sangat bergantung pada integrasi syslog yang stabil ke Security Onion.
2. **Prinsip Default Deny:** Membiarkan konfigurasi jaringan default tanpa firewall adalah risiko terbesar pada perangkat IoT. Penerapan aturan *whitelist* spesifik terbukti efektif mematikan 80% vektor pengintaian Red Team.
3. **Pemisahan Peran Red vs Blue:** Kolaborasi Purple Team memungkinkan iterasi cepat; setiap celah yang ditemukan penyerang dapat segera diformulasikan solusinya oleh defender dalam siklus yang terukur.

### 7.2 Rekomendasi Lanjutan untuk Sistem IoT Nyata (Mitra)
Di luar lingkup simulasi replika praktikum, tim merekomendasikan langkah penguatan strategis berikut untuk sistem Smart Cow fisik di lapangan:

1. **Enkripsi Transport Layer (TLS/HTTPS Obligatory):** Menerapkan sertifikat TLS pada dashboard web (port 443) dan enkripsi stream video (RTSP over TLS / RTSPS port 322) guna mencegah intersepsi data man-in-the-middle di jaringan kandang.
2. **Mutual Authentication (mTLS) pada Sensor ESP32:** Mikrokontroler pengirim data suhu dan kelembapan wajib menggunakan autentikasi sertifikat dua arah atau token API berotasi sebelum data diterima oleh gateway backend.
3. **Zero-Trust Network Access (ZTNA) / VPN Terisolasi:** Gateway kandang sebaiknya tidak mengekspos port administrasi ke jaringan publik. Akses remote teknisi wajib melalui *tunnel* VPN terenkripsi (misalnya WireGuard) dengan autentikasi multi-faktor (MFA).
4. **Enkripsi Data At-Rest:** Berkas basis data SQLite pada perangkat gateway di kandang perlu dienkripsi pada level filesystem (misalnya menggunakan LUKS atau SQLCipher) untuk memitigasi risiko pencurian fisik perangkat edge.
5. **Secure Boot & Firmware Integrity Monitoring:** Mengaktifkan verifikasi penandatanganan kriptografis pada firmware ESP32 dan verifikasi integritas berkas sistem operasi gateway secara berkala.

---

## 8. Lampiran & Tautan Artefak GitHub

Seluruh dokumen teknis, skrip pengujian, berkas konfigurasi, dan bukti tangkapan layar digital tersimpan secara transparan pada repositori proyek:

* **Repositori Resmi Proyek:** [GitHub - TEK1314-2026-Kel6-KelasA](https://github.com/Allmerr/TEK1314-2026-Kel6-KelasA)
* **Dokumen Perencanaan Jaringan:**
  * Topologi Lab: [`docs/design/topology.png`](../design/topology.png)
  * Rencana Alokasi IP: [`docs/design/ip_plan.md`](../design/ip_plan.md)
  * Deklarasi Tech Stack: [`docs/presentation-minggu9/tech_stack_declaration.md`](../presentation-minggu9/tech_stack_declaration.md)
* **Dokumen Pengujian & Mitigasi:**
  * Attack Plan Red Team: [`docs/redteam/attack_plan.md`](../redteam/attack_plan.md)
  * Hardening Checklist Blue Team: [`docs/blueteam/hardening_checklist.md`](../blueteam/hardening_checklist.md)
  * Purple Team Log Lengkap: [`docs/purple-team-log.md`](../purple-team-log.md)
  * Direktori Bukti Forensik: [`docs/blueteam/evidence/`](../blueteam/evidence/)
* **Artefak Final Fase 3:**
  * Incident Report Final (PDF): [`docs/final/incident_report.pdf`](incident_report.pdf)
  * Panduan & Naskah Video Demo (PDF): [`docs/final/video_demo_script.pdf`](video_demo_script.pdf)
  * Video Demo Purple Team Loop: [`docs/final/video_demo.mp4`](video_demo.mp4) (atau tautan streaming di README)
  * Slide Presentasi Final (PDF): [`docs/presentation-final/slides.pdf`](../presentation-final/slides.pdf)
