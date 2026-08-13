# HDIR - Aplikasi Absensi Online

Aplikasi absensi berbasis QR/NIK + selfie + validasi lokasi GPS.

**Arsitektur:**
- **Frontend** (`index.html`) → di-host statis di GitHub Pages (atau host statis lain). Ini yang membuat kamera & GPS bekerja normal — halaman berjalan di domain HTTPS milik kamu sendiri, bukan di dalam iframe Google yang membatasi izin kamera.
- **Backend** (`Code.gs`) → dijalankan di Google Apps Script sebagai Web App (JSON API murni). Database di Google Sheets, foto selfie di Google Drive.

---

## 1. Deploy Backend (Google Apps Script)

1. Buka [script.google.com](https://script.google.com) → **New Project**.
2. Hapus isi `Code.gs` default, tempel isi file `Code.gs` dari sini.
3. Jalankan fungsi `setupDatabase` sekali (pilih dari dropdown fungsi di toolbar editor → klik ▶ Run). Ini otomatis membuat:
   - Spreadsheet **`HDIR_Database`** dengan sheet `Karyawan`, `Absensi`, `Settings`.
   - Folder Drive **`HDIR_Photos`** untuk foto selfie.
   - Izinkan semua permission yang diminta saat pertama kali run.
4. **Deploy → New deployment**:
   - Select type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Klik **Deploy**
5. Salin URL yang diakhiri `/exec` — ini **WEB_APP_URL** kamu. Contoh:
   ```
   https://script.google.com/macros/s/AKfycbxxxxxxxxxxxxxxxxx/exec
   ```

> ⚠️ Setiap kali kamu mengedit `Code.gs` setelah ini, kamu **wajib** membuat versi deployment baru: **Deploy → Manage deployments → ikon pensil (Edit) → Version: New version → Deploy**. Menyimpan kode saja tidak otomatis memperbarui URL `/exec` yang sudah live.

---

## 2. Isi URL Backend ke Frontend

Buka `index.html`, cari baris ini di bagian atas tag `<script>` (dekat komentar `WAJIB DIISI`):

```js
const WEB_APP_URL = '';
```

Ganti jadi:

```js
const WEB_APP_URL = 'https://script.google.com/macros/s/AKfycbxxxxxxxxxxxxxxxxx/exec';
```

Simpan file. Kalau lupa mengisi ini, halaman akan menampilkan banner peringatan kuning otomatis saat dibuka.

---

## 3. Publish ke GitHub Pages

1. Buat repository baru di GitHub (bisa public atau private+Pages jika akun Pro).
2. Upload file `index.html` ke root repository (nama file **harus** persis `index.html` huruf kecil).
3. Buka **Settings → Pages** di repo tersebut.
4. Source: **Deploy from a branch** → Branch: `main` → Folder: `/ (root)` → **Save**.
5. Tunggu 1-2 menit, GitHub akan memberi URL seperti:
   ```
   https://username.github.io/nama-repo/
   ```
6. Buka URL tersebut — aplikasi HDIR sudah online dan bisa diakses siapa saja.

---

## 4. Uji Coba

- Login Admin default: password `admin123` (segera ganti lewat tab "Ubah Password").
- Atur koordinat & radius kantor di tab "Pengaturan Kantor" sebelum karyawan mulai absen.
- Tambahkan minimal satu karyawan lewat tab "Data Karyawan" untuk mulai testing absensi.
- Coba alur "Presensi / Absensi" dari halaman utama — izinkan akses kamera & lokasi saat diminta browser.

---

## Catatan Teknis

- Kamera & QR scanner punya mode fallback otomatis (buka kamera native HP / unggah foto) kalau `getUserMedia` gagal — berguna untuk browser lama atau kasus izin ditolak.
- Foto selfie dikompres otomatis (maks 720px, kualitas 72%) sebelum diupload, supaya proses absensi lebih cepat.
- Validasi radius lokasi dilakukan di **server**, bukan hanya di browser, supaya tidak bisa dimanipulasi lewat DevTools.
- Pengaturan kantor di-cache 60 detik di server (`CacheService`) agar pengecekan lokasi lebih cepat; cache otomatis dibersihkan begitu admin menyimpan perubahan pengaturan.
