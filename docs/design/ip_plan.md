# IP Plan — Kelompok 6, Kelas A

**Segmen lab:** `192.168.6.0/24`
**Skenario:** replika backend IoT / IP camera (web admin, SSH, RTSP)

| Hostname | IP Address | OS Direncanakan | Peran |
|---|---|---|---|
| `attacker-kel6` | `192.168.6.10` | Kali Linux 2026.2 | Attacker / Red Team |
| `target-kel6` | `192.168.6.20` | Ubuntu Server 26.04.1 LTS | Target / korban |
| `monitor-kel6` | `192.168.6.30` | Security Onion | Monitoring / Blue Team |

## Interface target

| Interface | IP | Fungsi |
|---|---|---|
| `enp0s3` | `192.168.6.20/24` | Internal Network `siber-kel6` |
| `enp0s8` | DHCP `10.0.3.15/24` | NAT (internet) |
| `enp0s9` | DHCP `192.168.1.24/24` | Management / SSH dari host |

## Layanan target

| Layanan | Port | Keterangan |
|---|---|---|
| OpenSSH | 22 | key-based auth, password auth off |
| Node.js | 80 | web admin replika |
| MediaMTX RTSP | 554 | stream video; UFW hanya mengizinkan `192.168.6.30` |
| rsyslog | 514/udp | kirim log ke `192.168.6.30` |
| chrony | 123/udp | NTP server untuk segmen lab |

## Catatan
- IP management `enp0s9` masih DHCP, bisa berubah (`.23` → `.24`).
- Security Onion sebaiknya punya interface monitoring/SPAN agar melihat trafik antar-node.
- Subnet ini harus unik dan tidak dipakai kelompok lain.
