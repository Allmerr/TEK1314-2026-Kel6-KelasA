# TEK1314-2026-Kel6-KelasA



# PBL Keamanan Siber — Kelompok 6

## Skenario Proyek

Proyek ini merancang sistem IoT untuk memantau suhu dan kelembapan kandang sapi serta mengaktifkan penyemprot secara otomatis. Sensor DHT22 terhubung ke ESP32 untuk membaca kondisi kandang. ESP32 mengirimkan data melalui Wi-Fi menggunakan MQTT ke server Ubuntu untuk ditampilkan pada dashboard. Kendali penyemprot dilakukan secara lokal oleh ESP32 melalui relay berdasarkan ambang suhu yang akan ditentukan, dengan batas waktu pengoperasian pompa.

Skenario keamanan berfokus pada pengujian komunikasi antara ESP32 dan server dalam jaringan laboratorium. Attacker Node menggunakan Kali Linux untuk melakukan reconnaissance dan menguji kontrol akses layanan MQTT. Monitoring Node menggunakan Wireshark untuk mengamati trafik serta membantu mengidentifikasi aktivitas pengujian dan anomali. SPAN digunakan untuk menyalin trafik port server ke Monitoring Node. ARP spoofing merupakan alternatif skenario pengujian MITM di laboratorium yang memerlukan pengaturan alur trafik tersendiri, bukan pengganti langsung konfigurasi SPAN.

## Rancangan Jaringan

Jaringan kelompok menggunakan subnet **192.168.60.0/24** dengan alokasi IP statis.

| Node | IP Address | Peran |
|---|---|---|
| attacker-06 | 192.168.60.10 | Perangkat pengujian Red Team |
| server-iot-06 | 192.168.60.20 | Target pengujian, MQTT broker, dan dashboard |
| monitor-06 | 192.168.60.30 | Analisis trafik oleh Blue Team; IP adaptor manajemen |
| esp32-kandang-06 | 192.168.60.40 | Pembacaan sensor dan kendali penyemprot |

## Dokumen Perancangan

- [Topologi jaringan](docs/design/topology.png)
- [IP plan dan rencana OS](docs/design/ip_plan.md)

## Status

Proyek berada pada tahap perancangan arsitektur dan skema IP. Instalasi perangkat, konfigurasi layanan, dan pengujian keamanan belum dilakukan.
