Materi Minggu 1: Pengenalan Sistem Terdistribusi dan Terdesentralisasi

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Buatkan ringkasan dari materi yang ada pada lampiran pdf, dengan penjelasan yang mudah dipahami tetapi sesuai dengan isi materi, berikan analogi singkat jika perlu dan berikan contoh agar lebih kebayang dari isi materi. untuk format sesuai dengan isi readme.md" (dengan lampiran PDF materi minggu 01)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Kenapa Sistem Terdistribusi & Terdesentralisasi Penting?

- Jumlah **data dan pengguna** sekarang sangat besar.
- Layanan modern harus **skalabel**, **fault-tolerant**, dan **tersedia 24/7** dengan performa tinggi.
- Tanpa kedua konsep ini, layanan seperti **Google, Netflix, dan Blockchain** tidak mungkin ada.

Dua istilah kunci:

| Istilah | Arti | Contoh |
|---|---|---|
| **Skalabilitas** | Kemampuan sistem berkembang menangani beban, pengguna, atau permintaan yang makin banyak **tanpa menurunkan kinerja/kualitas**. | Saat Harbolnas, toko online menambah server supaya tetap lancar. |
| **Fault-Tolerant** | Sistem tetap berjalan normal walau **sebagian komponen (hardware/software) rusak**. | Satu server Netflix mati, film tetap bisa diputar dari server lain. |

> **Analogi:** Restoran yang skalabel bisa menambah koki saat ramai. Restoran yang fault-tolerant tetap buka walau satu koki sakit.

## 2. Pengertian Sistem Terdistribusi

Menurut Tanenbaum & van Steen (2007):

> *"A collection of independent computers that appears to its users as a single coherent system."*

Artinya: **sekumpulan komputer independen yang di mata pengguna terlihat seperti satu sistem utuh.**

Ciri utamanya:
- **Komponen independen**: tiap komputer punya CPU, memori, dan OS sendiri.
- **Berkomunikasi lewat jaringan**.
- **Transparansi**: pengguna tidak perlu tahu detail di baliknya, yaitu:
  - **Lokasi**: tidak tahu data ada di server mana.
  - **Akses**: cara akses sama walau sumbernya beda.
  - **Replikasi**: tidak tahu data disalin ke banyak tempat.
  - **Kegagalan**: tidak sadar ada server yang mati.

> **Analogi:** Pelanggan minimarket waralaba merasa belanja di "satu toko", padahal stok, gudang, dan kasirnya tersebar di banyak tempat.
>
> **Contoh:** Saat membuka Google Drive, kita tidak tahu file kita disimpan di data center mana. Yang kita lihat cuma satu "Drive".

## 3. Arsitektur Sistem Dasar

| Arsitektur | Cara Kerja | Contoh |
|---|---|---|
| **Client-Server** | Client meminta layanan, server pusat yang melayani. | Browser (client) membuka website di web server. |
| **Peer-to-Peer (P2P)** | Semua node setara, masing-masing bisa jadi client sekaligus server. | BitTorrent: tiap pengguna sekaligus mengunduh dan membagikan file. |
| **Hybrid** | Gabungan keduanya: ada server pusat untuk koordinasi (mis. indeks/pencarian), lalu pertukaran data terjadi langsung antar-peer. | Aplikasi berbagi file yang memakai server pusat untuk mencari file, tetapi unduhannya langsung antar pengguna. |

> **Analogi:**
> - Client-Server = pelanggan dan pelayan restoran.
> - P2P = arisan atau barter antar tetangga, semua saling memberi dan menerima.
> - Hybrid = marketplace: aplikasinya mempertemukan, transaksinya langsung antara penjual dan pembeli.

## 4. Karakteristik Sistem Terdistribusi

1. **Transparansi lokasi**: pengguna tidak perlu tahu lokasi fisik sumber daya.
2. **Skalabilitas**: mudah ditambah node saat beban naik.
3. **Fault tolerance**: tetap jalan walau ada node yang gagal.
4. **Kinerja tinggi**: pekerjaan dibagi ke banyak mesin sehingga lebih cepat.
5. **Heterogenitas**: bisa terdiri dari hardware, OS, dan bahasa pemrograman yang berbeda-beda.

## 5. Pengertian Sistem Terdesentralisasi

- **Sentralisasi**: hanya ada **1 titik kontrol**. Kalau pusat mati, semua ikut mati.
- **Desentralisasi**: kontrol **dibagi ke banyak pihak**, sehingga **tidak ada single point of failure**.
- Keputusan diambil **secara kolektif melalui konsensus** (kesepakatan bersama).
- Contoh: **Blockchain, DNS, BitTorrent**.

> **Analogi:**
> - Sentralisasi = kerajaan: semua keputusan di tangan raja.
> - Desentralisasi = musyawarah desa: keputusan diambil bersama, tidak bergantung pada satu orang.
>
> **Contoh:** Pada Bitcoin, tidak ada bank pusat yang mengesahkan transaksi. Ribuan node di seluruh dunia bersepakat (konsensus) transaksi mana yang sah.

## 6. Sistem Terdistribusi vs Sistem Terdesentralisasi

| Aspek | Terdistribusi | Terdesentralisasi |
|---|---|---|
| Fokus | Pembagian **beban kerja** | Pembagian **kontrol** |
| Kepemilikan/kendali | Bisa tetap dikendalikan **satu pihak** | Kendali **tersebar** ke banyak pihak |
| Contoh | Google File System, MinIO Object Storage (sekarang di-fork ke Silo) | Blockchain |

> **Analogi:** Satu perusahaan punya banyak cabang. Pekerjaannya tersebar (terdistribusi), tetapi keputusannya tetap dari kantor pusat (tidak terdesentralisasi). Koperasi yang dimiliki dan diputuskan bersama oleh anggotanya, itu desentralisasi.
>
> **Contoh:** Google punya ribuan server di seluruh dunia (terdistribusi), tetapi semuanya tetap dikendalikan Google (tersentralisasi kontrolnya).

## 7. Studi Kasus

**Sistem Terdistribusi:**
- **Google File System / HDFS**: penyimpanan data berukuran sangat besar.
- **Apache Kafka & Apache Pulsar**: messaging & streaming data.
- **Netflix, Twitch, YouTube**: streaming video ke jutaan pengguna.
- **Instagram, Threads, Facebook, TikTok**: media sosial skala global.

**Sistem Terdesentralisasi:**
- **Blockchain**: Bitcoin, Ethereum, Algorand, Sui, Near, Aptos, dll.
- **DNS (Domain Name System)**: tidak ada satu server yang tahu semua domain; tugasnya dibagi ke banyak server bertingkat.
- **BitTorrent**: file dibagikan antar pengguna tanpa server pusat.

## 8. CAP Theorem (Brewer's Theorem, 2000)

Dikemukakan oleh **Eric Brewer**. Ada 3 sifat:

| Huruf | Sifat | Arti |
|---|---|---|
| **C** | Consistency | Setiap operasi baca mengembalikan **data tulisan terbaru**, atau **error**. |
| **A** | Availability | Setiap permintaan ke node yang tidak gagal **selalu dijawab tanpa error**, walau datanya **mungkin bukan yang terbaru**. |
| **P** | Partition Tolerance | Sistem **tetap berjalan** walau komunikasi antar-node **terputus/terpecah**. |

Pada *distributed data store*, **partition pasti bisa terjadi**, jadi tidak mungkin mendapatkan C dan A sekaligus. Saat partition terjadi, perancang harus memilih:

- **AP (Availability + Partition Tolerance)**: permintaan **selalu dilayani**, dengan risiko data **tidak sinkron** (mengorbankan konsistensi). Contoh menurut materi: MongoDB.
- **CP (Consistency + Partition Tolerance)**: permintaan (read/write) **ditolak** demi menjaga data tetap konsisten (mengorbankan availability). Contoh menurut materi: Apache Cassandra.

Setelah partition selesai, sistem bisa melakukan *auto healing* (menyinkronkan kembali).

> **Analogi:** Dua kasir toko yang memakai buku stok masing-masing, dan telepon antar-kasir putus (partition).
> - **Pilih AP:** kasir tetap melayani pembeli, dengan risiko stok tercatat beda di kedua buku.
> - **Pilih CP:** kasir berhenti melayani dulu sampai telepon tersambung lagi, supaya stok tetap benar.

> **Catatan:** Di banyak literatur lain, contoh ini sering disebut terbalik: Cassandra umumnya digolongkan **AP** dan MongoDB (konfigurasi default) **CP**. Untuk ujian, ikuti versi materi kuliah, tetapi sebaiknya dikonfirmasi ke dosen.

## 9. Konsistensi Data

| Jenis | Arti | Cocok untuk |
|---|---|---|
| **Strong Consistency** | Hasil baca **selalu sama/terbaru** di semua node. | **Bank**: saldo harus tepat, tidak boleh beda. |
| **Eventual Consistency** | Awalnya bisa berbeda, tetapi **lama-lama akan konsisten**. | **Media sosial**: jumlah *like* telat beberapa detik tidak masalah. |

> **Contoh:** Setelah transfer Rp100.000, saldo di ATM dan m-banking harus langsung sama (strong). Tetapi jumlah *like* foto Instagram yang dilihat teman bisa berbeda sebentar sebelum akhirnya sama (eventual).

## 10. Tantangan

- **Latency & throughput**
  - *Latency*: **seberapa lama** satu permintaan diproses.
  - *Throughput*: **seberapa banyak** permintaan yang bisa diproses per satuan waktu.
- **Keamanan**: serangan **DDoS** dan **Sybil attack**.
- **Kompleksitas manajemen node**: makin banyak node, makin sulit dikelola.
- **Governance (tata kelola)** pada sistem terdesentralisasi: siapa yang berhak memutuskan perubahan?

> **Analogi latency vs throughput:** Jalan tol. *Latency* = berapa lama satu mobil sampai tujuan. *Throughput* = berapa banyak mobil yang bisa lewat per jam.

**Sybil Attack**: penyerang membuat **banyak identitas palsu** (akun, node, pseudonim, bisa sampai ribuan) sekaligus untuk **mendominasi suara, pengaruh, atau reputasi** dalam jaringan. Biasanya menyerang sistem terdesentralisasi seperti P2P dan blockchain. Dampaknya: memanipulasi sistem, merusak integritas, melakukan tindakan tidak sah, hingga mengakses informasi sensitif. Nama "Sybil" diambil dari buku tahun 1973 tentang perempuan dengan *identity disorder*.

> **Contoh:** Seseorang membuat 1.000 akun palsu untuk memenangkan voting online, atau menjalankan ribuan node palsu agar menguasai keputusan di jaringan blockchain.

## 11. Tren Masa Depan

- **Cloud-native**: Kubernetes, Docker.
- **Edge & Fog Computing**: pemrosesan dilakukan lebih dekat ke pengguna/perangkat (contoh: CCTV pintar yang menganalisis video langsung di kamera).
- **Web3 & DeFi**: internet dan layanan keuangan berbasis blockchain tanpa perantara.
- **Federated Data & Learning (AI terdistribusi)**: model AI dilatih di banyak perangkat tanpa mengumpulkan data mentahnya ke pusat (contoh: keyboard HP yang belajar prediksi kata langsung di HP pengguna).

## 12. Ringkasan

- **Sistem terdistribusi = berbagi beban kerja.**
- **Sistem terdesentralisasi = berbagi kontrol.**
- Keduanya saling terkait tetapi berbeda fokus.
- Keduanya menjadi fondasi teknologi modern.

## 13. Diskusi

**1. Mengapa dunia semakin bergerak menuju distribusi & desentralisasi?**

- **Data dan pengguna tumbuh sangat besar**: satu server tidak sanggup lagi menanganinya, jadi beban kerja harus dibagi ke banyak mesin (distribusi).
- **Butuh skalabilitas**: saat pengguna melonjak, cukup menambah node.
- **Harus fault-tolerant dan tersedia 24/7**: kalau satu server mati, layanan tetap jalan.
- **Menghindari single point of failure dan ketergantungan pada satu pihak**: kontrol dibagi agar tidak ada pusat yang bisa gagal atau menyalahgunakan kuasa (desentralisasi).

**2. Contoh sehari-hari yang paling terasa:**

- **Netflix/YouTube**: video dikirim dari server terdekat, sehingga tetap lancar walau ditonton jutaan orang (terdistribusi).
- **Ojek online**: aplikasi memecah tugas ke banyak service, yaitu driver, lokasi, dan tarif (terdistribusi).
- **Media sosial**: jumlah *like* boleh telat sebentar (eventual consistency).
- **Transfer bank**: saldo harus langsung tepat (strong consistency).
- **Dompet kripto**: transaksi disahkan bersama oleh banyak node tanpa bank pusat (terdesentralisasi).
