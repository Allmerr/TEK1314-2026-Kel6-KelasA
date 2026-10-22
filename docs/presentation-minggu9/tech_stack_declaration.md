# Deklarasi Tech Stack — Minggu 9

**Kelompok:** 6 / Kelas A  
**Produk acuan:** SmartCow — Monitoring Kandang Sapi Berbasis IoT  
**Status:** Final, 22 Oktober 2026

## 1. Ringkasan Produk

SmartCow merupakan konsep sistem monitoring kandang sapi berbasis IoT. Sensor suhu dan kelembapan mengirimkan data ke gateway melalui HTTP REST, sedangkan kamera IP mengirimkan video menggunakan RTSP.

Dashboard web digunakan untuk menampilkan data sensor dan status kamera. Data sensor disimpan secara lokal di gateway.

SmartCow digunakan sebagai produk acuan untuk menentukan tech stack Target Node. Produk ini bersifat konseptual dan tidak harus menggunakan produk mitra. Penilaian difokuskan pada kesesuaian tech stack proyek acuan dengan Target Node.

## 2. Pemetaan Tech Stack

| Komponen | Proyek IoT Acuan | Target Node |
|---|---|---|
| Sistem operasi | Ubuntu Server LTS pada gateway | Ubuntu Server 26.04.1 LTS |
| Database | SQLite lokal di gateway | SQLite (`node:sqlite`), file berizin `640` |
| Komunikasi sensor | HTTP REST/JSON | HTTP REST pada port `80` |
| Komunikasi kamera | RTSP | RTSP pada port `554` |
| Dashboard | Web Node.js untuk data sensor, status kamera, dan login admin | Web Node.js pada port `80`, login lokal, dan data dari SQLite |
| Akses administrasi | SSH key-based dan web admin | SSH pada port `22` dengan autentikasi key, password authentication nonaktif, serta login web |
| Monitoring keamanan | Syslog ke collector terpusat | Syslog ke Security Onion (`192.168.6.30`) |

## 3. Kesesuaian dan Perbedaan

### Komponen yang direplikasi

Target Node menggunakan komponen yang mendekati rancangan sistem acuan, meliputi:

- Ubuntu Server sebagai sistem operasi backend dengan layanan yang dikelola `systemd`.
- SQLite sebagai penyimpanan lokal, termasuk skema data sensor.
- HTTP REST untuk pertukaran data sensor dan RTSP untuk video kamera.
- Dashboard Node.js untuk menampilkan data sensor, status kamera, dan autentikasi pengguna.
- SSH key-based untuk akses administrasi server.

### Komponen yang tidak direplikasi

Beberapa bagian sistem acuan belum diterapkan pada Target Node:

- **Sensor dan aktuator fisik:** data diganti dengan data dummy atau sintetis.
- **TLS pada dashboard:** replika masih menggunakan HTTP di jaringan lab.
- **High availability dan load balancer:** belum diterapkan karena replika menggunakan satu node.
- **Database terpusat:** penyimpanan masih menggunakan SQLite lokal.

## 4. Catatan Implementasi

Target Node merupakan replika untuk kebutuhan praktikum, bukan sistem produksi SmartCow. Seluruh data sensor yang digunakan bersifat dummy atau sintetis.

Firebase tidak digunakan lagi. Backend dan penyimpanan data berjalan secara lokal pada Target Node.

Perbedaan utama dengan rancangan acuan adalah belum adanya TLS pada dashboard dan penggunaan perangkat sensor fisik. Keduanya menjadi keterbatasan implementasi dan tidak dihitung sebagai kontrol keamanan yang telah diterapkan.

