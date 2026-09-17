# TEK1314-2026-Kel06-KelasA — Proyek PBL Keamanan Siber

**Mata Kuliah:** TEK1314 - Keamanan Siber (D4 Teknologi Rekayasa Komputer)
**Kelompok:** 6 — Kelas A
**Fase:** Design Phase (Pertemuan ke-4 / Minggu ke-3 Proyek PBL)

---

## Deskripsi Skenario

Proyek ini merancang sebuah medan perang jaringan (network lab) yang terdiri dari tiga peran utama: penyerang (Red Team), target, dan pemantau (Blue Team). Seluruh node dirancang berada dalam satu segmen jaringan 192.168.8.0/24, sesuai dengan kewajiban IP unik per kelompok berdasarkan Kontrak Kuliah Poin 3a.

Lingkungan ini digunakan untuk mensimulasikan aktivitas serangan terhadap aplikasi web yang memiliki kerentanan secara terkontrol, kemudian mengamati dan menganalisis aktivitas tersebut menggunakan Security Onion.

| Node                 | Peran                     | OS                | IP            |
| -------------------- | ------------------------- | ----------------- | ------------- |
| Attacker Node        | Red Team (penyerang)      | Kali Linux        | 192.168.6.10 |
| Target Node (Korban) | Server yang dieksploitasi | Ubuntu Server CLI | 192.168.6.20   |
| Monitoring Node      | Blue Team (pemantau)      | Security Onion    | 192.168.6.30 |

### Target & Layanan

Target adalah server web yang menjalankan layanan yang sengaja memiliki celah untuk didemonstrasikan:

- **Apache2 + PHP (port 80)** → potensi SQL Injection, XSS, file upload
- **OpenSSH (port 22)** → potensi brute-force & user enumeration
- **MySQL (port 3306)** → potensi brute-force kredensial / misconfig autentikasi

Alur serangan diharapkan dari **Attacker → Target**, sementara seluruh lalu lintas dipantau oleh **Monitoring Node (Security Onion)** untuk mendeteksi aktivitas mencurigakan (port scan, login berulang, anomali HTTP).

## Struktur & Dokumen Pendukung

```
docs/design/
├── topology.png   # Gambar desain jaringan (Attacker, Target, Monitoring)
├── ip_plan.md     # Tabel Hostname, IP Address, dan OS yang direncanakan
└── README.md      # Deskripsi skenario ini
```

Link deliverables:

- [Topologi](docs/design/topology.png)
- [IP Plan](docs/design/ip_plan.md)

---

## Anggota Tim & Peran

| Nama                       | NIM         | Peran                          |
| -------------------------- | ----------- | ------------------------------ |
| Hodelia Victory Dorola Marbun| J0404241100 | Lead                           |
| Muhammad Kevin Almer | J0404241163 | Red Team (Attacker)            |
| Rizki Ramadhani              | J0404241144 | Blue Team (Defender / Network) |
| Yuwandri Alfarizi        | J0404241010 | Blue Team (Defender / Network) |
