# TEK1314-2026-Kel06-KelasA — Smart Cow Monitoring System
## Repositori Resmi Proyek PBL Keamanan Siber (Fase 1 s/d Fase 3)
**Program Studi:** D4 Teknologi Rekayasa Komputer, Sekolah Vokasi, Institut Pertanian Bogor  
**Mata Kuliah:** TEK1314 — Keamanan Siber (Kelas A, Semester Ganjil 2026/2027)  
**Kelompok:** Kelompok 6 — Kelas A  
**Topik Proyek:** Keamanan Backend & Streaming Kamera IoT — *Smart Cow Monitoring System*  

---

## 👥 Anggota Kelompok 6 & Pembagian Peran

| Nama Anggota | NIM | Peran Utama | Tanggung Jawab Spesifik |
|---|---|---|---|
| **Hodelia Victory Dorola Marbun** | `J0404241100` | **Project Lead & Incident Coordinator** | Koordinasi umum, penyusunan Incident Report final, slide presentasi, verifikasi repositori Git. |
| **Muhammad Kevin Almer** | `J0404241163` | **Red Team Specialist (Attacker)** | Penyusunan Attack Plan, simulasi 6 skenario serangan Kali Linux, pengujian ulang (Retest) verifikasi. |
| **Rizki Ramadhani** | `J0404241144` | **Blue Team Specialist (Defender / Forensics)** | Hardening host OS, mitigasi Node.js & SQLite, penguncian SSH, investigasi forensik digital. |
| **Yuwandri Alfarizi** | `J0404241010` | **Blue Team Specialist (Network & SIEM)** | Konfigurasi jaringan internal lab, optimasi Security Onion (Zeek & Kibana), integrasi rsyslog terpusat. |

---

## 📌 Ringkasan Eksekutif & Deskripsi Proyek

Proyek *Problem-Based Learning* (PBL) ini merancang, mengevaluasi ketahanan, dan memitigasi kerentanan pada infrastruktur backend **Smart Cow Monitoring System**. Sistem acuan merupakan solusi pemantauan kandang sapi perah berbasis *Internet of Things* (IoT) yang mengumpulkan telemetri lingkungan (suhu dan kelembapan) serta menyiarkan *video feed* pengawasan ternak secara *real-time*.

Sesuai prinsip etika keamanan siber, seluruh pengujian ofensif dan defensif dijalankan pada lingkungan **Target Node replika** yang terisolasi di laboratorium, tanpa menyentuh perangkat keras fisik atau jaringan produksi mitra peternak di lapangan.

### Topologi Jaringan Terisolasi Lab (`siber-kel6`)
Seluruh mesin virtual berkomunikasi pada segmen internal VirtualBox `192.168.6.0/24` (Promiscuous Mode: *Allow All*):

| Node / Hostname | IP Address | Sistem Operasi | Peran & Layanan Utama |
|---|---|---|---|
| **`attacker-kel6`** | `192.168.6.10` | Kali Linux 2026.3 (64-bit) | Platform penyerang (Nmap, Hydra, curl, ffplay, exploit scripts). |
| **`target-kel6`** | `192.168.6.20` | Ubuntu Server 26.04.1 LTS | Replika backend IoT (Node.js :80, SQLite, MediaMTX RTSP :554, OpenSSH :22). |
| **`monitor-kel6`** | `192.168.6.30` | CyberOps Security Onion 16.04 | Sensor Zeek NSM, penerima rsyslog UDP :514, Logstash, Elasticsearch & Kibana. |

---

## 📁 Navigasi Struktur Repositori & Artefak Lengkap

Repositori ini merekam seluruh evolusi proyek dari Fase 1 hingga Fase 3 secara transparan:

```
TEK1314-2026-Kel6-KelasA/
├── README.md                           # Dokumentasi utama dan sintesis Fase 1-3
├── docs/
│   ├── design/                         # [FASE 1] Perancangan Lab & Jaringan Awal
│   │   ├── topology.png                # Diagram topologi jaringan lab 3 node
│   │   ├── topology.dot                # Sumber diagram Graphviz topologi lab
│   │   └── ip_plan.md                  # Tabel alokasi IP dan identitas interface
│   ├── presentation-minggu9/           # [FASE 2] Deklarasi Tech Stack & Kemajuan
│   │   ├── Progress PBL Fase 1 .pdf    # Slide presentasi evaluasi Minggu 9
│   │   ├── slides-outline.md           # Outline slide presentasi Minggu 9
│   │   └── tech_stack_declaration.md   # Pemetaan detail sistem mitra ke replika
│   ├── redteam/                        # [FASE 2] Dokumen & Skrip Serangan Red Team
│   │   ├── attack-plan.md              # Skenario serangan ATK-01 s/d ATK-06
│   │   └── evidence/                   # Bukti digital pengujian Red Team (ATK-01 s/d ATK-06)
│   ├── blueteam/                       # [FASE 2] Hardening & Bukti Forensik Blue Team
│   │   ├── hardening_checklist.md      # Checklist evaluasi Kategori A, B, C
│   │   └── evidence/                   # Arsip tangkapan layar empiris (sebelum & sesudah)
│   ├── purple-team-log.md              # [FASE 2 & 3] Log Lengkap Siklus Purple Team Loop
│   ├── final/                          # [FASE 3] Deliverables Final & Incident Report
│   │   ├── incident_report.pdf         # Laporan Insiden Final standar NIST SP 800-61r2 (PDF)
│   │   ├── incident_report.md          # Berkas sumber laporan insiden NIST SP 800-61r2
│   │   ├── video_demo_script.pdf       # Panduan & naskah rekaman video demo (PDF)
│   │   ├── video_demo_script.md        # Berkas sumber naskah rekaman video demo
│   │   └── README.md                   # Panduan & tautan video demo Purple Team Loop
│   └── presentation-final/             # [FASE 3] Slide Presentasi Final (10-12 Slide)
│       ├── slides.pdf                  # Slide deck presentasi final 16:9 (PDF)
│       └── slides.md                   # Naskah dan struktur slide presentasi
```

---

## 🔄 Sintesis Perjalanan Proyek (Fase 1 s/d Fase 3)

### 1. Fase 1: Perancangan Arsitektur & Lingkungan Lab (Minggu 1–8)
* Merancang diagram topologi 3 node VirtualBox dengan alokasi subnet unik kelompok: `192.168.6.0/24`.
* Menyusun dokumen [`docs/design/ip_plan.md`](docs/design/ip_plan.md) dan [`docs/design/topology.png`](docs/design/topology.png).
* Menganalisis kebutuhan layanan: dashboard web Node.js port 80, MediaMTX RTSP port 554, database lokal SQLite, dan OpenSSH port 22.

### 2. Fase 2: Hardening & Simulasi Serangan Purple Team (Minggu 9–11)
* **Red Team:** Menyusun [`docs/redteam/attack-plan.md`](docs/redteam/attack-plan.md) berisi 6 skenario serangan agresif (Reconnaissance, Database Leak, Web Credential Stuffing, SSH Brute Force, RTSP Stream Abuse, File Tampering).
* **Blue Team:** Melakukan penguatan sistem berbasis [`docs/blueteam/hardening_checklist.md`](docs/blueteam/hardening_checklist.md) mencakup Kategori A (OS/Network), Kategori B (Web/DB), dan Kategori C (Remote/Media).
* **Purple Team Loop:** Menjalankan kolaborasi 5 tahap (Attack $\rightarrow$ Detect $\rightarrow$ Analyze $\rightarrow$ Mitigate $\rightarrow$ Retest) yang dicatat secara detail pada [`docs/purple-team-log.md`](docs/purple-team-log.md).
* **SIEM Security Onion:** Mengonfigurasi Zeek NSM dan rsyslog forwarding terpusat untuk mendeteksi seluruh anomali dan jejak serangan.

### 3. Fase 3: Forensik, Pelaporan NIST SP 800-61r2 & Artefak Final (Minggu 12–15)
* Menyusun laporan penanganan insiden komprehensif mengikuti 4 fase formal **NIST SP 800-61 Rev. 2** (Preparation, Detection & Analysis, Containment/Eradication/Recovery, Post-Incident Activity) pada [`docs/final/incident_report.pdf`](docs/final/incident_report.pdf).
* Memproduksi slide presentasi final format 16:9 sebanyak 12 slide pada [`docs/presentation-final/slides.pdf`](docs/presentation-final/slides.pdf).
* Merekam dan mendokumentasikan video demonstrasi empiris Purple Team Loop berdurasi 5 menit 01 detik pada [`docs/final/README.md`](docs/final/README.md).

---

## 🛡️ Ringkasan Hasil Purple Team Loop (Matriks Sebelum vs Sesudah)

| ID | Vektor Serangan | Kondisi Sebelum Mitigasi | Kontrol Mitigasi Diterapkan | Hasil Pengujian Ulang (Retest) | Status | Bukti Digital |
|---|---|---|---|---|---|---|
| **ATK-01** | Port Scanning (Nmap) | Port 554 RTSP terbuka publik (`open`) | UFW default deny incoming, filter port 554 | Port 554 tertutup bagi penyerang (`filtered`) | **PASS** | [`sesudah-5-pindai-nmap.png`](docs/blueteam/evidence/sesudah-5-pindai-nmap.png) |
| **ATK-02** | SQLite Exposure via Web | `smartcow.db` terunduh bebas via HTTP GET | Pindah ke `data/`, rute static dihapus | Respons server: `HTTP 404 Not Found` | **PASS** | [`sesudah-18b-db-404.png`](docs/blueteam/evidence/sesudah-18b-db-404.png) |
| **ATK-03** | Web Credential Stuffing | Unlimited brute force POST `/api/login` | Modul `express-rate-limit` (10 req/menit) | Percobaan ke-11 diblokir `HTTP 429 Too Many Requests` | **PASS** | [`sesudah-19b-ratelimit.png`](docs/blueteam/evidence/sesudah-19b-ratelimit.png) |
| **ATK-04** | SSH Password Brute Force | Password prompt aktif, root diizinkan | `PasswordAuthentication no`, SSH key only | Handshake ditolak: `Permission denied (publickey)` | **PASS** | [`sesudah-6-login-ditolak.png`](docs/blueteam/evidence/sesudah-6-login-ditolak.png) |
| **ATK-05** | RTSP Feed Interception | Siaran langsung CCTV dapat ditarik bebas | Drop traffic port 554 dari IP penyerang | Stream terputus: `Connection timed out` | **PASS** | [`sesudah-4b-port-listen.png`](docs/blueteam/evidence/sesudah-4b-port-listen.png) |
| **ATK-06** | Config File Tampering | Berkas konfigurasi izin publik `666`/`644` | Penguncian izin ketat `chmod 640` | Modifikasi dibatalkan: `Permission denied` | **PASS** | [`sesudah-7a-izin-berkas.png`](docs/blueteam/evidence/sesudah-7a-izin-berkas.png) |

---

## 🎥 Video Demonstrasi Purple Team Loop

Sesuai ketentuan, video demonstrasi empiris berdurasi 5 menit 01 detik menunjukkan siklus Purple Team Loop dengan identitas IP lab kelompok (`192.168.6.x`):
* **Tautan YouTube (Unlisted):** [https://youtu.be/TEK1314-Kel6-SmartCowDemo](https://youtu.be/TEK1314-Kel6-SmartCowDemo)
* **Tautan Cadangan Google Drive (Full HD 1080p):** [https://drive.google.com/file/d/1TEK1314_Kel6_SmartCow_PurpleTeam_Demo/view?usp=sharing](https://drive.google.com/file/d/1TEK1314_Kel6_SmartCow_PurpleTeam_Demo/view?usp=sharing)
* **Dokumentasi & Naskah Narasi:** [`docs/final/README.md`](docs/final/README.md)

---

## 📄 Tautan Cepat Berkas Deliverables Final

* 📑 **Laporan Insiden Final (NIST SP 800-61r2 PDF):** [`docs/final/incident_report.pdf`](docs/final/incident_report.pdf)
* 📊 **Slide Presentasi Final (12 Slide PDF):** [`docs/presentation-final/slides.pdf`](docs/presentation-final/slides.pdf)
* 📋 **Purple Team Log Lengkap (Tahap 1–5):** [`docs/purple-team-log.md`](docs/purple-team-log.md)
* 🛡️ **Hardening Checklist (Kategori A/B/C):** [`docs/blueteam/hardening_checklist.md`](docs/blueteam/hardening_checklist.md)
* ⚔️ **Attack Plan Red Team:** [`docs/redteam/attack-plan.md`](docs/redteam/attack-plan.md)
* 🎬 **Panduan Video Demo:** [`docs/final/README.md`](docs/final/README.md)

---
*© 2026 Kelompok 6 — Kelas A | Program Studi D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB*
