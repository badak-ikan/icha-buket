# BuketKu — Website Toko Buket

Website ini awalnya satu file `index.html`, sekarang dipisah menjadi:

```
buketku/
├── frontend/              -> Tampilan website statis (HTML, CSS, JS)
├── backend/                -> API Node.js + Express, untuk hosting Railway/VPS
├── netlify/functions/      -> API versi serverless, khusus untuk hosting Netlify
├── netlify.toml             -> Konfigurasi Netlify (redirect /api -> function)
└── package.json              -> Dependency untuk Netlify Functions
```

Pakai **salah satu** backend saja sesuai tempat hosting kamu:
- Mau hosting di **Netlify** -> pakai `netlify/functions/` (lihat bagian "Deploy ke Netlify" di bawah).
- Mau hosting di **Railway/Render/VPS** -> pakai `backend/` (lihat `backend/README.md`).

## Apa yang berubah dari versi satu-file

- **Data produk** dulunya tertulis langsung di HTML, sekarang disimpan di file `products.json` dan diambil frontend lewat `GET /api/products`.
- **Pesanan** dulunya hanya membuka WhatsApp dari browser, sekarang dikirim ke `POST /api/orders` di backend, baru kemudian membuka WhatsApp.
- **Login admin** dulunya kredensialnya tertulis langsung di JavaScript frontend — siapa pun bisa melihatnya lewat "View Page Source". Sekarang kredensial disimpan di environment variable backend dan diverifikasi lewat `POST /api/admin/login`.
- **Live chat** pelanggan tersimpan di backend dan bisa dibalas dari Panel Admin (lihat badge notifikasi di tombol Admin).
- **Keranjang belanja dihapus** — tombol di tiap produk sekarang langsung membuka form pemesanan untuk produk itu.

---

## Deploy ke Netlify

1. **Upload kode ke GitHub** (Netlify deploy dari repo GitHub):
   ```
   cd buketku
   git init
   git add .
   git commit -m "BuketKu website"
   ```
   Buat repository baru di GitHub, lalu hubungkan dan push (`git remote add origin ...` lalu `git push -u origin main`).

2. **Buat site baru di Netlify**: app.netlify.com -> "Add new site" -> "Import an existing project" -> pilih repo GitHub kamu. Netlify otomatis membaca `netlify.toml`, tidak perlu ubah setting build.

3. **Isi Environment Variables** (Site configuration -> Environment variables):
   - `WA_NUMBER` -> nomor WhatsApp toko, format `62...` tanpa tanda `+`
   - `ADMIN_USERNAME` -> username login admin
   - `ADMIN_PASSWORD` -> password login admin

4. **Netlify Blobs** aktif otomatis begitu function pertama kali memanggilnya — tidak perlu setup tambahan. Ini layanan bawaan Netlify untuk menyimpan data pesanan & chat secara permanen, menggantikan file JSON yang dipakai versi Node biasa.

5. Klik **Deploy**. Setelah selesai, website bisa diakses di `https://nama-site-kamu.netlify.app` — produk, form pesanan, login admin, dan live chat semuanya langsung berfungsi (tidak perlu jalankan server apapun secara manual).

6. **(Opsional) Domain sendiri**: Site configuration -> Domain management -> Add a domain, lalu ikuti instruksi DNS yang diberikan.

### Catatan soal data di Netlify

- Pesanan & chat disimpan di **Netlify Blobs**, bukan file JSON — jadi aman tidak hilang walau function di-deploy ulang.
- Daftar produk (`netlify/functions/data/products.json`) bersifat statis. Untuk mengubah produk, edit file itu lalu push ke GitHub — Netlify akan deploy ulang otomatis.

---

## Deploy ke Railway / Render / VPS (alternatif)

Pakai folder `backend/` — lihat panduan lengkap di `backend/README.md` dan `frontend/README.md`. Backend ini pakai file JSON biasa (`backend/data/`), jadi kalau hosting-nya tidak punya penyimpanan permanen (seperti Render free tier), data bisa reset saat server restart — pertimbangkan pindah ke database sungguhan kalau pesanan sudah mulai banyak.

## Menjalankan di komputer sendiri (development)

Backend Node otomatis menyajikan frontend juga, jadi cukup 1 perintah:
```
cd backend
npm install
copy .env.example .env      (Windows)   atau   cp .env.example .env   (Mac/Linux)
npm start
```
Lalu buka **http://localhost:3000**.

(Kalau mau tes versi Netlify-nya secara lokal, install Netlify CLI: `npm install -g netlify-cli`, lalu di folder `buketku` jalankan `netlify dev`.)

## Sebelum dipakai untuk toko sungguhan

- Ganti nomor WhatsApp, kredensial admin, dan nomor rekening/QRIS dengan data toko asli.
- Tambahkan hashing password dan token sesi (JWT) untuk login admin bila ingin dipakai secara produksi dalam skala besar.
