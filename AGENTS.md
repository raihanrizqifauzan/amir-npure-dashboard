# AGENTS.md — Konteks pengembangan Amir NPURE Dashboard

Panduan untuk developer dan AI coding agent yang melanjutkan proyek ini. Berlaku untuk seluruh repository. Baca file ini dan README.md sebelum mengubah kode. Jika instruksi tugas terbaru dari pemilik proyek berbeda, utamakan instruksi terbaru tersebut dan jelaskan dampak perubahan aturan bisnis.

## 1. Tujuan dan sumber kebenaran

Dashboard analisis **sell-out distributor NPURE ke toko**. Pengguna utama adalah tim sales/non-IT yang perlu menelusuri penjualan, kontribusi produk, distributor, retensi, serta prioritas follow-up.

- Repository: https://github.com/raihanrizqifauzan/amir-npure-dashboard
- Branch produksi: `master`.
- URL produksi utama: https://raihanrizqifauzan.github.io/amir-npure-dashboard/
- Versi lama: https://amir-npure.ripplingbike3.chatgpt.site/
- GitHub Pages dan ChatGPT Sites adalah dua deployment terpisah. Push ke repository ini hanya memperbarui GitHub Pages sesuai workflow; tidak menyinkronkan ChatGPT Sites.
- Gunakan kode dan spreadsheet terkini sebagai sumber kebenaran. Jumlah baris, distributor, SKU, nilai penjualan, dan rentang tanggal dapat bertambah. Jangan hardcode hasil analisis dari dokumentasi atau percakapan lama.
- Data ini bukan sell-out toko ke konsumen akhir. Jangan mengganti maknanya menjadi penjualan konsumen, stok, profit, atau margin.

## 2. Stack dan cara menjalankan

Aplikasi statis, tanpa backend, database server, dependency npm, atau build framework. HTML + CSS + JavaScript biasa. Grafik dibuat dengan HTML/CSS, bukan library chart eksternal.

```sh
git clone https://github.com/raihanrizqifauzan/amir-npure-dashboard.git
cd amir-npure-dashboard
git switch master
git pull --ff-only origin master
git switch -c feat/nama-perubahan
python3 -m http.server 8080 --directory dist
```

Buka http://localhost:8080. Pada Windows, jika diperlukan gunakan `py -m http.server 8080 --directory dist`. Jangan menjalankan halaman lewat `file://`.

Pemeriksaan sintaks:
```sh
node --check dist/app.js
```

## 3. File yang diubah berdasarkan kebutuhan

| Kebutuhan | File / fungsi utama |
| --- | --- |
| Judul, copy, susunan section, field, navigasi | `dist/index.html` |
| Warna, ukuran, layout, responsif, label grafik | `dist/style.css` |
| Pembacaan spreadsheet dan normalisasi | `dist/app.js`: `SHEET_CSV`, `cleanGoogle`, `cleanLive`, `sourceDate`, `fetchLive` |
| Filter, ringkasan, tren | `render`, `defs`, `setupPickers`, `syncPickers` |
| Refresh dan mempertahankan filter | `refreshLive`, `rebuildPicks`, `showSync` |
| Kontribusi SKU per distributor | `renderSKU` |
| Type/Series vertikal 100% | `composition`, `rankedSegments`, `mixChart`, `renderMixCharts`, `chartPercent` |
| Analisis dan retensi | `renderAnalysis`, `eligiblePair`, `active`, `key` |
| Tabel rincian dan CSV | `renderDetail`, `table`, `download` |
| Logo | `dist/npure-logo.png` dan CSS `.brandcrop` |
| Auto-deploy | `.github/workflows/pages.yml` |

Kode awal masih padat dan CSS memiliki override berurutan. Baca seluruh definisi selector sebelum menambah override. Jika merapikan format, pisahkan perubahan format besar dari perubahan perilaku agar mudah direview. Pertahankan ID elemen yang dirujuk JavaScript.

## 4. Sumber data dan struktur

Google Sheets:
https://docs.google.com/spreadsheets/d/1VgHE_C_kA5EFQKaWJlHlu8CGxL0fI_M8_O1YYU3Lwpw/edit

Tab saat pembuatan: `Sheet1`, `gid=0`. Endpoint terdapat pada `SHEET_CSV`. Walaupun nama konstantanya CSV, respons yang digunakan sekarang adalah **Google Visualization JSON**, bukan CSV.

Header wajib yang dibaca:
`Tanggal`, `Week`, `Nama Distributor`, `Nama Toko`, `Grade`, `Jumlah Qty`, `Value`, `Value DPP`, `Code SKU`, `Nama SKU`, `SERIES`, `TYPE`.

Kolom mentah lain, seperti nama/kode SKU distributor dan Nama Toko Distributor, bukan identitas utama analisis.

Array hasil normalisasi yang dipakai aplikasi:

| Indeks | Isi |
| --- | --- |
| 0 | Tanggal ISO YYYY-MM-DD |
| 1 | Week |
| 2 | Nama Distributor |
| 3 | Nama Toko |
| 4 | Grade |
| 5 | Quantity numerik |
| 6 | Value numerik |
| 7 | Value DPP numerik |
| 8 | Code SKU pusat |
| 9 | Nama SKU pusat |
| 10 | Series |
| 11 | Type |

Jangan mengubah susunan indeks tanpa memperbarui seluruh penggunaan. Jangan mengambil angka dari teks terformat apabila nilai numerik asli tersedia: CSV terformat pernah membulatkan desimal dan membuat total berbeda dari sumber.

Sumber harus dapat dibaca tanpa login. Jangan menaruh token, password, service-account key, atau kredensial di frontend/repository. Spreadsheet sumber diperlakukan read-only oleh pekerjaan dashboard, kecuali pemilik secara eksplisit meminta perubahan spreadsheet.

## 5. Aturan bisnis yang perlu dipertahankan

### Tester dan normalisasi

- Pemilik mengonfirmasi baris tanpa pemetaan kode/nama SKU pusat adalah **tester**.
- Implementasi saat ini mengecualikan baris ketika Code SKU **atau** Nama SKU pusat kosong. Pertahankan aturan ini kecuali pemilik mengubah definisi tester.
- Grade diseragamkan ke huruf besar; `c` menjadi `C`.
- `DEKORATF` diseragamkan menjadi `DEKORATIF`.
- Error lookup yang diawali `#` ditandai “Belum dipetakan”. Nilai kategori kosong juga mendapat fallback pada grafik komposisi.
- Jangan menebak klasifikasi Type/Series dari nama produk tanpa mapping yang disepakati.
- Nama toko kosong ditandai “(Tanpa nama toko)” dan dikecualikan dari hitungan toko aktif/retensi.
- Identitas toko adalah pasangan distributor + nama toko, dengan trim, perapian spasi, dan huruf besar. Toko bernama sama di distributor berbeda dihitung terpisah.

### Filter, metrik, dan pembagi

- Filter: bulan, distributor, toko, grade, SKU, Series, Type, serta rentang tanggal.
- Filter bulan dan rentang tanggal bekerja **bersamaan**, bukan saling menggantikan.
- Metrik nilai: `Value` (indeks 6) atau `Value DPP` (indeks 7). Quantity tetap ditampilkan sebagai konteks volume.
- Ringkasan, grafik, dan analisis mengikuti seluruh filter global.
- Persentase SKU dalam distributor = nilai SKU pada distributor / total nilai distributor setelah filter.
- Persentase SKU terhadap total = nilai SKU pada distributor / seluruh nilai terfilter.
- Pencarian SKU, pilihan distributor khusus pada section SKU, dan batas 10/20/semua hanya mempersempit tampilan section tersebut; tidak mengubah pembagi/filter global.
- Toko aktif memiliki baris quantity > 0 dan metrik nilai > 0.

### Grafik Type dan Series

- Dua grafik terpisah, vertikal 100%.
- Satu kolom untuk satu distributor.
- Kolom distributor diurutkan dari total nilai terbesar ke terkecil.
- Dalam **setiap** kolom, kategori diurutkan mandiri berdasarkan nilainya: terbesar di bawah, terkecil di atas.
- Warna kategori konsisten antar-distributor dalam render yang sama. Jangan memberi warna berdasarkan peringkat segmen per distributor.
- Grafik komposisi memakai baris bernilai positif; negatif tetap masuk ringkasan/rincian dan perlu catatan bila ada.
- Jangan membesarkan tinggi minimum segmen kecil karena akan mengubah representasi persentase.
- Label di dalam segmen tampil bila ruang cukup. Nama dan persentase **seluruh kategori** wajib tetap terlihat di bawah masing-masing kolom, tanpa harus hover.
- Persentase kecil mempertahankan hingga dua desimal; nilai positif <0,01% ditulis `<0,01%`.
- Tooltip/detail saat hover, klik, atau fokus tetap tersedia. Pertahankan navigasi keyboard dan aksesibilitas.
- Layar kecil dapat menggeser grafik secara horizontal.

### Retensi dan analisis

- Repeat berarti toko membeli pada dua bulan kalender berurutan, bukan jumlah order.
- Pembagi retensi = toko aktif bulan sebelumnya; pembilang = toko tersebut yang kembali aktif pada bulan berikutnya.
- Gunakan pasangan bulan lengkap terakhir yang dapat dibandingkan dalam filter. Bulan parsial atau pilihan bulan tidak berurutan jangan dipaksa dibandingkan.
- Januari 2026 pada sumber awal dimulai 2 Januari; jangan menganggap selalu bulan penuh.
- Follow-up memakai riwayat penjualan sebelumnya untuk prioritas, bukan estimasi kerugian.
- Cross-sell merupakan kandidat untuk diverifikasi. Filter SKU yang sempit dapat membatasi deteksi peluang.
- Jangan menyimpulkan penyebab pertumbuhan, churn permanen, kebutuhan stok, margin, atau performa individu dari data ini saja.

## 6. Pembaruan langsung

- Fetch saat halaman dibuka, tombol Refresh data ditekan, dan tiap 300.000 ms ketika tab terlihat.
- Google dapat melakukan cache; waktu “terakhir diperiksa” bukan jaminan perubahan sudah langsung keluar dari cache Google.
- Saat refresh gagal, data terakhir yang berhasil dimuat dalam memori halaman dipertahankan dengan pesan kesalahan.
- Filter aktif dipertahankan sejauh opsinya masih tersedia setelah refresh.
- Jika pengguna memakai batas tanggal penuh dataset, rentang mengikuti batas data terbaru. Rentang khusus pengguna tetap dipertahankan.
- Penambahan data spreadsheet tidak membutuhkan commit atau deploy ulang.
- Jangan menyimpan snapshot data produksi ke repository sebagai pengganti fetch langsung.

## 7. Desain dan UX

- Bahasa antarmuka Indonesia; target user non-IT.
- Tema hijau/teal, latar terang, header gradient, logo NPURE.
- Filter dropdown dengan pencarian dan checkbox; jangan kembali ke multi-select Ctrl+klik.
- Gunakan penanda filter aktif, tombol reset, pesan loading/error/empty yang jelas.
- Pertahankan pemisahan ringkasan, tren, Type/Series, SKU, analisis, dan rincian.
- Logo sumber berbentuk persegi dengan banyak ruang kosong; CSS melakukan framing agar logo terlihat proporsional. Periksa framing ketika mengganti gambar.
- Jangan mengubah logo/merek atau menambah fitur di luar scope tugas.

## 8. Alur kerja tim dan deployment

1. Ambil kode terbaru dan buat branch fitur.
2. Terapkan adjustment yang diminta; jangan mengubah aturan bisnis diam-diam.
3. Jalankan pemeriksaan dan uji perilaku yang relevan.
4. Commit, push branch fitur, lalu buat pull request ke `master`.
5. Setelah review dan merge, workflow menerbitkan dashboard jika perubahan menyentuh `dist/**` atau `.github/workflows/pages.yml`.
6. Pastikan workflow sukses dan halaman produksi memuat perubahan.

```sh
git add dist/index.html dist/style.css dist/app.js
git commit -m "feat: jelaskan perubahan dashboard"
git push -u origin feat/nama-perubahan
```

Sesuaikan daftar file dengan perubahan nyata. Jangan commit file sensitif atau file sementara.

Workflow saat ini memeriksa `node --check dist/app.js`, mengunggah folder `dist`, lalu deploy ke environment `github-pages`. Tidak ada build npm. Perubahan dokumen saja tidak memicu deployment. Workflow dapat dijalankan manual melalui Actions → Deploy dashboard to GitHub Pages → Run workflow.

Repo publik dapat dibaca/di-clone siapa pun, tetapi akses push tetap dibatasi. Pemilik menambahkan rekan melalui Settings → Collaborators → Add people. Rekan tanpa izin tulis dapat fork, membuat perubahan, dan mengajukan PR dari fork. Jangan berbagi password atau akun GitHub pribadi.

## 9. Verifikasi sesuai perubahan

- Minimal: `node --check dist/app.js`.
- Perubahan tampilan: periksa desktop dan ponsel; label kecil tidak tumpang tindih/terpotong.
- Perubahan filter/data: uji satu/multi distributor, periode, Value/DPP, reset, refresh, dan hasil kosong.
- Perubahan perhitungan: gunakan fixture kecil dengan hasil manual yang diketahui, plus data sumber terkini bila tersedia.
- Grafik: jumlah proporsi tiap distributor ~100% sebelum pembulatan; urutan turun per kolom; pembagi berbeda antar-distributor; semua nama dan persentase tetap dapat dibaca.
- Retensi: pasangan bulan lengkap, pilihan tidak berurutan, dan bulan parsial.
- CSV: pastikan escaping dan isi sesuai filter/pencarian yang berlaku.
- Validasi source sheet dan tanggal campuran jika pembaca data berubah.
- Jangan menambahkan tes yang hanya meniru implementasi. Pilih pengujian yang membuktikan hasil bisnis.
- Laporkan apa yang diubah, apa yang diuji, dan batas verifikasi. Jangan mengklaim deployed hanya karena commit/push berhasil; cek Actions dan halaman produksi.

## 10. Prompt yang bisa diberikan ke agent lain

> Baca AGENTS.md dan README.md di repository ini. Saya ingin melakukan adjustment: [jelaskan perubahan].
> Mulai dari branch master terbaru, kerjakan di branch fitur, dan pertahankan aturan bisnis yang tidak termasuk scope.
> Jalankan pemeriksaan yang relevan, jelaskan dampak filter/perhitungan jika berubah, lalu buat pull request.
> Target produksi adalah GitHub Pages. Jangan deploy ke ChatGPT Sites atau mengubah spreadsheet sumber tanpa instruksi.
