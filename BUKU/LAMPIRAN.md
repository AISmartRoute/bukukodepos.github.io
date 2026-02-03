# LAMPIRAN

## Penjelasan Coding

Lampiran ini berfungsi sebagai pelengkap teknis yang menjelaskan logika, tujuan, dan alur setiap proses koding yang digunakan dalam buku ini. Seluruh penjelasan disusun untuk membantu pembaca memahami *mengapa* suatu langkah dilakukan, bukan hanya *bagaimana* kodenya ditulis.

---

## Logika 1 — Ekstraksi dan Penyederhanaan Data Batas Administrasi

### Tujuan

Mengambil data batas administrasi desa/kelurahan dari dataset geospasial nasional berskala besar, kemudian menyederhanakannya agar relevan dengan wilayah studi.

### Penjelasan Logika

Data batas wilayah diperoleh dari dataset **RBI10K** dalam format **Geodatabase (GDB)** yang mencakup seluruh wilayah Indonesia. Karena cakupannya sangat luas, dilakukan beberapa tahap awal:

1. **Identifikasi layer data**
   Struktur GDB dibaca untuk mengetahui layer administrasi yang tersedia.

2. **Pembacaan layer administrasi desa/kelurahan**
   Layer batas desa dibaca ke dalam GeoDataFrame agar dapat diproses secara spasial.

3. **Penyaringan wilayah studi**
   Data difilter berdasarkan nama kabupaten/kota sehingga hanya wilayah Pekalongan yang diproses.

4. **Pemeriksaan konsistensi atribut**
   Dilakukan pengecekan kesamaan nama desa, keberadaan data duplikat, serta kelengkapan kolom kecamatan.

5. **Penyederhanaan atribut**
   Hanya kolom yang relevan (desa, kecamatan, kabupaten, provinsi, dan geometri) yang dipertahankan.

### Output

* File GeoJSON batas administrasi Pekalongan (lengkap)
* File GeoJSON versi bersih yang siap digabung dengan data non-spasial

---

## Logika 2 — Akuisisi Data Atribut Kode Pos

### Tujuan

Membangun data atribut kode pos yang bersumber dari web resmi dan dapat dipetakan secara spasial.

### Penjelasan Logika

Data kode pos diperoleh melalui proses *web scraping* dari situs resmi Pos Indonesia, dengan tahapan:

1. **Pengambilan data bertahap (paging)**
   Sistem mengakses beberapa halaman hasil pencarian untuk memastikan seluruh data wilayah terambil.

2. **Ekstraksi tabel HTML**
   Tabel pada setiap halaman dikonversi menjadi struktur tabular.

3. **Penggabungan seluruh halaman**
   Seluruh hasil scraping digabung menjadi satu dataset utuh.

4. **Penyimpanan data atribut**
   Data disimpan dalam format CSV sebagai data non-spasial utama.

### Output

* File CSV kode pos wilayah Pekalongan
* Dataset atribut yang siap dicocokkan dengan data batas wilayah

---

## Logika 3 — Validasi dan Kompilasi Data Spasial–Atribut

### Tujuan

Menggabungkan data kode pos dengan batas wilayah secara akurat dan terverifikasi.

### Penjelasan Logika

Untuk menghindari kesalahan spasial, dilakukan validasi berlapis:

1. **Pembuatan kunci gabungan**
   Kolom gabungan desa–kecamatan dibentuk pada kedua dataset.

2. **Pengecekan kecocokan data**
   Dipastikan tidak ada unit wilayah yang hilang atau berlebih.

3. **Proses penggabungan (merge)**
   Data atribut kode pos digabungkan dengan geometri batas wilayah.

4. **Validasi hasil kompilasi**
   Dicek apakah seluruh baris memiliki geometri yang valid.

### Output

* GeoJSON final berisi kode pos, atribut wilayah, dan geometri batas administrasi

---

## Logika 4 — Representasi Titik Wilayah Menggunakan Centroid

### Tujuan

Menyediakan representasi titik wilayah tanpa bergantung pada layanan geocoding berbayar.

### Penjelasan Logika

Pendekatan berbasis centroid digunakan dengan tahapan:

1. Penghapusan dimensi Z pada geometri
2. Transformasi sistem koordinat ke UTM
3. Perhitungan titik centroid setiap polygon
4. Konversi kembali ke sistem koordinat geografis (WGS84)
5. Normalisasi atribut teks

### Output

* GeoDataFrame titik centroid
* Data koordinat untuk visualisasi dan analisis spasial sederhana

---

## Logika 5 — Visualisasi Peta Interaktif

### Tujuan

Menyajikan data geospasial dalam bentuk peta interaktif untuk membantu pembacaan pola ruang.

### Penjelasan Logika

Visualisasi dilakukan dengan:

* Memastikan kesamaan sistem koordinat (CRS)
* Menentukan pusat peta secara otomatis
* Mewarnai wilayah berdasarkan kecamatan
* Menampilkan polygon dan titik secara bersamaan
* Menyediakan *tooltip*, *popup*, dan kontrol layer

Pendekatan ini bersifat eksploratif dan bertujuan memudahkan pembaca memahami relasi spasial antara wilayah dan kode pos.


## Penjelasan Coding (Detail Blok Kode)

Lampiran ini menyajikan **penjelasan rinci per blok koding** yang digunakan dalam proses pengolahan data geospasial. Fokusnya adalah menjelaskan **fungsi, alasan penggunaan, dan hasil** dari setiap blok kode, sehingga pembaca dapat menelusuri alur teknis secara sistematis.

---

## Logika 1 — Ekstraksi dan Penyederhanaan Data Batas Administrasi

### Blok 1 — Membaca Struktur Geodatabase (GDB)

```python
import fiona

gdb_path = "RBI10K_ADMINISTRASI_DESA_20230928.gdb"
fiona.listlayers(gdb_path)
```

**Penjelasan:**
Blok ini digunakan untuk membaca struktur internal file Geodatabase (GDB) hasil unduhan RBI10K. Tujuannya adalah mengetahui **layer apa saja** yang tersedia sebelum data dimuat lebih lanjut.

**Hasil:**
Diperoleh daftar layer, antara lain:

```
['ADMINISTRASI_AR_DESAKEL']
```

Layer ini berisi batas administrasi desa/kelurahan.

---

### Blok 2 — Membaca Layer Administrasi ke GeoDataFrame

```python
import geopandas as gpd

layer = "ADMINISTRASI_AR_DESAKEL"

gdf = gpd.read_file(
    "RBI10K_ADMINISTRASI_DESA_20230928.gdb",
    layer=layer
)
```

**Penjelasan:**
Layer batas desa/kelurahan dibaca ke dalam **GeoDataFrame** agar dapat diproses menggunakan Python. Peringatan yang muncul menandakan ukuran data besar dan kompleksitas geometri.

**Hasil:**
Seluruh batas administrasi desa/kelurahan Indonesia berhasil dimuat.

---

### Blok 3 — Penyaringan Wilayah Studi (Kabupaten Pekalongan)

```python
gdf_pekalongan = gdf[
    gdf["WADMKK"].str.contains("Pekalongan", case=False, na=False)
]
```

**Penjelasan:**
Data difilter berdasarkan kolom kabupaten/kota (`WADMKK`) agar hanya wilayah Pekalongan yang diproses.

**Hasil:**
Diperoleh data wilayah Pekalongan dengan **312 baris** dan **27 kolom**.

---

### Blok 4 — Pemeriksaan Struktur Kolom

```python
list(gdf_pekalongan.columns)
```

**Penjelasan:**
Menampilkan seluruh nama kolom untuk memahami struktur atribut dan menentukan kolom yang relevan.

---

### Blok 5 — Pemeriksaan Konsistensi Nama Desa

```python
(gdf_pekalongan["WADMKD"] == gdf_pekalongan["NAMOBJ"]).all()
```

**Penjelasan:**
Memastikan konsistensi antara kolom kode administrasi desa dan nama objek wilayah.

**Hasil:**
`True`, menandakan data konsisten.

---

### Blok 6 — Pemeriksaan Duplikasi Nama Desa

```python
gdf_pekalongan["NAMOBJ"].duplicated().sum()
```

**Penjelasan:**
Menghitung jumlah nama desa/kelurahan yang muncul lebih dari satu kali.

**Hasil:**
Terdapat **11** nama desa yang duplikat.

---

### Blok 7 — Pemeriksaan Kelengkapan Data Kecamatan

```python
gdf_pekalongan["WADMKC"].isna().any()
```

**Penjelasan:**
Memastikan kolom kecamatan tidak memiliki nilai kosong karena digunakan sebagai kunci penggabungan.

**Hasil:**
`False`, tidak ada data kecamatan yang kosong.

---

### Blok 8 — Penyimpanan Data Batas Pekalongan (Lengkap)

```python
gdf_pekalongan.reset_index(drop=True).to_file(
    "bataspekalongan_all.geojson",
    driver="GeoJSON"
)
```

**Penjelasan:**
Menyimpan seluruh data batas wilayah Pekalongan sebagai arsip spasial lengkap.

---

### Blok 9 — Penyederhanaan Atribut

```python
gdf_pekalonganClean = gdf_pekalongan[
    ["NAMOBJ", "WADMKC", "WADMKK", "WADMPR", "geometry"]
].rename(columns={
    "NAMOBJ": "desa_kelurahan",
    "WADMKC": "kecamatan",
    "WADMKK": "kota_kabupaten",
    "WADMPR": "provinsi"
})
```

**Penjelasan:**
Menyederhanakan data dengan memilih kolom relevan dan menyeragamkan nama kolom.

---

### Blok 10 — Penyimpanan Data Clean

```python
gdf_pekalonganClean.reset_index(drop=True).to_file(
    "pekalonganClean.geojson",
    driver="GeoJSON"
)
```

**Hasil:**
File `pekalonganClean.geojson` siap digunakan untuk kompilasi data.

---

## Logika 2 — Akuisisi Data Atribut Kode Pos

### Blok 1 — Inisialisasi Library

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
```

**Penjelasan:**
Menyiapkan pustaka untuk pengambilan data web dan pengolahan tabel.

---

### Blok 2 — Parameter Akses Website

```python
base_url = "https://kodepos.posindonesia.co.id/CariKodepos"
payload = {"kodepos": "kab.pekalongan"}
headers = {"User-Agent": "Mozilla/5.0"}
```

**Penjelasan:**
Menentukan endpoint pencarian, parameter wilayah, dan header agar permintaan diterima server.

---

### Blok 3 — Scraping Bertahap per Halaman

```python
all_pages = []
total_pages = 12

for page in range(1, total_pages + 1):
    params = {"page": page}
    resp = requests.post(
        base_url,
        data=payload,
        headers=headers,
        params=params
    )

    soup = BeautifulSoup(resp.text, "html.parser")
    table = soup.find("table")
    df_page = pd.read_html(StringIO(str(table)))[0]
    all_pages.append(df_page)
```

**Penjelasan:**
Mengambil data kode pos dari setiap halaman hasil pencarian dan menyimpannya sementara.

---

### Blok 4 — Penggabungan & Penyimpanan Data

```python
df_kodepos = pd.concat(all_pages, ignore_index=True)
df_kodepos.to_csv("kodepos_pekalongan.csv", index=False)
```

**Hasil:**
Terbentuk dataset kode pos Pekalongan dengan **312 baris**.

---

## Logika 3 — Kompilasi Data Spasial–Atribut

### Blok — Penggabungan Data

```python
gdf_final = gdf_spasial.merge(
    df_kodepos[["kelurahan_clean", "kecamatan_clean", "kode_pos"]],
    on=["kelurahan_clean", "kecamatan_clean"],
    how="left"
)
```

**Penjelasan:**
Menggabungkan data kode pos ke data batas wilayah berdasarkan kunci desa–kecamatan.

---

## Logika 4 — Pembentukan Titik Centroid

```python
gdf_utm = gdf_kelurahan.to_crs(epsg=32749)
gdf_utm["centroid"] = gdf_utm.geometry.centroid
gdf_centroid = gdf_utm.set_geometry("centroid").to_crs(epsg=4326)
```

**Penjelasan:**
Mengubah representasi polygon menjadi titik centroid untuk mempermudah pembacaan sebaran.

---

## Logika 5 — Visualisasi Peta

```python
import folium

m = folium.Map(location=[center_lat, center_lon], zoom_start=12)
```

**Penjelasan:**
Membangun peta interaktif sebagai alat eksplorasi pola spasial.

---

Lampiran ini dimaksudkan sebagai rujukan teknis. Pembaca tidak diwajibkan menjalankan seluruh kode, namun diharapkan dapat memahami **alur logika** di balik setiap langkah pemrosesan data.
