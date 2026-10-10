---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

# Data Preprocessing dan Ekstraksi Fitur

## Preprocessing: Pemeriksaan Tanggal, Penanganan Outlier, dan Interpolasi

Notebook `data/polutan/polutan.ipynb` mengolah deret waktu harian untuk tiga polutan: karbon monoksida (CO), sulfur dioksida (SO2), dan nitrogen dioksida (NO2). Data yang dipakai pada tahap pemeriksaan dan preprocessing adalah `CO_Timeseries.csv`, `SO2_Timeseries.csv`, dan `NO2_Timeseries.csv` di folder `data/polutan/`. Setiap berkas berisi kolom `date` dan nilai polutan yang bersesuaian.

Notebook terlebih dahulu menguji kelengkapan tanggal pada rentang yang ditetapkan, 31 Agustus 2025 sampai 31 Agustus 2026. Hasil yang tersimpan di notebook melaporkan tanggal `2026-08-31` tidak ditemukan. Pemeriksaan ini hanya mendeteksi tanggal yang hilang; kode tersebut tidak menambahkan baris tanggal maupun mengisi nilainya. Karena itu, interpolasi nilai di bawah ini tidak dengan sendirinya membuat rentang tanggal menjadi lengkap.

Outlier dideteksi menggunakan Rentang Interkuartil (IQR). Nilai yang berada di bawah `Q1 - 1.5 * IQR` atau di atas `Q3 + 1.5 * IQR` ditandai sebagai `NaN`, lalu nilai kosong diisi dengan interpolasi linier. Notebook mengulangi penghitungan IQR dan imputasi sampai tidak ada lagi nilai yang terdeteksi sebagai outlier. `bfill()` dan `ffill()` mengisi nilai kosong yang berada di ujung deret dan tidak dapat diisi oleh interpolasi linier.

Jumlah outlier awal yang tercatat dalam output notebook adalah 6 untuk CO, 17 untuk SO2, dan 2 untuk NO2. Angka ini adalah jumlah sebelum proses pembersihan iteratif, bukan jumlah outlier yang tersisa pada data hasil.

Berikut adalah tahapan preprocessing untuk setiap polutan:

### 1. Karbon Monoksida (CO)

```{code-cell}
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("../../Data/Polutan/CO_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)
df['CO'] = pd.to_numeric(df['CO'], errors='coerce')

# Hitung IQR
Q1 = df['CO'].quantile(0.25)
Q3 = df['CO'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['CO'] < lower_bound) | (df['CO'] > upper_bound)]

print("Jumlah Outlier CO (IQR):", len(outliers_iqr))
```

Visualisasi batas ambang IQR terhadap distribusi data CO:

```{code-cell}
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['CO'], label="CO", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['CO'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data CO (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar CO")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

Penanganan outlier dan pengisian nilai yang hilang untuk **CO**:

```python
# Bersihkan outlier secara iteratif dan isi nilai NaN
df['CO_filled'] = df['CO'].copy()
while True:
    Q1 = df['CO_filled'].quantile(0.25)
    Q3 = df['CO_filled'].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR

    outliers = (df['CO_filled'] < lower_bound) | (df['CO_filled'] > upper_bound)
    if not outliers.any():
        break

    df['CO_filled'] = df['CO_filled'].mask(outliers)
    df['CO_filled'] = df['CO_filled'].interpolate(method='linear').bfill().ffill()

# Simpan hasil preprocessing
df_co = pd.DataFrame({"date": df['date'], "CO": df['CO_filled']})
df_co.to_csv("../../Data/Polutan/CO_After.csv", index=False)
print("Data CO berhasil diproses dan disimpan ke CO_After.csv")
```

### 2. Sulfur Dioksida (SO2)

```{code-cell}
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("../../Data/Polutan/SO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)
df['SO2'] = pd.to_numeric(df['SO2'], errors='coerce')

# Hitung IQR
Q1 = df['SO2'].quantile(0.25)
Q3 = df['SO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['SO2'] < lower_bound) | (df['SO2'] > upper_bound)]

print("Jumlah Outlier SO2 (IQR):", len(outliers_iqr))
```

Visualisasi batas ambang IQR terhadap distribusi data SO2:

```{code-cell}
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['SO2'], label="SO2", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['SO2'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data SO2 (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar SO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

Penanganan outlier dan pengisian nilai yang hilang untuk **SO2**:

```python
# Bersihkan outlier secara iteratif dan isi nilai NaN
df['SO2_filled'] = df['SO2'].copy()
while True:
    Q1 = df['SO2_filled'].quantile(0.25)
    Q3 = df['SO2_filled'].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR

    outliers = (df['SO2_filled'] < lower_bound) | (df['SO2_filled'] > upper_bound)
    if not outliers.any():
        break

    df['SO2_filled'] = df['SO2_filled'].mask(outliers)
    df['SO2_filled'] = df['SO2_filled'].interpolate(method='linear').bfill().ffill()

# Simpan hasil preprocessing
df_so2 = pd.DataFrame({"date": df['date'], "SO2": df['SO2_filled']})
df_so2.to_csv("../../Data/Polutan/SO2_After.csv", index=False)
print("Data SO2 berhasil diproses dan disimpan ke SO2_After.csv")
```

### 3. Nitrogen Dioksida (NO2)

```{code-cell}
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

df = pd.read_csv("../../Data/Polutan/NO2_Timeseries.csv")
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)
df['NO2'] = pd.to_numeric(df['NO2'], errors='coerce')

# Hitung IQR
Q1 = df['NO2'].quantile(0.25)
Q3 = df['NO2'].quantile(0.75)
IQR = Q3 - Q1

lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR

# Filter outlier
outliers_iqr = df[(df['NO2'] < lower_bound) | (df['NO2'] > upper_bound)]

print("Jumlah Outlier NO2 (IQR):", len(outliers_iqr))
```

Visualisasi batas ambang IQR terhadap distribusi data NO2:

```{code-cell}
plt.figure(figsize=(15,5))
plt.plot(df['date'], df['NO2'], label="NO2", linewidth=1)

plt.scatter(outliers_iqr['date'], outliers_iqr['NO2'],
            color='red', marker='o', label="Outliers")

plt.axhline(upper_bound, color='orange', linestyle='dashed', label="Upper Bound (IQR)")
plt.axhline(lower_bound, color='blue',   linestyle='dashed', label="Lower Bound (IQR)")

plt.title("Deteksi Outlier Data NO2 (Metode IQR)")
plt.xlabel("Tanggal")
plt.ylabel("Kadar NO2")
plt.legend()
plt.tight_layout()
plt.xticks(
    ticks=[df['date'].iloc[0], df['date'].iloc[-1]],
    labels=[df['date'].iloc[0].strftime('%Y-%m-%d'),
            df['date'].iloc[-1].strftime('%Y-%m-%d')]
)
plt.show()
```

Penanganan outlier dan pengisian nilai yang hilang untuk **NO2**:

```python
# Bersihkan outlier secara iteratif dan isi nilai NaN
df['NO2_filled'] = df['NO2'].copy()
while True:
    Q1 = df['NO2_filled'].quantile(0.25)
    Q3 = df['NO2_filled'].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR

    outliers = (df['NO2_filled'] < lower_bound) | (df['NO2_filled'] > upper_bound)
    if not outliers.any():
        break

    df['NO2_filled'] = df['NO2_filled'].mask(outliers)
    df['NO2_filled'] = df['NO2_filled'].interpolate(method='linear').bfill().ffill()

# Simpan hasil preprocessing
df_no2 = pd.DataFrame({"date": df['date'], "NO2": df['NO2_filled']})
df_no2.to_csv("../../Data/Polutan/NO2_After.csv", index=False)
print("Data NO2 berhasil diproses dan disimpan ke NO2_After.csv")
```

### 4. Visualisasi Gabungan Setelah Preprocessing

Setelah proses penanganan *outlier* dan imputasi selesai, ketiga berkas hasil dapat divisualisasikan. Nilai polutan yang kosong atau terdeteksi sebagai outlier telah diimputasi, tetapi tanggal yang tidak tersedia tidak ditambahkan oleh kode preprocessing. Sebelum ekstraksi fitur, pastikan rentang tanggal yang diperlukan memang tersedia jika model atau analisis membutuhkan kalender harian lengkap.

```{code-cell}
import pandas as pd
import matplotlib.pyplot as plt

# Memuat data yang telah diproses
df_co = pd.read_csv("../../Data/Polutan/CO_After.csv")
df_so2 = pd.read_csv("../../Data/Polutan/SO2_After.csv")
df_no2 = pd.read_csv("../../Data/Polutan/NO2_After.csv")

df_co['date'] = pd.to_datetime(df_co['date'])
df_so2['date'] = pd.to_datetime(df_so2['date'])
df_no2['date'] = pd.to_datetime(df_no2['date'])

# Membuat subplot untuk ketiga polutan
fig, axes = plt.subplots(3, 1, figsize=(15, 10), sharex=True)

# Plot CO
axes[0].plot(df_co['date'], df_co['CO'], color='blue', linewidth=1)
axes[0].set_title('Kadar CO (Setelah Preprocessing)')
axes[0].set_ylabel('Kadar CO')
axes[0].grid(True, linestyle='--', alpha=0.6)

# Plot SO2
axes[1].plot(df_so2['date'], df_so2['SO2'], color='green', linewidth=1)
axes[1].set_title('Kadar SO2 (Setelah Preprocessing)')
axes[1].set_ylabel('Kadar SO2')
axes[1].grid(True, linestyle='--', alpha=0.6)

# Plot NO2
axes[2].plot(df_no2['date'], df_no2['NO2'], color='red', linewidth=1)
axes[2].set_title('Kadar NO2 (Setelah Preprocessing)')
axes[2].set_xlabel('Tanggal')
axes[2].set_ylabel('Kadar NO2')
axes[2].grid(True, linestyle='--', alpha=0.6)

# Menyesuaikan tampilan sumbu X
plt.xticks(
    ticks=[df_co['date'].iloc[0], df_co['date'].iloc[-1]],
    labels=[df_co['date'].iloc[0].strftime('%Y-%m-%d'),
            df_co['date'].iloc[-1].strftime('%Y-%m-%d')]
)

plt.tight_layout()
plt.show()
```

Selain divisualisasikan dalam subplot terpisah, kita juga dapat menumpuk (*overlay*) ketiga polutan dalam satu grafik untuk membandingkan fluktuasinya secara langsung. Karena skala kadar polutan mungkin berbeda, perbandingan ini difokuskan pada pengamatan pola tren perubahannya.

```{code-cell}
plt.figure(figsize=(15, 6))

# Plot ketiga polutan dalam satu axis
plt.plot(df_co['date'], df_co['CO'], color='blue', label='CO', linewidth=1, alpha=0.8)
plt.plot(df_so2['date'], df_so2['SO2'], color='green', label='SO2', linewidth=1, alpha=0.8)
plt.plot(df_no2['date'], df_no2['NO2'], color='red', label='NO2', linewidth=1, alpha=0.8)

plt.title('Perbandingan Fluktuasi Kadar CO, SO2, dan NO2 (Overlay)')
plt.xlabel('Tanggal')
plt.ylabel('Kadar Polutan')
plt.grid(True, linestyle='--', alpha=0.6)
plt.legend()

# Menyesuaikan tampilan sumbu X
plt.xticks(
    ticks=[df_co['date'].iloc[0], df_co['date'].iloc[-1]],
    labels=[df_co['date'].iloc[0].strftime('%Y-%m-%d'),
            df_co['date'].iloc[-1].strftime('%Y-%m-%d')]
)

plt.tight_layout()
plt.show()
```

## Ekstraksi Fitur Deret Waktu (Time Series)

Notebook mengekstrak fitur dari satu tabel gabungan, bukan dengan menjalankan tiga sel terpisah pada berkas `*_After.csv`. Berkas masukan yang digunakan adalah `data/polutan/Polutan_Widang_Linear.csv`, dengan kolom `date`, `CO`, `NO2`, dan `SO2`. Salinan CSV yang tersedia saat catatan ini diperbarui memiliki 366 baris, dari `2025-08-30` sampai `2026-08-30`.

### Alur ekstraksi pada notebook

1. Baca tabel gabungan dan urutkan berdasarkan `date`.
2. Proses kolom `NO2`, `SO2`, dan `CO` satu per satu.
3. Untuk setiap polutan, hitung batas IQR sebagai pemeriksaan tambahan. Nilai di luar batas ditandai sebagai `NaN`.
4. Jadikan `date` sebagai indeks, lalu isi kekosongan dengan interpolasi berbasis waktu (`method='time'`), `ffill()`, dan `bfill()`.
5. Hitung 68 fitur TSFEL untuk setiap polutan. Fungsi `extract_one` meneruskan `fs=1` hanya jika fungsi fitur tersebut menerimanya; `to_scalar` merangkum keluaran array menjadi satu nilai.
6. Awali nama setiap fitur dengan nama polutan. Hasil disimpan sebagai satu baris lebar di `data/polutan/Widang_Linier.csv`, bukan ditranspose menjadi dua kolom.

Dengan tiga polutan dan 68 fitur per polutan, hasil yang diharapkan memiliki 204 kolom fitur. Kolom contoh: `NO2_abs_energy`, `SO2_calc_mean`, dan `CO_zero_cross`.

```{code-cell}
import pandas as pd
import numpy as np
import inspect
import tsfel.feature_extraction.features as tsfel_features

# Baca tabel gabungan polutan
df = pd.read_csv('../../Data/Polutan/Polutan_Widang_Linear.csv')

df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

pollutants = ['NO2', 'SO2', 'CO']
fs = 1

FEATURE_LIST = """abs_energy auc autocorr average_power calc_centroid calc_max calc_mean
calc_median calc_min calc_std calc_var dfa distance ecdf ecdf_percentile ecdf_percentile_count
ecdf_slope entropy fundamental_frequency higuchi_fractal_dimension hist_mode human_range_energy
hurst_exponent interq_range kurtosis lempel_ziv lpcc max_frequency max_power_spectrum
maximum_fractal_length mean_abs_deviation mean_abs_diff mean_diff median_abs_deviation
median_abs_diff median_diff median_frequency mfcc mse negative_turning neighbourhood_peaks
petrosian_fractal_dimension pk_pk_distance positive_turning power_bandwidth rms skewness slope
spectral_centroid spectral_decrease spectral_distance spectral_entropy spectral_kurtosis
spectral_positive_turning spectral_roll_off spectral_roll_on spectral_skewness spectral_slope
spectral_spread spectral_variation spectrogram_mean_coeff sum_abs_diff wavelet_abs_mean
wavelet_energy wavelet_entropy wavelet_std wavelet_var zero_cross""".split()


def to_scalar(result):
    if isinstance(result, dict) and "values" in result:
        result = result["values"]
    if isinstance(result, (list, tuple, np.ndarray)):
        arr = np.asarray(result, dtype=float)
        return float(np.nanmean(arr))
    return float(result)


def extract_one(fn_name, signal, fs):
    fn = getattr(tsfel_features, fn_name)
    params = inspect.signature(fn).parameters
    if "fs" in params:
        result = fn(signal, fs)
    else:
        result = fn(signal)
    return to_scalar(result)

combined_row = {}
for pollutant in pollutants:
    df_poly = df[['date', pollutant]].copy()
    df_poly[pollutant] = pd.to_numeric(df_poly[pollutant], errors='coerce')

    Q1 = df_poly[pollutant].quantile(0.25)
    Q3 = df_poly[pollutant].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    df_poly.loc[
        (df_poly[pollutant] < lower_bound) | (df_poly[pollutant] > upper_bound),
        pollutant,
    ] = np.nan

    df_clean = df_poly.set_index('date').interpolate(method='time').ffill().bfill()
    signal_1d = df_clean[pollutant].astype(float).values

    for fn_name in FEATURE_LIST:
        feature_key = f"{pollutant}_{fn_name}"
        combined_row[feature_key] = extract_one(fn_name, signal_1d, fs)

extracted_features_final = pd.DataFrame([combined_row])
output_filename = '../../Data/Polutan/Widang_Linier.csv'
extracted_features_final.to_csv(output_filename, index=False)
print(f"Total fitur yang disimpan: {extracted_features_final.shape[1]}")
```

### Makna ringkas fitur

Daftar TSFEL yang dipakai mencakup fitur statistik, temporal, frekuensi, dan wavelet. Contohnya, `calc_mean`, `calc_std`, dan `kurtosis` menggambarkan nilai pusat, sebaran, dan bentuk distribusi; `mean_abs_diff`, `autocorr`, dan `zero_cross` menggambarkan perubahan atau pola urutan sinyal; sedangkan fitur dengan awalan `spectral_` dan `wavelet_` merangkum karakteristik frekuensi dan transformasi wavelet. Nilai keluaran spesifik dapat dibaca langsung dari kolom berawalan nama polutan pada `Widang_Linier.csv`.

### Catatan tentang rentang tanggal

Pemeriksaan tanggal pada bagian preprocessing menggunakan rentang ekspektasi `2025-08-31` sampai `2026-08-31` dan output notebook mencatat `2026-08-31` sebagai tanggal yang hilang. Sementara itu, salinan `Polutan_Widang_Linear.csv` yang ada di folder data saat ini berisi tanggal `2025-08-30` sampai `2026-08-30`. Kedua fakta ini menunjukkan rentang input perlu diperiksa sebelum hasil dipakai untuk analisis yang mensyaratkan kalender harian lengkap. Interpolasi berbasis waktu mengisi nilai pada baris yang ada; ia tidak membuat baris untuk tanggal yang tidak ada.

## Kesimpulan

Alur notebook yang didokumentasikan di sini adalah:

1. Membentuk dan memeriksa deret waktu harian CO, SO2, dan NO2.
2. Menandai outlier berdasarkan IQR dan mengimputasinya secara iteratif dengan interpolasi linier, `bfill()`, dan `ffill()` ke berkas `CO_After.csv`, `SO2_After.csv`, dan `NO2_After.csv`.
3. Membaca tabel gabungan `Polutan_Widang_Linear.csv`, menerapkan pemeriksaan IQR tambahan per polutan, dan mengisi nilai kosong dengan interpolasi waktu.
4. Menghitung 68 fitur TSFEL untuk setiap polutan dan menyimpan satu baris dengan 204 kolom fitur ke `Widang_Linier.csv`.

Tanggal yang hilang tidak otomatis dibuat oleh langkah-langkah tersebut. Pastikan kelengkapan rentang tanggal secara terpisah sebelum menggunakan hasil untuk pemodelan deret waktu.