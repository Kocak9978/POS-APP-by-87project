# Kasir Pro — Aplikasi POS (Point of Sale) by 87PROJECT

Aplikasi **Point of Sale** berbasis web sederhana yang berjalan sepenuhnya di browser.
Tidak perlu server atau instalasi database — semua data tersimpan di `localStorage` browser.

## 🚀 Cara Menjalankan

Buka file **`index.html`** langsung di browser (double-click), atau jalankan server local:

```bash
# opsi 1: langsung buka file
xdg-open index.html

# opsi 2: server statis (opsional)
python3 -m http.server 8080
# lalu buka http://localhost:8080
```

## 🔑 Akun Demo

| Peran  | Nama Pengguna | Kata Sandi |
|--------|---------------|------------|
| Admin  | `admin`       | `admin123` |
| Owner  | `owner`       | `owner123` |

> Akun ini untuk **login server sinkronisasi** (`server/server.js`) — terpisah dari
> akun lokal yang didaftarkan lewat form login aplikasi (tidak ada akun demo
> di layar login web).

> **Owner** dapat mengakses semua menu termasuk Pengaturan (manajemen pengguna).
> **Admin** dapat mengakses Kasir, Produk, Riwayat, dan Dashboard.

## ✨ Fitur

- 🔐 **Login & sesi** — autentikasi dengan dua peran: *admin* dan *owner*
- 🌐 **Multi bahasa** — Bahasa Indonesia & English (beralih di pojok kanan atas)
- 📦 **Manajemen produk** — tambah, ubah, hapus produk; kategori, harga, dan stok
- 🛒 **Kasir / transaksi** — keranjang belanja, diskon (%), hitung kembalian otomatis
- 💳 **7 metode pembayaran** — Tunai, GoPay, OVO, DANA, QRIS, BCA VA, Kartu
  (metode bayar dicatat di tiap transaksi; non-tunai tanpa input uang)
- 🧾 **Riwayat transaksi** — cari, lihat detail (termasuk metode bayar), cetak ulang struk
- 📊 **Dashboard laporan** — pendapatan hari ini, produk terlaris, peringatan stok menipis
- 📈 **Laporan shift** — laporan per-kasir per-shift, detail transaksi per shift (owner: semua shift, admin: shift sendiri)
- 🖨️ **Cetak struk** — struk terformat untuk printer termal 58mm
- 📡 **Cetak Bluetooth ESC/POS** — cetak langsung ke printer termal via Bluetooth (Android)
- 📊 **Laporan keuangan ke Google Sheets** — transaksi & pengeluaran otomatis terkirim ke spreadsheet (tab *Transaksi*, *Penjualan*, *Pengeluaran*), siap pivot & chart
- 🔄 **Sinkronisasi data** — sinkronisasi multi-perangkat via server Node.js + SQLite

## 🖨️ Struk untuk Printer Termal 58mm

Struk sudah diformat khusus untuk printer termal **58mm** (area cetak ±48mm):

- Font monospace, teks hitam-putih (tanpa background berwarna agar tajam di kertas termal)
- Lebar konten 48mm, `@page { size: 58mm auto }` saat dicetak/PDF
- Nama item panjang otomatis dipangkas agar tidak meluber
- Ruang pemotongan kertas di bagian bawah
- Di aplikasi Android: cetak via Bluetooth ESC/POS atau dialog cetak sistem (plugin Printer)

## 📡 Cetak Bluetooth (Android)

Fitur ini hanya aktif di APK Android dan memerlukan plugin native `BluetoothEscpos`:

1. Buka **Pengaturan** → kartu **Printer Bluetooth (ESC/POS)**
2. Tekan **Muat Ulang** untuk melihat perangkat Bluetooth yang sudah dipasangkan
3. Pilih printer, tekan **Simpan Printer**
4. Tekan **Cetak Uji** untuk mengirim struk contoh
5. Setelah login & checkout, tekan **🖨️ Cetak Bluetooth** di halaman kasir/riwayat

> Printer harus sudah **dipasangkan (paired)** di pengaturan Bluetooth HP Android.

## 🔄 Sinkronisasi Data Antar-Perangkat

Fitur sinkronisasi memungkinkan dua perangkat (mis. dua kasir) berbagi data via jaringan lokal:

1. Jalankan server: `node server/server.js` (memerlukan **Node.js 22+** dan module `node:sqlite`)
2. Buka **Pengaturan** → kartu **Sinkronisasi Data**
3. Masukkan URL server, mis. `http://192.168.1.10:3000`
4. Masukkan username & password (`admin/admin123` atau `owner/owner123` di server)
5. Tekan **Hubungkan & Sinkronkan**
6. Data akan tersinkronisasi otomatis setiap 60 detik

> Server menyimpan data di `server/kasirpro.db`. Login ke server terpisah dari login lokal.
> Saat server dimulai ulang, token perlu login ulang.

## 📊 Laporan Keuangan ke Google Sheets

Setiap pembayaran & pencatatan pengeluaran **otomatis** dikirim ke Google Spreadsheet
anda (gratis, tanpa login Google di dalam aplikasi):

1. Buat Google Spreadsheet baru di <https://sheets.google.com>
2. **Extensions → Apps Script** → hapus isi `Code.gs` → tempel [`scripts/gs-laporan.gs`](scripts/gs-laporan.gs)
3. Ganti `KUNCI_RAHASIA` dengan kunci buatan Anda sendiri
4. **Deploy → New deployment → Web app** → *Execute as: Me*, *Who has access: Anyone* → salin URL `/exec`
5. **Pengaturan → Google Sheets** → tempel URL + kunci → **Simpan** → **Uji Koneksi**

Aplikasi membuat 3 tab otomatis: **Transaksi** (1 baris per transaksi),
**Penjualan** (1 baris per item → produk terlaris), **Pengeluaran**.
Tanpa internet, data masuk antrean lokal dan dikirim otomatis saat sinyal kembali.

> Panduan lengkap + cara membuat laporan laba rugi: [`docs/GOOGLE-SHEETS.md`](docs/GOOGLE-SHEETS.md)

## 📱 Aplikasi Android (.apk)

Aplikasi web ini dibungkus dengan **Capacitor** menjadi aplikasi Android (WebView native).

### Bangun APK Debug
```bash
npm install                      # install Capacitor CLI dll.
npm run sync:www                 # salin aset web ke www/
npx cap sync android             # sinkronkan aset ke proyek Android
cd android
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 ./gradlew assembleDebug
```

### Bangun APK Rilis (Ditandatangani Sendiri)
```bash
# Keystore sudah tersedia di android/dist/kasirpro-release.keystore
cd android
JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64 ./gradlew assembleRelease
```

APK debug dan rilis tersedia di:
```
dist/KasirPro-v1.0.0.apk                          # debug
dist/KasirPro-v1.0.0-release.apk                  # rilis (self-signed)
```

> **Persyaratan build:** JDK 21+, Android SDK (platform android-36 & build-tools 36).
> APK rilis ditandatangani dengan keystore self-signed di `dist/kasirpro-release.keystore`.
> Untuk produksi, ganti keystore dan password di `android/app/build.gradle` → `signingConfigs.release`.

## ⚙️ Catatan

- Data disimpan per-perangkat/browser (localStorage). Gunakan browser yang sama & `file://` yang sama.
- Kata sandi disimpan dalam bentuk teks biasa di localStorage (cukup untuk demo).
  Untuk produksi, hubungkan ke backend dengan autentikasi yang aman.
- Tombol **Hapus Semua Data** terdapat di Pengaturan (hanya owner).
- `www/` dan `android/` adalah hasil bundel dari sumber di root proyek; jalankan
  `npm run sync:www` setelah mengubah file web sebelum build APK.

  thanks to:
  
  ALLAH
  
  Orang Tua
  
  My Mine
