# Panduan Penggunaan Alat Penggantian Teks Esperanto dan Anotasi Ruby

## Daftar Isi
1. [Pendahuluan](#pendahuluan)
2. [Halaman Utama: Penggantian Teks Esperanto](#halaman-utama-penggantian-teks-esperanto)
   - [Persiapan Berkas JSON](#persiapan-berkas-json)
   - [Pemilihan Format Keluaran](#pemilihan-format-keluaran)
   - [Memasukkan Teks Esperanto](#memasukkan-teks-esperanto)
   - [Mengatur Cara Menampilkan Karakter Khusus](#mengatur-cara-menampilkan-karakter-khusus)
   - [Melihat dan Mengunduh Hasil](#melihat-dan-mengunduh-hasil)
3. [Halaman Pembuatan Berkas JSON](#halaman-pembuatan-berkas-json)
   - [Persiapan Berkas CSV](#persiapan-berkas-csv)
   - [Persiapan Berkas JSON untuk Pemecahan Akar Kata](#persiapan-berkas-json-untuk-pemecahan-akar-kata)
   - [Pengaturan Pemrosesan Paralel](#pengaturan-pemrosesan-paralel)
   - [Membuat dan Mengunduh Berkas JSON Final](#membuat-dan-mengunduh-berkas-json-final)
4. [Fitur Lanjutan](#fitur-lanjutan)
   - [Simbol % untuk Mencegah Penggantian](#simbol--untuk-mencegah-penggantian)
   - [Simbol @ untuk Penggantian Lokal](#simbol--untuk-penggantian-lokal)
   - [Pemrosesan Paralel](#pemrosesan-paralel)
5. [Contoh Penggunaan](#contoh-penggunaan)

## Pendahuluan

Aplikasi ini adalah alat untuk mengganti teks Esperanto dengan karakter Kanji (Han) atau menambahkan anotasi Ruby. Anotasi Ruby adalah cara untuk menampilkan teks kecil di atas karakter untuk menunjukkan cara baca atau arti. Anda dapat menggunakan aplikasi ini untuk:

- Mengganti kata-kata Esperanto dengan karakter Kanji
- Menambahkan anotasi Ruby pada teks Esperanto
- Menyesuaikan ukuran teks anotasi berdasarkan panjang kata
- Mengkonversi format karakter Esperanto (ĉ, ĝ, ĥ, ĵ, ŝ, ŭ)

Aplikasi ini memiliki dua halaman utama:
1. **Halaman Utama**: Untuk melakukan penggantian teks dan menambahkan anotasi
2. **Halaman Pembuatan Berkas JSON**: Untuk membuat berkas aturan penggantian yang digunakan pada halaman utama

## Halaman Utama: Penggantian Teks Esperanto

Halaman utama adalah tempat Anda melakukan penggantian teks Esperanto. Berikut adalah langkah-langkah penggunaannya:

### Persiapan Berkas JSON

Pertama, Anda perlu memuat berkas JSON yang berisi aturan penggantian:

1. Pilih salah satu opsi:
   - **Gunakan JSON bawaan (default)**: Menggunakan berkas JSON yang sudah disediakan aplikasi
   - **Unggah berkas**: Jika Anda memiliki berkas JSON kustom

2. Jika Anda ingin melihat contoh berkas JSON, klik "Unduh Contoh Berkas JSON" di bagian ekspander.

### Pemilihan Format Keluaran

Pilih format keluaran yang Anda inginkan dari menu dropdown. Tersedia beberapa opsi:

- **Format HTML dengan anotasi Ruby dan penyesuaian ukuran**: Menampilkan karakter Esperanto dengan anotasi Ruby di atasnya, dan ukuran teks Ruby disesuaikan secara otomatis
- **Format HTML dengan anotasi Ruby, penyesuaian ukuran, dan penggantian karakter Kanji**: Sama seperti di atas, tetapi karakter Esperanto diganti dengan Kanji
- **Format HTML**: Format HTML dasar tanpa penyesuaian ukuran
- **Format HTML dengan penggantian karakter Kanji**: Format HTML dasar dengan karakter Esperanto diganti Kanji
- **Format dengan tanda kurung**: Anotasi ditampilkan dalam tanda kurung setelah teks
- **Format dengan tanda kurung dan penggantian karakter Kanji**: Teks Esperanto dalam tanda kurung setelah Kanji
- **Hanya menyimpan teks yang telah digantikan**: Menampilkan hasil penggantian saja tanpa teks asli

### Memasukkan Teks Esperanto

Anda memiliki dua cara untuk memasukkan teks:

1. **Masukkan secara manual**: Ketik atau tempel teks Esperanto langsung ke area teks
2. **Unggah berkas**: Unggah berkas teks (UTF-8) yang berisi teks Esperanto

Setelah memilih metode masukan, ketik atau tempel teks Esperanto di area teks yang disediakan, atau unggah berkas teks.

### Mengatur Cara Menampilkan Karakter Khusus

Pilih cara menampilkan karakter khusus Esperanto (ĉ, ĝ, ĥ, ĵ, ŝ, ŭ) di hasil:

- **Karakter aksen di atas (ĉ → c + ˆ)**: Menampilkan karakter dengan aksen di atas
- **Format x (ĉ → cx)**: Menampilkan dalam gaya "x-system" Esperanto
- **Format ^ (ĉ → c^)**: Menampilkan dengan simbol topi (^) setelah konsonan

Setelah mengatur semua pilihan, klik tombol "Kirim" untuk memproses teks.

### Melihat dan Mengunduh Hasil

Setelah pemrosesan selesai, Anda akan melihat hasil dalam tab yang sesuai:

- Jika Anda memilih format HTML, ada dua tab:
  - **Pratinjau HTML**: Menampilkan hasil HTML yang dirender
  - **Hasil (Kode HTML)**: Menampilkan kode HTML mentah

- Jika Anda memilih format lain, hasil akan ditampilkan dalam tab "Teks Hasil"

Untuk menyimpan hasil, klik tombol "Unduh Hasil" untuk mengunduh file hasil.

## Halaman Pembuatan Berkas JSON

Halaman kedua digunakan untuk membuat berkas JSON kustom yang berisi aturan penggantian teks. Anda dapat mengakses halaman ini melalui navigasi sidebar di aplikasi.

### Persiapan Berkas CSV

Langkah pertama adalah menyiapkan berkas CSV yang berisi pasangan akar kata Esperanto dan teks penggantinya:

1. Pilih salah satu opsi:
   - **Unggah Berkas CSV**: Jika Anda memiliki berkas CSV sendiri
   - **Gunakan Default**: Menggunakan berkas CSV bawaan aplikasi

Format berkas CSV harus memiliki dua kolom:
- Kolom pertama: Akar kata Esperanto
- Kolom kedua: Teks pengganti (terjemahan bahasa Indonesia, Kanji, dll.)

Di bagian ekspander "Daftar Berkas Contoh", Anda dapat mengunduh beberapa contoh berkas CSV untuk referensi.

### Persiapan Berkas JSON untuk Pemecahan Akar Kata

Langkah berikutnya adalah memuat berkas JSON yang berisi aturan pemecahan akar kata:

1. Pilih opsi untuk berkas JSON pertama (aturan pemecahan akar kata):
   - **Unggah Berkas JSON**: Jika Anda memiliki berkas kustom
   - **Gunakan Default**: Menggunakan berkas bawaan aplikasi

2. Pilih opsi untuk berkas JSON kedua (teks pengganti):
   - **Unggah Berkas JSON**: Jika Anda memiliki berkas kustom
   - **Gunakan Default**: Menggunakan berkas bawaan aplikasi

Berkas JSON pertama menentukan bagaimana akar kata Esperanto dipecah, terutama untuk membedakan antara akar kata, akhiran kata kerja, dan bagian kata lainnya. Berkas JSON kedua berisi pengaturan penggantian teks khusus.

### Pengaturan Pemrosesan Paralel

Untuk berkas yang besar, Anda dapat menggunakan pemrosesan paralel untuk mempercepat pembuatan:

1. Buka bagian pengaturan lanjutan
2. Centang kotak "Gunakan pemrosesan paralel" jika diinginkan
3. Atur jumlah proses paralel (biasanya 2-6, tergantung pada kemampuan CPU komputer Anda)

### Membuat dan Mengunduh Berkas JSON Final

Setelah semua pengaturan selesai:

1. Klik tombol "Buat Berkas JSON untuk Penggantian"
2. Tunggu hingga proses selesai (bisa memakan waktu beberapa menit untuk data besar)
3. Setelah selesai, klik tombol "Unduh Daftar Penggantian Final" untuk menyimpan berkas JSON

Berkas JSON yang dihasilkan berisi tiga jenis daftar penggantian:
- Daftar penggantian global
- Daftar penggantian untuk akar kata dua karakter
- Daftar penggantian lokal untuk teks yang dibungkus dengan simbol "@"

## Fitur Lanjutan

### Simbol % untuk Mencegah Penggantian

Jika Anda ingin beberapa bagian teks tidak diganti, Anda bisa membungkusnya dengan simbol "%":

```
La %hundo% estas bela. (Bagian "hundo" tidak akan diganti)
```

Teks di dalam simbol "%" (hingga 50 karakter) akan dibiarkan apa adanya dalam hasil akhir.

### Simbol @ untuk Penggantian Lokal

Jika Anda ingin suatu bagian teks diganti secara lokal (hanya bagian itu), gunakan simbol "@":

```
La @kato@ estas bela. (Hanya "kato" yang akan diganti berdasarkan aturan lokal)
```

Teks di dalam simbol "@" (hingga 18 karakter) akan diganti berdasarkan daftar penggantian lokal.

### Pemrosesan Paralel

Untuk teks yang panjang, Anda dapat mengaktifkan pemrosesan paralel:

1. Buka bagian "Pengaturan Lanjutan (Pemrosesan Paralel)"
2. Centang kotak "Gunakan pemrosesan paralel"
3. Atur jumlah proses yang diinginkan (2-4)

Pemrosesan paralel dapat mempercepat pemrosesan teks yang panjang secara signifikan dengan memanfaatkan beberapa inti CPU.

## Contoh Penggunaan

### Contoh 1: Menambahkan Anotasi Ruby Dasar

**Input:**
```
La kato estas en la domo.
```

**Pengaturan:**
- Format: Format HTML dengan anotasi Ruby
- Karakter: Karakter aksen di atas

**Hasil:**
```html
<ruby>La<rt>Artikel</rt></ruby> <ruby>kato<rt>kucing</rt></ruby> <ruby>estas<rt>adalah</rt></ruby> <ruby>en<rt>di dalam</rt></ruby> <ruby>la<rt>artikel</rt></ruby> <ruby>domo<rt>rumah</rt></ruby>.
```

### Contoh 2: Penggantian dengan Karakter Kanji

**Input:**
```
Mi lernas Esperanton.
```

**Pengaturan:**
- Format: Format HTML dengan penggantian karakter Kanji
- Karakter: Format x

**Hasil:**
```html
<ruby>私<rt>Mi</rt></ruby> <ruby>学ぶ<rt>lernas</rt></ruby> <ruby>世界語<rt>Esperanton</rt></ruby>.
```

### Contoh 3: Menggunakan Simbol % dan @

**Input:**
```
%La kato% @dormas@ sur la tablo.
```

**Pengaturan:**
- Format: Format dengan tanda kurung
- Karakter: Format ^

**Hasil:**
```
La kato dormadi(tidur) sur la tablo(meja).
```

Dalam contoh ini, "La kato" tetap tidak berubah, "dormas" diganti berdasarkan aturan lokal, dan kata-kata lainnya diganti berdasarkan aturan global.

---

Panduan ini memberikan informasi dasar tentang cara menggunakan aplikasi penggantian teks Esperanto. Untuk informasi lebih lanjut, silakan kunjungi tautan repositori GitHub yang tercantum di bagian bawah halaman utama aplikasi.
