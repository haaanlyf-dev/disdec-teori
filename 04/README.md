Materi Minggu 4: Konsistensi dan Replikasi pada Sistem Terdistribusi

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Berdasarkan Materi Konsistensi dan Replikasi pada Sistem Terdistribusi yang ada di lampiran pdf(04), buatkan ringkasan materi tersebut agar mudah dipahami dan berikan logika jika memang diperlukan, berikan juga pertanyaan 5w 1h pada materi ini agar tau kegunaanya, untuk pertanyaan diskusi di jawab." (dengan lampiran: slide `04--konsistensi-dan-replikasi.pdf` dan folder `Lecture Notes-20261117` yang berisi panduan *Streaming Replication di PostgreSQL 18*, *Apache Ignite 3 Cluster*, serta source code-nya)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Apa Itu Konsistensi pada Sistem Terdistribusi?

**Konsistensi** = jaminan bahwa **semua node memiliki data yang sama (atau tampak sama)** pada waktu tertentu, walaupun data tersebar di banyak tempat dan bisa diubah bersamaan oleh banyak pengguna/proses.

Bayangkan satu aplikasi punya beberapa **salinan data** di server yang lokasinya berbeda. Jika data diubah di satu server, sistem harus memastikan server lain **akhirnya punya data yang sama**. Itulah inti konsistensi.

> **Analogi:** Satu grup punya beberapa salinan daftar piket. Kalau satu orang mengubah jadwal di salinannya, semua salinan lain harus ikut diubah. Kalau tidak, ada yang datang di hari yang salah.

## 2. Mengapa Konsistensi Penting?

Tanpa konsistensi, pengguna bisa melihat **versi data yang berbeda** tergantung server mana yang ia akses.

**Contoh dari materi (transfer bank):**
- Nasabah mentransfer uang dari rekening A ke B.
- Server 1 **sudah mengurangi** saldo A, tetapi server 2 **belum menambah** saldo B.
- Nasabah mencoba bertransaksi dari rekening B → **saldo tidak mencukupi**.
- Artinya sistem **tidak konsisten**: uangnya seolah "hilang di tengah jalan".

Intinya:
- Data disimpan di banyak node, sehingga **berpotensi berbeda** antar-node.
- **Tujuan:** pengguna melihat data yang **benar dan up-to-date**.
- Selalu ada **trade-off** antara Consistency, Availability, dan Partition Tolerance (**CAP Theorem**).

## 3. Teorema CAP

| Huruf | Arti |
|---|---|
| **C**onsistency | Semua node mempunyai data yang sama. |
| **A**vailability | Data tersedia saat pengguna membutuhkannya. |
| **P**artition Tolerance | Sistem tetap berjalan walau ada komponen jaringan yang gagal/terputus. |

**Partisi (koneksi putus) itu hal yang wajar.** Sebagus apa pun jaringannya, partisi tetap akan terjadi. Karena itu kondisi ideal C + A + P sekaligus **tidak mungkin** dicapai.

**Logika:** Saat jaringan antar-server putus, server yang menerima permintaan hanya punya 2 pilihan:
1. **Tetap menjawab** dengan data yang ia punya (mungkin data lama) → memilih **A**, mengorbankan **C**.
2. **Menolak/menunggu** sampai bisa memastikan data terbaru → memilih **C**, mengorbankan **A**.

Tidak ada pilihan ketiga, karena server tidak bisa tahu data di seberang selama jaringan terputus.

**Contoh dari materi (e-Commerce):**
- Server A melayani Indonesia barat, Server B melayani Indonesia timur. Jaringan antara keduanya **putus**.
- **Pilih AP:** pengguna di kedua wilayah tetap bisa belanja, tetapi **stok bisa tidak sinkron** sementara (bisa terjadi barang terjual melebihi stok).
- **Pilih CP:** sistem **menolak transaksi** sampai jaringan pulih, agar stok tetap konsisten (mengorbankan availability).

## 4. Model Konsistensi

**Model konsistensi** = aturan/jaminan tentang **bagaimana dan kapan** perubahan data terlihat oleh node atau klien lain, termasuk **seberapa cepat** semua salinan menjadi sama setelah ada update.

Diurutkan kira-kira dari yang **paling kuat** ke **paling lemah**:

| No | Model | Jaminan | Contoh |
|---|---|---|---|
| 7 | **Linearizability** | Paling kuat. Sistem terlihat seperti **satu node tunggal**: setelah write selesai, **semua** read (oleh klien mana pun) pasti melihatnya, dengan urutan global **real-time**. | Cek keunikan **username**: dua orang tidak bisa mendaftar username yang sama bersamaan. Dipakai di **etcd, ZooKeeper, Spanner**. |
| 1 | **Strong Consistency** | Setelah update, **semua node langsung melihat versi terbaru**. Biasanya memakai **Two-Phase Commit** atau **Paxos**. | **Perbankan**: saldo berubah, semua server harus segera tahu. |
| 6 | **Sequential Consistency** | Semua node melihat **urutan operasi yang sama**, tetapi **tidak harus real-time** (boleh ada delay). | Semua pengguna melihat **urutan posting yang sama**, walau ada sedikit delay. |
| 3 | **Causal Consistency** | Operasi yang **saling bergantung** (sebab-akibat) harus terlihat **berurutan**. Operasi yang tidak berhubungan boleh terlihat beda urutan. | **Komentar tidak boleh muncul sebelum postingannya.** |
| 4 | **Read-Your-Writes** | Setelah **saya** menulis, **saya** pasti melihat tulisan saya. Orang lain belum tentu melihatnya. | Pesan yang kita kirim langsung muncul di layar sendiri walau penerima belum melihatnya. |
| 5 | **Monotonic Read** | Kalau saya sudah melihat versi baru, saya **tidak akan melihat versi yang lebih lama** lagi. | Timeline media sosial **tidak "mundur"** ke versi lama setelah di-refresh. |
| 2 | **Eventual Consistency** | Paling lemah. Data **tidak langsung sama**, tetapi **akhirnya akan sama** jika tidak ada update baru. | **Amazon DynamoDB, Apache Cassandra**: update telat menyebar, tetapi akhirnya konsisten. |

*(Kolom "No" mengikuti nomor urut di slide.)*

> **Analogi Linearizability vs Sequential:** Linearizability seperti **siaran langsung**: semua orang melihat kejadian yang sama, saat itu juga. Sequential seperti **siaran tunda**: semua orang melihat urutan kejadian yang sama, tapi ada yang telat beberapa detik.

### BASE (pasangan Eventual Consistency)

| Huruf | Arti |
|---|---|
| **B**asically **A**vailable | Data tetap bisa dibaca/ditulis walau ada kegagalan sebagian atau jaringan terpisah. |
| **S**oft State | Replika **tidak harus konsisten setiap saat**; keadaan sistem bisa berubah seiring waktu. |
| **E**ventual Consistency | Pada akhirnya data akan konsisten di semua node jika tidak ada update baru. |

> **Logika BASE vs ACID:** Database tradisional memakai ACID (ketat, "harus benar sekarang"). BASE kebalikannya: "boleh belum benar sekarang, yang penting tetap melayani, nanti juga benar". BASE cocok untuk sistem yang mengutamakan **availability dan kecepatan**.

## 5. Trade-Off Latensi vs Konsistensi

**Semakin tinggi konsistensi yang dijamin, semakin besar latensi (waktu respon).**

```
Konsistensi ↑  →  Latensi ↑
Konsistensi ↓  →  Latensi ↓
```

| | Strong Consistency | Eventual Consistency |
|---|---|---|
| Cara kerja | **Menunggu** semua node sinkron dulu, baru menjawab | **Langsung menjawab**, sinkronisasi menyusul |
| Latensi | **Tinggi** (harus komunikasi lintas jaringan) | **Rendah** |
| Risiko | Lambat | Data mungkin **belum terbaru** |

**Contoh dari materi:** dua pusat data, **Jakarta** dan **Yogyakarta**. Pengguna mengubah data di Jakarta, lalu ada permintaan baca.
- **Strong:** tunggu sampai Jakarta dan Yogyakarta **sama persis**, baru menjawab. Data konsisten, respon lambat.
- **Eventual:** langsung jawab dari Jakarta atau Yogyakarta; update ke Yogyakarta menyusul. Respon cepat, data bisa berbeda sementara.

> **Logika:** Data harus "berjalan" lewat jaringan. Kalau sistem wajib menunggu konfirmasi dari Yogyakarta, waktu respon minimal = waktu bolak-balik Jakarta↔Yogyakarta. Itulah harga yang dibayar untuk konsistensi.

## 6. Pengertian Replikasi Data

**Replikasi data** = proses **membuat dan memelihara salinan data yang sama** di beberapa node/server.

**Contoh dari materi:** database utama di **Jakarta**, lalu untuk *high availability* ditambah salinan di **Yogyakarta** dan **Denpasar**. Menjaga agar ketiga salinan tetap sama adalah inti replikasi.

**Tujuan replikasi:**

| Tujuan | Penjelasan |
|---|---|
| **Availability** | Satu server gagal, server lain masih punya salinan. |
| **Performance** | Pengguna membaca dari server **terdekat**, jadi lebih cepat. |
| **Reliability** | Risiko **kehilangan data** turun karena ada cadangan di banyak tempat. |
| **Skalabilitas** | Beban **baca** bisa dibagi ke beberapa server. |

> **Hubungan konsistensi dan replikasi:** Replikasi **menciptakan** salinan, sedangkan konsistensi adalah **masalah** yang muncul karena salinan itu harus tetap sama. Tanpa replikasi tidak ada masalah konsistensi; dengan replikasi, kita harus memilih model konsistensinya.

## 7. Jenis Replikasi

| Jenis | Cara Kerja | Kelebihan / Kekurangan | Contoh |
|---|---|---|---|
| **Master–Slave** (Primary–Secondary) | **Satu master** menerima write; slave **hanya untuk read**. | ✅ Konsistensi kuat ❌ **Bottleneck** di master | PostgreSQL streaming replication |
| **Multi-Master** (Peer-to-Peer) | **Semua node** bisa read dan write. | ✅ Fleksibel ❌ **Rawan konflik** data | PostgreSQL logical replication, Cassandra, MongoDB, Apache Ignite |
| **Synchronous** | Update ditulis ke **semua node** sebelum dianggap selesai. | ✅ Konsisten ❌ Lambat | Google Spanner |
| **Asynchronous** | Update dikirim ke node lain **setelah** transaksi selesai di master. | ✅ Cepat ❌ Bisa ada **delay** data | MySQL async mode, DynamoDB |

> **Analogi:**
> - **Master–Slave** = guru menulis di papan, murid hanya menyalin. Tidak ada yang bentrok, tapi semua bergantung pada satu guru.
> - **Multi-Master** = kerja kelompok di satu dokumen, semua boleh menulis. Cepat, tapi bisa ada yang saling menimpa.

## 8. Strategi Sinkronisasi Replika

Strategi bisa dikelompokkan dari 4 sudut pandang:

| Berdasarkan | Pilihan |
|---|---|
| **Waktu sinkronisasi** | Synchronous, asynchronous, semi-synchronous |
| **Arah replikasi** | Master-slave (read-write → read), master-master (semua read-write, **peran & kemampuan setara**), master-master asymmetric (semua read-write, **peran & kemampuan berbeda**) |
| **Cara sinkronisasi** | Pull-based, push-based, hybrid |
| **Penanganan konflik** | LWW, Vector Clock, CRDT, Operational Transformation |

### 8.1 Waktu Sinkronisasi

| Mode | Sumber menunggu... | Konsistensi | Performa | Risiko hilang data |
|---|---|---|---|---|
| **Asynchronous** | Tidak menunggu siapa pun | Terendah | **Tercepat** | **Tertinggi** |
| **Semi-synchronous** | **Satu** replika menerima data | Seimbang | Seimbang | Sedang. Dipakai **MySQL**. |
| **Synchronous** | **Semua** replika commit | **Tertinggi** | Terendah | Terendah |

> **Logika risiko hilang data:** Pada asynchronous, master bilang "sukses" sebelum replika menerima data. Jika master langsung mati detik itu juga, data terakhir **belum sempat tersalin** ke mana pun, dan hilang. Semi-synchronous mengurangi risiko ini karena minimal ada **1 salinan lain** sebelum dinyatakan sukses.

### 8.2 Cara Sinkronisasi

| Cara | Penjelasan | Analogi |
|---|---|---|
| **Pull-based** | Replika **mengecek berkala** ke sumber: ada update → ambil, tidak ada → biarkan. | Kita membuka web pengumuman kampus tiap pagi untuk mengecek info baru. |
| **Push-based** | Sumber **aktif mengirim** update saat terjadi perubahan, memakai **CDC (Change Data Capture)** dan dikirim lewat **WebSockets** atau **SSE (Server-Sent Events)**. | Notifikasi WhatsApp: pesan datang sendiri tanpa kita cek. |
| **Hybrid** | Campuran keduanya. | |

### 8.3 Penanganan Konflik (Jika Ada Banyak Update Sekaligus)

| Metode | Cara Kerja |
|---|---|
| **LWW (Last Write Wins)** | Yang **timestamp-nya paling baru** yang dipakai, sisanya diabaikan. Sederhana, tetapi update lain bisa **hilang**. |
| **Vector Clock** | Urutan ditentukan dengan vector clock (lihat minggu 3), sehingga bisa membedakan update yang berurutan dan yang **concurrent** (konflik sungguhan). |
| **CRDT** (Conflict-free Replicated Data Type) | Struktur data **di tiap node** (bukan di server pusat) yang **dirancang agar selalu bisa digabung tanpa konflik**. Contoh: **Yjs**, dipakai untuk editor kolaboratif seperti Google Docs. |
| **OT** (Operational Transformation) | Operasi dari pihak lain **ditransformasi** (disesuaikan) berdasarkan informasi lokal sebelum diterapkan. |

**Contoh CRDT (dari materi):** Agnes Monica dan Bambang menghitung jumlah pengunjung website.

| Tahap | Map Agnes | Map Bambang |
|---|---|---|
| Awal | {Agnes: 0, Bambang: 0} | {Agnes: 0, Bambang: 0} |
| Update lokal | {Agnes: **5**, Bambang: 0} | {Agnes: 0, Bambang: **3**} |
| Sinkron (aturan **max** per orang) | {Agnes: max(5,0)=5, Bambang: max(0,3)=3} | sama |

**Total = 5 + 3 = 8.**

> **Logika kenapa CRDT tidak pernah konflik:** setiap orang **hanya mengubah "kolomnya" sendiri**, dan penggabungan memakai **max**. Fungsi max memberi hasil yang sama **berapa kali pun dan dalam urutan apa pun** digabung: max(5,0) = max(0,5) = max(max(5,0),0). Karena itu semua node pasti berakhir dengan nilai yang sama. Kalau memakai LWW, salah satu hitungan (5 atau 3) akan **tertimpa**, dan totalnya salah.

**Contoh OT (dari materi):** dokumen awal `"abc"`.
- Agnes menyisipkan `x` di indeks 0 → lokal `"xabc"`.
- Bambang menghapus indeks 2 (`c`) → lokal `"ab"`.

**Masalah tanpa transformasi:**
- Agnes menerapkan "hapus indeks 2" ke `"xabc"`, sehingga yang terhapus `b` → `"xac"`.
- Bambang menerapkan "sisipkan x di indeks 0" ke `"ab"` → `"xab"`.
- **`"xac"` ≠ `"xab"`**, dokumen keduanya berbeda.

**Solusi OT:**
- Di sisi Agnes: karena ia sudah menyisipkan 1 karakter **di depan** posisi yang dihapus, "hapus indeks 2" **ditransformasi** menjadi "hapus indeks **3**" → `"xab"`.
- Di sisi Bambang: sisipan `x` di indeks 0 diterapkan, lalu `c` dihapus di indeks yang sudah disesuaikan (3) → `"xab"`.
- Hasil keduanya sama: **`"xab"`**.

> **Logika OT:** posisi (indeks) itu **relatif**. Kalau ada karakter yang disisipkan **sebelum** posisi target, posisi target **bergeser +1**. OT menghitung pergeseran ini sebelum menerapkan operasi orang lain.

## 9. Protokol Replikasi

1. **Primary-based (Master-Slave):** satu node read-write, lainnya read.
2. **Quorum-based:** operasi berhasil jika dikonfirmasi oleh **sejumlah node tertentu**.
3. **State Machine Replication**
4. **Gossip Protocol**
5. **Chain Replication**

### 9.1 Quorum-Based

Operasi **write** dan **read** dianggap berhasil hanya jika dikonfirmasi oleh minimal sejumlah node.

**Syarat: `W + R > N`**
- N = total node
- W = minimal node yang harus konfirmasi **write**
- R = minimal node yang harus konfirmasi **read**

**Contoh dari materi:** N = 3 (Node A, B, C), W = 2, R = 2 → 2 + 2 = 4 > 3 ✅

**Skenario write `x = 100`:**
1. Permintaan dikirim **paralel** ke A, B, C.
2. A sukses (ACK), B sukses (ACK), C lambat/timeout.
3. Sudah ada **2 ACK = W**, jadi write dinyatakan **berhasil**, walau C belum selesai.

**Skenario read `x`:**
1. Sistem meminta data dari minimal **2 node** (R = 2), misalnya A dan C.
2. A memberi versi terbaru (`x = 100`), C mungkin masih versi lama.
3. Karena **A ikut dalam write tadi**, data terbaru **pasti** ada di salah satu node yang dibaca. Sistem memilih versi yang paling baru (berdasarkan nomor versi/timestamp).

> **Logika kenapa `W + R > N` menjamin data terbaru terbaca:** Ini prinsip **pigeonhole (sarang merpati)**. Ada 3 node. Write "menandai" 2 node, read "mengambil" 2 node. Total 2 + 2 = 4, padahal node hanya ada 3, jadi **minimal 1 node pasti terpilih dua kali**: ikut write *dan* ikut read. Node irisan itulah yang membawa data terbaru.
>
> | Write ke | Read dari | Irisan |
> |---|---|---|
> | A, B | A, C | **A** |
> | A, B | B, C | **B** |
> | A, B | A, B | **A, B** |
>
> Kalau `W + R ≤ N` (misalnya W = 1, R = 1, N = 3), bisa saja write ke A tetapi read dari C, sehingga **tidak ada irisan** dan data lama yang terbaca.

### 9.2 State Machine Replication

Beberapa replika **mengeksekusi perintah yang sama dalam urutan yang sama**, sehingga hasilnya pasti sama. Leader menerima permintaan, mencatatnya di **log**, lalu mengurutkannya; replika menerapkan perintah sesuai urutan itu.

**Contoh dari materi:**
1. Awal: replika A, B, C semuanya `x = 10`.
2. Klien mengirim `SET x = 20` ke **leader (A)**.
3. A memberi **nomor urut**, menambahkannya ke log, lalu mengirim perintah ke B dan C.
4. B dan C mengonfirmasi perintah sudah ada di log mereka, maka A menganggapnya **"committed"**.
5. A, B, C mengeksekusi `SET x = 20`, sehingga semua menjadi `x = 20`.
6. Leader mengirim hasil ke klien.

> **Logika:** Kalau **titik awal sama** dan **urutan perintah sama**, hasil akhirnya **pasti sama**, seperti dua orang yang mengikuti resep yang sama langkah demi langkah dengan bahan yang sama. Inilah dasar algoritma **Raft** dan **Paxos**.

### 9.3 Gossip Protocol

Dipakai di banyak database terdistribusi seperti **Apache Cassandra**, antara lain untuk **mengecek node mana yang hidup** dan untuk **replikasi data**.
1. Node A mendeteksi perubahan (data baru, atau tetangganya down).
2. **Setiap detik**, A memilih **1–3 peer secara acak** (misalnya B) dan membagikan statusnya.
3. B memperbarui statusnya, lalu memilih node acak lain (misalnya C) untuk diteruskan.
4. Setelah beberapa putaran, **semua node** mengetahui informasi itu.

> **Analogi:** Persis seperti **gosip di kampus**: satu orang cerita ke beberapa teman, teman cerita ke teman lain, sampai satu angkatan tahu.
>
> **Logika kenapa cepat:** Jika tiap putaran setiap node yang sudah tahu menyebarkan ke 1 node baru, jumlah yang tahu kira-kira **berlipat dua** tiap putaran: 1 → 2 → 4 → 8 → ... Untuk 1.000 node cukup sekitar **10 putaran** (2¹⁰ ≈ 1.024), bukan 1.000 kali kirim satu per satu. Selain itu, tidak ada node pusat yang bisa menjadi bottleneck.

### 9.4 Chain Replication

Node disusun seperti **rantai**: **head → node tengah → tail**.
- **Write** masuk ke **head**, lalu diteruskan berurutan sampai **tail**. Write baru dinyatakan sukses setelah sampai di tail.
- **Read** dilayani oleh **tail**.

> **Logika:** Data yang ada di tail pasti sudah melewati **semua** node di rantai. Jadi membaca dari tail selalu mendapat data yang sudah tersimpan di semua replika (konsistensi kuat). Contoh penggunaannya adalah **Azure Storage**.

## 10. Studi Kasus (dari Lecture Notes)

Kode lengkap ada di folder [`kode/`](kode/).

### 10.1 PostgreSQL 18 Streaming Replication (Master–Slave)

File: [`kode/postgresql/`](kode/postgresql/)

| File | Fungsi |
|---|---|
| `00_init.sql` | Dijalankan saat primary pertama kali dibuat: membuat **user `replicator`** (khusus replikasi) dan **replication slot**. |
| `docker-compose.yaml` | Menjalankan 2 container PostgreSQL 18.3: **primary** (port 5432) dan **replica** (port 5433). |
| `env.sh` | Alias shell agar perintah docker lebih singkat (opsional). |

**Konfigurasi penting di primary:**

| Opsi | Arti |
|---|---|
| `wal_level=replica` | WAL (*Write-Ahead Log*, catatan semua perubahan data) dibuat cukup lengkap untuk replikasi. |
| `max_wal_senders=10` | Maksimal 10 koneksi pengirim WAL ke replika. |
| `max_replication_slots=10` | Maksimal 10 replication slot. |
| `hot_standby=on` | Replika boleh dipakai untuk **read** selama menjadi standby. |
| `hot_standby_feedback=on` | Replika memberi tahu primary query apa yang sedang berjalan, agar data yang masih dibaca replika tidak dihapus primary. |

**Cara replika bergabung:** replika menjalankan `pg_basebackup` untuk menyalin **seluruh data** primary (diulang tiap 5 detik sampai berhasil). Opsi `-R` membuat konfigurasi standby otomatis, dan `-X stream` ikut mengalirkan WAL. Setelah itu replika menyala dan terus menerima **aliran (stream) WAL** dari primary.

**Pengujian:**
- Di replika, status *standby* bernilai **`t` (true)**: replika hanya mereplikasi dan melayani **read**.
- Tabel dibuat dan diisi di **primary**, lalu otomatis muncul di **replica**.

**High Availability:** jika primary mati, replika bisa **dipromosikan** menjadi primary (dari read-only menjadi read-write) dengan perintah **`pg_promote()`**.

> **Logika streaming replication:** Primary tidak mengirim "tabelnya", tetapi mengirim **catatan perubahan (WAL)**. Replika memutar ulang catatan itu dengan urutan yang sama, seperti state machine replication di bagian 9.2. **Replication slot** membuat primary **menyimpan WAL** yang belum diambil replika, sehingga kalau replika sempat mati, ia bisa melanjutkan dari posisi terakhir tanpa ketinggalan data.

### 10.2 Apache Ignite 3 Cluster (Master–Master)

File: [`kode/apache-ignite/`](kode/apache-ignite/)

| File | Fungsi |
|---|---|
| `docker-compose.yaml` | Menjalankan **3 node** Apache Ignite 3.1.0 (`node1`, `node2`, `node3`) yang saling mengenal lewat port 3344. |
| `env.sh` | Alias shell, termasuk menjalankan **Ignite CLI** lewat docker. |
| `sql/*.sql` | Contoh database musik (**Chinook**): tabel album, artis, lagu, pelanggan, invoice, dll. |

**Port:**
- **10300–10302**: REST API untuk administrasi (dipakai CLI).
- **10800–10802**: koneksi dari aplikasi.

**Langkah:**
1. `docker-compose up` → 3 node berjalan.
2. Masuk ke Ignite CLI, lalu inisialisasi cluster:
   `cluster init --name=ignite3 --metastorage-group=node1,node2,node3`
3. Jalankan file-file SQL di **node1** (`localhost:10300`).
4. Cek di **node2** (`10301`) dan **node3** (`10302`): data **sudah ada juga**. Artinya replikasi berjalan.

Disebut **Master–Master** karena **semua node bisa read-write** (semua punya kemampuan sebagai primary).

Di `schema.sql`, jumlah salinan diatur lewat **zone**:
```sql
CREATE ZONE IF NOT EXISTS Chinook WITH replicas=2, ...;
CREATE ZONE IF NOT EXISTS ChinookReplicated WITH replicas=3, partitions=25, ...;
```
- Tabel di zone `Chinook` disimpan **2 salinan** (di 2 dari 3 node).
- Tabel di zone `ChinookReplicated` disimpan **3 salinan** (di semua node).

> **Logika:** `replicas=2` artinya jika 1 node mati, setiap data masih punya 1 salinan lain, sehingga data tetap aman. `replicas=3` lebih aman lagi dan read bisa dilayani node mana saja, tetapi butuh ruang penyimpanan 3 kali lipat.

### 10.3 Perbandingan Kedua Studi Kasus

| Aspek | PostgreSQL Streaming Replication | Apache Ignite 3 |
|---|---|---|
| Jenis | Master–Slave | Master–Master |
| Yang bisa write | Hanya primary | Semua node |
| Replika | Read-only (hot standby) | Read-write |
| Jika primary/node mati | Promosikan replika dengan `pg_promote()` | Node lain tetap melayani, karena data punya salinan di node lain |
| Cocok untuk | Database transaksional dengan read lebih banyak dari write | Data skala besar yang butuh banyak node aktif |

## 11. Pertanyaan 5W + 1H

**What: Apa itu konsistensi dan replikasi?**
Replikasi adalah membuat dan menjaga **salinan data yang sama** di beberapa server. Konsistensi adalah **jaminan** bahwa salinan-salinan itu menunjukkan data yang sama (atau akhirnya sama) kepada pengguna.

**Why: Mengapa dibutuhkan?**
- **Replikasi:** supaya sistem **tidak mati** saat satu server rusak (availability), **lebih cepat** karena membaca dari server terdekat (performance), **data tidak hilang** (reliability), dan **beban baca terbagi** (skalabilitas).
- **Konsistensi:** supaya pengguna **tidak melihat data yang salah atau berbeda-beda**, misalnya saldo yang terpotong tapi tidak masuk ke rekening tujuan.

**Who: Siapa yang menggunakan/terlibat?**
- **Perancang sistem dan DBA** yang memilih model konsistensi dan jenis replikasi.
- **Perusahaan dengan pengguna besar:** bank (strong consistency), media sosial dan e-commerce (eventual consistency), layanan cloud (Google Spanner, Amazon DynamoDB, Azure Storage).
- **Pengguna akhir** yang merasakan dampaknya: aplikasi tetap bisa dipakai walau ada server bermasalah.

**When: Kapan digunakan?**
- Saat data **harus tetap tersedia** walau ada server yang gagal (*high availability*).
- Saat **jumlah pengguna besar** dan tersebar di banyak lokasi.
- Saat **beban baca** jauh lebih besar dari beban tulis.
- Saat perlu **cadangan data** (*disaster recovery*).
- Strong consistency dipakai saat **kesalahan data tidak bisa ditoleransi** (uang, stok, username); eventual consistency dipakai saat **kecepatan lebih penting** daripada ketepatan sesaat (*like*, *view*, timeline).

**Where: Di mana diterapkan?**
- **Database:** PostgreSQL, MySQL, MongoDB, Cassandra, Apache Ignite, DynamoDB, Spanner.
- **Sistem koordinasi:** etcd (Kubernetes), ZooKeeper.
- **Aplikasi kolaboratif:** Google Docs (OT/CRDT), editor berbasis Yjs.
- **Data center di lokasi berbeda**, misalnya Jakarta, Yogyakarta, dan Denpasar seperti contoh di materi.

**How: Bagaimana cara kerjanya?**
1. **Pilih jenis replikasi:** master-slave (satu penulis) atau multi-master (banyak penulis).
2. **Pilih waktu sinkronisasi:** synchronous (aman, lambat), semi-synchronous, atau asynchronous (cepat, berisiko).
3. **Pilih cara sinkronisasi:** pull, push (CDC + WebSocket/SSE), atau hybrid.
4. **Pilih protokol:** primary-based, quorum (`W + R > N`), state machine replication (Raft/Paxos), gossip, atau chain replication.
5. **Siapkan penanganan konflik** untuk multi-master: LWW, vector clock, CRDT, atau OT.
6. **Contoh nyata:** PostgreSQL mengalirkan WAL dari primary ke replika; Apache Ignite menyimpan beberapa salinan data di beberapa node sesuai pengaturan `replicas`.

## 12. Diskusi

**1. Apa kelebihan dan kekurangan strong vs eventual consistency?**

| | Strong Consistency | Eventual Consistency |
|---|---|---|
| **Kelebihan** | Data **selalu benar dan terbaru** di semua node. Mudah dipahami programmer (seperti satu database). Tidak ada konflik data. | **Cepat** (latensi rendah). **Tetap tersedia** walau ada partisi jaringan (AP). Mudah di-*scale* ke banyak node dan lokasi. |
| **Kekurangan** | **Lambat**, karena harus menunggu semua/mayoritas node. **Bisa menolak permintaan** saat jaringan bermasalah (mengorbankan availability). Sulit di-*scale* ke lokasi yang berjauhan. | Pengguna bisa melihat **data lama** sementara. Perlu **penanganan konflik** (LWW, vector clock, CRDT). Logika aplikasi lebih rumit. |
| **Cocok untuk** | Bank, pembayaran, stok barang, registrasi username | Media sosial (*like*, komentar, timeline), jumlah *view*, katalog produk, DNS |

> **Kesimpulan:** Tidak ada yang lebih baik secara mutlak. Pilihannya bergantung pada **akibat jika data salah sesaat**. Jika akibatnya fatal (uang hilang), pilih strong. Jika hanya kurang nyaman (jumlah *like* telat), pilih eventual. Banyak sistem besar memakai **keduanya**: strong untuk transaksi, eventual untuk fitur sosial.

**2. Bagaimana jika replikasi gagal di satu node?**

Jawabannya tergantung **node mana yang gagal** dan **jenis replikasinya**:

| Situasi | Yang Terjadi | Penanganan |
|---|---|---|
| **Replika (slave) gagal** pada master-slave | Primary tetap berjalan normal; pembacaan dialihkan ke replika lain. | Saat replika hidup lagi, ia **mengejar ketertinggalan** dari WAL yang disimpan primary lewat **replication slot**, lalu sinkron kembali. |
| **Primary (master) gagal** | Write tidak bisa dilakukan. | **Failover**: promosikan replika menjadi primary baru (`pg_promote()` di PostgreSQL), atau otomatis lewat **pemilihan leader** (Raft/Bully, lihat minggu 3). Pada mode **asynchronous**, data terakhir yang belum sempat tersalin bisa **hilang**. |
| **Mode synchronous** | Primary harus menunggu semua replika, sehingga write bisa **macet/tertunda** jika satu replika mati. | Pakai **semi-synchronous** atau **quorum**, supaya cukup menunggu sebagian replika. |
| **Quorum-based** | Selama node yang hidup masih ≥ W (untuk write) dan ≥ R (untuk read), sistem **tetap berjalan**. Contoh N = 3, W = 2: 1 node mati masih aman. | Node yang tertinggal diperbaiki saat hidup kembali (misalnya data disalin ulang dari node lain). |
| **Multi-master / gossip** (Cassandra, Ignite) | Node lain tetap melayani karena data punya salinan di node lain (`replicas=2/3`). Gossip protocol membuat semua node **cepat tahu** ada node yang down. | Saat node kembali, data disinkronkan ulang; konflik diselesaikan dengan LWW/vector clock/CRDT. |

> **Kesimpulan:** Replikasi justru dibuat **untuk menghadapi situasi ini**. Satu node gagal seharusnya **tidak menghentikan sistem**, selama jumlah salinan cukup dan ada mekanisme **deteksi kegagalan** (gossip/healthcheck), **failover** (promosi atau pemilihan leader), dan **pemulihan** (mengejar ketertinggalan data).

> **Catatan:** Di slide minggu 1, Apache Cassandra dicontohkan sebagai **CP**, sedangkan di materi ini Cassandra dicontohkan sebagai sistem **eventual consistency** (yang sejalan dengan **AP**). Versi minggu 4 ini sesuai dengan literatur pada umumnya.
