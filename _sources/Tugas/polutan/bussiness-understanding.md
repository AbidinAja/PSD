# Business Understanding

## 1. Latar Belakang
Kualitas udara merupakan indikator vital bagi kesehatan masyarakat dan keberlanjutan lingkungan. Seiring dengan peningkatan mobilitas kendaraan dan aktivitas industri di Kabupaten Tuban, potensi fluktuasi konsentrasi polutan di atmosfer semakin meningkat. 

Pemantauan difokuskan pada tiga gas polutan utama:
- **Nitrogen Dioksida (NO₂):** Indikator utama emisi dari sektor transportasi dan industri.
- **Karbon Monoksida (CO):** Gas beracun hasil pembakaran bahan bakar fosil yang tidak sempurna.
- **Belerang Dioksida (SO₂):** Polutan yang umumnya berasal dari pemrosesan industri ber-sulfur tinggi maupun aktivitas geologis.

Untuk mengamati pola polusi secara objektif dan berkelanjutan, proyek ini memanfaatkan data penginderaan jauh dari satelit **Sentinel-5P** melalui *Copernicus Data Space Ecosystem*. Data tersebut diolah menjadi deret waktu (*time series*) harian untuk merekam tingkat polusi di Kabupaten Tuban sejak Agustus 2025.

## 2. Rumusan Masalah
- Bagaimana tren harian konsentrasi gas polutan (NO₂, CO, dan SO₂) di wilayah Kabupaten Tuban?
- Apakah terdapat pola musiman (*seasonality*), tren jangka panjang, atau lonjakan ekstrem (*anomaly*) pada tingkat polusi udara di wilayah tersebut?

## 3. Tujuan Proyek
- **Otomatisasi Ekstraksi Data:** Mengembangkan alur pengolahan otomatis untuk mentransformasi citra satelit format spasial (NetCDF) menjadi dataset tabular (CSV) yang siap dianalisis.
- **Analisis Temporal (EDA):** Mengidentifikasi dinamika, pola distribusi, dan tren perubahan polutan udara dari waktu ke waktu.
- **Persiapan Pemodelan:** Menyediakan fondasi data historis yang bersih untuk kebutuhan analisis prediktif (*time series forecasting*) kualitas udara.

## 4. Manfaat Proyek
- **Pemerintah & Pembuat Kebijakan:** Menyediakan *data-driven insight* sebagai acuan perumusan kebijakan lingkungan, pengawasan emisi industri, dan tata ruang.
- **Masyarakat:** Memberikan edukasi transparan mengenai fluktuasi kualitas udara harian di daerah sekitar.
- **Akademisi & Data Scientist:** Menjadi studi kasus terapan mengenai ekstraksi data spasial beresolusi tinggi menjadi variabel *time series*.