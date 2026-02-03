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

