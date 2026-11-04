# flight-delays-pyspark-analysis
# Analisis Keterlambatan Penerbangan AS 2015 dengan PySpark

Midterm Project Big Data & AI, Program Studi S1 Informatika, Fakultas Informatika, Universitas Telkom.
Kelas BS1IF-47-GAB01.

| Nama | NIM |
|---|---|
| Muhammad Azigha Azhar | 103012300143 |
| Jehezkiel Beny Brian | 103012300241 |

## Ringkasan

Proyek ini menganalisis 5.819.079 catatan penerbangan domestik Amerika Serikat tahun 2015 dengan PySpark (mode lokal). Delay didefinisikan mengikuti BTS, yaitu `ARRIVAL_DELAY >= 15` menit. Setelah pembersihan tersisa 5.714.008 penerbangan valid dan 18,61% di antaranya terlambat.

Pertanyaan utama: maskapai, bandara, dan periode mana yang paling sering terlambat, dan seberapa baik fitur pra-keberangkatan membedakan penerbangan yang akan terlambat. Hasil lengkap ada di laporan PDF.

## Dataset

| Item | Keterangan |
|---|---|
| Nama | 2015 Flight Delays and Cancellations |
| Sumber | Kaggle, bersumber dari U.S. DOT Bureau of Transportation Statistics |
| Tautan | https://www.kaggle.com/datasets/usdot/flight-delays |
| Lisensi | CC0 (domain publik) |
| Tanggal akses | 2 November 2026 |
| Berkas | `flights.csv` (5.819.079 baris, 31 kolom, sekitar 592 MB), `airports.csv` (322 baris), `airlines.csv` (14 baris) |
| Data pribadi | Tidak ada. Data berupa catatan per penerbangan. |

### Cara memperoleh data

Data tidak disertakan di repositori karena berukuran besar. Ada dua cara:

1. Unduh manual dari tautan Kaggle di atas, ekstrak, lalu taruh ketiga CSV langsung di folder `data/`.
2. Dengan Kaggle CLI (butuh akun dan API token Kaggle):

```bash
pip install kaggle
kaggle datasets download -d usdot/flight-delays -p data/ --unzip
```

Pastikan hasil akhirnya `data/flights.csv`, `data/airports.csv`, dan `data/airlines.csv`.

## Struktur folder

```
flight-delays-pyspark/
├── notebooks/
│   └── 01_analisis_flight_delays.ipynb   # notebook utama (Tahap 1 sampai 6)
├── reports/
│   └── figures/                          # 10 grafik hasil notebook (PNG)
├── data/                                 # tidak di-commit, diisi manual
│   ├── flights.csv
│   ├── airports.csv
│   ├── airlines.csv
│   └── processed/                        # dibuat otomatis oleh notebook (Parquet)
├── requirements.txt
└── README.md
```

## Lingkungan

Versi yang dipakai saat notebook dijalankan:

| Komponen | Versi |
|---|---|
| Python | 3.14.6 |
| PySpark | 4.3.0 |
| Java | 25.0.3 |
| pandas | 2.3.3 |
| pyarrow | 26.0.0 |
| matplotlib | 3.11.2 |
| Sistem | macOS, satu laptop |

Konfigurasi Spark: `local[*]` (semua core), `spark.driver.memory = 4g`, `spark.sql.shuffle.partitions = 16`.

## Cara menjalankan

1. Pasang Java (JDK) dan pastikan `java -version` berjalan.
2. Buat virtual environment dan pasang dependensi:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

3. Siapkan data di folder `data/` (lihat bagian Cara memperoleh data).
4. Jalankan notebook dari folder `notebooks/`:

```bash
cd notebooks
jupyter lab
```

5. Buka `01_analisis_flight_delays.ipynb`, lalu pilih **Restart Kernel and Run All Cells**.

Seluruh notebook selesai dalam kurang dari 10 menit pada laptop dengan memori 16 GB. Path data memakai `../data/` sehingga notebook harus dijalankan dari folder `notebooks/`. Mode `overwrite` dipakai saat menulis Parquet, jadi notebook aman dijalankan berulang.

## Isi notebook

| Tahap | Isi |
|---|---|
| 1 | Setup Spark, pencatatan versi, ingestion dengan schema eksplisit, jumlah baris dan partisi |
| 2 | Processing: lima operasi PySpark (select/filter, groupBy, join, kolom turunan, window), dua pertanyaan dengan Spark SQL, dan pengukuran waktu CSV vs Parquet |
| 3 | EDA dan visualisasi: statistik deskriptif, distribusi, pola maskapai, bulan, jam, bandara, dan penyebab delay |
| 4 | Preprocessing dan validasi: duplikat, null, rentang nilai, konsistensi penyebab, pencilan, dengan hitungan baris sebelum dan sesudah |
| 5 | Baseline MLlib: Pipeline (StringIndexer, OneHotEncoder, StandardScaler, LogisticRegression berbobot), tuning dengan TrainValidationSplit, evaluasi dan analisis desil |
| 6 | Temuan, keterbatasan, dan rekomendasi |

Grafik disimpan otomatis ke `reports/figures/`.

## Reproducibility

- Seed 42 dipakai untuk `randomSplit` (80/20) dan `TrainValidationSplit`.
- Semua angka di laporan dihitung langsung oleh notebook, bukan disalin dari sumber lain.
- Data mentah tidak diubah. Hasil olahan ditulis ke `data/processed/`.

## Keterbatasan reproduksi

- Spark berjalan di mode lokal pada satu laptop. Waktu eksekusi (misalnya CSV vs Parquet) bergantung pada hardware, cache sistem operasi, dan I/O disk, sehingga angka bisa berbeda di mesin lain.
- Hasil `randomSplit` dengan seed tetap bisa sedikit berbeda bila jumlah core atau partisi berbeda, sehingga metrik model bisa bergeser tipis.
- Kuantil memakai `approxQuantile`, jadi nilainya perkiraan.
- Kode bandara pada bulan Oktober berupa angka, sehingga analisis bandara dan rute tidak mencakup bulan itu.
- Pembagian data acak per baris, bukan berbasis waktu, sehingga model belum diuji pada periode mendatang.
- `requirements.txt` berisi seluruh paket Jupyter dan dibuat dari satu lingkungan macOS. Versi paket terbaru mungkin perlu disesuaikan bila pemasangan gagal di sistem lain.
- Java harus terpasang terlebih dahulu. Versi yang diuji adalah Java 25.

## Penggunaan AI generatif

Claude (Anthropic) dipakai sebagai alat bantu diskusi dan peninjauan. Seluruh kode dijalankan dan hasilnya diverifikasi oleh penulis. Rincian ada di laporan (Bagian 8.2) dan lampiran individu.