# app-riset-pdp: Instrumen Komparasi Performa OpenCV.js vs face-api.js

![Face Detection](https://img.shields.io/badge/Face_Detection-OpenCV.js_vs_face--api.js-blue)
![Client-Side](https://img.shields.io/badge/Processing-Client--Side-purple)
![Scenarios](https://img.shields.io/badge/Factorial_Design-36_Scenarios-orange)
![Privacy](https://img.shields.io/badge/Privacy-by--Design-green)
![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey)
![DOI](https://img.shields.io/badge/DOI-10.5281%2Ffigshare.XXXXXXXX-blue)

Instrumen eksperimen laboratorium terkendali, berbasis **client-side**, untuk membandingkan performa dua pustaka deteksi wajah di peramban — **OpenCV.js** dan **face-api.js** — sebagai media ICT presensi digital mahasiswa. Dirancang untuk mengisi desain faktorial 36 skenario (3 tingkat pencahayaan × 4 sudut pose × 3 kelas perangkat) sebagaimana dilaporkan pada Sismadi, Manan, & Hartini (2026), *ACADEMIA: Jurnal Inovasi Riset Akademik*, 6(4), 3400–3408. [doi.org/10.51878/academia.v6i4.13792](https://doi.org/10.51878/academia.v6i4.13792)

Seluruh pemrosesan biometrik (deteksi wajah, ekstraksi fitur, verifikasi identitas) berjalan **sepenuhnya di peramban pengguna** — tidak ada data wajah yang dikirim ke server, sejalan dengan prinsip *privacy-by-design*. Instrumen ini berupa satu berkas HTML mandiri, tanpa proses build maupun instalasi dependensi.

Komponen pembanding sisi server tersedia terpisah di [api-riset-pdp](https://github.com/sismadi/api-riset-pdp) (Cloudflare Workers).

---

## Key Features

- **Dua Mesin Deteksi Berdampingan** — OpenCV.js (`Haar Cascade → LBP histogram 100×100 → cosine similarity`, threshold ≥ 0,85) dan face-api.js (`TinyFaceDetector → 68 landmark → descriptor 128-D → euclidean distance`, threshold ≤ 0,60), dengan overlay kotak deteksi & landmark *real-time*.
- **Desain Faktorial 36 Skenario** — 3 tingkat pencahayaan (100/300/500 lux) × 4 sudut pose (0°/15°/30°/45°) × 3 kelas perangkat (low-end/mid-range/high-end), dengan penanda skenario manual dan mode *runner* otomatis (1/3/5 trial per skenario).
- **Enrollment & Verifikasi Identitas** — pendaftaran wajah 5/10/15 frame per identitas ke kedua mesin sekaligus; pengujian dievaluasi sebagai tugas verifikasi identitas (termasuk penolakan yang tepat untuk kondisi "Tidak Dikenal").
- **Metrik Real-Time per Mesin** — latensi (ms), FPS-ekuivalen, estimasi penggunaan memori, dan jumlah trial tercatat, ditampilkan langsung selama pengujian.
- **Perbandingan Client vs Server** — memanggil endpoint API sungguhan (mis. `api-riset-pdp`) atau mensimulasikan jalur server-side dari parameter jaringan (RTT, bandwidth unggah/unduh, waktu proses server), lengkap dengan sesi 36-skenario otomatis untuk sisi server.
- **Ringkasan Agregat & Ekspor Data** — precision, recall, F1-Score, rata-rata latensi/FPS-eq per mesin dan per skenario; ekspor log mentah dan ringkasan dalam format CSV/JSON untuk dianalisis lebih lanjut (uji-t, Mann-Whitney U, Three-way ANOVA, dsb.) di luar instrumen.
- **Zero-Dependency Runtime** — satu berkas `index.html`; pustaka OpenCV.js dan face-api.js dimuat dari CDN. Tidak perlu Node.js, Webpack, atau build toolchain apa pun.

---

## Prerequisites & Installation

- Peramban modern dengan dukungan **WebGL** dan **getUserMedia** (disarankan Google Chrome atau Microsoft Edge terbaru).
- Kamera depan/webcam aktif dengan izin akses diberikan ke peramban.
- Akses melalui **HTTPS** atau `localhost` — sebagian besar peramban memblokir akses kamera pada halaman HTTP biasa.
- Koneksi internet pada saat pertama kali memuat halaman (untuk mengunduh pustaka OpenCV.js dan face-api.js dari CDN).

1. Clone repository ini:
   ```bash
   git clone https://github.com/sismadi/app-riset-pdp.git
   cd app-riset-pdp
   ```
2. Jalankan sebagai berkas statis, misalnya dengan Python:
   ```bash
   python -m http.server 8000
   ```
3. Buka `http://localhost:8000/index.html` di peramban. Tidak ada langkah build.

> Untuk pengambilan data resmi, disarankan hosting melalui layanan statis HTTPS seperti **GitHub Pages**, Cloudflare Pages, atau Netlify — lihat bagian *Repository Structure* untuk detail deployment.

---

## Usage

Antarmuka terbagi menjadi 4 tab:

| Tab | Isi |
|---|---|
| **1 · Persiapan** | Penanda skenario (lux/pose/perangkat), aktivasi kamera & status mesin deteksi, enrollment identitas |
| **2 · Pengujian Client** | Pencatatan trial manual (kedua mesin sekaligus) dan runner 36-skenario otomatis sisi client |
| **3 · Ringkasan Client** | Tabel ringkasan agregat & per-skenario (precision/recall/F1/latensi), tombol ekspor CSV/JSON |
| **4 · Server vs Client** | Uji jalur server (API sungguhan atau simulasi jaringan), runner 36-skenario otomatis sisi server, ringkasan perbandingan client vs server |

### Alur Kerja Singkat

1. Aktifkan kamera pada Tab 1 dan tunggu status kedua mesin siap.
2. Enroll setiap identitas (mis. `Mahasiswa_01`) pada kondisi standar (500 lux, 0°), 10 frame.
3. Pada Tab 2, pilih identitas *Ground Truth*, lalu jalankan trial manual atau mulai sesi **36 Skenario Otomatis**.
4. Lihat ringkasan pada Tab 3 dan unduh log/ringkasan (CSV/JSON).
5. (Opsional) Pada Tab 4, konfigurasikan endpoint `api-riset-pdp` atau parameter simulasi jaringan, lalu jalankan perbandingan client vs server.

### Model Estimasi Latensi Server (Mode Simulasi)

```
Total ≈ RTT + (Ukuran Payload × 8 / Bandwidth Unggah) + Waktu Proses Server
       + (Ukuran Respons × 8 / Bandwidth Unduh)
```

dengan ukuran respons diasumsikan kecil (± 2 KB berupa JSON koordinat/label).

---

## Repository Structure

```
app-riset-pdp/
└── index.html           # Instrumen mandiri — UI, logika pengujian, integrasi OpenCV.js & face-api.js
```

Repositori pembanding:

```
api-riset-pdp/           # https://github.com/sismadi/api-riset-pdp
├── src/index.js          # Cloudflare Worker — endpoint GET / dan POST /deteksi
└── wrangler.jsonc        # Konfigurasi deploy
```

---

## Metodologi Terkait

Instrumen ini digunakan untuk menghasilkan data pada eksperimen laboratorium terkendali dengan 180 observasi per pustaka (36 skenario × 5 trial), melibatkan 50 mahasiswa berusia 18–19 tahun, dengan skenario pengujian *closed-set*. Hasil selengkapnya (precision, recall, F1-Score, latensi, uji statistik) dilaporkan pada naskah publikasi terkait — lihat bagian *How to Cite*.

---

## How to Cite

```bibtex
@software{sismadi_app_riset_pdp_2026,
  author       = {Sismadi, Wawan},
  title        = {{app-riset-pdp: Instrumen Komparasi Performa OpenCV.js vs face-api.js
                  untuk Presensi Digital Mahasiswa}},
  year         = {2026},
  publisher    = {Figshare},
  doi          = {10.5281/figshare.XXXXXXXX},
  url          = {https://doi.org/10.5281/figshare.XXXXXXXX},
  note         = {Client-side face detection benchmarking instrument (OpenCV.js vs
                  face-api.js), 36-scenario factorial design. Companion server-side API:
                  https://github.com/sismadi/api-riset-pdp.
                  Repository: https://github.com/sismadi/app-riset-pdp}
}

@article{sismadi_academia_2026,
  author  = {Sismadi, Wawan and Manan, Abdul and Hartini, Estuti Fitri},
  title   = {Inovasi Media ICT Deteksi Wajah untuk Presensi Digital Mahasiswa:
             Analisis Akurasi dan Performa},
  journal = {ACADEMIA: Jurnal Inovasi Riset Akademik},
  volume  = {6},
  number  = {4},
  pages   = {3400--3408},
  year    = {2026},
  doi     = {10.51878/academia.v6i4.13792}
}
```
