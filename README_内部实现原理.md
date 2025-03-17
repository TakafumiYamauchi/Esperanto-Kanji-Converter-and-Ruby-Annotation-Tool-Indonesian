# Dokumentasi Teknis Aplikasi Penggantian Teks Esperanto dan Anotasi Ruby

## Daftar Isi
1. [Arsitektur Sistem](#arsitektur-sistem)
2. [Alur Data dan Proses](#alur-data-dan-proses)
3. [Komponen Utama](#komponen-utama)
4. [Mekanisme Penggantian Teks](#mekanisme-penggantian-teks)
5. [Struktur File JSON](#struktur-file-json)
6. [Pemrosesan Paralel](#pemrosesan-paralel)
7. [Algoritma Penyesuaian Ruby](#algoritma-penyesuaian-ruby)
8. [Strategi Pemecahan Akar Kata](#strategi-pemecahan-akar-kata)
9. [Optimisasi dan Caching](#optimisasi-dan-caching)
10. [Pola Desain dan Praktik Terbaik](#pola-desain-dan-praktik-terbaik)

## Arsitektur Sistem

Aplikasi ini dibangun menggunakan kerangka kerja Streamlit, yang memungkinkan pengembangan antarmuka web interaktif menggunakan Python. Aplikasi dibagi menjadi empat file utama yang saling terkait:

### 1. `main.py`
File utama yang menyediakan antarmuka pengguna dan mengatur alur eksekusi aplikasi. Ini berfungsi sebagai titik masuk aplikasi dan mengoordinasikan interaksi antar modul.

### 2. `Halaman untuk membuat file JSON untuk mengganti teks Esperanto dengan string (Kanji).py`
File halaman tambahan yang ditempatkan di folder `pages/` Streamlit. Halaman ini menyediakan fungsionalitas untuk membuat file JSON kustom berisi aturan penggantian.

### 3. `esp_text_replacement_module.py`
Modul yang berisi fungsi-fungsi utama untuk penggantian teks Esperanto, termasuk:
- Konversi karakter khusus Esperanto
- Fungsi penggantian safe_replace
- Penanganan placeholder
- Fungsi orkestrator untuk menjalankan proses penggantian menyeluruh
- Fungsi pemrosesan paralel

### 4. `esp_replacement_json_make_module.py`
Modul yang menyediakan fungsi-fungsi untuk membuat file JSON penggantian, meliputi:
- Pengukuran lebar teks
- Format output berbeda (HTML, kurung, dll)
- Fungsi pembantu untuk pembuatan file JSON
- Fungsi paralel untuk pembuatan kamus penggantian

Arsitektur ini mengikuti prinsip modularitas, dengan memisahkan logika bisnis (fungsi penggantian teks) dari antarmuka pengguna (Streamlit UI).

## Alur Data dan Proses

Aplikasi memiliki dua alur utama:

### Alur 1: Penggantian Teks (di `main.py`)

1. **Input**:
   - File JSON berisi aturan penggantian (default atau diunggah)
   - Teks Esperanto (dimasukkan manual atau diunggah)
   - Preferensi format output dan karakter Esperanto

2. **Pemrosesan**:
   - Memuat file JSON ke dalam tiga daftar penggantian utama
   - Menormalisasi teks input (konversi karakter Esperanto, normalisasi spasi)
   - Mengidentifikasi bagian-bagian yang dibungkus `%..%` dan `@..@`
   - Menerapkan penggantian global, lokal, dan penggantian akar kata 2 karakter
   - Memformat hasil sesuai pilihan user (HTML dengan Ruby, kurung, dll)

3. **Output**:
   - Menampilkan hasil dalam pratinjau yang sesuai (HTML, teks biasa)
   - Menyediakan tombol unduh untuk hasil

### Alur 2: Pembuatan File JSON (di halaman kedua)

1. **Input**:
   - File CSV berisi pasangan (akar kata Esperanto → terjemahan/Kanji)
   - File JSON untuk aturan pemecahan akar kata
   - File JSON untuk pengaturan teks pengganti khusus

2. **Pemrosesan**:
   - Mengimpor data dari CSV dan JSON
   - Menerapkan aturan pemecahan akar kata
   - Membangun daftar penggantian dengan prioritas yang tepat
   - Menggabungkan tiga jenis daftar penggantian

3. **Output**:
   - File JSON gabungan yang siap digunakan di halaman utama

## Komponen Utama

### 1. Sistem Penggantian Teks

Jantung aplikasi ini adalah sistem penggantian teks yang kompleks, yang menggunakan pendekatan "safe_replace" dengan placeholder untuk menghindari konflik penggantian. Sistem ini diterapkan melalui beberapa fungsi:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    """
    Melakukan penggantian teks aman menggunakan placeholder sebagai perantara.
    replacements adalah daftar tuple (old, new, placeholder).
    """
    valid_replacements = {}
    # Langkah 1: old → placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # Langkah 2: placeholder → new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

Fungsi orkestrator `orchestrate_comprehensive_esperanto_text_replacement()` mengoordinasikan seluruh proses penggantian melalui beberapa tahap:

1. Normalisasi teks dan konversi karakter Esperanto
2. Menangani bagian teks yang dibungkus `%..%` (untuk dilewati)
3. Menangani bagian teks yang dibungkus `@..@` (untuk penggantian lokal)
4. Melakukan penggantian global
5. Melakukan penggantian akar kata 2 karakter
6. Mengembalikan placeholder ke bentuk akhirnya
7. Menerapkan format HTML jika diperlukan

### 2. Sistem Konversi Karakter Esperanto

Aplikasi ini mendukung berbagai format karakter Esperanto dan menyediakan fungsi konversi antar format:

```python
x_to_circumflex = {
    'cx': 'ĉ', 'gx': 'ĝ', 'hx': 'ĥ', 'jx': 'ĵ', 'sx': 'ŝ', 'ux': 'ŭ',
    'Cx': 'Ĉ', 'Gx': 'Ĝ', 'Hx': 'Ĥ', 'Jx': 'Ĵ', 'Sx': 'Ŝ', 'Ux': 'Ŭ'
}
```

Berbagai kamus konversi mendukung transformasi antara:
- Format x-system (cx, gx, ...)
- Format sirkumfleks (ĉ, ĝ, ...)
- Format topi (c^, g^, ...)

### 3. Sistem Anotasi Ruby dan Format Output

Aplikasi menyediakan beberapa format output, dengan anotasi Ruby HTML sebagai fitur unggulan. Fungsi `output_format()` menangani konversi ke berbagai format output:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    """
    Menggabungkan teks utama dan konten ruby sesuai format yang dipilih.
    """
    if format_type == 'HTML格式_Ruby文字_大小调整':
        # Kode untuk ruby dengan penyesuaian ukuran
        ...
    elif format_type == 'HTML格式':
        return f'<ruby>{main_text}<rt>{ruby_content}</rt></ruby>'
    elif format_type == '括弧(号)格式':
        return f'{main_text}({ruby_content})'
    # ... format lainnya
```

## Mekanisme Penggantian Teks

Mekanisme penggantian teks dalam aplikasi ini menggunakan beberapa prinsip cerdas:

### 1. Pendekatan Placeholder

Aplikasi menggunakan placeholder unik sebagai "penanda tempat" sementara untuk menghindari konflik selama penggantian berurutan. Proses ini terdiri dari dua tahap:
- **Tahap 1**: Ganti teks asli dengan placeholder unik
- **Tahap 2**: Ganti placeholder dengan teks final

Pendekatan ini mengatasi masalah umum dalam penggantian string berurutan, di mana penggantian sebelumnya dapat memengaruhi penggantian berikutnya.

### 2. Tiga Jenis Daftar Penggantian

Aplikasi menggunakan tiga jenis daftar penggantian yang berbeda:

1. **replacements_final_list**: Untuk penggantian global di seluruh teks
2. **replacements_list_for_localized_string**: Untuk penggantian lokal pada bagian yang dibungkus `@..@`
3. **replacements_list_for_2char**: Untuk penggantian akar kata 2 karakter

Setiap daftar berisi tuple (old, new, placeholder) untuk digunakan dalam safe_replace.

### 3. Penanganan Khusus

Mekanisme penggantian juga menangani kasus khusus:

- **Bagian yang Dilewati**: Bagian teks yang dibungkus `%..%` (hingga 50 karakter) tidak akan diganti
- **Penggantian Lokal**: Bagian teks yang dibungkus `@..@` (hingga 18 karakter) akan diganti menggunakan daftar lokal
- **Kapitalisasi**: Aplikasi secara otomatis menangani penggantian untuk versi kapitalisasi dan huruf besar dari setiap kata

### 4. Algoritma Penggantian Teks Menyeluruh

Fungsi `orchestrate_comprehensive_esperanto_text_replacement()` adalah pusat mekanisme penggantian, yang melakukan:

1. Normalisasi spasi dan konversi karakter Esperanto
2. Identifikasi dan penanganan bagian dengan `%..%`
3. Identifikasi dan penanganan bagian dengan `@..@`
4. Penggantian global menggunakan `replacements_final_list`
5. Penggantian akar kata 2 karakter (dua kali berurutan)
6. Mengembalikan semua placeholder ke bentuk akhirnya
7. Pemformatan HTML jika perlu

Pendekatan bertahap ini memastikan penggantian yang tepat dan konsisten.

## Struktur File JSON

File JSON yang digunakan dalam aplikasi memiliki struktur khusus untuk mendukung mekanisme penggantian yang kompleks.

### 1. File JSON Utama (Gabungan Tiga Daftar)

File JSON utama mengandung tiga kunci utama:

```json
{
  "全域替换用のリスト(列表)型配列(replacements_final_list)": [
    [string_asli, string_pengganti, placeholder],
    ...
  ],
  "局部文字替换用のリスト(列表)型配列(replacements_list_for_localized_string)": [
    [string_asli, string_pengganti, placeholder],
    ...
  ],
  "二文字词根替换用のリスト(列表)型配列(replacements_list_for_2char)": [
    [string_asli, string_pengganti, placeholder],
    ...
  ]
}
```

Setiap daftar berisi array tuple yang mewakili aturan penggantian.

### 2. File JSON Pemecahan Akar Kata

File JSON kedua (opsional) menentukan cara memecah akar kata Esperanto:

```json
[
  ["am", "dflt", ["verbo_s1"]],
  ["ir", "50000", ["verbo_s1", "verbo_s2"]],
  ...
]
```

Setiap item berisi:
- Akar kata Esperanto
- Prioritas penggantian ("dflt" atau nilai numerik)
- Daftar kode-kode khusus yang menentukan cara memecah dan memperluas akar kata:
  - "verbo_s1": Tambahkan akhiran kata kerja (as, is, os, dll)
  - "verbo_s2": Tambahkan akhiran kata kerja dasar (i, u)
  - "ne": Tidak lakukan perluasan khusus
  - Akhiran tambahan lainnya

### 3. File JSON Penggantian Teks

File JSON ketiga (opsional) menentukan penggantian khusus:

```json
[
  ["ami/k", "90000", ["verbo_s1"], "爱/友"],
  ...
]
```

Setiap item berisi:
- String Esperanto dengan pemisah "/" opsional
- Prioritas penggantian
- Daftar kode perluasan
- String pengganti dengan pemisah "/" opsional

## Pemrosesan Paralel

Aplikasi mendukung pemrosesan paralel untuk meningkatkan performa saat menangani teks dan daftar penggantian besar.

### 1. Paralelisasi Penggantian Teks

Untuk teks panjang, aplikasi memecah teks menjadi beberapa segmen dan memprosesnya secara paralel:

```python
def parallel_process(
    text: str,
    num_processes: int,
    # parameter lainnya...
) -> str:
    # Pecah teks menjadi beberapa baris
    lines = re.findall(r'.*?\n|.+$', text)
    # Bagi baris-baris untuk num_processes worker
    # ...
    # Gunakan multiprocessing.Pool untuk pemrosesan paralel
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(process_segment, [...])
    # Gabungkan hasil
    return ''.join(results)
```

Pendekatan ini sangat meningkatkan performa pada teks yang panjang, terutama saat menggunakan daftar penggantian yang besar.

### 2. Paralelisasi Pembuatan JSON

Saat membuat file JSON penggantian, aplikasi juga menggunakan pemrosesan paralel untuk mempercepat pemrosesan daftar besar:

```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
    # Bagi data menjadi beberapa potongan
    # ...
    # Proses setiap potongan secara paralel
    with multiprocessing.Pool(num_processes) as pool:
        partial_dicts = pool.starmap(process_chunk_for_pre_replacements, [...])
    # Gabungkan hasil dari semua worker
    merged_dict = {}
    for partial_d in partial_dicts:
        # Gabungkan partial_d ke merged_dict
        # ...
    return merged_dict
```

### 3. Pengaturan Start Method Multiprocessing

Aplikasi mengatur metode start multiprocessing secara eksplisit untuk menghindari error Pickling:

```python
try:
    multiprocessing.set_start_method("spawn")
except RuntimeError:
    pass  # Abaikan jika metode start sudah diatur
```

Ini penting untuk kompatibilitas dengan Streamlit dan sistem operasi yang berbeda.

## Algoritma Penyesuaian Ruby

Aplikasi menggunakan algoritma cerdas untuk menyesuaikan ukuran teks Ruby berdasarkan rasio panjang teks Ruby dan teks utama.

### 1. Pengukuran Lebar Teks

Aplikasi menggunakan kamus lebar karakter untuk mengukur lebar teks dengan akurat:

```python
def measure_text_width_Arial16(text, char_widths_dict: Dict[str, int]) -> int:
    """
    Menghitung lebar total teks berdasarkan kamus lebar karakter.
    """
    total_width = 0
    for ch in text:
        char_width = char_widths_dict.get(ch, 8)
        total_width += char_width
    return total_width
```

Kamus ini berasal dari file `Unicode_BMP全范围文字幅(宽)_Arial16.json`.

### 2. Penyesuaian Ukuran Ruby

Berdasarkan rasio lebar Ruby dan teks utama, aplikasi menentukan ukuran Ruby yang optimal:

```python
if format_type == 'HTML格式_Ruby文字_大小调整':
    width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
    width_main = measure_text_width_Arial16(main_text, char_widths_dict)
    ratio_1 = width_ruby / width_main

    if ratio_1 > 6:
        # Ruby sangat panjang, gunakan ukuran sangat kecil dan bagi menjadi 3 baris
        return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
    elif ratio_1 > (9/3):
        # Ruby panjang, gunakan ukuran kecil dan bagi menjadi 2 baris
        return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
    elif ratio_1 > (9/4):
        return f'<ruby>{main_text}<rt class="XS_S">{ruby_content}</rt></ruby>'
    # ... dan seterusnya untuk rasio yang berbeda
```

### 3. Pembagian Teks Ruby

Untuk Ruby yang sangat panjang, aplikasi membaginya menjadi beberapa baris menggunakan tag `<br>`:

```python
def insert_br_at_half_width(text, char_widths_dict: Dict[str, int]) -> str:
    """
    Masukkan <br> di tengah teks berdasarkan lebar karakter.
    """
    total_width = measure_text_width_Arial16(text, char_widths_dict)
    half_width = total_width / 2
    current_width = 0
    insert_index = None

    # Cari titik tengah berdasarkan lebar
    for i, ch in enumerate(text):
        char_width = char_widths_dict.get(ch, 8)
        current_width += char_width
        if current_width >= half_width:
            insert_index = i + 1
            break

    # Sisipkan <br> pada indeks yang ditemukan
    if insert_index is not None:
        result = text[:insert_index] + "<br>" + text[insert_index:]
    else:
        result = text
    return result
```

Pendekatan ini memastikan teks Ruby tetap terbaca bahkan saat sangat panjang.

## Strategi Pemecahan Akar Kata

Aplikasi menggunakan strategi khusus untuk memecah akar kata Esperanto.

### 1. Identifikasi Akar Kata dan Akhiran

Esperanto adalah bahasa aglutinasi dengan struktur morfologi yang teratur:
- Akar kata + akhiran gramatikal (o, a, e, i, as, is, os, us, ...)
- Prefiks dan sufiks derivasional (re-, mal-, -ig, -iĝ, ...)

Aplikasi memiliki daftar akar kata Esperanto yang disimpan di file eksternal dan diproses melalui parameter `E_stem_with_Part_Of_Speech_list`.

### 2. Penanganan Khusus untuk Akhiran Umum

Aplikasi secara khusus menangani berbagai jenis akhiran:

```python
# Akhiran kata kerja
verb_suffix_2l = {
    'as':'as', 'is':'is', 'os':'os', 'us':'us','at':'at','it':'it','ot':'ot',
    'ad':'ad','iĝ':'iĝ','ig':'ig','ant':'ant','int':'int','ont':'ont'
}

# Daftar kata dengan akhiran "an" dan "on"
AN = [['dietan', '/diet/an/', '/diet/an'], ['afrikan', '/afrik/an/', '/afrik/an'], ...]
ON = [['duon', '/du/on/', '/du/on'], ['okon', '/ok/on/', '/ok/on'], ...]

# Akar kata 2 karakter
suffix_2char_roots = ['ad', 'ag', 'am', 'ar', 'as', 'at', ...]
prefix_2char_roots = ['al', 'am', 'av', 'bo', 'di', 'du', ...]
standalone_2char_roots = ['al', 'ci', 'da', 'de', 'di', 'do', ...]
```

### 3. Prioritas Penggantian Berdasarkan Panjang

Aplikasi menetapkan prioritas penggantian berdasarkan panjang kata, dengan preferensi pada kata yang lebih panjang:

```python
replacement_priority_by_length = len(esperanto_Word_before_replacement) * 10000
```

Pendekatan ini memastikan bahwa kata-kata lengkap diganti terlebih dahulu, sebelum mencoba mengganti bagian-bagiannya.

### 4. Ekspansi Akar Kata

Saat membuat file JSON, aplikasi memperluas akar kata dengan berbagai akhiran:

```python
# Untuk kata kerja
if "verbo_s1" in i[2]:
    for k1, k2 in verb_suffix_2l_2.items():
        pre_replacements_dict_3[esperanto_Word_before_replacement + k1] = [
            Replaced_String + k2,
            replacement_priority_by_length + len(k1) * 10000
        ]
    i[2].remove("verbo_s1")
```

Ini menghasilkan banyak entri penggantian dari satu aturan, meningkatkan jangkauan penggantian.

## Optimisasi dan Caching

Aplikasi menggunakan beberapa teknik optimisasi untuk meningkatkan performa.

### 1. Streamlit Caching

Fungsi pembacaan file JSON menggunakan dekorator `@st.cache_data` untuk caching hasil:

```python
@st.cache_data
def load_replacements_lists(json_path: str) -> Tuple[List, List, List]:
    """
    Memuat file JSON dan me-return tiga daftar penggantian.
    Hasil di-cache untuk mempercepat pemuatan file JSON besar.
    """
    with open(json_path, 'r', encoding='utf-8') as f:
        data = json.load(f)
    # ... ekstrak dan return daftar
```

Ini secara signifikan meningkatkan performa saat menangani file JSON besar (50MB+).

### 2. Pemrosesan Paralel

Seperti dijelaskan sebelumnya, aplikasi menggunakan multiprocessing untuk memparalelkan:
- Penggantian teks untuk teks panjang
- Pembuatan kamus penggantian dari dataset besar

### 3. Penggunaan Placeholder

Penggunaan placeholder unik adalah optimisasi cerdas yang:
- Menghindari konflik selama penggantian
- Memungkinkan penggantian dua arah yang aman
- Mengurangi kemungkinan error pada pola yang kompleks

### 4. Prioritas Penggantian

Aplikasi menggunakan sistem prioritas penggantian yang memastikan penggantian paling spesifik dilakukan terlebih dahulu:

```python
# Urutkan berdasarkan prioritas (lebih tinggi dulu)
pre_replacements_list_2 = sorted(pre_replacements_list_1, key=lambda x: x[2], reverse=True)
```

Ini meningkatkan akurasi dan konsistensi penggantian.

## Pola Desain dan Praktik Terbaik

Aplikasi menunjukkan beberapa pola desain dan praktik terbaik yang baik untuk dipelajari.

### 1. Pemisahan Kekhawatiran (Separation of Concerns)

Kode dibagi menjadi modul berbeda dengan tanggung jawab yang terdefinisi dengan baik:
- `main.py`: Antarmuka pengguna Streamlit
- Halaman JSON: Pembuatan file penggantian
- Modul penggantian: Logika inti penggantian teks
- Modul pembuatan JSON: Logika pembuatan daftar penggantian

### 2. Komposisi Fungsi

Fungsi kompleks dibangun melalui komposisi fungsi yang lebih kecil dan lebih sederhana:

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
    # Panggil berbagai fungsi yang lebih kecil dalam urutan tertentu
    text = unify_halfwidth_spaces(text)
    text = convert_to_circumflex(text)
    # ...dan seterusnya
```

### 3. Type Hinting

Penggunaan type hinting Python untuk meningkatkan kejelasan dan keamanan tipe:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    # implementasi
```

### 4. Penanganan Error

Aplikasi menangani kemungkinan error, terutama pada operasi I/O:

```python
try:
    (replacements_final_list,
     replacements_list_for_localized_string,
     replacements_list_for_2char) = load_replacements_lists(default_json_path)
    st.success("Berkas JSON bawaan berhasil dimuat.")
except Exception as e:
    st.error(f"Gagal memuat berkas JSON bawaan: {e}")
    st.stop()
```

### 5. Paralelisasi yang Efisien

Aplikasi menggunakan paralelisasi secara selektif, hanya ketika bermanfaat:

```python
if num_processes <= 1 or num_lines <= 1:
    # Gunakan pemrosesan sekuensial untuk kasus sederhana
    return orchestrate_comprehensive_esperanto_text_replacement(...)
else:
    # Gunakan pemrosesan paralel untuk kasus kompleks
    # ...
```

Dalam bagian selanjutnya, kita akan melihat implementasi khusus untuk fitur-fitur utama aplikasi dan bagaimana mereka berinteraksi untuk membentuk sistem penggantian teks yang kuat.

# Dokumentasi Teknis Aplikasi Penggantian Teks Esperanto dan Anotasi Ruby (Lanjutan)

## 11. Analisis Implementasi Fungsi Utama

Mari kita membahas lebih dalam implementasi beberapa fungsi kunci yang menjadi inti aplikasi ini.

### Safe Replace: Jantung Mekanisme Penggantian

Fungsi `safe_replace()` adalah fondasi dari seluruh sistem penggantian teks:

```python
def safe_replace(text: str, replacements: List[Tuple[str, str, str]]) -> str:
    valid_replacements = {}
    # Tahap 1: old → placeholder
    for old, new, placeholder in replacements:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new
    # Tahap 2: placeholder → new
    for placeholder, new in valid_replacements.items():
        text = text.replace(placeholder, new)
    return text
```

Keunggulan pendekatan ini:

1. **Penghindaran Konflik**: Penggantian langsung dari "abc" ke "xyz" dapat mengganggu penggantian lain. Misalnya, jika ada aturan penggantian "ab" → "pq" dan "bc" → "rs", secara langsung akan menghasilkan "pqc" atau "abrs" tergantung urutan, bukan "pqrs" yang diinginkan. Dengan pendekatan placeholder, hasilnya selalu konsisten.

2. **Efisiensi**: Hanya melakukan penggantian untuk string yang benar-benar ada dalam teks, meminimalisir operasi string yang tidak perlu.

3. **Kejelasan**: Memisahkan proses penggantian menjadi dua langkah yang jelas, memudahkan debugging dan pemeliharaan.

### Orkestrator Penggantian Komprehensif

Fungsi `orchestrate_comprehensive_esperanto_text_replacement()` mengoordinasikan seluruh proses penggantian:

```python
def orchestrate_comprehensive_esperanto_text_replacement(
    text,
    placeholders_for_skipping_replacements,
    replacements_list_for_localized_string,
    placeholders_for_localized_replacement,
    replacements_final_list,
    replacements_list_for_2char,
    format_type
) -> str:
    # 1, 2) Normalisasi spasi + konversi karakter Esperanto
    text = unify_halfwidth_spaces(text)
    text = convert_to_circumflex(text)

    # 3) Proses teks dengan %...% (bagian yang dilewati)
    replacements_list_for_intact_parts = create_replacements_list_for_intact_parts(
        text, placeholders_for_skipping_replacements)
    sorted_replacements_list_for_intact_parts = sorted(
        replacements_list_for_intact_parts, key=lambda x: len(x[0]), reverse=True)
    for original, place_holder_ in sorted_replacements_list_for_intact_parts:
        text = text.replace(original, place_holder_)

    # 4) Proses teks dengan @...@ (penggantian lokal)
    tmp_replacements_list_for_localized_string_2 = create_replacements_list_for_localized_replacement(
        text, placeholders_for_localized_replacement, replacements_list_for_localized_string)
    sorted_replacements_list_for_localized_string = sorted(
        tmp_replacements_list_for_localized_string_2, key=lambda x: len(x[0]), reverse=True)
    for original, place_holder_, replaced_original in sorted_replacements_list_for_localized_string:
        text = text.replace(original, place_holder_)

    # 5) Penggantian global dengan safe_replace (implementasi inline)
    valid_replacements = {}
    for old, new, placeholder in replacements_final_list:
        if old in text:
            text = text.replace(old, placeholder)
            valid_replacements[placeholder] = new

    # 6) Penggantian akar kata 2 karakter (dilakukan dua kali)
    # ... (kode untuk penggantian akar kata 2 karakter)

    # 7) Kembalikan placeholder ke bentuk akhir
    # ... (kode untuk mengembalikan placeholder)

    # 8) Penyesuaian format HTML jika perlu
    if "HTML" in format_type:
        text = text.replace("\n", "<br>\n")
        text = re.sub(r"   ", "&nbsp;&nbsp;&nbsp;", text)
        text = re.sub(r"  ", "&nbsp;&nbsp;", text)

    return text
```

Pendekatan bertahap ini memungkinkan:

1. **Penanganan Kasus Khusus**: Penanganan terpisah untuk bagian teks yang perlu dilewati (%...%) dan bagian dengan penggantian lokal (@...@).

2. **Pemrosesan Berurutan yang Tepat**: Urutan pemrosesan sangat penting untuk hasil yang benar. Aplikasi ini memproses teks dengan cara yang memastikan penggantian tidak saling mengganggu.

3. **Fleksibilitas Format**: Penyesuaian format akhir (HTML, kurung, dll) dilakukan setelah semua penggantian selesai.

## 12. Sistem Placeholder: Detail Implementasi

Sistem placeholder adalah kunci dari kehandalan aplikasi. Mari kita lihat detailnya:

### 1. Sumber Placeholder

Aplikasi menggunakan tiga set placeholder yang berbeda, yang dimuat dari file eksternal:

```python
imported_placeholders_for_global_replacement = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_$20987$-$499999$_全域替换用.txt'
)
imported_placeholders_for_2char_replacement = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_$13246$-$19834$_二文字词根替换用.txt'
)
imported_placeholders_for_local_replacement = import_placeholders(
    './Appの运行に使用する各类文件/占位符(placeholders)_@20374@-@97648@_局部文字列替换用.txt'
)
```

File-file ini berisi ribuan string placeholder unik (seperti `$25874$`, `@35971@`, dll), memastikan tidak ada konflik.

### 2. Penanganan Placeholder untuk Bagian yang Dilewati (%...%)

Untuk bagian yang perlu dipertahankan apa adanya:

```python
def find_percent_enclosed_strings_for_skipping_replacement(text: str) -> List[str]:
    """Ekstrak teks dalam format '%xxx%' (maksimum 50 karakter)."""
    matches = []
    used_indices = set()
    for match in PERCENT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))
    return matches

def create_replacements_list_for_intact_parts(text: str, placeholders: List[str]) -> List[Tuple[str, str]]:
    """Buat daftar penggantian untuk bagian yang dilewati (%xxx%)."""
    matches = find_percent_enclosed_strings_for_skipping_replacement(text)
    replacements_list_for_intact_parts = []
    for i, match in enumerate(matches):
        if i < len(placeholders):
            replacements_list_for_intact_parts.append([f"%{match}%", placeholders[i]])
        else:
            break
    return replacements_list_for_intact_parts
```

Pendekatan ini:
- Mengidentifikasi semua teks yang dibungkus `%...%`
- Mengganti setiap bagian dengan placeholder unik
- Mencatat pasangan bagian asli dan placeholder untuk pemulihan nanti

### 3. Penanganan Placeholder untuk Penggantian Lokal (@...@)

Untuk bagian yang memerlukan penggantian lokal:

```python
def find_at_enclosed_strings_for_localized_replacement(text: str) -> List[str]:
    """Ekstrak teks dalam format '@xxx@' (maksimum 18 karakter)."""
    matches = []
    used_indices = set()
    for match in AT_PATTERN.finditer(text):
        start, end = match.span()
        if start not in used_indices and end-2 not in used_indices:
            matches.append(match.group(1))
            used_indices.update(range(start, end))
    return matches

def create_replacements_list_for_localized_replacement(
    text, placeholders: List[str],
    replacements_list_for_localized_string: List[Tuple[str, str, str]]
) -> List[List[str]]:
    """Buat daftar penggantian untuk bagian dengan penggantian lokal (@xxx@)."""
    matches = find_at_enclosed_strings_for_localized_replacement(text)
    tmp_list = []
    for i, match in enumerate(matches):
        if i < len(placeholders):
            replaced_match = safe_replace(match, replacements_list_for_localized_string)
            tmp_list.append([f"@{match}@", placeholders[i], replaced_match])
        else:
            break
    return tmp_list
```

Pendekatan ini:
- Mengidentifikasi semua teks yang dibungkus `@...@`
- Menerapkan penggantian lokal pada teks tersebut
- Menyimpan teks asli, placeholder, dan hasil penggantian untuk pemulihan nanti

## 13. Algoritma Pemecahan Akar Kata Esperanto

Esperanto adalah bahasa dengan struktur morfologi yang sangat sistematis. Aplikasi menggunakan algoritma khusus untuk memecah kata-kata Esperanto berdasarkan akar kata, awalan, dan akhiran.

### 1. Penanganan Kata Kerja dan Akhirannya

Kata kerja Esperanto memiliki akhiran khusus untuk waktu dan modus, yang ditangani dengan cara berikut:

```python
# Kamus akhiran kata kerja
verb_suffix_2l = {
    'as':'as',  # kala kini
    'is':'is',  # kala lampau
    'os':'os',  # kala mendatang
    'us':'us',  # modus kondisional
    'at':'at',  # partisip pasif kala kini
    'it':'it',  # partisip pasif kala lampau
    'ot':'ot',  # partisip pasif kala mendatang
    'ad':'ad',  # tindakan berkelanjutan
    'iĝ':'iĝ',  # menjadi
    'ig':'ig',  # menyebabkan
    'ant':'ant', # partisip aktif kala kini
    'int':'int', # partisip aktif kala lampau
    'ont':'ont'  # partisip aktif kala mendatang
}

# Dalam fungsi pembuatan daftar penggantian:
if "verbo_s1" in i[2]:
    for k1, k2 in verb_suffix_2l_2.items():
        pre_replacements_dict_3[esperanto_Word_before_replacement + k1] = [
            Replaced_String + k2,
            replacement_priority_by_length + len(k1) * 10000
        ]
    i[2].remove("verbo_s1")
```

Ini memungkinkan satu akar kata kerja (misalnya "am" - "cinta") untuk diperluas menjadi berbagai bentuk ("amas" - "mencintai", "amis" - "mencintai (lampau)", dll).

### 2. Penanganan Akar Kata 2 Karakter

Akar kata 2 karakter mendapat perlakuan khusus karena bisa berfungsi sebagai awalan, akhiran, atau kata mandiri:

```python
# Daftar akar kata 2 karakter
suffix_2char_roots = ['ad', 'ag', 'am', 'ar', ...] # akhiran
prefix_2char_roots = ['al', 'am', 'av', 'bo', ...] # awalan
standalone_2char_roots = ['al', 'ci', 'da', 'de', ...] # kata mandiri

# Pembuatan daftar penggantian untuk akar kata 2 karakter
replacements_list_for_suffix_2char_roots = []
for i in range(len(suffix_2char_roots)):
    replaced_suffix = remove_redundant_ruby_if_identical(
        safe_replace(suffix_2char_roots[i], temporary_replacements_list_final))
    # Tambahkan tiga bentuk (normal, huruf besar, awal kapital)
    replacements_list_for_suffix_2char_roots.append([
        "$" + suffix_2char_roots[i],
        "$" + replaced_suffix,
        "$" + imported_placeholders_for_2char_replacement[i]
    ])
    # ... (tambahkan versi huruf besar dan awal kapital)
```

Pendekatan ini membantu aplikasi mengenali komponen kata yang kecil namun penting.

### 3. Penanganan Akhiran Khusus "an" dan "on"

Akhiran "an" (anggota) dan "on" (pecahan) mendapat penanganan khusus:

```python
# Daftar kata dengan akhiran "an" dan "on"
AN = [['dietan', '/diet/an/', '/diet/an'], ['afrikan', '/afrik/an/', '/afrik/an'], ...]
ON = [['duon', '/du/on/', '/du/on'], ['okon', '/ok/on/', '/ok/on'], ...]

# Pemrosesan kata-kata dengan akhiran "an"
for an in AN:
    if an[1].endswith("/an/"):
        i2 = an[1]
        i3 = re.sub(r"/an/$", "", i2)
        i4 = i3 + "/an/o"  # bentuk nomina
        i5 = i3 + "/an/a"  # bentuk adjektiva
        i6 = i3 + "/an/e"  # bentuk adverbia
        i7 = i3 + "/a/n/"  # bentuk adjektiva dengan akusatif

        # Buat aturan penggantian untuk setiap bentuk
        pre_replacements_dict_3[i4.replace('/', '')] = [
            safe_replace(i4, temporary_replacements_list_final)
                .replace("</rt></ruby>", "%%%")
                .replace('/', '')
                .replace("%%%", "</rt></ruby>"),
            (len(i4.replace('/', '')) - 1) * 10000 + 3000
        ]
        # ... (hal serupa untuk i5, i6, i7)
```

Pendekatan ini memastikan kata seperti "afrikan" (orang Afrika) diproses dengan benar, membedakannya dari kata seperti "hunda" (milik anjing) dengan akhiran "-a" biasa.

## 14. Detail Implementasi Anotasi Ruby

Anotasi Ruby adalah fitur utama aplikasi. Mari kita lihat implementasinya secara rinci.

### 1. Format HTML Ruby Dasar

Format Ruby HTML dasar sangat sederhana:

```html
<ruby>teks-utama<rt>anotasi</rt></ruby>
```

Namun, aplikasi memperluasnya menjadi format yang lebih canggih, dengan penyesuaian ukuran dan posisi:

```html
<ruby>teks-utama<rt class="M_M">anotasi</rt></ruby>
```

### 2. CSS untuk Penyesuaian Ruby

Aplikasi menyertakan CSS khusus untuk mengontrol tampilan Ruby:

```css
/* Bagian dari ruby_style_head dalam apply_ruby_html_header_and_footer */
:root {
  --ruby-color: blue;
  --ruby-font-size: 0.5em;
}

ruby {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  vertical-align: top !important;
  line-height: 2.0 !important;
  margin: 0 !important;
  padding: 0 !important;
  font-size: 1rem !important;
}

rt {
  display: block !important;
  font-size: var(--ruby-font-size);
  color: var(--ruby-color);
  line-height: 1.05;
  text-align: center;
}

rt.XXXS_S {
  --ruby-font-size: 0.3em;
  margin-top: -8.3em !important;
  transform: translateY(-0em) !important;
}

/* ... kelas ukuran lainnya (XXS_S, XS_S, dst) */
```

CSS ini menggunakan variabel CSS kustom, flexbox, dan transformasi untuk membuat tata letak yang fleksibel dan responsif.

### 3. Penyesuaian Ukuran Berdasarkan Rasio Lebar

Algoritma utama untuk menyesuaikan ukuran Ruby:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    if format_type == 'HTML格式_Ruby文字_大小调整':
        width_ruby = measure_text_width_Arial16(ruby_content, char_widths_dict)
        width_main = measure_text_width_Arial16(main_text, char_widths_dict)
        ratio_1 = width_ruby / width_main

        if ratio_1 > 6:
            # Ruby sangat panjang, gunakan ukuran terkecil dan bagi menjadi tiga baris
            return f'<ruby>{main_text}<rt class="XXXS_S">{insert_br_at_third_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/3):
            # Ruby panjang, gunakan ukuran kecil dan bagi menjadi dua baris
            return f'<ruby>{main_text}<rt class="XXS_S">{insert_br_at_half_width(ruby_content, char_widths_dict)}</rt></ruby>'
        elif ratio_1 > (9/4):
            return f'<ruby>{main_text}<rt class="XS_S">{ruby_content}</rt></ruby>'
        # ... dst untuk rasio yang berbeda
    # ... format lainnya
```

Aplikasi menghitung rasio lebar antara teks Ruby dan teks utama, lalu memilih kelas CSS yang sesuai berdasarkan rasio tersebut.

### 4. Pembagian Teks Ruby yang Panjang

Untuk teks Ruby yang panjang, aplikasi membaginya menjadi beberapa baris:

```python
def insert_br_at_third_width(text, char_widths_dict: Dict[str, int]) -> str:
    """Sisipkan tag <br> pada posisi 1/3 dan 2/3 dari lebar total teks."""
    total_width = measure_text_width_Arial16(text, char_widths_dict)
    third_width = total_width / 3
    thresholds = [third_width, third_width*2]
    current_width = 0
    insert_indices = []
    found_first = False

    # Cari posisi untuk menyisipkan <br>
    for i, ch in enumerate(text):
        char_width = char_widths_dict.get(ch, 8)
        current_width += char_width
        if not found_first and current_width >= thresholds[0]:
            insert_indices.append(i+1)
            found_first = True
        elif found_first and current_width >= thresholds[1]:
            insert_indices.append(i+1)
            break

    # Sisipkan <br> pada indeks yang ditemukan
    result = text
    for idx in reversed(insert_indices):
        result = result[:idx] + "<br>" + result[idx:]
    return result
```

Pendekatan ini memastikan teks Ruby yang panjang tetap terbaca dan tidak terlalu melebar.

## 15. Optimisasi Performa Lanjutan

Aplikasi menggunakan beberapa teknik optimisasi lanjutan untuk menangani data besar.

### 1. Multiprocessing untuk Penggantian Teks

Untuk teks panjang, aplikasi memecahnya menjadi beberapa segmen dan memprosesnya secara paralel:

```python
def parallel_process(
    text: str,
    num_processes: int,
    # parameter lainnya...
) -> str:
    if num_processes <= 1:
        # Proses sekuensial jika hanya 1 proses
        return orchestrate_comprehensive_esperanto_text_replacement(...)

    # Pecah teks menjadi beberapa baris
    lines = re.findall(r'.*?\n|.+$', text)
    num_lines = len(lines)

    if num_lines <= 1:
        # Gunakan proses sekuensial untuk teks pendek
        return orchestrate_comprehensive_esperanto_text_replacement(...)

    # Hitung rentang baris untuk setiap proses
    lines_per_process = max(num_lines // num_processes, 1)
    ranges = [(i * lines_per_process, (i + 1) * lines_per_process)
              for i in range(num_processes)]
    ranges[-1] = (ranges[-1][0], num_lines)  # Proses terakhir menangani sisanya

    # Proses setiap segmen secara paralel
    with multiprocessing.Pool(processes=num_processes) as pool:
        results = pool.starmap(
            process_segment,
            [
                (
                    lines[start:end],
                    placeholders_for_skipping_replacements,
                    replacements_list_for_localized_string,
                    placeholders_for_localized_replacement,
                    replacements_final_list,
                    replacements_list_for_2char,
                    format_type
                )
                for (start, end) in ranges
            ]
        )

    # Gabungkan hasil
    return ''.join(results)
```

Teknik ini dapat meningkatkan performa secara signifikan pada teks yang panjang.

### 2. Penghapusan Ruby Redundan

Aplikasi menghapus tag Ruby yang redundan untuk teks yang sama:

```python
def remove_redundant_ruby_if_identical(text: str) -> str:
    """
    Hapus tag Ruby jika teks utama dan anotasi Ruby sama persis.
    Contoh: <ruby>xxx<rt>xxx</rt></ruby> → xxx
    """
    def replacer(match: re.Match) -> str:
        group1 = match.group(1)
        group2 = match.group(2)
        if group1 == group2:
            return group1
        else:
            return match.group(0)

    replaced_text = IDENTICAL_RUBY_PATTERN.sub(replacer, text)
    return replaced_text
```

Ini mengurangi ukuran output dan meningkatkan keterbacaan.

### 3. Caching Hasil Penggantian

Dalam `parallel_build_pre_replacements_dict()`, aplikasi menyimpan hasil penggantian dalam kamus untuk menghindari penggantian berulang:

```python
def parallel_build_pre_replacements_dict(
    E_stem_with_Part_Of_Speech_list: List[List[str]],
    replacements: List[Tuple[str, str, str]],
    num_processes: int = 4
) -> Dict[str, List[str]]:
    # ... (kode pemecahan tugas)

    # Jalankan pemrosesan paralel
    with multiprocessing.Pool(num_processes) as pool:
        partial_dicts = pool.starmap(
            process_chunk_for_pre_replacements,
            [(chunk, replacements) for chunk in chunks]
        )

    # Gabungkan hasil
    merged_dict = {}
    for partial_d in partial_dicts:
        for E_root, val in partial_d.items():
            replaced_stem, pos_str = val
            if E_root not in merged_dict:
                merged_dict[E_root] = [replaced_stem, pos_str]
            else:
                # Gabungkan informasi kelas kata jika diperlukan
                existing_replaced_stem, existing_pos_str = merged_dict[E_root]
                existing_pos_list = existing_pos_str.split(',')
                new_pos_list = pos_str.split(',')
                pos_merged = list(set(existing_pos_list) | set(new_pos_list))
                pos_merged_str = ",".join(sorted(pos_merged))
                merged_dict[E_root] = [existing_replaced_stem, pos_merged_str]

    return merged_dict
```

Pendekatan ini mempercepat pembuatan file JSON secara signifikan.

## 16. Perluasan dan Kustomisasi Aplikasi

Aplikasi dirancang untuk memudahkan perluasan dan kustomisasi.

### 1. Menambahkan Format Output Baru

Untuk menambahkan format output baru, Anda perlu memodifikasi fungsi `output_format()`:

```python
def output_format(main_text, ruby_content, format_type, char_widths_dict):
    # ... format yang ada

    # Tambahkan format baru
    elif format_type == 'FORMAT_BARU':
        return f'<span class="custom">{main_text}「{ruby_content}」</span>'
```

Anda juga perlu menambahkannya ke variabel `options` di file UI:

```python
options = {
    # ... opsi yang ada
    "Format Kustom Baru": "FORMAT_BARU"
}
```

### 2. Menyesuaikan Aturan Penggantian

Anda dapat membuat aturan penggantian kustom dengan:

1. Membuat file CSV dengan format:
   ```
   akar_kata_esperanto,teks_pengganti
   am,cinta
   bird,burung
   ```

2. Membuat file JSON untuk pemecahan akar kata:
   ```json
   [
     ["am", "dflt", ["verbo_s1"]],
     ["bird", "dflt", ["o", "oj", "on"]]
   ]
   ```

3. Menggunakan halaman "Pembuatan JSON" untuk membuat file JSON final

4. Mengunggah file JSON saat menggunakan aplikasi utama

### 3. Menambahkan Dukungan Bahasa Baru

Untuk menambahkan bahasa baru, Anda perlu:

1. Membuat file CSV dengan terjemahan dalam bahasa target
2. Membuat file JSON dengan aturan pemecahan akar kata yang sesuai
3. Menyesuaikan model UI untuk bahasa tersebut

## 17. Panduan Debugging dan Pemecahan Masalah

Beberapa masalah umum dan solusinya:

### 1. Penggantian yang Tidak Konsisten

Jika penggantian menghasilkan output yang tidak diharapkan:

- Periksa urutan prioritas penggantian (kata yang lebih panjang harus memiliki prioritas lebih tinggi)
- Pastikan placeholder unik dan tidak bentrok
- Periksa apakah ada konflik antara penggantian global dan lokal

### 2. Masalah Performa

Jika aplikasi berjalan lambat:

- Aktifkan pemrosesan paralel dan atur jumlah proses yang sesuai
- Periksa ukuran file JSON (terlalu besar dapat memperlambat pemuatan)
- Pertimbangkan untuk memecah teks input menjadi bagian yang lebih kecil

### 3. Masalah Tampilan Ruby

Jika anotasi Ruby tidak tampil dengan benar:

- Periksa apakah CSS dirender dengan benar
- Periksa rasio lebar teks Ruby dan teks utama
- Pastikan tag HTML dihasilkan dengan benar

## 18. Kesimpulan dan Praktik Terbaik

Dalam mengembangkan aplikasi serupa, ada beberapa praktik terbaik yang dapat diambil dari aplikasi ini:

### 1. Modularisasi dan Pemisahan Kekhawatiran

Pisahkan logika bisnis inti (penggantian teks) dari antarmuka pengguna dan fungsi pembantu.

### 2. Strategi Penggantian yang Aman

Gunakan pendekatan placeholder untuk menghindari konflik penggantian.

### 3. Prioritaskan Performa

Implementasikan paralelisasi dan caching untuk menangani data besar dengan efisien.

### 4. Rancang untuk Fleksibilitas

Buat sistem yang dapat diperluas dan disesuaikan tanpa perubahan besar pada kode inti.

### 5. Uji dengan Kasus Edge

Uji aplikasi dengan berbagai kasus, termasuk teks yang sangat panjang, karakter khusus, dan pola yang tidak biasa.

Aplikasi penggantian teks Esperanto ini menunjukkan penerapan prinsip-prinsip ini dengan sangat baik, menghasilkan alat yang kuat, fleksibel, dan efisien untuk menangani teks multilingual dan anotasi.
