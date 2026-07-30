# STUDI KASUS BERBASIS PROYEK — HARI 1
## Setup Workspace Proyek "Nusantara Apps"
### (Integrasi Bab 1: Sesi Terminal Linux + Bab 2: File & Direktori)

---

## INFORMASI UMUM

| Item | Keterangan |
|---|---|
| Digunakan setelah | Bab 1 & Bab 2 selesai diajarkan (akhir Hari 1) |
| Durasi pengerjaan | 45–60 menit |
| Sifat | Individu (masing-masing peserta di VM sendiri) |
| Akses | User biasa (non-root); beberapa poin bonus butuh `sudo`/root |
| Output akhir | 1 file laporan `laporan_setup.txt` di dalam folder proyek |

---

# BAGIAN SOAL (Untuk Peserta)

## Latar Belakang Skenario

Anda baru diterima sebagai **Junior Linux Administrator** di PT Digital Nusantara. Hari ini adalah hari pertama Anda bekerja. Tim proyek **"Nusantara Apps"** akan segera mulai bekerja, dan Anda ditugaskan oleh Team Lead untuk **menyiapkan workspace kerja** di server Linux sebelum tim developer & tester mulai menggunakannya besok.

Team Lead memberi Anda instruksi kerja berikut. Kerjakan **secara berurutan**, karena beberapa tahap saling berkaitan.

> Catatan: Manajemen user (`useradd`, dsb.) belum dibahas — akan dipelajari di Bab 7. Untuk saat ini, seluruh pekerjaan dilakukan di dalam **home directory Anda sendiri**.

---

### BAGIAN A — Orientasi & Investigasi Sistem
*(Bab 1: identitas diri, identitas mesin, user aktif, manual)*

Sebelum membangun apa pun, Team Lead ingin Anda memastikan Anda mengenal server yang akan dipakai.

1. Tampilkan identitas Anda di sistem ini (username, UID, GID, group).
2. Tampilkan nama server (hostname) dan detail lengkap versi kernel/OS yang berjalan.
3. Cek siapa saja yang sedang login ke server saat ini (gunakan 2 cara berbeda).
4. Anda ingin memasang jadwal kerja tim di terminal menggunakan kalender bulan berjalan, tapi lupa nama perintahnya. Gunakan `apropos`/`man -k` dengan kata kunci **"calendar"** untuk menemukannya, lalu jalankan perintah tersebut.
5. Pastikan Anda sedang berada di home directory Anda sendiri (buktikan dengan 2 cara).

---

### BAGIAN B — Membangun Struktur Direktori Proyek
*(Bab 2: mkdir, navigasi direktori)*

Buat struktur folder proyek berikut **di dalam home directory Anda**, dengan nama folder utama **`nusantara-apps`**:

```
nusantara-apps/
├── dokumen/
├── source-code/
├── testing/
├── shared/
└── backup/
```

6. Buat seluruh struktur di atas **dalam satu baris perintah** (gunakan opsi yang memungkinkan pembuatan banyak folder sekaligus).
7. Tampilkan struktur folder yang sudah dibuat untuk memverifikasi (gunakan `ls`).
8. Masuk ke folder `dokumen/`, lalu buat 2 file berikut (isi bebas, tapi **wajib mengandung** kata **"Nusantara Apps"** dan **"deadline"**):
   - `requirement.txt`
   - `meeting-notes.txt`

---

### BAGIAN C — Pengelolaan & Pencarian File
*(Bab 1: cp, mv, grep, file / Bab 2: navigasi)*

9. Buat salinan cadangan dari `requirement.txt` ke dalam folder `backup/` dengan nama `requirement-backup.txt`.
10. Team Lead menanyakan kapan deadline proyek. Cari baris yang mengandung kata **"deadline"** di dalam `requirement.txt` tanpa membuka isi filenya secara penuh.
11. Periksa tipe file dari `requirement.txt` untuk memastikan itu adalah file teks biasa.
12. Anda tidak sengaja salah menaruh `meeting-notes.txt` ke folder `source-code/`. Pindahkan file tersebut, lalu kembalikan lagi ke folder `dokumen/`.

---

### BAGIAN D — Hak Akses & Keamanan File
*(Bab 2: chmod, umask)*

13. `requirement.txt` bersifat **rahasia**. Atur permission agar **hanya owner** yang bisa membaca & menulis file tersebut — group dan other **tidak boleh punya akses sama sekali**.
14. Folder `source-code/` boleh **dimasuki dan dibaca isinya oleh group**, tetapi **group tidak boleh menulis** ke dalamnya. Other tidak boleh akses sama sekali. Gunakan notasi **oktal**.
15. Cek nilai `umask` default sistem Anda saat ini. Kemudian:
    - Ubah `umask` sementara menjadi **027**
    - Masuk ke folder `source-code/`, buat file baru bernama `draft.txt`
    - Tampilkan permission file tersebut dan **jelaskan** mengapa hasilnya seperti itu berdasarkan nilai umask
16. **(BONUS — opsional, butuh akses `sudo`)**: Jika tersedia user lain di sistem, ubah kepemilikan folder `testing/` ke user tersebut menggunakan `chown`.

---

### BAGIAN E — Symbolic Link & Hard Link untuk Sharing Resource
*(Bab 2: ln, ln -s, inode)*

Tim developer dan tester membutuhkan **1 file standar coding** yang sama, tanpa harus menyalinnya berkali-kali (supaya kalau ada update, cukup 1 kali edit).

17. Buat file `coding-standard.txt` di dalam folder `shared/` (isi bebas, misalnya aturan penamaan variable).
18. Buat **symbolic link** dari file tersebut ke dalam folder `source-code/` dan `testing/`, dengan nama file yang sama.
19. Buat **hard link** dari `requirement-backup.txt` (folder `backup/`) ke folder `dokumen/` dengan nama `requirement-final.txt`. Buktikan dengan perintah yang menampilkan nomor **inode** bahwa kedua file tersebut menunjuk ke data yang sama.
20. Hapus file `requirement-backup.txt` di folder `backup/`. Buktikan bahwa data **tidak hilang** karena masih ada di `requirement-final.txt`.

---

### BAGIAN F — Laporan Akhir

21. Buat laporan akhir bernama `laporan_setup.txt` **di dalam folder `nusantara-apps/`** yang berisi hasil `ls -laR` dari seluruh struktur proyek (bukti pekerjaan hari ini), lalu tampilkan isinya ke layar untuk verifikasi terakhir.

---

## Checklist Deliverable (dicentang peserta sebelum submit)

- [ ] Struktur folder `nusantara-apps` lengkap 5 subfolder
- [ ] `requirement.txt` permission `600`
- [ ] `source-code/` permission `750`
- [ ] `draft.txt` dibuat setelah umask diubah ke `027`
- [ ] Symbolic link `coding-standard.txt` ada di `source-code/` dan `testing/`
- [ ] Hard link `requirement-final.txt` terbukti se-inode dengan backup asal
- [ ] File asal backup dihapus, data tetap ada di hard link
- [ ] `laporan_setup.txt` berhasil dibuat berisi `ls -laR`

---
---

# KUNCI JAWABAN (Untuk Instruktur)

## Bagian A — Orientasi & Investigasi Sistem

**1. Identitas diri**
```bash
id
whoami
```
Contoh output:
```
uid=1000(peserta1) gid=1000(peserta1) groups=1000(peserta1),10(wheel)
```

**2. Identitas mesin**
```bash
hostname
uname -a
```

**3. User yang sedang aktif (2 cara)**
```bash
who
w
```

**4. apropos + kalender**
```bash
apropos calendar
# atau: man -k calendar
```
Hasil biasanya menunjukkan:
```
cal (1) - displays a calendar
```
Lalu jalankan:
```bash
cal
```
> Catatan instruktur: jika `apropos` tidak menampilkan hasil, database `mandb` mungkin belum dibangun — jalankan `sudo mandb` terlebih dahulu (khusus versi Linux modern).

**5. Verifikasi home directory (2 cara)**
```bash
pwd
echo $HOME
```

---

## Bagian B — Struktur Direktori Proyek

**6. Buat struktur sekaligus**
```bash
mkdir -p ~/nusantara-apps/{dokumen,source-code,testing,shared,backup}
```
> Poin penilaian: peserta harus menggunakan **brace expansion** `{}` atau minimal `mkdir -p` dengan multiple argumen — bukan 5 perintah `mkdir` terpisah (walau itu juga tetap benar secara fungsi, brace expansion menunjukkan pemahaman lebih efisien).

**7. Verifikasi struktur**
```bash
ls -l ~/nusantara-apps
```
Output:
```
drwxr-xr-x  dokumen
drwxr-xr-x  source-code
drwxr-xr-x  testing
drwxr-xr-x  shared
drwxr-xr-x  backup
```

**8. Buat 2 file dokumen**
```bash
cd ~/nusantara-apps/dokumen

cat > requirement.txt << EOF
Project: Nusantara Apps
Deadline: 30 Agustus 2026
Scope: Mobile & Web App
EOF

cat > meeting-notes.txt << EOF
Meeting Kickoff - Nusantara Apps
Deadline internal: 25 Agustus 2026
Catatan: seluruh tim wajib hadir
EOF
```

---

## Bagian C — Pengelolaan & Pencarian File

**9. Copy ke backup**
```bash
cd ~/nusantara-apps
cp dokumen/requirement.txt backup/requirement-backup.txt
```

**10. Cari kata "deadline"**
```bash
grep -i deadline dokumen/requirement.txt
```
Output:
```
Deadline: 30 Agustus 2026
```

**11. Cek tipe file**
```bash
file dokumen/requirement.txt
```
Output:
```
dokumen/requirement.txt: ASCII text
```

**12. Pindah lalu kembalikan**
```bash
mv dokumen/meeting-notes.txt source-code/
mv source-code/meeting-notes.txt dokumen/
```

---

## Bagian D — Hak Akses & Keamanan File

**13. Permission 600 untuk file rahasia**
```bash
chmod 600 dokumen/requirement.txt
ls -l dokumen/requirement.txt
```
Output:
```
-rw------- 1 peserta1 peserta1 ... requirement.txt
```

**14. Permission 750 untuk folder source-code**
```bash
chmod 750 source-code
ls -ld source-code
```
Output:
```
drwxr-x--- ... source-code
```
Penjelasan: `7`=owner rwx, `5`=group r-x (bisa masuk & baca, **tidak bisa tulis**), `0`=other tidak ada akses.

**15. Umask**
```bash
umask                     # cek default, umumnya 0022
umask 027                 # ubah sementara
cd source-code
touch draft.txt
ls -l draft.txt
```
Output:
```
-rw-r----- 1 peserta1 peserta1 0 ... draft.txt
```
**Penjelasan perhitungan** (poin kunci untuk dinilai):
- Base permission file baru = `666` (rw-rw-rw-)
- Umask `027` menghapus bit sebagai berikut (biner per grup owner/group/other):
  - `666` = `110 110 110`
  - `027` = `000 010 111`
  - Hasil = base **AND NOT(umask)** → `110 100 000` = **640** = `rw-r-----`
- Jadi: owner tetap `rw-`, group hanya `r--` (bit write dihapus oleh umask), other `---` (semua bit dihapus)

**16. Bonus — chown (opsional, butuh sudo)**
```bash
sudo chown user_lain:group_lain ~/nusantara-apps/testing
ls -ld ~/nusantara-apps/testing
```
> Jika tidak ada user lain di sistem lab, poin ini bisa diskip / dijelaskan secara konseptual saja oleh instruktur.

---

## Bagian E — Symbolic Link & Hard Link

**17. File di shared/**
```bash
cd ~/nusantara-apps
cat > shared/coding-standard.txt << EOF
Coding Standard - Nusantara Apps
1. Gunakan snake_case untuk nama variable
2. Setiap fungsi wajib memiliki komentar
EOF
```

**18. Symbolic link ke source-code & testing**
```bash
ln -s ../shared/coding-standard.txt source-code/coding-standard.txt
ln -s ../shared/coding-standard.txt testing/coding-standard.txt

ls -l source-code/coding-standard.txt
```
Output menunjukkan tipe file `l` (link):
```
lrwxrwxrwx 1 peserta1 peserta1 ... coding-standard.txt -> ../shared/coding-standard.txt
```

**19. Hard link + bukti inode sama**
```bash
ln backup/requirement-backup.txt dokumen/requirement-final.txt
ls -li backup/requirement-backup.txt dokumen/requirement-final.txt
```
Output (perhatikan **nomor inode di kolom pertama sama**, dan **link count = 2**):
```
123456 -rw-r--r-- 2 peserta1 peserta1 ... backup/requirement-backup.txt
123456 -rw-r--r-- 2 peserta1 peserta1 ... dokumen/requirement-final.txt
```

**20. Hapus salah satu, data tetap ada**
```bash
rm backup/requirement-backup.txt
cat dokumen/requirement-final.txt      # isi file masih utuh
ls -li dokumen/requirement-final.txt   # link count sekarang menjadi 1
```
**Penjelasan konsep:** Hard link menunjuk ke **inode/data block yang sama**. Data baru benar-benar terhapus jika **semua** hard link ke inode tersebut dihapus. Ini berbeda dengan symbolic link yang akan menjadi **broken link** jika file aslinya dihapus.

---

## Bagian F — Laporan Akhir

**21. Laporan setup**
```bash
cd ~/nusantara-apps
ls -laR > laporan_setup.txt
cat laporan_setup.txt
```

---

## RUBRIK PENILAIAN (untuk Instruktur)

| Bagian | Aspek Dinilai | Poin |
|---|---|---|
| A | Investigasi sistem benar & lengkap (5 soal) | 15 |
| B | Struktur direktori benar (termasuk efisiensi mkdir -p / brace expansion) | 15 |
| C | Operasi file (cp/mv/grep/file) tepat sasaran | 15 |
| D | Permission (600/750) & pemahaman perhitungan umask 027 | 25 |
| E | Symbolic link & hard link terbukti dengan inode, paham konsep hapus | 25 |
| F | Laporan akhir tersimpan & lengkap | 5 |
| **Total** | | **100** |

**Kriteria Lulus:** ≥ 70 poin dianggap kompeten pada Bab 1 & 2.

---
*Studi kasus ini dirancang sebagai penutup Hari 1, menggabungkan seluruh skill Bab 1 & 2 dalam satu alur kerja proyek yang realistis dan mudah diperiksa oleh instruktur secara objektif (semua jawaban dapat diverifikasi lewat output terminal).*
