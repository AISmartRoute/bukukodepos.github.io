<div style="display:flex; justify-content:space-between;">

<span>⬅️ <a href="bab-3">Bab III – Praktik Mengubah Kode Pos Menjadi Data Geospasial</a></span>
<span><a href="bab-5">Bab V – Memaknai Pola: Kode Pos, Layanan, dan Ruang</a> ➡️</span>

</div>

# BAB IV

## Ketika Peta Mulai Bercerita

Bab-bab sebelumnya telah membangun fondasi untuk memahami hubungan antara angka, ruang, dan data geospasial.

Bab 1 memperkenalkan kode pos sebagai bagian dari cerita ruang Indonesia, bukan sekadar penanda administratif.
Bab 2 membahas Sistem Informasi Geografis (SIG) sebagai kerangka untuk membaca ruang melalui data.
Bab 3 menunjukkan bagaimana kerangka tersebut diterapkan secara teknis hingga menghasilkan data geospasial yang terstruktur.

Pada akhir Bab 3, sebuah dataset telah terbentuk secara utuh. Batas wilayah, atribut administratif, dan kode pos telah terkompilasi dalam satu kesatuan data spasial.

Namun, dalam kajian geospasial, selesainya proses pengolahan data tidak serta-merta berarti selesainya proses pemahaman ruang. Data yang telah rapi masih perlu dibaca, dan pembacaan tersebut tidak cukup dilakukan melalui tabel atau struktur atribut semata. Diperlukan cara lain agar data dapat dipahami secara lebih intuitif dan analitis.

Di sinilah visualisasi spasial memegang peran penting.

Bab 4 berfokus pada tahap ketika peta tidak lagi diperlakukan sebagai hasil teknis, tetapi mulai digunakan sebagai alat untuk berpikir dan mengeksplorasi ruang. Melalui visualisasi, data geospasial diterjemahkan ke dalam bentuk yang memungkinkan pola, relasi, dan ketidakteraturan ruang menjadi terlihat.

Bab ini tidak bertujuan menyajikan peta yang rumit secara teknis, melainkan memperlihatkan bagaimana perubahan cara merepresentasikan data dapat menghasilkan cara pandang yang berbeda terhadap ruang yang sama.

---

## 4.1 Dari Data Menjadi Representasi Visual Ruang

Pada akhir Bab 3, proses kompilasi data geospasial telah menghasilkan sebuah dataset yang terstruktur dan siap digunakan. Batas wilayah kelurahan telah digabungkan dengan atribut administratif serta informasi kode pos ke dalam satu kesatuan data spasial yang konsisten dan tervalidasi.

Secara metodologis, tahap pengolahan data dapat dianggap selesai. Namun, dalam kajian geospasial, selesainya proses teknis tidak serta-merta berarti selesainya proses pemahaman ruang.

Data yang telah tersusun rapi masih berada pada level struktural: ia menyimpan informasi tentang lokasi dan identitas wilayah, tetapi belum sepenuhnya menjelaskan bagaimana wilayah-wilayah tersebut saling berhubungan atau pola apa yang muncul dari susunannya.

Untuk menjembatani kebutuhan tersebut, diperlukan proses visualisasi. Dalam konteks SIG, visualisasi tidak dipahami sebagai upaya memperindah peta, melainkan sebagai bagian dari proses berpikir analitis.

Melalui visualisasi, pola awal mulai terlihat, keteraturan maupun ketidakteraturan ruang dapat dikenali, dan asumsi terhadap data dapat diuji secara visual.

---

## 4.2 Peta sebagai Bahasa Visual dalam Sistem Informasi Geografis

Dalam kajian Sistem Informasi Geografis, peta dipahami sebagai bahasa visual—sarana untuk menyampaikan informasi spasial melalui simbol, warna, posisi, dan relasi antarobjek.

### 4.2.1 Peta sebagai Sistem Tanda

Setiap elemen visual dalam peta membawa makna. Posisi menunjukkan hubungan relatif antarobjek, bentuk geometris menekankan aspek ruang tertentu, sementara warna dan ukuran simbol membantu membedakan kategori serta hierarki visual.

### 4.2.2 Visualisasi Tidak Pernah Netral

Setiap peta merupakan hasil dari serangkaian pilihan visual yang disengaja. Pilihan tersebut memengaruhi cara data dibaca dan ditafsirkan, sehingga visualisasi tidak pernah sepenuhnya netral.

### 4.2.3 Kesesuaian antara Tujuan Analisis dan Bentuk Peta

Bentuk visualisasi harus selalu selaras dengan tujuan analisis. Tidak ada satu bentuk peta yang paling benar untuk semua kebutuhan.

### 4.2.4 Risiko Salah Tafsir dalam Visualisasi

Visualisasi yang tidak dirancang dengan cermat berpotensi menimbulkan salah tafsir. Oleh karena itu, peta perlu selalu dibaca secara kritis.

### 4.2.5 Peta sebagai Alat Eksplorasi Awal

Dalam Bab 4, peta digunakan sebagai alat eksplorasi untuk mengenali pola awal dan memunculkan pertanyaan analitis lanjutan.

---

## 4.3 Representasi Spasial Wilayah: Polygon dan Titik

Dalam kajian ini, dua bentuk representasi utama digunakan: polygon dan titik (centroid). Keduanya menekankan aspek ruang yang berbeda.

### 4.3.1 Polygon sebagai Representasi Batas Wilayah

Polygon menekankan batas dan luasan wilayah administratif, namun dapat mendominasi visual ketika membaca pola sebaran.

### 4.3.2 Titik (Centroid) sebagai Representasi Posisi

Centroid menyederhanakan wilayah menjadi satu titik pusat untuk mempermudah pembacaan sebaran dan kedekatan spasial.

### 4.3.3 Konsekuensi Analitis dari Pilihan Representasi

Pilihan representasi menentukan aspek ruang yang menjadi lebih terlihat dan aspek yang tersembunyi.

### 4.3.4 Representasi sebagai Pergeseran Sudut Pandang

Mengubah representasi berarti mengubah cara memandang data, bukan mengubah datanya sendiri.

---

## 4.4 Visualisasi Sebaran Kode Pos Berdasarkan Batas Wilayah

Visualisasi awal dilakukan dengan merepresentasikan kode pos sebagai atribut wilayah kelurahan dalam bentuk polygon.

Bagian ini berfungsi sebagai pijakan awal untuk membaca pola spasial sebelum dilakukan penyederhanaan representasi.

---

## 4.5 Dari Polygon ke Centroid: Penyederhanaan Representasi Wilayah

Penyederhanaan representasi dilakukan dengan mengubah wilayah polygon menjadi titik centroid untuk mempermudah pembacaan pola sebaran.

```python
# reproyeksi ke sistem koordinat metrik
gdf_utm = gdf_kelurahan.to_crs(epsg=32749)

# perhitungan centroid wilayah
gdf_utm["centroid"] = gdf_utm.geometry.centroid

# pengembalian ke sistem koordinat geografis
gdf_centroid = gdf_utm.set_geometry("centroid").to_crs(epsg=4326)
```

---

## 4.6 Studi Kasus: Data Kantor Pos dan Keterbatasan Geocoding

Data Kantor Pos digunakan sebagai studi kasus untuk menunjukkan keterbatasan geocoding dan alternatif pendekatan berbasis wilayah.

```python
# pengaitan data kantor pos dengan centroid wilayah
gdf_kantorpos = gdf_centroid.merge(
    df_kantorpos,
    on=["kecamatan", "kelurahan"],
    how="left"
)
```

---

## 4.7 Visualisasi Sebaran Kantor Pos Berbasis Centroid

Visualisasi dilakukan dengan menampilkan titik centroid di atas batas wilayah administratif untuk membaca pola sebaran layanan.

```python
ax = gdf_kelurahan.plot(edgecolor="gray", linewidth=0.5)
gdf_kantorpos.plot(ax=ax, color="blue", markersize=20)
```

Visualisasi ini bersifat eksploratif dan bertujuan membantu pembacaan pola, bukan menunjukkan lokasi presisi.

---

## Quiz Reflektif

1. Mengapa data yang rapi belum cukup untuk memahami ruang tanpa visualisasi?
2. Apa perbedaan membaca data dalam tabel dan melalui peta?
3. Mengapa peta disebut sebagai bahasa visual?
4. Apa konsekuensi analitis dari pilihan representasi spasial?

---

Pada tahap ini, peta tidak lagi berfungsi sebagai hasil akhir, melainkan sebagai cara untuk mulai berpikir.

Peta mulai bercerita bukan karena ia berbicara sendiri, tetapi karena kita belajar membacanya dengan lebih s
