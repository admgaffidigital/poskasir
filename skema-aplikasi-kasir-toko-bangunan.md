# Skema Aplikasi Kasir Toko Bangunan (Web-Based)

## 1. Ringkasan Sistem

Aplikasi berbasis web untuk mengelola operasional toko bangunan secara menyeluruh: penjualan (kasir), pembelian ke supplier, stok produk, keuangan (hutang/piutang), pelanggan & loyalty point, karyawan, serta laporan.

**Arsitektur umum:**
- Frontend: SPA (React/Vue) atau server-rendered (Laravel Blade / Next.js)
- Backend: REST API (Node.js/Express, Laravel, atau Django)
- Database: MySQL/PostgreSQL
- Autentikasi: JWT/Session + Role-Based Access Control (RBAC)
- Cetak struk: thermal printer via ESC/POS (browser print / plugin)

---

## 2. Struktur Role Pengguna

| Role | Akses |
|---|---|
| **Owner/Admin** | Semua menu, termasuk pengaturan toko & laporan keuangan |
| **Kasir** | Transaksi penjualan, cek stok, cek pelanggan |
| **Gudang/Staf Produk** | Menu produk, kategori, stok, pembelian |
| **Keuangan** | Hutang, piutang, laporan |
| **Manager** | Laporan, karyawan, approval |

---

## 3. Struktur Menu Aplikasi

```
├── 1. Login
├── 2. Dashboard (ringkasan penjualan, stok kritis, hutang jatuh tempo)
├── 3. Transaksi (Kasir/POS)
├── 4. Produk & Kategori
├── 5. Pembelian (ke Supplier)
├── 6. Daftar Pelanggan
├── 7. Daftar Supplier
├── 8. Hutang & Piutang
├── 9. Loyalty Poin Pelanggan
├── 10. Karyawan
├── 11. Laporan
└── 12. Pengaturan Toko
```

---

## 4. Skema Database per Modul

### 4.1 Login & User

**tabel `users`**
| Field | Tipe | Ket |
|---|---|---|
| id | INT PK | |
| nama | VARCHAR | |
| username | VARCHAR (unique) | |
| password | VARCHAR (hashed) | |
| role_id | INT FK -> roles.id | |
| foto | VARCHAR | opsional |
| status | ENUM(aktif, nonaktif) | |
| created_at, updated_at | TIMESTAMP | |

**tabel `roles`**
| id | nama_role | permission (JSON) |

---

### 4.2 Produk & Kategori

**tabel `kategori_produk`**
| id | nama_kategori | deskripsi |

**tabel `produk`**
| Field | Tipe | Ket |
|---|---|---|
| id | INT PK | |
| kode_produk (SKU) | VARCHAR | barcode |
| nama_produk | VARCHAR | |
| kategori_id | FK | |
| satuan | VARCHAR | pcs, sak, batang, kg, m3, dll |
| harga_beli | DECIMAL | |
| harga_jual | DECIMAL | |
| stok | INT | |
| stok_minimum | INT | untuk notifikasi restock |
| lokasi_rak | VARCHAR | opsional |
| gambar | VARCHAR | |
| status | ENUM(aktif, nonaktif) | |

**tabel `stok_log`** (kartu stok / mutasi)
| id | produk_id | jenis (masuk/keluar/adjustment) | qty | referensi (no transaksi/pembelian) | tanggal | keterangan |

---

### 4.3 Transaksi (Kasir/POS)

**tabel `transaksi_penjualan`**
| Field | Tipe | Ket |
|---|---|---|
| id | INT PK | |
| no_transaksi | VARCHAR | invoice |
| pelanggan_id | FK (nullable) | umum/member |
| kasir_id | FK -> users | |
| tanggal | DATETIME | |
| subtotal | DECIMAL | |
| diskon | DECIMAL | |
| pajak | DECIMAL | opsional |
| total | DECIMAL | |
| metode_bayar | ENUM(tunai, transfer, qris, kartu, piutang) | |
| status_bayar | ENUM(lunas, belum_lunas) | jika piutang |
| poin_didapat | INT | |
| created_at | TIMESTAMP | |

**tabel `transaksi_penjualan_detail`**
| id | transaksi_id | produk_id | qty | harga_satuan | subtotal | diskon_item |

---

### 4.4 Pembelian (ke Supplier)

**tabel `transaksi_pembelian`**
| id | no_pembelian | supplier_id | user_id | tanggal | total | status (pending/diterima) | jatuh_tempo | status_bayar (lunas/hutang) |

**tabel `transaksi_pembelian_detail`**
| id | pembelian_id | produk_id | qty | harga_beli_satuan | subtotal |

---

### 4.5 Pelanggan

**tabel `pelanggan`**
| id | nama | no_hp | alamat | email | tipe (umum/member) | tanggal_daftar | total_poin |

---

### 4.6 Supplier

**tabel `supplier`**
| id | nama_supplier | kontak | no_hp | alamat | email | keterangan |

---

### 4.7 Hutang & Piutang

**tabel `hutang`** (toko ke supplier)
| id | supplier_id | pembelian_id (FK) | jumlah | jatuh_tempo | sisa_hutang | status (belum/lunas/sebagian) |

**tabel `piutang`** (pelanggan ke toko)
| id | pelanggan_id | transaksi_id (FK) | jumlah | jatuh_tempo | sisa_piutang | status |

**tabel `pembayaran_hutang`**
| id | hutang_id | tanggal_bayar | jumlah_bayar | metode | keterangan |

**tabel `pembayaran_piutang`**
| id | piutang_id | tanggal_bayar | jumlah_bayar | metode | keterangan |

---

### 4.8 Loyalty Poin

**tabel `pengaturan_poin`**
| id | nominal_belanja_per_poin (misal: Rp10.000 = 1 poin) | poin_ke_rupiah (misal: 1 poin = Rp100) | min_penukaran |

**tabel `poin_log`**
| id | pelanggan_id | transaksi_id | jenis (dapat/tukar) | jumlah_poin | keterangan | tanggal |

---

### 4.9 Karyawan

**tabel `karyawan`**
| id | user_id (FK, nullable jika bukan user sistem) | nama | jabatan | no_hp | alamat | tanggal_masuk | gaji_pokok | status |

**tabel `absensi`** *(opsional, jika diperlukan)*
| id | karyawan_id | tanggal | jam_masuk | jam_keluar | status |

---

### 4.10 Pengaturan Toko

**tabel `pengaturan_toko`**
| id | nama_toko | alamat | no_telp | email | logo | npwp | format_invoice | pajak_persen | mata_uang | footer_struk |

---

## 5. Alur Proses Bisnis Utama

### A. Alur Transaksi Penjualan (Kasir)
1. Kasir login → pilih menu Transaksi
2. Scan/cari produk → masuk keranjang
3. Pilih pelanggan (opsional, untuk dapat poin)
4. Sistem hitung subtotal, diskon, pajak, total
5. Pilih metode bayar (tunai/transfer/qris/piutang)
6. Jika piutang → otomatis buat record di tabel `piutang`
7. Stok produk otomatis berkurang (`stok_log`)
8. Poin loyalty otomatis dihitung & ditambahkan
9. Cetak struk

### B. Alur Pembelian ke Supplier
1. Staf buat PO (purchase order) ke supplier
2. Barang diterima → update stok (`stok_log` jenis masuk)
3. Jika pembayaran tidak langsung lunas → buat record `hutang`
4. Finance bayar hutang bertahap → dicatat di `pembayaran_hutang`

### C. Alur Hutang & Piutang
- Dashboard menampilkan daftar jatuh tempo
- Notifikasi H-3 sebelum jatuh tempo (opsional)
- Pembayaran sebagian (cicilan) didukung, sisa otomatis terhitung

### D. Alur Loyalty Poin
1. Setiap transaksi lunas oleh pelanggan member → poin dihitung otomatis
2. Pelanggan bisa tukar poin jadi diskon saat transaksi berikutnya
3. Riwayat poin bisa dilihat di profil pelanggan

---

## 6. Menu Laporan (Rekomendasi Isi)

- Laporan Penjualan (harian/mingguan/bulanan, per kasir, per produk)
- Laporan Pembelian
- Laporan Stok (stok menipis, stok mati/slow moving)
- Laporan Hutang & Piutang (jatuh tempo, outstanding)
- Laporan Laba Rugi sederhana (omzet - HPP - biaya)
- Laporan Poin Loyalty
- Laporan Kinerja Karyawan/Kasir

---

## 7. Rekomendasi Tech Stack

| Layer | Opsi |
|---|---|
| Frontend | React.js + Tailwind, atau Laravel Blade |
| Backend | Laravel (PHP) — cocok untuk POS UKM, atau Node.js + Express |
| Database | MySQL |
| Autentikasi | Laravel Sanctum / JWT |
| Cetak Struk | ESC/POS via WebUSB atau plugin printer thermal |
| Hosting | VPS (DigitalOcean/local hosting) untuk 1 cabang; multi-cabang bisa pertimbangkan cloud + sync |

---

## 8. Prioritas Pengembangan (Saran Bertahap)

1. **Tahap 1 (MVP):** Login, Produk & Kategori, Transaksi, Laporan Penjualan dasar
2. **Tahap 2:** Pembelian, Supplier, Hutang, Stok log
3. **Tahap 3:** Pelanggan, Piutang, Loyalty Poin
4. **Tahap 4:** Karyawan, Pengaturan Toko lanjutan, Laporan lengkap
