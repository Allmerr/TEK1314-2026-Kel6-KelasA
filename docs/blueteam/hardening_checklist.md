# Hardening Checklist — Target Node

**Target:** `192.168.6.20` (`target-kel6`)  
**OS:** Ubuntu Server 26.04.1 LTS  
**Standar:** Minimal 8 kontrol, dengan minimal 2 kontrol di setiap kategori A, B, dan C. Setiap kontrol dilengkapi bukti sebelum dan sesudah konfigurasi.

Seluruh bukti tersimpan di folder [`evidence/`](evidence/README.md). File dengan awalan `sebelum-` berisi kondisi awal, sedangkan `sesudah-` berisi hasil setelah hardening.

Waktu pada screenshot target menggunakan **UTC**.

## Ringkasan Status

| Kategori | Selesai | Sebagian | Belum | Total |
|---|---:|---:|---:|---:|
| A. Access & Authentication | 5 | 1 | 1 | 7 |
| B. Service & Attack Surface | 5 | 0 | 0 | 5 |
| C. Data & Application | 5 | 0 | 0 | 5 |
| **Total** | **15** | **1** | **1** | **17** |

## A. Access & Authentication

| Kontrol | Status | Bukti |
|---|---|---|
| Menonaktifkan SSH root login (`PermitRootLogin no`) | Selesai | [Sebelum](evidence/sebelum-1-ssh.png) · [Konfigurasi](evidence/sesudah-1a-konfigurasi-ssh.png) · [Efektif](evidence/sesudah-1b-ssh-efektif.png) |
| Menonaktifkan password authentication dan mewajibkan SSH key | Selesai | [Sebelum](evidence/sebelum-1-ssh.png) · [Efektif](evidence/sesudah-1b-ssh-efektif.png) · [Login ditolak](evidence/sesudah-6-login-ditolak.png) |
| Membatasi percobaan autentikasi dengan `MaxAuthTries 3` | Selesai | [Konfigurasi efektif](evidence/sesudah-1b-ssh-efektif.png) |
| Membatasi akses SSH ke user `kel6` (`AllowUsers kel6`) | Selesai | [Konfigurasi efektif](evidence/sesudah-1b-ssh-efektif.png) |
| Menonaktifkan X11 forwarding | Selesai | [Konfigurasi efektif](evidence/sesudah-1b-ssh-efektif.png) — nilai berasal dari drop-in SSH (lihat Catatan 1). |
| Mengaktifkan Fail2ban untuk SSH | Sebagian | [Sebelum](evidence/sebelum-2-fail2ban.png) · [Konfigurasi](evidence/sesudah-2a-konfigurasi-fail2ban.png) · [Status](evidence/sesudah-2b-status-fail2ban.png) · [Uji 6 kali gagal](evidence/sesudah-9a-bruteforce.png) |

## B. Service & Attack Surface

| Kontrol | Status | Bukti |
|---|---|---|
| Mengaktifkan UFW dengan kebijakan default `deny (incoming)` | Selesai | [Sebelum](evidence/sebelum-3-firewall.png) · [Aturan](evidence/sesudah-3a-aturan-firewall.png) · [Status](evidence/sesudah-3b-status-firewall.png) |
| Membatasi akses port 22, 80, dan 123 ke `192.168.6.0/24` | Selesai | [Sebelum](evidence/sebelum-4-port.png) · [Port listen](evidence/sesudah-4b-port-listen.png) |
| Membatasi akses RTSP port 554 hanya dari `192.168.6.30` | Selesai | [Port listen](evidence/sesudah-4b-port-listen.png). Bukti `ufw status numbered` belum tersedia. |
| Menonaktifkan protokol MediaMTX yang tidak digunakan: RTMP, HLS, WebRTC, SRT, API, metrics, dan playback | Selesai | [Konfigurasi](evidence/sesudah-4a-konfigurasi-mediamtx.png) · [Port listen](evidence/sesudah-4b-port-listen.png) — lihat Catatan 2. |
| Memeriksa dan menerapkan update OS serta paket | Selesai | [Tidak ada pembaruan](evidence/sesudah-8b-tidak-ada-pembaruan.png) |

## C. Data & Application

| Kontrol | Status | Bukti |
|---|---|---|
| Membatasi permission file konfigurasi `mediamtx.yml` menjadi `640` | Selesai | [Sebelum](evidence/sebelum-7-izin-berkas.png) · [Sesudah](evidence/sesudah-7a-izin-berkas.png) |
| Membatasi permission database SQLite menjadi `640` (DB, WAL, dan SHM) | Selesai | [Permission database](evidence/sesudah-18a-izin-db.png) |
| Memastikan database tidak dapat diakses melalui HTTP | Selesai | [Pengujian 404 dan path traversal](evidence/sesudah-18b-db-404.png) |
| Menerapkan autentikasi lokal menggunakan password scrypt dan session 12 jam | Selesai | Bukti login berhasil dan data tabel `users` di SQLite (screenshot terbaru). |
| Membatasi login hingga 10 percobaan per menit per IP | Selesai | [Respons 429](evidence/sesudah-19b-ratelimit.png) — seluruh percobaan uji mendapat respons 429 karena IP sudah terkena limit. |

