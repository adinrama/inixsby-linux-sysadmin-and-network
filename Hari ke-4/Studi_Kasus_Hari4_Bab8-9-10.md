# STUDI KASUS BERBASIS PROYEK — HARI 4
## Mengatasi Krisis Storage & Performa "Nusantara Apps"
### (Integrasi Bab 8: Manajemen Disk + Bab 9: LVM + Bab 10: Manajemen Proses)

---

## INFORMASI UMUM

| Item | Keterangan |
|---|---|
| Digunakan setelah | Bab 8, Bab 9, & Bab 10 selesai diajarkan (akhir Hari 4) |
| Durasi pengerjaan | 100–110 menit |
| Sifat | Individu, **wajib akses root/sudo** |
| Prasyarat | VM sudah ditambahkan **1 virtual disk kosong baru** (mis. `/dev/sdb`, minimal 20GB) — dilakukan lewat setting VirtualBox/VMware **sebelum** sesi dimulai, agar disk utama (`/dev/sda`) tidak terganggu |
| Output akhir | 1 file `laporan-storage-proses.txt` + beberapa script pendukung |

## ⚠️ CATATAN PERSIAPAN LAB

- Pastikan `/dev/sdb` (disk kosong tambahan) sudah terdeteksi sebelum mulai: `sudo fdisk -l`
- Seluruh partisi/LVM di studi kasus ini dibuat di `/dev/sdb`, **bukan** `/dev/sda` — untuk menghindari kerusakan sistem utama
- Disarankan snapshot VM sebelum memulai Bagian B (migrasi LVM), karena melibatkan banyak operasi berurutan

---

# BAGIAN SOAL (Untuk Peserta)

## Latar Belakang Skenario

**"Nusantara Apps" resmi live dan mulai dipakai banyak user.** Dua bulan berjalan, Anda sebagai Junior Linux Administrator mulai menerima beberapa laporan masalah dari tim:

> 📩 *"Server penuh, upload user gagal terus!"*
> 📩 *"Aplikasi kerasa lambat belakangan ini, kayak ada proses yang aneh."*
> 📩 *"Log makin numpuk, belum ada yang bersihin otomatis."*

Anda harus menyelesaikan masalah ini bertahap — dan karena ini server produksi yang sedang dipakai, **downtime harus seminimal mungkin**.

---

### BAGIAN A — Menambah Kapasitas Storage Awal
*(Bab 8: fdisk, mkfs, mount, fstab, fuser, fsck)*

Tim melaporkan folder data produksi (upload file user & log) sering penuh. Diputuskan menambahkan 1 partisi khusus untuk data produksi.

1. Deteksi disk baru yang baru ditambahkan ke VM menggunakan `dmesg` dan `fdisk -l`.
2. Buat partisi baru di disk tersebut sebesar **5GB**, tipe **83 (Linux)**, menggunakan `fdisk`. Jangan lupa `partprobe` setelahnya.
3. Format partisi tersebut dengan `mkfs.ext4`.
4. Buat folder `/data-produksi`, mount partisi baru ke folder tersebut.
5. Catat UUID partisi, daftarkan ke `/etc/fstab` agar **otomatis ter-mount setiap boot**, lalu uji dengan `mount -a` dan verifikasi setelah reboot.
6. Simulasikan situasi dimana **2 user sedang aktif** mengakses folder `/data-produksi` secara bersamaan (2 terminal berbeda, masing-masing `cd` ke folder tersebut). Dari terminal ketiga (root), gunakan perintah yang tepat untuk **mengetahui siapa saja** yang sedang menahan filesystem tersebut — informasi ini penting sebelum melakukan maintenance.
7. Lakukan pemeriksaan filesystem (`fsck`) pada partisi tersebut — ingat, filesystem harus dalam kondisi **unmounted** dulu sebelum bisa diperiksa.

---

### BAGIAN B — Migrasi ke LVM untuk Fleksibilitas Jangka Panjang
*(Bab 9: pvcreate, vgcreate, lvcreate, lvextend, resize2fs, vgextend)*

Dua bulan kemudian, `/data-produksi` (5GB) **penuh lagi**. Team Lead sadar partisi biasa merepotkan karena harus backup-format ulang setiap kali mau menambah kapasitas. Diputuskan pindah ke **LVM** agar kapasitas storage bisa ditambah kapan saja **tanpa reformat dan tanpa downtime besar**.

8. Siapkan **2 partisi baru** di disk yang sama, masing-masing **5GB**, dengan tipe **8e (Linux LVM)**.
9. Jadikan **kedua partisi** tersebut sebagai Physical Volume.
10. Buat Volume Group bernama **`nusantara-vg`** menggunakan **PV pertama saja** — PV kedua disimpan dulu sebagai cadangan ekspansi di masa depan.
11. Cek total PE yang tersedia di VG tersebut, lalu buat Logical Volume bernama **`data-lv`** menggunakan **sekitar 80%** dari total PE yang tersedia (sisakan sedikit untuk buffer).
12. Format LV tersebut, mount ke `/data-lvm`, daftarkan ke `/etc/fstab` seperti Bagian A.
13. Satu bulan kemudian, `/data-lvm` penuh lagi. **Tanpa reformat**, tambahkan kapasitas dengan **meng-extend LV** menggunakan sisa PE yang masih tersedia di VG, lalu terapkan perubahan agar filesystem juga ikut membesar.
14. Ternyata PE di VG sudah benar-benar habis. Tambahkan **PV kedua** ke VG yang sama, lalu extend LV lagi menggunakan PE baru tersebut. Verifikasi kapasitas akhir dengan `df -h`.

---

### BAGIAN C — Monitoring & Manajemen Proses
*(Bab 10: ps, nice/renice, background/foreground, sinyal, top, cron)*

Server mulai terasa lambat. Ada laporan proses tertentu memakan resource berlebihan, dan tim ingin ada maintenance otomatis rutin agar log tidak menumpuk lagi.

15. Jalankan simulasi "proses berat" menggunakan script `loop.sh` (dari folder Script/ materi Bab 4) **di background**, lalu temukan PID-nya menggunakan `ps`.
16. Turunkan prioritas (nice value) proses tersebut agar tidak mengganggu proses lain yang lebih penting. Buktikan perubahan prioritasnya.
17. Jalankan 1 proses simulasi lain (misal pencarian file besar) di background secara bersamaan. Gunakan `jobs` untuk melihat daftar proses background yang berjalan, lalu bawa salah satu proses ke foreground.
18. Buat **2 versi script loop**: satu bisa dihentikan dengan `Ctrl+C`, satu lagi **"kebal"** terhadap `Ctrl+C` (menggunakan `trap`). Jalankan versi yang kebal, buktikan `Ctrl+C` tidak berhasil menghentikannya, lalu hentikan paksa menggunakan sinyal yang **tidak bisa ditangkap/diabaikan oleh proses manapun**.
19. Gunakan `top` untuk memantau seluruh proses secara real-time selama beberapa detik. Catat proses dengan pemakaian %CPU tertinggi saat itu.
20. Buat script maintenance otomatis yang setiap **tengah malam** akan mem-backup file log (dengan penamaan tanggal) lalu mengosongkan file log aslinya. Jadwalkan script ini agar berjalan otomatis menggunakan `crontab`.

---

### BAGIAN D — Laporan Akhir

21. Buat laporan `laporan-storage-proses.txt` yang merangkum: status storage (`df -h`), status LVM (`lvs`, `vgs`), daftar jadwal cron aktif (`crontab -l`), dan proses background yang masih berjalan (`jobs`).

---

## Checklist Deliverable

- [ ] Partisi `/data-produksi` (5GB) berhasil dibuat, mount, dan auto-mount via fstab
- [ ] `fuser` berhasil mendeteksi 2 user yang sedang aktif di folder tersebut
- [ ] `fsck` berhasil dijalankan tanpa error
- [ ] LVM `nusantara-vg` + `data-lv` berhasil dibuat dan di-mount
- [ ] LV berhasil di-extend 2 kali (dari PE sisa & dari PV baru) tanpa reformat
- [ ] Proses `loop.sh` berhasil diturunkan prioritasnya dengan `renice`
- [ ] `jobs` & `fg` berhasil mengontrol proses background
- [ ] Proses dengan `trap` terbukti kebal `Ctrl+C`, berhasil dihentikan dengan `kill -9`
- [ ] Script maintenance log terjadwal otomatis via `crontab`
- [ ] Laporan akhir lengkap

---
---

# KUNCI JAWABAN (Untuk Instruktur)

## Bagian A — Menambah Kapasitas Storage Awal

**1. Deteksi disk baru**
```bash
sudo dmesg | grep sd
sudo fdisk -l
```
Pastikan `/dev/sdb` muncul di daftar (belum ada partisi).

**2. Buat partisi 5GB tipe 83**
```bash
sudo fdisk /dev/sdb
```
```
Command (m for help): n
Partition type: p
Partition number (1-4, default 1): [Enter]
First sector: [Enter]
Last sector, +sectors or +size{K,M,G}: +5G
Command (m for help): w
```
```bash
sudo partprobe
sudo fdisk -l /dev/sdb    # verifikasi /dev/sdb1 sudah muncul
```

**3. Format partisi**
```bash
sudo mkfs.ext4 /dev/sdb1
```

**4. Mount ke /data-produksi**
```bash
sudo mkdir /data-produksi
sudo mount /dev/sdb1 /data-produksi
df -h /data-produksi
```

**5. Auto-mount via fstab**
```bash
sudo blkid /dev/sdb1
# contoh output: /dev/sdb1: UUID="a1b2c3d4-..." TYPE="ext4"

sudo vi /etc/fstab
```
Tambahkan baris:
```
UUID=a1b2c3d4-....   /data-produksi   ext4   defaults   0 0
```
```bash
sudo mount -a                    # test tanpa reboot, harus tanpa error
sudo reboot
mount | grep data-produksi       # verifikasi setelah reboot
```

**6. Deteksi user aktif dengan fuser**
Terminal 1:
```bash
su - budi
cd /data-produksi
```
Terminal 2:
```bash
su - tono
cd /data-produksi
```
Terminal 3 (root):
```bash
fuser -mu /data-produksi
```
Output contoh:
```
/data-produksi:  2451c(budi)  2465c(tono)
```
**Poin pembahasan:** opsi `-m` untuk filesystem (mounted), `-u` untuk menampilkan nama user pemilik proses. Ini krusial sebelum `umount` — kalau langsung `umount` tanpa cek, akan muncul error `device is busy`.

**7. fsck**
```bash
sudo umount /data-produksi
sudo fsck /dev/sdb1
sudo mount /dev/sdb1 /data-produksi
```
> Jika dicoba `fsck` saat masih ter-mount, akan muncul pesan: `/dev/sdb1 is mounted. e2fsck: Cannot continue, aborting.` — ini bagus untuk didemokan sebagai bukti aturan "harus unmount dulu".

---

## Bagian B — Migrasi ke LVM

**8. Buat 2 partisi baru tipe 8e**
```bash
sudo fdisk /dev/sdb
```
```
n -> p -> 2 -> [Enter] -> +5G
n -> p -> 3 -> [Enter] -> +5G
t -> 2 -> 8e
t -> 3 -> 8e
p    (verifikasi)
w
```
```bash
sudo partprobe
```

**9. pvcreate**
```bash
sudo pvcreate /dev/sdb2
sudo pvcreate /dev/sdb3
sudo pvs
```

**10. vgcreate dari PV pertama**
```bash
sudo vgcreate nusantara-vg /dev/sdb2
sudo vgdisplay nusantara-vg
```

**11. Cek total PE, buat LV 80%**
```bash
sudo vgdisplay nusantara-vg | grep "Total PE"
# contoh: Total PE   1279
```
Hitung 80% ≈ 1023 PE:
```bash
sudo lvcreate -l 1023 -n data-lv nusantara-vg
sudo lvdisplay /dev/nusantara-vg/data-lv
```

**12. Format & mount**
```bash
sudo mkfs.ext4 -j /dev/nusantara-vg/data-lv
sudo mkdir /data-lvm
sudo mount /dev/nusantara-vg/data-lv /data-lvm

sudo blkid /dev/nusantara-vg/data-lv
sudo vi /etc/fstab
# UUID=<uuid>   /data-lvm   ext4   defaults   0 0
sudo mount -a
```

**13. Extend LV dari sisa PE**
```bash
sudo pvdisplay /dev/sdb2       # cek "Free PE" yang tersisa
sudo lvextend -l +256 /dev/nusantara-vg/data-lv    # sesuaikan angka dgn sisa Free PE
sudo resize2fs /dev/nusantara-vg/data-lv
df -h /data-lvm
```
**Poin pembahasan:** `lvextend` memperbesar Logical Volume, tapi filesystem-nya **belum otomatis membesar** sampai `resize2fs` dijalankan — ini poin yang sering terlewat peserta.

**14. Tambah PV kedua & extend lagi**
```bash
sudo vgextend nusantara-vg /dev/sdb3
sudo vgs                        # verifikasi VSize bertambah, 2 PV sekarang
sudo pvdisplay /dev/sdb3        # cek Total PE dari PV baru
sudo lvextend -l +1279 /dev/nusantara-vg/data-lv
sudo resize2fs /dev/nusantara-vg/data-lv
df -h /data-lvm
```
**Poin pembahasan kunci untuk seluruh Bagian B:** Semua proses ini dilakukan **tanpa unmount, tanpa reformat, dan tanpa downtime** — inilah keunggulan LVM dibanding partisi biasa (Bagian A), yang seharusnya menjadi kesimpulan utama yang ditangkap peserta.

---

## Bagian C — Monitoring & Manajemen Proses

**15. Jalankan proses berat & cari PID**
```bash
cd ~/nusantara-apps/scripts
cp ~/Script/loop.sh .
chmod +x loop.sh
./loop.sh &
```
```bash
ps -ef | grep loop.sh
```
Output contoh:
```
peserta1  5123  4980  0 10:15 pts/0  00:00:00 /bin/bash ./loop.sh
```

**16. Turunkan prioritas dengan renice**
```bash
ps -l                       # cek NI (nice) awal, biasanya 0
renice -n 10 -p 5123
ps -l                       # verifikasi NI sudah jadi 10
```
**Poin pembahasan:** nice value lebih tinggi = prioritas lebih rendah. Cocok untuk proses simulasi berat yang tidak kritikal, agar CPU lebih diprioritaskan ke proses aplikasi utama.

**17. jobs & fg**
```bash
find / -name "*.log" 2>/dev/null > /tmp/hasil-cari.log &
jobs
```
Output:
```
[1]+  Running    ./loop.sh &
[2]+  Running    find / -name "*.log" ... &
```
```bash
fg %1        # bawa job 1 (loop.sh) ke foreground
# tekan Ctrl+Z untuk suspend, atau Ctrl+C untuk interrupt (karena loop.sh tanpa trap)
```

**18. Script "kebal" trap + kill -9**

`loop-normal.sh` (bisa di-interrupt):
```bash
#!/bin/bash
while :
do
    echo "proses normal berjalan..."
    sleep 5
done
```

`loop-kebal.sh` (menangkap sinyal, mirip `Script/loop2.sh`):
```bash
#!/bin/bash
trap "" 1 2 3 9
while :
do
    echo "proses kebal berjalan..."
    sleep 5
done
```
Uji:
```bash
chmod +x loop-kebal.sh
./loop-kebal.sh
# tekan Ctrl+C -> TIDAK berhenti (SIGINT/2 berhasil ditangkap trap)
```
Di terminal lain:
```bash
ps -ef | grep loop-kebal
kill -9 <PID>
```
**⚠️ Poin pembahasan PENTING (sering jadi miskonsepsi):** Meskipun script menuliskan `trap "" 1 2 3 9` (menyertakan sinyal 9), pada kenyataannya **SIGKILL (sinyal 9) tidak pernah bisa ditangkap, diblokir, atau diabaikan oleh proses manapun** — ini adalah desain kernel Unix/Linux yang disengaja, supaya administrator **selalu punya cara pasti** untuk menghentikan proses apapun. Baris `9` di dalam `trap` pada praktiknya **tidak berpengaruh sama sekali**. Instruktur bisa jadikan ini poin diskusi menarik: *"Kalau semua sinyal bisa diblokir termasuk SIGKILL, bagaimana cara admin memaksa proses berhenti?"*

**19. top**
```bash
top
```
Amati kolom `%CPU`, `%MEM`, `PID`, `COMMAND`. Tekan `q` untuk keluar.

**20. Maintenance log otomatis via cron**
```bash
touch /data-produksi/app.log
echo "log dummy untuk simulasi" >> /data-produksi/app.log

vi ~/nusantara-apps/scripts/kosongkan_log.sh
```
```bash
#!/bin/bash
# kosongkan_log.sh - maintenance otomatis
tgl=`date +%Y%m%d`
nama1=/data-produksi/app.log
nama2=/data-produksi/app.log.$tgl

cp $nama1 $nama2
> $nama1
echo "Log dibackup ke $nama2 dan dikosongkan pada `date`"
```
```bash
chmod +x ~/nusantara-apps/scripts/kosongkan_log.sh
crontab -e
```
Tambahkan baris:
```
0 0 * * * /home/peserta1/nusantara-apps/scripts/kosongkan_log.sh >> /home/peserta1/nusantara-apps/scripts/cron.log 2>&1
```
```bash
crontab -l    # verifikasi
```
**Poin pembahasan:** gunakan **path absolut** (bukan `~` atau path relatif) di dalam `crontab`, karena cron dijalankan tanpa environment shell interaktif sehingga `~` tidak selalu ter-resolve dengan benar.

---

## Bagian D — Laporan Akhir

**21. Laporan akhir**
```bash
cd ~/nusantara-apps
{
  echo "=== LAPORAN STORAGE & PROSES - NUSANTARA APPS ==="
  echo ""
  echo "--- Status Storage ---"
  df -h
  echo ""
  echo "--- Status LVM ---"
  sudo lvs
  sudo vgs
  echo ""
  echo "--- Jadwal Cron Aktif ---"
  crontab -l
  echo ""
  echo "--- Proses Background Masih Berjalan ---"
  jobs
} > laporan-storage-proses.txt

cat laporan-storage-proses.txt
```

---

## RUBRIK PENILAIAN (untuk Instruktur)

| Bagian | Aspek Dinilai | Poin |
|---|---|---|
| A | Partisi biasa dibuat, mount, fstab, `fuser`, `fsck` berhasil | 20 |
| B | Rantai LVM lengkap (PV→VG→LV) + 2x extend tanpa reformat berhasil | 30 |
| C | `renice`, `jobs/fg`, `trap`+`kill -9`, `top`, cron maintenance berhasil semua | 40 |
| D | Laporan akhir lengkap & akurat | 10 |
| **Total** | | **100** |

**Kriteria Lulus:** ≥ 70 poin dianggap kompeten pada Bab 8, 9, & 10.

**Kesalahan umum yang sering ditemukan (checklist review cepat instruktur):**
- Lupa `partprobe` setelah `fdisk`, sehingga kernel belum mengenali partisi baru
- Menulis UUID di `/etc/fstab` dengan salah ketik (typo 1 karakter saja bikin boot gagal masuk emergency mode) — ingatkan copy-paste dari `blkid`, jangan ketik manual
- Lupa `resize2fs` setelah `lvextend` — mengira kapasitas otomatis bertambah padahal belum
- Salah paham bahwa `trap` bisa memblokir SIGKILL — ini miskonsepsi paling penting untuk diluruskan
- Menjalankan `crontab -e` lalu keluar tanpa save dengan benar, atau menuliskan path relatif/`~` di dalam crontab

---
*Studi kasus ini melanjutkan narasi "Nusantara Apps" — dari fase infrastruktur dasar (Hari 3) ke tantangan operasional nyata saat aplikasi mulai banyak dipakai: storage, skalabilitas, dan performa.*
