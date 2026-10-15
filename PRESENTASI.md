# Dokumen Presentasi — ACO Delivery Quest

Panduan lengkap untuk presentasi. Setiap bagian = **1 slide**. Isi "Tampilkan di slide" ditulis di layar, "Yang diucapkan" = naskah pembicara. Estimasi durasi total: **10-12 menit**.

---

## Slide 1 — Judul

**Tampilkan di slide**
> # ACO Delivery Quest — Setu
> ### Optimasi Rute Pengiriman Paket dengan Ant Colony Optimization
> ### Kasus Nyata Kecamatan Setu, Tangerang Selatan · Peta Leaflet + OSM · Rute Ikut Jalan (OSRM)
> Nama: ................  |  NIM: ................
> Implementasi: Python (numpy, matplotlib) → Game Web interaktif (HTML + Leaflet.js)

**Yang diucapkan**
"Selamat pagi/siang. Saya akan mempresentasikan proyek optimasi rute pengiriman paket menggunakan algoritma Ant Colony Optimization, disingkat ACO. Saya membuat simulasi ini dalam bentuk game interaktif di browser, dan kali ini memakai kasus nyata pengiriman paket ke rumah pelanggan di daerah Setu, Tangerang Selatan, dengan peta asli. Yang istimewa, jarak dan rutenya mengikuti jalan sungguhan, bukan garis lurus."

---

## Slide 2 — Latar Belakang & Masalah

**Tampilkan di slide**
> **Masalah: Vehicle Routing / Travelling Salesman Problem (TSP)**
> - 1 Gudang mengirim ke banyak rumah pelanggan, lalu kembali ke Gudang
> - Mencari urutan kunjungan dengan **total jarak paling pendek**
> - Jumlah rute meledak: untuk 5 rumah ada **120 kemungkinan**, 6 rumah **720**, 10 rumah >**3,6 juta** rute
> - Mencoba semua rute (brute-force) jadi tidak realistis
> **Solusi:** algoritma metaheuristik → Ant Colony Optimization

**Yang diucapkan**
"Permasalahan utamanya: satu gudang harus mengirim paket ke beberapa rumah pelanggan dan kembali lagi. Kita ingin urutan kunjungan yang jaraknya paling pendek. Masalahnya, jumlah kombinasi rute bertambah secara faktorial. Untuk lima rumah saja ada 120 kemungkinan, untuk enam rumah ada 720, dan untuk sepuluh rumah lebih dari tiga juta. Menghitung semuanya satu per satu tidak efisien. Karena itu saya pakai pendekatan metaheuristik, yaitu ACO, yang meniru perilaku semut nyata mencari makanan."

---

## Slide 3 — Apa itu Ant Colony Optimization?

**Tampilkan di slide**
> **Inspirasi dari alam:** semut asli menemukan jalur terpendek ke makanan lewat **feromon**.
> - Semut menandai jalurnya dengan feromon
> - Jalur pendek dilalui lebih cepat & lebih sering → feromon menumpuk
> - Semut lain cenderung mengikuti feromon terkuat → lama-kelamaan **semua memilih jalur terpendek**
> **Konsep kunci:** *stigmergy* — komunikasi tidak langsung melalui jejak kimia.

**Yang diucapkan**
"ACO meniru cara kerja semut. Semut nyata menandai jalan yang mereka lalui dengan zat kimia bernama feromon. Semakin pendek sebuah jalur, semakin cepat dilalui, sehingga semakin sering dilewati dan feromonnya makin kuat. Semut berikutnya cenderung mengikuti feromon yang kuat, sehingga lama-kelamaan seluruh koloni memilih jalur terpendek. Inilah yang disebut stigmergy: komunikasi tidak langsung lewat jejak. Prinsip ini yang saya tiru di dalam program."

---

## Slide 4 — Data Masalah

**Tampilkan di slide**
> **Lokasi: pengiriman paket ke rumah pelanggan di Kecamatan Setu, Tangerang Selatan.**
> - **Gudang (tetap)**: Gudang Hub Setu, sekitar pusat Kecamatan Setu — titik mulai & akhir
> - **Rumah pelanggan**: diacak di dalam batas Kecamatan Setu, lalu digeser (snap) ke jalan terdekat
> - Batas wilayah = **polygon asli Kecamatan Setu** dari OpenStreetMap (relation `10989433`)
> - Titik di luar area Setu otomatis ditolak
>
> **Jarak = jarak tempuh melalui jalan asli (jaringan jalan) via OSRM** — hasil dalam km; otomatis memakai Haversine (garis lurus) bila OSRM tidak tersedia.

**Contoh koordinat (seed tetap, 6 rumah):**

| Titik | Tempat | Latitude | Longitude |
|---|---|---|---|
| 0 | Gudang Hub Setu | -6.346759 | 106.676385 |
| 1 | Rumah 1 | -6.334613 | 106.669301 |
| 2 | Rumah 2 | -6.347096 | 106.669065 |
| 3 | Rumah 3 | -6.343332 | 106.649216 |
| 4 | Rumah 4 | -6.341034 | 106.658807 |
| 5 | Rumah 5 | -6.353729 | 106.659972 |
| 6 | Rumah 6 | -6.355129 | 106.712319 |

**Yang diucapkan**
"Datanya adalah pengiriman nyata ke rumah-rumah pelanggan di Kecamatan Setu, Tangerang Selatan. Satu gudang pusat sebagai titik mulai dan akhir, lalu enam rumah pelanggan. Rumah-rumah ini diacak di dalam batas Kecamatan Setu, jadi setiap kali dijalankan sebarannya bisa berbeda. Yang penting, batas wilayahnya saya ambil dari peta OpenStreetMap, dan jarak antar titik bukan garis lurus, melainkan jarak tempuh melalui jalan asli. Kalau layanan peta jalan sedang tidak bisa diakses, program otomatis kembali ke rumus Haversine sebagai cadangan."

---

## Slide 4b — Matriks Jarak Jalan (opsional, jika ada waktu)

**Tampilkan di slide**

|  | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|---|
| **0 Gudang** | 0.00 | 2.12 | 0.94 | 3.77 | 2.88 | 2.65 | 5.01 |
| **1 Rumah 1** | 2.12 | 0.00 | 2.23 | 4.66 | 3.77 | 4.06 | 7.03 |
| **2 Rumah 2** | 0.94 | 2.23 | 0.00 | 3.31 | 2.43 | 2.88 | 5.86 |
| **3 Rumah 3** | 3.77 | 4.66 | 3.31 | 0.00 | 2.36 | 3.92 | 8.68 |
| **4 Rumah 4** | 2.88 | 3.77 | 2.43 | 2.36 | 0.00 | 3.04 | 7.80 |
| **5 Rumah 5** | 2.65 | 4.05 | 2.88 | 3.90 | 3.02 | 0.00 | 7.56 |
| **6 Rumah 6** | 4.81 | 6.82 | 5.65 | 8.47 | 7.59 | 7.35 | 0.00 |

*(satuan km, jarak tempuh jalan asli via OSRM; **tidak simetris** karena ada jalan satu arah)*

**Yang diucapkan**
"Dari koordinat rumah tadi dibentuk matriks jarak dalam kilometer, tapi ini bukan jarak garis lurus — ini jarak tempuh melalui jalan asli. Perhatikan matriksnya tidak simetris: jarak dari titik lima ke titik empat sedikit berbeda dengan arah sebaliknya, karena ada jalan satu arah dan rute memutar. Matriks inilah yang dipakai algoritma untuk menilai seberapa jauh setiap pilihan rumah berikutnya."

---

## Slide 5 — Parameter Algoritma

**Tampilkan di slide**
> | Parameter | Simbol | Nilai | Peran |
> |---|---|---|---|
> | Jumlah semut | — | 10 | Banyak rute dicoba per iterasi |
> | Jumlah iterasi | — | 50 | Berapa kali proses diulang |
> | Pengaruh feromon | α (alpha) | 1.0 | Seberapa ikut jejak semut lain |
> | Pengaruh jarak | β (beta) | 2.0 | Seberapa mengutamakan jarak pendek |
> | Evaporasi | ρ (rho) | 0.5 | Penguapan feromon tiap iterasi |
> | Konstanta feromon | Q | 100 | Jumlah feromon yang disimpan tiap rute |

**Yang diucapkan**
"Ada beberapa parameter penting. Jumlah semut menentukan berapa rute yang dicoba setiap iterasi. Alpha mengatur seberapa besar semut mengikuti feromon dari semut sebelumnya, dan betha mengatur seberapa besar semut memilih jarak yang pendek. Evaporasi membuat feromon lama menguap supaya algoritma tidak terjebak pada solusi awal dan tetap bisa menjelajah. Q menentukan seberapa banyak feromon yang ditaburkan pada sebuah rute."

---

## Slide 6 — Rumus Inti

**Tampilkan di slide**
> **1. Peluang memilih lokasi berikutnya:**
>
> ```
>         (τ_ij)^α · (1/d_ij)^β
> P_ij = ─────────────────────────
>         Σ (τ_ik)^α · (1/d_ik)^β
> ```
> τ = feromon, d = jarak, α & β = parameter
>
> **2. Evaporasi feromon:**
> ```
> τ_ij ← (1 − ρ) · τ_ij
> ```
> **3. Deposit feromon dari tiap semut:**
> ```
> τ_ij ← τ_ij + Q / L        (L = total jarak rute semut)
> ```

**Yang diucapkan**
"Ini inti matematisnya. Rumus pertama menghitung peluang semut memilih lokasi berikutnya. Pembilangnya gabungan feromon dipangkat alpha dan kebalikan jarak dipangkat betha. Semakin banyak feromon dan semakin dekat jaraknya, semakin besar peluang dipilih. Lalu feromon menguap sesuai rumus kedua, dan setiap semut menambahkan feromon sebesar Q dibagi total jaraknya — artinya rute yang pendek mendapat feromon lebih banyak."

---

## Slide 7 — Alur Algoritma

**Tampilkan di slide**
> 1. **Inisialisasi** — semua feromon = 1
> 2. **Bangun rute** — setiap semut memilih lokasi berikutnya secara probabilistik dari Gudang, mengunjungi semua pelanggan sekali, lalu kembali ke Gudang
> 3. **Evaporasi** — feromon dikalikan (1 − ρ)
> 4. **Deposit** — tiap rute menambah Q / jarak pada edge yang dilewati (dua arah)
> 5. **Evaluasi** — simpan rute dengan jarak terbaik
> 6. **Ulangi** langkah 2-5 sebanyak jumlah iterasi
> 7. Ambil **rute terbaik** sebagai solusi

**Yang diucapkan**
"Alur kerjanya seperti ini: pertama semua feromon diisi nilai satu. Setiap semut membangun rute dari gudang, memilih langkah berikutnya secara acak terbobot, sampai semua pelanggan dikunjungi lalu kembali ke gudang. Setelah semua semut selesai, feromon menguap, lalu setiap rute menaburkan feromon sesuai kualitasnya. Rute terbaik dicatat. Proses ini diulang puluhan kali sampai rute yang ditemukan makin pendek, dan pada akhirnya kita pakai rute terbaiknya."

---

## Slide 8 — Implementasi: dari Python ke Web

**Tampilkan di slide**
> | Aspek | Versi Python | Versi Game Web (Setu) |
> |---|---|---|
> | Bahasa | Python | HTML + JavaScript (vanilla) |
> | Array | `numpy` | `Array` / `Math` |
> | Pemilihan acak | `random.choices` | *roulette wheel* manual |
> | Data lokasi | koordinat abstrak | **rumah pelanggan acak di Kec. Setu** |
> | Jarak | Euclidean | **jarak tempuh jalan (OSRM), fallback Haversine** |
> | Batas area | — | **polygon Kec. Setu dari OSM** |
> | Peta | matplotlib (statis) | **Leaflet.js + OpenStreetMap (live)** |
> | Pengguna | statis, tiap run dijalankan ulang | interaktif, real-time |

**Yang diucapkan**
"Awalnya proyek ini saya buat di Python menggunakan numpy dan matplotlib, dengan visualisasi statis. Kemudian saya port seluruh logikanya ke JavaScript tanpa mengubah algoritmanya. Yang berubah hanya medianya: data lokasi kini rumah pelanggan acak di Kecamatan Setu, jarak dihitung dari jaringan jalan asli lewat OSRM dengan cadangan Haversine, batas wilayahnya polygon kecamatan dari OpenStreetMap, dan visualisasi digambar di atas peta nyata menggunakan Leaflet. Fungsi inti ACO sama persis."

---

## Slide 8b — Cuplikan Kode Kunci

**Tampilkan di slide**
```javascript
// 1) Matriks jarak JALAN dari OSRM (1 request, meter -> km)
const url = OSRM_BASE + '/table/v1/driving/' +
            titik.map(t => t.lng + ',' + t.lat).join(';') + '?annotations=distance';
dist = (await (await fetch(url)).json()).distances
         .map(row => row.map(m => m / 1000));   // km, asimetris

// 2) Peluang memilih rumah berikutnya (jarak jalan)
const p = Math.pow(pher[cur][nx], alpha) * Math.pow(1 / dist[cur][nx], beta);

// 3) Evaporasi + deposit feromon
pher[i][j] *= (1 - evaporasi);            // menguap
pher[a][b] += Q / total_jarak;            // disimpan semut

// 4) Fallback bila OSRM tak tersedia: Haversine (garis lurus)
const d = 2 * 6371 * Math.asin(Math.sqrt(
  Math.sin(dLat/2)**2 + Math.cos(la1)*Math.cos(la2)*Math.sin(dLng/2)**2));
```

**Yang diucapkan**
"Ini cuplikan kode inti. Pertama, matriks jarak diambil sekali dari layanan OSRM, berisi jarak tempuh melalui jalan untuk semua pasangan titik, satuannya kilometer. Perhatikan matriks ini bisa tidak simetris karena jalan satu arah. Baris berikutnya menghitung peluang memilih rumah berikutnya sesuai rumus tadi, memakai jarak jalan. Lalu ada evaporasi dan deposit feromon. Terakhir, kalau layanan OSRM gagal, saya siapkan cadangan rumus Haversine untuk jarak garis lurus."

---

## Slide 9 — Fitur Game / Demo (LIVE)

**Tampilkan di slide**
> **Yang bisa dilakukan di simulasi (peta nyata Kecamatan Setu):**
> - Peta asli Leaflet + OpenStreetMap, otomatis di-zoom ke batas Kecamatan Setu
> - **Batas wilayah Setu** (garis putus-putus) — titik di luar area ditolak
> - Gudang tetap (kuning "G") + rumah pelanggan acak di dalam Setu, di-snap ke jalan
> - **Rute terbaik mengikuti jalan asli** (garis emas)
> - Semut beranimasi; garis bercahaya = kekuatan feromon
> - Slider **Jumlah Rumah** + tombol **Acak Ulang Rumah**
> - Atur α, β, evaporasi, Q, semut, iterasi secara langsung
> - Grafik konvergensi (km) update real-time; HUD menampilkan sumber jarak (OSRM/Haversine)
> - Klik peta = tambah rumah (di dalam Setu), drag = pindah, klik kanan = hapus

**Yang diucapkan**
"Dan ini hasilnya. Peta yang tampil adalah peta asli Kecamatan Setu dari OpenStreetMap, lengkap dengan batas wilayahnya berupa garis putus-putus. Titik kuning bertanda G adalah gudang, titik biru bernomor adalah rumah pelanggan yang diacak di dalam area dan digeser tepat ke jalan terdekat. Titik hijau adalah semut yang bergerak. Garis yang makin tebal dan terang berarti feromonnya makin kuat. Jalur emas adalah rute terbaik, dan penting: jalur itu **mengikuti jalan asli**, bukan garis lurus. Saya juga bisa mengatur jumlah rumah dan mengacaknya ulang, lalu algoritma menghitung ulang dari awal. *(lakukan demo singkat di layar)*"

> **Tips demo:** biarkan beberapa iterasi berjalan, lalu klik **Selesaikan Semua Iterasi** untuk mempercepat dan perlihatkan jalur emas yang berkelok mengikuti jalan + grafik menurun. Tekan **Acak Ulang Rumah** untuk menunjukkan sebaran rumah yang berubah.

---

## Slide 10 — Hasil

**Tampilkan di slide**
> **Contoh rute terbaik (seed tetap, 6 rumah acak), jarak tempuh jalan:**
> ```
> Gudang → Rumah 5 → Rumah 3 → Rumah 4 → Rumah 2 → Rumah 1 → Rumah 6 → Gudang
> ```
> **Total jarak jalan ≈ 25.41 km** (sama dengan solusi optimal brute-force 720 rute)
>
> | Segmen | Jarak jalan (km) |
> |---|---|
> | Gudang → Rumah 5 | 2.65 |
> | Rumah 5 → Rumah 3 | 3.90 |
> | Rumah 3 → Rumah 4 | 2.36 |
> | Rumah 4 → Rumah 2 | 2.43 |
> | Rumah 2 → Rumah 1 | 2.23 |
> | Rumah 1 → Rumah 6 | 7.03 |
> | Rumah 6 → Gudang | 4.81 |
> | **Total** | **≈ 25.41** |
>
> Grafik konvergensi: jarak terbaik **menurun** lalu stabil (konvergen)

**Yang diucapkan**
"Hasil akhirnya, dari enam rumah acak, rute terbaik yang ditemukan adalah gudang menuju rumah lima, rumah tiga, rumah empat, rumah dua, rumah satu, rumah enam, lalu kembali ke gudang, dengan total jarak tempuh sekitar 25 koma 4 kilometer melalui jalan asli. Menariknya, bila saya coba semua 720 kemungkinan rute secara paksa, hasil optimalnya juga 25 koma 4 kilometer, jadi ACO berhasil menemukan solusi optimal. Karena rumahnya diacak, setiap kali dijalankan hasilnya bisa berbeda. Grafik konvergensi menunjukkan jarak terbaik terus menurun di iterasi awal lalu stabil, artinya algoritma sudah konvergen."

---

## Slide 11 — Kesimpulan

**Tampilkan di slide**
> - ACO berhasil menemukan rute pengiriman terpendek dari 720 kemungkinan (6 rumah acak)
> - Dapat menyelesaikan masalah TSP / routing dengan efisien
> - Parameter α, β, ρ, Q memengaruhi kecepatan & kualitas konvergensi
> - **Jarak & rute mengikuti jaringan jalan asli (OSRM)** dengan fallback Haversine
> - Visualisasi game membuat proses kerja algoritma mudah dipahami
>
> **Pengembangan lanjutan:**
> - Bandingkan ACO vs brute-force / algoritma genetika
> - Multi-gudang, kapasitas kendaraan, & jam operasional
> - Detail geometri tiap sisi jalan agar animasi semut tepat di lajur

**Yang diucapkan**
"Kesimpulannya, algoritma ACO berhasil menemukan rute terpendek dan menyelesaikan permasalahan routing ini dengan efisien dibanding mencoba semua kemungkinan. Yang membuat versi ini lebih nyata, jarak dan rutenya mengikuti jalan asli, bukan garis lurus. Parameter-parameternya berpengaruh pada kualitas hasil, dan membungkusnya dalam bentuk game membuat cara kerja algoritma jauh lebih mudah dipahami. Ke depannya, simulasi ini bisa dikembangkan untuk membandingkan dengan algoritma lain, mendukung banyak gudang dan kapasitas kendaraan, atau menambahkan jam operasional dan kondisi lalu lintas."

---

## Slide 12 — Penutup

**Tampilkan di slide**
> # Terima Kasih
> **Ada pertanyaan?**

**Yang diucapkan**
"Terima kasih atas perhatiannya. Saya siap menjawab pertanyaan."

---

## Lampiran — Antisipasi Pertanyaan (Q&A)

**T: Mengapa memilih ACO, bukan algoritma lain?**
> "ACO mudah diimplementasikan, cocok untuk masalah kombinatorial seperti TSP, dan sifatnya adaptif — bisa diperluas ke banyak variasi. Selain itu visualnya menarik karena meniru perilaku semut asli."

**T: Apa fungsi evaporasi, kenapa feromon harus menguap?**
> "Agar feromon lama yang mungkin berasal dari rute jelek menghilang, sehingga semut tetap bisa menjelajah opsi baru dan tidak terjebak pada solusi awal yang belum tentu terbaik. Ini menjaga keseimbangan antara eksplorasi dan eksploitasi."

**T: Apakah solusinya pasti optimal?**
> "Untuk 6 rumah, ya — hasilnya sama dengan optimal setelah mencoba semua 720 kemungkinan rute. ACO menemukan hasil yang sama dengan solusi optimal di kasus kecil ini. Untuk jumlah rumah lebih banyak, ACO bersifat aproksimasi namun cepat dan hasilnya cukup baik."

**T: Bagaimana jarak dihitung, kenapa bisa tidak simetris?**
> "Jarak diambil dari layanan OSRM yang menghitung rute melalui jaringan jalan asli, jadi berupa jarak tempuh, bukan garis lurus. Karena ada jalan satu arah dan rute yang memutar, jarak A ke B bisa berbeda dengan B ke A. Kalau layanan OSRM tidak tersedia, program otomatis memakai rumus Haversine sebagai cadangan."

**T: Mengapa ada rumus Haversine kalau sudah pakai OSRM?**
> "Haversine adalah cadangan agar simulasi tetap berjalan meski layanan jalan sedang tidak bisa diakses. Haversine menghitung jarak garis lurus di permukaan bumi. Sumber jarak yang sedang dipakai ditampilkan di panel, jadi kita tahu apakah hasilnya berbasis jalan atau garis lurus."

**T: Bagaimana kalau jumlah pelanggan banyak, apakah tetap akurat?**
> "Untuk kasus besar ACO bersifat aproksimasi — tidak dijamin optimal, tapi cepat dan solusinya cukup baik. Jumlah semut dan iterasi bisa dinaikkan untuk meningkatkan kualitas; jumlah rumah juga bisa diatur lewat slider."

**T: Apa bedanya kode ini dengan versi Python asli?**
> "Logikanya identik. Yang berubah hanya bahasa dan tampilan: numpy diganti fungsi JavaScript, matplotlib diganti peta interaktif Leaflet + OpenStreetMap, jarak Euclidean diganti jarak jalan OSRM dengan cadangan Haversine, ditambah batas wilayah dan kontrol interaktif."

**T: Kenapa α dan β nilainya 1 dan 2?**
> "Itu nilai umum yang direkomendasikan. β lebih besar dari α memberi penekanan pada jarak sehingga semut cenderung memilih rute pendek, sementara α menjaga pengaruh feromon. Bisa diubah lewat slider untuk melihat efeknya."

---

## Lampiran B — Kode Inti ACO & Penjelasan (Pheromone, dll)

> Kutipan kode di bawah diambil dari `aco_serpong.html`. Nomor baris bisa **bergeser**
> kalau file diedit; nama fungsinya tetap jadi acuan yang aman. Komentar penjelasan
> ditambahkan agar tiap baris mudah dijelaskan saat sidang/laporan.

**Alur singkat algoritma dalam kode:**
`buildPher` (isi feromon awal) → `buildRoute`/`pickNext` (semut 1 per 1 jalan, tiap langkah
diundi pakai feromon + jarak) → `routeLength` (hitung panjang rute) → `finalize`
(feromon menguap lalu ditambah sesuai kualitas rute) → ulang untuk tiap iterasi → rute terbaik.

### B.1 Inisialisasi Feromon
`aco_serpong.html:420`

```javascript
function buildPher(){
  const n = nodes.length;                                        // jumlah titik (1 gudang + rumah)
  pher = Array.from({length:n}, function(){                    // buat matriks n x n
    return new Array(n).fill(1);                               // semua sisi mulai τ = 1
  });
}
```
**Penjelasan:** sebelum semut berjalan, belum ada feromon. Semua jalur diberi nilai 1
(`τ = 1`) supaya peluangnya merata, lalu feromon akan "berevolusi" di setiap iterasi.

### B.2 Panjang Rute (Nilai L)
`aco_serpong.html:424`

```javascript
function routeLength(r){
  let t = 0;                                                    // total jarak rute
  for(let i = 0; i < r.length - 1; i++)                        // telusuri tiap segmen
    t += dist[r[i]][r[i+1]];                                   // jumlahkan jarak antar titik
  return t;                                                    // hasil = L (km)
}
```
**Penjelasan:** `r` adalah urutan indeks titik, mis. `[0,3,1,4,2,0]` (mulai & berakhir di gudang).
Fungsi menjumlahkan elemen matriks `dist` sepanjang urutan itu. Nilai `L` inilah yang jadi
tolak ukur kualitas semut.

### B.3 Peluang Memilih Rumah Berikutnya — INTI PHEROMONE
`aco_serpong.html:429`

```javascript
function pickNext(cur, unvisited){
  let sum = 0;
  const w = [];
  for(let k = 0; k < unvisited.length; k++){
    const nx = unvisited[k];                                   // kandidat rumah berikutnya
    const d  = Math.max(dist[cur][nx], 1e-6);                  // jarak cur->nx (hindari bagi 0)
    const p  = Math.pow(pher[cur][nx], P.alpha)               // (1) pengaruh FEROMON  : τ^α
             * Math.pow(1 / d, P.beta);                       // (2) pengaruh JARAK     : (1/d)^β
    w.push(p);                                                // simpan bobot
    sum += p;                                                 // total bobot
  }
  if(!(sum > 0)) return unvisited[Math.floor(Math.random() * unvisited.length)];
  let r = Math.random() * sum;                                // --- roulette wheel ---
  for(let i = 0; i < unvisited.length; i++){                  // acak tapi berpihak ke bobot besar
    r -= w[i];
    if(r <= 0) return unvisited[i];                           // rumah terpilih
  }
  return unvisited[unvisited.length - 1];
}
```
**Penjelasan:** untuk setiap kandidat dihitung bobotnya dengan rumus ACO:

$$P_{cur\to nx} = \frac{\tau_{cur,nx}^{\alpha}\cdot (1/d_{cur,nx})^{\beta}}{\sum \tau^{\alpha}\cdot(1/d)^{\beta}}$$

- `pher[cur][nx]` = feromon sisi → makin **sering** dilalui, makin besar peluang dipilih (α).
- `1/d` = makin **dekat**, makin besar peluang dipilih (β).
- Pemilihan akhir memakai *roulette wheel* (`Math.random() * sum`): bersifat acak, tetapi
  jalur dengan bobot besar lebih mungkin terpilih. Ini yang menjaga keseimbangan
  eksplorasi (coba jalur baru) dan eksploitasi (pakai jalur bagus).

### B.4 Membangun Rute Satu Semut
`aco_serpong.html:447`

```javascript
function buildRoute(){
  const route = [0];                                          // mulai dari GUDANG (indeks 0)
  const unvisited = [];
  for(let i = 1; i < nodes.length; i++) unvisited.push(i);    // semua rumah belum dikunjungi
  while(unvisited.length){
    const nx = pickNext(route[route.length - 1], unvisited);  // pilih langkah berikutnya
    route.push(nx);                                           // kunjungi rumah itu
    unvisited.splice(unvisited.indexOf(nx), 1);               // tandai sudah dikunjungi
  }
  route.push(0);                                              // kembali ke GUDANG
  return route;
}
```
**Penjelasan:** satu semut selalu mulai dari gudang, memilih rumah berikutnya satu per satu
(sesuai aturan pheromone), setiap rumah hanya dikunjungi sekali, lalu pulang ke gudang.

### B.5 Satu Batch Semut
`aco_serpong.html:459`

```javascript
function genBatch(){
  const rs = [], ds = [];
  for(let k = 0; k < P.ants; k++){                            // sejumlah "jumlah semut"
    const r = buildRoute();                                   // tiap semut bikin rute sendiri
    rs.push(r);                                               // kumpulkan rutenya
    ds.push(routeLength(r));                                  // beserta panjangnya
  }
  return {rs:rs, ds:ds};                                      // dikembalikan sebagai batch
}
```
**Penjelasan:** dalam satu iterasi, `P.ants` semut jalan bersamaan dan menghasilkan
kumpulan rute (dan panjangnya). Kumpulan ini nanti dievaluasi di `finalize`.

### B.6 Evaporasi + Deposit Feromon + Simpan Rute Terbaik — INTI PHEROMONE
`aco_serpong.html:468`

```javascript
function finalize(routes, ds){
  const n = pher.length;
  for(let i = 0; i < n; i++)
    for(let j = 0; j < n; j++)
      pher[i][j] *= (1 - P.evap);                             // (A) EVAPORASI: τ ← (1-ρ)·τ
  let sum = 0;
  for(let k = 0; k < routes.length; k++){
    const L = ds[k];
    sum += L;
    if(L > 0){
      const add = P.Q / L;                                    // (B) jumlah feromon = Q / L
      const r = routes[k];
      for(let i = 0; i < r.length - 1; i++){
        const a = r[i], b = r[i+1];
        pher[a][b] += add;                                    // deposit ke sisi yang dilalui
        pher[b][a] += add;                                    // (dua arah, jarak asimetris)
      }
    }
    if(L < bestDist){                                         // (C) simpan rute terbaik
      bestDist = L;
      bestRoute = routes[k].slice();
      bestGeomDirty = true;                                   // tandai perlu ambil geometri jalan
    }
  }
  avgLast = routes.length ? sum / routes.length : 0;          // rata-rata jarak batch
  history.push(bestDist);                                     // untuk grafik konvergensi
  iteration++;
}
```
**Penjelasan:**
- **(A) Evaporasi** `τ ← (1−ρ)·τ`: mengurangi feromon di semua sisi agar jalur lama/buruk
  terlupakan dan semut tidak terjebak di solusi awal.
- **(B) Deposit** `τ += Q/L`: semut menambah feromon pada sisi yang dilaluinya. Karena dibagi
  `L`, rute yang **lebih pendek** (L kecil) menyumbang feromon **lebih banyak** → lama-lama
  jadi jalur favorit.
- **(C)** rute terbaik sejauh ini diingat di `bestDist`/`bestRoute`, dan `history` dipakai
  untuk grafik konvergensi.

### B.7 Alur Iterasi (Animasi & Selesaikan)
`aco_serpong.html:496`

```javascript
function afterIter(){                                          // dipanggil setelah 1 iterasi
  if(iteration >= P.iters){                                   // sudah cukup iterasi?
    running = false;                                          // berhenti
    toast('Iterasi selesai! Jarak terbaik: ' + fmt(bestDist) + ' km');
  }else if(running){ startAnimIter(); }                       // lanjut iterasi berikutnya
}
function stepInstant(){                                       // tombol "Selesaikan Semua"
  if(iteration >= P.iters) return false;
  const b = genBatch();                                       // bikin rute semut
  finalize(b.rs, b.ds);                                       // update feromon + best
  afterIter();
  return true;
}
function startAnimIter(){                                     // versi beranimasi
  const b = genBatch();
  animRoutes = b;                                             // rute untuk digambar
  ants = b.rs.map(function(r){ return {route:r, leg:1, t:0, done:r.length <= 2}; });
  if(hasDOM) spawnAntMarkers(ants.length);                    // munculkan titik semut di peta
}
```
**Penjelasan:** satu "iterasi" = `genBatch` (semua semut jalan) + `finalize` (feromon
diperbarui). `startAnimIter` menampilkan prosesnya sebagai animasi; `stepInstant` langsung
melompat ke hasil akhir.

### B.8 Sumber Jarak (OSRM + Fallback Haversine)
`aco_serpong.html:380`

```javascript
/* Ambil matriks jarak jalan asli dari OSRM (1 request). Fallback: Haversine. */
function loadMatrix(){
  const gen = ++matrixGen;                                    // penanda request terbaru
  distReady = false;
  const coords = nodes.map(function(n){ return n.lng + ',' + n.lat; }).join(';');
  const url = OSRM_BASE + '/table/v1/driving/' + coords + '?annotations=distance';
  fetch(url, { signal: abortTimer(20000) })                   // batas waktu 20 detik
    .then(function(r){ if(!r.ok) throw new Error('http ' + r.status); return r.json(); })
    .then(function(j){
      if(gen !== matrixGen) return;                           // abaikan bila sudah usang
      if(!j || j.code !== 'Ok' || !j.distances) throw new Error('bad osrm');
      dist = j.distances.map(function(row){
        return row.map(function(m){ return (m == null ? 0 : m / 1000); }); // meter -> km
      });
      snapCoords(j);                                          // geser titik tepat ke jalan
      distSource = 'osrm';                                    // tandai sumber jarak
      distReady = true; kick();                               // lanjutkan simulasi
    })
    .catch(function(){                                        // OSRM gagal -> pakai Haversine
      if(gen !== matrixGen) return;
      buildDistHaversine();
      distSource = 'haversine';
      distReady = true; kick();
    });
}
```
`aco_serpong.html:354`

```javascript
function buildDistHaversine(){                                // cadangan: jarak garis lurus (Bumi)
  const n = nodes.length;
  dist = Array.from({length:n}, function(){ return new Array(n).fill(0); });
  for(let i = 0; i < n; i++)
    for(let j = i + 1; j < n; j++){
      const d = haversineKm(nodes[i], nodes[j]);              // rumus Haversine -> km
      dist[i][j] = d;
      dist[j][i] = d;                                         // simetris
    }
}
```
`aco_serpong.html:365`

```javascript
function snapCoords(j){                                       // geser koordinat ke jalan terdekat
  const pts = (j && (j.destinations || j.sources)) || null;   // balikan OSRM berisi posisi snap
  if(!pts) return;
  for(let i = 0; i < nodes.length && i < pts.length; i++){
    const p = pts[i];
    if(!p) continue;
    if(p.name) nodes[i].road = p.name;                        // simpan nama jalan
    const loc = p.location;
    if(loc && loc.length === 2 && insideSetu(loc[1], loc[0])){ // pastikan tetap di dalam Setu
      nodes[i].lat = loc[1];
      nodes[i].lng = loc[0];
    }
  }
}
```
**Penjelasan:** matriks `dist` (dipakai `routeLength` dan `pickNext`) diisi dari OSRM berupa
**jarak tempuh jalan asli** (bisa tidak simetris karena jalan satu arah). Jika layanan OSRM
tidak tersedia, `buildDistHaversine` menghitung jarak garis lurus sebagai cadangan. `snapCoords`
memindahkan marker tepat ke jalan dan menyimpan nama jalannya.

---

### Tabel: Parameter → Simbol → Baris Kode → Efek

| Parameter (slider) | Simbol | Baris kode | Efek |
|---|---|---|---|
| `alpha` | α | 435 | Bobot kepercayaan pada **feromon** (makin besar → makin ikut jalur ramai) |
| `beta` | β | 435 | Bobot kepercayaan pada **jarak** (makin besar → makin pilih yang dekat) |
| `evap` | ρ | 471 | **Laju penguapan** feromon (besar → cepat lupa jalur lama) |
| `Q` | Q | 478 | **Jumlah feromon** yang disimpan tiap rute (besar → feromon cepat menumpuk) |
| `ants` | — | 461 | **Jumlah semut** per iterasi (banyak → penjelajahan lebih luas) |
| `iters` | — | 497 | **Jumlah iterasi** (banyak → konvergensi lebih matang, lebih lambat) |

### Tabel: Rumus (Slide 6) → Baris Kode

| Rumus | Arti | Baris kode |
|---|---|---|
| `τ = 1` | Inisialisasi feromon | 422 |
| `P = τ^α · (1/d)^β` | Peluang pilih jalur berikutnya | 435 |
| `τ ← (1−ρ)·τ` | Evaporasi feromon | 471 |
| `τ ← τ + Q/L` | Deposit feromon oleh semut | 482–483 |
| `L = Σ d(i,j)` | Panjang rute total | 426 |
| `bestL← min(bestL, L)` | Simpan rute terbaik | 486–488 |

### Narasi Siap-Tempel untuk Laporan

> Pada setiap iterasi, beberapa semut membangun rute secara paralel. Semut mulai dari gudang
> dan memilih rumah berikutnya secara probabilistik: jalur yang memiliki feromon kuat
> (sering dilewati semut sebelumnya) dan jarak yang pendek memiliki peluang lebih besar untuk
> dipilih. Setelah semua semut selesai, feromon pada seluruh jalur menguap sebesar faktor ρ
> agar jalur yang kurang baik terlupakan, kemudian setiap semut menambahkan feromon pada jalur
> yang dilaluinya sebesar Q/L — sehingga rute yang lebih pendek menerima tambahan feromon lebih
> besar. Akibatnya, dari iterasi ke iterasi feromon terakumulasi pada jalur-jalur yang paling
> efisien, dan algoritma konvergen menuju rute terbaik. Jarak antar titik yang dipakai adalah
> jarak tempuh jalan asli (OSRM); bila layanan tidak tersedia, dipakai jarak garis lurus
> (Haversine) sebagai cadangan.