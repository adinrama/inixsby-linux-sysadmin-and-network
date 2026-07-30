# STUDI KASUS BERBASIS PROYEK — HARI 2
## Membangun Toolkit Administrasi "Nusantara Apps"
### (Integrasi Bab 3: Pemrograman Shell + Bab 4: Pemrograman Shell 2)

---

## INFORMASI UMUM

| Item | Keterangan |
|---|---|
| Digunakan setelah | Bab 3 & Bab 4 selesai diajarkan (akhir Hari 2) |
| Durasi pengerjaan | 60–90 menit |
| Sifat | Individu (masing-masing peserta di VM sendiri) |
| Prasyarat | Sudah menyelesaikan Studi Kasus Hari 1 (folder `~/nusantara-apps/` sudah ada) |
| Output akhir | Kumpulan script `.sh` + 1 script gabungan `admin-toolkit.sh` |

> Jika ada peserta yang belum menyelesaikan Studi Kasus Hari 1, instruktur cukup memberi perintah cepat berikut agar folder tersedia:
> ```bash
> mkdir -p ~/nusantara-apps/{dokumen,source-code,testing,shared,backup}
> cd ~/nusantara-apps/dokumen
> echo "Project: Nusantara Apps - Deadline: 30 Agustus 2026" > requirement.txt
> echo "Meeting notes - deadline internal 25 Agustus 2026" > meeting-notes.txt
> touch catatan.txt progress.txt
> ```

---

# BAGIAN SOAL (Untuk Peserta)

## Latar Belakang Skenario

Workspace proyek **"Nusantara Apps"** sudah siap sejak Hari 1. Sekarang tim mulai bekerja, dan Team Lead menyadari banyak pekerjaan administratif yang **berulang setiap hari**: mengecek file, membackup dokumen, memonitor proses, dan lain-lain.

Anda ditugaskan membangun **Toolkit Administrasi** berupa kumpulan shell script yang nantinya akan dipakai tim setiap hari melalui satu menu terpusat. Kerjakan bertahap — setiap bagian akan menjadi 1 file script, dan di akhir seluruhnya akan digabung menjadi satu sistem menu utuh.

Simpan semua script di dalam folder baru:
```bash
mkdir -p ~/nusantara-apps/scripts
cd ~/nusantara-apps/scripts
```

---

### BAGIAN A — Script Dasar: Variable & Substitusi
*(Bab 3: variable, read, command substitution, parameter khusus `$$`)*

Buat script **`info-sistem.sh`** yang menjadi halaman pembuka toolkit.

1. Definisikan variable `PROJECT="Nusantara Apps"` dan tampilkan dengan `echo`.
2. Gunakan `read` untuk meminta input **nama** dan **role** pengguna (gunakan `echo -n` agar prompt tampil sebaris dengan input).
3. Gunakan **command substitution** (backtick atau `$()`) untuk mengambil tanggal (`date`), nama server (`hostname`), dan user aktif (`whoami`), simpan masing-masing ke variable.
4. Tampilkan seluruh informasi di atas dalam format rapi, **termasuk PID dari script yang sedang berjalan** (`$$`).

---

### BAGIAN B — Validasi Parameter & Kondisi
*(Bab 3: `$0` `$1` `$#`, `test`/`[ ]`, `if-elif-else`, `$?`)*

Buat script **`cek-file.sh`** yang dipanggil dengan format: `./cek-file.sh <nama_file>`

5. Validasi jumlah parameter harus **tepat 1** (`$#`). Jika tidak, tampilkan pesan penggunaan yang **menyertakan nama script itu sendiri** (`$0`), lalu `exit` dengan status `1`.
6. Gunakan `test`/`[ ]` untuk memeriksa apakah file yang diberikan ($1): file biasa (`-f`), bisa dibaca (`-r`), bisa ditulis (`-w`), atau ternyata sebuah direktori (`-d`). Susun dengan `if-elif-else`.
7. Sebelum masuk ke pengecekan di atas, jalankan `[ -f "$FILE" ]` **secara terpisah**, lalu langsung tampilkan `$?` di baris berikutnya untuk membuktikan bagaimana status exit bekerja.

---

### BAGIAN C — Operator Aritmatika & Logika
*(Bab 3: `expr`, operator bilangan bulat, operator logika `-a`, `&&`)*

Buat script **`cek-kuota.sh`** untuk mensimulasikan monitoring kapasitas penyimpanan tim.

8. Minta input **kapasitas terpakai (MB)** dan **kapasitas maksimum (MB)** via `read`. Hitung persentase pemakaian menggunakan `expr` — **perhatikan urutan operasi** agar hasil tidak selalu 0 akibat pembulatan integer.
9. Gunakan `elif` berjenjang dengan operator `-ge`/`-lt` untuk menentukan 3 status:
   - **AMAN** jika < 70%
   - **WARNING** jika 70% – 89%
   - **KRITIS** jika ≥ 90%
10. Tambahkan satu baris terpisah yang mengombinasikan `&&` untuk menampilkan pesan **"Kirim notifikasi ke Team Lead!"** hanya jika status KRITIS **dan** kapasitas terpakai lebih dari 0.

---

### BAGIAN D — Menu Interaktif
*(Bab 4: instruksi dummy `:`, `while`, `case...esac`, `break`)*

Buat script **`menu-admin.sh`** sebagai kerangka menu utama toolkit.

11. Gunakan `while` dengan kondisi **dummy (`:`)** agar menu tampil berulang terus-menerus.
12. Gunakan `case...esac` untuk menangani pilihan 1–4:
    1. Cek Proses Aktif
    2. Backup Dokumen
    3. Info Sistem
    4. Keluar

    Sertakan opsi default (`*`) untuk menangani input yang tidak valid.
13. Gunakan `break` agar program benar-benar keluar dari loop saat user memilih opsi **Keluar**.

---

### BAGIAN E — Otomasi Batch dengan For Loop
*(Bab 4: `for` dengan wildcard, `for` gaya C)*

Buat script **`backup-dokumen.sh`** untuk membackup seluruh dokumen proyek secara otomatis.

14. Gunakan `for file in *.txt` (dengan **wildcard**) untuk melakukan iterasi ke semua file `.txt` di dalam folder `~/nusantara-apps/dokumen/`.
15. Setiap file yang diproses harus di-copy ke folder `~/nusantara-apps/backup/` dengan **suffix tanggal** pada nama filenya. Hitung **total file yang berhasil dibackup** menggunakan variable counter yang diakumulasi dengan `expr`.
16. Tambahkan **`for` gaya C** (`for (( i=1; i<=3; i++ ))`) untuk menampilkan pesan "Backup selesai - percobaan ke-$i" sebanyak 3 kali di akhir proses (simulasi verifikasi ulang).

---

### BAGIAN F — Fungsi & Modularisasi
*(Bab 4: fungsi, `return`, variable `local` vs global)*

17. Buat fungsi **`konfirmasi()`** (mirip Lab 2 Bab 4 di materi) yang menanyakan **(Y)es/(N)o/(C)ontinue**, lalu `return` dengan nilai: `0` untuk Yes, `1` untuk No, `2` untuk Continue. Gunakan variable **`local`** di dalam fungsi ini.
18. Buat fungsi **`cek_proses()`** yang menghitung total proses aktif di sistem (`ps -ef | wc -l`), simpan hasilnya ke variable **`local`** di dalam fungsi.
19. Panggil kedua fungsi tersebut dari "program utama" di luar fungsi, gunakan `$?` untuk menangkap hasil `konfirmasi()`. Kemudian, **coba tampilkan variable yang didefinisikan `local` di dalam fungsi dari luar fungsi** — amati dan jelaskan hasilnya.

---

### BAGIAN G — Integrasi Akhir: Toolkit Lengkap

20. Gabungkan seluruh script dari **Bagian D, E, dan F** menjadi **satu file utuh**: `admin-toolkit.sh`. Menu utama harus bisa:
    - Memanggil fungsi `cek_proses()` untuk opsi "Cek Proses Aktif"
    - Memanggil fungsi `konfirmasi()` **sebelum** menjalankan backup — backup hanya dijalankan jika user menjawab Yes atau Continue
    - Memanggil logika backup (dari Bagian E) untuk opsi "Backup Dokumen"
    - Tetap menggunakan `while :` + `case` + `break` sebagai kerangka menu

    Uji jalankan `admin-toolkit.sh` dari awal sampai keluar (opsi 4), pastikan seluruh alur berjalan tanpa error.

---

## Checklist Deliverable

- [ ] `info-sistem.sh` menampilkan variable, input, substitusi, dan `$$`
- [ ] `cek-file.sh` memvalidasi `$#`, menggunakan `$0`, dan `if-elif-else` dengan test operator file
- [ ] `cek-kuota.sh` menghitung persentase dengan `expr` dan `elif` berjenjang tanpa error logika
- [ ] `menu-admin.sh` berjalan berulang (`while :`) dan `break` bekerja dengan benar
- [ ] `backup-dokumen.sh` berhasil membackup semua `.txt` dengan `for` wildcard
- [ ] Fungsi `konfirmasi()` & `cek_proses()` berjalan, dan **perbedaan local/global bisa dijelaskan peserta**
- [ ] `admin-toolkit.sh` sebagai integrasi akhir berjalan end-to-end tanpa error

---
---

# KUNCI JAWABAN (Untuk Instruktur)

## Bagian A — `info-sistem.sh`

```bash
#!/bin/bash
# info-sistem.sh - Bab 3: variable, read, command substitution, $$

PROJECT="Nusantara Apps"
echo "Selamat datang di Toolkit Administrasi $PROJECT"
echo ""

echo -n "Masukkan nama Anda   : "
read nama
echo -n "Masukkan role Anda (Lead/Dev/Tester): "
read role

TANGGAL=`date`
HOST=`hostname`
USER_AKTIF=`whoami`

echo ""
echo "=================================================="
echo "Proyek        : $PROJECT"
echo "Nama Pengguna : $nama"
echo "Role          : $role"
echo "Waktu         : $TANGGAL"
echo "Server        : $HOST"
echo "User Sistem   : $USER_AKTIF"
echo "PID Script    : $$"
echo "=================================================="
```

**Poin pembahasan instruktur:**
- `$$` akan berbeda nilainya **setiap kali script dijalankan ulang** — bagus untuk menunjukkan konsep PID yang unik per proses.
- Perhatikan penggunaan backtick `` `perintah` `` — bisa juga diganti `$(perintah)` (sintaks lebih modern), keduanya sah diterima.

---

## Bagian B — `cek-file.sh`

```bash
#!/bin/bash
# cek-file.sh - Bab 3: parameter $0 $1 $#, test, if-elif-else, $?

if [ $# -ne 1 ]
then
    echo "Usage: $0 <nama_file>"
    exit 1
fi

FILE=$1

# Demo $? terpisah
[ -f "$FILE" ]
echo "Status exit dari pengecekan [-f \"$FILE\"] adalah: $?"
echo ""

if [ -f "$FILE" ]
then
    echo "$FILE adalah file biasa."
    if [ -r "$FILE" ] && [ -w "$FILE" ]
    then
        echo "Akses: dapat dibaca DAN ditulis."
    elif [ -r "$FILE" ]
    then
        echo "Akses: hanya dapat dibaca (read-only)."
    else
        echo "Akses: TIDAK ada permission baca."
    fi
elif [ -d "$FILE" ]
then
    echo "$FILE adalah sebuah DIREKTORI, bukan file biasa."
else
    echo "$FILE tidak ditemukan sama sekali."
    exit 1
fi
```

**Uji coba:**
```bash
./cek-file.sh                                   # Usage: ./cek-file.sh <nama_file>
./cek-file.sh ~/nusantara-apps/dokumen/requirement.txt
./cek-file.sh ~/nusantara-apps/source-code       # akan terdeteksi sebagai direktori
./cek-file.sh file-tidak-ada.txt
```
**Penjelasan `$?`:** Nilai `$?` selalu berasal dari **perintah paling terakhir yang dieksekusi**. Instruktur bisa tekankan bahwa jika `echo` diletakkan di antara pengecekan dan pembacaan `$?`, nilainya akan berubah menjadi exit status `echo` (biasanya 0) — bukan lagi dari `test`.

---

## Bagian C — `cek-kuota.sh`

```bash
#!/bin/bash
# cek-kuota.sh - Bab 3: expr, operator bilangan bulat, elif, &&

echo -n "Masukkan kapasitas terpakai (MB): "
read terpakai
echo -n "Masukkan kapasitas maksimum (MB) : "
read maksimum

# PENTING: kalikan dulu baru dibagi, agar tidak selalu jadi 0 (integer division)
persen=`expr $terpakai \* 100 / $maksimum`

echo ""
echo "Pemakaian saat ini: $persen%"

if [ "$persen" -ge 90 ]
then
    echo "STATUS: KRITIS! Segera hapus file yang tidak perlu."
elif [ "$persen" -ge 70 -a "$persen" -lt 90 ]
then
    echo "STATUS: WARNING, kapasitas mulai menipis."
else
    echo "STATUS: AMAN."
fi

[ "$persen" -ge 90 ] && [ "$terpakai" -gt 0 ] && echo "Kirim notifikasi ke Team Lead!"
```

**Poin pembahasan instruktur:**
- Kesalahan umum peserta: menulis `expr $terpakai / $maksimum \* 100` — hasil pembagian dibulatkan ke bawah dulu (integer), sehingga jika `terpakai < maksimum` hasilnya sering **0%** meskipun sudah terpakai banyak. Urutan **kali dulu baru bagi** adalah kuncinya.
- Bandingkan dengan `Script/pembagian.sh` yang sudah didemokan di Bab 3 — di sana pakai `bc` untuk hasil desimal. Bisa jadi diskusi: *"Kapan sebaiknya pakai `expr`, kapan pakai `bc`?"*
- `-a` di sini adalah operator AND **di dalam** `test`, sedangkan `&&` adalah AND di **level shell** (dua `[ ]` terpisah) — keduanya benar, tapi tunjukkan bedanya.

---

## Bagian D — `menu-admin.sh`

```bash
#!/bin/bash
# menu-admin.sh - Bab 4: while :, case, break

while :
do
    echo "======================================"
    echo " MENU ADMINISTRASI - NUSANTARA APPS"
    echo "======================================"
    echo "1. Cek Proses Aktif"
    echo "2. Backup Dokumen"
    echo "3. Info Sistem"
    echo "4. Keluar"
    echo -n "Pilihan Anda: "
    read pilihan

    case $pilihan in
        1)
            ps -ef | head -10
            ;;
        2)
            echo "(placeholder) menjalankan backup..."
            ;;
        3)
            bash info-sistem.sh
            ;;
        4)
            echo "Terima kasih, keluar dari toolkit."
            break
            ;;
        *)
            echo "Pilihan tidak valid! Silakan pilih 1-4."
            ;;
    esac
    echo ""
done

echo "Program selesai."
```

**Poin pembahasan instruktur:**
- `:` sebagai kondisi selalu bernilai TRUE (dummy), sehingga `while :` = infinite loop yang **hanya** bisa dihentikan lewat `break` di dalam `case`.
- Tekankan pentingnya opsi `*)` sebagai **default case** — tanpa ini, input salah (misalnya huruf) tidak akan ditangani.

---

## Bagian E — `backup-dokumen.sh`

```bash
#!/bin/bash
# backup-dokumen.sh - Bab 4: for wildcard, for gaya C

cd ~/nusantara-apps/dokumen || { echo "Folder dokumen tidak ditemukan"; exit 1; }

tanggal=`date +%Y%m%d`
jumlah=0

for file in *.txt
do
    cp "$file" ~/nusantara-apps/backup/"$file"."$tanggal".bak
    echo "Backup: $file -> $file.$tanggal.bak"
    jumlah=`expr $jumlah + 1`
done

echo ""
echo "Total file yang di-backup: $jumlah"
echo ""

for (( i=1; i<=3; i++ ))
do
    echo "Backup selesai - percobaan verifikasi ke-$i"
done
```

**Contoh output:**
```
Backup: requirement.txt -> requirement.txt.20260729.bak
Backup: meeting-notes.txt -> meeting-notes.txt.20260729.bak

Total file yang di-backup: 2

Backup selesai - percobaan verifikasi ke-1
Backup selesai - percobaan verifikasi ke-2
Backup selesai - percobaan verifikasi ke-3
```

**Poin pembahasan instruktur:**
- `for file in *.txt` — wildcard `*` di-**expand oleh shell sebelum** `for` dieksekusi, bukan oleh `for` itu sendiri (konsep globbing).
- Jika folder `dokumen/` kosong dari file `.txt`, secara default `file` akan bernilai literal string `*.txt` (karena tidak ada yang cocok) — ini bisa jadi bahan diskusi edge-case menarik.

---

## Bagian F — Fungsi & Modularisasi

```bash
#!/bin/bash
# fungsi-toolkit.sh - Bab 4: fungsi, return, local vs global

konfirmasi() {
    local jawaban
    echo -n "(Y)es/(N)o/(C)ontinue? [Y] "
    read jawaban
    jawaban=`echo "$jawaban" | tr '[a-z]' '[A-Z]'`

    if [ "$jawaban" = "" -o "$jawaban" = "Y" ]
    then
        return 0
    elif [ "$jawaban" = "C" ]
    then
        return 2
    else
        return 1
    fi
}

cek_proses() {
    local total
    total=`ps -ef | wc -l`
    echo "Jumlah proses aktif saat ini: $total"
}

# ===== Program Utama =====
cek_proses

konfirmasi
hasil=$?

if [ $hasil -eq 0 ]
then
    echo "Anda memilih YES -> melanjutkan backup..."
elif [ $hasil -eq 2 ]
then
    echo "Anda memilih CONTINUE -> proses ditunda sementara."
else
    echo "Anda memilih NO -> dibatalkan."
fi

echo ""
echo "Mencoba akses variable 'total' di luar fungsi cek_proses(): [$total]"
```

**Contoh output baris terakhir:**
```
Mencoba akses variable 'total' di luar fungsi cek_proses(): []
```

**Penjelasan konsep (poin kunci untuk dinilai):**
- Variable `total` dan `jawaban` dideklarasikan `local` **di dalam** fungsi, sehingga **tidak dikenal** di luar fungsi tersebut — terbukti dari nilai yang kosong `[]`.
- `return 0/1/2` **bukan** mengembalikan nilai data, melainkan **status exit fungsi**, sehingga harus ditangkap segera lewat `$?` **tepat setelah** fungsi dipanggil (sebelum perintah lain dijalankan, karena `$?` akan berubah).

---

## Bagian G — Integrasi Akhir: `admin-toolkit.sh`

```bash
#!/bin/bash
# admin-toolkit.sh - Integrasi penuh Bab 3 & Bab 4

PROJECT="Nusantara Apps"
DOKUMEN_DIR=~/nusantara-apps/dokumen
BACKUP_DIR=~/nusantara-apps/backup

# ---------- FUNGSI ----------
konfirmasi() {
    local jawaban
    echo -n "(Y)es/(N)o/(C)ontinue? [Y] "
    read jawaban
    jawaban=`echo "$jawaban" | tr '[a-z]' '[A-Z]'`

    if [ "$jawaban" = "" -o "$jawaban" = "Y" ]
    then
        return 0
    elif [ "$jawaban" = "C" ]
    then
        return 2
    else
        return 1
    fi
}

cek_proses() {
    local total
    total=`ps -ef | wc -l`
    echo "Jumlah proses aktif saat ini: $total"
}

jalankan_backup() {
    cd "$DOKUMEN_DIR" || { echo "Folder dokumen tidak ditemukan"; return 1; }
    tanggal=`date +%Y%m%d`
    jumlah=0
    for file in *.txt
    do
        cp "$file" "$BACKUP_DIR"/"$file"."$tanggal".bak
        echo "Backup: $file -> $file.$tanggal.bak"
        jumlah=`expr $jumlah + 1`
    done
    echo "Total file yang di-backup: $jumlah"
}

# ---------- MENU UTAMA ----------
while :
do
    echo "======================================"
    echo " ADMIN TOOLKIT - $PROJECT"
    echo "======================================"
    echo "1. Cek Proses Aktif"
    echo "2. Backup Dokumen"
    echo "3. Info Sistem Singkat"
    echo "4. Keluar"
    echo -n "Pilihan Anda: "
    read pilihan

    case $pilihan in
        1)
            cek_proses
            ;;
        2)
            konfirmasi
            hasil=$?
            if [ $hasil -eq 0 -o $hasil -eq 2 ]
            then
                jalankan_backup
            else
                echo "Backup dibatalkan oleh user."
            fi
            ;;
        3)
            echo "Server: `hostname` | User: `whoami` | Tanggal: `date`"
            ;;
        4)
            echo "Terima kasih telah menggunakan Admin Toolkit $PROJECT."
            break
            ;;
        *)
            echo "Pilihan tidak valid! Silakan pilih 1-4."
            ;;
    esac
    echo ""
done
```

**Skenario uji end-to-end yang harus dicoba peserta:**
1. Jalankan `bash admin-toolkit.sh`
2. Pilih `1` → pastikan jumlah proses tampil
3. Pilih `2` → jawab `N` → pastikan backup **dibatalkan**
4. Pilih `2` lagi → jawab `Y` (atau Enter) → pastikan backup **berjalan** dan file baru muncul di `~/nusantara-apps/backup/`
5. Pilih `3` → pastikan info singkat tampil
6. Pilih angka lain (misal `9`) → pastikan pesan "Pilihan tidak valid" muncul dan menu tampil lagi
7. Pilih `4` → pastikan program benar-benar keluar (bukan lanjut ke iterasi berikutnya)

---

## RUBRIK PENILAIAN (untuk Instruktur)

| Bagian | Aspek Dinilai | Poin |
|---|---|---|
| A | Variable, `read`, command substitution, `$$` benar | 10 |
| B | Validasi `$#`/`$0`, test operator file, `if-elif-else`, pemahaman `$?` | 15 |
| C | `expr` dengan urutan operasi benar, `elif` berjenjang, `&&`/`-a` tepat | 15 |
| D | `while :` + `case` + `break` berfungsi tanpa infinite-loop macet | 15 |
| E | `for` wildcard bekerja benar, counter dengan `expr`, `for` gaya C | 15 |
| F | Fungsi & `return` benar, **bisa menjelaskan local vs global** | 15 |
| G | Integrasi akhir berjalan end-to-end tanpa error (skenario uji 7 langkah) | 15 |
| **Total** | | **100** |

**Kriteria Lulus:** ≥ 70 poin dianggap kompeten pada Bab 3 & 4.

**Kesalahan umum yang sering ditemukan (checklist review cepat instruktur):**
- Lupa `chmod +x` sebelum menjalankan `./script.sh` (harus pakai `bash script.sh` atau chmod dulu)
- Spasi di sekitar tanda `=` saat assignment variable (`VAR = nilai` → error)
- Lupa tanda kutip di sekitar `$1`/`$FILE` saat nama file mengandung spasi
- Lupa backslash pada `\*` di dalam `expr` (tanpa itu, shell menganggapnya sebagai wildcard)
- Menaruh perintah lain di antara pengecekan kondisi dan `echo $?`, sehingga nilai `$?` yang tertangkap salah

---
*Studi kasus ini melanjutkan proyek "Nusantara Apps" dari Hari 1, sehingga peserta merasakan progres nyata: dari sekadar mengatur file & folder (Hari 1) menjadi mengotomasi pekerjaan tersebut dengan shell script (Hari 2).*
