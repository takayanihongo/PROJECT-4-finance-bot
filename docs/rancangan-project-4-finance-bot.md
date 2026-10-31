# Rancangan Proyek 4 — Finance Bot WhatsApp

**Status dokumen:** Rancangan awal  
**Versi:** 1.0  
**Tanggal:** 31 Oktober 2026

---

## 1. Ringkasan proyek

Finance Bot adalah bot WhatsApp untuk membantu mencatat dan memantau keuangan pribadi/rumah tangga melalui pesan chat biasa.

Contoh pesan:
- `tadi beli bensin 20 ribu`
- `ibu dapat uang 500000`
- `pengeluaran bulan ini`
- `berapa saldo sekarang?`

Bot akan membaca pesan, mengidentifikasi maksudnya, memvalidasi data, menyimpan transaksi ke database, lalu mengirim balasan.

**Tujuan awal:** membuat versi sederhana yang bisa digunakan oleh keluarga terlebih dahulu. Pengembangan untuk banyak pengguna atau usaha kecil dilakukan setelah versi awal stabil.

## 2. Tujuan dan batasan versi pertama

### Target utama (MVP)

1. Menerima dan mengirim pesan WhatsApp.
2. Mencatat pemasukan.
3. Mencatat pengeluaran.
4. Menyimpan nominal, kategori, deskripsi, dan waktu transaksi.
5. Menampilkan saldo.
6. Menampilkan riwayat transaksi.
7. Menampilkan ringkasan pemasukan dan pengeluaran berdasarkan periode.
8. Membatasi akses agar pengguna hanya dapat mengakses data yang menjadi haknya.

### Belum menjadi prioritas

- Dashboard website.
- Integrasi bank atau dompet digital.
- OCR struk belanja.
- Grafik interaktif.
- Multi-cabang atau akuntansi bisnis kompleks.
- AI yang boleh menjalankan query database secara bebas.

Fitur tambahan tersebut dipertimbangkan setelah MVP berfungsi dan aman.

## 3. Perangkat dan lingkungan

| Perangkat | Peran |
|---|---|
| Laptop ASUS Windows 11 | Menulis kode, debugging, pengujian, dan Git |
| Redmi Note 5 Pro (whyred) | Perangkat host untuk menjalankan bot |
| Nubia Neo 2 5G | Perangkat penguji dari sisi pengguna |
| GitHub | Menyimpan riwayat kode proyek |

### Teknologi yang direncanakan

| Teknologi | Fungsi |
|---|---|
| JavaScript | Bahasa pemrograman |
| Node.js | Runtime backend |
| npm | Mengelola package dan script |
| Baileys | Koneksi WhatsApp tidak resmi untuk prototipe |
| SQLite | Database lokal |
| Termux | Lingkungan Linux di Android |
| Git | Version control |
| GitHub | Remote repository |

**Catatan risiko WhatsApp:** Baileys bukan WhatsApp Business Platform resmi. Perubahan WhatsApp dapat membuat integrasi rusak dan penggunaan library tidak resmi memiliki risiko terhadap akun. Untuk layanan produksi yang serius, evaluasi WhatsApp Business Platform resmi.

## 4. Arsitektur sistem

Alur utama:

```text
Pengguna WhatsApp
       |
       v
WhatsApp Bot (Baileys)
       |
       v
Message Handler
       |
       v
Intent Parser / AI Parser
       |
       v
Validasi dan Otorisasi
       |
       v
Business Logic
       |
       v
Database SQLite
       |
       v
Hasil diproses menjadi balasan
       |
       v
Pengguna WhatsApp
```

### Tanggung jawab setiap bagian

- **WhatsApp Bot:** menerima dan mengirim pesan.
- **Message Handler:** memeriksa pesan masuk dan meneruskan ke proses yang sesuai.
- **Intent Parser:** mengenali perintah atau informasi transaksi.
- **Validasi dan Otorisasi:** memastikan nominal, jenis transaksi, dan identitas pengguna valid.
- **Business Logic:** menghitung saldo, ringkasan, dan riwayat.
- **Database:** menyimpan data secara persisten.

AI hanya bertugas mengusulkan hasil ekstraksi terstruktur. Backend tetap wajib memvalidasi dan menentukan apakah data boleh disimpan. AI tidak diberi akses database bebas.

## 5. Rancangan fitur

### A. Pencatatan transaksi

Contoh:
`beli bensin 20000`

Hasil yang diharapkan:

```json
{
  "type": "expense",
  "amount": 20000,
  "category": "transportasi",
  "description": "beli bensin"
}
```

Bot meminta konfirmasi jika pesan ambigu atau data penting belum jelas.

### B. Pemasukan

Contoh:
`gaji masuk 1500000`

Data disimpan sebagai transaksi `income`.

### C. Cek saldo

Contoh:
`saldo`

Bot menghitung:

```text
Total pemasukan - Total pengeluaran
```

Saldo dihitung dari transaksi yang tercatat, bukan dianggap sama dengan saldo rekening bank.

### D. Riwayat transaksi

Contoh:
`riwayat 10`

Bot menampilkan sepuluh transaksi terbaru milik pengguna yang sedang meminta.

### E. Ringkasan periode

Contoh:
`laporan bulan ini`

Bot menampilkan total pemasukan, total pengeluaran, dan selisih untuk periode tersebut.

## 6. Rancangan database

Database awal bernama `finance.db`.

### Tabel `users`

| Kolom | Tipe | Keterangan |
|---|---|---|
| id | INTEGER | Primary key |
| phone_number | TEXT | Nomor WhatsApp unik |
| display_name | TEXT | Nama tampilan |
| created_at | TEXT | Waktu dibuat |
| is_active | INTEGER | Status pengguna |

### Tabel `transactions`

| Kolom | Tipe | Keterangan |
|---|---|---|
| id | INTEGER | Primary key |
| user_id | INTEGER | Foreign key ke users |
| type | TEXT | `income` atau `expense` |
| amount | INTEGER | Nominal rupiah positif |
| category | TEXT | Kategori transaksi |
| description | TEXT | Keterangan |
| occurred_at | TEXT | Waktu transaksi terjadi |
| created_at | TEXT | Waktu data dicatat |
| updated_at | TEXT | Waktu data diperbarui |

Relasi: satu pengguna dapat memiliki banyak transaksi (one-to-many).

Index awal:
- `(user_id, occurred_at)`
- `(user_id, type, occurred_at)`

Foreign key SQLite harus diaktifkan pada setiap koneksi database dengan `PRAGMA foreign_keys = ON`.

## 7. Struktur folder proyek

Struktur yang direncanakan:

```text
finance-bot/
├── src/
│   ├── index.js
│   ├── config/
│   │   └── ...
│   ├── database/
│   │   ├── database.js
│   │   └── ...
│   ├── whatsapp/
│   │   └── ...
│   ├── handlers/
│   │   └── ...
│   ├── services/
│   │   └── ...
│   ├── utils/
│   │   └── ...
│   └── ...
├── tests/
├── data/
│   └── finance.db
├── .env
├── .gitignore
├── package.json
└── README.md
```

Struktur ini adalah target bertahap. Folder atau file yang belum dibuat tidak dianggap sudah tersedia.

## 8. Keamanan dan perlindungan data

1. Identitas pengguna diambil dari metadata WhatsApp yang telah diverifikasi, bukan dari `user_id` yang diketik pengguna.
2. Setiap query transaksi wajib dibatasi berdasarkan `user_id` yang telah diotorisasi.
3. Gunakan prepared statements, bukan menyusun SQL dari input mentah.
4. Validasi nominal: harus berupa bilangan bulat positif dan berada dalam batas yang masuk akal.
5. Pesan ambigu tidak langsung disimpan; minta klarifikasi atau konfirmasi.
6. Jangan menyimpan token, kredensial, file sesi WhatsApp, atau database ke GitHub.
7. File database dan kredensial masuk `.gitignore`.
8. Buat backup database secara berkala dan uji pemulihannya.
9. Batasi siapa yang boleh memakai bot, terutama pada versi awal keluarga.
10. Jangan mengirim isi transaksi pribadi ke layanan AI tanpa memahami kebijakan privasi dan meminimalkan data yang dikirim.

## 9. Tahapan pengerjaan dan kriteria selesai

| Fase | Pekerjaan | Kriteria selesai |
|---|---|---|
| 1 | Environment laptop | Node.js, npm, Git, dan VS Code berjalan |
| 2 | Struktur proyek | `npm start` berjalan dan Git aktif |
| 3 | Database SQLite | Tabel, relasi, dan tes baca/tulis berhasil |
| 4 | Setup Redmi | Termux, Node.js, Git, dan proyek siap |
| 5 | Integrasi WhatsApp | Bot dapat menerima dan membalas pesan uji |
| 6 | Logika transaksi | Income, expense, saldo, riwayat, dan laporan berfungsi |
| 7 | Keamanan | Isolasi data pengguna dan validasi diuji |
| 8 | AI parser | Pesan natural dapat diubah menjadi data terstruktur dan dikonfirmasi |
| 9 | Deployment | Bot berjalan stabil, restart, backup, dan pemulihan diuji |

### Status berdasarkan progres sampai 31 Oktober 2026

- [x] Fase 1 — Environment laptop.
- [x] Fase 2 — Struktur dasar proyek dan script npm.
- [ ] Konfirmasi push repository GitHub.
- [ ] Fase 3 — Database SQLite.
- [ ] Fase 4 — Setup Redmi Note 5 Pro.
- [ ] Fase 5 — Integrasi WhatsApp.
- [ ] Fase 6 — Logika transaksi.
- [ ] Fase 7 — Pengujian keamanan.
- [ ] Fase 8 — Integrasi AI.
- [ ] Fase 9 — Deployment stabil.

## 10. Rencana setup Redmi Note 5 Pro

Termux sudah terpasang menurut pengguna. Langkah berikutnya:

1. Periksa versi Android dan arsitektur perangkat.
2. Perbarui package Termux.
3. Pasang kebutuhan dasar seperti Git dan Node.js yang kompatibel.
4. Pastikan versi Node.js mendukung fitur yang dipakai proyek.
5. Ambil kode dari GitHub setelah repository dipastikan benar.
6. Pasang dependency dari `package.json`.
7. Jalankan tes database dan tes aplikasi.
8. Baru hubungkan akun WhatsApp bot.

Jangan memasang atau menautkan akun WhatsApp sebelum koneksi dan penyimpanan sesi dipahami. Simpan sesi secara privat dan jangan unggah ke repository.

## 11. Risiko operasional

- Android dapat menghentikan proses latar belakang karena optimasi baterai.
- HP dapat mati, panas, kehilangan koneksi internet, atau kehabisan ruang.
- SQLite cocok untuk aplikasi lokal skala kecil, tetapi perlu backup dan strategi migrasi.
- Penyimpanan sesi WhatsApp harus dilindungi.
- Baileys dapat berubah mengikuti perubahan WhatsApp.
- Bot yang berjalan di satu HP bukan jaminan ketersediaan 24/7.

Untuk penggunaan awal, sediakan backup database dan prosedur menyalakan ulang bot secara manual. Evaluasi server/VPS atau platform resmi jika kebutuhan meningkat.

## 12. Definisi selesai proyek versi pertama

Proyek MVP dianggap berhasil jika:

- Pengguna yang diizinkan dapat mengirim pesan transaksi.
- Bot mengidentifikasi jenis transaksi dan nominal dengan benar.
- Pengguna dapat mengonfirmasi atau membatalkan transaksi.
- Data tersimpan dan tetap ada setelah bot dimulai ulang.
- Saldo, riwayat, dan ringkasan menghasilkan perhitungan yang benar.
- Pengguna tidak dapat membaca atau mengubah transaksi pengguna lain.
- Database dapat dicadangkan dan dipulihkan.

---

**Prinsip pengembangan:** bangun fondasi sederhana, uji setiap fase, baru tambahkan kompleksitas. Jangan menambahkan AI sebelum pencatatan transaksi dasar bekerja dengan benar.
