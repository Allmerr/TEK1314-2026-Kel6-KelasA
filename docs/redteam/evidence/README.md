# Bukti Digital Pengujian Red Team (Evidence — ATK-01 s/d ATK-06)

**Mata Kuliah:** TEK1314 — Keamanan Siber (D4 Rekayasa Komputer SV IPB)  
**Kelompok:** Kelompok 6 — Kelas A  
**Topik:** Smart Cow Monitoring System (Replika Backend IoT & RTSP Streaming)  
**Operator Red Team:** Muhammad Kevin Almer (`J0404241163`)  
**Node Penyerang:** `192.168.6.10` (`attacker-kel6` — Kali Linux 2026.3)  
**Node Sasaran:** `192.168.6.20` (`target-kel6` — Ubuntu Server 26.04)  
**Standar Waktu Bukti:** Universal Time Coordinated (**UTC**)

---

## 1. Struktur Direktori Bukti Forensik

Struktur berkas bukti pengujian empiris disusun per skenario serangan (`ATK-01` s/d `ATK-06`) sesuai panduan kurikulum:

```text
docs/redteam/evidence/
├── README.md                           # Dokumentasi metadata dan pemetaan bukti
├── ATK-01/                             # Skenario 1: Port Scanning & Reconnaissance
│   ├── sebelum-port-scan.png           # Bukti Port 22, 80, 554 berstatus OPEN
│   └── sesudah-port-scan.png           # Bukti Port 554 berstatus FILTERED (Pass)
├── ATK-02/                             # Skenario 2: Unauthorized Web Access
│   ├── sebelum-unauthorized.png        # Akses endpoint publik tanpa autentikasi
│   └── sesudah-unauthorized.png        # Respons HTTP 401 Unauthorized (Pass)
├── ATK-03/                             # Skenario 3: Web Login Brute-Force & Rate Limiting
│   ├── sebelum-bruteforce.png          # Upaya login berulang tanpa pembatasan
│   └── sesudah-ratelimit-429.png       # Respons HTTP 429 Too Many Requests (Pass)
├── ATK-04/                             # Skenario 4: SSH Password Brute-Force
│   ├── sebelum-ssh-password.png        # Prompt password login aktif (rentan)
│   └── sesudah-ssh-keyonly.png         # Ditolak: Permission denied (publickey) (Pass)
├── ATK-05/                             # Skenario 5: SQLite Database Exposure
│   ├── sebelum-db-exposure.png         # Pengecekan file basis data di web root
│   └── sesudah-db-404.png              # Respons HTTP 404 Not Found (Pass)
└── ATK-06/                             # Skenario 6: Unauthorized RTSP Video Feed Access
    ├── sebelum-rtsp-access.png         # Port listen RTSP 554 terbuka
    └── sesudah-rtsp-filtered.png       # Probing RTSP gagal / Connection timed out (Pass)
```

---

## 2. Matriks Verifikasi Hasil Pengujian (Status: Pass 100%)

| ID | Vektor Pengujian | Hasil Sebelum Mitigasi | Hasil Aktual Setelah Mitigasi | Status | Bukti Digital |
|---|---|---|---|:---:|---|
| **ATK-01** | Network Port Scan (Nmap) | Port 554 RTSP `open` | Port 554 RTSP `filtered` (drop UFW) | **Pass** | [Bukti ATK-01](ATK-01/sesudah-port-scan.png) |
| **ATK-02** | Unauthorized Web Endpoint | Endpoint web dapat diakses bebas | Respons `HTTP 401 Unauthorized` | **Pass** | [Bukti ATK-02](ATK-02/sesudah-unauthorized.png) |
| **ATK-03** | Web Credential Stuffing | Brute force diproses terus (HTTP 200/401) | Percobaan 11 & 12 diblokir `HTTP 429` | **Pass** | [Bukti ATK-03](ATK-03/sesudah-ratelimit-429.png) |
| **ATK-04** | SSH Password Brute-Force | Prompt password muncul di terminal | Ditolak `Permission denied (publickey)` | **Pass** | [Bukti ATK-04](ATK-04/sesudah-ssh-keyonly.png) |
| **ATK-05** | SQLite File Exposure | File basis data dapat diunduh | Respons `HTTP 404 Not Found` | **Pass** | [Bukti ATK-05](ATK-05/sesudah-db-404.png) |
| **ATK-06** | RTSP Video Feed Interception | Feed video CCTV dapat disedot bebas | Probing koneksi `Connection timed out` | **Pass** | [Bukti ATK-06](ATK-06/sesudah-rtsp-filtered.png) |
