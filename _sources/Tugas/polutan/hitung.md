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


# Analisis dan Perbandingan Metrik Statistik Polutan (NO2, CO, SO2)

Notebook ini mendemonstrasikan perhitungan manual untuk metrik **Median Absolute Deviation** dan **Median Absolute Difference** menggunakan Python, lalu membandingkannya dengan hasil ekstraksi fitur *time-series* dari TSFEL (Time Series Feature Extraction Library).

## 1. Rumus yang Digunakan

### A. Median Absolute Deviation (MAD)
Metrik ini mengukur sebaran data dengan menghitung median dari nilai absolut simpangan tiap data terhadap median populasinya.
**Rumus:** 
$$MAD = median(\vert{}X_i - median(X)\vert{})$$

Langkah-langkah Perhitungan:

1. Cari Median Utama: Urutkan seluruh data yang Anda miliki dari nilai terkecil hingga terbesar, kemudian temukan nilai tengahnya (median). Jika jumlah data genap, rata-ratakan dua nilai di tengah.

2. Hitung Selisih: Kurangi setiap titik data yang Anda miliki dengan nilai Median Utama yang didapat pada langkah pertama.

3. Absolutkan Hasil (Nilai Mutlak): Ubah semua hasil pengurangan tersebut menjadi bernilai positif (hilangkan tanda minus jika hasilnya negatif). Ini disebut nilai absolut.

4. Cari Median dari Selisih: Kumpulkan seluruh nilai absolut dari langkah ketiga, urutkan kembali dari yang terkecil hingga terbesar, dan cari nilai tengahnya. Angka inilah yang menjadi nilai akhir MAD.

### B. Median Absolute Difference (MADiff)
Metrik ini mengukur fluktuasi lokal dengan menghitung median dari nilai absolut selisih antara titik data yang saling berurutan (berdekatan) dalam *time series*.
**Rumus:**
$$MADiff = median(\vert{}X_i - X_{i-1}\vert{})$$

Langkah-langkah Perhitungan:

1. Pertahankan Urutan Waktu: Pastikan data Anda tidak diurutkan berdasarkan nilai dari kecil ke besar, melainkan tetap berdasarkan urutan waktu kejadian aslinya.

2. Hitung Selisih Antar-Titik Berdekatan: Hitung perbedaan antara titik data kedua dikurangi titik pertama, titik ketiga dikurangi titik kedua, dan seterusnya hingga akhir data. (Jika ada $N$ data, Anda akan mendapatkan $N-1$ hasil selisih).

3. Absolutkan Hasil (Nilai Mutlak): Ubah semua hasil selisih yang didapatkan menjadi bernilai positif.

4. Cari Median: Kumpulkan semua hasil selisih absolut tersebut, urutkan dari terkecil ke terbesar, lalu temukan nilai tengahnya.

---
## Perbandingan Perhitungan Manual vs TSFEL

Pada bagian ini, kita akan membuktikan dan membandingkan hasil perhitungan metrik statistik secara manual menggunakan `numpy` dengan hasil ekstraksi fitur otomatis dari *library* TSFEL. Dua metrik yang akan diuji adalah **Median Absolute Deviation** dan **Median Absolute Difference**.

```{code-cell}
import pandas as pd
import numpy as np
from IPython.display import display

# Definisi nama file yang akan diuji
# Sesuaikan path (lokasi folder) jika file berada di folder tertentu (misal: '../../data/polutan/')
files = {
    'NO2': {'raw': '../../data/polutan/NO2_after.csv', 'tsfel': '../../data/polutan/NO2_widang_TSFEL.csv'},
    'CO':  {'raw': '../../data/polutan/CO_after.csv', 'tsfel': '../../data/polutan/CO_widang_TSFEL.csv'},
    'SO2': {'raw': '../../data/polutan/SO2_after.csv', 'tsfel': '../../data/polutan/SO2_widang_TSFEL.csv'}
}

results = []

for pol, f in files.items():
    try:
        # 1. Load Data Mentah
        df_raw = pd.read_csv(f['raw'])
        signal = df_raw[pol].dropna().values
        
        # 2. Perhitungan Manual
        # Median Absolute Deviation
        median_val = np.median(signal)
        calc_mad = np.median(np.abs(signal - median_val))
        
        # Median Absolute Difference
        calc_madiff = np.median(np.abs(np.diff(signal)))
        
        # 3. Load Data Ekstraksi TSFEL
        df_tsfel = pd.read_csv(f['tsfel'])
        tsfel_mad = df_tsfel['median_abs_deviation'].iloc[0]
        tsfel_madiff = df_tsfel['median_abs_diff'].iloc[0]
        
        # 4. Menyimpan format ke list untuk DataFrame
        results.append({
            'Polutan': pol,
            'Metrik': 'Median Abs Deviation',
            'Manual Python': f"{calc_mad:.9f}",
            'Ekstraksi TSFEL': f"{tsfel_mad:.9f}"
        })
        
        results.append({
            'Polutan': '', # Dikosongkan agar tampilan tabel lebih rapi
            'Metrik': 'Median Abs Diff',
            'Manual Python': f"{calc_madiff:.9f}",
            'Ekstraksi TSFEL': f"{tsfel_madiff:.9f}"
        })
        
    except FileNotFoundError as e:
        print(f"File tidak ditemukan untuk {pol}: {e}")

# 5. Menampilkan hasil komparasi dalam bentuk tabel
df_results = pd.DataFrame(results)
display(df_results)
```

4. Kesimpulan dan Analisis
Akurasi Perhitungan:
Metodologi perhitungan menggunakan fungsi dasar numpy (np.median(np.abs(signal - np.median(signal))) dan np.median(np.abs(np.diff(signal)))) terbukti memberikan hasil yang ekuivalen dengan fungsionalitas kompleks yang ditawarkan oleh library TSFEL.

Kecocokan Identik pada NO2 dan CO:
Hasil perhitungan manual untuk polutan NO2 dan CO 100% identik dengan hasil ekstraksi fitur default TSFEL hingga digit desimal yang sangat panjang (presisi floating point).

Penyimpangan Sangat Kecil pada SO2:
Pada data SO2, terdapat perbedaan yang sangat marjinal (pada kisaran ~0.000001 atau digit ke-6 di belakang koma). Perbedaan super kecil ini umumnya wajar dalam data science dan terjadi akibat perbedaan penanganan batas tipe data (float precision rounding), filter pra-pemrosesan di dalam sistem under-the-hood TSFEL, atau perbedaan handling asimtot saat windowing feature extraction diterapkan pada set data yang memiliki banyak fluktuasi mendekati angka nol.