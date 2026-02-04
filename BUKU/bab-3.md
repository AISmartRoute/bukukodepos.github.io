<div style="display:flex; justify-content:space-between;">
<span>⬅️ <a href="bab-2.md">Bab II – Kode Pos dan Wilayah</a></span>
<span><a href="bab-4.md">Bab IV – Studi Kasus</a> ➡️</span>
</div>

# BAB III  
## Membangun Ruang dari Angka: Praktik Mengubah Kode Pos Menjadi Data Geospasial

Jika dua bab sebelumnya membahas konsep ruang dan cerita di balik lima digit kode pos, maka Bab 3 menjadi titik di mana gagasan tersebut mulai dipraktikkan. Pada bab ini, kode pos tidak lagi diperlakukan sebagai deretan angka administratif, melainkan sebagai pintu masuk untuk membangun representasi ruang.

Bab ini mengajak masuk ke dapur kerja geospasial—bukan untuk memamerkan kecanggihan teknologi, melainkan untuk menunjukkan bahwa peta dapat dibangun dari data yang sangat sederhana, selama kita memahami bagaimana data tersebut disusun dan dihubungkan. Fokus utama bab ini bukan pada kompleksitas alat, tetapi pada cara berpikir: bagaimana data nonspasial dapat diberi konteks ruang.

Pendekatan yang digunakan ditulis secara bertahap dan perlahan. Penjelasan lebih menekankan alasan di balik setiap proses, bukan sekadar urutan teknis atau potongan kode. Dengan pendekatan ini, bab ini dapat diikuti tanpa latar belakang pemrograman atau Sistem Informasi Geografis (SIG) yang mendalam.

Hasil akhir dari bab ini adalah sebuah dataset geospasial yang menggabungkan batas wilayah administratif dengan informasi kode pos. Dataset tersebut dapat digunakan sebagai dasar untuk visualisasi peta maupun analisis spasial sederhana, sekaligus membuka kemungkinan pengembangan lebih lanjut di luar lingkup buku ini.

Pada praktiknya, hanya digunakan dua sumber data utama:

- Data batas wilayah kelurahan/desa dalam format GeoJSON yang disediakan oleh Badan Informasi Geospasial (BIG).
- Data kode pos yang diperoleh melalui proses *web scraping* dari situs resmi PT Pos Indonesia.

---

## 3.1 Dari Angka ke Ruang: Gambaran Umum Proses

Sebelum masuk ke baris-baris kode, penting untuk memahami apa yang sebenarnya sedang dibangun pada bab ini.

Kode pos pada dasarnya merupakan data nonspasial. Ia tidak memiliki bentuk, luas, maupun posisi geografis. Sebaliknya, data GeoJSON dari BIG merupakan data spasial, karena setiap entri merepresentasikan wilayah tertentu dalam bentuk *polygon*.

Tujuan utama bab ini adalah menautkan informasi kode pos ke wilayah administratif yang memiliki representasi ruang. Dengan demikian, setiap kelurahan pada peta tidak hanya memiliki bentuk wilayah, tetapi juga membawa informasi kode pos yang melekat padanya.

Secara sederhana, proses yang dilakukan dalam bab ini dapat diringkas ke dalam beberapa langkah berikut:

1. Mengunduh data batas wilayah kelurahan dari BIG  
2. Mengambil data kode pos dari situs Pos Indonesia menggunakan Python  
3. Menyelaraskan penulisan nama wilayah pada kedua sumber data  
4. Menggabungkan data kode pos ke dalam data GeoJSON  
5. Menyimpan hasilnya sebagai dataset geospasial baru  

Urutan ini menjadi kerangka kerja yang akan diikuti sepanjang Bab 3, dengan setiap langkah dibahas secara bertahap pada subbab berikutnya.

> **Catatan visual:**  
> Pada bagian ini idealnya disisipkan diagram alur sederhana yang memperlihatkan perpindahan dari data tabel → data spasial → peta.

---

## 3.2 Menyiapkan Lingkungan Kerja

Sebelum masuk ke baris-baris kode, ada satu hal mendasar yang perlu dipahami terlebih dahulu: di mana dan dengan apa proses ini dilakukan.

Lingkungan kerja dalam konteks bab ini tidak hanya merujuk pada perangkat lunak yang digunakan, tetapi juga pada cara membangun ruang belajar yang masuk akal dan tidak terasa menakutkan. Tujuannya bukan untuk menunjukkan konfigurasi teknis yang rumit, melainkan untuk memperkenalkan alat-alat yang membantu menjelaskan alur berpikir di balik pengolahan data geospasial.

Seluruh proses pada bab ini dilakukan menggunakan bahasa pemrograman **Python**. Pilihan ini diambil bukan karena Python merupakan satu-satunya alat yang tersedia, melainkan karena Python bersifat terbuka, relatif mudah dipelajari, dan memiliki ekosistem pustaka yang kuat untuk pengolahan data, termasuk data geospasial.

### 3.2.1 Python dan Media Kerja

Python digunakan sebagai bahasa utama untuk membaca, membersihkan, dan menggabungkan data. Agar proses eksplorasi lebih mudah diikuti, contoh kode pada bab ini dijalankan menggunakan **Jupyter Notebook**.

Jupyter Notebook memungkinkan kode dijalankan secara bertahap, dengan hasil yang dapat langsung dilihat pada setiap langkah. Pendekatan ini memudahkan proses memahami data, sekaligus memberi ruang untuk mencoba, mengubah, dan mengamati dampak dari setiap baris kode tanpa harus menjalankan seluruh program dari awal.

Bagi yang belum memiliki Python di komputer pribadi, Python dapat dipasang melalui distribusi seperti **Anaconda**, atau digunakan secara daring melalui layanan seperti **Google Colab**. Dengan pilihan ini, proses belajar tidak bergantung pada spesifikasi perangkat tertentu.

> **Catatan visual (opsional):**  
> Ilustrasi sederhana alur kerja Jupyter Notebook (kode → output → visualisasi).

---

### 3.2.2 Pustaka Python yang Digunakan

Untuk mengolah data pada bab ini, digunakan beberapa pustaka Python dengan fungsi yang saling melengkapi. Nama-nama pustaka ini mungkin terdengar teknis pada awalnya, namun tidak perlu dipahami secara mendalam sekaligus.

- **Pandas**  
  Digunakan untuk mengelola data berbentuk tabel, seperti membaca file CSV hasil *scraping* kode pos, membersihkan teks nama wilayah, serta menggabungkan data berdasarkan kolom tertentu.

- **GeoPandas**  
  Digunakan untuk membaca dan mengolah data geospasial dalam format GeoJSON. Melalui GeoPandas, data batas wilayah kelurahan dapat diperlakukan seperti tabel biasa, sambil tetap mempertahankan informasi bentuk wilayah (*polygon*).

- **Requests** dan **BeautifulSoup**  
  Digunakan untuk mengambil data kode pos dari halaman web Pos Indonesia. *Requests* berfungsi untuk mengakses halaman web, sementara *BeautifulSoup* digunakan untuk membaca struktur halaman dan mengekstrak data yang diperlukan.

Pustaka-pustaka ini dipilih karena bersifat terbuka, banyak digunakan, dan memiliki dokumentasi yang luas.

---

### 3.2.3 Gambaran Alur Kerja

Secara ringkas, peran setiap alat dalam bab ini dapat dipahami sebagai berikut:

- Python berfungsi sebagai kerangka utama pengolahan data  
- Pandas menangani data berbentuk tabel  
- GeoPandas menangani data yang memiliki representasi ruang  
- Requests dan BeautifulSoup menjembatani proses pengambilan data dari web  

Dengan lingkungan kerja ini, tahap selanjutnya dapat dimulai, yaitu pengambilan dan pengolahan data GeoJSON dari BIG serta data kode pos dari situs Pos Indonesia.
## 3.3 Dari Peta Nasional ke Wilayah yang Kita Kenal

Pada bagian sebelumnya, telah dibahas bahwa kode pos merupakan angka yang mewakili ruang. Namun agar angka tersebut benar-benar dapat “menjadi peta”, diperlukan satu elemen penting: batas wilayah yang nyata.

Batas wilayah inilah yang menjadi wadah bagi kode pos. Tanpa batas wilayah, kode pos akan tetap berada pada level administratif—angka tanpa bentuk, tanpa konteks ruang.

Pada bagian ini, digunakan data batas wilayah yang disediakan oleh Badan Informasi Geospasial (BIG). Data ini bersifat resmi, detail, dan mencakup seluruh wilayah Indonesia. Karena cakupannya sangat luas, data tersebut perlu dipilih dan disederhanakan agar sesuai dengan kebutuhan kajian.

---

### 3.3.1 Mengunduh Data RBI dari BIG: Mengapa Ukurannya Besar?

Badan Informasi Geospasial menyediakan data batas wilayah nasional melalui dataset **Rupa Bumi Indonesia (RBI)**. Dataset ini dirancang untuk berbagai keperluan pemetaan, sehingga memuat informasi yang sangat lengkap dan detail.

Sebagai konsekuensinya, ukuran data RBI dapat mencapai sekitar **2 GB** apabila seluruh wilayah Indonesia diproses sekaligus. Untuk memudahkan distribusi, data ini biasanya disediakan dalam bentuk file terkompresi (ZIP).

Setelah file ZIP diekstrak, akan diperoleh sebuah folder dengan ekstensi **`.gdb`**, yang dikenal sebagai *Geodatabase*. Geodatabase dapat dibayangkan sebagai lemari arsip digital yang di dalamnya menyimpan banyak peta dan tabel dalam satu kesatuan.

---

### 3.3.2 Melihat Isi Geodatabase: Apa yang Ada di Dalamnya?

Karena Geodatabase dapat berisi banyak jenis data, langkah pertama bukan langsung mengolahnya, melainkan memahami struktur isi data tersebut.

Untuk keperluan ini digunakan pustaka **Fiona**, yang berfungsi membaca struktur data geospasial tanpa memuat seluruh isinya ke dalam memori.

```python
import fiona

gdb_path = "RBI10K_ADMINISTRASI_DESA_20230928.gdb"
fiona.listlayers(gdb_path)
```
---

### 3.3.3 Membaca Data Batas Desa ke dalam Python

Setelah mengetahui layer yang dibutuhkan, langkah berikutnya adalah membaca data tersebut ke dalam Python agar dapat diolah lebih lanjut.

Proses ini dilakukan menggunakan pustaka **GeoPandas**, yang memungkinkan data geospasial diperlakukan seperti tabel biasa, sambil tetap mempertahankan informasi bentuk wilayah.

```python
import geopandas as gpd

layer = "ADMINISTRASI_AR_DESAKEL"

gdf = gpd.read_file(
    "RBI10K_ADMINISTRASI_DESA_20230928.gdb",
    layer=layer
)
```
### 3.3.4 Memilih Wilayah Studi: Fokus ke Kabupaten Pekalongan

Pada Bab 1 dan Bab 2, telah ditekankan pentingnya konteks lokal—wilayah yang benar-benar dikenal dan memiliki keterkaitan langsung dengan pembahasan. Oleh karena itu, data batas wilayah yang berskala nasional perlu dipersempit menjadi wilayah studi yang lebih spesifik, yaitu **Kabupaten Pekalongan**.

Pemilihan wilayah studi dilakukan berdasarkan kolom nama kabupaten/kota yang tersedia pada data administratif hasil pembacaan Geodatabase.

```python
import geopandas as gpd

gdf_pekalongan = gdf[
    gdf["WADMKK"].str.contains("Pekalongan", case=False, na=False)
]
```

Pada potongan kode di atas, fungsi `str.contains()` digunakan untuk memilih seluruh baris data yang mengandung kata *Pekalongan* pada kolom `WADMKK`. Parameter `case=False` memastikan pencarian tidak peka terhadap huruf besar–kecil, sedangkan `na=False` mencegah error akibat nilai kosong.

Dengan langkah ini, hanya data yang berkaitan dengan Kabupaten Pekalongan yang diambil dari keseluruhan data nasional. Hasilnya berupa:

* **312 baris data**, masing-masing mewakili satu desa atau kelurahan
* **27 kolom atribut**, yang mencakup informasi administratif dan spasial

Data yang semula mencakup seluruh wilayah Indonesia kini berubah menjadi data lokal yang lebih fokus, lebih ringan untuk diolah, dan lebih mudah dipahami dalam konteks kajian pada buku ini.

### 3.3.5 Membaca Data Spasial sebagai Tabel Wilayah

Meskipun data yang digunakan bersifat spasial, langkah awal untuk memahaminya tetap dilakukan dengan membaca struktur tabelnya. Pendekatan ini membantu melihat atribut apa saja yang melekat pada setiap wilayah sebelum data digunakan lebih lanjut.

```python
list(gdf_pekalongan.columns)
```

Secara umum, kolom-kolom dalam data ini dapat dipahami sebagai:

* **Nama wilayah**, yang mencakup nama desa atau kelurahan serta kecamatan
* **Identitas administratif**, seperti provinsi dan kabupaten/kota
* **Informasi tambahan**, termasuk kode administratif
* **Bentuk wilayah**, yang tersimpan pada kolom `geometry`

Setiap baris data merepresentasikan satu wilayah administratif yang memiliki identitas dan bentuk ruang yang jelas.

---

### 3.3.6 Mengecek Kerapian Data Sebelum Digunakan

Sebelum data digunakan untuk digabungkan dengan data kode pos, perlu dilakukan pengecekan awal untuk memahami karakteristik data. Pengecekan ini tidak bertujuan menilai benar atau salahnya data, melainkan untuk mengenali pola dan potensi kendala yang mungkin muncul.

Beberapa hal yang diperhatikan pada tahap ini antara lain:

* konsistensi penulisan nama desa dan kecamatan
* keberadaan nilai kosong pada kolom administratif
* kemungkinan satu nama desa muncul lebih dari satu kali karena perbedaan batas wilayah

Pemahaman terhadap karakter data pada tahap ini akan sangat membantu pada proses penggabungan data di tahap selanjutnya.

---

### 3.3.7 Menyederhanakan Data agar Mudah Digabungkan

Karena tujuan utama bab ini adalah menggabungkan batas wilayah dengan data kode pos, tidak seluruh kolom pada data spasial diperlukan. Oleh karena itu, data disederhanakan dengan hanya mempertahankan kolom-kolom yang relevan.

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

Langkah ini menghasilkan struktur data yang lebih ringkas, lebih mudah dibaca, dan siap digunakan untuk proses penggabungan dengan data kode pos.

---

### 3.3.8 Menyimpan Hasil sebagai GeoJSON

Setelah data disederhanakan, langkah terakhir pada bagian ini adalah menyimpannya ke dalam format yang dapat digunakan kembali pada proses berikutnya.

```python
gdf_pekalonganClean.reset_index(drop=True).to_file(
    "pekalonganClean.geojson",
    driver="GeoJSON"
)
```

File **`pekalonganClean.geojson`** ini menjadi peta dasar untuk bagian-bagian selanjutnya. Setiap wilayah di dalamnya telah memiliki bentuk ruang dan identitas administratif yang jelas.

Pada tahap ini terlihat bahwa membangun peta tidak dimulai dari visualisasi, melainkan dari proses memahami, memilih, dan menyiapkan data. Data nasional dipersempit menjadi data lokal, struktur tabel dirapikan, dan wilayah disiapkan agar siap menerima informasi baru.

Langkah berikutnya adalah membawa **lima digit kode pos** masuk ke dalam peta ini, yang akan dibahas pada **Bagian 3.4**.
## 3.4 Mengambil Data Kode Pos: Dari Tabel Web ke Data Siap Olah

Jika pada bagian sebelumnya telah disiapkan wadah ruang berupa batas wilayah kelurahan, maka pada bagian ini mulai dimasukkan isi ke dalam wadah tersebut. Isi yang dimaksud adalah kode pos—lima digit angka yang selama ini dikenal melalui alamat surat, tetapi jarang diperlakukan sebagai bagian dari peta.

Berbeda dengan data batas wilayah dari Badan Informasi Geospasial (BIG), data kode pos tidak tersedia dalam bentuk file unduhan seperti CSV atau GeoJSON. Informasi kode pos disajikan dalam bentuk tabel pada halaman web resmi PT Pos Indonesia. Agar data tersebut dapat diolah lebih lanjut, diperlukan langkah tambahan untuk mengekstraknya menjadi data tabular.

Pendekatan yang digunakan pada bagian ini adalah *web scraping*, yaitu proses mengambil data terstruktur dari halaman web menggunakan Python. Perlu dicatat bahwa teknik ini sangat bergantung pada struktur halaman web. Perubahan kecil pada tampilan situs dapat menyebabkan kode berhenti bekerja. Oleh karena itu, *web scraping* dipahami sebagai solusi yang bersifat pragmatis—berguna untuk kebutuhan analisis, namun bukan pendekatan ideal jangka panjang untuk pengelolaan data resmi.

---

### 3.4.1 Mengapa Perlu Web Scraping?

Web scraping digunakan pada bagian ini karena:

* data kode pos tidak disediakan sebagai file terbuka yang dapat diunduh,
* informasi kode pos ditampilkan dalam bentuk tabel HTML,
* tabel tersebut dapat dibaca dan diubah menjadi data tabular menggunakan Python.

Dengan pendekatan ini, data tidak hanya dapat diambil satu kali, tetapi juga dapat diperbarui kapan pun diperlukan dengan menjalankan ulang proses yang sama.

---

### 3.4.2 Menyiapkan Alat untuk Mengambil Data Web

Untuk melakukan web scraping, digunakan beberapa pustaka Python yang telah diperkenalkan pada Bagian 3.2. Pada tahap ini, pustaka-pustaka tersebut mulai digunakan secara praktis.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
```

Secara ringkas, peran masing-masing pustaka adalah sebagai berikut:

* `requests` bertugas mengambil halaman web,
* `BeautifulSoup` membaca dan menelusuri struktur halaman,
* `pandas` mengubah tabel HTML menjadi DataFrame,
* `StringIO` membantu membaca tabel HTML sebagai teks.

---

### 3.4.3 Mengakses Halaman Pencarian Kode Pos

Langkah berikutnya adalah menentukan alamat halaman web yang berisi data kode pos serta kata kunci wilayah yang ingin diambil.

```python
base_url = "https://kodepos.posindonesia.co.id/CariKodepos"
payload = {"kodepos": "kab.pekalongan"}
headers = {"User-Agent": "Mozilla/5.0"}
```

Pada potongan kode di atas:

* `base_url` menunjukkan alamat halaman pencarian kode pos,
* `payload` berisi kata kunci wilayah, dalam hal ini Kabupaten Pekalongan,
* `headers` digunakan agar permintaan dari Python dikenali sebagai permintaan dari peramban biasa.

Tanpa penggunaan `headers`, permintaan otomatis sering kali ditolak oleh server web.

---

### 3.4.4 Mengambil Data dari Banyak Halaman

Hasil pencarian kode pos tidak ditampilkan dalam satu halaman, melainkan terbagi ke beberapa halaman. Oleh karena itu, pengambilan data dilakukan secara berulang menggunakan perulangan.

```python
all_pages = []
total_pages = 12

for page in range(1, total_pages + 1):
    print("Mengambil data dari halaman", page)

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

Pada setiap perulangan:

* Python mengakses satu halaman hasil pencarian,
* tabel kode pos diambil dari halaman tersebut,
* tabel diubah menjadi DataFrame,
* hasilnya disimpan untuk digabungkan pada tahap berikutnya.

---

### 3.4.5 Menggabungkan Seluruh Data Kode Pos

Setelah seluruh halaman berhasil diambil, semua DataFrame digabungkan menjadi satu tabel.

```python
df_kodepos = pd.concat(all_pages, ignore_index=True)
```

Untuk memastikan data berhasil diambil, jumlah baris data dapat diperiksa sebagai berikut:

```python
print("Total data kode pos:", len(df_kodepos))
df_kodepos.head()
```

Untuk wilayah Kabupaten Pekalongan, proses ini menghasilkan **312 baris data**, yang masing-masing mewakili satu desa atau kelurahan.

---

### 3.4.6 Menyimpan Data Kode Pos sebagai File CSV

Agar data kode pos dapat digunakan kembali tanpa perlu melakukan scraping ulang, data disimpan dalam format CSV.

```python
df_kodepos.to_csv("kodepos_pekalongan.csv", index=False)
```

File CSV ini bersifat nonspasial, namun menjadi jembatan penting antara data angka kode pos dan batas wilayah yang telah disiapkan sebelumnya.

Pada bagian ini terlihat bahwa data spasial dan nonspasial sering kali berasal dari sumber yang sangat berbeda. Namun, dengan pendekatan yang tepat, tabel web dapat diubah menjadi data yang rapi dan siap diolah.

Bagian berikutnya akan membahas bagaimana data kode pos ini dipadukan dengan data batas wilayah kelurahan, sehingga lima digit angka benar-benar menjadi bagian dari ruang.
## 3.4 Mengambil Data Kode Pos: Dari Tabel Web ke Data Siap Olah

Jika pada bagian sebelumnya telah disiapkan wadah ruang berupa batas wilayah kelurahan, maka pada bagian ini mulai dimasukkan isi ke dalam wadah tersebut. Isi yang dimaksud adalah kode pos—lima digit angka yang selama ini dikenal melalui alamat surat, tetapi jarang diperlakukan sebagai bagian dari peta.

Berbeda dengan data batas wilayah dari Badan Informasi Geospasial (BIG), data kode pos tidak tersedia dalam bentuk file unduhan seperti CSV atau GeoJSON. Informasi kode pos disajikan dalam bentuk tabel pada halaman web resmi PT Pos Indonesia. Agar data tersebut dapat diolah lebih lanjut, diperlukan langkah tambahan untuk mengekstraknya menjadi data tabular.

Pendekatan yang digunakan pada bagian ini adalah *web scraping*, yaitu proses mengambil data terstruktur dari halaman web menggunakan Python. Perlu dicatat bahwa teknik ini sangat bergantung pada struktur halaman web. Perubahan kecil pada tampilan situs dapat menyebabkan kode berhenti bekerja. Oleh karena itu, *web scraping* dipahami sebagai solusi yang bersifat pragmatis—berguna untuk kebutuhan analisis, namun bukan pendekatan ideal jangka panjang untuk pengelolaan data resmi.

---

### 3.4.1 Mengapa Perlu Web Scraping?

Web scraping digunakan pada bagian ini karena:

* data kode pos tidak disediakan sebagai file terbuka yang dapat diunduh,
* informasi kode pos ditampilkan dalam bentuk tabel HTML,
* tabel tersebut dapat dibaca dan diubah menjadi data tabular menggunakan Python.

Dengan pendekatan ini, data tidak hanya dapat diambil satu kali, tetapi juga dapat diperbarui kapan pun diperlukan dengan menjalankan ulang proses yang sama.

---

### 3.4.2 Menyiapkan Alat untuk Mengambil Data Web

Untuk melakukan web scraping, digunakan beberapa pustaka Python yang telah diperkenalkan pada Bagian 3.2. Pada tahap ini, pustaka-pustaka tersebut mulai digunakan secara praktis.

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from io import StringIO
```

Secara ringkas, peran masing-masing pustaka adalah sebagai berikut:

* `requests` bertugas mengambil halaman web,
* `BeautifulSoup` membaca dan menelusuri struktur halaman,
* `pandas` mengubah tabel HTML menjadi DataFrame,
* `StringIO` membantu membaca tabel HTML sebagai teks.

---

### 3.4.3 Mengakses Halaman Pencarian Kode Pos

Langkah berikutnya adalah menentukan alamat halaman web yang berisi data kode pos serta kata kunci wilayah yang ingin diambil.

```python
base_url = "https://kodepos.posindonesia.co.id/CariKodepos"
payload = {"kodepos": "kab.pekalongan"}
headers = {"User-Agent": "Mozilla/5.0"}
```

Pada potongan kode di atas:

* `base_url` menunjukkan alamat halaman pencarian kode pos,
* `payload` berisi kata kunci wilayah, dalam hal ini Kabupaten Pekalongan,
* `headers` digunakan agar permintaan dari Python dikenali sebagai permintaan dari peramban biasa.

Tanpa penggunaan `headers`, permintaan otomatis sering kali ditolak oleh server web.

---

### 3.4.4 Mengambil Data dari Banyak Halaman

Hasil pencarian kode pos tidak ditampilkan dalam satu halaman, melainkan terbagi ke beberapa halaman. Oleh karena itu, pengambilan data dilakukan secara berulang menggunakan perulangan.

```python
all_pages = []
total_pages = 12

for page in range(1, total_pages + 1):
    print("Mengambil data dari halaman", page)

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

Pada setiap perulangan:

* Python mengakses satu halaman hasil pencarian,
* tabel kode pos diambil dari halaman tersebut,
* tabel diubah menjadi DataFrame,
* hasilnya disimpan untuk digabungkan pada tahap berikutnya.

---

### 3.4.5 Menggabungkan Seluruh Data Kode Pos

Setelah seluruh halaman berhasil diambil, semua DataFrame digabungkan menjadi satu tabel.

```python
df_kodepos = pd.concat(all_pages, ignore_index=True)
```

Untuk memastikan data berhasil diambil, jumlah baris data dapat diperiksa sebagai berikut:

```python
print("Total data kode pos:", len(df_kodepos))
df_kodepos.head()
```

Untuk wilayah Kabupaten Pekalongan, proses ini menghasilkan **312 baris data**, yang masing-masing mewakili satu desa atau kelurahan.

---

### 3.4.6 Menyimpan Data Kode Pos sebagai File CSV

Agar data kode pos dapat digunakan kembali tanpa perlu melakukan scraping ulang, data disimpan dalam format CSV.

```python
df_kodepos.to_csv("kodepos_pekalongan.csv", index=False)
```

File CSV ini bersifat nonspasial, namun menjadi jembatan penting antara data angka kode pos dan batas wilayah yang telah disiapkan sebelumnya.

Pada bagian ini terlihat bahwa data spasial dan nonspasial sering kali berasal dari sumber yang sangat berbeda. Namun, dengan pendekatan yang tepat, tabel web dapat diubah menjadi data yang rapi dan siap diolah.

Bagian berikutnya akan membahas bagaimana data kode pos ini dipadukan dengan data batas wilayah kelurahan, sehingga lima digit angka benar-benar menjadi bagian dari ruang.
## 3.5 Mengompilasi Data: Menyatukan Ruang dan Angka

Pada bagian-bagian sebelumnya, telah disiapkan dua jenis data yang berbeda sifatnya. Data batas wilayah kelurahan dalam format GeoJSON memiliki representasi ruang berupa polygon, sedangkan data kode pos dalam format CSV hanya berisi informasi angka dan nama wilayah.

Pada bagian inilah kedua data tersebut disatukan. Proses ini menjadi titik penting ketika kode pos—yang sejak awal hanya berupa deretan angka—akhirnya melekat pada wilayah yang nyata dan dapat dipetakan.

---

### 3.5.1 Mengapa Data Perlu Dikompilasi?

Secara sederhana, kompilasi data berarti menggabungkan dua sumber informasi ke dalam satu struktur data yang utuh.

* Data GeoJSON dari BIG menjelaskan **di mana** sebuah wilayah berada dan bagaimana bentuknya.
* Data kode pos menjelaskan **angka identitas** yang digunakan pada wilayah tersebut.

Tanpa proses kompilasi, kita hanya akan memiliki peta tanpa kode pos, atau daftar kode pos tanpa konteks ruang. Melalui kompilasi, peta mulai “berbicara”: setiap wilayah tidak hanya memiliki bentuk, tetapi juga identitas angka yang dapat dianalisis dan divisualisasikan.

---

### 3.5.2 Memuat Kembali Kedua Dataset

Langkah pertama adalah memuat kembali dua data yang telah disiapkan sebelumnya, yaitu file GeoJSON hasil penyederhanaan data BIG dan file CSV hasil pengambilan data kode pos.

```python
import geopandas as gpd
import pandas as pd

# memuat data spasial
gdf_spasial = gpd.read_file("pekalonganClean.geojson")

# memuat data kode pos
df_kodepos = pd.read_csv("kodepos_pekalongan.csv")
```

Pada tahap ini, `gdf_spasial` berisi batas wilayah kelurahan beserta atribut administratifnya, sedangkan `df_kodepos` berisi data kode pos tanpa informasi geometri.

---

### 3.5.3 Menyamakan Penulisan Nama Wilayah

Meskipun kedua dataset memuat informasi kelurahan dan kecamatan, penulisan nama wilayah tidak selalu identik. Perbedaan huruf besar–kecil, spasi, atau variasi penulisan kecil lainnya dapat menyebabkan proses penggabungan gagal.

Oleh karena itu, sebelum data digabungkan, nama wilayah perlu diseragamkan.

```python
gdf_spasial["kelurahan_clean"] = (
    gdf_spasial["desa_kelurahan"]
    .str.lower()
    .str.strip()
)

gdf_spasial["kecamatan_clean"] = (
    gdf_spasial["kecamatan"]
    .str.lower()
    .str.strip()
)

# pada data kode pos
df_kodepos["kelurahan_clean"] = (
    df_kodepos["kelurahan"]
    .str.lower()
    .str.strip()
)

df_kodepos["kecamatan_clean"] = (
    df_kodepos["kecamatan"]
    .str.lower()
    .str.strip()
)
```

Langkah ini terlihat sederhana, namun sangat menentukan keberhasilan kompilasi data. Dalam praktik pengolahan data, persoalan sering kali muncul bukan pada teknologi, melainkan pada ketidakkonsistenan penulisan.

---

### 3.5.4 Menggabungkan Data Kode Pos ke Data Spasial

Setelah kolom kunci disiapkan, proses penggabungan dapat dilakukan. Penggabungan dilakukan berdasarkan nama kelurahan dan kecamatan, karena data kode pos tidak memiliki kode wilayah administratif resmi.

```python
gdf_final = gdf_spasial.merge(
    df_kodepos[["kelurahan_clean", "kecamatan_clean", "kode_pos"]],
    on=["kelurahan_clean", "kecamatan_clean"],
    how="left"
)
```

Metode `left` digunakan untuk memastikan seluruh wilayah tetap dipertahankan, meskipun terdapat kode pos yang tidak ditemukan padanannya.

---

### 3.5.5 Memeriksa Hasil Kompilasi

Sebelum data disimpan, hasil kompilasi perlu diperiksa untuk memastikan proses berjalan dengan baik.

```python
print("Jumlah wilayah:", len(gdf_final))
print("Jumlah kode pos kosong:", gdf_final["kode_pos"].isna().sum())
```

Untuk memastikan hasilnya masuk akal, beberapa baris awal data dapat ditampilkan.

```python
gdf_final[[
    "desa_kelurahan",
    "kecamatan",
    "kota_kabupaten",
    "kode_pos"
]].head()
```

Pada tahap ini, terlihat bahwa kode pos telah menjadi bagian dari tabel wilayah, bukan lagi data yang terpisah.

---

### 3.5.6 Menyimpan Data Geospasial Hasil Kompilasi

Langkah terakhir pada bagian ini adalah menyimpan data hasil kompilasi ke dalam format GeoJSON.

```python
gdf_final.to_file(
    "GIS_pekalongan.geojson",
    driver="GeoJSON"
)
```

File **`GIS_pekalongan.geojson`** menjadi keluaran utama dari proses kompilasi. Di dalamnya, setiap wilayah kelurahan telah memiliki representasi ruang, identitas administratif, dan informasi kode pos.

---

## 3.6 Membuat Pilihan: Bagaimana Angka Mulai Diberi Ruang

Bab ini tidak lahir dari keyakinan bahwa data yang digunakan telah sempurna. Justru sebaliknya. Ia berangkat dari satu kegelisahan sederhana: bagaimana mungkin lima digit kode pos—yang selama ini hanya dikenal sebagai bagian dari alamat—dapat dibaca sebagai data geospasial?

Kode pos tidak memiliki koordinat. Ia tidak menunjuk satu titik di peta. Ia hadir sebagai angka yang melekat pada nama wilayah. Dalam kondisi ini, pendekatan geospasial yang lazim digunakan tidak dapat langsung diterapkan. Tidak ada lintang, tidak ada bujur, dan tidak ada lokasi yang bisa segera diplot.

Namun, di titik inilah peluang muncul.

Alih-alih memaksakan kode pos menjadi titik geografis yang presisi, bab ini memilih pendekatan yang lebih sederhana: memberi kode pos sebuah wadah ruang. Wadah tersebut adalah wilayah administratif tempat kode pos tersebut berlaku. Dengan cara ini, kode pos tidak diperlakukan sebagai lokasi, melainkan sebagai informasi yang melekat pada ruang.


### 3.6.1 Ketika Data Tidak Disediakan untuk Dipetakan

Apabila data kode pos tersedia dalam bentuk terbuka lengkap dengan koordinat, proses pada bab ini mungkin akan jauh lebih singkat. Namun kenyataannya, data tersebut disajikan sebagai tabel informasi layanan pada halaman web, bukan sebagai dataset geospasial.

Alih-alih melihat kondisi ini sebagai keterbatasan, bab ini memperlakukannya sebagai titik awal eksplorasi. Pertanyaannya bergeser dari “bagaimana mendapatkan koordinat?” menjadi “apa yang dapat dilakukan dengan data yang tersedia?”.

Pendekatan pengambilan data langsung dari tabel web memungkinkan seluruh informasi kode pos dikumpulkan sebagai data tabular. Data ini kemudian dipadukan dengan data geografis lain. Dengan perubahan sudut pandang ini, data yang tidak dirancang sebagai data spasial tetap dapat dibaca secara geografis.


### 3.6.2 Menempelkan Angka pada Wilayah

Setelah data kode pos berhasil dikumpulkan, langkah berikutnya bukanlah mencari titik lokasi, melainkan menentukan tempat yang paling masuk akal untuk melekatkan angka tersebut.

Dalam konteks ini, wilayah administratif menjadi pilihan yang logis. Kode pos selalu terkait dengan kelurahan dan kecamatan tertentu. Artinya, meskipun tidak memiliki koordinat, kode pos tetap memiliki keterikatan ruang.

Dengan menempelkan kode pos pada batas wilayah kelurahan, angka tersebut mulai memiliki representasi spasial. Ia tidak menunjuk satu titik, tetapi mengisi sebuah area. Pendekatan ini menggeser cara pandang: dari upaya mencari lokasi yang tepat menuju pemahaman wilayah yang dilayani oleh satu kode pos.

Pada titik ini, kode pos mulai berubah fungsi—dari sekadar angka alamat menjadi atribut ruang.


### 3.6.3 Mengapa Wilayah Disederhanakan Menjadi Titik

Ketika batas wilayah ditampilkan secara utuh, peta menjadi kaya bentuk dan detail. Namun kekayaan visual ini sering kali justru menyulitkan pembacaan pola. Wilayah yang luas tampak dominan, sementara wilayah kecil di kawasan padat cenderung tenggelam.

Untuk mempermudah pembacaan, setiap wilayah kemudian disederhanakan menjadi satu titik pusat. Titik ini tidak dimaksudkan sebagai lokasi aktivitas atau bangunan tertentu. Ia berfungsi sebagai alat bantu visual untuk melihat posisi relatif antarwilayah tanpa terganggu oleh perbedaan bentuk dan ukuran.

Penyederhanaan ini memungkinkan peta dibaca dengan cara yang berbeda. Kedekatan, pengelompokan, dan sebaran menjadi lebih mudah diamati. Wilayah-wilayah yang sebelumnya tampak terpisah mulai terlihat sebagai bagian dari pola yang lebih besar.


### 3.6.4 Pilihan yang Disengaja, Bukan Jalan Pintas

Pendekatan pada bab ini bukanlah hasil dari keterbatasan teknis, melainkan pilihan yang disengaja. Fokus utama buku ini bukan pada ketepatan lokasi mikro, melainkan pada cara data administratif dapat dibaca sebagai struktur ruang.

Dengan menempelkan kode pos pada wilayah, lalu menyederhanakan wilayah tersebut menjadi titik, pembacaan geospasial menjadi lebih selaras dengan tujuan analisis. Pendekatan ini menunjukkan bahwa data tidak selalu harus presisi untuk menjadi bermakna. Yang lebih penting adalah kesesuaian antara tujuan pembacaan dan cara data direpresentasikan.

Pada titik ini, lima digit angka tidak lagi berdiri sendiri. Ia telah menjadi bagian dari ruang.


---

## 3.7 Ketika Data Bisa Diulang dan Perlu Dijaga

Sejak awal, bab ini tidak ditujukan untuk menghasilkan peta yang bersifat sekali jadi. Proses yang ditunjukkan dirancang agar dapat diulang, dimodifikasi, dan dikembangkan.

Seluruh langkah pada bab ini menggunakan perangkat lunak terbuka dan pustaka yang dapat diakses secara bebas. Tujuannya bukan untuk menunjukkan kecanggihan teknis, melainkan untuk membuka kemungkinan eksplorasi yang dapat dilakukan kembali pada konteks dan wilayah yang berbeda.

Namun, kemampuan untuk mengulang proses selalu datang bersama tanggung jawab.


### 3.7.1 Ketika Proses Bisa Diulang

Proses yang dapat diulang memberi ruang untuk bereksperimen. Data kode pos dapat diperbarui, digabungkan ulang, atau dibandingkan dengan wilayah lain tanpa harus memulai dari awal. Peta yang dihasilkan pada bab ini bukanlah hasil akhir, melainkan salah satu kemungkinan dari serangkaian pilihan yang diambil.

Dengan memahami alur kerja secara utuh—mulai dari pengambilan data hingga pembentukan dataset geospasial—setiap langkah dapat disesuaikan dengan kebutuhan masing-masing. Wilayah studi dapat diganti, skala analisis diubah, dan data lain ditambahkan sebagai lapisan baru.

Nilai utama dari bab ini tidak terletak pada peta yang dihasilkan, melainkan pada cara berpikir yang dapat diterapkan kembali di konteks lain.


### 3.7.2 Ketika Data Perlu Dijaga

Kemudahan mengakses dan mengolah data sering kali membuat batas penggunaan data menjadi kabur. Data kode pos yang diambil melalui halaman web disajikan sebagai informasi layanan, bukan sebagai dataset resmi untuk analisis spasial.

Oleh karena itu, penggunaan data pada bab ini perlu dipahami dalam konteks pembelajaran dan eksplorasi. Data digunakan tanpa memodifikasi isi informasi dan tanpa tujuan komersial. Tujuannya adalah menunjukkan kemungkinan pembacaan spasial, bukan menggantikan sistem resmi atau menarik kesimpulan kebijakan.

Kesadaran terhadap konteks data dan cara penggunaannya merupakan bagian penting dari literasi data geospasial.


### 3.7.3 Batas antara Teknik dan Tanggung Jawab

Bab ini menunjukkan bahwa kemampuan menggunakan alat dan metode dapat dipelajari relatif cepat. Namun, kemampuan tersebut tanpa pemahaman konteks berisiko menghasilkan pembacaan ruang yang keliru.
Peta bukanlah kebenaran tunggal, melainkan representasi yang selalu terikat pada sumber data dan metode yang digunakan. Oleh karena itu, peta yang dihasilkan dari data administratif perlu dibaca sebagai alat bantu pemahaman, bukan sebagai gambaran lengkap realitas.
Pada akhirnya, keterampilan geospasial tidak hanya ditentukan oleh apa yang bisa dilakukan secara teknis, tetapi juga oleh kesadaran kapan dan bagaimana alat serta metode tersebut digunakan. Di sinilah teknik dan tanggung jawab bertemu.


---

## Quiz Reflektif

1.	Mengapa kode pos tidak dapat langsung dipetakan sebagai titik geografis?
2.	Dalam bab ini, mengapa wilayah administratif dipilih sebagai “wadah ruang” bagi kode pos?
3.	Apa makna penyederhanaan wilayah menjadi satu titik pusat dalam konteks pembacaan pola, bukan presisi lokasi?
4.	Mengapa proses menyamakan penulisan nama wilayah menjadi langkah krusial dalam kompilasi data?
5.	Setelah mengikuti alur Bab 3, menurutmu apa yang lebih penting dalam praktik SIG: ketepatan teknis atau kesesuaian metode dengan tujuan analisis? Jelaskan alasannya.

> **Penting:**  
> Tidak ada jawaban benar atau salah. Yang penting adalah kesadaran bahwa angka dapat menyimpan petunjuk ruang.

---
Pada bab ini, ruang tidak dibangun dari peta,
melainkan dari keputusan-keputusan kecil tentang data.

Lima digit kode pos tidak pernah memiliki bentuk.
Ia tidak tahu di mana harus diletakkan, tidak memiliki koordinat, dan tidak menunjuk satu titik pun di permukaan bumi.
Namun ia tetap melekat pada wilayah—pada nama kelurahan, pada kecamatan, pada ruang hidup manusia.

Membangun ruang dari angka berarti menerima satu kenyataan sederhana:
peta tidak selalu lahir dari presisi, tetapi dari kesesuaian cara pandang.
Ketika angka ditempelkan pada wilayah, dan wilayah disederhanakan agar bisa dibaca,
yang terbentuk bukan kebenaran mutlak, melainkan representasi yang masuk akal.

Bab ini menunjukkan bahwa praktik geospasial bukan soal mencari bentuk paling sempurna,
melainkan soal memahami apa yang sedang kita baca,
dan mengapa kita memilih cara tertentu untuk membacanya.


<div style="display:flex; justify-content:space-between;">
<span>⬅️ <a href="bab-2.md">Bab II – Kode Pos dan Wilayah</a></span>
<span><a href="bab-4.md">Bab IV – Studi Kasus</a> ➡️</span>
</div>


