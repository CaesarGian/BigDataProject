# Tugas Mid-term Big Data: Analisis Kecelakaan Lalu Lintas California (SWITRS)

Proyek individu analisis data berskala besar menggunakan PySpark pada Google Colab.

| | |
|---|---|
| **Nama** | Caesar Gian Indrarizky |
| **NIM** | 103012300218 |
| **File utama** | `Mid_Term_Big_Data_103012300218_Caesar_Gian_I__Revisi_.ipynb` |

## 1. Ringkasan

Notebook ini mencari faktor yang paling memengaruhi jumlah dan tingkat keparahan insiden lalu lintas di California. Alur kerjanya:

1. Memasang PySpark 4.0.4 dan membuat `SparkSession` mode lokal.
2. Mengunduh dataset SWITRS dari Kaggle memakai token dari Colab Secrets.
3. Membaca **100.000 baris pertama** tabel `collisions` dari SQLite lewat pandas, lalu memindahkannya ke PySpark DataFrame.
4. Membandingkan waktu eksekusi DataFrame API dan Spark SQL.
5. Membuat EDA berupa 4 grafik jumlah insiden (county, populasi, keparahan, cuaca).
6. Menghitung **persentase insiden parah** (severe injury dan fatal) per kondisi cuaca dan per kategori populasi, lalu menampilkannya dalam 2 grafik tambahan. Langkah ini mengubah jumlah mentah menjadi proporsi.
7. Membersihkan data (cast tipe, hapus duplikat, buang `chp_shift` tidak valid) beserta 3 cek kualitas.
8. Melatih baseline Logistic Regression untuk memprediksi `is_dense_area`.

## 2. Tautan Dataset

- **Nama:** California Traffic Collision Data from SWITRS
- **Tautan:** https://www.kaggle.com/datasets/alexgude/california-traffic-collision-data-from-switrs
- **Slug kagglehub:** `alexgude/california-traffic-collision-data-from-switrs`
- **Versi yang terunduh pada run tercatat:** versi 6 (terlihat dari path `.../versions/6/switrs.sqlite`)
- **Tanggal akses:** 7 November 2026 (sesuai tanggal cetak PDF notebook)
- **Format:** satu file SQLite bernama `switrs.sqlite` (sekitar 2 GB menurut notebook)
- **Tabel di dalamnya:** `case_ids`, `collisions`, `victims`, `parties`
- **Tabel yang dipakai:** hanya `collisions`

Cek halaman Kaggle di atas untuk lisensi dan ketentuan penggunaan sebelum menyebarluaskan ulang datanya. File data **tidak disertakan** di repositori ini.

## 3. Petunjuk Memperoleh File Dataset

Ada dua cara.

### Cara A: Otomatis lewat `kagglehub` (yang dipakai notebook)

1. Buat akun Kaggle, lalu buat API token di **Settings > API** pada profil Kaggle.
2. Di Google Colab, buka **Secrets** (ikon kunci di sidebar kiri), tambahkan secret bernama persis **`KAGGLE_API_TOKEN`** berisi token Anda, lalu aktifkan **Notebook access** untuk notebook ini.
3. Sel 9 notebook memuat token tersebut dan mengunduh dataset:

   ```python
   import os
   import kagglehub
   from google.colab import userdata

   os.environ["KAGGLE_API_TOKEN"] = userdata.get("KAGGLE_API_TOKEN")
   dataset_path = kagglehub.dataset_download(
       "alexgude/california-traffic-collision-data-from-switrs"
   )
   ```

   Pada run tercatat, hasilnya berada di `/root/.cache/kagglehub/datasets/alexgude/california-traffic-collision-data-from-switrs/versions/6`, dan folder itu berisi `switrs.sqlite`.

4. Untuk mengunci versi data, tambahkan nomor versi pada slug: `"alexgude/california-traffic-collision-data-from-switrs/versions/6"`. Notebook saat ini belum melakukannya, dan format ini sebaiknya dicoba dulu sebelum dipakai.

Token **tidak boleh** ditulis di dalam sel notebook maupun muncul di tangkapan layar. Jangan mengunggahnya ke GitHub atau LMS.

### Cara B: Manual (untuk lingkungan non-Colab)

1. Buka tautan dataset di Kaggle dan klik **Download**.
2. Ekstrak arsip hingga diperoleh `switrs.sqlite`.
3. Letakkan di folder `data/`, ubah `sqlite_db_path` di sel 9 menjadi `data/switrs.sqlite`, dan hapus baris `from google.colab import userdata`, pemanggilan `userdata.get(...)`, serta `kagglehub.dataset_download(...)`, karena modul `google.colab` hanya ada di Colab.

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

Konfigurasi Spark: mode `local[*]`, `spark.driver.memory=4g`, `spark.executor.memory=4g`, nama aplikasi `MidtermProject_BigData`. Metadata notebook mencatat akselerator GPU T4, tetapi **tidak ada bagian kode yang memakai GPU**, sehingga runtime CPU biasa sudah cukup.

Untuk instalasi di luar Colab:

```bash
pip install pyspark==4.0.4 pandas==2.2.3 matplotlib==3.10.0 seaborn==0.13.2 kagglehub==1.0.3
```

Pastikan Java 17 atau 21 terpasang (`java -version`). Spark 4.x tidak dirancang untuk Java 8, silakan verifikasi di dokumentasi Spark resmi.

## 5. Struktur Folder

Kondisi pengumpulan saat ini hanya satu file notebook. Semua gambar (diagram pipeline, tangkapan layar versi, dan sebagainya) tertanam di dalam notebook.

```
.
├── Mid_Term_Big_Data_103012300218_Caesar_Gian_I__Revisi_.ipynb
└── README.md
```

Jika dijalankan secara lokal, struktur yang disarankan:

```
.
├── Mid_Term_Big_Data_103012300218_Caesar_Gian_I__Revisi_.ipynb
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
5. Sel kode dijalankan berurutan (nomor adalah indeks sel pada file `.ipynb`, dihitung dari 0 dan sudah termasuk sel markdown):

| Sel | Isi |
|---|---|
| 7 | `pip install -q pyspark==4.0.4` |
| 8 | Buat `SparkSession` |
| 9 | Muat token dari Secrets, unduh dataset, baca 100.000 baris, buat Spark DataFrame, cetak skema dan sampel |
| 10 | Filter, kolom turunan `is_dense_area`, groupBy/orderBy, Spark SQL, ukur waktu (dengan action `.count()`) |
| 13 | EDA, 4 grafik jumlah insiden |
| 14 | Fungsi `severity_rate`, persentase insiden parah per cuaca dan per populasi |
| 15 | Grafik persentase insiden parah |
| 18 | Casting, deduplikasi, pembersihan, 3 cek kualitas |
| 21 | Baseline Logistic Regression |
| 43 | Cetak versi Python dan library |

### Secara lokal

1. Pasang Java 17 atau 21, lalu buat virtual environment dan pasang paket dari bagian 4.
2. Siapkan dataset secara manual (bagian 3, Cara B).
3. Buka notebook dengan Jupyter dan jalankan semua sel dari atas. Perintah `!java -version` di sel 43 memakai sintaks IPython dan mungkin perlu disesuaikan di Windows.

### Hasil yang diharapkan (untuk verifikasi)

**Pembersihan dan model**

| Pengecekan | Nilai di notebook |
|---|---|
| Versi Spark yang berjalan | 4.0.4 |
| Baris sebelum pembersihan | 100.000 |
| Baris setelah pembersihan | 43.280 |
| Baris dibuang | 56.720 |
| ID duplikat, populasi kosong, shift tidak valid | 0, 0, 0 |
| Area Under ROC (AUC) | 0,5320 |
| Akurasi baseline | 0,8036 |

**Persentase insiden parah (severe injury + fatal), sel 14, dihitung dari 100.000 baris sebelum pembersihan**

| Kondisi cuaca | Total | Parah | Persen parah |
|---|---|---|---|
| fog | 704 | 29 | 4,12 |
| other | 110 | 4 | 3,64 |
| clear | 71.329 | 1.899 | 2,66 |
| cloudy | 20.191 | 491 | 2,43 |
| raining | 6.736 | 118 | 1,75 |
| snowing | 358 | 6 | 1,68 |

| Kategori populasi | Total | Parah | Persen parah |
|---|---|---|---|
| unincorporated | 22.164 | 928 | 4,19 |
| >250000 | 25.348 | 648 | 2,56 |
| 2500 to 10000 | 1.242 | 29 | 2,33 |
| 50000 to 100000 | 18.556 | 359 | 1,93 |
| 10000 to 25000 | 4.036 | 78 | 1,93 |
| 25000 to 50000 | 8.497 | 162 | 1,91 |
| 100000 to 250000 | 19.717 | 357 | 1,81 |
| <2500 | 440 | 1 | 0,23 |

**Waktu eksekusi** pada run tercatat: DataFrame API 3,7038 detik dan Spark SQL 4,7887 detik. Angka ini **akan berbeda** pada setiap run dan mesin. Pada beberapa run sebelumnya urutan tercepatnya bahkan terbalik, jadi tidak perlu sama persis.

## 7. Seed

| Lokasi | Nilai | Fungsi |
|---|---|---|
| `df_ml.randomSplit([0.7, 0.3], seed=42)` (sel 21) | **42** | Pembagian data latih 70% dan uji 30% |

Jangan ubah nilai ini jika ingin mendapatkan metrik yang sama. Tidak ada seed global lain (misalnya untuk Python `random` atau NumPy) karena tahapan lain tidak memakai komponen acak. `StringIndexer`, `VectorAssembler`, `LogisticRegression`, dan semua agregasi Spark SQL dengan parameter bawaan bersifat deterministik.

## 8. Daftar Keterbatasan Reproduksi

1. **Sampel bukan keseluruhan data.** Notebook hanya membaca `SELECT * FROM collisions LIMIT 100000` tanpa `ORDER BY`. Urutan baris bergantung pada isi file SQLite. Jika versi dataset di Kaggle berubah, 100.000 baris pertamanya bisa berbeda dan semua angka di bagian 6 ikut berubah.
2. **Versi dataset tidak dikunci.** `kagglehub.dataset_download` dipanggil tanpa nomor versi. Run tercatat memakai versi 6, tetapi run di masa depan bisa mengambil versi yang lebih baru atau versi yang tersimpan di cache Colab.
3. **Sampel mewakili rentang waktu yang terpotong.** Karena diambil dari awal tabel, sampel belum tentu mencakup siklus tahunan penuh, sehingga kesimpulan tidak boleh digeneralisasi ke seluruh California.
4. **Butuh akun Kaggle, token aktif, dan internet.** Tanpa secret `KAGGLE_API_TOKEN` yang valid dan akses notebook yang diaktifkan, sel 9 gagal. Selain itu, `from google.colab import userdata` membuat notebook **hanya berjalan di Colab** tanpa modifikasi.
5. **Bergantung pada lingkungan bawaan Colab.** PySpark sudah dikunci ke 4.0.4, tetapi Python (3.13.16), Java (OpenJDK 21), pandas, matplotlib, seaborn, dan kagglehub mengikuti image runtime Colab. Jika Colab memperbarui image-nya, versi ini bisa berubah dan hasil bisa bergeser.
6. **Hasil `randomSplit` dapat bergantung pada partisi.** Dengan seed tetap pun, pembagian data bisa sedikit berbeda jika jumlah partisi atau konfigurasi `local[*]` (jumlah core) berbeda antar mesin.
7. **Waktu eksekusi tidak reproduksibel dan perbandingannya belum setara.** Pengukuran memakai action `.count()`, tetapi hasilnya bergantung pada beban Colab dan jumlah core. Kedua kueri tidak mengerjakan tugas yang identik (satu memfilter dan mengelompokkan per county, satu mengelompokkan per `jurisdiction` dengan `LIMIT 10`), dan hanya satu kali pengukuran, sehingga selisihnya tidak bisa dibaca sebagai bukti satu metode lebih unggul.
8. **Konversi pandas ke Spark mengubah tipe data.** Kolom bertipe `object` diisi `""` lalu dijadikan string, sehingga nilai kosong menjadi string kosong, bukan `NULL`. Karena itu filter lokasi memakai `!= ""` (bukan `isNotNull()`). Kolom `jurisdiction` juga harus di-cast kembali ke `double`.
9. **Definisi dan batas persentase parah.** "Parah" berarti `collision_severity` bernilai `severe injury` atau `fatal`. Kategori dengan kurang dari 100 insiden dibuang (`min_n=100`), dan kategori kecil yang lolos (misalnya salju dengan 358 insiden atau populasi di bawah 2.500 jiwa dengan 440 insiden) memiliki angka yang tidak stabil. Persentase ini menunjukkan seberapa sering kecelakaan berakhir parah **jika kecelakaan terjadi**. Ia belum dibagi eksposur seperti jumlah penduduk, kendaraan, atau frekuensi kondisi cuaca, sehingga belum menyatakan risiko terjadinya kecelakaan.
10. **Grafik populasi EDA berbeda cakupan dengan tabel proporsi.** Grafik jumlah per populasi (sel 13) mengecualikan kategori `unincorporated` (22.164 insiden), sedangkan perhitungan persentase parah (sel 14) menyertakannya.
11. **Kebutuhan memori.** Driver diset 4 GB. Pada runtime dengan RAM lebih kecil, proses bisa mengalami Out of Memory. Pengaturan `spark.executor.memory` tidak berpengaruh pada mode `local`.
12. **Output visual bisa berbeda tampilan.** Seaborn 0.13.2 menampilkan `FutureWarning` untuk argumen `palette` tanpa `hue`. Ini tidak memengaruhi angka, tetapi tampilan dapat berbeda pada versi seaborn lain.
13. **Tautan referensi eksternal.** Jurnal rujukan dan tangkapan layar alat AI di bagian akhir notebook tidak dapat diverifikasi ulang secara otomatis.


## 9. Referensi

1. Dataset: [California Traffic Collision Data from SWITRS](https://www.kaggle.com/datasets/alexgude/california-traffic-collision-data-from-switrs)
2. [Predicting Traffic Accident Severity Using Machine Learning Techniques](https://dergipark.org.tr/en/pub/tdfd/article/1136432)
3. [Analyzing Autonomous Vehicle Collision Types to Support Sustainable Transportation Systems: A Machine Learning and Association Rules Approach](https://www.mdpi.com/2071-1050/16/22/9893)
4. [Comparison of Statistical and Machine-Learning Models on Road Traffic Accident Severity Classification](https://www.mdpi.com/2073-431X/11/5/80)
