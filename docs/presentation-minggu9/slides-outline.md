# Outline Slide — Presentasi Minggu 9

Maks. 6–8 slide. Konversi ke PDF sebelum praktikum: `docs/presentation-minggu9/slides.pdf`.

## Slide 1 — Judul
- Judul: Simulasi SOC untuk Backend IoT
- Kelompok 6 / Kelas A
- Anggota + peran

## Slide 2 — Konteks Proyek IoT
- Produk acuan: SmartCow — monitoring kandang sapi (suhu, kelembapan, kamera)
- Masalah keamanan: data operasional, ketersediaan, akses tidak sah
- Fokus: backend (server, dashboard, API), bukan sensor fisik

## Slide 3 — Deklarasi Tech Stack & Replika
- Tabel: tech asli → replika Target Node
- Target: Ubuntu Server 26.04, Node.js, SQLite, SSH, RTSP
- Data dummy, bukan produksi

## Slide 4 — Arsitektur Simulasi
- `topology.png`
- IP: Attacker `.10`, Target `.20`, Monitoring `.30`
- SPAN/mirror ke Security Onion

## Slide 5 — Progres Hardening & Baseline
- 17 kontrol: A 7, B 5, C 5
- Sebelum/sesudah: port terbuka, SSH, fail2ban, SQLite
- Baseline traffic: syslog, NTP, RTSP

## Slide 6 — Live Demo Security Onion
- Tampilkan alert/log
- Contoh: ping/HTTP/RTSP dummy
- Catatan: belum perlu serangan sungguhan

## Slide 7 — Attack Plan Fase 2
| Skenario | Tools | Target |
|---|---|---|
| Recon | nmap | `.20:22/80/554` |
| SSH brute force | hydra | `.20:22` |
| RTSP access | ffprobe | `.20:554` |

## Slide 8 — Kendala & Pembelajaran
- Kendala: RAM/disk, DHCP management, Elastic Stack
- Pembelajaran: pentingnya bukti before/after dan baseline
- Rencana Fase 2: Purple Team Loop 5 tahap
