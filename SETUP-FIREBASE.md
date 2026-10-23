# Setup Firebase untuk PAIlab (proyek lms-pai-kelas-9)

1. Firebase Console > Build > Authentication > Get started > tab Sign-in method > aktifkan **Anonymous**.
2. Build > Realtime Database > Create database. Pilih lokasi (mis. Singapore), mulai dalam mode terkunci.
3. Tab Rules di Realtime Database: tempel isi `database.rules.json`, klik Publish.
4. Project settings (ikon roda) > Your apps > tambah app Web (</>). Salin `apiKey`, `appId`, dan `databaseURL` (ada di halaman Realtime Database, bagian atas data).
5. Di `index.html`, cari `const FB={cfg:{...}}` lalu isi `apiKey`, `databaseURL`, `appId`. Ganti juga `pin:"1234"` dengan PIN guru milikmu.
6. Upload `index.html` ke hosting (Firebase Hosting, GitHub Pages, dll). Tambahkan domain hosting di Authentication > Settings > Authorized domains.

Catatan keamanan: PIN guru hanya menyembunyikan tampilan panel di browser. Data ringkasan siswa (nama, kelas, XP, skor) bisa dibaca setiap pengguna aplikasi yang login anonim, karena dipakai untuk peringkat. Jangan simpan data sensitif di sini.
