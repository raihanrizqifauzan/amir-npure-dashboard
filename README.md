# Amir NPURE — Sell-Out Dashboard

Dashboard penjualan distributor NPURE dengan data langsung dari Google Sheets. Dibangun memakai HTML, CSS, dan JavaScript tanpa backend atau build step.

Dashboard saat ini: https://amir-npure.ripplingbike3.chatgpt.site/

## Menjalankan lokal

Dari direktori proyek:

```sh
python3 -m http.server 8080 --directory dist
```

Buka http://localhost:8080. Jangan membuka index.html lewat file:// karena pembacaan data membutuhkan HTTP.

## Struktur

- `dist/index.html`: struktur halaman dan logo.
- `dist/style.css`: tampilan responsif dan grafik.
- `dist/app.js`: pembacaan Google Sheets, filter, perhitungan, analisis dan ekspor CSV.
- `dist/npure-logo.png`: aset logo NPURE yang diberikan pemilik proyek.

## Sumber data dan refresh

Sumber: https://docs.google.com/spreadsheets/d/1VgHE_C_kA5EFQKaWJlHlu8CGxL0fI_M8_O1YYU3Lwpw/edit

Tab sumber memakai `gid=0`. Konfigurasi endpoint ada di konstanta `SHEET_CSV` dalam `dist/app.js`; meskipun namanya demikian, format respons saat ini adalah Google Visualization JSON supaya angka tetap memiliki presisi asli.

Spreadsheet harus dapat dibaca tanpa autentikasi. Data dimuat ketika halaman dibuka, saat tombol Refresh data ditekan, dan setiap lima menit ketika tab aktif. Google dapat melakukan cache. Jika refresh gagal, data terakhir dalam memori halaman dipertahankan dan pesan kesalahan ditampilkan.

Filter dipertahankan ketika data diperbarui. Rentang tanggal yang masih menggunakan batas penuh data akan mengikuti batas terbaru; rentang yang dipilih khusus tetap dipertahankan.

## Aturan bisnis

- Baris tanpa Code SKU atau Nama SKU pusat dianggap tester dan dikecualikan.
- Grade diseragamkan ke huruf besar; DEKORATF menjadi DEKORATIF.
- Toko kosong tidak masuk toko aktif dan retensi.
- Identitas toko menggunakan distributor + nama toko setelah trim dan normalisasi kapitalisasi.
- Toko aktif memiliki QTY dan metrik nilai positif.
- Metrik nilai dapat dipilih: Value atau Value DPP.
- Type dan Series ditampilkan dalam grafik vertikal 100% terpisah. Satu kolom untuk setiap distributor; segmen diurutkan terbesar di bawah. Persentase menggunakan total distributor dalam cakupan filter aktif.
- Grafik 100% menggunakan nilai positif. Nilai negatif tetap masuk ringkasan dan rincian, dengan catatan pada grafik.
- Label lengkap semua kategori tersedia di bawah tiap kolom. Persentase kecil mempertahankan hingga dua desimal; nilai positif di bawah 0,01% ditandai <0,01%.
- Repeat berarti toko membeli pada dua bulan kalender berurutan, bukan jumlah order. Analisis menggunakan pasangan bulan lengkap terakhir yang tersedia dalam filter.
- Analisis follow-up dan cross-sell merupakan kandidat verifikasi, bukan forecast, profit, stok, atau bukti churn permanen.

## Deployment

Publish direktori `dist` pada hosting statis. Tidak ada perintah build yang diperlukan.

**Repository GitHub ini tidak otomatis memperbarui URL ChatGPT Sites yang tercantum di atas.** Deployment GitHub ke hosting lain harus dikonfigurasi tersendiri. Jangan menganggap push ke repository sebagai publikasi ke Sites.

Untuk tim: gunakan branch fitur, pull request, dan review sebelum merge. Setelah koneksi hosting dipasang, branch produksi dapat diatur deploy otomatis. Atur akses anggota tim melalui pengaturan repository; jangan membagikan akun pribadi atau token.

## Pemeriksaan sebelum perubahan diterbitkan

```sh
node --check dist/app.js
```

Periksa filter distributor/periode, Value dan Value DPP, refresh, kondisi tanpa data, jumlah segmen 100%, urutan segmen, label kecil, ekspor CSV, dan tampilan ponsel. Pastikan judul kolom sumber tetap sesuai pembaca data.

## Aset dan akses data

Logo NPURE adalah aset merek pemiliknya. Repository ini tidak memberikan lisensi atas logo atau data sumber. Endpoint spreadsheet terlihat pada kode yang berjalan di browser; penggunaan tanpa backend membutuhkan sumber yang bisa dibaca publik.
