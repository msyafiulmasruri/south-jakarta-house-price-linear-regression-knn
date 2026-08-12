# South Jakarta House Price Prediction

Project ini memprediksi harga rumah di wilayah Jakarta Selatan menggunakan dua algoritma regresi: **Linear Regression** dan **K-Nearest Neighbors (KNN) Regressor**. Alur yang dilakukan mencakup exploratory data analysis (EDA), data cleaning, encoding, identifikasi dan penanganan outlier, feature scaling, training model, evaluasi, hingga penyimpanan model untuk digunakan kembali.

## Sumber Data

Dataset yang digunakan diambil dari Kaggle:

- Dataset: [Daftar Harga Rumah](https://www.kaggle.com/datasets/wisnuanggara/daftar-harga-rumah) oleh wisnuanggara

Dataset asli berisi 1001 baris data rumah di Jakarta Selatan dengan kolom:

| Kolom | Deskripsi |
|---|---|
| HARGA | Harga rumah (target/label), dalam Rupiah |
| LT | Luas Tanah (m2) |
| LB | Luas Bangunan (m2) |
| JKT | Jumlah Kamar Tidur |
| JKM | Jumlah Kamar Mandi |
| GRS | Garasi (ADA / TIDAK ADA) |
| KOTA | Kota (isinya konstan "JAKSEL" di semua baris, tidak digunakan sebagai fitur) |

## Struktur Repository

```
Linear Regression and KNN - South Jakarta House Price Prediction/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   │   └── HARGA RUMAH JAKSEL.xlsx        # Data mentah asli
│   └── processed/
│       └── data_clean.csv                 # Data setelah cleaning dan outlier handling
│
├── notebooks/
│   └── Linear Regression and KNN - South Jakarta House Price Prediction.ipynb
│
└── models/
    ├── model_linear_regression.pkl
    ├── model_knn_regressor.pkl
    └── scaler.pkl
```

## Alur Kerja

1. **Import Library** — memuat seluruh dependency yang dibutuhkan di satu bagian awal.
2. **Load Dataset** — membaca `HARGA RUMAH JAKSEL.xlsx` dengan `header=1` karena header sebenarnya ada di baris kedua file Excel.
3. **Exploratory Data Analysis (EDA)** — statistik deskriptif, pengecekan missing value, pengecekan kolom yang tidak informatif, distribusi target, korelasi antar fitur numerik, serta visualisasi outlier lewat boxplot.
4. **Preprocessing** — kolom `KOTA` dibuang karena nilainya konstan di semua baris sehingga tidak memberi informasi apapun ke model.
5. **Encoding** — kolom `GRS` (ADA / TIDAK ADA) diubah menjadi biner (1 / 0) menggunakan mapping manual, karena hanya terdiri dari dua kategori.
6. **Identifikasi dan Handle Outlier** — outlier pada kolom `HARGA`, `LT`, dan `LB` dideteksi menggunakan metode IQR (Interquartile Range), lalu ditangani dengan capping (membatasi nilai ekstrem ke batas atas/bawah IQR) agar tidak ada baris data yang hilang.
7. **Simpan Data Bersih** — dataset yang sudah melalui seluruh tahap cleaning disimpan ke `data/processed/data_clean.csv` agar bisa dipakai ulang tanpa mengulang proses dari file Excel mentah.
8. **Dataset Splitting** — data dibagi menjadi data latih (80%) dan data uji (20%) dengan `random_state=42` agar hasil split konsisten setiap dijalankan ulang.
9. **Feature Scaling** — fitur distandarisasi menggunakan `StandardScaler`, di-fit hanya pada data latih untuk menghindari data leakage ke data uji.
10. **Training Model** — dua model dilatih: `LinearRegression` dan `KNeighborsRegressor` (dengan `n_neighbors=19`).
11. **Evaluasi Model** — performa kedua model diukur menggunakan MSE, RMSE, MAE, dan R2 Score pada data uji.
12. **Visualisasi Prediksi vs Aktual** — scatter plot antara nilai aktual dan prediksi untuk masing-masing model, dengan garis referensi prediksi sempurna.
13. **Save Model** — model dan scaler disimpan menggunakan `joblib` ke folder `models/`, agar dapat digunakan kembali tanpa training ulang.

## Hasil Evaluasi Model

| Metrik | Linear Regression | KNN Regressor (k=19) |
|---|---|---|
| MSE | 34,234,230,002,129,653,760.00 | 35,538,807,361,502,736,384.00 |
| RMSE | 5,851,002,478.39 | 5,961,443,395.81 |
| MAE | 3,850,899,289.42 | 3,993,912,542.55 |
| R2 | 0.6486 | 0.6352 |

Catatan: nilai MSE, RMSE, dan MAE berskala besar karena satuannya mengikuti target (Rupiah, dalam skala miliaran). Metrik R2 lebih relevan untuk membandingkan performa relatif antar model. Pada dataset ini, Linear Regression memberikan hasil yang sedikit lebih baik dibanding KNN.

## Cara Menjalankan

1. Clone repository ini
   ```
   git clone <url-repo-ini>
   cd "Linear Regression and KNN - South Jakarta House Price Prediction"
   ```

2. Buat virtual environment (opsional tapi disarankan)
   ```
   python -m venv venv
   venv\Scripts\activate      (Windows)
   source venv/bin/activate   (Mac/Linux)
   ```

3. Install dependency
   ```
   pip install -r requirements.txt
   ```

4. Buka notebook
   ```
   jupyter notebook notebooks/
   ```
   Jalankan seluruh cell secara berurutan dari atas ke bawah.

## Menggunakan Model yang Sudah Disimpan

Model dan scaler yang sudah disimpan dapat dimuat kembali tanpa perlu training ulang:

```python
import joblib

loaded_model = joblib.load('../models/model_linear_regression.pkl')
loaded_scaler = joblib.load('../models/scaler.pkl')

# Urutan kolom harus sama seperti X_train: LT, LB, JKT, JKM, GRS
data_baru = [[500, 300, 4, 3, 1]]
data_baru_scaled = loaded_scaler.transform(data_baru)

prediksi = loaded_model.predict(data_baru_scaled)
print(f"Prediksi harga: Rp {prediksi[0]:,.0f}")
```

## Pengembangan Lebih Lanjut

- Transformasi log pada target `HARGA` untuk menangani distribusi yang skewed.
- Hyperparameter tuning menggunakan `GridSearchCV`, terutama untuk parameter `n_neighbors` pada KNN.
- Menambahkan fitur lain yang relevan (misal lokasi spesifik, jarak ke jalan utama) untuk meningkatkan performa model.
- Mencoba model lain seperti Random Forest atau Gradient Boosting sebagai pembanding.
