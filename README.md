# Personal Project: Customer Churn Prediction

## Daftar Isi
1.  [Ringkasan Proyek](#-ringkasan-proyek)
2.  [Struktur & Alur Kerja](#-struktur--alur-kerja)
3.  [Penjelasan Modul & Logika Teknis](#-penjelasan-modul--logika-teknis)
    * [1. Data Loading & Cleaning](#1-data-loading--cleaning)
    * [2. Exploratory Data Analysis (EDA)](#2-exploratory-data-analysis-eda)
    * [3. Preprocessing Data (Fitur Utama)](#3-preprocessing-data-fitur-utama)
    * [4. Modeling](#4-modeling)
4.  [Evaluasi & Hasil](#-evaluasi--hasil)
5.  [Kesimpulan & Rekomendasi](#-kesimpulan--rekomendasi)

---

## Ringkasan Project
Project ini bertujuan untuk membangun model *Machine Learning* menggunakan **Logistic Regression** untuk memprediksi apakah seorang pelanggan akan berhenti berlangganan (*Churn*) atau tidak. Dataset yang digunakan adalah data telekomunikasi (*Telco Customer*) yang mencakup demografi, layanan yang digunakan, dan informasi akun.

**Metrik Utama:** Fokus pada **Recall** (untuk menangkap sebanyak mungkin pelanggan yang berpotensi churn) dengan tetap menjaga **Akurasi** yang wajar.

---

## Struktur & Alur Kerja
Alur kerja project mengikuti standar *Data Science Life Cycle* untuk mencegah kebocoran data (*data leakage*):

1.  **Persiapan Data:** Pembersihan data mentah.
2.  **Splitting Data:** Memisahkan data latih (*Train*) dan uji (*Test*) di awal.
3.  **EDA:** Menganalisis pola dan distribusi data.
4.  **Preprocessing:** Imputasi, penanganan *outlier*, *encoding*, dan *scaling*.
5.  **Resampling:** Menangani ketidakseimbangan kelas (*imbalance*) hanya pada data latih.
6.  **Modeling:** Melatih algoritma Klasifikasi.
7.  **Evaluasi:** Mengukur performa model pada data uji.

---

## Penjelasan Modul & Logika Teknis

### 1. Data Loading & Cleaning
Modul ini bertugas membersihkan data mentah agar siap dianalisis.
* **Fitur:**
    * Menghapus kolom yang tidak relevan (`customerID`, `UpdatedAt`).
    * Menghapus duplikasi data berdasarkan ID Pelanggan (mengambil data terbaru berdasarkan periode).
    * Standarisasi nilai kategorikal yang tidak konsisten (Contoh: "Wanita" & "Laki-Laki" diubah menjadi format standar "Female" & "Male").
* **Logika:** Validitas ID pelanggan dicek untuk memastikan satu pelanggan hanya diwakili oleh satu baris data unik.

### 2. Exploratory Data Analysis (EDA)
Modul analisis untuk memahami karakteristik data.
* **Univariat Analysis:**
    * Distribusi Churn: Ditemukan **26% Churn** vs **74% Tidak Churn** (Imbalanced Dataset).
* **Bivariat Analysis:**
    * **Numerik:** Pelanggan dengan `MonthlyCharges` tinggi dan `tenure` (masa berlangganan) rendah cenderung Churn.
    * **Kategorikal:** Faktor seperti *Senior Citizen*, *Tanpa Pasangan*, dan *Paperless Billing* memiliki kecenderungan Churn lebih tinggi.
* **Deteksi Outlier:** Menggunakan *Boxplot* untuk mendeteksi pencilan pada fitur `tenure`, `MonthlyCharges`, dan `TotalCharges`.

### 3. Preprocessing Data (Fitur Utama)
Bagian paling krusial di mana data diubah menjadi format yang bisa diterima model mesin.

#### a. Handling Missing Values
* **Metode:** Imputasi menggunakan nilai **Median**.
* **Logika Penting:** Imputer dilatih (`.fit()`) hanya pada *X_train* untuk mendapatkan nilai median, lalu nilai tersebut diterapkan (`.transform()`) ke *X_train* dan *X_test*. Ini mencegah data masa depan (*test set*) mempengaruhi pengisian data latih.

#### b. Handling Outlier (Capping)
* **Metode:** *Interquartile Range* (IQR).
* **Logika:** Batas atas dan bawah dihitung berdasarkan distribusi data *Train*. Data di luar batas ini "dipotong" (*capped*) ke nilai batas terdekat. Batas yang sama dari *Train* diterapkan ke *Test*.

#### c. Encoding & Scaling
* **Encoding:** Mengubah data kategori menjadi angka menggunakan **One-Hot Encoding (OHE)**.
* **Scaling:** Menyamakan skala data numerik menggunakan **Standard Scaler** agar model Logistic Regression (yang berbasis jarak/gradien) dapat bekerja optimal.

#### d. Resampling (Handling Imbalanced Data)
* **Metode:** **SMOTE-ENN** (Kombinasi *Oversampling* minoritas dan *Undersampling* mayoritas untuk membersihkan *boundary*).
* **Penerapan:** HANYA dilakukan pada **X_train**. Data *Test* dibiarkan asli (tidak seimbang) agar evaluasi mencerminkan kondisi dunia nyata.

### 4. Modeling
* **Algoritma:** Logistic Regression.
* **Alasan:** Model yang sederhana, mudah diinterpretasikan (*explainable*), dan bekerja baik sebagai *baseline* untuk masalah klasifikasi biner.

---

## Evaluasi & Hasil

Evaluasi dilakukan menggunakan *Classification Report*, *Confusion Matrix*, dan *Learning Curve*.

### 1. Performa pada Data Uji (Testing Set)
Ini adalah indikator performa yang sebenarnya.

| Metrik | Skor (Churn Class / 1) | Keterangan |
| :--- | :--- | :--- |
| **Akurasi** | **76%** | Kemampuan umum model benar memprediksi. |
| **Recall** | **76%** | Kemampuan mendeteksi pelanggan yang *akan* Churn. |
| **Precision** | **53%** | Ketepatan saat model memprediksi Churn. |
| **F1-Score** | **62%** | Rata-rata harmonis Precision dan Recall. |

**Analisis Bisnis:**
* **Recall 76%** artinya model berhasil menangkap sebagian besar pelanggan yang berisiko pergi. Ini sangat baik untuk strategi retensi pelanggan.
* **Precision 53%** artinya ada *false alarm*; beberapa pelanggan setia mungkin terdeteksi sebagai *churner*.

### 2. Diagnosa Model (Learning Curve)
* **Training Score:** ~92%
* **Validation Score:** ~78%
* **Diagnosa:** Terdapat celah (*gap*) yang cukup lebar antara skor latihan dan validasi. Ini mengindikasikan **Overfitting** (High Variance), di mana model terlalu menghafal data latih (khususnya data hasil SMOTE).

---

## Kesimpulan & Rekomendasi

### Kesimpulan
1.  Project ini telah mengikuti metodologi *Data Science* yang benar dan valid (mencegah kebocoran data).
2.  Model Logistic Regression yang dibangun cukup agresif dalam mendeteksi Churn (Recall tinggi), namun masih memiliki tingkat kesalahan positif palsu yang moderat.
3.  Penggunaan SMOTE-ENN berhasil membantu model mengenali kelas minoritas (*Churners*).

### Rekomendasi Pengembangan
1.  **Ganti Model:** Mencoba algoritma berbasis pohon seperti **Random Forest** atau **XGBoost** yang biasanya lebih tahan terhadap outlier dan overfitting.
2.  **Feature Engineering:** Mengurangi fitur yang saling berkorelasi tinggi hasil *One-Hot Encoding* (gunakan parameter `drop='first'`).
3.  **Hyperparameter Tuning:** Melakukan *GridSearchCV* pada Logistic Regression (misal: mengatur parameter `C`) untuk mengurangi efek overfitting.
