# Polutan di Kabupaten Tuban

Selamat datang di sub-bab **Polutan di Kabupaten Tuban**. Bagian ini merupakan sebuah studi kasus Sains Data menyeluruh yang berfokus pada pemantauan dan analisis kualitas udara (khususnya gas polutan) di wilayah Kabupaten Tuban. Sebagai salah satu kawasan strategis di jalur pesisir utara (Pantura) yang menopang aktivitas industri berat—seperti pabrik semen, petrokimia, dan pembangkit listrik tenaga uap (PLTU)—pemantauan kualitas udara di Tuban menjadi isu lingkungan yang krusial.

Melalui pemanfaatan data citra satelit **Copernicus Sentinel-5P**, proyek ini melacak fluktuasi konsentrasi tiga jenis gas polutan utama yang rentan dihasilkan oleh emisi pabrik dan mobilitas transportasi angkutan berat, yaitu:
- **Nitrogen Dioksida (NO₂)**
- **Karbon Monoksida (CO)**
- **Belerang Dioksida (SO₂)**

Proyek ini mendemonstrasikan siklus utuh dari sains data (*Data Science Lifecycle*), mulai dari memahami masalah bisnis di balik polusi udara, hingga teknik mengumpulkan dan membersihkan datanya. 

Anda dapat menelusuri tahapan-tahapan proyek ini melalui halaman-halaman berikut:
1. **Business Understanding:** Membahas latar belakang, rumusan masalah, tujuan, dan manfaat mengapa analisis kualitas udara ini sangat penting untuk dilakukan, terutama di daerah dengan intensitas industri yang tinggi.
2. **Data Understanding:** Menjelaskan langkah-langkah teknis pengumpulan data (_crawling_) langsung dari satelit, penentuan bounding box wilayah (Kabupaten Tuban), hingga ekstraksi format spasial (NetCDF) menjadi dataset tabular (_Time Series_) yang siap dianalisis.