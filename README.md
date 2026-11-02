# Tugas Mid-term Big Data: Analisis Kecelakaan Lalu Lintas California (SWITRS)

Proyek individu analisis data berskala besar menggunakan PySpark pada Google Colab.

| | |
|---|---|
| **Nama** | Caesar Gian Indrarizky |
| **NIM** | 103012300218 |
| **File utama** | `Mid_Term_Big_Data_103012300218_Caesar_Gian_I_Revisi_.ipynb` |

## 1. Ringkasan

Notebook ini menjawab pertanyaan: faktor apa yang paling memengaruhi jumlah dan tingkat keparahan insiden lalu lintas di California. Alur kerjanya:

1. Memasang PySpark 4.0.4 dan membuat `SparkSession` mode lokal.
2. Mengunduh dataset SWITRS dari Kaggle memakai token dari Colab Secrets.
3. Membaca **100.000 baris pertama** tabel `collisions` dari SQLite lewat pandas, lalu memindahkannya ke PySpark DataFrame.
4. Membandingkan waktu eksekusi DataFrame API dan Spark SQL.
5. Membuat EDA berupa 4 grafik (county, populasi, keparahan, cuaca).
6. Membersihkan data (cast tipe, hapus duplikat, buang `chp_shift` tidak valid) beserta 3 cek kualitas.
7. Melatih baseline Logistic Regression untuk memprediksi `is_dense_area`.

## 2. Tautan Dataset

- **Nama:** California Traffic Collision Data from SWITRS
- **Tautan:** https://www.kaggle.com/datasets/alexgude/california-traffic-collision-data-from-switrs
- **Slug kagglehub:** `alexgude/california-traffic-collision-data-from-switrs`
- **Format:** satu file SQLite bernama `switrs.sqlite` (sekitar 2 GB menurut notebook)
- **Tabel di dalamnya:** `case_ids`, `collisions`, `victims`, `parties`
- **Tabel yang dipakai:** hanya `collisions`

Cek halaman Kaggle di atas untuk lisensi dan ketentuan penggunaan sebelum menyebarluaskan ulang datanya. File data **tidak disertakan** di repositori ini.

## 3. Petunjuk Memperoleh File Dataset

Ada dua cara.

### Cara A: Otomatis lewat `kagglehub` (yang dipakai notebook)

1. Buat akun Kaggle, lalu buat API token di **Settings > API** pada profil Kaggle.
2. Di Google Colab, buka **Secrets** (ikon kunci di sidebar kiri), tambahkan secret dengan nama persis **`KAGGLE_API_TOKEN`**, isi dengan token Anda, lalu aktifkan **Notebook access** untuk notebook ini.
3. Sel 8 notebook akan memuat token tersebut dan mengunduh dataset:

   ```python
   import os
   import kagglehub
   from google.colab import userdata

   os.environ["KAGGLE_API_TOKEN"] = userdata.get("KAGGLE_API_TOKEN")
   dataset_path = kagglehub.dataset_download(
       "alexgude/california-traffic-collision-data-from-switrs"
   )
   ```

   Pada Colab, path hasilnya `/kaggle/input/california-traffic-collision-data-from-switrs`, dan folder itu berisi `switrs.sqlite`.

Token **tidak ditulis** di dalam notebook. Jangan pernah mengunggah token ke GitHub atau LMS.

### Cara B: Manual (untuk lingkungan non-Colab)

1. Buka tautan dataset di Kaggle dan klik **Download**.
2. Ekstrak arsip hingga diperoleh `switrs.sqlite`.
3. Letakkan di folder `data/`, ubah `sqlite_db_path` di sel 8 menjadi `data/switrs.sqlite`, dan hapus baris `from google.colab import userdata` beserta pemanggilan `userdata.get(...)` serta `kagglehub.dataset_download(...)`, karena modul `google.colab` hanya tersedia di Colab.

## 4. Versi Python dan PySpark

Nilai berikut dicetak langsung oleh sel pengecekan versi di notebook (runtime Google Colab):

| Komponen | Versi |
|---|---|
| Python | 3.13.16 |
| PySpark / Apache Spark | 4.0.4 (dikunci lewat `pip install pyspark==4.0.4`) |
| pandas | 2.2.3 |
| matplotlib | 3.10.0 |
| seaborn | 0.13.2 |
| kagglehub | 1.0.3 |
| Java (OpenJDK) | 21.0.12.1 (bawaan Colab, tidak dipasang manual) |

Konfigurasi Spark: mode `local[*]`, `spark.driver.memory=4g`, nama aplikasi `MidtermProject_BigData`. Metadata notebook mencatat akselerator GPU T4, tetapi **tidak ada bagian kode yang memakai GPU**, sehingga runtime CPU biasa sudah cukup.

Untuk instalasi di luar Colab:

```bash
pip install pyspark==4.0.4 pandas==2.2.3 matplotlib==3.10.0 seaborn==0.13.2 kagglehub==1.0.3
```

Pastikan Java 17 atau 21 terpasang (`java -version`). Spark 4.x tidak dirancang untuk Java 8, silakan verifikasi di dokumentasi Spark resmi.

## 5. Struktur Folder

Kondisi pengumpulan saat ini hanya satu file notebook. Semua gambar (diagram pipeline, tangkapan layar versi, dan sebagainya) tertanam di dalam notebook.

```
.
├── Mid_Term_Big_Data_103012300218_Caesar_Gian_I_Revisi_.ipynb
└── README.md
```

Jika dijalankan secara lokal, struktur yang disarankan:

```
.
├── Mid_Term_Big_Data_103012300218_Caesar_Gian_I_Revisi_.ipynb
├── README.md
├── requirements.txt          # opsional, isi sesuai tabel versi di atas
└── data/
    └── switrs.sqlite         # diunduh sendiri, JANGAN di-commit
```

Tambahkan `data/` ke `.gitignore` karena ukurannya sekitar 2 GB.

## 6. Cara Menjalankan

### Di Google Colab (disarankan)

1. Unggah notebook ke Colab (**File > Upload notebook**).
2. Siapkan secret `KAGGLE_API_TOKEN` seperti pada bagian 3 (Cara A).
3. Pastikan koneksi internet stabil.
4. Pilih **Runtime > Run all**.
5. Sel dijalankan berurutan:

| Sel | Isi |
|---|---|
| 6 | `pip install -q pyspark==4.0.4` |
| 7 | Buat `SparkSession` |
| 8 | Muat token dari Secrets, unduh dataset, baca 100.000 baris, buat Spark DataFrame |
| 9 | Filter, kolom turunan, groupBy/orderBy, Spark SQL, ukur waktu (dengan action `.count()`) |
| 12 | EDA, 4 grafik |
| 15 | Casting, deduplikasi, pembersihan, 3 cek kualitas |
| 18 | Baseline Logistic Regression |
| 40 | Cetak versi Python dan library |

### Secara lokal

1. Pasang Java 17 atau 21, lalu buat virtual environment dan pasang paket dari bagian 4.
2. Siapkan dataset secara manual (bagian 3, Cara B).
3. Buka notebook dengan Jupyter dan jalankan semua sel dari atas. Abaikan perintah `!java -version` jika memakai Windows tanpa shell yang mendukung tanda seru.

### Hasil yang diharapkan (untuk verifikasi)

| Pengecekan | Nilai di notebook |
|---|---|
| Versi Spark yang berjalan | 4.0.4 |
| Baris sebelum pembersihan | 100.000 |
| Baris setelah pembersihan | 43.280 |
| Baris dibuang | 56.720 |
| ID duplikat, populasi kosong, shift tidak valid | 0, 0, 0 |
| Area Under ROC (AUC) | 0,5320 |
| Akurasi baseline | 0,8036 |

Waktu eksekusi pada run yang tercatat: DataFrame API 3,9046 detik dan Spark SQL 3,2217 detik. Angka ini **akan berbeda** pada setiap run dan mesin, jadi tidak perlu sama persis.

## 7. Seed

| Lokasi | Nilai | Fungsi |
|---|---|---|
| `df_ml.randomSplit([0.7, 0.3], seed=42)` (sel 18) | **42** | Pembagian data latih 70% dan uji 30% |

Jangan ubah nilai ini jika ingin mendapatkan metrik yang sama. Tidak ada seed global lain (misalnya untuk Python `random` atau NumPy) karena tahapan lain tidak memakai komponen acak. `StringIndexer`, `VectorAssembler`, dan `LogisticRegression` dengan parameter bawaan bersifat deterministik.

## 8. Daftar Keterbatasan Reproduksi

1. **Sampel bukan keseluruhan data.** Notebook hanya membaca `SELECT * FROM collisions LIMIT 100000` tanpa `ORDER BY`. Urutan baris bergantung pada isi file SQLite. Jika versi dataset di Kaggle berubah, 100.000 baris pertamanya bisa berbeda dan angka 43.280 serta metrik model ikut berubah.
2. **Versi dataset tidak dikunci.** `kagglehub.dataset_download` dipanggil tanpa menentukan versi, sehingga mengambil versi terbaru atau versi yang tersimpan di cache Colab.
3. **Sampel mewakili rentang waktu yang terpotong.** Karena diambil dari awal tabel, sampel belum tentu mencakup siklus tahunan penuh, sehingga kesimpulan tidak boleh digeneralisasi ke seluruh California.
4. **Butuh akun Kaggle, token aktif, dan internet.** Tanpa secret `KAGGLE_API_TOKEN` yang valid dan akses notebook yang diaktifkan, sel 8 gagal. Selain itu, `from google.colab import userdata` membuat notebook **hanya berjalan di Colab** tanpa modifikasi.
5. **Bergantung pada lingkungan bawaan Colab.** PySpark sudah dikunci ke 4.0.4, tetapi Python (3.13.16), Java (OpenJDK 21), pandas, matplotlib, seaborn, dan kagglehub mengikuti image runtime Colab. Jika Colab memperbarui image-nya, versi ini bisa berubah dan hasil bisa bergeser. Versi pada bagian 4 adalah yang tercatat saat run.
6. **Hasil `randomSplit` dapat bergantung pada partisi.** Dengan seed tetap pun, pembagian data bisa sedikit berbeda jika jumlah partisi atau konfigurasi `local[*]` (jumlah core) berbeda antar mesin.
7. **Waktu eksekusi tidak reproduksibel dan perbandingannya belum setara.** Pengukuran kini memakai action `.count()`, tetapi hasilnya tetap bergantung pada beban Colab dan jumlah core. Pengukuran pertama (DataFrame API) juga menanggung biaya pemanasan awal (JVM, konversi data dari pandas), sehingga cenderung lebih lambat. Selain itu, kedua kueri tidak mengerjakan tugas yang identik (satu memfilter dan mengelompokkan per county, satu mengelompokkan per `jurisdiction` dengan `LIMIT 10`), jadi selisih 3,9 vs 3,2 detik tidak bisa dibaca sebagai bukti satu metode lebih unggul.
8. **Konversi pandas ke Spark mengubah tipe data.** Kolom bertipe `object` diisi `""` lalu dijadikan string, sehingga nilai kosong menjadi string kosong, bukan `NULL`. Karena itu filter lokasi memakai `!= ""` (bukan `isNotNull()`). Kolom `jurisdiction` juga harus di-cast kembali ke `double`.
9. **Kebutuhan memori.** Driver diset 4 GB. Pada runtime dengan RAM lebih kecil, proses bisa mengalami Out of Memory. Pengaturan `spark.executor.memory` tidak berpengaruh pada mode `local`.
10. **Output visual bisa berbeda tampilan.** Seaborn 0.13.2 menampilkan `FutureWarning` untuk argumen `palette` tanpa `hue`. Ini tidak memengaruhi angka, tetapi tampilan dapat berbeda pada versi seaborn lain.
11. **Tautan referensi eksternal.** Jurnal rujukan dan tangkapan layar alat AI di bagian akhir notebook tidak dapat diverifikasi ulang secara otomatis.

## 9. Deklarasi Penggunaan AI

Sesuai deklarasi di notebook, tugas ini dikerjakan secara individual dan memakai alat AI generatif secara terbatas (Paperguide AI untuk pencarian referensi, Gemini untuk memperbaiki instalasi Spark). Seluruh interpretasi, insight, dan kesimpulan merupakan hasil pemahaman penulis.

## 10. Referensi

1. Dataset: [California Traffic Collision Data from SWITRS](https://www.kaggle.com/datasets/alexgude/california-traffic-collision-data-from-switrs)
2. [Predicting Traffic Accident Severity Using Machine Learning Techniques](https://dergipark.org.tr/en/pub/tdfd/article/1136432)
3. [Analyzing Autonomous Vehicle Collision Types to Support Sustainable Transportation Systems: A Machine Learning and Association Rules Approach](https://www.mdpi.com/2071-1050/16/22/9893)
4. [Comparison of Statistical and Machine-Learning Models on Road Traffic Accident Severity Classification](https://www.mdpi.com/2073-431X/11/5/80)
