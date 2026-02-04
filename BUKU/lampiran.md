<div style="display:flex; justify-content:space-between;">

<span>⬅️ <a href="bab-5">Bab V – Memaknai Pola: Kode Pos, Layanan, dan Ruang</a></span>
<span><a href="penutup">Penutup</a> ➡️</span>

</div>

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


## Deskripsi Coding Block

Lampiran ini menyajikan **penjelasan rinci per blok koding** yang digunakan dalam proses pengolahan data geospasial. Fokusnya adalah menjelaskan **fungsi, alasan penggunaan, dan hasil** dari setiap blok kode, sehingga pembaca dapat menelusuri alur teknis secara sistematis.

---

## Logika 1 — Ekstraksi dan Penyederhanaan Data Batas Administrasi

---

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

## Logika 2 — Akusisi Atribut Kode Pos

---

### Blok 1 — Inisialisasi Library

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
```

**Penjelasan**
Blok ini memanggil library yang dibutuhkan untuk melakukan pengambilan data dari web dan mengelola tabel hasil scraping.

**Hasil**
Lingkungan Python siap digunakan untuk proses akuisisi data kode pos.

---

### Blok 2 — Penentuan Parameter Akses Website

```python
base_url = "https://kodepos.posindonesia.co.id/CariKodepos"
payload = {"kodepos": "kab.pekalongan"}
headers = {"User-Agent": "Mozilla/5.0"}
```

**Penjelasan**
Blok ini mendefinisikan alamat endpoint pencarian kode pos, parameter wilayah studi, serta header agar permintaan dikenali sebagai akses dari browser.

**Hasil**
Parameter scraping siap digunakan untuk mengambil data kode pos dari website.

---

### Blok 3 — Inisialisasi Penampung Data dan Informasi Paging

```python
all_pages = []
total_pages = 12
```

**Penjelasan**
List disiapkan untuk menampung hasil scraping setiap halaman. Variabel `total_pages` menunjukkan jumlah halaman hasil pencarian yang perlu diambil.

**Hasil**
Struktur penampung data berhasil dibuat.

---

### Blok 4 — Proses Scraping Data per Halaman

```python
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

    if table is None:
        continue

    df_page = pd.read_html(StringIO(str(table)))[0]
    all_pages.append(df_page)
```

**Penjelasan**
Blok ini melakukan pengambilan data secara berulang untuk setiap halaman hasil pencarian, mengekstrak tabel HTML, dan mengonversinya menjadi DataFrame.

**Hasil**
Data kode pos dari setiap halaman berhasil dikumpulkan dalam bentuk list DataFrame.

---

### Blok 5 — Penggabungan Seluruh Hasil Scraping

```python
df_kodepos = pd.concat(all_pages, ignore_index=True)
```

**Penjelasan**
Seluruh DataFrame hasil scraping digabungkan menjadi satu dataset utuh.

**Hasil**
Terbentuk satu DataFrame kode pos wilayah Pekalongan.

---

### Blok 6 — Pemeriksaan Awal Data

```python
print("Jumlah baris:", len(df_kodepos))
df_kodepos.head()
```

**Penjelasan**
Blok ini digunakan untuk memastikan jumlah data dan struktur kolom sesuai dengan wilayah studi.

**Hasil**
Diperoleh dataset dengan **312 baris data**.

---

### Blok 7 — Penyimpanan Data Kode Pos

```python
df_kodepos.to_csv("kodepos_pekalongan.csv", index=False)
```

**Penjelasan**
Dataset kode pos disimpan dalam format CSV agar dapat digunakan kembali tanpa melakukan scraping ulang.

**Hasil**
File `kodepos_pekalongan.csv` berhasil dibuat dan siap digunakan pada proses kompilasi data spasial.

---

## Logika 3 — Validasi dan Kompilasi Data Spasial–Atribut

---

### Blok 1 — Pemanggilan Data Atribut dan Spasial

```python
import pandas as pd
import geopandas as gpd

# data atribut kode pos
df_kodepos = pd.read_csv("kodepos_pekalongan.csv")

# data batas wilayah (geometri)
gdf_batas = gpd.read_file("bataspekalongan.geojson")
```

**Penjelasan**
Blok ini memuat dua dataset utama: data atribut kode pos (CSV) dan data batas wilayah (GeoJSON).

**Hasil**
Kedua dataset berhasil dimuat ke dalam DataFrame dan GeoDataFrame.

---

### Blok 2 — Pembentukan Kunci Gabungan Desa–Kecamatan

```python
# membuat key gabungan pada data kode pos
df_kodepos["desa_kec"] = (
    df_kodepos["desa_kelurahan"] + " | " + df_kodepos["kecamatan"]
)

# membuat key gabungan pada data batas wilayah
gdf_batas["desa_kec"] = (
    gdf_batas["desa_kelurahan"] + " | " + gdf_batas["kecamatan"]
)
```

**Penjelasan**
Blok ini membentuk kunci penggabungan berbasis kombinasi nama desa/kelurahan dan kecamatan untuk menyamakan identitas wilayah.

**Hasil**
Kolom kunci `desa_kec` berhasil dibuat pada kedua dataset.

---

### Blok 3 — Pemeriksaan Jumlah Unit Wilayah

```python
print("Kodepos :", df_kodepos["desa_kec"].nunique())
print("Batas   :", gdf_batas["desa_kec"].nunique())
```

**Penjelasan**
Blok ini memeriksa jumlah unit wilayah unik pada masing-masing dataset sebelum proses penggabungan.

**Hasil**
Jumlah unit wilayah pada kedua dataset sama, yaitu **312**.

---

### Blok 4 — Deteksi Ketidaksesuaian Data

```python
tidak_cocok_kodepos = (
    df_kodepos[["desa_kec"]]
    .drop_duplicates()
    .merge(
        gdf_batas[["desa_kec"]].drop_duplicates(),
        on="desa_kec",
        how="left",
        indicator=True
    )
    .query("_merge == 'left_only'")
)


tidak_cocok_batas = (
    gdf_batas[["desa_kec"]]
    .drop_duplicates()
    .merge(
        df_kodepos[["desa_kec"]].drop_duplicates(),
        on="desa_kec",
        how="left",
        indicator=True
    )
    .query("_merge == 'left_only'")
)
```

**Penjelasan**
Blok ini digunakan untuk mendeteksi desa–kecamatan yang hanya muncul pada salah satu dataset.

**Hasil**
Objek validasi berhasil dibuat untuk pengecekan kecocokan data.

---

### Blok 5 — Validasi Akhir Kesiapan Data

```python
if tidak_cocok_kodepos.empty and tidak_cocok_batas.empty:
    print("✅ Semua data DESA–KECAMATAN cocok antara kodepos dan batas wilayah")
```

**Penjelasan**
Blok ini memastikan tidak ada ketidaksesuaian data sebelum penggabungan dilakukan.

**Hasil**
Seluruh unit desa–kecamatan dinyatakan cocok.

---

### Blok 6 — Pemeriksaan Ketersediaan Kolom Kunci

```python
assert "desa_kec" in df_kodepos.columns
assert "desa_kec" in gdf_batas.columns
```

**Penjelasan**
Blok ini berfungsi sebagai pengaman untuk memastikan kolom kunci tersedia pada kedua dataset.

**Hasil**
Proses dapat dilanjutkan tanpa error.

---

### Blok 7 — Proses Penggabungan Data

```python
gdf_final = df_kodepos.merge(
    gdf_batas[["desa_kec", "geometry"]],
    on="desa_kec",
    how="left"
)

gdf_final = gpd.GeoDataFrame(
    gdf_final,
    geometry="geometry",
    crs=gdf_batas.crs
)
```

**Penjelasan**
Blok ini menggabungkan data atribut kode pos dengan geometri batas wilayah dan mengonversinya menjadi GeoDataFrame.

**Hasil**
Terbentuk GeoDataFrame hasil kompilasi data spasial–atribut.

---

### Blok 8 — Pemeriksaan Hasil Kompilasi

```python
print("Total baris:", len(gdf_final))
print("Geometry kosong:", gdf_final.geometry.isna().sum())
```

**Penjelasan**
Blok ini digunakan untuk memastikan seluruh baris data memiliki geometri yang valid.

**Hasil**
Jumlah baris **312** dan tidak terdapat geometri kosong.

---

### Blok 9 — Penyimpanan Data GeoJSON Final

```python
gdf_final.to_file(
    "GIS_pekalongan.geojson",
    driver="GeoJSON"
)
```

**Penjelasan**
Blok ini menyimpan hasil akhir kompilasi data ke dalam format GeoJSON.

**Hasil**
File `GIS_pekalongan.geojson` berhasil dibuat dan siap digunakan untuk analisis dan visualisasi lanjutan.

---

## Logika 4 — Representasi Titik Wilayah Menggunakan Centroid

---

### Blok 1 — Pemanggilan Library Geospasial

```python
import geopandas as gpd
from shapely.ops import transform
```

**Penjelasan**
Blok ini memanggil pustaka yang diperlukan untuk pengolahan data geospasial dan manipulasi geometri.

**Hasil**
Lingkungan pemrosesan geospasial siap digunakan.

---

### Blok 2 — Memuat Data Hasil Kompilasi

```python
# membaca data hasil kompilasi
gdf_kel = gpd.read_file("GIS_pekalongan.geojson")
```

**Penjelasan**
Blok ini memuat data geospasial hasil kompilasi kode pos dan batas wilayah yang telah dibuat pada tahap sebelumnya.

**Hasil**
Data geospasial Pekalongan berhasil dimuat ke dalam GeoDataFrame.

---

### Blok 3 — Penghapusan Dimensi Z pada Geometri

```python
# fungsi untuk menghapus dimensi Z
def drop_z(geom):
    if geom.has_z:
        return transform(lambda x, y, z=None: (x, y), geom)
    return geom

# menerapkan fungsi pada seluruh geometri
gdf_kel["geometry"] = gdf_kel.geometry.apply(drop_z)
```

**Penjelasan**
Blok ini menghilangkan dimensi Z (ketinggian) pada geometri polygon untuk mencegah kesalahan pada proses reproyeksi dan perhitungan centroid.

**Hasil**
Seluruh geometri disederhanakan menjadi bentuk dua dimensi.

---

### Blok 4 — Reproyeksi ke Sistem Koordinat UTM

```python
# reproyeksi ke sistem koordinat UTM
gdf_kel_utm = gdf_kel.to_crs(epsg=32749)
```

**Penjelasan**
Data direproyeksikan ke sistem koordinat UTM agar perhitungan centroid dilakukan dalam satuan meter, sehingga hasilnya lebih akurat secara geometris.

**Hasil**
Data geospasial berada pada sistem koordinat UTM.

---

### Blok 5 — Perhitungan Titik Centroid

```python
# menghitung centroid polygon
gdf_kel_utm["centroid"] = gdf_kel_utm.geometry.centroid
```

**Penjelasan**
Blok ini menghitung titik centroid (titik tengah) dari setiap polygon desa/kelurahan.

**Hasil**
Kolom `centroid` berhasil ditambahkan ke GeoDataFrame.

---

### Blok 6 — Menetapkan Centroid sebagai Geometri Aktif

```python
# menjadikan centroid sebagai geometri aktif dan kembali ke WGS84
gdf_centroid = (
    gdf_kel_utm
    .set_geometry("centroid")
    .to_crs(epsg=4326)
)
```

**Penjelasan**
Blok ini menetapkan titik centroid sebagai geometri utama dan mengonversi kembali sistem koordinat ke WGS84 agar kompatibel dengan peta web dan GIS umum.

**Hasil**
Terbentuk GeoDataFrame berbentuk titik dengan koordinat lintang–bujur.

---

### Blok 7 — Normalisasi Atribut Teks

```python
kolom_teks = [
    "desa_kelurahan",
    "kecamatan",
    "kota_kabupaten",
    "provinsi",
    "desa_kec"
]

for col in kolom_teks:
    if col in gdf_centroid.columns:
        gdf_centroid[col] = (
            gdf_centroid[col]
            .astype(str)
            .str.upper()
            .str.strip()
        )
```

**Penjelasan**
Blok ini menstandarkan penulisan atribut teks agar konsisten untuk keperluan visualisasi dan analisis lanjutan.

**Hasil**
Atribut teks telah dirapikan dan seragam.

---

### Blok 8 — Penyimpanan Data Titik Centroid

```python
gdf_centroid.to_file(
    "centroidPekalongan.geojson",
    driver="GeoJSON"
)
```

**Penjelasan**
Blok ini menyimpan data titik centroid ke dalam format GeoJSON.

**Hasil**
File `centroidPekalongan.geojson` berhasil dibuat dan siap digunakan untuk visualisasi dan analisis spasial sederhana.

---

## Logika 5 — Visualisasi Peta Interaktif 

---

### Blok 1 — Pemanggilan Library Visualisasi

```python
import geopandas as gpd
import folium
import random
```

**Penjelasan**
Blok ini memanggil pustaka yang digunakan untuk membaca data geospasial dan membangun peta interaktif.

**Hasil**
Lingkungan visualisasi peta siap digunakan.

---

### Blok 2 — Memuat Data Polygon dan Titik

```python
gdf_kel = gpd.read_file("GIS_pekalongan.geojson")
gdf_centroid = gpd.read_file("centroidPekalongan.geojson")

gdf_centroid = gdf_centroid.rename_geometry("centroid")
```

**Penjelasan**
Blok ini memuat data batas wilayah (polygon) dan data titik centroid. Nama kolom geometri pada data centroid diubah untuk membedakan dari geometri polygon.

**Hasil**
Data polygon dan titik siap digunakan dalam satu peta.

---

### Blok 3 — Penyamaan Sistem Koordinat (CRS)

```python
if gdf_kel.crs is None or gdf_kel.crs.to_epsg() != 4326:
    gdf_kel = gdf_kel.to_crs(epsg=4326)

if gdf_centroid.crs is None or gdf_centroid.crs.to_epsg() != 4326:
    gdf_centroid = gdf_centroid.to_crs(epsg=4326)
```

**Penjelasan**
Blok ini memastikan seluruh layer menggunakan sistem koordinat WGS84 agar dapat ditampilkan secara akurat pada peta web.

**Hasil**
Semua data berada pada sistem koordinat yang sama.

---

### Blok 4 — Menentukan Titik Pusat Peta

```python
center_lat = gdf_centroid.geometry.y.mean()
center_lon = gdf_centroid.geometry.x.mean()
```

**Penjelasan**
Titik pusat peta dihitung dari rata-rata posisi seluruh centroid wilayah.

**Hasil**
Peta terpusat secara otomatis pada wilayah studi.

---

### Blok 5 — Membuat Objek Peta Dasar

```python
m = folium.Map(
    location=[center_lat, center_lon],
    zoom_start=12,
    tiles="OpenStreetMap"
)
```

**Penjelasan**
Blok ini membuat peta dasar interaktif menggunakan Folium.

**Hasil**
Peta dasar siap diisi layer spasial.

---

### Blok 6 — Penentuan Warna Kecamatan

```python
kecamatan_list = sorted(gdf_kel["kecamatan"].unique())

warna_kecamatan = {
    kec: "#{:06x}".format(random.randint(0, 0xFFFFFF))
    for kec in kecamatan_list
}
```

**Penjelasan**
Setiap kecamatan diberi warna acak untuk memudahkan pembeda visual antarwilayah.

**Hasil**
Skema warna kecamatan berhasil dibuat.

---

### Blok 7 — Fungsi Style Batas Kelurahan

```python
def style_kelurahan(feature):
    kec = feature["properties"]["kecamatan"]
    return {
        "fillColor": warna_kecamatan.get(kec, "lightgray"),
        "color": "black",
        "weight": 1,
        "dashArray": "4,4",
        "fillOpacity": 0.35
    }
```

**Penjelasan**
Blok ini mendefinisikan gaya visual batas kelurahan.

**Hasil**
Kelurahan memiliki tampilan warna dan garis yang konsisten.

---

### Blok 8 — Fungsi Style Batas Kecamatan

```python
def style_kecamatan(feature):
    return {
        "fillColor": "transparent",
        "color": "black",
        "weight": 2,
        "fillOpacity": 0
    }
```

**Penjelasan**
Blok ini mendefinisikan gaya visual batas kecamatan sebagai batas administratif utama.

**Hasil**
Batas kecamatan tampil lebih tegas dibanding batas kelurahan.

---

### Blok 9 — Menambahkan Layer Batas Kelurahan

```python
folium.GeoJson(
    gdf_kel,
    name="Batas Kelurahan",
    style_function=style_kelurahan,
    highlight_function=lambda x: {
        "fillColor": "white",
        "color": "black",
        "weight": 2,
        "fillOpacity": 0.6
    },
    tooltip=folium.GeoJsonTooltip(
        fields=["desa_kelurahan", "kecamatan"],
        aliases=["Kelurahan:", "Kecamatan:"]
    )
).add_to(m)
```

**Penjelasan**
Blok ini menambahkan layer polygon kelurahan beserta tooltip informasi.

**Hasil**
Batas kelurahan tampil interaktif dan informatif.

---

### Blok 10 — Menambahkan Layer Batas Kecamatan

```python
gdf_kecamatan = gdf_kel.dissolve(by="kecamatan").reset_index()

folium.GeoJson(
    gdf_kecamatan,
    name="Batas Kecamatan",
    style_function=style_kecamatan,
    tooltip=folium.GeoJsonTooltip(
        fields=["kecamatan"],
        aliases=["Kecamatan:"]
    )
).add_to(m)
```

**Penjelasan**
Polygon kelurahan digabung menjadi batas kecamatan untuk memperjelas hierarki wilayah.

**Hasil**
Layer batas kecamatan tampil rapi di atas peta.

---

### Blok 11 — Menambahkan Titik Centroid Kode Pos

```python
for _, row in gdf_centroid.iterrows():
    lat = row["centroid"].y
    lon = row["centroid"].x

    folium.CircleMarker(
        location=[lat, lon],
        radius=3,
        color="blue",
        fill=True,
        fill_color="blue",
        fill_opacity=0.85,
        popup=f"""
        <b>KODE POS:</b> {row.get('kodepos','-')}<br>
        <b>KELURAHAN:</b> {row.get('desa_kelurahan','-')}<br>
        <b>KECAMATAN:</b> {row.get('kecamatan','-')}
        """
    ).add_to(m)
```

**Penjelasan**
Blok ini menampilkan titik centroid kode pos sebagai marker interaktif.

**Hasil**
Sebaran kode pos dapat dibaca secara visual pada peta.

---

### Blok 12 — Kontrol Layer dan Tampilan Peta

```python
folium.LayerControl(collapsed=False).add_to(m)
m
```

**Penjelasan**
Blok ini menambahkan kontrol layer dan menampilkan peta interaktif.

**Hasil**
Peta interaktif lengkap siap digunakan untuk eksplorasi spasial.
<img width="732" height="438" alt="image" src="https://github.com/user-attachments/assets/2a322084-eaca-4401-946b-6dc8fcf47c9d" />

<div style="display:flex; justify-content:space-between;">

<span>⬅️ <a href="bab-5">Bab V – Memaknai Pola: Kode Pos, Layanan, dan Ruang</a></span>
<span><a href="penutup">Penutup</a> ➡️</span>

</div>

