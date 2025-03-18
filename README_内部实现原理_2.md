# Dokumentasi Teknis: Aplikasi Penggantian Teks Esperanto dengan Karakter Kanji dan Anotasi Ruby

## Daftar Isi
1. [Arsitektur Aplikasi](#1-arsitektur-aplikasi)
2. [Komponen Utama dan Aliran Data](#2-komponen-utama-dan-aliran-data)
3. [Struktur Data Kunci](#3-struktur-data-kunci)
4. [Algoritma Penggantian Teks](#4-algoritma-penggantian-teks)
5. [Mekanisme Placeholder](#5-mekanisme-placeholder)
6. [Pemrosesan Paralel](#6-pemrosesan-paralel)
7. [Format Output](#7-format-output)
8. [Pembuatan Berkas JSON](#8-pembuatan-berkas-json)
9. [Penanganan Karakter Esperanto](#9-penanganan-karakter-esperanto)
10. [Optimasi dan Pertimbangan Teknis](#10-optimasi-dan-pertimbangan-teknis)

## 1. Arsitektur Aplikasi

Aplikasi ini menggunakan framework Streamlit untuk antarmuka pengguna dan terdiri dari beberapa komponen Python yang saling terhubung. Aplikasi ini dibangun dengan arsitektur modular, di mana fungsi-fungsi pemrosesan teks dipisahkan dari logika antarmuka pengguna.

### 1.1 Struktur File

Aplikasi terdiri dari empat file Python utama:

1. **main.py** - Halaman utama aplikasi Streamlit untuk melakukan penggantian teks
2. **Halaman untuk membuat file JSON untuk mengganti teks Esperanto dengan string (Kanji).py** - Halaman kedua aplikasi Streamlit untuk pembuatan berkas JSON aturan penggantian
3. **esp_text_replacement_module.py** - Modul berisi fungsi-fungsi untuk penggantian teks Esperanto
4. **esp_replacement_json_make_module.py** - Modul berisi fungsi-fungsi untuk membuat berkas JSON aturan penggantian

### 1.2 Diagram Alur Sederhana

```
[Input Pengguna] → main.py → esp_text_replacement_module.py → [Output Teks yang Diproses]
                 ↓
[Input Berkas CSV/JSON] → Halaman JSON → esp_replacement_json_make_module.py → [Output Berkas JSON]
```

### 1.3 Dependensi Eksternal

Aplikasi bergantung pada beberapa pustaka Python:
- `streamlit` - untuk antarmuka web
- `pandas` - untuk manipulasi data CSV
- `multiprocessing` - untuk pemrosesan paralel
- `re` - untuk operasi ekspresi reguler
- `json` - untuk pemrosesan berkas JSON

## 2. Komponen Utama dan Aliran Data

Aplikasi memiliki dua alur kerja utama yang terpisah namun saling terkait:

### 2.1 Alur Penggantian Teks (main.py)

1. Pengguna memasukkan teks Esperanto atau mengunggah berkas
2. Aplikasi memuat berkas JSON aturan penggantian (default atau diunggah)
3. Sistem memanggil fungsi `orchestrate_comprehensive_esperanto_text_replacement` dari `esp_text_replacement_module.py`
4. Teks diproses dan dikonversi sesuai format output yang dipilih
5. Hasil ditampilkan ke pengguna dan tersedia untuk diunduh

### 2.2 Alur Pembuatan Berkas JSON (Halaman JSON)

1. Pengguna mengunggah atau menggunakan berkas CSV default (akar kata Esperanto → terjemahan)
2. Pengguna mengunggah atau menggunakan berkas JSON default (aturan pemecahan akar kata)
3. Sistem memproses CSV dan JSON untuk membuat daftar penggantian
4. Daftar penggantian dioptimalkan dan dikonversi ke format JSON
5. Berkas JSON dihasilkan dan tersedia untuk diunduh

### 2.3 Komunikasi Antar Komponen

Kedua alur ini berfungsi secara independen, tetapi berkas JSON yang dihasilkan oleh alur kedua menjadi input untuk alur pertama. Ini memungkinkan pengguna membuat aturan penggantian kustom menggunakan halaman kedua, kemudian menggunakan aturan tersebut pada halaman utama untuk memproses teks.

## 3. Struktur Data Kunci

Beberapa struktur data kunci yang digunakan dalam aplikasi:

### 3.1 Daftar Penggantian (Replacements List)

Struktur data utama untuk aturan penggantian adalah tuple/list dengan format:
```python
(teks_lama, teks_baru, placeholder)
```

Contoh:
```python
("vorto", "<ruby>vorto<rt>kata</rt></ruby>", "$20987$")
```

Tiga jenis daftar penggantian utama:
1. `replacements_final_list` - untuk penggantian global
2. `replacements_list_for_2char` - untuk penggantian akar kata dua karakter
3. `replacements_list_for_localized_string` - untuk penggantian lokal (bagian yang dibungkus @)

### 3.2 Kamus Penggantian Karakter Esperanto

Beberapa kamus digunakan untuk mengkonversi karakter khusus Esperanto antara format yang berbeda:

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
```

### 3.3 Struktur Berkas JSON

Berkas JSON yang dihasilkan memiliki struktur dengan tiga kunci utama:
```json
{
  "全域替换用のリスト(列表)型配列(replacements_final_list)": [...],
  "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)": [...],
  "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)": [...]
}
```

## 4. Algoritma Penggantian Teks

Algoritma penggantian teks adalah inti dari aplikasi ini. Mari kita lihat langkah-langkah utamanya:

### 4.1 Fungsi Orkestrator Utama

Fungsi utama yang mengatur proses penggantian teks adalah `orchestrate_comprehensive_esperanto_text_replacement` di `esp_text_replacement_module.py`:

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text, 
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
):
```

### 4.2 Langkah-langkah Algoritma

1. **Persiapan Teks**:
   - Normalisasi spasi (menggunakan `unify_halfwidth_spaces`)
   - Konversi karakter Esperanto ke format seragam (menggunakan `convert_to_circumflex`)

2. **Identifikasi Bagian Khusus**:
   - Temukan teks yang dibungkus `%...%` untuk dilewati
   - Temukan teks yang dibungkus `@...@` untuk penggantian lokal

3. **Substitusi Bertahap**:
   - Ganti bagian khusus dengan placeholder sementara
   - Lakukan penggantian global menggunakan `replacements_final_list`
   - Lakukan penggantian dua karakter menggunakan `replacements_list_for_2char`
   - Ganti kembali placeholder dengan teks yang sudah diproses

4. **Pemformatan Hasil**:
   - Sesuaikan dengan format output yang dipilih (HTML, kurung, dll)
   - Tambahkan header dan footer HTML jika diperlukan

### 4.3 Fungsi Kunci `safe_replace()`

Fungsi `safe_replace()` adalah komponen penting dalam algoritma ini:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    valid_replacements = {}
    # Langkah 1: Ganti old → placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # Langkah 2: Ganti placeholder → new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

Pendekatan dua tahap ini sangat penting untuk menghindari konflik penggantian dan memastikan penggantian yang benar.

## 5. Mekanisme Placeholder

Aplikasi ini menggunakan mekanisme placeholder yang canggih untuk menghindari konflik selama proses penggantian.

### 5.1 Tujuan Placeholder

Placeholder digunakan untuk:
1. Melindungi bagian teks yang sudah diganti agar tidak diganti lagi
2. Mencegah penggantian sebagian teks yang merupakan bagian dari teks yang lebih panjang
3. Menangani bagian teks yang perlu dilewati atau diganti secara lokal

### 5.2 Jenis Placeholder

Aplikasi menggunakan tiga set placeholder yang berbeda:
1. `placeholders_for_skipping_replacements` - untuk bagian yang dibungkus dengan %
2. `placeholders_for_localized_replacement` - untuk bagian yang dibungkus dengan @
3. `imported_placeholders_for_global_replacement` - untuk penggantian global
4. `imported_placeholders_for_2char_replacement` - untuk penggantian akar kata dua karakter

### 5.3 Implementasi

Placeholder disimpan dalam berkas teks terpisah dan dimuat saat aplikasi dimulai:
```python
imported_placeholders_for_global_replacement = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_$20987$-$499999$_全域替换用.txt'
)
```

Fungsi `import_placeholders()` membaca placeholder dari berkas teks:
```python
def import_placeholders(filename: str) -> List[str]:
    with open(filename, 'r') as file:
        placeholders = [line.strip() for line in file if line.strip()]
    return placeholders
```

## 6. Pemrosesan Paralel

Untuk meningkatkan kinerja dengan teks panjang, aplikasi ini menggunakan pemrosesan paralel melalui modul `multiprocessing` Python.

### 6.1 Implementasi di Main.py

Fungsi utama untuk pemrosesan paralel adalah `parallel_process()`:

```python
def parallel_process(
    text: str,
    num_processes: int,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type: str
) -> str:
```

### 6.2 Algoritma Pemrosesan Paralel

1. Teks input dibagi menjadi beberapa segmen berdasarkan jumlah proses
2. Setiap segmen diproses secara paralel oleh fungsi `process_segment()`
3. Hasil dari semua segmen digabungkan kembali
4. Pendekatan ini secara signifikan meningkatkan kinerja untuk teks panjang

### 6.3 Pertimbangan Khusus untuk Streamlit

Streamlit bekerja dengan model proses yang unik, sehingga diperlukan pengaturan khusus untuk `multiprocessing`:

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # Abaikan jika metode start sudah ditetapkan
```

Ini memastikan bahwa pemrosesan paralel bekerja dengan benar dalam konteks Streamlit.

## 7. Format Output

Aplikasi mendukung berbagai format output untuk teks yang diproses.

### 7.1 Fungsi `output_format()`

Format output ditentukan oleh fungsi `output_format()` di `esp_replacement_json_make_module.py`:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
```

### 7.2 Format yang Didukung

1. **HTML dengan Anotasi Ruby dan Penyesuaian Ukuran**:
   ```html
   <ruby>teks<rt class="S_S">anotasi</rt></ruby>
   ```

2. **HTML dengan Penggantian Karakter dan Anotasi Ruby**:
   ```html
   <ruby>漢字<rt class="S_S">teks</rt></ruby>
   ```

3. **Format HTML Dasar**:
   ```html
   <ruby>teks<rt>anotasi</rt></ruby>
   ```

4. **Format Kurung**:
   ```
   teks(anotasi)
   ```

5. **Format Kurung dengan Penggantian**:
   ```
   漢字(teks)
   ```

6. **Hanya Teks Pengganti**:
   ```
   漢字
   ```

### 7.3 Penyesuaian Ukuran Anotasi Ruby

Salah satu fitur canggih adalah penyesuaian ukuran anotasi Ruby berdasarkan rasio panjang teks:

```python
if ratio_1 > 6:
    return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
elif ratio_1 > (9/3):
    return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
# ...dan seterusnya
```

## 8. Pembuatan Berkas JSON

Proses pembuatan berkas JSON adalah proses kompleks yang melibatkan beberapa langkah.

### 8.1 Sumber Data

Berkas JSON dibuat dari beberapa sumber:
1. Berkas CSV dengan pasangan (akar kata Esperanto, terjemahan)
2. Berkas JSON dengan aturan pemecahan akar kata
3. Definisi khusus untuk akhiran dan awalan kata

### 8.2 Langkah Pemrosesan Utama

1. Baca data CSV dan konversi ke format kamus
2. Tambahkan informasi prioritas penggantian berdasarkan panjang kata
3. Proses kata dengan akhiran khusus (misalnya `-an`, `-on`, akhiran kata kerja)
4. Terapkan aturan pemecahan akar kata kustom
5. Buat tiga daftar penggantian terpisah untuk kebutuhan berbeda
6. Tambahkan variasi kapitalisasi untuk setiap entri

### 8.3 Perhitungan Prioritas

Prioritas penggantian dihitung terutama berdasarkan panjang kata:

```python
replacement_priority_by_length = len(esperanto_Word_before_replacement) * 10000
```

Kata yang lebih panjang diberi prioritas lebih tinggi untuk memastikan penggantian yang tepat.

## 9. Penanganan Karakter Esperanto

Aplikasi menangani berbagai format karakter Esperanto dengan fleksibel.

### 9.1 Format Karakter yang Didukung

1. **Format Circumflex (ĉ, ĝ, ĥ, ĵ, ŝ, ŭ)** - karakter dengan tanda di atas
2. **Format-X (cx, gx, hx, jx, sx, ux)** - format yang umum digunakan ketika karakter circumflex tidak tersedia
3. **Format-Topi (c^, g^, h^, j^, s^, u^)** - format alternatif

### 9.2 Konversi Format

Fungsi `replace_esperanto_chars()` digunakan untuk konversi format:

```python
def replace_esperanto_chars(text, char_dict: Dict[str, str]) -> str:
    for original_char, converted_char in char_dict.items():
        text = text.replace(original_char, converted_char)
    return text
```

### 9.3 Normalisasi Input

Sebelum pemrosesan, teks input dinormalisasi:
1. Mengkonversi semua karakter Esperanto ke format circumflex
2. Menormalkan spasi dengan `unify_halfwidth_spaces()`
3. Kemudian mengkonversi ke format output yang diinginkan setelah pemrosesan

## 10. Optimasi dan Pertimbangan Teknis

Aplikasi ini menggunakan beberapa teknik optimasi untuk meningkatkan kinerja dan ketangguhan.

### 10.1 Caching dengan Streamlit

Aplikasi menggunakan dekorator `@st.cache_data` untuk meng-cache hasil pemuatan berkas JSON:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
```

Ini menghindari pemuatan berulang kali dari berkas JSON yang besar.

### 10.2 Optimasi untuk Teks Panjang

Untuk teks panjang, aplikasi:
1. Hanya menampilkan pratinjau (250 baris pertama) untuk menghemat memori
2. Menawarkan pemrosesan paralel untuk meningkatkan kecepatan
3. Memungkinkan pengguna mengunduh hasil lengkap

### 10.3 Penanganan Kesalahan

Aplikasi mencakup penanganan kesalahan yang komprehensif:
1. Pengecekan berkas yang diunggah
2. Penanganan format karakter yang tidak valid
3. Penanganan kesalahan selama pemrosesan paralel
4. Validasi struktur berkas JSON

### 10.4 Keterbatasan dan Kasus Tepi

Beberapa keterbatasan dan kasus tepi yang perlu diperhatikan:
1. Jumlah maksimum karakter dalam bagian `%...%` dibatasi hingga 50
2. Jumlah maksimum karakter dalam bagian `@...@` dibatasi hingga 18
3. Prioritas penggantian perlu dikelola dengan hati-hati untuk menghindari konflik
4. Berkas JSON yang sangat besar dapat menyebabkan masalah kinerja

---

Dalam dokumen teknis ini, kami telah menjelajahi aspek utama dari aplikasi penggantian teks Esperanto dengan karakter Kanji dan anotasi Ruby. Pemahaman yang mendalam tentang komponen-komponen ini akan memungkinkan pengembang untuk memahami, memodifikasi, atau memperluas aplikasi sesuai kebutuhan mereka.