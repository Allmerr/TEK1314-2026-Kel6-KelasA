# Purple Team Log — Siklus Pengujian, Deteksi, dan Mitigasi

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer SV IPB)  
**Kelompok:** 6 — Kelas A  
**Skenario:** Smart Cow Monitoring System (Replika Backend IoT & RTSP Streaming)  
**Fokus:** Siklus Kolaborasi Red Team vs Blue Team (Purple Team Loop — Minggu 9 s/d 11)  
**Status:** Selesai & Terverifikasi (Fase 3)

---

## 1. Ringkasan Topologi & Lingkungan Pengujian

| Node | Hostname | IP Address | Peran | Komponen Utama |
|---|---|---|---|---|
| **Attacker Node** | `attacker-kel6` | `192.168.6.10` | Red Team (Penyerang) | Kali Linux 2026.3, Nmap, Curl, Burp Suite, Hydra |
| **Target Node** | `target-kel6` | `192.168.6.20` | Korban (Replika IoT) | Ubuntu Server 26.04.1 LTS, Node.js :80, SQLite, MediaMTX :554, OpenSSH :22 |
| **Monitoring Node** | `monitor-kel6` | `192.168.6.30` | Blue Team (Pemantau) | Security Onion (Zeek NIDS, Snort, Syslog-ng, Logstash, Elasticsearch, Kibana) |

Segmen Jaringan Lab: VirtualBox Internal Network `siber-kel6` (`192.168.6.0/24`) dengan Promiscuous Mode `allow-all`. Seluruh waktu pencatatan insiden menggunakan standar **UTC**.

---

## 2. Metodologi Purple Team Loop

Purple Team Loop menerapkan siklus iteratif lima tahap:

```
[Tahap 1: Attack Execution] ──► [Tahap 2: Detection Analysis] ──► [Tahap 3: Defense Gap Analysis]
           ▲                                                                   │
           │                                                                   ▼
[Tahap 5: Retest & Verification] ◄─────────────────────────────── [Tahap 4: Mitigation & Hardening]
```

1. **Tahap 1 — Attack Execution:** Red Team mengeksekusi skenario serangan terukur sesuai `attack-plan.md` dari node Kali Linux (`192.168.6.10`).
2. **Tahap 2 — Detection Analysis:** Blue Team menganalisis visibilitas serangan pada Security Onion (`192.168.6.30`) serta log lokal Target Node.
3. **Tahap 3 — Defense Gap Analysis:** Evaluasi bersama mengapa serangan berhasil dan identifikasi kelemahan konfigurasi.
4. **Tahap 4 — Mitigation & Hardening:** Blue Team menerapkan kontrol keamanan pada Target Node (`hardening_checklist.md`).
5. **Tahap 5 — Retest & Verification:** Red Team mengulang serangan yang sama untuk membuktikan kontrol keamanan berhasil menggagalkan serangan.

---

## 3. Matriks Skenario Purple Team Loop

| ID | Nama Skenario | Target Layanan | Hasil Sebelum Mitigasi | Hasil Setelah Mitigasi | Status Akhir |
|---|---|---|---|---|---|
| **ATK-01** | Port Scanning & Reconnaissance | Host Network Target | Semua port terekspos, UFW inaktif | Port 554 filtered, hanya port 22 & 80 terbuka | **Pass** |
| **ATK-02** | Unauthorized SQLite Exposure | Web Node.js :80 | DB dapat diakses/di-traverse via HTTP | HTTP 404 Not Found, permissions 640 | **Pass** |
| **ATK-03** | Web Admin Login Brute-Force | Web Login :80 | Request tak terbatas, no throttling | HTTP 429 Too Many Requests (Rate Limited) | **Pass** |
| **ATK-04** | SSH Password Brute-Force | OpenSSH :22 | Password login aktif, root diizinkan | Publickey only, root ditolak, MaxAuthTries 3 | **Pass** |
| **ATK-05** | Unauthorized RTSP Access | MediaMTX :554 | 7 port media terbuka, RTSP publik | Protokol nonaktif, UFW drop RTSP selain .30 | **Pass** |
| **ATK-06** | Configuration Tampering | Sistem Berkas | Permissions file permisif (666/644) | File permission terkunci 640, non-root user | **Pass** |

---

## 4. Rincian Pelaksanaan Siklus per Skenario

### ATK-01: Port Scanning & Network Reconnaissance

* **Tahap 1 — Attack Execution:**
  Red Team menjalankan pemindaian menyeluruh terhadap target dari Kali Linux:
  ```bash
  nmap -sV -p- 192.168.6.20
  ```
  *Hasil Sebelum:* Seluruh port layanan (22, 80, 554, 1935, 8888, 8889, 9997) terdeteksi terbuka (`open`). Bukti: [`sebelum-6-pindai-nmap.png`](blueteam/evidence/sebelum-6-pindai-nmap.png), [`sebelum-4-port.png`](blueteam/evidence/sebelum-4-port.png).

* **Tahap 2 — Detection Analysis:**
  Security Onion mencatat lonjakan traffic TCP SYN pada Zeek `conn.log` (`id.orig_h: 192.168.6.10`, `conn_state: S0/REJ`). Snort memicu rule alert deteksi network scan.

* **Tahap 3 — Defense Gap Analysis:**
  Firewall UFW pada Target Node belum diaktifkan (`Status: inactive`). Seluruh port listening dapat diakses langsung oleh penyerang tanpa kontrol akses jaringan. Bukti: [`sebelum-3-firewall.png`](blueteam/evidence/sebelum-3-firewall.png).

* **Tahap 4 — Mitigation & Hardening:**
  1. Mengaktifkan UFW dengan kebijakan default drop:
     ```bash
     sudo ufw default deny incoming
     sudo ufw default allow outgoing
     ```
  2. Mengizinkan port esensial hanya dari subnet lab:
     ```bash
     sudo ufw allow from 192.168.6.0/24 to any port 22 proto tcp
     sudo ufw allow from 192.168.6.0/24 to any port 80 proto tcp
     sudo ufw allow from 192.168.6.0/24 to any port 123 proto udp
     ```
  3. Membatasi port RTSP 554 secara eksklusif ke Monitoring Node (`192.168.6.30`). Bukti: [`sesudah-3a-aturan-firewall.png`](blueteam/evidence/sesudah-3a-aturan-firewall.png), [`sesudah-3b-status-firewall.png`](blueteam/evidence/sesudah-3b-status-firewall.png).

* **Tahap 5 — Retest & Verification:**
  Red Team menjalankan ulang pemindaian nmap:
  ```bash
  nmap -sT -p 22,80,554 192.168.6.20
  ```
  *Hasil Verifikasi:* Port 554 berstatus `filtered` (koneksi di-drop oleh firewall), port 22 dan 80 tetap `open` untuk fungsionalitas legal. Pemindaian full-port membuktikan attack surface berkurang drastis. Bukti: [`sesudah-5-pindai-nmap.png`](blueteam/evidence/sesudah-5-pindai-nmap.png).

---

### ATK-02: Unauthorized Web Access & SQLite Information Disclosure

* **Tahap 1 — Attack Execution:**
  Red Team menguji kebocoran database SQLite replika IoT SmartCow:
  ```bash
  curl -I http://192.168.6.20/smartcow.db
  curl -I http://192.168.6.20/../smartcow.db
  curl -I http://192.168.6.20/data/smartcow.db
  ```
  *Hasil Sebelum:* Pada konfigurasi awal, berkas database atau endpoint internal dapat diakses atau dibaca akibat pemetaan direktori web yang terbuka.

* **Tahap 2 — Detection Analysis:**
  Syslog Target meneruskan access log ke Security Onion via `rsyslog` (UDP 514). Logstash memproses event ke indeks `logstash-syslog-*`, menampakkan query penyerang terhadap path `.db`.

* **Tahap 3 — Defense Gap Analysis:**
  Berkas database berada di dalam direktori publik atau tidak dilindungi oleh route handler eksplisit, serta izin berkas di sistem operasi masih permisif.

* **Tahap 4 — Mitigation & Hardening:**
  1. Membatasi permission berkas database pada level filesystem:
     ```bash
     chmod 640 /home/kel6/SmartCow/local/data/smartcow.db*
     chown kel6:kel6 /home/kel6/SmartCow/local/data/smartcow.db*
     ```
     Bukti: [`sesudah-18a-izin-db.png`](blueteam/evidence/sesudah-18a-izin-db.png).
  2. Menambahkan filter route pada Express.js untuk menolak segala akses ke ekstensi database dan path traversal, mengembalikan status `404 Not Found`.

* **Tahap 5 — Retest & Verification:**
  Red Team menguji 5 variasi path request database:
  ```bash
  curl -s -o /dev/null -w "%{http_code}\n" http://192.168.6.20/smartcow.db
  curl -s -o /dev/null -w "%{http_code}\n" http://192.168.6.20/data/smartcow.db
  curl -s -o /dev/null -w "%{http_code}\n" http://192.168.6.20/static/../smartcow.db
  ```
  *Hasil Verifikasi:* Seluruh request mengembalikan status code `404 Not Found`. Berkas database SQLite aman dan tidak bocor melalui HTTP. Bukti: [`sesudah-18b-db-404.png`](blueteam/evidence/sesudah-18b-db-404.png).

---

### ATK-03: Web Admin Login Brute-Force & Credential Stuffing

* **Tahap 1 — Attack Execution:**
  Red Team melakukan simulasi serangan password spraying dan brute-force otomatis terhadap formulir login dashboard (`/api/login`):
  ```bash
  for i in {1..12}; do curl -s -X POST http://192.168.6.20/api/login -d "user=admin&pass=pass$i"; done
  ```
  *Hasil Sebelum:* Sistem terus memproses seluruh request login tanpa hambatan, memungkinkan penyerang menebak kredensial tanpa batas waktu.

* **Tahap 2 — Detection Analysis:**
  Security Onion mencatat tingginya volume koneksi HTTP POST dari `192.168.6.10` pada Zeek `http.log` dan log autentikasi rsyslog.

* **Tahap 3 — Defense Gap Analysis:**
  Endpoint login tidak dilengkapi mekanisme rate limiting atau account lockout.

* **Tahap 4 — Mitigation & Hardening:**
  1. Mengintegrasikan middleware `express-rate-limit` pada endpoint `/api/login`:
     * Maksimal: **10 request per menit per IP address**.
     * Respon pelanggaran: `HTTP 429 Too Many Requests`.
  2. Memperkuat algoritma hashing password menggunakan `scrypt` dengan salt acak dan membatasi masa berlaku sesi menjadi maksimal 12 jam.

* **Tahap 5 — Retest & Verification:**
  Red Team mengulang pengiriman request login berturut-turut sebanyak 12 kali:
  *Hasil Verifikasi:* Setelah percobaan ke-10, Target Node secara konsisten merespon dengan status code **`429 Too Many Requests`**. Percobaan brute-force penyerang berhasil dihentikan otomatis. Bukti: [`sesudah-19b-ratelimit.png`](blueteam/evidence/sesudah-19b-ratelimit.png).

---

### ATK-04: SSH Brute-Force & Unauthorized Access

* **Tahap 1 — Attack Execution:**
  Red Team mencoba melakukan brute-force kredensial SSH terhadap user `root` dan user sistem lainnya dari Kali Linux:
  ```bash
  ssh root@192.168.6.20
  hydra -l analyst -P /usr/share/wordlists/rockyou.txt ssh://192.168.6.20
  ```
  *Hasil Sebelum:* Port SSH meminta password dan merespon prompt login root. Bukti: [`sebelum-1-ssh.png`](blueteam/evidence/sebelum-1-ssh.png), [`sebelum-5-login.png`](blueteam/evidence/sebelum-5-login.png).

* **Tahap 2 — Detection Analysis:**
  Syslog Target Node mencatat event `Failed password for root from 192.168.6.10 port ... ssh2`. Security Onion mengindeks event ke `logstash-syslog-*` dan Zeek menandai kegagalan handshake SSH berulang.

* **Tahap 3 — Defense Gap Analysis:**
  Konfigurasi OpenSSH default masih mengizinkan `PermitRootLogin yes`, `PasswordAuthentication yes`, serta belum ada jail Fail2ban yang aktif. Bukti: [`sebelum-2-fail2ban.png`](blueteam/evidence/sebelum-2-fail2ban.png).

* **Tahap 4 — Mitigation & Hardening:**
  1. Menyesuaikan `/etc/ssh/sshd_config.d/hardening.conf`:
     ```text
     PermitRootLogin no
     PasswordAuthentication no
     PubkeyAuthentication yes
     MaxAuthTries 3
     AllowUsers kel6
     X11Forwarding no
     ```
     Bukti: [`sesudah-1a-konfigurasi-ssh.png`](blueteam/evidence/sesudah-1a-konfigurasi-ssh.png), [`sesudah-1b-ssh-efektif.png`](blueteam/evidence/sesudah-1b-ssh-efektif.png).
  2. Mengaktifkan daemon Fail2ban (`sshd` jail) untuk memonitor log autentikasi dan memblokir IP yang melanggar. Bukti: [`sesudah-2a-konfigurasi-fail2ban.png`](blueteam/evidence/sesudah-2a-konfigurasi-fail2ban.png), [`sesudah-2b-status-fail2ban.png`](blueteam/evidence/sesudah-2b-status-fail2ban.png).

* **Tahap 5 — Retest & Verification:**
  Red Team mencoba login SSH dari Kali Linux menggunakan password:
  ```bash
  ssh kel6@192.168.6.20
  ```
  *Hasil Verifikasi:* Server menolak koneksi sebelum prompt password muncul:
  ```text
  kel6@192.168.6.20: Permission denied (publickey).
  ```
  Percobaan koneksi berulang 6 kali langsung digagalkan dan koneksi berikutnya ditutup paksa. Bukti: [`sesudah-6-login-ditolak.png`](blueteam/evidence/sesudah-6-login-ditolak.png), [`sesudah-9a-bruteforce.png`](blueteam/evidence/sesudah-9a-bruteforce.png).

---

### ATK-05: Unauthorized RTSP Stream Access & Protocol Abuse

* **Tahap 1 — Attack Execution:**
  Red Team mencoba membaca feed kamera RTSP dan memindai port multimedia:
  ```bash
  ffprobe -v error -rtsp_transport tcp rtsp://192.168.6.20:554/live/stream
  ```
  *Hasil Sebelum:* MediaMTX mendengarkan pada berbagai port (RTMP, HLS, WebRTC, SRT, API). Siapa pun di jaringan lokal dapat mengakses feed RTSP secara bebas.

* **Tahap 2 — Detection Analysis:**
  Zeek mendeteksi sesi RTSP dan traffic multimedia pada port non-standar. Syslog Security Onion memetakan aktivitas komunikasi streaming.

* **Tahap 3 — Defense Gap Analysis:**
  Fitur default MediaMTX mengaktifkan seluruh modul protokol streaming yang tidak digunakan oleh sistem replika kandang sapi, memperluas bidang serangan secara tidak perlu.

* **Tahap 4 — Mitigation & Hardening:**
  1. Menutup protokol yang tidak digunakan di `mediamtx.yml`:
     ```yaml
     rtmp: no
     hls: no
     webrtc: no
     srt: no
     api: no
     metrics: no
     playback: no
     ```
     Bukti: [`sesudah-4a-konfigurasi-mediamtx.png`](blueteam/evidence/sesudah-4a-konfigurasi-mediamtx.png).
  2. Mengonfigurasi UFW agar port 554 hanya dapat dijangkau oleh Security Onion (`192.168.6.30`):
     ```bash
     sudo ufw allow from 192.168.6.30 to any port 554 proto tcp
     ```

* **Tahap 5 — Retest & Verification:**
  1. Pemeriksaan port aktif pada target membuktikan hanya port RTSP yang tersisa dari MediaMTX. Bukti: [`sesudah-4b-port-listen.png`](blueteam/evidence/sesudah-4b-port-listen.png).
  2. Pengujian dari Kali Linux (`192.168.6.10`):
     * Pemindaian port 554 berstatus **`filtered`**.
     * Request stream `ffprobe` mengalami *Connection timed out* / ditolak firewall. Hanya node monitoring berizin yang dapat mengakses feed.

---

### ATK-06: Privilege Escalation & Weak File Permissions

* **Tahap 1 — Attack Execution:**
  Red Team melakukan audit izin berkas konfigurasi kritis dan database pada target:
  *Hasil Sebelum:* File konfigurasi `mediamtx.yml` dan `app.js` memiliki izin baca/tulis dunia luar (`666` atau `644`), memungkinkan user lain membaca kunci/konfigurasi rahasia. Bukti: [`sebelum-7-izin-berkas.png`](blueteam/evidence/sebelum-7-izin-berkas.png).

* **Tahap 2 — Detection Analysis:**
  Inspeksi lokal dan log file integrity monitoring menunjukkan potensi modifikasi tanpa izin.

* **Tahap 3 — Defense Gap Analysis:**
  Ketiadaan prinsip least privilege pada penataan kepemilikan dan permission berkas aplikasi.

* **Tahap 4 — Mitigation & Hardening:**
  Mengubah permission berkas konfigurasi dan script aplikasi menjadi ketat:
  ```bash
  chmod 640 app.js mediamtx.yml
  chown kel6:kel6 app.js mediamtx.yml
  ```
  Memastikan paket sistem selalu terupdate dengan `apt update && apt upgrade`. Bukti: [`sesudah-8b-tidak-ada-pembaruan.png`](blueteam/evidence/sesudah-8b-tidak-ada-pembaruan.png).

* **Tahap 5 — Retest & Verification:**
  Verifikasi izin berkas menggunakan `ls -la`:
  ```text
  -rw-r----- 1 kel6 kel6  app.js
  -rw-r----- 1 kel6 kel6  mediamtx.yml
  -rw-r----- 1 kel6 kel6  smartcow.db
  ```
  User lain dan service unprivileged tidak dapat membaca maupun memodifikasi berkas konfigurasi. Bukti: [`sesudah-7a-izin-berkas.png`](blueteam/evidence/sesudah-7a-izin-berkas.png).

---

## 5. Kesimpulan Siklus Purple Team

Pelaksanaan Purple Team Loop berhasil membuktikan bahwa pendekatan terpadu antara pengujian ofensif (Red Team) dan penguatan defensif (Blue Team) menghasilkan pertahanan berlapis (*defense-in-depth*) yang efektif:

1. **Visibilitas Penuh:** Seluruh aktivitas jaringan dan log sistem dari Target Node berhasil dialirkan ke Security Onion (`192.168.6.30`) dan divisualisasikan secara real-time di Kibana Discover. Bukti: [`monitoring-baseline-log.png`](blueteam/evidence/monitoring-baseline-log.png).
2. **Mitigasi Efektif:** Seluruh 6 skenario serangan yang disimulasikan dari Kali Linux (`192.168.6.10`) berhasil dimitigasi dan terverifikasi gagal pada Tahap 5 pengujian ulang.
3. **Penyelarasan Nilai Akhir:** Hasil siklus ini menjadi landasan empiris bagi penyusunan **Incident Report Final (NIST SP 800-61r2)** dan materi slide presentasi akhir.
