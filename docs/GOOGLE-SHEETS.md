# 📊 Laporan Keuangan → Google Sheets

Fitur ini mengirim **transaksi penjualan** dan **pengeluaran** dari aplikasi kasir
(*My Cash-POS APP*) ke **Google Spreadsheet** secara otomatis setiap kali ada pembayaran.
Dari sana Anda bisa membuat **laporan laba rugi**, **penjualan per produk**, dan **chart**
menggunakan pivot table bawaan Google Sheets.

> Tidak ada biaya. Google Apps Script gratis.

---

## Cara setup (sekali saja)

### 1️⃣ Buat spreadsheet
Buka <https://sheets.google.com> → **Blank spreadsheet**.

### 2️⃣ Buka Apps Script
Di spreadsheet tadi: menu **Extensions → Apps Script**.

### 3️⃣ Tempel kodenya
Hapus seluruh isi `Code.gs`, lalu **tempel** isi file
[`scripts/gs-laporan.gs`](../../scripts/gs-laporan.gs) dari repo ini.
Klik **Simpan** (Ctrl+S).

> **Penting:** ubah baris `var KUNCI_RAHASIA = 'GANTI-DENGAN-KUNCI-ANDA';`
> menjadi kunci buatan Anda sendiri. Contoh: `'87project-rk-2026'`.
> Kunci ini yang mengizinkan aplikasi mengirim data, jadi jangan dibuat terlalu mudah ditebak.

### 4️⃣ Deploy sebagai Web App
Klik **Deploy → New deployment** → pilih tipe **Web app** → **Deploy**:

| Pengaturan | Pilih |
|---|---|
| Execute as | **Me** |
| Who has access | **Anyone** |

Google akan minta izin ("Google hasn't verified this app") → klik **Advanced** →
**Go to (nama proyek) (unsafe)** → **Allow**. Ini normal untuk script buatan sendiri.

Lalu **salin URL Web App** yang muncul. Bentuknya:
```
https://script.google.com/macros/s/AKfycbXXXXXXXXXXXXXXXX/exec
```

### 5️⃣ Tempel di aplikasi
Buka aplikasi → **Pengaturan → Google Sheets — Laporan Keuangan**:

1. **Apps Script Web App URL** ← tempel URL tadi
2. **Kunci Rahasia** ← isi kunci yang Anda buat di langkah 3
3. Ketuk **Simpan**
4. Ketuk **Uji Koneksi** → kalau muncul ✅, cek spreadsheet Anda,
   akan ada baris `TES-KONEKSI-...` di tab **Transaksi**

Selesai. Setiap pembayaran & pencatatan pengeluaran otomatis terkirim.

### ✅ Cara memastikan endpoint sudah benar

Buka **URL `/exec` itu langsung di browser**. Seharusnya muncul JSON seperti ini:

```json
{ "ok": true, "app": "My Cash-POS APP", "status": "aktif — endpoint siap menerima data",
  "kunciSudahDiatur": true, "tabTersedia": ["Transaksi","Penjualan","Pengeluaran"] }
```

Kalau `kunciSudahDiatur` bernilai `false`, berarti `KUNCI_RAHASIA` di script masih
placeholder — ganti, lalu **Deploy ulang**.

> Kalau yang muncul di browser adalah `Script function not found: doGet`, berarti
> kode belum tersimpan atau nama episodenya berubah. Pastikan tetap `doGet`/`doPost`.

---

## Data yang dikirim

App membuat 3 tab otomatis (dengan judul kolom) di spreadsheet Anda:

### Tab `Transaksi` — 1 baris per transaksi

| Kolom | Isi |
|---|---|
| ID_Transaksi | `TRX-20261005-001` |
| Tanggal / Waktu | Tanggal & jam transaksi |
| Kasir / Toko | Username kasir & nama toko |
| Jml_Item / Qty_Total | Jumlah item & total kuantitas |
| Subtotal | Total sebelum diskon |
| Diskon_Persen / Diskon_Rp | Persentase & rupiah diskon |
| **Total** | **Total yang dibayar** |
| Metode_Bayar / Tunai / Kembalian | Detail pembayaran |
| Shift / Status | Shift kasir, `paid` / `refunded` |

### Tab `Penjualan` — 1 baris per item

| Kolom | Isi |
|---|---|
| ID_Transaksi | ID transaksi induknya |
| Produk / Kategori | Nama produk & kategorinya |
| Qty / Harga_Satuan | Jumlah & harga per item |
| Subtotal_Item | Qty × harga |

### Tab `Pengeluaran` — 1 baris per pengeluaran

| Kolom | Isi |
|---|---|
| ID_Pengeluaran | `EXP-XXXX` |
| Keterangan | Catatan pengeluaran |
| Jumlah | Nominal |
| Kasir | Siapa yang mencatat |

---

## Cara membuat laporan keuangan

### ✅ Laporan Laba Rugi harian
1. Menu **Data → Buat pivot table**
2. Sumber: tab **Transaksi**, baris `Tanggal`, nilai `Total`
3. Untuk pengeluaran: pivot dari tab **Pengeluaran**, nilai `Jumlah`
4. Laba = Omzet − Pengeluaran

### ✅ Penjualan per produk
Pivot dari tab **Penjualan**: baris `Produk`, kolom `Kategori`, nilai `Subtotal_Item`.

### ✅ Grafik omzet harian
Pilih data pivot → **Sisipkan → Diagram** → pilih tipe garis/batang.

### 💡 Tips analisis cepat
- Tambah kolom turunan di spreadsheet: `=Tanggal` sudah berformat, jadi bisa difilter per bulan.
- Gunakan filter tanggal untuk laporan bulanan.
- Kunci baris pertama tiap tab sudah di-*freeze* otomatis.

---

## Cara kerja (offline-safe)

| Keadaan | Yang terjadi |
|---|---|
| App **online** | Data dikirim langsung setelah pembayaran |
| App **offline** / tidak ada sinyal | Data masuk **antrean** lokal (maks 500 baris) |
| Internet kembali | Tekan **Kirim Tertunda** (atau buka app lagi, antrean dikirim otomatis) |
| Pengiriman gagal | Antrean **tidak dihapus**, dicoba lagi nanti |
| Data terkirim 2× | **Otomatis dilewati** — spreadsheet mengecek ID, jadi tidak ada data kembar |

Tombol lain di Pengaturan:

- **Kirim Tertunda (n)** — mengirim antrean yang tersisa
- **Kirim Ulang Semua** — membuat ulang antrean dari seluruh riwayat transaksi.
  Aman karena duplikat otomatis dilewati di sisi spreadsheet.

---

## Troubleshooting

| Masalah | Penyebab & Solusi |
|---|---|
| **"Belum dikonfigurasi"** | URL atau Kunci Rahasia kosong. Isi keduanya lalu **Simpan**. |
| **URL berakhiran `/dev`** | Itu hanya untuk pemilik script. Salin URL `/exec` dari **Deploy**. |
| **"Koneksi diblokir browser"** / `Failed to fetch` | 3 hal yang perlu dicek: (1) URL harus `/exec`, (2) di Deploy, *Who has access* = **Anyone** (bukan *Anyone with Google account*), (3) ada internet. |
| **`Kunci rahasia salah`** | Kunci di app ≠ `KUNCI_RAHASIA` di `gs-laporan.gs`. Samakan, lalu **Deploy ulang**. |
| **`KUNCI_RAHASIA ... masih placeholder`** | Baris `KUNCI_RAHASIA` di script belum diganti. Ganti lalu Deploy ulang. |
| **HTTP 401 / 403 / Sign in** | Deployment hanya boleh untuk akun Google. Edit deployment → *Who has access: Anyone* → Deploy ulang. |
| **`Script function not found`** | Nama fungsi di script berubah / belum disimpan. Pastikan kode tetap `doPost` lalu Deploy ulang. |
| **`Out of quota` / 403 quota** | Kuota Apps Script harian habis. Tunggu reset, atau kurangi pengiriman dengan memakai **Kirim Ulang Semua** satu kali. |
| **Tidak ada baris masuk** | Pastikan tab tidak disembunyikan; cek juga status **Kirim Tertunda** di Pengaturan. |
| **Mau pindah ke sheet lain** | Buat spreadsheet baru, ulangi langkah 2–5, lalu tekan **Kirim Ulang Semua**. |
| **Kolom angka jadi teks** | Pilih kolom → menu **Data → Pisahkan teks ke angka** (`Data → Split text to columns`). |

---

## 🔒 Privasi data

Semua data dikirim ke spreadsheet **milik Anda sendiri**. Tidak ada server
perantara — aplikasi speak langsung ke `script.google.com`. Kalau butuh
menghapus semua jejak, hapus spreadsheet-nya.