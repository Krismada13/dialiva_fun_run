# Dialiva Fun Run 2026 — BIB Number Searcher

Halaman web statis untuk peserta **Dialiva Fun Run** (19 Juli 2026) mengecek nomor BIB mereka secara mandiri, menggunakan nama lengkap + nomor HP yang dipakai saat mendaftar.

🔗 Lokasi pengambilan race pack: Klinik Dialiva Istimewa Yogyakarta, Bakungan, Wedomartani, Sleman, Yogyakarta.

---

## 📁 Struktur File

```
.
├── Index.html                          # Markup halaman (form pencarian + 3 state hasil)
├── Dialiva Fun Run BIB Searcher.css     # Styling (tema merah/biru, kartu BIB ala race bib asli)
└── Dialiva Fun Run BIB Searcher.js      # Logic pencarian + data dummy peserta
```

> ⚠️ **Catatan penamaan file:** `Index.html` memanggil CSS & JS dengan nama berspasi (`Dialiva Fun Run BIB Searcher.css/js`), tapi file yang diupload pakai underscore (`Dialiva_Fun_Run_BIB_Searcher.css/js`). Samakan dulu nama filenya (pilih salah satu konvensi) sebelum deploy, supaya `<link>` dan `<script src>` di HTML tidak 404.

---

## 🧠 Cara Kerja (versi saat ini)

1. User mengisi **nama lengkap** dan **nomor HP** di form.
2. Klik "Cari BIB Number" → tombol menunjukkan loading state (~700ms delay simulasi).
3. JS mencocokkan input ke array `DUMMY_REGISTRANTS` di `Dialiva Fun Run BIB Searcher.js`:
   - Nama dicocokkan case-insensitive, exact match.
   - Nomor HP dinormalisasi dulu (`normalizePhone()` — buang karakter non-digit, ubah prefix `62` → `0`) sebelum dibandingkan.
4. Jika ketemu → tampil **state-found**: kartu BIB lengkap (nomor BIB, kategori, no. pendaftaran, status, jadwal pengambilan race pack, nama peserta).
5. Jika tidak ketemu → tampil **state-notfound**: pesan + link WhatsApp panitia.

### Tiga UI State (`#state-idle`, `#state-notfound`, `#state-found`)
Dikontrol lewat class `.active` yang di-toggle oleh `setState(stateId)` di JS — hanya satu state yang tampil dalam satu waktu.

---

## 🗃️ Data Peserta (saat ini: dummy/hardcoded)

Saat ini **18 data dummy** di-hardcode langsung di `DUMMY_REGISTRANTS` (hasil konversi dari export CSV Google Sheet pendaftaran asli). Field per peserta:

| Field | Keterangan |
|---|---|
| `name` | Nama lengkap (dicocokkan saat search) |
| `bibName` | Nama yang akan dicetak di BIB (belum ditampilkan di UI saat ini) |
| `phone` | Nomor HP (dicocokkan saat search, setelah normalisasi) |
| `jerseySize` | Ukuran jersey (belum ditampilkan di UI saat ini) |
| `bib` | Nomor BIB, format `M-XXX` (pria) / `F-XXX` (wanita) |
| `regid` | Nomor pendaftaran — saat ini sama dengan `bib` |
| `category` | Kategori lomba (semua "5K" saat ini) |
| `cot` | Best Race Time — placeholder `00:00:00` |
| `status` | Status verifikasi pendaftaran |

> 🔴 **Ini bukan sumber data final.** Karena hardcoded di file JS, setiap ada peserta baru/update data harus edit & redeploy file ini manual. Lihat bagian **Next Steps** di bawah untuk rencana koneksi ke Google Sheet secara live.

---

## ⚠️ Hal yang perlu diperhatikan / potensi bug

- **Typo CSS:** `.meta-item.cot .val` punya `white-space: nowwarp;` — bukan property CSS valid (seharusnya `nowrap`). Browser akan mengabaikan baris ini diam-diam, jadi teks "Best Race Time" bisa wrap meski tidak diinginkan.
- **SVG pakai CSS variable yang tidak ada:** di `state-notfound`, stroke SVG memakai `stroke="var(--terracotta-deep)"` — variabel ini tidak didefinisikan di `:root` (yang ada `--red-deep`). Akibatnya ikon "tidak ditemukan" mungkin tidak berwarna sesuai desain (fallback ke default/transparent).
- **`r-pickup` dan elemen jadwal pengambilan** sudah di-hardcode langsung di HTML (15–16 Juli 2026), bukan diisi dari JS — jika tanggal berubah, edit langsung di `Index.html`.
- **Tombol "Simpan Tangkapan Layar"** (`#screenshotHint`) belum punya event listener / fungsi apa pun di JS — saat ini hanya elemen visual, klik tidak melakukan apa-apa.
- **Nama file dengan spasi** (lihat catatan struktur file di atas) berisiko bermasalah di banyak environment hosting/static server.

---

## 🚀 Next Steps (rencana ke depan)

Sesuai diskusi sebelumnya, arah pengembangan project ini:

1. **Koneksi live ke Google Sheets** — ganti `DUMMY_REGISTRANTS` dengan fetch ke Google Sheets API (lewat Apps Script Web App atau Google Sheets API + service account), supaya data selalu sinkron dengan sheet pendaftaran asli tanpa perlu redeploy.
2. **Migrasi ke Next.js (Pages Router)** — versi yang sedang dikerjakan paralel sudah punya API route kerja dan pola integrasi Google Sheets terdokumentasi. Versi statis ini bisa jadi fallback/prototipe cepat, atau dipakai sebagai referensi UI untuk di-port ke komponen React.
3. **Perbaikan kecil:** fix typo `nowwarp` → `nowrap`, fix variable CSS `--terracotta-deep` → `--red-deep`, samakan penamaan file (spasi vs underscore).
4. **Fungsi tombol screenshot** — tambahkan handler (misal pakai `html2canvas` untuk capture kartu BIB jadi gambar) atau hapus tombolnya jika tidak akan diimplementasikan.

---

## 🎨 Desain

Tema visual mengikuti identitas Dialiva Fun Run: merah (`--red: #F82D31`) sebagai warna utama aksi, biru (`--blue-deep: #1F52C9`) untuk branding/info, dengan kartu BIB bergaya tiket lomba asli (perforasi, lubang gantung, nomor besar ala Archivo Black). Font: **Archivo Black**, **Archivo Expanded**, **Plus Jakarta Sans** (Google Fonts).

Mendukung `prefers-reduced-motion` dan responsive breakpoint di `420px`.
