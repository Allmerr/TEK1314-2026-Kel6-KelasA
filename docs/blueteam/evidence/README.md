# Bukti Hardening — Target Node `192.168.6.20`

Diambil 21–22 Okt 2026. Nama file memakai bahasa Indonesia: `sebelum-*` (before) dan
`sesudah-*` (after). Semua waktu di screenshot adalah **UTC** (WIB = UTC+7).

## A. Access & Authentication

| Sebelum | Sesudah | Isi |
|---|---|---|
| `sebelum-1-ssh.png` | `sesudah-1a-konfigurasi-ssh.png` | isi `sshd_config` |
| `sebelum-5-login.png` | `sesudah-1b-ssh-efektif.png` | nilai efektif `sshd -T` |
| — | `sesudah-6-login-ditolak.png` | login password ditolak dari Kali |
| `sebelum-2-fail2ban.png` | `sesudah-2a-konfigurasi-fail2ban.png` | `jail.local` |
| — | `sesudah-2b-status-fail2ban.png` | service fail2ban aktif |
| — | `sesudah-9a-bruteforce.png` | 6x gagal + koneksi ke-7 ditolak |

## B. Service & Attack Surface

| Sebelum | Sesudah | Isi |
|---|---|---|
| `sebelum-3-firewall.png` | `sesudah-3a-aturan-firewall.png` | aturan UFW |
| — | `sesudah-3b-status-firewall.png` | status UFW verbose |
| `sebelum-4-port.png` | `sesudah-4b-port-listen.png` | `ss -tlnp` |
| `sebelum-6-pindai-nmap.png` | `sesudah-5-pindai-nmap.png` | `nmap -sV -p-` |
| — | `sesudah-4a-konfigurasi-mediamtx.png` | alamat port MediaMTX |
| — | `sesudah-8b-tidak-ada-pembaruan.png` | `apt list --upgradable` kosong |

## C. Data & Application

| Sesudah | Isi |
|---|---|
| `sesudah-7a-izin-berkas.png` | permission `app.js` & `mediamtx.yml` 640 |
| `sesudah-18a-izin-db.png` | permission `smartcow.db`, `.db-wal`, `.db-shm` 640 |
| `sesudah-18b-db-404.png` | 5 path DB termasuk traversal → `404` |
| `sesudah-19b-ratelimit.png` | 12x login gagal → `429` (IP sudah kena limit) |

## D. Monitoring (bukan bukti hardening)

| File | Isi |
|---|---|
| `monitoring-baseline-log.png` | Kibana Discover Security Onion, query `192.168.6.20`; log syslog target masuk |

## Belum ada

- `sesudah-9b-blokir-fail2ban.png` — `fail2ban-client status sshd` + `iptables -L f2b-sshd`
- `sesudah-7b-akses-ditolak.png` — `sudo -u nobody cat app.js` → `Permission denied`
- `sesudah-8a-riwayat-patch.png` — `tail /var/log/apt/history.log`
- `sesudah-20-rtsp-30.png` — `ufw status numbered` (554 hanya dari `.30`)
- `monitoring-alert-serangan.png` — halaman **Alerts** Security Onion (bukan Discover)

## Salah konfigurasi (bukan bukti after)

| File | Isi |
|---|---|
| `salah-konfigurasi-ufw-544.png` | UFW membuka port **544**, bukan 554; `.30` tidak akan bisa RTSP |

Jangan pakai file itu sebagai bukti hardening. Simpan hanya sebagai catatan proses.

## Catatan

1. `sebelum-1-ssh.png` menampilkan `X11Forwarding yes`, sedangkan `sesudah-1b` menunjukkan
   nilai efektif `no`. Nilai `no` berasal dari drop-in di `/etc/ssh/sshd_config.d/` yang
   belum terdokumentasi — perlu screenshot tambahan.
2. `sesudah-4a-konfigurasi-mediamtx.png` hanya menampilkan alamat port, belum flag
   `rtmp: no`, `hls: no`, dst. Bukti port tertutup saat ini bertumpu pada `ss` dan nmap.
3. Target punya tiga IP: `192.168.6.20` (internal), `10.0.3.15` (NAT), dan `192.168.1.28`
   (management, DHCP — bergeser dari `.24`).
4. Zona waktu: target UTC, Kali `-0400` (selisih 4 jam).
