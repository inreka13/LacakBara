# LacakBara

**Sistem intelijen satelit berbasis web untuk memprioritaskan hotspot kebakaran hutan dan lahan di Indonesia.**

> Status: perancangan MVP dan protokol penelitian  
> Sasaran awal: kompetisi inovasi, prototipe web, dan artikel ilmiah

## Ringkasan

LacakBara membantu pengguna menyaring banyak deteksi hotspot menjadi daftar lokasi yang lebih layak diprioritaskan untuk verifikasi. Sistem menggabungkan intensitas anomali termal, confidence satelit, persistensi waktu, kesepakatan antarsatelit, dan konteks spasial ke dalam skor prioritas yang dapat dijelaskan.

Produk akhirnya berupa peta web Indonesia dengan pembaruan berkala, peringkat wilayah, riwayat hotspot, penjelasan faktor risiko, citra pendukung, serta ekspor laporan ringkas.

LacakBara **bukan pengganti peringatan resmi dan tidak menyatakan setiap hotspot sebagai kebakaran terkonfirmasi**. Hotspot merupakan anomali termal yang perlu diverifikasi menggunakan bukti tambahan dan informasi lapangan.

## Masalah yang Diselesaikan

Data hotspot satelit tersedia secara terbuka, tetapi pengguna masih menghadapi beberapa persoalan:

- jumlah titik dapat sangat banyak dan tersebar;
- satu kejadian dapat terdeteksi berulang oleh beberapa satelit;
- hotspot sesaat belum tentu menunjukkan kejadian yang persisten;
- confidence tinggi saja belum cukup menggambarkan urgensi lokasi;
- peta hotspot umumnya belum langsung menjelaskan alasan suatu lokasi perlu diprioritaskan;
- petugas dan masyarakat membutuhkan ringkasan yang cepat dipahami, bukan hanya data mentah.

LacakBara mengubah data tersebut menjadi **informasi prioritas yang transparan dan dapat ditelusuri**.

## Tujuan

1. Mengintegrasikan data hotspot terbuka untuk wilayah Indonesia.
2. Mengelompokkan deteksi berdekatan yang kemungkinan berasal dari kejadian sama.
3. Mengukur persistensi dan kesepakatan deteksi antarsatelit.
4. Memberikan skor prioritas beserta alasan yang mudah dipahami.
5. Menyediakan visualisasi temporal dan spasial melalui aplikasi web.
6. Mengevaluasi apakah pemeringkatan multisumber lebih andal daripada threshold confidence sederhana.

## Pengguna Sasaran

- BPBD dan pemerintah daerah;
- pengelola kawasan hutan, perkebunan, dan konservasi;
- relawan kebencanaan;
- peneliti lingkungan dan penginderaan jauh;
- jurnalis data dan masyarakat umum.

## Fitur MVP

### 1. Peta hotspot Indonesia

- Hotspot 24 jam dan 7 hari terakhir.
- Filter berdasarkan satelit, waktu, confidence, dan wilayah.
- Simbol dan warna yang tidak menyamakan hotspot dengan kebakaran terkonfirmasi.

### 2. Pengelompokan kejadian

- Menggabungkan deteksi yang berdekatan secara spasial dan temporal.
- Mengurangi duplikasi tampilan.
- Menampilkan jumlah observasi dan durasi aktivitas setiap klaster.

### 3. Skor prioritas yang dapat dijelaskan

Setiap klaster menerima skor awal berdasarkan:

- `C`: confidence deteksi;
- `F`: Fire Radiative Power yang telah dinormalisasi;
- `P`: persistensi hotspot;
- `M`: kesepakatan antarsatelit;
- `E`: konteks paparan, misalnya kedekatan dengan permukiman atau kawasan penting.

Formulasi konseptual:

```text
PriorityScore = wc*C + wf*F + wp*P + wm*M + we*E
```

Bobot tidak ditentukan secara arbitrer untuk hasil akhir. Bobot akan dikalibrasi dan divalidasi menggunakan data historis, analisis sensitivitas, serta perbandingan terhadap baseline.

### 4. Halaman detail kejadian

- Lokasi dan wilayah administrasi.
- Waktu deteksi pertama dan terakhir.
- Jumlah pengamatan.
- Satelit yang mendeteksi.
- Nilai confidence dan Fire Radiative Power.
- Timeline aktivitas.
- Faktor yang menaikkan atau menurunkan skor.

### 5. Ringkasan wilayah

- Peringkat provinsi dan kabupaten/kota.
- Jumlah hotspot dan klaster aktif.
- Distribusi tingkat prioritas.
- Perubahan dibanding periode sebelumnya.

### 6. Ekspor laporan

- CSV untuk analisis.
- Ringkasan PDF pada tahap berikutnya.
- Tautan lokasi yang dapat dibagikan.

## Sumber Data Terbuka

| Sumber | Data | Penggunaan |
|---|---|---|
| [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/web-services/) | MODIS dan VIIRS active fire/hotspot | Sumber utama deteksi termal |
| [Copernicus Data Space](https://dataspace.copernicus.eu/) | Sentinel-1 dan Sentinel-2 | Verifikasi konteks, vegetasi, dan perubahan permukaan pada fase lanjutan |
| [BNPB Satu Data](https://data.bnpb.go.id/) | Riwayat kejadian dan dampak bencana | Validasi temporal dan studi kasus |
| [OpenStreetMap](https://www.openstreetmap.org/) | Permukiman, jalan, dan fasilitas | Analisis konteks paparan |

Penggunaan setiap dataset harus mengikuti lisensi, ketentuan atribusi, keterlambatan pembaruan, resolusi spasial, dan batasan kualitas masing-masing sumber.

## Arsitektur Konseptual

```text
NASA FIRMS / sumber terbuka lain
                |
                v
       Ingestion & validation
                |
                v
  Cleaning, deduplication, clustering
                |
                v
    Feature engineering & scoring
                |
        +-------+-------+
        |               |
        v               v
 Spatial database   Evaluation store
        |
        v
        API
        |
        v
 Web map, ranking, timeline, report
```

## Rancangan Metodologi Penelitian

### Baseline

1. Threshold confidence FIRMS.
2. Confidence ditambah Fire Radiative Power.
3. Skor usulan multisumber dengan persistensi, kesepakatan satelit, dan konteks spasial.

### Strategi evaluasi

- Pemisahan data berdasarkan waktu dan wilayah, bukan pembagian acak per titik.
- Pengujian generalisasi antarperiode dan antarprovinsi.
- Analisis sensitivitas terhadap radius klaster, jendela waktu, dan bobot fitur.
- Ablation study untuk mengukur kontribusi setiap komponen skor.
- Pemeriksaan kasus false positive dan false negative secara terpisah.

### Metrik

- precision, recall, dan F1-score;
- false-alarm rate;
- lead time;
- calibration error;
- ranking quality, seperti Precision@K atau nDCG;
- stabilitas peringkat terhadap perubahan parameter;
- waktu respons pipeline dan API.

Definisi ground truth harus ditetapkan secara eksplisit sebelum eksperimen. Science-quality fire products, burned-area products, catatan kejadian BNPB, dan bukti citra dapat digunakan sesuai ketersediaan, tetapi tidak boleh dianggap setara tanpa prosedur penyelarasan.

## Rancangan Antarmuka

### Beranda

- Peta Indonesia.
- Ringkasan hotspot 24 jam dan 7 hari.
- Lima wilayah dengan prioritas tertinggi.
- Filter waktu dan sumber satelit.

### Detail klaster

- Peta lokal.
- Timeline pengamatan.
- Breakdown skor.
- Data mentah yang mendasari kesimpulan.
- Tautan citra pendukung jika tersedia.

### Analitik

- Grafik jumlah hotspot per wilayah dan waktu.
- Perbandingan confidence, intensitas, dan persistensi.
- Riwayat perubahan prioritas.

## Tumpukan Teknologi yang Disarankan

| Lapisan | Teknologi |
|---|---|
| Frontend | Next.js/React dan MapLibre GL atau Leaflet |
| Backend API | FastAPI |
| Pemrosesan geospasial | Python, GeoPandas, Shapely, Rasterio |
| Database | PostgreSQL dan PostGIS |
| Penjadwalan | Cron worker atau task queue ringan |
| Deployment | Container/Docker pada fase implementasi |

Pemilihan teknologi dapat disederhanakan untuk MVP. Tidak semua komponen harus dibangun pada iterasi pertama.

## Struktur Repositori yang Direncanakan

```text
LacakBara/
|-- README.md
|-- docs/
|   |-- research-protocol.md
|   `-- data-dictionary.md
|-- backend/
|   |-- app/
|   |-- pipelines/
|   `-- tests/
|-- frontend/
|   |-- src/
|   `-- public/
|-- notebooks/
|-- data/
|   `-- README.md
`-- infra/
```

Folder kode belum dibuat karena proyek masih berada pada tahap perancangan. README ini tidak mengasumsikan bahwa perintah instalasi atau layanan aplikasi sudah tersedia.

## Roadmap

### Fase 0 - Definisi penelitian

- [ ] Menentukan wilayah dan periode studi awal.
- [ ] Menetapkan definisi klaster dan ground truth.
- [ ] Menyusun data dictionary.
- [ ] Menentukan baseline dan protokol evaluasi.

### Fase 1 - MVP LacakBara

- [ ] Mengambil data FIRMS untuk Indonesia.
- [ ] Membersihkan dan menyimpan data hotspot.
- [ ] Menampilkan peta 24 jam dan 7 hari.
- [ ] Membuat pengelompokan spasial-temporal.
- [ ] Menampilkan skor awal dan penjelasannya.
- [ ] Membuat halaman peringkat wilayah.

### Fase 2 - Validasi ilmiah

- [ ] Mengumpulkan data historis lintas tahun.
- [ ] Membangun baseline.
- [ ] Menjalankan spatial-temporal validation.
- [ ] Melakukan ablation dan sensitivity analysis.
- [ ] Mendokumentasikan error dan keterbatasan.

### Fase 3 - Pengayaan multisumber

- [ ] Menambahkan citra Sentinel.
- [ ] Menambahkan konteks vegetasi dan perubahan area terbakar.
- [ ] Menambahkan paparan permukiman dan fasilitas.
- [ ] Menguji integrasi cuaca jika sumber dan lisensinya sesuai.

### Fase 4 - Paket lomba dan publikasi

- [ ] Menyusun video demo dan skenario presentasi.
- [ ] Menambahkan studi kasus historis Indonesia.
- [ ] Menyusun dokumentasi reproduksibilitas.
- [ ] Menulis manuskrip dan supplementary material.

## Posisi Novelty

LacakBara tidak mengklaim menemukan hotspot baru dari citra mentah pada fase MVP. Kontribusi yang ditargetkan adalah:

1. pengelompokan spasial-temporal untuk mengurangi duplikasi deteksi;
2. pemeringkatan hotspot yang menggabungkan intensitas, persistensi, multisatelit, dan konteks paparan;
3. penjelasan faktor pembentuk prioritas;
4. evaluasi generalisasi lintas wilayah dan waktu;
5. antarmuka keputusan yang menghubungkan data satelit dengan kebutuhan verifikasi lapangan.

Judul artikel sementara:

> **Explainable Multi-Source Satellite Hotspot Prioritization for Indonesian Forest and Peatland Fire Monitoring**

## Nilai untuk Kompetisi

- Mengangkat persoalan nasional yang mendesak.
- Memakai data terbuka sehingga dapat direplikasi dan diperluas.
- Tidak membutuhkan pemasangan sensor di seluruh wilayah.
- Memiliki demo peta yang visual dan mudah dipahami.
- Menjelaskan keputusan, bukan hanya menampilkan prediksi.
- Berpotensi dikembangkan menjadi modul kedua, **FloodScope**, tanpa mencampurkan kedua eksperimen ilmiah dalam satu paper.

## Batasan dan Etika

- Hotspot bukan bukti tunggal kebakaran aktif.
- Resolusi dan waktu lintasan satelit membatasi ketelitian sistem.
- Tutupan awan memengaruhi citra optik.
- Nilai prioritas adalah rekomendasi riset, bukan status resmi kedaruratan.
- Sistem tidak boleh digunakan sebagai satu-satunya dasar evakuasi, penindakan, atau alokasi sumber daya.
- Lokasi sensitif dan data pengguna tidak boleh dipublikasikan tanpa dasar hukum dan kebutuhan yang jelas.
- Antarmuka harus menampilkan sumber, waktu pembaruan, dan tingkat ketidakpastian.

## Lisensi

Lisensi kode proyek belum ditetapkan. Lisensi proyek tidak menggantikan lisensi dan kewajiban atribusi dari dataset pihak ketiga.

## Referensi Awal

- NASA FIRMS Web Services: <https://firms.modaps.eosdis.nasa.gov/web-services/>
- NASA FIRMS Active Fire Data: <https://firms.modaps.eosdis.nasa.gov/content/active_fire/>
- Copernicus Data Space Ecosystem: <https://dataspace.copernicus.eu/>
- Copernicus API Documentation: <https://documentation.dataspace.copernicus.eu/APIs.html>
- BNPB Satu Data: <https://data.bnpb.go.id/>

