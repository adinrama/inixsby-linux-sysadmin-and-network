# STUDI KASUS BERBASIS PROYEK — HARI 3
## Setup Server Produksi "Nusantara Apps"
### (Integrasi Bab 5: Konfigurasi Pasca Instalasi + Bab 6: Proses Startup + Bab 7: Manajemen Pemakai)

---

## INFORMASI UMUM

| Item | Keterangan |
|---|---|
| Digunakan setelah | Bab 5, Bab 6, & Bab 7 selesai diajarkan (akhir Hari 3) |
| Durasi pengerjaan | 90–100 menit |
| Sifat | Individu, **wajib akses root/sudo** |
| Prasyarat | VM Linux dengan akses console langsung (bukan hanya SSH, karena ada tahap edit GRUB) |
| Output akhir | 1 file laporan `setup-server.txt` |

## ⚠️ CATATAN KEAMANAN LAB (WAJIB DIBACA SEBELUM MULAI)

- Studi kasus ini melibatkan **reboot**, **edit GRUB**, dan **hapus user** — semuanya berpotensi merusak sistem jika salah langkah.
- **Wajib buat SNAPSHOT VM** sebelum memulai Bagian A poin 4 (reset password root) dan sebelum Bagian B (uji reboot service).
- Jika lab dilakukan di server bersama/production sungguhan, **JANGAN** lakukan bagian reset password root — cukup dijelaskan secara konseptual oleh instruktur (demo di layar, peserta tidak praktik langsung).
- Instruktur disarankan menyediakan **VM khusus per peserta** yang boleh "dirusak" dengan aman.

---

# BAGIAN SOAL (Untuk Peserta)

## Latar Belakang Skenario

Proyek **"Nusantara Apps"** akan segera masuk tahap deployment. Perusahaan baru saja menyediakan **1 unit server produksi baru** yang masih kosong (fresh install). Sebagai Junior Linux Administrator, Anda ditugaskan oleh Team Lead untuk:

1. Memverifikasi & mengkonfigurasi dasar server tersebut
2. Membuat service khusus yang mewakili aplikasi "Nusantara Apps" agar otomatis berjalan setiap kali server menyala
3. Menyiapkan struktur user & akses tim yang akan bekerja di server ini — karena tim sudah bertambah besar

Kerjakan berurutan, karena bagian selanjutnya bergantung pada hasil bagian sebelumnya (khususnya Bagian C butuh service dari Bagian B).

---

### BAGIAN A — Verifikasi & Konfigurasi Dasar Server
*(Bab 5: dmesg, fdisk, hwclock, GRUB, rescue mode)*

1. Server ini baru pertama kali dinyalakan. Periksa hardware disk yang terpasang menggunakan `dmesg` dan `fdisk -l`.
2. Cek waktu sistem saat ini. Ternyata jamnya tidak sesuai zona waktu Indonesia (WIB). Sesuaikan menggunakan `hwclock`, lalu verifikasi kembali.
3. Sebelum melakukan konfigurasi apapun ke bootloader, buat **backup file konfigurasi GRUB** terlebih dahulu (praktik keamanan standar).
4. **(Simulasi Insiden — wajib snapshot VM dulu!)** Team Lead yang sebelumnya setup server ini lupa memberi tahu Anda password root, dan ternyata dia sudah resign. Lakukan **prosedur reset password root** melalui GRUB2 rescue mode, lalu reboot dan buktikan Anda bisa login dengan password root yang baru.

---

### BAGIAN B — Custom Startup Script "Nusantara Apps"
*(Bab 6: init.d script, chkconfig/systemd, run level/target)*

5. Buat script layanan (service) di `/etc/init.d/nusantara-apps` yang mensimulasikan aplikasi berjalan, dengan opsi `start`, `stop`, `status`, dan `restart` (gunakan `case`). Script harus mencatat statusnya menggunakan file penanda di `/var/lock/subsys/`.
6. Beri izin eksekusi, lalu uji secara manual: `start`, `status`, `stop`, `status` lagi.
7. Daftarkan service ini agar **otomatis berjalan setiap boot** — gunakan `chkconfig` (jika sistem masih SysV/legacy) **atau** buat unit file `systemd` (jika sistem sudah modern). Pilih salah satu sesuai VM yang Anda pakai.
8. Reboot server, lalu **buktikan** service berjalan otomatis tanpa harus dijalankan manual (cek status service & log sistem).
9. Karena ini **server produksi tanpa GUI**, ubah default target/runlevel server menjadi mode **multi-user tanpa X-Window** (bukan graphical).

---

### BAGIAN C — Manajemen Pemakai & Struktur Tim
*(Bab 7: groupadd, useradd, primary/secondary group, sudo, usermod, userdel)*

Tim proyek sudah bertambah. Susunan tim yang perlu di-setup:

| Nama User | Peran | Primary Group | Secondary Group |
|---|---|---|---|
| budi | Developer | developer | - |
| sari | Developer | developer | - |
| tono | Tester | tester | developer *(agar bisa akses source code untuk testing)* |
| rina | Project Manager | management | - |

10. Buat 3 group sesuai tabel di atas: `developer`, `tester`, `management`.
11. Buat ke-4 user sesuai tabel, pastikan primary & secondary group-nya tepat seperti yang diminta.
12. Set password untuk seluruh user baru tersebut.
13. Verifikasi hasilnya: tampilkan baris terkait di `/etc/passwd` dan `/etc/group`, pastikan `tono` benar-benar tercatat sebagai anggota **secondary** di group `developer`.
14. Team Lead ingin tim **developer** bisa me-restart service `nusantara-apps` (dari Bagian B) **tanpa perlu memasukkan password root**, supaya mereka mandiri saat deploy. Konfigurasikan ini melalui `/etc/sudoers` (gunakan `visudo`), berlaku untuk **seluruh anggota group** `developer` sekaligus (bukan didaftarkan satu-satu).
15. Login sebagai `budi`, buktikan dia bisa me-restart service `nusantara-apps` melalui `sudo` **tanpa diminta password**.
16. Terjadi perubahan tim: `tono` mengajukan resign namun proses administrasi belum final (kunci dulu akunnya sementara), sedangkan `sari` sudah benar-benar pindah ke proyek lain dan datanya boleh dihapus permanen. Lakukan tindakan yang sesuai untuk masing-masing.

---

### BAGIAN D — Laporan Akhir

17. Buat laporan `setup-server.txt` yang merangkum: daftar group & user yang aktif saat ini, status service `nusantara-apps`, dan default target/runlevel server saat ini.

---

## Checklist Deliverable

- [ ] Waktu sistem sudah sesuai WIB
- [ ] Backup GRUB config tersimpan
- [ ] Password root berhasil direset via rescue mode (jika dipraktikkan)
- [ ] Service `nusantara-apps` start/stop/status manual berhasil
- [ ] Service terdaftar otomatis jalan saat boot (terbukti setelah reboot)
- [ ] Default target sudah multi-user (non-GUI)
- [ ] 3 group & 4 user dibuat sesuai tabel primary/secondary
- [ ] Sudo delegasi group `developer` berhasil (tanpa password)
- [ ] `tono` terkunci (locked), `sari` terhapus permanen beserta home directory
- [ ] `setup-server.txt` lengkap dan tersimpan

---
---

# KUNCI JAWABAN (Untuk Instruktur)

## Bagian A — Verifikasi & Konfigurasi Dasar Server

**1. Cek hardware disk**
```bash
dmesg | grep sda
fdisk -l /dev/sda
```

**2. Cek & sesuaikan waktu sistem**
```bash
date
hwclock --show
sudo hwclock --set --date "2026-07-30 09:00:00"
hwclock --show
```
> Alternatif GUI: `system-config-date` (jika environment desktop tersedia).

**3. Backup GRUB**
```bash
# GRUB2 (modern - CentOS 7+):
sudo cp /boot/grub2/grub.cfg /boot/grub2/grub.cfg.orig

# GRUB Legacy (jika VM masih pakai versi lama):
sudo cp /boot/grub/grub.conf /boot/grub/grub.conf.orig
```

**4. Reset password root via GRUB2 Rescue Mode**
Langkah (mengacu Catatan Bab 05 poin 6):
```
1. Restart VM: sudo reboot
2. Saat menu GRUB muncul, tekan tombol "e" untuk edit
3. Cari baris yang diawali "linux16" atau "linux", arahkan kursor ke akhir baris
4. Ubah kata "ro" menjadi: rw init=/sysroot/bin/sh
5. Tekan Ctrl+x untuk boot dengan parameter tersebut
6. Setelah masuk shell darurat, jalankan:
   chroot /sysroot
   passwd root
   (masukkan password baru 2x)
   touch /.autorelabel      <- wajib jika SELinux aktif
   exit
   reboot
7. Setelah reboot normal, login sebagai root dengan password baru
```
**Poin pembahasan instruktur:**
- Ini membuktikan bahwa **fisik akses ke server = kontrol penuh** — alasan kenapa keamanan ruang server/BIOS/GRUB password itu penting (bisa dikaitkan ke materi opsional "proteksi GRUB dengan `grub2-mkpasswd-pbkdf2`").
- `touch /.autorelabel` penting di sistem dengan SELinux aktif, agar context file `/etc/shadow` yang baru diubah tidak menyebabkan error login setelah reboot.

---

## Bagian B — Custom Startup Script

**5. Buat script `/etc/init.d/nusantara-apps`**
```bash
sudo vi /etc/init.d/nusantara-apps
```
Isi:
```bash
#!/bin/sh
#
# nusantara-apps
# Simulasi service aplikasi Nusantara Apps
# chkconfig: 35 95 05
# description: Menjalankan layanan Nusantara Apps

case "$1" in
    start)
        echo -n "Menjalankan Nusantara Apps service..."
        touch /var/lock/subsys/nusantara-apps
        echo " [OK]"
        ;;
    stop)
        echo -n "Menghentikan Nusantara Apps service..."
        rm -f /var/lock/subsys/nusantara-apps
        echo " [OK]"
        ;;
    status)
        if [ -f /var/lock/subsys/nusantara-apps ]
        then
            echo "Nusantara Apps service sedang AKTIF"
        else
            echo "Nusantara Apps service TIDAK aktif"
        fi
        ;;
    restart|reload)
        $0 stop
        $0 start
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|reload|status}"
        exit 1
esac
exit 0
```

**6. Uji manual**
```bash
sudo chmod 755 /etc/init.d/nusantara-apps
sudo /etc/init.d/nusantara-apps start
sudo /etc/init.d/nusantara-apps status
sudo /etc/init.d/nusantara-apps stop
sudo /etc/init.d/nusantara-apps status
```

**7. Daftarkan otomatis saat boot**

**[LEGACY] via chkconfig:**
```bash
sudo chkconfig --add nusantara-apps
sudo chkconfig --list nusantara-apps
ls /etc/rc3.d | grep nusantara-apps
ls /etc/rc5.d | grep nusantara-apps
```

**[MODERN] via systemd unit file (alternatif):**
```bash
sudo vi /etc/systemd/system/nusantara-apps.service
```
```ini
[Unit]
Description=Nusantara Apps Service (Simulasi)
After=network.target

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/bin/touch /var/lock/subsys/nusantara-apps
ExecStop=/bin/rm -f /var/lock/subsys/nusantara-apps

[Install]
WantedBy=multi-user.target
```
```bash
sudo systemctl daemon-reload
sudo systemctl enable nusantara-apps
sudo systemctl start nusantara-apps
sudo systemctl status nusantara-apps
```

**8. Uji reboot & verifikasi**
```bash
sudo shutdown -r now
# setelah login kembali:
sudo /etc/init.d/nusantara-apps status
# atau: systemctl status nusantara-apps

cat /var/log/messages | grep nusantara-apps
# atau (modern): journalctl -u nusantara-apps
```
**Poin pembahasan instruktur:** jika memilih jalur `chkconfig`, ingatkan peserta soal catatan penting di materi: bila service tidak jalan otomatis saat shutdown/rc0, biasanya karena file penanda di `/var/lock/subsys/` belum ada — solusinya `touch /var/lock/subsys/nusantara-apps` secara manual sekali (lihat Catatan Bab 06).

**9. Ubah default target ke multi-user (non-GUI)**
```bash
systemctl get-default
sudo systemctl set-default multi-user.target
systemctl get-default          # verifikasi hasilnya: multi-user.target
```
**[LEGACY setara]:** edit `/etc/inittab`, ubah baris `id:5:initdefault:` menjadi `id:3:initdefault:`

---

## Bagian C — Manajemen Pemakai & Struktur Tim

**10. Buat 3 group**
```bash
sudo groupadd developer
sudo groupadd tester
sudo groupadd management
```

**11. Buat 4 user sesuai tabel**
```bash
sudo useradd -g developer budi
sudo useradd -g developer sari
sudo useradd -g tester -G developer tono
sudo useradd -g management rina
```

**12. Set password**
```bash
sudo passwd budi
sudo passwd sari
sudo passwd tono
sudo passwd rina
```

**13. Verifikasi**
```bash
grep -E "budi|sari|tono|rina" /etc/passwd
grep developer /etc/group
grep tester /etc/group
grep management /etc/group
```
Output yang diharapkan (contoh):
```
budi:x:1001:1001::/home/budi:/bin/bash
sari:x:1002:1001::/home/sari:/bin/bash
tono:x:1003:1002::/home/tono:/bin/bash
rina:x:1004:1003::/home/rina:/bin/bash

developer:x:1001:tono      <- tono tercatat sbg SECONDARY member
tester:x:1002:
management:x:1003:
```
**Poin pembahasan instruktur:** budi & sari **tidak** tercatat sebagai anggota di `/etc/group` baris `developer`, karena itu adalah **primary group** mereka (dicatat cukup di `/etc/passwd`). Hanya `tono` yang muncul di situ karena itu **secondary group**-nya.

**14. Setup sudo untuk group developer (tanpa password)**
```bash
sudo visudo
```
Tambahkan baris (sesuai service yang dipakai di Bagian B):
```
## Izinkan seluruh anggota group developer restart service nusantara-apps tanpa password
%developer ALL=(root) NOPASSWD: /etc/init.d/nusantara-apps, /etc/init.d/nusantara-apps *
```
Atau jika pakai systemd:
```
%developer ALL=(root) NOPASSWD: /usr/bin/systemctl restart nusantara-apps, /usr/bin/systemctl start nusantara-apps, /usr/bin/systemctl stop nusantara-apps, /usr/bin/systemctl status nusantara-apps
```

**15. Uji delegasi sudo**
```bash
su - budi
sudo /etc/init.d/nusantara-apps restart
# atau: sudo systemctl restart nusantara-apps
```
Harus berhasil **tanpa** diminta password root. Coba juga dengan user `rina` (bukan anggota developer) — seharusnya **ditolak / diminta password** karena tidak termasuk dalam delegasi.

**16. Offboarding tim**
```bash
# tono: resign, tapi proses belum final -> kunci dulu
sudo usermod -L tono

# sari: sudah pasti pindah proyek lain -> hapus permanen + home directory
sudo userdel -r sari
grep sari /etc/passwd     # harus kosong, membuktikan sudah terhapus
```
**Poin pembahasan instruktur:** `usermod -L` mengunci password (menambahkan `!` di depan hash password di `/etc/shadow`) sehingga user **tidak bisa login**, tapi datanya masih utuh — cocok untuk status "belum final". `userdel -r` bersifat **permanen** dan menghapus home directory — hanya dipakai jika benar-benar yakin data tidak diperlukan lagi.

---

## Bagian D — Laporan Akhir

**17. Laporan setup server**
```bash
cd ~/nusantara-apps
{
  echo "=== LAPORAN SETUP SERVER - NUSANTARA APPS ==="
  echo ""
  echo "--- Group Aktif ---"
  grep -E "developer|tester|management" /etc/group
  echo ""
  echo "--- User Aktif ---"
  grep -E "budi|sari|tono|rina" /etc/passwd
  echo ""
  echo "--- Status Service nusantara-apps ---"
  sudo /etc/init.d/nusantara-apps status
  echo ""
  echo "--- Default Target/Runlevel ---"
  systemctl get-default
} > setup-server.txt

cat setup-server.txt
```

---

## RUBRIK PENILAIAN (untuk Instruktur)

| Bagian | Aspek Dinilai | Poin |
|---|---|---|
| A | Verifikasi hardware, waktu sistem benar, backup GRUB, reset password root berhasil | 25 |
| B | Custom script berjalan, terdaftar otomatis boot (terbukti setelah reboot), default target benar | 30 |
| C | Group & user sesuai tabel (primary/secondary tepat), sudo delegasi berhasil tanpa password, offboarding tepat (lock vs delete) | 35 |
| D | Laporan lengkap dan akurat | 10 |
| **Total** | | **100** |

**Kriteria Lulus:** ≥ 70 poin dianggap kompeten pada Bab 5, 6, & 7.

**Kesalahan umum yang sering ditemukan (checklist review cepat instruktur):**
- Lupa `chroot /sysroot` sebelum `passwd root` saat rescue mode (password akan tersimpan salah, di luar sistem yang benar)
- Lupa `chmod 755` pada script init.d sehingga muncul "Permission denied" saat dieksekusi
- Salah menaruh `-g` (primary) vs `-G` (secondary) saat `useradd`, sehingga hasil di `/etc/group` terbalik
- Menulis aturan sudo di `/etc/sudoers` tanpa `%` di depan nama group (tanpa `%`, sistem akan menganggapnya sebagai nama **user**, bukan **group**, sehingga tidak berlaku untuk semua anggota developer)
- Menjalankan `visudo` lalu keluar tanpa menyimpan (`:wq` di mode vi) sehingga perubahan tidak tersimpan

---
*Studi kasus ini melanjutkan narasi "Nusantara Apps" dari Hari 1 & 2 — kini proyek memasuki tahap infrastruktur produksi yang membutuhkan kolaborasi tim yang lebih besar dengan pengelolaan akses yang tepat.*
