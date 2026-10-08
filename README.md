# Sistem Administrasi Surat Organisasi — GitHub Pages + Supabase

Versi ini dibuat untuk kebutuhan hosting murah/gratis: **frontend berjalan di GitHub Pages**, sedangkan **Supabase** menangani login, database PostgreSQL, dan penyimpanan file. GitHub Pages hanya menyajikan file statis; aplikasi memanggil Supabase dari browser.

## File penting

- `index.html` — aplikasi utama
- `config.js` — isi URL dan publishable/anon key Supabase
- `config.example.js` — contoh konfigurasi
- `supabase_schema.sql` — database, trigger, function nomor surat, RLS, dan policy Storage
- `.github/workflows/pages.yml` — deploy otomatis ke GitHub Pages
- `.nojekyll` — penanda untuk hosting statis

## 1. Buat project Supabase

Buat project baru di Supabase. Dari **Project Settings / API**, salin **Project URL** dan **Publishable key/anon key**. Jangan pernah memasukkan `service_role` key ke `index.html`; key rahasia hanya boleh dipakai di server. Supabase mendokumentasikan `createClient(url, publishable_key)` untuk penggunaan browser.

Buka **SQL Editor**, lalu jalankan seluruh isi `supabase_schema.sql`.

## 2. Buat Storage bucket

Buka **Storage → New bucket**, lalu buat bucket:

`surat-files`

Atur sebagai **Private**.

Policy Storage sudah tersedia di `supabase_schema.sql`.

## 3. Buat akun pertama

Buka **Authentication → Users → Add user**. Buat email dan password untuk akun pertama.

Setelah akun dibuat, trigger database otomatis membuat baris `profiles` dengan role `Pengurus`.

Jadikan akun pertama Admin dengan SQL:

```sql
update public.profiles
set role = 'Admin', jabatan = 'Administrator'
where email = 'email-anda@contoh.com';
```

Tambahkan pengguna lain dari menu **Pengguna** setelah login sebagai Admin. Jika Email Confirmation diaktifkan, pengguna baru perlu mengonfirmasi email sebelum login.

## 4. Isi `config.js`

Ganti:

```js
window.SUPABASE_CONFIG = {
  url: 'https://PROJECT-ID.supabase.co',
  key: 'SUPABASE_PUBLISHABLE_OR_ANON_KEY'
};
```

dengan nilai project Anda.

**Catatan keamanan:** publishable/anon key memang digunakan di frontend, tetapi keamanan data bergantung pada RLS. Jangan memakai service role key di browser.

## 5. Upload ke GitHub

Buat repository, misalnya:

`surat-organisasi`

Upload isi folder ini ke branch `main`.

Di GitHub pilih:

**Settings → Pages → Build and deployment → Source → GitHub Actions**

Workflow `pages.yml` akan otomatis deploy setelah Anda push ke `main`.

URL umumnya:

`https://USERNAME.github.io/NAMA-REPOSITORY/`

## 6. Redirect password / URL aplikasi

Pada Supabase, atur **Authentication → URL Configuration** sehingga Site URL berisi URL GitHub Pages Anda, contoh:

`https://USERNAME.github.io/surat-organisasi/`

Tambahkan URL tersebut ke Redirect URLs bila diperlukan untuk reset password.

## Alur aplikasi

### Surat masuk

Diterima → Dicatat → Diverifikasi → Disposisi/Diteruskan → Diproses → Selesai → Arsip

### Surat keluar

Permintaan → Draft → Pemeriksaan → Persetujuan → Penomoran → Tanda Tangan → Dikirim → Arsip

Pada detail surat keluar tersedia tindakan workflow kontekstual seperti **Kirim ke Pemeriksaan**, **Minta Revisi**, **Ajukan Persetujuan**, **Setujui**, **Minta Revisi**, **Beri Nomor**, **Sudah Ditandatangani**, **Siap Dikirim**, **Tandai Sudah Dikirim**, dan **Arsipkan** sesuai status/role.

## Batasan yang perlu diketahui

- GitHub Pages tidak menjalankan Python/SQLite. Karena itu versi ini tidak membutuhkan `server.py`.
- Laporan “Export Excel/CSV” memakai CSV yang dapat dibuka di Excel/LibreOffice.
- Cetak laporan memakai dialog print browser sehingga dapat disimpan sebagai PDF.
- TTE kriptografis belum diintegrasikan; aplikasi hanya menyimpan file final dan metadata tanda tangan.
- Backup database dilakukan dari fasilitas Supabase atau mekanisme yang Anda pilih, bukan dari GitHub Pages.
