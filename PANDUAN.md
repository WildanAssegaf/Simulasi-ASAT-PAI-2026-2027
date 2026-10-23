# Panduan Pemasangan — Simulasi ASAT PAI Kelas VIII (2026/2027)

Aplikasi terdiri dari dua file:

- `index.html` — aplikasi siswa (HTML, CSS, JavaScript; satu file, tanpa library).
- `Code.gs` — backend Google Apps Script yang menulis data ke Google Spreadsheet.

## 1. Struktur Spreadsheet

Jalankan `setup()` sekali (langkah 2) dan lima sheet berikut dibuat otomatis lengkap dengan header.

| Sheet | Isi | Aturan anti-ganda |
|---|---|---|
| `Siswa` | ID_Siswa (contoh `VIIIA-07`), Nama, Absen, Kelas, Pertama_Tercatat, Terakhir_Aktif | Satu baris per siswa. Kunci: kelas + nomor absen. |
| `Latihan` | Key, ID_Siswa, Nama, Absen, Kelas, Bab, Nama_Bab, Percobaan, Nilai_Terakhir, Nilai_Tertinggi, Benar_Terakhir, Jumlah_Soal, Pertama_Tercatat, Terakhir_Dikerjakan, Submission_ID_Terakhir | Satu baris per siswa per bab. Latihan ulang memperbarui baris (Percobaan bertambah, Nilai_Tertinggi dijaga). |
| `ASAT` | ID_Siswa, Nama, Absen, Kelas, Waktu_Selesai, Nilai_Akhir, Skor_PG, Skor_PGK, Skor_SebabAkibat, Skor_Menjodohkan, Skor_Isian, Skor_Bab1–Bab5 (maks. 20 per bab), Durasi_Menit, Cara_Selesai, Submission_ID | Satu baris per siswa. Percobaan kedua ditolak dan nilai pertama tetap. |
| `Detail_Jawaban` | Waktu, Submission_ID, Jenis (Latihan/ASAT), ID_Siswa, Nama, Kelas, Absen, Bab, No, ID_Soal, Tipe, Soal, Jawaban_Siswa, Kunci, Status (Benar/Sebagian/Salah/Kosong), Poin | Ditulis hanya jika Submission_ID belum pernah tercatat. |
| `Log_Kirim` | Submission_ID, Waktu, Jenis, ID_Siswa, Hasil | Daftar semua ID kiriman. Dipakai untuk menolak kiriman ulang. |

Penilaian: soal berbasis HOTS dengan lima jenis. Nilai maksimal ASAT 100, latihan per bab 100.

| Jenis soal | ASAT | Bobot ASAT | Latihan per bab (10 poin/soal) |
|---|---|---|---|
| Pilihan ganda (kasus, analisis, evaluasi) | 20 | 2 | 5 |
| Pilihan ganda kompleks (pilih semua yang benar) | 5 | 3 | 2 |
| Sebab–akibat (pernyataan dan alasan, 5 opsi) | 5 | 3 | 1 |
| Menjodohkan | 5 | 4 | 1 |
| Isian singkat berbasis argumen | 5 | 2 | 1 |

Nilai sebagian: pilihan ganda kompleks = (pilihan benar − pilihan salah) ÷ jumlah kunci, minimum 0. Menjodohkan = pasangan benar ÷ jumlah pasangan. Isian = kelompok kata kunci yang cocok ÷ jumlah kelompok.

**Penting saat memperbarui dari versi lama:** hapus sheet `ASAT` lama (atau isinya) sebelum memakai backend baru, karena kolomnya berubah. Data tersimpan di perangkat siswa memakai awalan baru (`pai2_`), sehingga ASAT dan latihan versi lama tidak terbawa.

## 2. Pasang backend (Google Apps Script)

1. Buka Google Drive, buat Google Spreadsheet baru, beri nama misalnya `ASAT PAI VIII 2026-2027`.
2. Klik **Ekstensi → Apps Script**.
3. Hapus isi `Code.gs` bawaan, lalu tempel seluruh isi file `Code.gs` dari paket ini. Simpan.
4. Pilih fungsi `setup` pada menu di bagian atas, lalu klik **Jalankan**. Izinkan akses saat diminta (pilih akun, **Lanjutan → Buka … (tidak aman) → Izinkan**). Hal ini normal untuk skrip buatan sendiri.
5. Klik **Deploy → Deployment baru → ikon roda gigi → Aplikasi web**:
   - Deskripsi: `ASAT PAI v1`
   - Jalankan sebagai: **Saya**
   - Yang memiliki akses: **Siapa saja**
6. Klik **Deploy**, lalu salin **URL aplikasi web** (berakhiran `/exec`).

Jika skrip dibuat terpisah dari Spreadsheet, isi konstanta `SPREADSHEET_ID` di bagian atas `Code.gs` dengan ID Spreadsheet (bagian URL antara `/d/` dan `/edit`).

Uji cepat: tempel URL `/exec` di browser. Hasil yang benar: `{"ok":true,"status":"aktif",...}`.

## 3. Hubungkan aplikasi

1. Buka `index.html` dengan editor teks.
2. Cari baris berikut di dalam `CONFIG`:
   ```js
   API_URL: "PASTE_URL_WEB_APP_DI_SINI",
   ```
3. Ganti dengan URL `/exec` tadi. Simpan.

Jika `API_URL` belum diisi, aplikasi tetap berjalan dalam **mode lokal** (nilai hanya disimpan di perangkat siswa) dan menampilkan peringatan kuning di beranda.

## 4. Bagikan ke siswa

Pilih salah satu cara berikut.

- **GitHub Pages (disarankan, gratis)**: buat repositori, unggah `index.html`, aktifkan Pages. Bagikan tautannya lewat grup kelas. Tautan dapat dibuka di HP Android maupun komputer.
- **Netlify Drop**: seret `index.html` ke https://app.netlify.com/drop dan bagikan tautannya.
- **Server sekolah / hosting**: unggah `index.html` ke folder web.
- Membuka file langsung dari penyimpanan HP juga bisa, tetapi pengiriman data lebih andal bila halaman dibuka lewat `https://`.

## 5. Cara kerja anti-ganda

1. Setiap hasil membawa `submissionId` unik. Server menolak ID yang sama (klik ganda, sinyal putus lalu kirim ulang, dan sebagainya).
2. Hasil masuk ke antrean di perangkat siswa dan dikirim ulang otomatis sampai server membalas sukses. Hasil tidak hilang saat sinyal buruk.
3. ASAT dikunci per siswa (kelas + nomor absen). Percobaan kedua, termasuk dari perangkat lain, ditolak dengan nilai pertama dipertahankan.
4. `LockService` mencegah dua permintaan menulis ke sheet bersamaan.
5. Nilai dihitung ulang di server dari status jawaban, bukan dari angka kiriman klien.

## 6. Tugas guru

- **Mengulang ASAT untuk satu siswa**: di Spreadsheet buka menu **ASAT PAI → Izinkan siswa mengulang ASAT**, isi kelas dan absen. Baris ASAT dihapus, riwayat di `Detail_Jawaban` tetap. Jika ASAT lama tersimpan di perangkat siswa, minta siswa memakai perangkat lain atau menghapus data situs browser.
- **Rekap nilai**: lihat sheet `ASAT` dan `Latihan`. Gunakan **Data → Buat filter** atau tabel pivot per kelas.
- **Mengubah soal**: edit array `BAB` pada `index.html`. Soal ditulis dengan lima fungsi di atas `const BAB`:
  - `PG(q, benar, [3 pengecoh], pembahasan)`
  - `PGK(q, [opsi], [indeks yang benar], pembahasan)` (indeks mulai dari 0)
  - `SA(pernyataan, alasan, kode 0–4, pembahasan)`: 0 = benar-benar-berhubungan, 1 = benar-benar-tidak berhubungan, 2 = benar-salah, 3 = salah-benar, 4 = salah-salah
  - `JD(q, [kiri], [kanan], [indeks kanan tiap butir kiri], pembahasan)`; kanan boleh lebih banyak sebagai pengecoh
  - `IS(q, [[variasi kata kunci kelompok 1], [kelompok 2]], contohJawaban, pembahasan)`; kata kunci ditulis huruf kecil tanpa tanda baca

  Urutan opsi diacak otomatis. Komposisi per bab harus tetap: latihan 5 PG + 2 PGK + 1 SA + 1 JD + 1 IS, ASAT 4 PG + 1 PGK + 1 SA + 1 JD + 1 IS, agar server menerimanya.
- **Mengubah durasi**: ubah `ASAT_MENIT` di `CONFIG`. Batas durasi di `Code.gs` (angka 90 pada `simpanAsat_`) perlu disesuaikan juga.
- **Setelah mengubah `Code.gs`**: pilih **Deploy → Kelola deployment → Edit → Versi baru → Deploy**. URL tetap sama.

## 7. Batasan yang perlu diketahui

- Kunci jawaban berada di `index.html`, sehingga siswa yang mahir dapat melihatnya lewat alat pengembang browser. Aplikasi ini cocok untuk **simulasi dan latihan**, bukan ujian bernilai tinggi yang diawasi ketat.
- Identitas tidak memakai kata sandi. Siswa yang salah mengetik nomor absen akan tercatat sebagai nomor tersebut.
- Isian singkat berbasis argumen dinilai otomatis dengan mencocokkan kata kunci pada jawaban siswa (huruf besar/kecil dan tanda baca diabaikan). Jawaban yang benar tetapi memakai kata lain, atau salah ketik, dapat dinilai kurang. Guru dapat memeriksa kolom `Jawaban_Siswa` pada `Detail_Jawaban` bila perlu penilaian manual.
- Kuota Apps Script gratis cukup untuk satu angkatan (sekitar 200 siswa), tetapi hindari seluruh kelas menekan "Kumpulkan" pada detik yang sama bila jaringan lemah. Antrean otomatis akan mencoba lagi.

## 8. Cakupan materi dan soal

| Bab | Latihan (5 PG, 2 PGK, 1 SA, 1 JD, 1 IS) | ASAT (4 PG, 1 PGK, 1 SA, 1 JD, 1 IS) |
|---|---|---|
| 1. Melestarikan Alam dan Menjaga Kehidupan | 10 | 8 |
| 2. Kitab-Kitab Allah dan Toleransi | 10 | 8 |
| 3. Jujur, Amanah, Integritas, dan Munafik | 10 | 8 |
| 4. Salat Gerhana, Istiska, dan Jenazah | 10 | 8 |
| 5. Kejayaan Ilmu Daulah Abbasiyah | 10 | 8 |
| **Total** | **50** | **40** |

Bank soal disusun dari materi PAI dan Budi Pekerti Kelas VIII (Kurikulum Merdeka). Sebelum dipakai, mohon guru PAI meninjau ulang soal dan pembahasan agar sesuai dengan buku pegangan dan mazhab yang diajarkan di sekolah (misalnya detail tata cara salat gerhana dan jumlah lapis kafan).

## 9. EXP, Level, dan Leaderboard Realtime (Supabase)

Setiap soal yang dijawab benar memberi EXP. EXP dijumlahkan dan tampil di papan peringkat dengan podium juara 1–3. Data disimpan di Supabase (Postgres) dan diperbarui langsung (realtime) ke semua siswa yang sedang membuka halaman peringkat. Google Spreadsheet tetap dipakai untuk rekap nilai.

**EXP per soal (maksimum):** pilihan ganda 10, pilihan ganda kompleks 15, sebab–akibat 15, menjodohkan 20, isian argumen 15. Soal yang dijawab sebagian memberi EXP sebanding skornya. EXP tiap soal diberikan **sekali**: mengulang latihan tidak menambah EXP, hanya bisa menaikkan bila skor soal itu lebih tinggi dari sebelumnya. Total maksimum seluruh soal: **1.175 EXP** (latihan 650, ASAT 525).

**Level:** 1 Pemula (0), 2 Pencari Ilmu (100), 3 Penjaga Amanah (250), 4 Cendekia (450), 5 Ahli Hikmah (700), 6 Bintang Hikmah (1.000).

### Pasang
1. Project Supabase `asat-pai-viii` sudah dibuat (ref `empsnqxmtkxarazfnkbb`, wilayah Singapura).
2. Buka **SQL Editor**, tempel seluruh isi `supabase.sql`, lalu **Run**. Aman dijalankan berulang. Ini membuat tabel, keamanan (RLS), fungsi `catat_exp`, dan menyalakan realtime untuk tabel `siswa_lb`.
3. `SUPABASE_URL` dan `SUPABASE_KEY` di `CONFIG` pada `index.html` sudah terisi. `SUPABASE_KEY` adalah kunci publik (anon) yang memang aman berada di halaman web. Jangan pernah menaruh kunci `service_role` di `index.html`.
4. Uji: kerjakan satu latihan, lalu buka **🏆 Peringkat** dari beranda.

### Keamanan
- Siswa tidak bisa menulis langsung ke tabel. Satu-satunya jalan menulis adalah fungsi `catat_exp`, yang memeriksa kelas, nomor absen, nama, daftar soal, dan membatasi skor 0–1. EXP hanya naik dan tidak pernah turun.
- Karena kunci jawaban ada di `index.html`, siswa mahir tetap dapat memalsukan skor. Dampaknya dibatasi: EXP maksimum satu siswa tetap 1.175. Cocok untuk motivasi dan latihan, bukan penilaian resmi. Nilai resmi tetap dari Spreadsheet.
- Papan peringkat memperlihatkan **nama** dan kelas siswa kepada siapa pun yang memegang tautan aplikasi. Gunakan nama panggilan bila perlu dan sampaikan ke siswa.

### Tugas guru (SQL Editor)
- Reset semua EXP: `truncate public.exp_soal; update public.siswa_lb set total_exp = 0, soal_benar = 0;`
- Hapus satu siswa: `delete from public.siswa_lb where id = 'VIIIA-07';`
- Lihat peringkat: `select nama, kelas, total_exp from public.siswa_lb order by total_exp desc, updated_at asc;`
- Ubah EXP per jenis soal: ubah kolom `exp_max` pada tabel `soal_meta` **dan** konstanta `EXPMAX` di `index.html` agar tampilan sama.

### Batasan paket gratis Supabase
Project gratis dijeda otomatis bila tidak dipakai sekitar satu minggu (bisa diaktifkan kembali dari dashboard) dan memiliki batas sambungan realtime bersamaan. Aplikasi hanya membuka sambungan realtime saat halaman peringkat dibuka, dan otomatis memakai penyegaran tiap 15 detik bila realtime gagal.

## 10. Soal bergambar dan ayat Al-Qur'an

- **Ayat:** ditampilkan dalam kotak khusus dengan teks Arab, nama surah/ayat, dan terjemah yang dapat dibuka. Data ada di objek `AY` dan dipasang ke soal lewat `AYAT['L1-01'] = [AY.rum41]` (18 soal memakai ayat). **Mohon guru memeriksa teks Arab dan terjemah dengan mushaf pegangan sekolah** sebelum dipakai.
- **Gambar:** dibuat dari kode (SVG), tanpa file gambar. Ada diagram batang (`chartBar`), diagram gerhana (`svgGerhana`), garis waktu Abbasiyah (`svgTimeline`), dan ilustrasi banjir (`svgBanjir`). Dipasang lewat `GAMBAR['L1-03'] = ...` (7 soal bergambar).
- **Font Arab:** aplikasi memuat font Amiri dari Google Fonts bila ada internet, dan memakai font Arab bawaan perangkat bila tidak.

## 11. Tema terang dan gelap

- Tombol ☀️ / 🌙 ada di bagian atas semua halaman. Satu ketukan mengganti tema.
- Pertama kali dibuka, tema mengikuti pengaturan perangkat siswa (mode gelap atau terang). Setelah siswa memilih sendiri, pilihan itu diingat di perangkatnya.
- Seluruh warna memakai variabel CSS di bagian `:root` (terang) dan `:root[data-theme="gelap"]` (gelap) pada `index.html`. Untuk mengubah warna tema, cukup ubah variabel di kedua blok itu.
- Kontras teks sudah diperiksa otomatis pada 14 tampilan di kedua tema (login, beranda, materi, soal, pembahasan, hasil, peringkat, intro ASAT, dan jendela dialog).
