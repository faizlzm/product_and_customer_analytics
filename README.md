# Product & Customer Analytics: Segmentasi Pelanggan dan Mitigasi Risiko Churn

Analisis 2.747 baris pesanan sebuah distributor otomotif B2B dengan Python (RFM dan analisis Pareto) dan Tableau untuk memetakan perilaku mitra, mendeteksi akun bernilai tinggi yang berhenti memesan sebagai peringatan dini risiko churn, dan menyusun strategi retensi akun kunci.

## Daftar Isi

- [Latar Belakang](#latar-belakang)
- [Problem Statement](#problem-statement)
- [Struktur Repositori](#struktur-repositori)
- [Dataset](#dataset)
- [Metodologi](#metodologi)
- [Temuan Utama](#temuan-utama)
- [Rekomendasi](#rekomendasi)
- [Catatan Selisih Angka dengan Laporan PDF](#catatan-selisih-angka-dengan-laporan-pdf)
- [Keterbatasan](#keterbatasan)
- [Cara Menjalankan](#cara-menjalankan)
- [Penulis](#penulis)

---

## Latar Belakang

Pada bisnis distribusi kendaraan B2B, kelangsungan usaha bertumpu pada hubungan jangka panjang dengan jaringan dealer dan akun fleet. Nilai kontrak besar dengan basis mitra yang terbatas membuat kehilangan satu akun terasa langsung pada pendapatan. Ancaman utamanya adalah **churn diam-diam**: mitra mengurangi volume pesanan atau berhenti bertransaksi tanpa keluhan formal.

## Problem Statement

> Bagaimana memetakan perilaku transaksi mitra, mendeteksi akun yang mengalami anomali siklus pemesanan sedini mungkin sebagai peringatan dini risiko churn, serta merumuskan strategi retensi akun kunci yang aplikatif?

## Struktur Repositori

```text
product_and_customer_analytics/
├── automobile_sales_data.csv                    # Dataset mentah (2.747 baris, 20 kolom)
├── product_and_customer_analysis.ipynb          # Data preparation, RFM, dan analisis risiko churn
├── Book1.twb                                    # Workbook Tableau (9 worksheet)
├── Product & Customer Analytics_compressed.pdf  # Laporan presentasi (16 slide)
└── README.md
```

Worksheet Tableau: `AOV`, `Action Table`, `Behavioral Risk`, `Pareto Concentration`, `Product Preference`, `Segment Revenue vs Count`, `Total Customers`, `Total Order`, `Total Revenue`.

## Dataset

Data faktur pesanan distributor otomotif B2B global.

- **Jumlah baris:** 2.747 (satu baris per item pesanan)
- **Jumlah kolom:** 20
- **Periode pesanan:** 6 Januari 2018 – 31 Mei 2020
- **Cakupan:** 89 pelanggan, 19 negara, 7 lini produk, 109 kode produk, 298 nomor pesanan

| Kelompok | Kolom |
| --- | --- |
| Pesanan | `ORDERNUMBER`, `ORDERLINENUMBER`, `ORDERDATE`, `STATUS`, `DEALSIZE` |
| Produk | `PRODUCTLINE`, `PRODUCTCODE`, `MSRP` |
| Nilai | `QUANTITYORDERED`, `PRICEEACH`, `SALES` |
| Pelanggan | `CUSTOMERNAME`, `PHONE`, `ADDRESSLINE1`, `CITY`, `POSTALCODE`, `COUNTRY`, `CONTACTFIRSTNAME`, `CONTACTLASTNAME` |
| Lainnya | `DAYS_SINCE_LASTORDER` |

Status pesanan: Shipped 2.541, Cancelled 60, Resolved 47, On Hold 44, In Process 41, Disputed 14. Ukuran transaksi: Medium 1.349, Small 1.246, Large 152.

> Dataset memuat nama kontak dan nomor telepon pelanggan. Gunakan hanya untuk keperluan pembelajaran.

## Metodologi

### 1. Data Preparation

| Pemeriksaan | Hasil | Tindakan |
| --- | --- | --- |
| Duplikasi | 0 baris | Tidak perlu penanganan |
| Missing value | 0 pada seluruh 20 kolom | Tidak perlu penanganan |
| Tipe data | `ORDERDATE` berupa teks `DD/MM/YYYY` | Dikonversi ke datetime |
| Teks pelanggan | Spasi berlebih pada `CUSTOMERNAME` | `strip` (89 pelanggan unik) |
| Integritas angka | Tidak ada nilai ≤ 0; `SALES ≈ QUANTITYORDERED × PRICEEACH` (selisih maks. $0,04) | Tidak perlu penanganan |
| Status transaksi | 159 baris berstatus Cancelled, Disputed, On Hold, atau In Process | **Disaring.** Hanya `Shipped` dan `Resolved` yang dihitung agar pendapatan riil tidak terdistorsi |
| Outlier (IQR) pada `SALES` | 74 baris di atas $7.916,88, seluruhnya `DEALSIZE = Large` | **Dipertahankan** karena merupakan pesanan borongan yang valid |

Hasil: **2.588 baris**, 89 pelanggan, 281 pesanan, pendapatan **$9,17 juta** (dari $9,76 juta sebelum penyaringan).

### 2. RFM dan Segmentasi

Tanggal referensi adalah satu hari setelah pesanan terakhir yang sah (18 Mei 2020). Setiap akun diagregasi menjadi:

| Metrik | Definisi |
| --- | --- |
| Recency | Hari sejak pesanan terakhir |
| Frequency | Jumlah nomor pesanan unik |
| Monetary | Total `SALES` |

Setiap metrik diberi skor kuantil 1–4, lalu dipetakan ke enam segmen dengan aturan di notebook:

| Segmen | Aturan |
| --- | --- |
| Champions | R ≥ 3, F ≥ 3, M ≥ 3 |
| Loyal Customers | R ≥ 2, F ≥ 3 |
| Potential Loyalists | R ≥ 3, F ≤ 2 |
| **At Risk** | R ≤ 2 dan (M ≥ 3 atau F ≥ 3) |
| Hibernating | R = 2, F ≤ 2 |
| Lost | Sisanya |

Segmen At Risk diberi label *High Risk*, Lost dan Hibernating *Medium Risk*, Potential Loyalists *Low-Medium Risk*, sisanya *Low Risk*.

### 3. Analisis Lanjutan

Kurva Pareto untuk konsentrasi pendapatan, preferensi lini produk per segmen (heatmap), matriks Recency vs Monetary untuk kuadran bahaya, dan daftar akun target intervensi beserta kontak. Hasil akhirnya digabung menjadi dataset master untuk Tableau.

## Temuan Utama

> Angka di bawah dihitung ulang dari `automobile_sales_data.csv` dan cocok dengan output tersimpan di notebook. Selisih dengan laporan PDF dijelaskan di [bagian berikutnya](#catatan-selisih-angka-dengan-laporan-pdf).

### 1. Bisnis B2B dengan nilai per akun besar

| Metrik | Nilai |
| --- | ---: |
| Total pendapatan | $9.169.812 |
| Jumlah pesanan | 281 |
| Jumlah akun pelanggan | 89 |
| Rata-rata nilai pesanan | $32.633 |
| Rata-rata pendapatan per akun | $103.032 |

### 2. Pendapatan terkonsentrasi pada segelintir mitra

- 17 akun teratas (19,1% dari 89 akun) menyumbang **$3,66 juta atau 39,96%** dari total pendapatan.
- Dua mitra teratas, **Euro Shopping Channel** ($795 ribu; 8,67%) dan **Mini Gifts Distributors Ltd.** ($648 ribu; 7,06%), bersama-sama menghasilkan 15,7%. Mitra peringkat ketiga hanya $201 ribu (2,19%).

### 3. Profil segmen RFM

| Segmen | Akun | Porsi akun | Pendapatan | Porsi pendapatan | Rata-rata recency |
| --- | ---: | ---: | ---: | ---: | ---: |
| Champions | 22 | 24,7% | $3.929.187 | 42,8% | 88 hari |
| Loyal Customers | 22 | 24,7% | $1.588.334 | 17,3% | 140 hari |
| **At Risk** | **14** | **15,7%** | **$1.568.313** | **17,1%** | **249 hari** |
| Lost | 17 | 19,1% | $939.544 | 10,2% | 360 hari |
| Potential Loyalists | 7 | 7,9% | $712.988 | 7,8% | 92 hari |
| Hibernating | 7 | 7,9% | $431.446 | 4,7% | 216 hari |

Champions dan Loyal Customers (44 akun) menguasai 60,2% pendapatan.

### 4. Segmen At Risk adalah ancaman utama

- **14 akun** bernilai tinggi (rata-rata $112 ribu per akun) belum memesan selama **185–447 hari** (rata-rata 249 hari, sekitar 8,3 bulan).
- Nilai pendapatan historisnya **$1,57 juta (17,1% dari total)**, hampir setara dengan seluruh segmen Loyal Customers.
- Lini produk dominan segmen ini: **Classic Cars (43,4%)** dan **Vintage Cars (18,8%)**, atau 62,2% dari belanja historisnya. Mereka adalah pembeli model koleksi dalam jumlah borongan, bukan pembeli eceran.
- Akun At Risk dengan belanja tertinggi: Saveley & Henriot, Co. ($142.874; 443 hari tidak aktif), Rovelli Gifts ($137.956), dan Online Diecast Creations Co. ($131.685).

## Rekomendasi

1. **Emergency outreach untuk akun At Risk.** Hubungi langsung (telepon atau kunjungan) dan audit hambatan seperti harga, ketersediaan model klasik, atau kendala pengiriman. Tawarkan insentif diskon pesanan ulang untuk lini Classic Cars.
2. **VIP protection program untuk Champions.** Beri akses awal ke miniatur edisi terbatas dan tawarkan kontrak pasokan tahunan dengan skema insentif.
3. **Account upselling untuk Potential Loyalists.** Tawarkan paket bundel produk komplementer untuk menaikkan transaksi dari Small ke Medium.
4. **Efisiensi biaya untuk Lost dan Hibernating.** Cukup jangkau lewat channel promosi berkala yang murah.

## Catatan Selisih Angka dengan Laporan PDF

Beberapa angka di laporan PDF (dan teks markdown di notebook) berbeda dari output kode dan hitung ulang dari dataset:

| Item | Laporan PDF | Hasil kode notebook (dipakai di README) |
| --- | --- | --- |
| Jumlah akun At Risk | 12 akun (13,5%) | **14 akun (15,7%)** |
| Pendapatan At Risk | $1.273.975 (13,89%) | **$1.568.313 (17,10%)** |
| Rata-rata belanja per akun At Risk | $106 ribu | **$112 ribu** |
| Rata-rata recency At Risk | 258 hari | **249 hari** |
| Classic Cars / Vintage Cars pada At Risk | 34,8% / 21,5% | **43,4% / 18,8%** |
| Top 20% pelanggan | 18 akun (teks notebook) | **17 akun** (19,1%) |
| Champions + Loyal | 42 akun, 62,94% (teks notebook) | **44 akun, 60,2%** |

Daftar target intervensi di notebook juga berjudul "12 akun", padahal tabelnya menampilkan 14 nama. Kesimpulan arah analisis tidak berubah, tetapi angka di slide dan teks sebaiknya diperbarui agar konsisten dengan hasil kode.

## Penulis

**Ahmad Faiz Ali Azmi**
Lulusan Teknik Informatika, Universitas Brawijaya. Peserta Offline Bootcamp Data Analyst Batch 4 di dibimbing.id.

GitHub: [@faizlzm](https://github.com/faizlzm)
