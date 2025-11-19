# bukukodepos.github.io
Buku "Rahasia Dibalik Lima Angka" 

Anggota Kelompok 
1. Ady Chandra :
2. Aset Mulyadi :
3. Fajar Kurnia Rohman :
4. Riska Rafiela Muslimah :
5. Rizqi Akdam Kurnia :

**MILESTONE 1** — Pengumpulan & Kurasi Materi (Satu Kali Saja)

Tujuan: Semua referensi terkumpul sebelum mulai menulis atau ngoding.

Materi yang dikumpulkan:
1. Kode Pos
   - Struktur penomoran
   - Sumber data resmi
   - Hubungan dengan administrasi wilayah
2. SIG Dasar
   - Konsep layer, atribut, geometri
   - Jenis file spasial (GeoJSON, Shapefile, dll.)
3. Konversi Data Kode Pos ke Data Geospasial (khusus untuk persiapan Bab teknis)
   - Tutorial dasar Python dan jupyter notebook
   - Contoh penggunaan pandas, geopandas, folium
   - Penjelasan Koding blok-per-blok (sesuai catatan dosen)
4. Penggunaan Data Geospasial
   - Analisis spasial
   - Visualisasi peta
   - Studi kasus logistik / e-commerce
5. Data GeoJSON / Shapefile
   - Batas kelurahan, desa, kecamatan, kab/kota
   - Format administrasi wilayah yang diperlukan untuk join data

📌 Catatan:
- Semua pengumpulan data dilakukan di awal saja, supaya tidak bolak-balik cari sumber.

**MILESTONE 2** — Belajar & Mempraktikkan Konversi Data (Tidak diulang)

Tujuan: Memahami teknik yang nanti dijelaskan di Bab 3 buku.

Langkah-langkah:
1. Belajar dasar Python untuk geospasial
2. Mempelajari alur konversi:
   - cleaning → geocoding/koordinat → merging → export
3. Membuat contoh kecil:
   - kode pos 1 kabupaten → GeoDataFrame → peta folium , pilihan sementara pekalongan
4. Simpan screenshot / hasil koding untuk digunakan di Bab 3

📌 Catatan penting:
- Bagian “mempelajari cara mengubah data” tidak diulang di milestone lain.
- Nanti Bab 3 hanya menuliskan ulang hasil dari milestone ini, bukan mengulang kerja baru.

**MILESTONE 3** — Menulis Isi Buku (Bab 1–6)

Milestone ini fokus pada penulisan, bukan lagi mencari data atau belajar ulang.

1. Bab 1 — Latar Belakang
   - ringkas materi dari milestone 1.1 + 1.2
2. Bab 2 — Membaca Indonesia Lewat Kode Pos
   - gunakan hasil studi dari milestone 1.1
3. Bab 3 — Konversi Data Kode Pos → Geospasial
   - ambil hasil milestone 2
   - jelaskan per block kode Python, bukan membuat ulang
   - gunakan contoh nyata yang sudah kamu coba
4. Bab 4 — Analisis Visual
   - gunakan data hasil konversi (dari milestone 2)
   - buat visual di folium + chart tambahan bila perlu
   - TIDAK melakukan konversi ulang
5. Bab 5 — Penerapan
   - gunakan materi milestone 1.4 + contoh dari Bab 4
   - fokus ke narasi/logika, bukan kerja teknis baru
6. Bab 6 — Penutup
   - refleksi & literasi geospasial
   - tidak ada pekerjaan baru

📌 Penjelasan:
- Bab 3 adalah satu-satunya bagian yang memakai proses konversi data.
- Bab 4 hanya menganalisis hasilnya, bukan mengulang proses.

**MILESTONE 4** — Finalisasi Konten Teknis (Sekali Saja)

1. membuat peta final
2. memastikan kode dapat dijalankan ulang
3. menyiapkan grafik, legend, screenshot
4. merapikan GeoJSON / file pendukung

📌 Tidak ada coding baru selain finalisasi.

**MILESTONE 5** — Revisi Akhir & Layout Buku

1. proofreading
2. penyelarasan visual
3. layout PDF / EPUB
4. glosarium dan daftar pustaka
