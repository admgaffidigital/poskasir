# Prompt Siap Pakai untuk Antigravity — Aplikasi Kasir Toko Bangunan

> Cara pakai: paste isi setiap "PROMPT" di bawah ke Antigravity secara berurutan (Tahap 1 dulu, tunggu selesai, baru lanjut Tahap 2, dst). Sebelum mulai, upload/lampirkan file `skema-aplikasi-kasir-toko-bangunan.md` sebagai referensi ke project.

---

## Setup Awal (jalankan sekali di awal, sebelum Tahap 1)

```
PROMPT:

Saya sedang membangun aplikasi web kasir untuk toko bangunan. Saya lampirkan file skema
(skema-aplikasi-kasir-toko-bangunan.md) yang berisi struktur menu, skema database, dan alur
bisnis lengkap. Tolong pelajari file itu sebagai acuan utama untuk seluruh project ini.

Stack yang saya inginkan:
- Backend: [ISI: Laravel / Node.js Express / Django]
- Database: [ISI: MySQL / PostgreSQL]
- Frontend: [ISI: React + Tailwind / Laravel Blade / Vue]

Tolong:
1. Buatkan struktur folder project awal sesuai stack di atas
2. Buatkan file migration/schema database untuk SEMUA tabel yang ada di skema (jangan
   generate fitur dulu, cukup struktur database dan skeleton project)
3. Buatkan seeder role dasar (admin, kasir, gudang, keuangan, manager) sesuai skema

Jangan generate modul fitur lain dulu, saya akan minta bertahap.
```

---

## Tahap 1 — MVP (Login, Produk & Kategori, Transaksi, Laporan dasar)

```
PROMPT:

Lanjutkan project kasir toko bangunan ini (acuan tetap file skema yang sudah dilampirkan).
Sekarang buatkan modul-modul berikut secara lengkap (backend + frontend):

1. LOGIN
   - Form login dengan validasi
   - Autentikasi berbasis role (admin, kasir, gudang, keuangan, manager)
   - Redirect ke dashboard sesuai role setelah login
   - Middleware proteksi route berdasarkan role

2. PRODUK & KATEGORI
   - CRUD kategori produk
   - CRUD produk (kode produk/SKU, nama, kategori, satuan, harga beli, harga jual, stok,
     stok minimum, status aktif/nonaktif)
   - Pencarian & filter produk berdasarkan kategori/nama/kode
   - Validasi stok tidak boleh minus

3. TRANSAKSI (KASIR/POS)
   - Halaman kasir: cari produk (nama/scan kode), tambah ke keranjang, ubah qty, hapus item
   - Hitung otomatis: subtotal, diskon, total
   - Pilih metode bayar (tunai, transfer, qris)
   - Simpan transaksi ke tabel transaksi_penjualan dan transaksi_penjualan_detail
   - Stok produk otomatis berkurang setelah transaksi tersimpan (catat ke stok_log)
   - Cetak/preview struk sederhana (bisa print browser)

4. LAPORAN DASAR
   - Laporan penjualan harian/mingguan/bulanan (filter tanggal)
   - Total omzet, jumlah transaksi, produk terlaris

Tolong sertakan validasi input di setiap form dan tampilkan pesan error yang jelas.
```

---

## Tahap 2 — Pembelian, Supplier, Hutang, Stok Log

```
PROMPT:

Lanjutkan project kasir toko bangunan ini. Sekarang buatkan modul berikut:

1. DAFTAR SUPPLIER
   - CRUD data supplier (nama, kontak, no hp, alamat, email)

2. PEMBELIAN (KE SUPPLIER)
   - Form buat pembelian: pilih supplier, tambah produk & qty & harga beli
   - Status pembelian: pending / diterima
   - Saat status "diterima", stok produk otomatis bertambah (catat ke stok_log jenis masuk)
   - Input jatuh tempo pembayaran & status bayar (lunas/hutang)

3. HUTANG (ke supplier)
   - Otomatis buat record hutang saat pembelian berstatus "hutang"
   - Halaman daftar hutang dengan filter jatuh tempo & status (belum/lunas/sebagian)
   - Fitur bayar hutang (cicilan), update sisa hutang otomatis
   - Riwayat pembayaran hutang per supplier

4. STOK LOG / KARTU STOK
   - Halaman riwayat mutasi stok per produk (masuk/keluar/adjustment)
   - Notifikasi/badge untuk produk dengan stok di bawah stok minimum

Pastikan perhitungan sisa hutang selalu konsisten setiap ada pembayaran baru.
```

---

## Tahap 3 — Pelanggan, Piutang, Loyalty Poin

```
PROMPT:

Lanjutkan project kasir toko bangunan ini. Sekarang buatkan modul berikut:

1. DAFTAR PELANGGAN
   - CRUD data pelanggan (nama, no hp, alamat, email, tipe: umum/member)
   - Tambahkan opsi pilih pelanggan saat transaksi di menu Kasir (integrasi ke Tahap 1)

2. PIUTANG (dari pelanggan)
   - Tambahkan opsi metode bayar "piutang" di halaman transaksi kasir
   - Otomatis buat record piutang saat transaksi dengan metode piutang
   - Halaman daftar piutang dengan filter jatuh tempo & status
   - Fitur bayar piutang (cicilan), update sisa piutang otomatis

3. LOYALTY POIN PELANGGAN
   - Halaman pengaturan poin: nominal belanja per poin, nilai tukar poin ke rupiah,
     minimum penukaran
   - Poin otomatis bertambah setiap transaksi lunas oleh pelanggan member
   - Fitur tukar poin jadi diskon saat transaksi baru
   - Riwayat poin per pelanggan (dapat/tukar)

Pastikan perhitungan poin dan piutang terintegrasi dengan modul Transaksi yang sudah ada,
jangan buat ulang dari nol.
```

---

## Tahap 4 — Karyawan, Pengaturan Toko, Laporan Lengkap

```
PROMPT:

Lanjutkan project kasir toko bangunan ini. Sekarang buatkan modul berikut untuk melengkapi
seluruh sistem:

1. KARYAWAN
   - CRUD data karyawan (nama, jabatan, no hp, alamat, tanggal masuk, gaji pokok, status)
   - Hubungkan ke akun user sistem (jika karyawan tsb punya akses login)

2. PENGATURAN TOKO
   - Halaman pengaturan: nama toko, alamat, no telp, email, logo, npwp, format invoice,
     persen pajak, mata uang, footer struk
   - Terapkan pengaturan ini ke tampilan struk & invoice

3. LAPORAN LENGKAP
   - Laporan penjualan (per kasir, per produk, per periode)
   - Laporan pembelian
   - Laporan stok (stok menipis, slow moving)
   - Laporan hutang & piutang (outstanding, jatuh tempo)
   - Laporan laba rugi sederhana (omzet - HPP - biaya)
   - Laporan poin loyalty
   - Semua laporan bisa difilter tanggal dan diekspor ke Excel/PDF

Setelah semua modul jadi, tolong cek ulang: apakah ada bug integrasi antar modul (misalnya
stok tidak update, poin tidak bertambah, hutang/piutang tidak sinkron) dan perbaiki.
```

---

## Tips Tambahan Saat Pakai Antigravity

- **Satu tahap = satu sesi/perintah besar.** Jangan gabung beberapa tahap sekaligus, agar Antigravity fokus dan hasil kode lebih mudah ditelusuri jika ada error.
- **Selalu review hasil sebelum lanjut ke tahap berikutnya** — cek apakah struktur tabel yang dibuat konsisten dengan skema.
- Jika Antigravity membuat penamaan tabel/kolom berbeda dari skema, cukup tambahkan di prompt: *"gunakan penamaan tabel persis seperti di file skema, jangan diubah."*
- Untuk kebutuhan spesifik toko bangunan (misalnya satuan sak/batang/m3, harga grosir per kuantitas tertentu), sebutkan eksplisit di prompt Tahap 1 karena ini belum otomatis tercakup di skema dasar.
