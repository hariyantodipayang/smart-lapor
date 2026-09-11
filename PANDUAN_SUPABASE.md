# Panduan Menghubungkan Smart Lapor ke Supabase

Panduan ini ditulis untuk dikerjakan sekali duduk, kurang lebih 20 menit.
Tidak perlu paham database. Cukup ikuti urutannya.

---

## Dulu, pahami dulu apa yang sedang kita lakukan

Aplikasi Smart Lapor punya dua "otak":

- **Tampilan** — file `index.html`, `app.js`, `db.js`. Ini yang dilihat warga.
  Nanti dititipkan gratis di GitHub Pages.
- **Gudang data** — tempat laporan warga, foto, dan status penanganan disimpan.
  Ini yang belum ada. Supabase-lah yang akan jadi gudang itu.

Selama gudangnya belum ada, aplikasi berjalan dalam **Mode Demo**: data hanya
menempel di browser orang yang membukanya, dan hilang kalau cache dibersihkan.
Bagus untuk presentasi, tidak bisa dipakai melayani warga sungguhan.

Tugas kita hari ini: bikin gudangnya, lalu kasih tahu aplikasi di mana alamat
gudang itu.

> **Catatan penting.** Di `config.js` tadi ada alamat Supabase
> `zuoxkzvenesetorghsqf.supabase.co` beserta kuncinya. Itu bukan milik Anda —
> kemungkinan sisa contoh. Alamat itu sudah saya kosongkan, karena alamat gudang
> yang salah lebih merepotkan daripada tidak punya gudang sama sekali: aplikasi
> akan terus mencoba menghubungi pintu yang tidak pernah ada.

---

## Langkah 1 — Daftar Supabase

1. Buka **https://supabase.com**, klik **Start your project**.
2. Masuk pakai akun GitHub Anda (paling praktis, karena repo-nya juga di GitHub).

Gratis. Paket gratis Supabase sudah lebih dari cukup untuk skala satu kabupaten:
500 MB database dan 1 GB penyimpanan foto.

---

## Langkah 2 — Buat project

Klik **New project**, lalu isi:

| Kolom | Isi |
|---|---|
| Name | `smart-lapor` |
| Database Password | buat sandi kuat, **lalu catat di tempat aman** |
| Region | **Southeast Asia (Singapore)** |

Soal **Region**: pilih Singapura, jangan yang lain. Ini menentukan di kota mana
server fisiknya berdiri. Semakin dekat dengan Kepahiang, semakin cepat aplikasi
terasa di HP warga. Amerika bisa saja, tapi setiap klik akan terasa berat.

Soal **Database Password**: sandi ini berbeda dari sandi login Supabase Anda.
Anda mungkin tidak akan pernah memakainya, tapi kalau suatu saat butuh, tidak
ada cara memulihkannya. Catat.

Setelah klik **Create new project**, tunggu 1–2 menit sampai selesai menyala.

---

## Langkah 3 — Jalankan skema database

Ini langkah yang paling penting. Di sinilah meja, rak, dan laci di dalam gudang
dibangun.

1. Di menu kiri, klik **SQL Editor**.
2. Klik **New query**.
3. Buka file **`supabase-schema.sql`** yang ada di folder ini, **blok seluruh
   isinya** (Ctrl+A), salin, lalu tempel ke kotak SQL Editor.
4. Klik **Run** (atau Ctrl+Enter).

Tunggu sampai muncul tulisan *Success*. Kalau ada baris berwarna merah, salin
pesan errornya dan tanyakan ke saya.

Sekali jalan, file itu membuat semuanya sekaligus:

- **4 tabel** — `kategori`, `laporan`, `tindak_lanjut`, dan `profiles`.
- **7 kategori bawaan** — inilah isi dropdown "Jenis Permasalahan" di formulir.
  Selama langkah ini belum dijalankan, dropdown itu kosong.
- **Aturan keamanan (RLS)** — warga boleh mengirim laporan dan mengecek status,
  tapi tidak bisa mengubah status laporan orang lain. Hanya petugas yang login
  yang bisa.
- **Bucket `foto`** — lemari khusus penyimpan foto laporan.
- **Trigger otomatis** — setiap akun baru yang Anda buat langsung punya profil
  petugas, tanpa perlu menyalin UUID manual.

---

## Langkah 4 — Ambil alamat & kunci gudang

1. Klik ikon gerigi **Project Settings** (pojok kiri bawah) → **API**.
2. Salin dua hal:
   - **Project URL** — bentuknya `https://xxxxxxxxxxxx.supabase.co`
   - **anon public** key — teks panjang yang diawali `eyJ...`

Buka file **`config.js`** di folder ini, tempelkan ke dua baris ini:

```js
SUPABASE_URL: "https://xxxxxxxxxxxx.supabase.co",
SUPABASE_ANON_KEY: "eyJhbGciOi...",
```

Pastikan tanda kutip dan koma di ujung tidak ikut terhapus.

> **Aman tidak, kunci ini dipajang di repo publik?** Aman. `anon key` memang
> dirancang untuk diekspos — dia hanya semacam nomor pintu. Yang menjaga isinya
> adalah aturan RLS yang sudah dipasang di Langkah 3.
>
> Yang **tidak boleh** dipajang adalah `service_role` key, yang letaknya
> bersebelahan di halaman yang sama. Kunci itu membuka semua pintu dan
> mengabaikan semua aturan. Jangan pernah menempelkannya ke `config.js`.

---

## Langkah 5 — Buat akun petugas

1. Menu kiri → **Authentication** → **Users** → **Add user** → *Create new user*.
2. Isi Email dan Password.
3. **Centang "Auto Confirm User"** — kalau tidak dicentang, akun itu menunggu
   verifikasi email yang tidak akan pernah datang, dan login akan gagal terus.
4. Klik **Create user**.

Akun baru otomatis berperan `petugas`. Untuk menaikkannya jadi admin, kembali ke
**SQL Editor**, tempel dan Run (ganti emailnya):

```sql
update public.profiles set peran = 'admin', nama = 'Admin DLH'
where id = (select id from auth.users where email = 'admin@dlhkepahiang.go.id');
```

Tiga peran yang tersedia:

- `admin` — bisa semuanya, termasuk mengelola kategori
- `petugas` — memperbarui status laporan di lapangan
- `pimpinan` — melihat dashboard monitoring

---

## Langkah 6 — Uji coba

Buka `index.html` (klik dua kali). Perhatikan badge di kanan atas:

- Tertulis **TERHUBUNG SUPABASE** → berhasil.
- Masih **MODE DEMO** → `config.js` belum tersimpan, atau ada kutip/koma yang
  hilang saat menempel.

Lalu uji tiga hal ini berurutan:

1. Buka formulir laporan — dropdown "Jenis Permasalahan" harus terisi 7 pilihan.
2. Kirim satu laporan percobaan lengkap dengan foto. Catat nomor tiketnya.
3. Masuk menu **Petugas / Admin**, login dengan akun Langkah 5, lalu ubah status
   laporan tadi. Cek lewat menu "Cek Status" — riwayatnya harus muncul.

Kalau ketiganya lancar, gudang Anda sudah berdiri.

---

## Langkah 7 — Unggah ulang ke GitHub

`config.js` sudah berubah, jadi repo di GitHub perlu diperbarui. Unggah ulang
`config.js` **dan** `supabase-schema.sql` (file ini baru saya tambahkan ke
folder — sebelumnya tidak ikut terunggah).

GitHub Pages akan menayangkan ulang otomatis dalam 1–2 menit.

---

## Kalau ada yang tidak beres

| Gejala | Penyebab biasanya |
|---|---|
| Dropdown kategori kosong | Skema di Langkah 3 belum dijalankan |
| Badge tetap "MODE DEMO" | `config.js` masih kosong, atau sintaksnya rusak (kutip/koma hilang) |
| Login ditolak terus | "Auto Confirm User" lupa dicentang di Langkah 5 |
| Login berhasil tapi menu petugas kosong | Baris `update ... set peran` di Langkah 5 belum dijalankan |
| Foto gagal diunggah | Bucket `foto` tidak terbentuk — jalankan ulang bagian 4 pada `supabase-schema.sql` |
| Laporan tersimpan tapi hilang di dashboard | Kemungkinan aplikasi masih menunjuk project lama; kosongkan cache browser (Ctrl+Shift+R) |

Buka **Logs** di menu kiri Supabase kalau ingin melihat apa yang sebenarnya
terjadi di balik layar. Semua permintaan tercatat di sana, lengkap dengan pesan
penolakannya.

---

*Dinas Lingkungan Hidup Kabupaten Kepahiang — Smart Lapor*
