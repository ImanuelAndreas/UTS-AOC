# ACO Delivery Quest — Kasus Nyata Kecamatan Setu

Simulasi **Ant Colony Optimization (ACO)** untuk optimasi rute pengiriman paket dari **Gudang** ke beberapa **rumah pelanggan**, lalu kembali ke Gudang. Rute **mengikuti jaringan jalan asli** (bukan garis lurus) dan seluruh operasi dibatasi di wilayah **Kecamatan Setu, Tangerang Selatan**.

## Struktur File

```
UTSSI/
├── aco_serpong.html  # Versi utama: peta nyata Leaflet + OSM, jarak & rute ikut jalan (OSRM) (single file)
├── aco_game.html     # Arsip versi lama: Canvas HTML5 dengan koordinat abstrak X,Y
├── README.md
└── PRESENTASI.md
```

## Cara Menjalankan

1. Buka `aco_serpong.html` dengan **double-click** (Chrome / Edge / Firefox).
2. Perlu **koneksi internet**: Leaflet.js + tile OpenStreetMap, batas wilayah (Nominatim), dan jarak/rute jalan (OSRM) dimuat online.
3. Simulasi langsung berjalan: batas Setu dimuat → rumah acak dibuat → jarak jalan dihitung → ACO berjalan.

> Versi lama `aco_game.html` tetap 100% offline (Canvas, tanpa CDN).

## Lokasi & Wilayah

- **Gudang** (titik mulai & akhir, tetap): **Gudang Hub Setu** di sekitar pusat Kecamatan Setu (~ -6.3468, 106.6760). Bisa di-drag (tetap harus di dalam area Setu).
- **Rumah pelanggan**: dibuat **acak** di dalam batas **Kecamatan Setu** (polygon OSM, relation `10989433`), lalu digeser ke **jalan terdekat** (snap OSRM) sehingga koordinat rumah berada di tepi jalan.
- **Batas wilayah** ditampilkan sebagai garis putus-putus. Titik (klik peta / drag marker) di luar Setu **ditolak**.
- Atur **Jumlah Rumah** (2–15) dan tekan **Acak Ulang Rumah** untuk membuat ulang sebaran rumah.

## Sumber Jarak

| Sumber | Kapan dipakai | Keterangan |
|---|---|---|
| **OSRM (jalan)** | Normal (online) | Matriks jarak antar semua titik diambil dari OSRM `table` service — **jarak tempuh melalui jalan**, satuan km. Rute terbaik digambar mengikuti **geometri jalan asli** (OSRM `route`). |
| **Haversine (lurus)** | Jika OSRM gagal/timeout | Fallback otomatis ke jarak garis lurus (`d = 2R·arcsin(√(sin²(Δφ/2)+cosφ₁·cosφ₂·sin²(Δλ/2)))`, `R = 6371 km`). |

Status sumber jarak tampil di panel **Rumah Pelanggan** (OSRM / Haversine).

## Fitur

| Fitur | Keterangan |
|---|---|
| Peta nyata | Leaflet + OpenStreetMap; otomatis di-zoom ke batas Kecamatan Setu |
| Batas wilayah | Polygon Kec. Setu dari OSM; titik di luar area ditolak |
| Gudang tetap + rumah acak | 1 gudang tetap, rumah pelanggan acak di dalam Setu, di-snap ke jalan |
| Jarak ikut jalan | Matriks jarak jalan OSRM (fallback Haversine) |
| Rute ikut jalan | Garis emas rute terbaik mengikuti geometri jalan (OSRM `route`, geojson) |
| Jejak pheromone | `L.Polyline` antar lokasi; tebal & terang = feromon kuat |
| Animasi semut | `L.circleMarker` bergerak menyusuri rute; kecepatan mengikuti jarak tempuh jalan |
| Grafik konvergensi | Canvas kecil di sidebar, jarak terbaik (km) per iterasi, real-time |
| Parameter live | Slider α, β, evaporasi, Q, jumlah semut, jumlah iterasi, jumlah rumah |
| HUD | Iterasi, jarak terbaik (km), rata-rata (km), jumlah titik, sumber jarak, rute teks |
| Kontrol | Mulai, Jeda, Step +1, Reset, Selesaikan Semua Iterasi, kecepatan animasi 0.5x–10x |

## Kontrol

| Aksi | Cara |
|---|---|
| Tambah rumah | Klik area kosong **di dalam** peta Setu |
| Pindah lokasi | Drag marker (ditolak bila keluar area Setu) |
| Hapus rumah | Klik kanan marker (Gudang tidak bisa dihapus) |
| Jumlah rumah | Slider **Jumlah Rumah** (ubah → rumah dibuat ulang) |
| Acak rumah | Tombol **Acak Ulang Rumah** |
| Jalankan / jeda | Tombol **Mulai** / **Jeda** |
| Maju 1 iterasi | Tombol **Step +1** (instan, tanpa animasi) |
| Hitung sampai habis | Tombol **Selesaikan Semua Iterasi** |
| Ubah parameter | Slider di panel kanan |

## Parameter ACO

| Parameter | Default | Fungsi |
|---|---|---|
| Alpha (α) | 1.0 | Pengaruh feromon. Besar → semut makin mengikuti jejak semut lain |
| Beta (β) | 2.0 | Pengaruh jarak (visibilitas 1/jarak). Besar → semut makin memilih jarak pendek |
| Evaporasi | 0.50 | Fraksi feromon yang menguap tiap iterasi |
| Pheromone (Q) | 100 | Jumlah feromon yang dideposit per rute (Q / total jarak) |
| Jumlah Semut | 10 | Rute yang dibuat per iterasi |
| Jumlah Iterasi | 50 | Batas iterasi maksimum |
| Jumlah Rumah | 6 | Banyak rumah pelanggan acak (2–15) |

## Cara Kerja Algoritma

1. **Bangun rute**: tiap semut memilih lokasi berikutnya secara probabilistik dari `pheromone^α × (1/jarak_jalan)^β`, dimulai dari Gudang, semua rumah dikunjungi tepat satu kali, lalu kembali ke Gudang.
2. **Evaporasi**: seluruh matriks feromon dikalikan `(1 − evaporasi)`.
3. **Deposit**: tiap rute menambahkan `Q / total_jarak` pada setiap edge yang dilewatinya.
4. **Evaluasi**: rute dengan jarak terbaik disimpan; geometri jalan rute terbaik diambil dari OSRM.
5. Ulangi sampai batas iterasi tercapai.

> Matriks jarak jalan bersifat **asimetris** (jalan satu arah), dan ACO memang memakai arah `jarak[kini][berikut]`.

## Contoh Hasil (seed 20261015, 6 rumah acak)

Rute terbaik (optimal brute-force 720 rute atas jarak jalan):

```
Gudang Hub Setu → Rumah 5 → Rumah 3 → Rumah 4 → Rumah 2 → Rumah 1 → Rumah 6 → Gudang Hub Setu
Total jarak ≈ 25.41 km (jarak jalan; Haversine akan lebih pendek karena garis lurus)
```

Jumlah rute acak tiap run berbeda (rumah diacak), sehingga hasil optimal juga berbeda-beda.

## Catatan Pengembangan

- Sumber jarak default = **OSRM** (jaringan jalan). Bila `router.project-osrm.org` tidak tersedia → fallback **Haversine** (kode inti ACO tidak berubah, hanya isi matriks `dist`).
- Batas wilayah diambil dari **Nominatim** (`lookup` relation `R10989433`); bila gagal → fallback kotak bbox Kecamatan Setu.
- Batas polygon & titik diuji via `pointInPolygon`; semua titik dipastikan berada di dalam Setu.
- Logika ACO dipisah dari DOM dan diekspor via `module.exports` saat dijalankan di Node.js (fungsi `haversineKm`, `insideSetu`, `generateHouses`, `buildDistHaversine`, dll) sehingga bisa diuji headless.
- Layanan OSRM/Nominatim publik hanya untuk penggunaan ringan (demo/tugas).
- Ide pengembangan: cache geometri jalan per-sisi agar semut juga melintas tepat di jalan, atau rute mengikuti profil lalu lintas.
