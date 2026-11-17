Materi Minggu 2: Proses dan Komunikasi Antar Proses pada Sistem Terdistribusi

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Buatkan ringkasan dari materi yang ada pada lampiran pdf, dengan penjelasan yang mudah dipahami tetapi sesuai dengan isi materi, berikan analogi singkat jika perlu dan berikan contoh agar lebih kebayang dari isi materi. untuk format sesuai dengan isi readme.md" (dengan lampiran PDF materi minggu 02)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Apa Itu Proses?

**Proses** adalah **program yang sedang dijalankan**. Program hanyalah file berisi instruksi; begitu dijalankan oleh OS, ia menjadi proses yang punya memori, status, dan sumber daya sendiri.

> **Analogi:** Resep masakan = program. Kegiatan memasak resep tersebut = proses.
>
> **Contoh:** Membuka 3 jendela Chrome berarti ada beberapa proses Chrome yang berjalan (bisa dilihat di Task Manager).

## 2. Manajemen Proses

- **Process creation**: membuat proses baru.
- **Process migration**: memindahkan proses dari satu node/mesin ke node lain (misalnya karena node asal terlalu sibuk).
- **Process monitoring**: memantau kondisi proses (berjalan, berhenti, pemakaian CPU/memori).

## 3. Sistem Terdistribusi dan Komunikasi

- Sistem terdistribusi = sekumpulan komputer/**node** independen yang bekerja bersama sebagai **satu kesatuan logis**.
- **Setiap node menjalankan proses.**
- Agar terlihat sebagai satu sistem, proses-proses di node berbeda **harus saling berkomunikasi**:
  `Node (proses) <-> Node (proses) <-> Node (proses) ...`

**Contoh nyata:**
1. **Multiplayer game**: posisi tiap pemain dikirim ke server dan pemain lain.
2. **Gojek/Grab**: aplikasi pengguna berkomunikasi dengan banyak service: (1) driver, (2) pelacakan lokasi, (3) perhitungan biaya, dst.
3. **Chat/video call WhatsApp**: komunikasi *point-to-point* di atas infrastruktur terdistribusi.

## 4. Model Komunikasi Pesan

| Model | Cara Kerja | Analogi |
|---|---|---|
| **Langsung** | Pengirim terhubung langsung ke penerima memakai **socket**. | Telepon langsung ke nomor teman. |
| **Tidak langsung** | Pesan dititipkan ke **mailbox/message queue**, penerima mengambilnya nanti (mis. perintah `write` di Linux). | Menitipkan surat di kotak pos. |
| **Pub/Sub** | Pengirim (*publisher*) mengirim ke sebuah **topik**; semua yang berlangganan (*subscriber*) topik itu menerima pesannya. | Channel YouTube: saat upload video, semua subscriber dapat notifikasi. |

## 5. Proses pada Sistem Terdistribusi

- Proses bisa berada di **node yang berbeda**.
- **Tidak ada shared memory**, karena memori tiap node terpisah secara fisik.
- Karena itu dibutuhkan **IPC (Inter-Process Communication)** khusus untuk sistem terdistribusi.

## 6. IPC (Inter-Process Communication)

Ada 4 jenis dasar:
1. **Shared Memory**: **tidak cocok** untuk sistem terdistribusi, karena proses tidak bisa mengakses memori node lain.
2. **Message Passing**
3. **Remote Procedure Call (RPC)**
4. **Middleware**

> **Analogi shared memory:** Dua orang menulis di papan tulis yang sama di satu ruangan. Ini tidak mungkin dilakukan jika mereka berada di kota berbeda; mereka harus berkirim pesan.

### 6.1 Message Passing

Proses berkomunikasi dengan **saling mengirim dan menerima pesan** (`send` / `receive`).
- Standar: **MPI (Message Passing Interface)**, versi terbaru 5.0.
- Implementasi Python: **mpi4py**.

> **Contoh:** Simulasi cuaca dijalankan di 100 komputer; tiap komputer menghitung satu wilayah, lalu bertukar hasil batas wilayahnya lewat pesan MPI.

### 6.2 RPC (Remote Procedure Call)

Memanggil **fungsi yang berada di komputer lain seolah-olah fungsi lokal**. Detail jaringan disembunyikan.
- Contoh: **gRPC** (dari Google).

> **Analogi:** Memesan makanan lewat telepon: kita cukup bilang "pesan nasi goreng", tidak perlu tahu cara dapurnya bekerja.
>
> **Contoh:** Di kode cukup menulis `hitungOngkir(asal, tujuan)`, padahal fungsi tersebut dijalankan di server lain.

### 6.3 MOM (Message-Oriented Middleware)

- **Infrastruktur perantara** untuk komunikasi dan pertukaran data melalui **pesan** (*message*), yaitu unit informasi yang mandiri.
- Pesan dikirim dan diterima secara **asinkron**: pengirim tidak perlu menunggu penerima siap.

Standar dan protokol MOM:
1. **XATMI** (X/Open group): *Distributed Transaction Processing*.
2. **AMQP** (Advanced Message Queuing Protocol): biasanya untuk aplikasi enterprise (contoh: RabbitMQ).
3. **HLA IEEE 1516**: dari IEEE dan SISO, untuk simulasi.
4. **MQTT** (MQ Telemetry Transport): untuk perangkat dengan sumber daya terbatas (IoT).
5. **DDS** (Data Distribution Service): dari OMG.
6. **XMPP** (eXtensible Messaging and Presence Protocol): untuk chat/pesan instan.

> **Analogi:** MOM seperti **jasa kurir**. Pengirim menyerahkan paket ke kurir lalu kembali bekerja; kurir yang memastikan paket sampai, walaupun penerima sedang tidak di rumah.
>
> **Contoh:** Sensor suhu IoT mengirim data via MQTT ke broker setiap menit; aplikasi dashboard mengambil data dari broker kapan saja.

## 7. Studi Kasus: Microservices

**Microservices** = aplikasi besar dipecah menjadi banyak **service kecil yang berdiri sendiri**, masing-masing punya tugas spesifik dan saling berkomunikasi (lewat REST, gRPC, atau message broker).

> **Contoh:** Aplikasi e-commerce dipecah menjadi service *user*, *produk*, *keranjang*, *pembayaran*, dan *pengiriman*. Jika service pengiriman error, pengguna masih bisa melihat produk.

## 8. Komunikasi Sinkron dan Asinkron

| Jenis | Cara Kerja | Analogi |
|---|---|---|
| **Sinkron** | Pengirim **menunggu** balasan sebelum lanjut. | Telepon: harus menunggu lawan bicara menjawab. |
| **Asinkron** | Pengirim **langsung lanjut** tanpa menunggu balasan; balasan diproses nanti. | Chat WhatsApp: kirim pesan lalu lanjut beraktivitas. |

## 9. Berbagai Masalah pada Komunikasi Antar Proses

| Masalah | Penjelasan | Solusi |
|---|---|---|
| **Reliability** | Pesan harus sampai **lengkap dan berurutan**, walau ada paket hilang atau crash. | (1) *Acknowledgement*, (2) transmisi ulang, (3) protokol andal seperti **TCP**. |
| **Ordering** | Pesan harus diproses dalam **urutan yang logis**. | (1) Pipes FIFO, (2) message queue, (3) shared memory, (4) socket & message passing, (5) RPC. |
| **Security** | Pesan bisa disadap atau dipalsukan. | **Enkripsi** dan **autentikasi**. |
| **Performance** | Komunikasi lewat jaringan memakan waktu. | Perhatikan **latency** dan **bandwidth**. |

> **Contoh ordering:** Pesan "hapus akun" diproses sebelum "buat akun" → hasilnya kacau. Urutan harus dijaga.

## 10. Protokol Komunikasi

- **Protokol dasar:** TCP (andal, berurutan), UDP (cepat, tanpa jaminan), HTTP.
- **Protokol utama di atas HTTP:**
  - **REST**: berbasis resource dan URL (`GET /produk/1`).
  - **GraphQL**: client menentukan sendiri data apa saja yang diminta dalam satu query.
  - **gRPC**: RPC berbasis HTTP/2, cepat, memakai format biner (Protocol Buffers).

> **Contoh:** Video call cenderung memakai UDP (lebih baik ada frame yang hilang daripada telat), sedangkan transfer file memakai TCP (tidak boleh ada data yang hilang).

## 11. Konsep Sinkronisasi (Clock)

- Tiap node punya **clock (jam) lokal sendiri**, dan jam-jam ini **lama-lama bergeser** (*clock drift*) serta dipengaruhi latensi jaringan.
- **Sinkronisasi clock** = menyelaraskan jam semua komputer lewat jaringan ke waktu yang sama dan konsisten.
- Penting untuk: **pengurutan kejadian yang akurat**, **konsistensi data**, dan **operasi terkoordinasi**.
- Cara umum:
  - Sumber waktu dari **atomic clock** (menghasilkan **TAI** / International Atomic Time), **GPS**, dan **UTC**.
  - Disebarkan dengan protokol **NTP** (Network Time Protocol) atau **PTP** (Precision Time Protocol). **PTP lebih presisi** dari NTP.

> **Analogi:** Beberapa teman janjian jam 7, tetapi jam tangan masing-masing selisih beberapa menit. Supaya kompak, semua mencocokkan jam ke satu acuan (misalnya jam di HP yang tersinkron internet).

## 12. Logical Clock (Lamport)

Jam fisik bisa tidak akurat, jadi **Leslie Lamport** mengusulkan **logical clock**: penghitung angka yang **tidak terkait dengan jam dunia nyata**, hanya untuk menentukan **urutan kejadian**.

**Aturan:**
1. Setiap ada kejadian → logical clock **bertambah 1**.
2. Saat mengirim pesan → sertakan angka logical clock.
3. Saat menerima pesan → bandingkan angka di pesan dengan clock lokal, **ambil yang terbesar, lalu +1**.

**Contoh kasus (dari materi):** urutan nyata: penjualan (12:00:01) → update stok (12:00:02) → faktur (12:00:03). Jika memakai timestamp jam nyata, urutan bisa tertukar karena latensi atau masalah jaringan/hardware. Dengan Lamport clock (semua mulai dari 0):

| Langkah | Service | Proses | Clock |
|---|---|---|---|
| 1 | Sales Service | Mencatat penjualan, 0 + 1, kirim ke Product dengan clock 1 | **1** |
| 2 | Product Service | max(0, 1) = 1, lalu +1, kirim ke Invoice dengan clock 2 | **2** |
| 3 | Invoice Service | max(0, 2) = 2, lalu +1, buat faktur | **3** |

Hasil urutan pasti benar: **1 Sales → 2 Product → 3 Invoice**.

> **Analogi:** Nomor antrean di bank. Kita tidak peduli jam berapa datang; yang penting nomor kecil dilayani lebih dulu.

Lamport clock masih berupa **scalar clock** (satu angka), lalu dikembangkan menjadi:
1. **Vector clocks**: Colin Fidge & Friedemann Mattern (1988)
2. **Plausible clocks**: Torres-Rojas & Ahamad (1999)
3. **Chain clocks**: Agarwal & Garg (2005)
4. **Interval Tree Clocks**: Almeida dkk. (2008)
5. **Bloom Clocks**: Lum Ramabaja (2019)

## 13. Koordinasi Proses

Mekanisme agar banyak proses independen di berbagai komputer bisa **bekerja sama secara harmonis** untuk tujuan bersama, dengan mengelola komunikasi, sinkronisasi, dan akses ke sumber daya bersama **tanpa konflik** seperti **deadlock** atau **race condition**.

Contoh tools: **Apache ZooKeeper, etcd, Consul, ClickHouse Keeper, Oxia, KRaft**.

> **Analogi:** Seperti wasit/pengatur lalu lintas di persimpangan: memastikan kendaraan bergiliran agar tidak tabrakan atau macet total.

## 14. Tantangan dan Solusi

| Tantangan | Solusi |
|---|---|
| **Message loss** (pesan hilang) | Retransmission & acknowledgement |
| **Deadlock** (saling menunggu selamanya) | Timeout, algoritma deteksi deadlock |
| **Data inconsistency** | Consensus protocol (**Raft, Paxos**) |
| **Security** | **SSL/TLS** (enkripsi), **OAuth2** (otorisasi) |

## 15. Current Issues

- **Edge & Fog Computing**: pemrosesan lebih dekat ke perangkat/pengguna.
- **Observability & tracing**: memantau sistem lewat **telemetry** (log, metrik, trace).
- **Container orchestration**: **Kubernetes, Docker Swarm**.

## 16. Diskusi

**1. Apakah GraphQL merupakan contoh dari RPC?**

**Bisa dibilang mirip RPC, tetapi bukan RPC murni.**

- **Mirip RPC:** client mengirim *query* (membaca data) atau *mutation* (mengubah data) dengan nama dan parameter, misalnya `createOrder(produkId: 1)`. Ini mirip memanggil fungsi di server. Semua request juga dikirim ke **satu endpoint** (biasanya `POST /graphql`), seperti pola RPC.
- **Berbeda dari RPC:** GraphQL berfokus pada **data dan skema (graph)**. Client bebas memilih **field apa saja** yang ingin diambil, sedangkan pada RPC klasik bentuk balasan sudah ditentukan oleh fungsinya.

> **Analogi:** RPC seperti memesan **paket menu** (isinya sudah ditentukan). GraphQL seperti **prasmanan**: kita sendiri yang memilih lauk apa saja yang diambil.

**2. Apakah REST merupakan contoh dari RPC?**

**Bukan.** REST adalah **gaya arsitektur berbasis resource**, bukan berbasis pemanggilan fungsi.

| Aspek | RPC | REST |
|---|---|---|
| Fokus | **Aksi/fungsi** (kata kerja) | **Resource/data** (kata benda) |
| Contoh | `getUser(1)`, `deleteUser(1)` | `GET /users/1`, `DELETE /users/1` |
| Endpoint | Biasanya satu endpoint per fungsi | Satu URL per resource, aksinya ditentukan **HTTP method** (GET, POST, PUT, DELETE) |
| Sifat | Bebas | *Stateless*, memanfaatkan standar HTTP (status code, caching) |

Namun dalam praktiknya, banyak API yang mengaku REST sebenarnya bergaya RPC, misalnya `POST /hapusUser`.

> **Analogi:** RPC = menyuruh orang melakukan sesuatu ("tolong ambilkan buku"). REST = menunjuk benda lalu memakai aksi standar ("buku ini, **ambil**").

**3. Apakah Telemetry merupakan solusi yang etis?**

**Bisa etis, tergantung cara penerapannya.** Telemetry (pengumpulan otomatis log, metrik, dan trace) sangat penting untuk *observability*: mendeteksi error, memantau performa, dan memperbaiki sistem terdistribusi yang rumit. Masalah etisnya muncul ketika telemetry ikut mengumpulkan **data pribadi pengguna**.

**Etis jika:**
- **Transparan**: pengguna diberi tahu data apa yang dikumpulkan dan untuk apa.
- **Ada persetujuan (opt-in/opt-out)**: pengguna boleh menolak.
- **Data minimal dan dianonimkan**: hanya data teknis yang diperlukan, tanpa identitas pribadi.
- **Aman dan sesuai regulasi**: dienkripsi dan mematuhi aturan seperti **UU PDP** (Indonesia) atau **GDPR** (Eropa).

**Tidak etis jika:** dikumpulkan diam-diam, mencakup data sensitif (lokasi, isi pesan, kebiasaan pengguna), atau dijual ke pihak lain.

> **Contoh:** Windows dan Chrome mengirim laporan *crash* untuk memperbaiki bug, dan pengguna bisa mematikannya di pengaturan (cenderung etis). Aplikasi yang diam-diam merekam lokasi pengguna lalu menjualnya ke pengiklan jelas tidak etis.
