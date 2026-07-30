# STUDI KASUS BERBASIS PROYEK — HARI 5 (CAPSTONE)
## Go-Live Checklist "Nusantara Apps"
### (Integrasi Bab 11: Backup Restore + Bab 12: Software Management + Bab 13: Jaringan TCP/IP + Bab 14: FTP Server)

---

## INFORMASI UMUM

| Item | Keterangan |
|---|---|
| Digunakan setelah | Bab 11, 12, 13, & 14 selesai diajarkan (akhir Hari 5) |
| Durasi pengerjaan | 110–120 menit |
| Sifat | Individu, **wajib 2 VM** (App Server & File Server) |
| Prasyarat | VM Hari 1-4 (App Server) + **1 VM baru** (File Server) dalam 1 jaringan internal/host-only yang sama |
| Output akhir | 1 file `go-live-checklist.txt` di App Server |

## ⚠️ CATATAN PERSIAPAN LAB

- Siapkan **2 VM** dalam jaringan **host-only/internal network** yang sama:

| VM | Peran | Hostname | IP |
|---|---|---|---|
| VM 1 (VM yang sudah dipakai Hari 1-4) | App Server | `app-server` | `192.168.100.10` |
| VM 2 (VM baru) | File Server | `file-server` | `192.168.100.20` |

- Pastikan kedua VM bisa saling terhubung secara fisik (network adapter mode: **Host-only** atau **Internal Network**, bukan NAT saja)
- Firewall/SELinux boleh dinonaktifkan sementara untuk kelancaran lab (`sudo systemctl stop firewalld`), dengan catatan ke peserta bahwa ini **hanya untuk lab**, bukan praktik produksi sesungguhnya

---

# BAGIAN SOAL (Untuk Peserta)

## Latar Belakang Skenario

**Besok adalah hari go-live "Nusantara Apps"!** Sebagai Junior Linux Administrator, Anda ditugaskan Team Lead untuk menyelesaikan **checklist kesiapan akhir** sebelum aplikasi resmi dirilis ke publik:

1. Memastikan **prosedur disaster recovery** (backup & restore) benar-benar berfungsi
2. Memastikan **software pendukung** sudah terinstall dengan benar
3. Menghubungkan App Server dengan **File Server baru** yang akan menyimpan backup & menerima file deployment
4. Menyiapkan **FTP Server** di File Server sebagai jalur transfer file resmi antar server

Kerjakan berurutan — Bagian C & D membutuhkan **2 VM aktif bersamaan**.

---

### BAGIAN A — Disaster Recovery Drill: Backup & Restore
*(Bab 11: tar, cpio, gzip, restore parsial)*
**Dikerjakan di App Server**

1. Buat **backup penuh** seluruh folder proyek `~/nusantara-apps/` ke dalam satu file arsip `.tar` di home directory Anda.
2. Verifikasi isi arsip tersebut **tanpa** mengekstraknya.
3. Kompres arsip tersebut agar hemat storage saat dikirim/disimpan.
4. **Simulasikan bencana**: hapus folder `~/nusantara-apps/` sepenuhnya (pastikan sudah snapshot VM dulu!). Lakukan **restore penuh** dari arsip terkompresi tadi, buktikan seluruh isi proyek kembali utuh.
5. Sebagai redundansi (2 metode backup berbeda itu praktik yang baik), buat juga backup dengan format **cpio** dari folder yang sama.
6. Simulasikan skenario: seorang anggota tim tidak sengaja menghapus **1 file saja** (`requirement.txt`). Lakukan **restore parsial** — ambil hanya file tersebut dari salah satu arsip backup, **tanpa** merestore seluruh folder.

---

### BAGIAN B — Instalasi Software Pendukung
*(Bab 12: rpm, yum)*
**Dikerjakan di App Server (dan/atau File Server jika diminta)**

7. Sebelum go-live, tim security minta dilakukan port-scanning terhadap server. Cek apakah `nmap` sudah terinstall; jika belum, install menggunakan `yum`.
8. File Server nanti akan menjalankan FTP Server. Install `vsftpd`, lalu tampilkan info detail paket tersebut (versi, ukuran, deskripsi).
9. Tampilkan daftar **file konfigurasi** yang dibawa oleh paket `vsftpd` (bukan seluruh file, hanya yang berkategori konfigurasi).

---

### BAGIAN C — Konfigurasi Jaringan Antar Server
*(Bab 13: ifconfig, /etc/hosts, ping, netstat, nmap)*
**Dikerjakan di KEDUA VM**

10. Set IP address statis di **kedua VM** sesuai tabel skema IP di atas melalui file konfigurasi interface (bukan `ifconfig` manual yang hilang saat reboot).
11. Uji konektivitas dua arah: `ping` dari App Server ke File Server, dan sebaliknya.
12. Edit `/etc/hosts` di **kedua VM** agar masing-masing bisa saling memanggil menggunakan **nama host** (`app-server` / `file-server`), bukan IP. Uji ulang dengan `ping` memakai nama host.
13. Sebagai bagian dari security check pra go-live, dari App Server lakukan **port scanning** ke File Server menggunakan `nmap` untuk melihat port apa saja yang terbuka saat ini (sebelum FTP diaktifkan di Bagian D).

---

### BAGIAN D — FTP Server untuk Transfer File
*(Bab 14: vsftpd, konfigurasi, FTP client)*
**Server di File Server, Client di App Server**

14. Di **File Server**: pastikan `vsftpd` aktif dan otomatis jalan saat boot. Konfigurasikan `local_enable=YES` dan `write_enable=YES` di file konfigurasinya.
15. Di **File Server**: buat 1 user khusus untuk keperluan transfer file, misal `backup-user`, beserta passwordnya.
16. Ulangi `nmap` dari App Server ke File Server (seperti soal 13) — bandingkan hasilnya, **port berapa yang sekarang muncul** yang sebelumnya tidak ada?
17. Dari **App Server** (sebagai client FTP): koneksi ke File Server, login sebagai `backup-user`, lalu **upload** file backup terkompresi (hasil Bagian A) ke File Server menggunakan mode transfer yang sesuai untuk file biner/terkompresi.
18. Sebagai verifikasi, dari sesi FTP yang sama, **download kembali** file yang baru saja di-upload ke folder `/tmp` di App Server, lalu keluar dari sesi FTP dan pastikan file tersebut benar ada di `/tmp`.

---

### BAGIAN E — Laporan Akhir: Go-Live Checklist

19. Buat laporan akhir `go-live-checklist.txt` di App Server yang merangkum status dari **seluruh checklist**: hasil backup/restore, software terinstall, status jaringan, dan status FTP Server — sebagai bukti sistem **siap untuk go-live**.

---

## Checklist Deliverable

- [ ] Full backup `.tar.gz` berhasil dibuat & di-restore penuh setelah simulasi bencana
- [ ] Backup kedua (format `cpio`) berhasil dibuat
- [ ] Restore parsial (1 file saja) berhasil tanpa restore seluruh folder
- [ ] `nmap` & `vsftpd` terinstall dengan benar
- [ ] IP statis kedua VM aktif & bertahan setelah reboot (via file konfigurasi, bukan manual)
- [ ] `ping` dua arah berhasil (via IP maupun nama host setelah `/etc/hosts` diedit)
- [ ] `nmap` menunjukkan perbedaan port sebelum & sesudah FTP aktif
- [ ] Upload & download file via FTP client berhasil
- [ ] `go-live-checklist.txt` lengkap dan menyatakan status siap go-live

---
---

# KUNCI JAWABAN (Untuk Instruktur)

## Bagian A — Disaster Recovery Drill (di App Server)

**1. Full backup ke tar**
```bash
cd ~
tar -cvf nusantara-apps-backup.tar nusantara-apps/
```

**2. Verifikasi isi tanpa ekstrak**
```bash
tar -tf nusantara-apps-backup.tar
```

**3. Kompres arsip**
```bash
gzip nusantara-apps-backup.tar
ls -l nusantara-apps-backup.tar.gz
file nusantara-apps-backup.tar.gz
```

**4. Simulasi bencana + restore penuh**
```bash
# --- WAJIB SNAPSHOT VM DULU ---
rm -rf ~/nusantara-apps
ls ~                              # folder sudah hilang, buktikan

# RESTORE:
cd ~
zcat nusantara-apps-backup.tar.gz | tar xvf -
# ATAU langsung dalam 1 perintah:
# tar -xvfz nusantara-apps-backup.tar.gz

ls ~/nusantara-apps                # verifikasi seluruh struktur kembali utuh
```

**5. Backup redundan format cpio**
```bash
cd ~/nusantara-apps
find . | cpio -ov > ~/nusantara-apps-backup.cpio
cpio -it < ~/nusantara-apps-backup.cpio    # verifikasi isi
```

**6. Restore parsial (1 file saja)**
```bash
mkdir ~/restore-test
cd ~/restore-test

# Simulasi file terhapus:
rm ~/nusantara-apps/dokumen/requirement.txt

# Restore HANYA file itu dari arsip tar:
cd ~
tar -xvf nusantara-apps-backup.tar nusantara-apps/dokumen/requirement.txt

cat ~/nusantara-apps/dokumen/requirement.txt   # verifikasi file kembali
```
> Catatan: karena `nusantara-apps-backup.tar` sudah di-`gzip` di langkah 3, untuk demo poin ini gunakan arsip `.tar` biasa (buat ulang tar tanpa kompresi khusus untuk skenario ini), **atau** ekstrak dulu dengan `zcat file.tar.gz | tar xvf - nusantara-apps/dokumen/requirement.txt` (path spesifik tetap bisa diberikan meski sumbernya `.tar.gz`).

**Poin pembahasan instruktur:** Ini adalah skill paling praktis di dunia nyata — restore **1 file spesifik** jauh lebih cepat daripada restore seluruh backup saat user hanya kehilangan 1 file.

---

## Bagian B — Instalasi Software Pendukung

**7. Cek & install nmap**
```bash
rpm -q nmap
# jika muncul: "package nmap is not installed"
sudo yum install nmap
rpm -q nmap                # verifikasi terinstall
```

**8. Install vsftpd + info detail**
```bash
sudo yum install vsftpd
rpm -qi vsftpd
```
Output mencakup: Name, Version, Release, Size, Summary, Description.

**9. File konfigurasi paket vsftpd**
```bash
rpm -qc vsftpd
```
Output contoh:
```
/etc/logrotate.d/vsftpd
/etc/pam.d/vsftpd
/etc/vsftpd/ftpusers
/etc/vsftpd/user_list
/etc/vsftpd/vsftpd.conf
```

---

## Bagian C — Konfigurasi Jaringan Antar Server

**10. Set IP statis (dilakukan di KEDUA VM)**

Di **App Server**:
```bash
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
```
DEVICE=eth0
BOOTPROTO=none
IPADDR=192.168.100.10
NETMASK=255.255.255.0
ONBOOT=yes
```
```bash
sudo systemctl restart network
# atau (versi lama): sudo ifdown eth0 && sudo ifup eth0
ifconfig eth0
```

Di **File Server** (analog, IPADDR=192.168.100.20):
```bash
sudo vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
```
DEVICE=eth0
BOOTPROTO=none
IPADDR=192.168.100.20
NETMASK=255.255.255.0
ONBOOT=yes
```
```bash
sudo systemctl restart network
ifconfig eth0
```

**11. Ping dua arah**
```bash
# dari App Server:
ping -c 3 192.168.100.20

# dari File Server:
ping -c 3 192.168.100.10
```

**12. Edit /etc/hosts di kedua VM**

App Server (`/etc/hosts`):
```
192.168.100.10   app-server
192.168.100.20   file-server
```
File Server (`/etc/hosts`):
```
192.168.100.10   app-server
192.168.100.20   file-server
```
Uji:
```bash
# dari app-server:
ping -c 2 file-server
# dari file-server:
ping -c 2 app-server
```

**13. Port scanning sebelum FTP aktif**
```bash
# dari App Server:
nmap file-server
```
Output contoh (sebelum vsftpd start):
```
Not shown: 998 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
111/tcp  open  rpcbind
```

---

## Bagian D — FTP Server untuk Transfer File

**14. Aktifkan vsftpd di File Server**
```bash
# (di File Server)
sudo vi /etc/vsftpd/vsftpd.conf
```
Pastikan baris berikut:
```
local_enable=YES
write_enable=YES
```
```bash
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo systemctl status vsftpd
```

**15. Buat user FTP**
```bash
# (di File Server)
sudo useradd backup-user
sudo passwd backup-user
```

**16. nmap ulang, bandingkan hasil**
```bash
# dari App Server:
nmap file-server
```
Output contoh (**setelah** vsftpd aktif):
```
PORT     STATE SERVICE
21/tcp   open  ftp        <- MUNCUL BARU
22/tcp   open  ssh
111/tcp  open  rpcbind
```
**Poin pembahasan:** Ini mendemonstrasikan langsung kenapa `nmap` penting untuk **audit keamanan** — setiap service yang di-`start` akan langsung "terlihat" dari luar lewat port yang terbuka.

**17. Upload file via FTP client (dari App Server)**
```bash
ftp file-server
```
```
Name (file-server:peserta1): backup-user
Password: ********
ftp> binary
ftp> put nusantara-apps-backup.tar.gz
```

**18. Download & verifikasi**
```bash
ftp> lcd /tmp
ftp> get nusantara-apps-backup.tar.gz
ftp> quit
```
```bash
ls -l /tmp/nusantara-apps-backup.tar.gz
```
**Poin pembahasan:** tekankan kembali penggunaan mode `binary` untuk file terkompresi (`.tar.gz`) — jika salah menggunakan mode `ascii`, file akan **korup** saat ditransfer.

---

## Bagian E — Laporan Akhir: Go-Live Checklist

**19. Laporan lengkap**
```bash
cd ~/nusantara-apps
{
  echo "======================================================"
  echo "  GO-LIVE CHECKLIST - NUSANTARA APPS"
  echo "  Tanggal: `date`"
  echo "======================================================"
  echo ""
  echo "[1] BACKUP & DISASTER RECOVERY"
  ls -la ~/nusantara-apps-backup.tar.gz 2>/dev/null && echo "  -> Full backup (tar.gz): OK" || echo "  -> Full backup: TIDAK DITEMUKAN"
  ls -la ~/nusantara-apps-backup.cpio 2>/dev/null && echo "  -> Backup redundan (cpio): OK" || echo "  -> Backup cpio: TIDAK DITEMUKAN"
  echo ""
  echo "[2] SOFTWARE PENDUKUNG"
  rpm -q nmap vsftpd
  echo ""
  echo "[3] STATUS JARINGAN"
  ifconfig eth0 | grep "inet "
  ping -c 1 file-server > /dev/null 2>&1 && echo "  -> Koneksi ke file-server: OK" || echo "  -> Koneksi ke file-server: GAGAL"
  echo ""
  echo "[4] STATUS FTP SERVER"
  nmap file-server | grep ftp
  echo ""
  echo "======================================================"
  echo "  STATUS AKHIR: SIAP GO-LIVE"
  echo "======================================================"
} > go-live-checklist.txt

cat go-live-checklist.txt
```

---

## RUBRIK PENILAIAN (untuk Instruktur)

| Bagian | Aspek Dinilai | Poin |
|---|---|---|
| A | Backup penuh, kompresi, simulasi bencana + restore penuh berhasil, backup redundan cpio, restore parsial berhasil | 30 |
| B | `nmap` & `vsftpd` terinstall, `rpm -qi`/`rpm -qc` benar | 15 |
| C | IP statis persisten di 2 VM, ping 2 arah berhasil (IP & hostname), `nmap` sebelum FTP aktif | 20 |
| D | vsftpd aktif & auto-start, user FTP dibuat, `nmap` menunjukkan port 21 muncul, upload/download berhasil dgn mode binary | 25 |
| E | Laporan akhir lengkap, akurat, dan mencerminkan status nyata sistem | 10 |
| **Total** | | **100** |

**Kriteria Lulus:** ≥ 70 poin dianggap kompeten pada Bab 11, 12, 13, & 14 — sekaligus menjadi nilai **capstone akhir pelatihan**.

**Kesalahan umum yang sering ditemukan (checklist review cepat instruktur):**
- Restore parsial gagal karena peserta menuliskan path yang salah (tidak sama persis dengan struktur path yang tersimpan di dalam arsip — cek dulu dengan `tar -tf` sebelum ekstrak spesifik)
- Set IP dengan `ifconfig` langsung (bukan lewat file `ifcfg-eth0`) — hilang setelah reboot, sehingga hasil tidak persisten
- Lupa bahwa kedua VM harus satu network adapter mode yang sama (Host-only/Internal) — kalau salah satu masih NAT-only, `ping` tidak akan pernah berhasil
- Upload file `.tar.gz` dalam mode `ascii` (default FTP kadang ascii) sehingga file korup saat di-download kembali
- Lupa `firewalld`/SELinux memblokir port FTP meski `vsftpd` sudah `start` — perlu `sudo systemctl stop firewalld` (khusus environment lab)

---

## 🎓 REFLEKSI PENUTUP PROYEK 5 HARI

Studi kasus ini menutup perjalanan proyek **"Nusantara Apps"** yang dibangun bertahap selama 5 hari:

| Hari | Yang Dibangun | Bab |
|---|---|---|
| 1 | Struktur folder proyek, permission, link | Bab 1–2 |
| 2 | Toolkit otomasi shell script (menu, fungsi, backup otomatis) | Bab 3–4 |
| 3 | Server produksi: custom service, struktur tim & akses | Bab 5–7 |
| 4 | Solusi storage (LVM) & manajemen proses/performa | Bab 8–10 |
| 5 | Disaster recovery, software, jaringan antar server, FTP — **siap go-live** | Bab 11–14 |

Instruktur dapat menutup sesi dengan menekankan bahwa **seluruh skill ini saling terhubung** dalam pekerjaan nyata seorang Linux System Administrator — bukan topik yang berdiri sendiri-sendiri.

---
*Studi kasus ini adalah bagian dari rangkaian 5 hari proyek "Nusantara Apps". Gunakan bersama Studi Kasus Hari 1-4 untuk evaluasi praktikum harian secara utuh.*
