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
* **Total Durasi:** 07 menit 15 detik (Memenuhi ketentuan standar: 5–8 menit)

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
| **00:00 – 00:45** | 45 dtk | **Pendahuluan & Topologi Lab** | Tampilan arsitektur 3 VM di VirtualBox dan CLI `ip a` dari ketiga node. | Penjelasan identitas Kelompok 6 Kelas A, sistem replika Smart Cow, dan pembagian IP subnet `192.168.6.0/24`. |
| **00:45 – 01:45** | 60 dtk | **Tahap 1: Eksekusi Serangan Awal** | Terminal Kali Linux (`.10`) menjalankan Nmap, Hydra HTTP login, dan ffplay RTSP. | Red Team menunjukkan bahwa sistem awal rentan: port 554 RTSP terbuka, brute force login direspon HTTP 200, dan stream CCTV dapat dibajak. |
| **01:45 – 02:45** | 60 dtk | **Tahap 2: Cek Deteksi SIEM Baseline** | Browser Kibana Discover pada Security Onion (`.30`). | Blue Team memverifikasi kemunculan log insiden: Zeek `notice.log` (Port Scan), Zeek `http.log` (lonjakan HTTP 401), dan Syslog rsyslog `Failed password`. |
| **02:45 – 03:45** | 60 dtk | **Tahap 3: Analisis Ancaman & Kesenjangan** | Diagram alur mitigasi dan evaluasi resiko NIST SP 800-61r2. | Penjelasan akar masalah: ketiadaan rate limiting, port listener tidak terbatas, dan izin akses file yang terlalu longgar. |
| **03:45 – 05:15** | 90 dtk | **Tahap 4: Penerapan Kontrol Mitigasi** | Terminal SSH Target Node (`.20`). | Blue Team menerapkan kontrol hardening secara live: aktivasi UFW firewall, injeksi modul `express-rate-limit`, penguncian `sshd_config`, dan `chmod 640`. |
| **05:15 – 06:45** | 90 dtk | **Tahap 5: Pengujian Ulang (Retest)** | Terminal Kali Linux berdampingan dengan Kibana Discover live log. | Red Team mengulang seluruh serangan. Hasil: Nmap port 554 *filtered*, HTTP login ke-11 diblokir kode *429 Too Many Requests*, dan SSH password ditolak (*Permission denied*). Status: **100% Pass**. |
| **06:45 – 07:15** | 30 dtk | **Kesimpulan & Penutup** | Ringkasan matriks verifikasi dan repositori GitHub Kelompok 6. | Pernyataan penutup mengenai keberhasilan mitigasi berlapis (*defense-in-depth*) dan lessons learned tim. |

---

## 4. Naskah Narasi Lengkap (Voice-Over / Text Script)

### [00:00 - 00:45] Pembukaan & Topologi
> *"Halo, selamat pagi/siang dosen penguji dan rekan-rekan. Kami dari Kelompok 6 Kelas A mempersembahkan video demonstrasi Purple Team Loop dan penanganan insiden siber pada replika Smart Cow Monitoring System. Topologi lab kami terdiri dari tiga node virtual pada subnet 192.168.6.0/24: Attacker Node pada 192.168.6.10, Target Node backend IoT pada 192.168.6.20, dan Monitoring Node Security Onion pada 192.168.6.30."*

### [00:45 - 01:45] Tahap 1: Serangan Awal
> *"Pada tahap pertama, Red Team melakukan pengujian agresif. Ketika penyerang memindai port 554 RTSP, port tersebut berstatus open. Kemudian, penyerang melancarkan credential stuffing terhadap API login web. Server backend merespons seluruh permintaan tanpa batasan kecepatan. Penyerang juga dapat menyedot live feed video CCTV kandang sapi secara langsung tanpa hambatan."*

### [01:45 - 02:45] Tahap 2: Deteksi SIEM
> *"Beralih ke Security Onion pada IP 192.168.6.30. Melalui antarmuka Kibana Discover, kita dapat melihat bahwa seluruh anomali ini terekam dengan jelas. Zeek NSM mencatat pola SYN Stealth Scan pada notice.log, lonjakan status respons HTTP 401 pada http.log, serta ratusan entri audit syslog failed password yang dikirimkan secara real-time via UDP 514."*

### [02:45 - 03:45] Tahap 3: Analisis Kesenjangan
> *"Berdasarkan deteksi tersebut, Blue Team mengklasifikasikan insiden ini ke dalam kategori Reconnaissance, Brute Force Attrition, dan Data Interception sesuai pedoman NIST SP 800-61r2. Kami mengidentifikasi bahwa ketiadaan packet filtering dan middleware pembatas laju adalah akar masalah utama."*

### [03:45 - 05:15] Tahap 4: Penerapan Mitigasi
> *"Masuk ke tahap mitigasi pada Target Server 192.168.6.20. Blue Team mengaktifkan firewall UFW dengan kebijakan default deny, membatasi akses streaming hanya untuk entitas terpercaya. Selanjutnya, kami mengintegrasikan middleware express-rate-limit dengan kuota 10 request per menit pada endpoint login, mengunci otentikasi SSH hanya berbasis Public Key, dan merestriksi izin berkas ke 640."*

### [05:15 - 06:45] Tahap 5: Verifikasi Pengujian Ulang
> *"Sekarang saatnya pembuktian di Tahap 5. Red Team dari Kali Linux meluncurkan kembali serangan yang sama persis. Pertama, pemindaian Nmap port 554 kini berstatus FILTERED. Kedua, saat pengujian kamus login diulang, tepat pada percobaan ke-11, server langsung memblokir penyerang dengan respons HTTP 429 Too Many Requests. Ketiga, percobaan login SSH via password langsung digagalkan dengan pesan Permission denied publickey. Seluruh serangan berhasil ditangkal 100%!"*

### [06:45 - 07:15] Penutup
> *"Demonstrasi ini membuktikan bahwa strategi pertahanan berlapis (defense-in-depth) berhasil menutup celah keamanan secara tuntas. Terima kasih atas perhatian Bapak/Ibu dosen penguji."*

---

## 5. Hubungan dengan Berkas Bukti Digital (Evidence Index)

Tangkapan layar hasil tahapan video ini terarsip lengkap pada folder repositori:
* Bukti Deteksi Port Scan: `docs/blueteam/evidence/sebelum-3a-pindai-port.png` vs `docs/blueteam/evidence/sesudah-5-pindai-nmap.png`
* Bukti Rate Limiting Web: `docs/blueteam/evidence/sesudah-19b-ratelimit.png`
* Bukti Penolakan SSH: `docs/blueteam/evidence/sesudah-6-login-ditolak.png`
* Bukti Log SIEM Kibana: `docs/blueteam/evidence/sesudah-1-kibana-siem.png`
