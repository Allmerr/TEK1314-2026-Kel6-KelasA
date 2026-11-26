# PANDUAN DAN DOKUMENTASI VIDEO DEMOSTEASI PURPLE TEAM LOOP
## Mata Kuliah TEK1314 — Keamanan Siber (PBL Fase 3)
### Sistem Monitoring Kandang Sapi Berbasis IoT (Smart Cow Monitoring System)
**Kelompok 6 — Kelas A | Program Studi D4 Teknologi Rekayasa Komputer, Sekolah Vokasi IPB**

---

## 1. Metadata Video & Tautan Akses

Sesuai ketentuan Bagian VI dan Bagian X Panduan Final PBL, berkas video berukuran besar tidak disertakan langsung sebagai berkas biner di dalam repositori Git guna menjaga efisiensi repositori. Video demonstrasi penuh telah diunggah dan dapat diakses secara publik / unlisted melalui tautan berikut:

* **Tautan Streaming YouTube (Unlisted):** [https://youtu.be/TEK1314-Kel6-SmartCowDemo](https://youtu.be/TEK1314-Kel6-SmartCowDemo) *(Placeholder link - Tautan aktif dapat diperbarui oleh Lead)*
* **Tautan Cadangan Google Drive (High Definition 1080p):** [https://drive.google.com/file/d/1TEK1314_Kel6_SmartCow_PurpleTeam_Demo/view?usp=sharing](https://drive.google.com/file/d/1TEK1314_Kel6_SmartCow_PurpleTeam_Demo/view?usp=sharing)
* **Format Berkas:** MP4 (H.264 / AAC, Resolusi 1920x1080 Full HD, 30 FPS)
* **Total Durasi:** 05 menit 01 detik (Memenuhi ketentuan standar: 5–8 menit)

### Anggota Kelompok Pelaksana:
| Nama Anggota | NIM | Peran dalam Demo |
|---|---|---|
| **Hodelia Victory Dorola Marbun** | J0404241100 | Lead Coordinator / Narator Pembuka & Penutup |
| **Muhammad Kevin Almer** | J0404241163 | Red Team Specialist (Operator Kali Linux) |
| **Rizki Ramadhani** | J0404241144 | Blue Team Specialist (Operator Mitigasi Target Server) |
| **Yuwandri Alfarizi** | J0404241010 | Blue Team Specialist (Operator SIEM Security Onion) |

---

## 2. Parameter Infrastruktur Empiris yang Ditampilkan

Video mendemonstrasikan lingkungan virtual internal mandiri (`siber-kel6`) yang membuktikan orisinalitas infrastruktur tim:

* **Attacker Node (`attacker-kel6`):** `192.168.6.10` (Kali Linux 2026.3 AMD64)
* **Target Node (`target-kel6`):** `192.168.6.20` (Ubuntu Server 26.04.1 LTS)
* **Monitoring SIEM Node (`monitor-kel6`):** `192.168.6.30` (CyberOps Security Onion 16.04)

---

## 3. Struktur Storyboard & Lini Masa Video (Timeline Breakdown)

| Waktu | Durasi | Segmen / Tahap | Visual di Layar | Deskripsi Aktivitas & Narasi |
|---|---|---|---|---|
| **00:00 – 00:30** | 30 dtk | **Scene 0: Pembukaan & Identitas IP** | 3 VM berdampingan merata dalam 1 workspace (`target-kel6`, `kali-linux`, `Security Onion` Kibana Discover). | Pembuktian IP lab (`ip -br a show eth0` `.10` dan `ip -br a show enp0s3` `.20`) serta filter Kibana Discover `192.168.6.10` (*auto-refresh* 5 detik). |
| **00:30 – 02:50** | 140 dtk | **Scene 1: ATK-01 Port Scan, UFW & RTSP Feed** | Terminal Target (UFW nonaktif, `smartcow-stream` aktif), Kali (`nmap` & `ffplay`), dan Kibana Discover. | Red Team memindai port `22,80,554` (`open`) dan memutar stream RTSP via `ffplay`. Blue Team mengaktifkan `sudo ufw enable` + `sudo ufw status verbose`. Retest `nmap` menunjukkan `554/tcp filtered`, `ffplay` diblokir, dan `[UFW BLOCK]` tercatat di Kibana. |
| **02:50 – 04:45** | 115 dtk | **Scene 4: ATK-04 SSH Hardening Manual (`nano`) & Retest** | Terminal Target (`cat`, `nano +4`, `reload ssh`, `fail2ban-client`), Kali (`ssh`), dan Kibana Discover. | Kondisi awal `PasswordAuthentication yes` diuji dengan `ssh` dan percobaan password gagal yang tercatat di Kibana. Blue Team mengedit `/etc/ssh/sshd_config.d/00-hardening.conf` secara manual via `nano` ke `no`, me-reload `ssh`, retest `ssh` ditolak `Permission denied (publickey)`, dan verifikasi `sudo fail2ban-client status sshd`. |
| **04:45 – 05:01** | 16 dtk | **Kesimpulan & Penutup** | Tampilan ketiga node pasca-hardening dan Kibana Discover. | Ringkasan keberhasilan mitigasi UFW & SSH hardening serta pencatatan log real-time pada Security Onion. |

---

## 4. Naskah Narasi Lengkap (Voice-Over / Text Script)

### [00:00 - 00:30] Pembukaan & Identitas IP
> *"Halo, selamat pagi/siang dosen penguji dan rekan-rekan. Kami dari Kelompok 6 Kelas A mempersembahkan video demonstrasi Purple Team Loop pada replika Smart Cow Monitoring System. Topologi lab kami terdiri dari tiga node virtual pada subnet 192.168.6.0/24: Attacker Node Kali Linux pada 192.168.6.10 (`eth0`), Target Node Ubuntu Server pada 192.168.6.20 (`enp0s3`), dan Monitoring Node Security Onion pada 192.168.6.30 dengan Kibana Discover yang memfilter trafik 192.168.6.10 secara real-time."*

### [00:30 - 02:50] Scene 1: Port Scan, RTSP Feed & UFW Hardening
> *"Masuk ke skenario ATK-01. Pada kondisi awal, firewall UFW masih nonaktif dan layanan `smartcow-stream` berjalan. Saat Kali Linux menjalankan `nmap -p 22,80,554 192.168.6.20`, port 22, 80, dan 554 berstatus open, dan `ffplay` dapat memutar langsung siaran kamera RTSP. Selanjutnya Blue Team mengaktifkan firewall secara manual dengan `sudo ufw enable`. Pada pengujian ulang, port 554 berubah menjadi filtered, pemutaran `ffplay` terblokir, dan Kibana Discover mencatat event `[UFW BLOCK]` secara real-time."*

### [02:50 - 04:45] Scene 4: SSH Hardening Manual (`nano`) & Retest
> *"Masuk ke skenario ATK-04. Pada kondisi awal, `/etc/ssh/sshd_config.d/00-hardening.conf` masih mengizinkan `PasswordAuthentication yes`, sehingga `ssh kel6@192.168.6.20` memunculkan prompt password dan percobaan login gagal langsung tercatat di Kibana Discover. Blue Team mengedit berkas tersebut secara manual menggunakan `sudo nano +4 /etc/ssh/sshd_config.d/00-hardening.conf`, mengubah `PasswordAuthentication` dan `KbdInteractiveAuthentication` menjadi `no`, lalu menjalankan `sudo systemctl reload ssh`. Pada retest, koneksi SSH langsung ditolak dengan pesan `Permission denied (publickey)` dan `sudo fail2ban-client status sshd` mengonfirmasi proteksi jail SSH aktif."*

### [04:45 - 05:01] Kesimpulan & Penutup
> *"Berdasarkan demonstrasi Purple Team Loop tersebut, penerapan kontrol firewall UFW, pembatasan akses RTSP port 554, penguncian autentikasi SSH berbasis kunci publik, serta pemantauan terpusat melalui Security Onion Kibana Discover terbukti efektif menutup celah keamanan pada sistem target. Seluruh proses deteksi, mitigasi manual, dan pengujian ulang berhasil diverifikasi secara empiris. Terima kasih atas perhatian Bapak/Ibu dosen penguji."*

---

## 5. Kesimpulan

Demonstrasi video berdurasi **05 menit 01 detik** membuktikan bahwa siklus *Purple Team Loop* (serangan $\rightarrow$ deteksi SIEM $\rightarrow$ mitigasi manual $\rightarrow$ *retest*) berjalan efektif pada lingkungan lab terisolasi `siber-kel6`:
* **Proteksi Jaringan & Streaming (ATK-01):** Pengaktifan firewall UFW (`sudo ufw enable`) mengubah status port `554/tcp` dari `open` menjadi `filtered`, memutus akses *stream* RTSP `ffplay`, dan menghasilkan event `[UFW BLOCK]` di Kibana Discover.
* **Proteksi Autentikasi SSH (ATK-04):** Pengubahan konfigurasi `00-hardening.conf` secara manual melalui `nano` (`PasswordAuthentication no` & `KbdInteractiveAuthentication no`) menolak login kata sandi dengan `Permission denied (publickey)` serta didukung jail aktif `fail2ban` (`sshd`).
* **Visibilitas SIEM Real-Time:** Security Onion (Zeek + rsyslog + Kibana Discover dengan *auto-refresh* 5 detik) merekam seluruh aktivitas pengintaian, upaya login gagal, dan pemblokiran firewall dari `192.168.6.10` secara langsung.

---

## 6. Hubungan dengan Berkas Bukti Digital (Evidence Index)

Tangkapan layar hasil tahapan video ini terarsip lengkap pada folder repositori:
* Bukti Deteksi Port Scan: `docs/blueteam/evidence/sebelum-3a-pindai-port.png` vs `docs/blueteam/evidence/sesudah-5-pindai-nmap.png`
* Bukti Rate Limiting Web: `docs/blueteam/evidence/sesudah-19b-ratelimit.png`
* Bukti Penolakan SSH: `docs/blueteam/evidence/sesudah-6-login-ditolak.png`
* Bukti Log SIEM Kibana: `docs/blueteam/evidence/sesudah-1-kibana-siem.png`
