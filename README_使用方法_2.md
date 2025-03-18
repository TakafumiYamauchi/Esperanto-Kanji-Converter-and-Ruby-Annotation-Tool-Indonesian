# Panduan Penggunaan Aplikasi Penggantian Teks Esperanto dengan Karakter Han (Kanji) dan Anotasi Ruby

## Daftar Isi
1. Pendahuluan dan Tujuan Aplikasi
2. Memulai dengan Halaman Utama
3. Cara Menggunakan Aplikasi
4. Format Keluaran yang Tersedia
5. Membuat Berkas JSON Kustom
6. Pengaturan Lanjutan
7. Tips Penggunaan 
8. Pertanyaan yang Sering Diajukan (FAQ)

## 1. Pendahuluan dan Tujuan Aplikasi

Aplikasi ini dirancang untuk membantu pengguna yang bekerja dengan teks Esperanto, memungkinkan pengubahan atau penyertaan teks Esperanto dengan karakter Han (Kanji) serta penambahan anotasi Ruby (teks kecil di atas karakter utama). Aplikasi ini sangat berguna untuk:

- Pembelajar bahasa Esperanto yang ingin memahami kata-kata melalui karakter Han
- Penerjemah dan pengajar yang perlu membuat materi dengan anotasi visual
- Peneliti linguistik yang bekerja dengan perbandingan akar kata Esperanto
- Siapa pun yang ingin membuat teks Esperanto dengan anotasi visual yang jelas

Aplikasi ini berbasis web (Streamlit) dan tersedia dalam 14 bahasa berbeda, termasuk bahasa Indonesia.

## 2. Memulai dengan Halaman Utama

Ketika Anda membuka aplikasi, Anda akan melihat halaman utama dengan judul "Mengganti Teks Esperanto dengan Karakter Kanji atau Menambahkan Anotasi HTML". Halaman ini memiliki beberapa bagian utama:

### Bagian Atas
- Judul dan deskripsi singkat
- Opsi untuk memilih berkas JSON (bawaan atau unggahan)
- Tautan untuk mengunduh contoh berkas JSON

### Bagian Tengah
- Pengaturan lanjutan (pemrosesan paralel)
- Pilihan format keluaran
- Opsi sumber teks masukan (manual atau unggahan berkas)
- Area teks untuk memasukkan teks Esperanto
- Pilihan format karakter khusus Esperanto

### Bagian Bawah
- Tombol "Kirim" dan "Batal"
- Area hasil dan pratinjau
- Tombol unduh hasil
- Tautan ke versi bahasa lain dari aplikasi

## 3. Cara Menggunakan Aplikasi

Berikut adalah langkah-langkah dasar untuk menggunakan aplikasi:

### Langkah 1: Memilih Berkas JSON
Berkas JSON berisi aturan penggantian akar kata Esperanto dengan karakter Han atau terjemahan.

- Pilih "Gunakan JSON bawaan (default)" jika Anda baru memulai
- Atau pilih "Unggah berkas" untuk menggunakan berkas JSON kustom Anda sendiri

### Langkah 2: Pilih Format Keluaran
Format keluaran menentukan bagaimana teks Esperanto dan karakter Han akan ditampilkan:

- **Format HTML dengan anotasi Ruby dan penyesuaian ukuran**: Menampilkan teks Esperanto dengan anotasi Ruby di atasnya, ukuran Ruby disesuaikan berdasarkan panjang teks
- **Format HTML dengan anotasi Ruby, penyesuaian ukuran, dan penggantian karakter Kanji**: Menampilkan karakter Han dengan teks Esperanto sebagai anotasi Ruby
- **Format HTML**: Format HTML dasar tanpa penyesuaian ukuran
- **Format HTML dengan penggantian karakter Kanji**: Format HTML dasar dengan karakter Han sebagai teks utama
- **Format dengan tanda kurung**: Menampilkan teks dalam format "teks(terjemahan)"
- **Format dengan tanda kurung dan penggantian karakter Kanji**: Menampilkan dalam format "terjemahan(teks)"
- **Hanya menyimpan teks yang telah digantikan**: Hanya menampilkan hasil penggantian tanpa teks asli

### Langkah 3: Masukkan Teks Esperanto
Ada dua cara untuk memasukkan teks:

- **Masukkan secara manual**: Ketik atau tempel teks Esperanto Anda di kotak teks
- **Unggah berkas**: Unggah berkas teks (format UTF-8) yang berisi teks Esperanto

#### Penanda Khusus dalam Teks:
Anda dapat menggunakan penanda khusus untuk mengontrol penggantian:

- **Teks dalam %...%**: Bagian teks yang dibungkus dengan tanda persen **tidak akan digantikan**
  Contoh: `Mi parolas %Esperanton% flue.` → Kata "Esperanton" tidak akan digantikan

- **Teks dalam @...@**: Bagian teks yang dibungkus dengan tanda @ akan digantikan **secara lokal**
  Contoh: `@Bonvolu@ sidi.` → Hanya kata "Bonvolu" yang akan digantikan

### Langkah 4: Pilih Format Karakter Esperanto
Ada tiga pilihan untuk karakter khusus Esperanto (ĉ, ĝ, ĥ, ĵ, ŝ, ŭ):

- **Karakter aksen di atas (ĉ → c + ˆ)**: Menggunakan karakter dengan tanda aksen
- **Format x (ĉ → cx)**: Menggunakan notasi x (cx, gx, hx, jx, sx, ux)
- **Format ^ (ĉ → c^)**: Menggunakan notasi topi (c^, g^, h^, j^, s^, u^)

### Langkah 5: Proses Teks
- Klik tombol "Kirim" untuk memproses teks
- Hasil akan ditampilkan di area bawah
- Untuk teks HTML, dua tab akan ditampilkan: "Pratinjau HTML" dan "Hasil (Kode HTML)"
- Untuk format teks lain, hasil akan ditampilkan di tab "Teks Hasil"

### Langkah 6: Unduh Hasil
- Klik tombol "Unduh Hasil" untuk menyimpan hasil sebagai berkas
- Hasil disimpan dalam format HTML atau teks sesuai dengan format keluaran yang dipilih

## 4. Format Keluaran yang Tersedia

Aplikasi menawarkan beberapa format keluaran yang dapat dipilih sesuai kebutuhan Anda:

### Format HTML dengan Anotasi Ruby
```html
<ruby>paroli<rt class="M_M">話す</rt></ruby>
```
Teks Esperanto "paroli" dengan anotasi Ruby "話す" di atasnya.

### Format HTML dengan Penggantian Karakter Kanji
```html
<ruby>話す<rt>paroli</rt></ruby>
```
Karakter Han "話す" sebagai teks utama dengan teks Esperanto "paroli" sebagai anotasi.

### Format dengan Tanda Kurung
```
paroli(話す)
```
Teks Esperanto diikuti oleh terjemahan dalam tanda kurung.

### Format dengan Tanda Kurung dan Penggantian
```
話す(paroli)
```
Terjemahan diikuti oleh teks Esperanto dalam tanda kurung.

### Hanya Teks yang Digantikan
```
話す
```
Hanya menampilkan hasil penggantian tanpa teks asli.

## 5. Membuat Berkas JSON Kustom

Aplikasi ini memungkinkan Anda membuat berkas JSON kustom untuk penggantian teks. Untuk mengakses fitur ini, klik tautan halaman "Membuat Berkas JSON untuk Menggantikan (Karakter Han) dalam Teks Esperanto" di bagian navigasi.

### Langkah-langkah Membuat Berkas JSON:

#### Langkah 1: Persiapkan Berkas CSV
- Pilih "Unggah Berkas CSV" atau "Gunakan Default"
- Berkas CSV harus berisi dua kolom: akar kata Esperanto dan terjemahan/karakter Han
- Format: `paroli,話す` (kata Esperanto, terjemahan)

#### Langkah 2: Persiapkan Berkas JSON untuk Aturan Pemecahan Akar Kata
- Pilih "Unggah Berkas JSON" atau "Gunakan Default"
- Berkas ini menentukan cara memecah akar kata Esperanto dan menerapkan akhiran

#### Langkah 3: Persiapkan Berkas JSON untuk Teks Pengganti
- Pilih "Unggah Berkas JSON" atau "Gunakan Default"
- Berkas ini menentukan teks pengganti khusus untuk kata-kata tertentu

#### Langkah 4: Pengaturan Pemrosesan Paralel
- Aktifkan pemrosesan paralel jika memproses data besar
- Tentukan jumlah proses yang berjalan bersamaan

#### Langkah 5: Buat Berkas JSON
- Klik tombol "Buat Berkas JSON untuk Penggantian"
- Tunggu proses selesai
- Unduh berkas JSON yang dihasilkan

### Contoh Berkas yang Tersedia:
Aplikasi menyediakan beberapa contoh berkas yang dapat diunduh sebagai referensi:
- Contoh CSV akar kata Esperanto dengan terjemahan bahasa Indonesia
- Contoh berkas JSON untuk aturan pemecahan akar kata
- Contoh berkas Excel dengan daftar 14 bahasa

## 6. Pengaturan Lanjutan

### Pemrosesan Paralel
Untuk teks yang sangat panjang atau berkas JSON yang besar, Anda dapat mengaktifkan pemrosesan paralel:

1. Buka bagian "Pengaturan Lanjutan (Pemrosesan Paralel)"
2. Centang kotak "Gunakan pemrosesan paralel"
3. Tentukan jumlah proses yang berjalan bersamaan (2-4)

Pemrosesan paralel dapat mempercepat proses secara signifikan untuk dataset besar, tetapi mungkin tidak memberikan keuntungan untuk teks pendek.

### Penanganan Karakter Khusus

Aplikasi ini mendukung berbagai format karakter khusus Esperanto:

- Format aksen (ĉ, ĝ, ĥ, ĵ, ŝ, ŭ)
- Format x (cx, gx, hx, jx, sx, ux)
- Format ^ (c^, g^, h^, j^, s^, u^)

Semua format ini akan dikonversi secara internal ke format yang Anda pilih pada keluaran.

## 7. Tips Penggunaan

### Memaksimalkan Hasil Terbaik
- Gunakan format HTML dengan penyesuaian ukuran untuk teks dengan anotasi yang panjang
- Gunakan penanda %...% untuk bagian yang ingin dipertahankan seperti nama diri atau istilah teknis
- Gunakan penanda @...@ untuk bagian yang ingin diganti secara independen dari konteksnya
- Untuk teks panjang, aktifkan pemrosesan paralel untuk kecepatan lebih baik

### Menyesuaikan Berkas JSON
Jika Anda ingin membuat berkas JSON kustom:
1. Unduh contoh berkas CSV dan JSON yang disediakan
2. Sesuaikan dengan editor CSV/JSON atau spreadsheet
3. Simpan dalam format UTF-8
4. Unggah berkas kustom Anda ke aplikasi

### Membuat Materi Pembelajaran
Untuk membuat materi pembelajaran dengan anotasi yang jelas:
1. Pilih format HTML dengan anotasi Ruby dan penyesuaian ukuran
2. Masukkan teks Esperanto Anda
3. Proseskan hasil
4. Unduh berkas HTML yang dihasilkan
5. Tempelkan hasilnya ke dokumen atau situs web Anda

## 8. Pertanyaan yang Sering Diajukan (FAQ)

### Bagaimana cara mengunduh berkas hasil?
Setelah memproses teks, klik tombol "Unduh Hasil" di bawah area tampilan hasil.

### Berkas apa yang didukung untuk diunggah?
Aplikasi mendukung berkas teks (.txt), CSV (.csv), dan Markdown (.md) dengan encoding UTF-8.

### Dapatkah saya menggunakan aplikasi ini secara offline?
Tidak, aplikasi ini berjalan di platform Streamlit Cloud dan memerlukan koneksi internet.

### Bagaimana jika hasil tidak sesuai dengan yang diharapkan?
- Periksa format keluaran yang Anda pilih
- Pastikan berkas JSON yang Anda gunakan berisi akar kata yang sesuai
- Periksa penanda %...% dan @...@ dalam teks Anda
- Pertimbangkan untuk membuat berkas JSON kustom dengan penggantian yang lebih sesuai

### Apakah saya bisa mengganti karakter Han dengan karakter/huruf lain?
Ya, Anda bisa membuat berkas CSV kustom dengan akar kata Esperanto dan terjemahan dalam bahasa atau huruf apa pun, tidak terbatas pada karakter Han.

### Di mana saya bisa menemukan versi aplikasi dalam bahasa lain?
Di bagian bawah halaman utama, Anda akan menemukan tautan ke versi aplikasi dalam 14 bahasa berbeda.

---

Aplikasi ini didesain untuk menjadi fleksibel dan mudah digunakan, memungkinkan Anda bekerja dengan teks Esperanto dan anotasinya dengan berbagai cara. Jika Anda memiliki pertanyaan lebih lanjut atau memerlukan bantuan, silakan merujuk ke repositori GitHub aplikasi ini yang tautannya disediakan di bagian bawah halaman aplikasi.