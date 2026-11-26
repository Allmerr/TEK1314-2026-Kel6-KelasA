# FINAL PRESENTATION: INCIDENT REPORT & PURPLE TEAM DEFENSE
## Smart Cow Monitoring System — Replika Keamanan Backend IoT
**Mata Kuliah:** TEK1314 — Keamanan Siber (Kelas A)  
**Institusi:** D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB  
**Kelompok:** Kelompok 6  

---

### Profil Tim Kelompok 6
1. **Hodelia Victory Dorola Marbun** (NIM: J0404241100) — *Project Lead & Incident Coordinator*
2. **Muhammad Kevin Almer** (NIM: J0404241163) — *Red Team Specialist (Attacker)*
3. **Rizki Ramadhani** (NIM: J0404241144) — *Blue Team Specialist (Defender / Forensics)*
4. **Yuwandri Alfarizi** (NIM: J0404241010) — *Blue Team Specialist (Network Security & SIEM)*

---

## Slide 1: Ringkasan Proyek & Perjalanan Fase 1–3

### Evolusi Siklus PBL (Minggu 1 s/d Minggu 15)
* **Fase 1 (Minggu 1–8) — Arsitektur & Perancangan Baseline:**
  * Menganalisis sistem telemetri kandang sapi mitra (suhu, kelembapan, video streaming CCTV sapi).
  * Merancang topologi lab 3 node (`192.168.6.0/24`): Attacker (`.10`), Target Server (`.20`), dan Monitor SIEM (`.30`).
  * Deklarasi pemetaan *tech stack* asli ke replika lab yang aman dan terkontrol.
* **Fase 2 (Minggu 9–11) — Hardening & Simulasi Serangan (Purple Team Loop):**
  * Penerapan *Security Baseline Checklist* Kategori A, B, dan C pada Target Node.
  * Eksekusi 6 skenario serangan agresif oleh Red Team (ATK-01 s/d ATK-06).
  * Deteksi insiden secara *real-time* menggunakan Security Onion (Zeek NSM, Syslog, Suricata).
  * Siklus Purple Team Loop: Serang $\rightarrow$ Analisis Deteksi $\rightarrow$ Terapkan Kontrol $\rightarrow$ Uji Ulang (Verifikasi).
* **Fase 3 (Minggu 12–15) — Forensik, Pelaporan NIST SP 800-61r2 & Penyerahan Artefak:**
  * Investigasi forensik digital & rekonstruksi *attack timeline*.
  * Penyusunan Incident Report standar NIST SP 800-61r2 dan rekaman video demo verifikasi.

---

## Slide 2: Ringkasan Eksekutif Incident Report (NIST SP 800-61r2)

### Status Insiden & Hasil Penanganan
* **Sistem yang Dilindungi:** Backend replika Smart Cow Monitoring System (Dashboard Node.js Port 80, MediaMTX RTSP Port 554, SQLite Database, OpenSSH Port 22).
* **Insiden yang Disimulasikan:** 6 skenario serangan siber mencakup *Reconnaissance*, *Information Disclosure*, *Credential Brute-Force*, *Video Stream Hijacking*, dan *System File Tampering*.
* **Status Awal (Sebelum Mitigasi):** Kerentanan tinggi (*High Severity*). Port RTSP terbuka ke publik tanpa autentikasi, API web rentan *credential stuffing*, SSH mengizinkan *password authentication*, dan berkas database dapat diakses langsung melalui URI HTTP.
* **Hasil Akhir (Setelah Mitigasi):**
  * Seluruh 6 vektor serangan berhasil dimitigasi (**Tingkat Keberhasilan: 100% Pass**).
  * Red Team gagal menembus kembali pertahanan pada pengujian ulang (*Retest*).
  * Visibilitas audit log 100% terpantau terpusat pada Security Onion Kibana.

---

## Slide 3: Arsitektur Lab & Pemetaan Tech Stack

### Topologi Jaringan Terisolasi (`siber-kel6`)
* **Attacker Node (`192.168.6.10` / Kali Linux 2026.3):**
  * Peran: Menjalankan Nmap, Hydra, curl, ffplay/VLC, dan skrip eksploitasi.
* **Target Node (`192.168.6.20` / Ubuntu Server 26.04.1 LTS):**
  * Peran: Backend replika Smart Cow (Node.js REST API, SQLite, MediaMTX RTSP, OpenSSH).
* **Monitoring Node (`192.168.6.30` / CyberOps Security Onion):**
  * Peran: Sniffing Zeek NSM pada `eth1` promiscuous mode, rsyslog receiver UDP 514, Kibana Dashboard.

### Pemetaan Sistem IoT Lapangan ke Lab Replika
| Komponen | Sistem IoT Asli Mitra | Target Node Replika Lab |
|---|---|---|
| Gateway/OS | Raspberry Pi / Industrial Edge OS | Ubuntu Server 26.04.1 LTS VM |
| Telemetri | Sensor ESP32 (HTTP REST / JSON) | Node.js Express Server (Port 80) |
| Video CCTV | Kamera RTSP H.264 Kandang Sapi | MediaMTX RTSP Server (Port 554) |
| Database | SQLite lokal (`smartcow.db`) | SQLite Engine (`node:sqlite`) |
| SIEM & Audit | Logging manual gateway | Rsyslog forwarding ke Security Onion |

---

## Slide 4: Status Akhir Hardening Baseline (Kategori A, B, C)

### Evaluasi Checklist Keamanan Sebelum vs Sesudah Hardening
* **Kategori A — Sistem Operasi & Jaringan Host:**
  * *Firewall Host (UFW):* Status awal *Inactive* $\rightarrow$ Diaktifkan: Port 80 & 22 dibuka selektif, Port 554 difilter ketat (`Pass`).
  * *Izin Berkas Sensitif:* Awalnya `644`/`666` $\rightarrow$ Dikunci ke `640` untuk database dan file konfigurasi (`Pass`).
  * *Update & Patch:* Kernel dan paket inti diperbarui ke versi stabil LTS (`Pass`).
* **Kategori B — Layanan Aplikasi & Basis Data (Node.js & SQLite):**
  * *Isolasi Web Path:* Endpoint `/smartcow.db` dinonaktifkan; database dipindah ke luar direktori publik web (`Pass`).
  * *Rate Limiting:* Awalnya tanpa batas $\rightarrow$ Dibatasi maksimal 10 request/menit untuk API autentikasi (`Pass`).
  * *Input Sanitization & Parameterized Query:* Mencegah SQL Injection pada query telemetri (`Pass`).
* **Kategori C — Akses Manajemen & Streaming (SSH & RTSP):**
  * *SSH Hardening:* `PasswordAuthentication no`, `PermitRootLogin no`, penegakan kunci RSA/Ed25519 (`Pass`).
  * *MediaMTX Hardening:* Menonaktifkan 7 protokol default yang tidak digunakan (RTMP, HLS, WebRTC, SRT) (`Pass`).

---

## Slide 5: Alur Kerja Purple Team Loop & Demo Video

### Metodologi Purple Team: Kolaborasi Aktif Red & Blue Team
```
[ Tahap 1: Attack Execution ] (Red Team via Kali Linux 192.168.6.10)
              │
              ▼
[ Tahap 2: Baseline Detection Check ] (Security Onion 192.168.6.30 & Syslog)
              │
              ▼
[ Tahap 3: Threat Analysis & Gap Assessment ] (Analisis Kategori Insiden & Resiko)
              │
              ▼
[ Tahap 4: Mitigation & Hardening Implementation ] (Blue Team pada Target 192.168.6.20)
              │
              ▼
[ Tahap 5: Retest & Empirical Verification ] (Pengujian Ulang oleh Red Team -> PASS)
```

### Video Demo Highlight (Durasi: 5–8 Menit)
* Menampilkan verifikasi empiris live dengan identitas host/IP kelompok (`192.168.6.x`).
* Membuktikan transisi status serangan dari *Vulnerable* menjadi *Blocked/Mitigated*.

---

## Slide 6: Hasil Deteksi & Mitigasi (Skenario ATK-01 & ATK-02)

### ATK-01: Reconnaissance (Port Scanning & Service Fingerprinting)
* **Aksi Red Team:** Pemindaian Nmap agresif (`nmap -sS -sV -p- 192.168.6.20`).
* **Deteksi Awal:** Security Onion merekam lonjakan koneksi TCP SYN pada Zeek `conn.log` dan `notice.log` (Scan::Port_Scan).
* **Mitigasi Blue Team:** Penerapan UFW *default deny incoming*, menutup seluruh port yang tidak digunakan, serta membatasi port 554 hanya untuk IP terdaftar.
* **Hasil Verifikasi:** Port 554 berubah status dari `OPEN` menjadi `FILTERED`. Penyerang kehilangan visibilitas topologi port.

### ATK-02: SQLite Database Exposure via Web Path
* **Aksi Red Team:** Permintaan HTTP GET `/smartcow.db` menggunakan `curl` dan browser.
* **Deteksi Awal:** Web access log mencatat download berkas database; Zeek `http.log` mendeteksi MIME `application/x-sqlite3`.
* **Mitigasi Blue Team:** Menghapus static routing ke direktori database, memindahkan berkas database ke folder privat `data/`, dan menyetel aturan penolakan eksplisit (404/403).
* **Hasil Verifikasi:** Permintaan `curl http://192.168.6.20/smartcow.db` menghasilkan `HTTP/1.1 404 Not Found`.

---

## Slide 7: Hasil Deteksi & Mitigasi (Skenario ATK-03 & ATK-04)

### ATK-03: Web Authentication Brute Force (Credential Stuffing)
* **Aksi Red Team:** Serangan kamus ribuan percobaan login POST `/api/login` menggunakan Hydra/skrip Python.
* **Deteksi Awal:** Lonjakan ratusan status respons `HTTP 401 Unauthorized` terekam pada Zeek `http.log` dan Kibana dashboard.
* **Mitigasi Blue Team:** Integrasi *middleware* `express-rate-limit` pada endpoint login dengan batas 10 request per jendela waktu 1 menit.
* **Hasil Verifikasi:** Percobaan login ke-11 dan seterusnya langsung dijawab `HTTP/1.1 429 Too Many Requests`. Serangan kamus lumpuh.

### ATK-04: SSH Remote Access Brute Force
* **Aksi Red Team:** Serangan *dictionary attack* login SSH port 22 (`hydra -l root -P passwords.txt ssh://192.168.6.20`).
* **Deteksi Awal:** Rsyslog meneruskan anomali `Failed password for root` secara masif ke Security Onion Kibana.
* **Mitigasi Blue Team:** Menonaktifkan `PasswordAuthentication`, menolak login `root`, mengaktifkan autentikasi *SSH Key Pair*, dan mengaktifkan proteksi *Fail2ban*.
* **Hasil Verifikasi:** Percobaan SSH dengan password langsung ditolak di level handshake: `Permission denied (publickey)`.

---

## Slide 8: Hasil Deteksi & Mitigasi (Skenario ATK-05 & ATK-06)

### ATK-05: Unauthorized RTSP Video Feed Interception
* **Aksi Red Team:** Menarik siaran langsung kamera kandang via `ffplay -rtsp_transport tcp rtsp://192.168.6.20:554/live/cow1`.
* **Deteksi Awal:** Zeek mendeteksi aliran stream RTSP konstan dari IP `.20` ke penyerang `.10` dengan volume *traffic* bandwidth besar.
* **Mitigasi Blue Team:** Aturan firewall UFW spesifik: `ufw deny from 192.168.6.10 to any port 554`, menonaktifkan API publik MediaMTX, dan mengisolasi routing RTSP.
* **Hasil Verifikasi:** Perintah `ffplay` penyerang berakhir dengan `Connection timed out`. Feed video pengawasan sapi aman dari intipan ilegal.

### ATK-06: Configuration File Tampering & Privilege Abuse
* **Aksi Red Team:** Upaya modifikasi skrip server `app.js` dan konfigurasi `mediamtx.yml` melalui akun pengguna non-privilege.
* **Deteksi Awal:** Audit Syslog mendeteksi upaya akses tidak sah dan modifikasi berkas kritis.
* **Mitigasi Blue Team:** Penguncian kepemilikan berkas ke `kel6:kel6` dan pembatasan izin ke hak akses minimal `chmod 640` (hanya owner read/write, group read).
* **Hasil Verifikasi:** Akses tulis oleh pengguna asing atau proses tak berizin langsung dibatalkan sistem: `Permission denied`.

---

## Slide 9: Analisis Forensik & Taksonomi Insiden (NIST SP 800-61r2)

### Taksonomi & Klasifikasi Insiden Siber
| ID Insiden | Vektor Insiden NIST | Kategori Taksonomi | Dampak Fungsional | Dampak Informasi |
|---|---|---|---|---|
| **ATK-01** | *External / Network Probe* | Reconnaissance | None (Tidak mengganggu layanan) | Low (Pemetaan port) |
| **ATK-02** | *Web Application / Leak* | Data Leakage | Low (Sistem tetap berjalan) | **High** (Basis data terunduh) |
| **ATK-03** | *Web Authentication / Attrition* | Brute Force | Medium (Beban CPU & log melonjak) | Medium (Percobaan bypass) |
| **ATK-04** | *Unauthorized Access / Attrition* | Brute Force | Medium (Ancaman takeover host) | **High** (Potensi kompromi root) |
| **ATK-05** | *Network Eavesdropping / Stream* | Data Interception | Low (Beban bandwidth kamera) | **High** (Privasi kandang bocor) |
| **ATK-06** | *Privilege Abuse / Tampering* | Integrity Violation | **High** (Potensi persistensi malware) | High (Kerusakan konfigurasi) |

### Bukti Forensik Terintegrasi (Security Onion)
* **Zeek Network Logs:** Bukti empiris pada `conn.log`, `http.log`, dan `notice.log`.
* **Centralized Rsyslog:** Merekam jejak otentikasi SSH dan aktivitas sistem secara *tamper-evident*.
* **Kibana Discover Dashboard:** Visualisasi lonjakan serangan dan validasi penurunan anomali pasca-mitigasi.

---

## Slide 10: Kendala & Pembelajaran Sepanjang Proyek

### Kendala Teknis yang Dihadapi & Solusinya
1. **Logstash Ingest Lockout pada Security Onion:**
   * *Kendala:* Eksekusi `so-import-pcap` mengunci konfigurasi Logstash ke mode impor pcap sehingga log *live* rsyslog tidak muncul di Kibana.
   * *Solusi:* Menghapus variabel `LOGSTASH_OUTPUT_INGEST`, mengonfigurasi `LOGSTASH_MINIMAL="yes"`, dan menaikkan JVM Heap menjadi 512MB di `/etc/logstash/jvm.options`.
2. **Keterbatasan Memori Host (RAM 8 GB):**
   * *Kendala:* Menjalankan 3 VM secara paralel (Kali Linux GUI, Ubuntu Server, Security Onion) membebani alokasi RAM.
   * *Solusi:* Optimasi RAM Security Onion (2.5 GB), Target (1.5 GB), dan Kali (2 GB) dengan tuning alokasi swap.
3. **Kombinasi Layanan Multi-Port pada Target Node:**
   * *Kendala:* MediaMTX secara *default* membuka banyak port media (RTMP, WebRTC, HLS) yang memperbesar *attack surface*.
   * *Solusi:* Mematikan seluruh listener protokol sekunder pada berkas `mediamtx.yml`.

### Pembelajaran Kunci Tim
* Keamanan bukan fitur tambahan, melainkan arsitektur berlapis (*Defense-in-Depth*).
* Pemantauan SIEM terpusat adalah fondasi utama validasi pertahanan siber.
* Sinergi Purple Team mempercepat penemuan dan penutupan celah secara drastis dibanding pengujian terisolasi.

---

## Slide 11: Rekomendasi Lanjutan untuk Sistem IoT Nyata (Mitra)

### Rencana Peningkatan Keamanan untuk Implementasi di Peternakan Sapi
1. **Wajib Enkripsi Transport Layer (TLS & RTSPS):**
   * Mengamankan dashboard peternak menggunakan HTTPS (Port 443) dengan sertifikat TLS valid.
   * Mengenkripsi transmisi feed video kamera kandang via RTSPS (Port 322) mencegah intersepsi di jaringan lokal.
2. **Mutual Authentication (mTLS) pada Sensor ESP32:**
   * Memvalidasi keaslian perangkat sensor IoT pengirim telemetri suhu/kelembapan melalui sertifikat digital dua arah.
3. **Zero-Trust Remote Access / Dedicated VPN:**
   * Mengisolasi seluruh akses administrasi teknisi di balik jaringan VPN privat (WireGuard / OpenVPN) dengan autentikasi multi-faktor (MFA), tanpa mengekspos port SSH ke internet.
4. **Enkripsi Penyimpanan Edge (Data at-Rest):**
   * Menerapkan enkripsi disk penuh (LUKS) atau SQLCipher pada berkas database SQLite untuk memitigasi risiko pencurian fisik perangkat gateway di kandang terbuka.
5. **Firmware Integrity & Automated Patch Management:**
   * Mengaktifkan *Secure Boot* pada mikrokontroler dan pembaruan keamanan otomatis pada sistem operasi edge gateway.

---

## Slide 12: Penutup & Tautan Artefak GitHub

### Kesimpulan Akhir
Proyek PBL Keamanan Siber Kelompok 6 berhasil merancang infrastruktur replika yang realistis, menguji ketahanan sistem melalui simulasi serangan terukur, mendeteksi insiden menggunakan SIEM Security Onion, dan menerapkan kontrol mitigasi berlapis dengan tingkat keberhasilan verifikasi **100% Pass**.

### Repositori & Dokumentasi Lengkap
* **GitHub Repository:** [https://github.com/Allmerr/TEK1314-2026-Kel6-KelasA](https://github.com/Allmerr/TEK1314-2026-Kel6-KelasA)
* **Dokumen Final Fase 3:**
  * Incident Report (NIST SP 800-61r2): `docs/final/incident_report.pdf`
  * Purple Team Log Lengkap: `docs/purple-team-log.md`
  * Slide Presentasi Final: `docs/presentation-final/slides.pdf`
  * Video Demo & Panduan Retest: `docs/final/README.md` & `docs/final/video_demo.mp4`

---
*Terima Kasih — Sesi Tanya Jawab (Q&A)*
