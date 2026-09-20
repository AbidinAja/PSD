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

# Data Understanding
## Data Collection

Pada proyek ini, data polutan udara dikumpulkan dalam bentuk deret waktu harian untuk tiga variabel utama, yaitu CO, SO₂, dan NO₂. Data tersebut disimpan dalam format CSV dan dapat dibuka menggunakan library Pandas untuk dianalisis lebih lanjut.

Dataset yang digunakan dalam proyek ini adalah file yang berada di folder:

- ../../data/polutan/CO.csv
- ../../data/polutan/SO2.csv
- ../../data/polutan/NO2.csv

### Hasil CSV
Berikut adalah tampilan awal dari dataset tersebut.

1. CO

```{code-cell}
:tags: [hide-input]
import pandas as pd

df = pd.read_csv("../../data/polutan/CO.csv")
df.head(5)
```

2. SO₂

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../../data/polutan/SO2.csv")
df.head(5)
```

3. NO₂

```{code-cell}
:tags: [hide-input]
df = pd.read_csv("../../data/polutan/NO2.csv")
df.head(5)
```

---

## Eksplorasi Data
### Missing Values

_Missing values_ adalah nilai yang hilang atau tidak terisi pada dataset. Dalam data deret waktu polutan, kondisi ini bisa terjadi karena data tidak tercatat pada hari tertentu atau sensor tidak berhasil membaca konsentrasi pada waktu tertentu. Pada tahap ini, kita mengecek dua hal:

1. tanggal yang hilang dari rentang waktu
2. jumlah nilai kosong pada kolom konsentrasi polutan

#### 1. Tanggal yang Hilang

1. CO

```{code-cell}
import pandas as pd

df = pd.read_csv("../../data/polutan/CO.csv")
df['date'] = pd.to_datetime(df['date'])

start_date = df['date'].min()
end_date = df['date'].max()
full_range = pd.date_range(start=start_date, end=end_date, freq='D')

missing_dates = full_range.difference(df['date'])

print(f"Jumlah hari missing: {len(missing_dates)}")
print(missing_dates)
```

2. SO₂

```{code-cell}
import pandas as pd

df = pd.read_csv("../../data/polutan/SO2.csv")
df['date'] = pd.to_datetime(df['date'])

start_date = df['date'].min()
end_date = df['date'].max()
full_range = pd.date_range(start=start_date, end=end_date, freq='D')

missing_dates = full_range.difference(df['date'])

print(f"Jumlah hari missing: {len(missing_dates)}")
print(missing_dates)
```

3. NO₂

```{code-cell}
import pandas as pd

df = pd.read_csv("../../data/polutan/NO2.csv")
df['date'] = pd.to_datetime(df['date'])

start_date = df['date'].min()
end_date = df['date'].max()
full_range = pd.date_range(start=start_date, end=end_date, freq='D')

missing_dates = full_range.difference(df['date'])

print(f"Jumlah hari missing: {len(missing_dates)}")
print(missing_dates)
```

#### 2. Data yang Hilang

Selain tanggal, kita juga mengecek jumlah nilai `NaN` pada kolom konsentrasi polutan.

1. CO

```{code-cell}
df = pd.read_csv("../../data/polutan/CO.csv")
print("Jumlah missing value CO:", df['CO'].isna().sum())
```

2. SO₂

```{code-cell}
df = pd.read_csv("../../data/polutan/SO2.csv")
print("Jumlah missing value SO2:", df['SO2'].isna().sum())
```

3. NO₂

```{code-cell}
df = pd.read_csv("../../data/polutan/NO2.csv")
print("Jumlah missing value NO2:", df['NO2'].isna().sum())
```

---

### Outliers

_Outliers_ adalah data yang sangat jauh dari pola umum dataset. Pada data kualitas udara, outlier bisa terjadi karena sensor membaca nilai ekstrem atau adanya kejadian khusus seperti peningkatan aktivitas industri atau kondisi cuaca tertentu. Untuk mengetahui keberadaan outlier, kita menggunakan metode **Isolation Forest** dari pustaka `scikit-learn`.

Isolation Forest bekerja dengan membangun pohon keputusan acak untuk memisahkan data. Observasi yang lebih mudah dipisahkan dianggap lebih tidak biasa, sehingga dapat dideteksi sebagai outlier. Hasil prediksi `-1` menandakan data tersebut merupakan outlier.

#### 1. CO

```{code-cell}
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

df = pd.read_csv("../../data/polutan/CO.csv")
df = df.dropna(subset=['CO']).copy()
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

model = IsolationForest(contamination=0.05, random_state=42)
pred = model.fit_predict(df[['CO']])

df['anomaly'] = pred
outliers = df[df['anomaly'] == -1]

print("Jumlah outlier CO:", len(outliers))
print(outliers[['date', 'CO']].head())

plt.figure(figsize=(15, 5))
plt.plot(df['date'], df['CO'], label='CO', linewidth=1)
plt.scatter(outliers['date'], outliers['CO'], color='red', label='Outlier')
plt.title('Deteksi Outlier CO (Isolation Forest)')
plt.xlabel('Tanggal')
plt.ylabel('Kadar CO')
plt.legend()
plt.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

#### 2. SO₂

```{code-cell}
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

df = pd.read_csv("../../data/polutan/SO2.csv")
df = df.dropna(subset=['SO2']).copy()
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

model = IsolationForest(contamination=0.05, random_state=42)
pred = model.fit_predict(df[['SO2']])

df['anomaly'] = pred
outliers = df[df['anomaly'] == -1]

print("Jumlah outlier SO2:", len(outliers))
print(outliers[['date', 'SO2']].head())

plt.figure(figsize=(15, 5))
plt.plot(df['date'], df['SO2'], label='SO2', linewidth=1)
plt.scatter(outliers['date'], outliers['SO2'], color='red', label='Outlier')
plt.title('Deteksi Outlier SO2 (Isolation Forest)')
plt.xlabel('Tanggal')
plt.ylabel('Kadar SO2')
plt.legend()
plt.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

#### 3. NO₂

```{code-cell}
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

df = pd.read_csv("../../data/polutan/NO2.csv")
df = df.dropna(subset=['NO2']).copy()
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date').reset_index(drop=True)

model = IsolationForest(contamination=0.05, random_state=42)
pred = model.fit_predict(df[['NO2']])

df['anomaly'] = pred
outliers = df[df['anomaly'] == -1]

print("Jumlah outlier NO2:", len(outliers))
print(outliers[['date', 'NO2']].head())

plt.figure(figsize=(15, 5))
plt.plot(df['date'], df['NO2'], label='NO2', linewidth=1)
plt.scatter(outliers['date'], outliers['NO2'], color='red', label='Outlier')
plt.title('Deteksi Outlier NO2 (Isolation Forest)')
plt.xlabel('Tanggal')
plt.ylabel('Kadar NO2')
plt.legend()
plt.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

---

## Visualisasi Gabungan (CO, SO₂, NO₂)

Setelah proses pengecekan missing value dan outlier dilakukan, kita dapat melihat pola fluktuasi ketiga polutan secara bersamaan. Karena skala nilai ketiganya berbeda, maka dilakukan normalisasi Min-Max agar semua grafik bisa dibandingkan dalam satu sumbu yang sama.

Rumus normalisasi Min-Max adalah:

$$
X_{scaled} = \frac{X - X_{min}}{X_{max} - X_{min}}
$$

Dengan rumus ini, semua nilai akan berada pada rentang 0 sampai 1.

```{code-cell}
:tags: [hide-input]
import pandas as pd
import matplotlib.pyplot as plt

df_co = pd.read_csv("../../data/polutan/CO.csv")
df_so2 = pd.read_csv("../../data/polutan/SO2.csv")
df_no2 = pd.read_csv("../../data/polutan/NO2.csv")

df_co['date'] = pd.to_datetime(df_co['date'])
df_so2['date'] = pd.to_datetime(df_so2['date'])
df_no2['date'] = pd.to_datetime(df_no2['date'])

# Normalisasi Min-Max

df_co['CO_scaled'] = (df_co['CO'] - df_co['CO'].min()) / (df_co['CO'].max() - df_co['CO'].min())
df_so2['SO2_scaled'] = (df_so2['SO2'] - df_so2['SO2'].min()) / (df_so2['SO2'].max() - df_so2['SO2'].min())
df_no2['NO2_scaled'] = (df_no2['NO2'] - df_no2['NO2'].min()) / (df_no2['NO2'].max() - df_no2['NO2'].min())

plt.figure(figsize=(15, 6))
plt.plot(df_co['date'], df_co['CO_scaled'], label='CO', linewidth=1.5, color='blue')
plt.plot(df_so2['date'], df_so2['SO2_scaled'], label='SO2', linewidth=1.5, color='green')
plt.plot(df_no2['date'], df_no2['NO2_scaled'], label='NO2', linewidth=1.5, color='purple')

plt.title('Fluktuasi Kadar Polutan CO, SO2, dan NO2 (Setelah Normalisasi Min-Max)')
plt.xlabel('Tanggal')
plt.ylabel('Nilai Ternormalisasi')
plt.legend()
plt.grid(True, linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()
```

---

## Kesimpulan

Dari eksplorasi data, kita mengetahui bahwa dataset polutan masih mungkin mengandung:

- tanggal yang hilang
- nilai kosong (`NaN`)
- outlier yang perlu dibersihkan sebelum proses ekstraksi fitur

Tahap berikutnya adalah melakukan **preprocessing** untuk menangani missing value dan outlier, lalu menghasilkan data yang lebih bersih dan siap digunakan pada tahap ekstraksi fitur.
