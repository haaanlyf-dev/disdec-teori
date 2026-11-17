Materi Minggu 3: Sinkronisasi dan Koordinasi pada Sistem Terdistribusi

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Buatkan ringkasan dari materi yang ada pada lampiran pdf, dengan penjelasan yang mudah dipahami tetapi sesuai dengan isi materi, berikan analogi singkat jika perlu dan berikan contoh agar lebih kebayang dari isi materi. untuk format sesuai dengan isi readme.md" (dengan lampiran PDF materi minggu 03)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Apa Itu Sinkronisasi?

**Sinkronisasi** = koordinasi **proses, data, dan sumber daya** di berbagai node agar operasinya **konsisten dan andal**.

Sinkronisasi membantu:
- Menjaga **konsistensi data**.
- Mengelola **proses yang berjalan bersamaan** (konkuren).
- Membuat perilaku sistem **koheren** di semua node.
- Mencegah **konflik data (data race)**.

### Contoh Race Condition di E-Commerce

| Langkah | User A | User B | Stok Produk X |
|---|---|---|---|
| 1 | Baca stok → 10 (aman) | | 10 |
| 2 | | Baca stok → 10 (aman) | 10 |
| 3 | Beli, stok = 10 − 1 | | **9** |
| 4 | | Beli, stok = 10 − 1 | **9** |

- **Kenyataan:** ada 2 pembelian, seharusnya stok = **8**.
- **Tercatat di software:** stok = **9**. Data jadi salah karena keduanya membaca nilai lama secara bersamaan.

> **Analogi:** Dua orang mengambil uang dari satu dompet bersama. Keduanya melihat isi dompet Rp100.000, masing-masing mengambil Rp10.000 dan mencatat "sisa Rp90.000". Padahal sisa sebenarnya Rp80.000.

## 2. Mengapa Sinkronisasi Penting?

- **Integritas data**: mencegah data race dan race condition.
- **Sinkronisasi state**: state aplikasi sama di semua node. Contoh:
  1. **Game multiplayer online**: posisi, *health*, dan inventaris pemain dikirim ke semua client agar game tetap konsisten.
  2. **Keranjang belanja web**: isi keranjang sama antara browser dan database, di berbagai perangkat dan sesi (buka di HP dan laptop isinya sama).
- **Koordinasi tasks**.
- **Pengelolaan resources**.
- **Mekanisme recovery**: antara lain dengan menyimpan (*maintain*) state.
- **Utilisasi efisien**: mencegah operasi yang dikerjakan dua kali (redundan).
- **Load balancing**.
- **Mencegah deadlock**.
- **Memungkinkan operasi yang skalabel**.

### Deadlock

**Deadlock** = kondisi ketika beberapa proses **saling menunggu sumber daya yang dipegang proses lain**, sehingga tidak ada yang bisa lanjut.

> **Analogi:** Dua mobil berhadapan di jembatan sempit; masing-masing menunggu yang lain mundur, akhirnya tidak ada yang bergerak.
>
> **Contoh:** Proses P1 memegang tabel `Order` dan menunggu tabel `Stok`, sementara P2 memegang `Stok` dan menunggu `Order`.

## 3. Tantangan dari Sinkronisasi

- **Network latency & partitioning**: pesan bisa terlambat atau jaringan terputus.
- **Skalabilitas**: makin banyak node, makin sulit disinkronkan.
- **Fault-tolerant system**: sinkronisasi harus tetap berjalan walau ada node yang gagal.
- **Concurrency control**: mengatur banyak proses yang mengakses data yang sama.
- **Data consistency**.
- **Sinkronisasi waktu**: jam tiap node bisa berbeda.

## 4. Sinkronisasi Waktu Fisik

**Pengertian:** menyelaraskan **jam nyata (real-time clock)** semua node dengan satu **standar waktu umum**.

**Kenapa penting?**
- **Korelasi log**: mencocokkan log dari banyak server saat *debugging*.
- **Penjadwalan event**: misalnya backup otomatis jam 00:00 di semua server.
- **Menjaga konsistensi sistem**.

Dilakukan dengan **protokol** dan/atau sumber waktu berbasis hardware: **atomic clock** dan/atau **GPS** (yang tersinkron dengan atomic clock).

### Standar Waktu

| Standar | Keterangan |
|---|---|
| **GMT** (Greenwich Mean Time) | Sudah tidak digunakan sebagai standar. |
| **UTC** (Universal Coordinated Time) | Didefinisikan oleh **ITU**, standar sejak **1963**. Berbasis TAI. |
| **TAI** (*Temps Atomique International*) | Waktu atom internasional, dasar dari UTC. |
| **LTC** (Coordinated Lunar Time) | Waktu standar untuk bulan, **masih dikembangkan**. |

### Rantai Sinkronisasi Waktu

```
Master Atomic Clock → Atomic Clock di satelit GPS → Sinyal waktu GPS
→ GPS Dongle → Server Linux (NTP/PTP server)
→ Client (Linux, Windows, Mac, HP, dll.)
```

> **Analogi:** Seperti estafet. Jam paling akurat di "pusat" diteruskan bertahap sampai ke HP kita.

**Lokasi sumber atomic clock** (sekitar 30 satelit membawa atomic clock):
- **Satelit GPS**: 31 satelit di *Medium Earth Orbit*, dioperasikan U.S. Space Force.
- **Deep Space Atomic Clock (DSAC)**: diluncurkan NASA tahun 2019 di orbit rendah untuk uji stabilitas.
- **ISS (misi ACES)**: menempatkan *cold-atom clock* di orbit rendah bumi.
- **Ground Control**: pengelolaan master clock di darat, yaitu U.S. Naval Observatory (Washington, D.C.) dan Schriever Space Force Base (Colorado).

Di Linux, GPS dongle biasanya terbaca sebagai perangkat serial seperti `/dev/ttyACM0`.

### Protokol Sinkronisasi Waktu

**1. NTP (Network Time Protocol)**
- Protokol paling umum untuk sinkronisasi jam lewat internet/jaringan.
- Tersusun bertingkat (*stratum*): stratum 0 = sumber waktu (atomic clock/GPS), stratum 1 = server yang terhubung langsung ke sumber, stratum 2 = server yang sinkron ke stratum 1, dan seterusnya.
- Client menghitung selisih waktu (*offset*) dan lama perjalanan pesan (*delay*) dengan server, lalu menyesuaikan jamnya.
- Akurasi umumnya dalam orde **milidetik**.

> **Contoh:** Windows/HP kita otomatis menyetel jam dengan sinkronisasi ke server waktu seperti `time.windows.com` atau `pool.ntp.org`.

**2. PTP (Precision Time Protocol)**
- **Lebih presisi** daripada NTP (bisa sampai orde **sub-mikrodetik**).
- Standar **IEEE 1588**:
  - IEEE 1588-2002: versi awal.
  - IEEE 1588-2008 (**PTP v2**): **tidak backward compatible** dengan 2002, memperkenalkan konsep *profile*.
  - IEEE 1588-2019: perbaikan yang **backward compatible** dengan versi 2008.
  - **IEEE 802.1AS (gPTP)**: adaptasi PTP untuk *Audio Video Bridging* (AVB) dan *Time-Sensitive Networking* (TSN).
- Implementasi di Linux: **linuxptp** (`https://linuxptp.nwtime.org/`).
- **Syarat:** NIC (kartu jaringan) harus mendukung **hardware timestamping**.

> **Contoh penggunaan PTP:** bursa saham (transaksi dicatat per mikrodetik), jaringan telekomunikasi 5G, dan sistem audio/video profesional.

## 5. Sinkronisasi Waktu Logis

- Metode untuk menentukan **urutan sebab-akibat (causal order) kejadian**, **tidak terkait** dengan waktu fisik dunia nyata.
- Berlaku sifat **transitif**: jika **A terjadi sebelum B** dan **B sebelum C**, maka **A terjadi sebelum C**.
- Dasarnya adalah **Lamport Logical Clock** (lihat materi minggu 2).

### Vector Clocks

Lamport clock hanya memakai **1 angka**, sehingga tidak bisa membedakan apakah dua kejadian **berurutan** atau **terjadi sendiri-sendiri (concurrent)**. **Vector clock** memakai **array angka**, satu "spot" untuk setiap proses.

**Kondisi awal (3 proses: A di index 0, B di index 1, C di index 2):**
```
A: [0, 0, 0]
B: [0, 0, 0]
C: [0, 0, 0]
```

**Aturan:**
- Setiap ada event (internal event atau kirim pesan), tambahkan **1 pada spot milik sendiri**.
- Saat menerima pesan, ambil nilai **maksimum per elemen** (*element-wise max*) antara clock sendiri dan clock di pesan.

**Contoh langkah (dari materi):**

| Langkah | Kejadian | A | B | C |
|---|---|---|---|---|
| 1 | A melakukan internal event | [1,0,0] | [0,0,0] | [0,0,0] |
| 2 | A menambah clock lalu mengirim pesan ke B | [2,0,0] | | |
| 3 | B menerima, ambil max([2,0,0], [0,0,0]) | | [2,0,0] | |
| 4 | B menambah clock lalu mengirim pesan ke C | | [2,1,0] | |
| 5 | C menerima: max = [2,1,0], lalu internal event → +1 | | | [2,1,1] |
| 6 | C melakukan internal event lagi | | | [2,1,2] |

**Posisi akhir:** A = [2,0,0], B = [2,1,0], C = [2,1,2].

**Cara membandingkan:**
- **X terjadi sebelum Y** jika **semua angka X ≤ angka Y** dan **minimal satu lebih kecil**.
- **Concurrent (independen)** jika tidak semua komponen ≤ (ada yang lebih besar, ada yang lebih kecil).

| Perbandingan | Hasil |
|---|---|
| B [0,0,0] vs A [1,0,0] | B [0,0,0] terjadi **lebih dulu** |
| A [2,0,0] vs B [2,1,0] | A terjadi **lebih dulu** |
| C [2,1,2] vs A [2,0,0] | Menurut materi: **independen** |

> **Catatan:** Jika aturan di atas diterapkan, [2,0,0] ≤ [2,1,2] di semua elemen, sehingga A sebenarnya terjadi **sebelum** C (masuk akal, karena pesan A diteruskan lewat B ke C). Contoh event yang benar-benar *concurrent* misalnya [1,0,0] vs [0,0,1]. Sebaiknya dikonfirmasi ke dosen.

> **Analogi:** Vector clock seperti **buku catatan bersama** di mana tiap anggota kelompok punya kolom sendiri. Dari catatan itu bisa diketahui siapa sudah "tahu" kabar dari siapa.
>
> **Contoh:** Google Docs atau database terdistribusi (mis. Amazon Dynamo) memakai konsep serupa untuk mendeteksi apakah dua edit saling bertabrakan (*concurrent*) atau berurutan.

Implementasi kode dapat dilihat pada file `vclocks.py` dari materi.

## 6. Tantangan Mutual Exclusion

**Mutual exclusion (mutex)** = prinsip yang memastikan **hanya satu proses/thread** yang boleh mengakses **sumber daya bersama** (printer, file, database) **dalam satu waktu**, demi mencegah konflik dan menjaga integritas data.

- **Critical Section (CS):** bagian kode yang mengakses sumber daya bersama. Tujuan mutex: **eksekusi CS tidak boleh tumpang-tindih**.
- Jika lebih dari satu proses masuk bersamaan → **inkonsistensi atau kerusakan data**.

> **Analogi:** Toilet umum dengan satu kunci. Hanya satu orang boleh masuk; yang lain menunggu sampai pintu dibuka.

**Prinsip yang harus dipenuhi:**

| Prinsip | Arti |
|---|---|
| **Mutual Exclusion** | Hanya 1 proses yang bekerja di sumber daya bersama. |
| **Progress** | Jika tidak ada yang sedang memakai CS, proses lain boleh masuk (tidak dibiarkan kosong tanpa alasan). |
| **Bounded Waiting** | Tidak ada proses yang menunggu **selamanya**. |
| **Fairness** | Permintaan dilayani dalam urutan yang wajar. |

### Pendekatan Solusi

**1. Centralized Algorithm**
- Ada **satu node koordinator**.
- Alur: proses **minta izin** → koordinator **memberi akses** jika CS kosong → setelah selesai, proses melepas, lalu koordinator mengizinkan antrean berikutnya.
- ✅ Sederhana, biaya rendah.
- ❌ **Single point of failure**, rawan **bottleneck**.

> **Analogi:** Satpam yang memegang kunci ruang rapat. Kalau satpamnya tidak ada, tidak ada yang bisa masuk.

**2. Ricart–Agrawala**
- **Terdistribusi penuh** (tanpa koordinator).
- Proses yang ingin masuk CS mengirim **permintaan bertimestamp** ke semua proses lain.
- Proses boleh masuk CS setelah **semua proses lain menjawab OK**.
- Butuh **2 × (N − 1) pesan** per CS: N−1 permintaan dan N−1 balasan.
- ✅ Fault tolerant (tidak bergantung pada satu koordinator).
- ❌ **Overhead pesan tinggi**.

> **Contoh:** Dengan 5 proses, satu kali masuk CS butuh 2 × (5 − 1) = **8 pesan**.
>
> **Analogi:** Ingin memakai ruang bersama, kita harus minta izin ke **semua** teman satu kos dan menunggu semuanya bilang "boleh".

**3. Token-Based Algorithm**
- Ada **satu token unik** yang beredar di antara proses.
- Hanya proses yang **memegang token** yang boleh masuk CS.
- Jika token hilang → perlu **mekanisme pemulihan**.
- ✅ Hanya **1 pesan** per entri CS.
- ❌ **Sulit mendeteksi** token yang hilang.

> **Analogi:** Permainan "tongkat estafet" atau mikrofon dalam diskusi: hanya yang memegang mikrofon yang boleh bicara.

## 7. Pemilihan Koordinator (Leader Election)

Beberapa node membutuhkan **satu koordinator/pemimpin** untuk:
- Mengelola sumber daya bersama.
- Menyinkronkan operasi.
- Membuat keputusan global.

Jika **koordinator gagal** atau **node baru bergabung**, sistem harus **memilih pemimpin baru secara otomatis**.

**Algoritma:**

| Algoritma | Cara Kerja Singkat | Analogi |
|---|---|---|
| **Bully Algorithm** | Node yang mendeteksi koordinator mati mengirim pesan pemilihan ke node dengan ID lebih besar. Node dengan **ID tertinggi** yang masih hidup menjadi koordinator. | Yang "paling senior" otomatis jadi ketua. |
| **Ring Algorithm** | Node tersusun dalam **cincin**; pesan pemilihan diedarkan berkeliling sambil mengumpulkan ID, lalu ID tertinggi dipilih. | Kertas absen diedarkan keliling kelas, lalu dipilih nomor terbesar. |
| **Raft / Paxos** | Berbasis **konsensus**: kandidat menjadi pemimpin jika mendapat **suara mayoritas**. | Pemilihan ketua kelas dengan voting suara terbanyak. |

**Contoh nyata:**
- **Apache ZooKeeper** memakai **Zab** (ZooKeeper Atomic Broadcast, mirip Paxos).
- **Kubernetes** memakai **Raft** melalui **etcd**.

> **Contoh:** Pada cluster database 3 node, jika node *leader* mati, dua node lain melakukan voting dan dalam hitungan detik memilih leader baru, sehingga aplikasi tetap berjalan.
