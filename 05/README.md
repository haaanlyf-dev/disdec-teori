Materi Minggu 5: Fault Tolerance

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Berdasarkan Materi Fault Tolerance yang ada di lampiran pdf(05), buatkan ringkasan materi tersebut agar mudah dipahami dan berikan logika jika memang diperlukan, berikan juga pertanyaan 5w 1h pada materi ini agar tau kegunaanya." (dengan lampiran: slide `05--fault-tolerant-sys.pdf` dan panduan `web-app-py-blacksheep-load-balancing-nginx.pdf`)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Pengertian Fault Tolerance

**Fault tolerance** = kemampuan sistem untuk **tetap beroperasi dengan benar** walaupun **satu atau lebih komponennya gagal/error**.

Kemampuan ini sangat penting untuk sistem:
- **High availability**: harus selalu bisa diakses (contoh: e-commerce, m-banking).
- **Mission critical**: kegagalan menyebabkan kerugian besar (contoh: sistem pembayaran, bursa saham).
- **Life-critical**: kegagalan bisa membahayakan nyawa (contoh: sistem rumah sakit, kendali pesawat).

> **Analogi:** Mobil dengan **ban serep**. Ban bocor (komponen gagal), tetapi perjalanan tetap bisa dilanjutkan.
>
> **Contoh:** Pesawat punya lebih dari satu mesin. Jika satu mesin mati, pesawat masih bisa terbang dan mendarat dengan aman.

## 2. Arti Penting Fault Tolerance pada Sistem Terdistribusi-Terdesentralisasi

- Sistem terdistribusi/terdesentralisasi punya **banyak komponen** (server, jaringan, disk, proses).
- **Koneksi antar-komponen** bisa bermasalah.
- **Semakin kritis aplikasinya**, semakin tinggi tuntutan untuk tetap berfungsi. Karena itu kebutuhan **high availability** harus diantisipasi sejak perancangan.

> **Logika:** Misalkan satu server punya peluang 99% untuk berjalan normal. Jika sistem butuh **100 server yang semuanya harus hidup**, peluang semuanya hidup = 0,99¹⁰⁰ ≈ **37%**. Artinya **makin banyak komponen, makin pasti ada yang gagal**. Karena itu sistem terdistribusi **harus dirancang dengan anggapan bahwa kegagalan itu normal**, bukan pengecualian.

## 3. Jenis Fault

| Jenis | Sifat | Contoh |
|---|---|---|
| **Transient Fault** | **Sementara**, muncul sekali lalu hilang | Gangguan listrik sesaat karena **petir** |
| **Intermittent Fault** | **Muncul dan hilang berulang-ulang** | Hardware **overheat**: mati, dingin, hidup lagi, panas, mati lagi |
| **Permanent Fault** | **Tetap**, tidak hilang sampai diperbaiki | Harddisk rusak total, chip terbakar |
| **Byzantine Fault** | Komponen memberi hasil **acak/tidak konsisten**, bahkan bisa "berbohong" | Node mengirim data berbeda ke node lain yang berbeda (karena bug, atau sengaja diserang) |

> **Analogi Byzantine:** Dalam rapat, ada anggota yang bilang "setuju" ke si A tetapi "tidak setuju" ke si B. Ini lebih berbahaya daripada anggota yang **diam** (mati), karena informasinya **menyesatkan**.
>
> **Logika (tambahan):** Untuk tetap benar walau ada **f** node Byzantine, umumnya dibutuhkan minimal **3f + 1** node. Contoh: untuk mentoleransi 1 node "pembohong", perlu minimal 4 node. Blockchain memakai prinsip *Byzantine Fault Tolerance* karena node-nodenya tidak saling percaya.

## 4. Klasifikasi Fault Berdasarkan Sumber

| Sumber | Contoh |
|---|---|
| **Hardware Faults** | Disk crash, power failure (listrik mati) |
| **Software Faults** | Bug, memory leak, race condition |
| **Communication Faults** | Packet loss, network partition (jaringan terputus) |

> **Contoh:** Aplikasi ojek online tiba-tiba error.
> - Server di data center mati listrik → **hardware**.
> - Update aplikasi terbaru ada bug saat menghitung tarif → **software**.
> - Kabel fiber optik antar-kota putus → **communication**.

## 5. Teknik untuk Mencapai Fault Tolerance

Ada 5 teknik utama:
1. Redundansi dan Replikasi
2. Load Balancing
3. Checkpointing dan Rollback Recovery
4. Heartbeat dan Failure Detection
5. Consensus dan Coordination

Bagian 5.5 (Retry) dan 5.6 (Circuit Breaker) di bawah bukan teknik tambahan. Di slide, keduanya dibahas sebagai **lanjutan dari Heartbeat dan Failure Detection**, yaitu cara menangani komponen yang terdeteksi gagal.

### 5.1 Redundansi dan Replikasi

- Resource **tidak hanya satu**, tetapi **direplikasi ke beberapa node** (terjadi redundansi).
- Tujuan: **tidak ada single point of failure**.
- Detailnya ada di materi minggu 4 (Konsistensi dan Replikasi).

> **Analogi:** Menyimpan file tugas di laptop, Google Drive, **dan** flashdisk. Laptop rusak, tugas masih aman.

### 5.2 Load Balancing

- Aplikasi di-*scale* menjadi **lebih dari satu instance** (satu instance = satu proses aplikasi penuh yang aktif).
- Permintaan pengguna dibagi oleh **load balancer** (misalnya **HAProxy** atau **nginx**) ke instance-instance tersebut.
- Jika satu instance mati, load balancer tetap bisa mengarahkan permintaan ke instance lain.
- Praktiknya dibahas di bagian 8 (Studi Kasus).

> **Analogi:** Antrean di bank dengan beberapa teller. **Satpam (load balancer)** mengarahkan nasabah ke teller yang kosong. Kalau satu teller istirahat, nasabah diarahkan ke teller lain.

### 5.3 Checkpointing dan Rollback Recovery

- **Checkpointing:** menyimpan **status sistem secara berkala** ke media penyimpanan.
- **Rollback Recovery:** setelah terjadi kegagalan, sistem **dikembalikan ke checkpoint terakhir**.
- Sistem bisa **lanjut dari status tersimpan, bukan dari awal**, sehingga kehilangan data dan waktu pemrosesan lebih sedikit.

> **Analogi:** **Save game.** Kalau kalah di level 10, kita mulai lagi dari save point terakhir di level 9, bukan dari level 1.
>
> **Logika trade-off:** Checkpoint **terlalu sering** membuat sistem lambat karena sibuk menyimpan. Checkpoint **terlalu jarang** membuat banyak pekerjaan hilang saat gagal (misalnya checkpoint tiap 1 jam berarti maksimal 1 jam pekerjaan harus diulang). Intervalnya dipilih sesuai seberapa mahal pekerjaan yang hilang.
>
> **Contoh:** Proses training AI selama 3 hari menyimpan checkpoint model setiap 1 jam. Jika server mati di jam ke-50, training dilanjutkan dari jam ke-50, bukan dari nol.

### 5.4 Heartbeat dan Failure Detection

- **Heartbeat:** **sinyal periodik** yang dikirim antar-komponen untuk menunjukkan "saya masih hidup".
- **Failure Detection:** memakai heartbeat untuk menentukan apakah komponen gagal. Jika heartbeat **tidak diterima dalam jangka waktu tertentu (timeout)**, komponen **dianggap gagal** dan tidak responsif.
- Pola ini krusial agar kegagalan **cepat terdeteksi dan ditangani**.

> **Analogi:** Absen "ping" di grup WhatsApp kelompok. Kalau satu anggota tidak menjawab dalam 1 jam, dianggap tidak aktif dan tugasnya dialihkan ke anggota lain.
>
> **Logika memilih timeout:** Timeout **terlalu pendek** membuat node yang hanya lambat (misalnya jaringan sedang padat) **dikira mati** (*false positive*), sehingga terjadi failover yang tidak perlu. Timeout **terlalu panjang** membuat node yang benar-benar mati **lama baru ketahuan**, sehingga pengguna lama mendapat error. Karena itu biasanya node baru dianggap mati setelah **beberapa heartbeat berturut-turut** hilang.

**Contoh dari materi (`check-server.py`):** script sederhana untuk mengecek apakah server hidup.
```
$ python check-server.py localhost 44777
Server up
$ python check-server.py localhost 80
Server down
```
Untuk diagnostik yang lebih lengkap, dibutuhkan program yang lebih kompleks.

### 5.5 Retry dengan Library `tenacity`

Aplikasi sering bergantung pada **komponen di luar dirinya** (database, API lain, layanan cloud), dan koneksi ke sana **selalu mungkin gagal**. Library Python **tenacity** dipakai untuk **mencoba ulang (retry)** secara otomatis (contoh di materi: `check-retry.py`).

> **Logika:** Banyak kegagalan bersifat **transient** (sementara). Mencoba lagi beberapa detik kemudian sering kali **berhasil**. Retry biasanya diberi **jeda yang makin lama** (*exponential backoff*, misalnya 1 → 2 → 4 → 8 detik) dan **batas maksimal percobaan**, supaya server yang sedang kesulitan tidak dibanjiri permintaan.

### 5.6 Circuit Breaker

Pola desain di **microservices** untuk **mencegah kegagalan berantai (cascading failure)** dengan **memblokir sementara** panggilan ke layanan yang sedang gagal. Bekerja seperti **state machine** dengan 3 status:

| Status | Perilaku | Pindah ke... |
|---|---|---|
| **Closed** (normal) | Permintaan **diizinkan** dan dipantau kegagalannya | **Open**, jika jumlah kegagalan mencapai **ambang batas** |
| **Open** (putus) | Permintaan **langsung ditolak** tanpa memanggil layanan, sehingga layanan punya waktu untuk pulih | **Half-open**, setelah **batas waktu** tertentu |
| **Half-open** (uji coba) | Hanya **sejumlah kecil** permintaan diizinkan untuk menguji layanan | **Closed** jika uji coba berhasil; **Open** lagi jika gagal |

```
         gagal ≥ ambang batas
 CLOSED ─────────────────────► OPEN
   ▲                             │
   │ uji coba berhasil           │ batas waktu habis
   │                             ▼
   └──────────────────────── HALF-OPEN
          (uji coba gagal → kembali ke OPEN)
```

> **Analogi:** **MCB/sekring listrik di rumah.** Saat terjadi korsleting, MCB **turun (open)** sehingga listrik terputus dan kerusakan tidak menjalar. Setelah diperbaiki, MCB dinaikkan lagi (closed).
>
> **Logika kenapa perlu:** Tanpa circuit breaker, jika layanan *pembayaran* lambat, layanan *checkout* ikut menunggu sampai timeout untuk **setiap** permintaan. Thread/koneksinya habis, *checkout* ikut mati, lalu layanan lain yang bergantung pada *checkout* ikut mati juga (**berantai**). Dengan circuit breaker, *checkout* langsung tahu "pembayaran sedang bermasalah" dan bisa menampilkan pesan cepat, tanpa ikut tumbang.

**Contoh dari materi (`check-circuit-breaker.py`):**
- Saat server di `localhost:44777` **mati**: `Max retries exceeded ... Connection refused`.
- Saat server **hidup**: `Successfully accessed http://localhost:44777`.

### 5.7 Consensus dan Coordination

- **Consensus:** proses ketika beberapa node **menyepakati satu nilai, status, atau urutan operasi**, bahkan saat terjadi kegagalan dan penundaan jaringan.
- **Coordination:** praktik yang **lebih luas** untuk mengelola interaksi antar-node agar bekerja sama secara efektif dan konsisten. **Coordination sering memakai consensus** sebagai komponen kunci.

**Contoh dari materi:**

| Kasus | Peran Consensus |
|---|---|
| **Database leader election** | Leader dipilih untuk menangani write, misalnya dengan algoritma **Raft**. |
| **Update file di cloud storage** (Google Drive) | Perubahan di-*commit* ke **mayoritas server**. Begitu mayoritas setuju, update dianggap final, lalu *eventually* direplikasi ke semua server. |
| **Cryptocurrency dan blockchain** | Consensus dipakai untuk **memvalidasi dan menyetujui transaksi**. |

> **Logika "mayoritas":** Dengan 5 server, mayoritas = 3. Dua kelompok mayoritas (3 + 3 = 6 > 5) **pasti berbagi minimal 1 server**, sehingga tidak mungkin ada dua keputusan berbeda yang sama-sama "disetujui mayoritas". Sistem juga tetap jalan walau **2 server mati**. Prinsipnya sama dengan quorum `W + R > N` di minggu 4.

## 6. Actor Model untuk Supervisi Proses

**Actor model** = model pemrograman konkuren di mana program terdiri dari banyak **actor**:
- Tiap actor punya **mailbox** (kotak pesan) sendiri.
- Actor **tidak berbagi memori**; mereka hanya berkomunikasi dengan **mengirim pesan** ke mailbox actor lain.
- Actor memproses pesan **satu per satu** dari mailbox-nya.

**Supervisi (Supervisor–Worker):** actor disusun seperti **pohon**:
- **Top level supervisor** mengawasi beberapa **supervisor**, dan supervisor mengawasi **actor pekerja**.
- Supervisor **mengecek apakah actor masih hidup** (*check if alive*).
- Jika ada actor yang **gagal (crash)**, supervisor **me-restart** actor itu, tanpa mengganggu actor lain.

> **Analogi:** Struktur **perusahaan**: direktur → manajer → karyawan. Karyawan berkomunikasi lewat **memo** (pesan), bukan saling membaca pikiran (shared memory). Kalau satu karyawan sakit, **manajernya** yang mencari pengganti; direktur dan divisi lain tetap bekerja normal.
>
> **Logika "let it crash":** Daripada menulis penanganan error yang rumit di setiap bagian kode, actor dibiarkan **crash** jika ada error, lalu supervisor **me-restart-nya ke kondisi awal yang bersih**. Karena actor tidak berbagi memori, crash satu actor **tidak merusak data actor lain**.

## 7. Implementasi Fault Tolerance di Berbagai Bahasa Pemrograman

| Bahasa | Dukungan |
|---|---|
| **Erlang / Elixir** | **Built-in** actor model dan Supervisor–Worker (dipakai oleh WhatsApp dan sistem telekomunikasi). |
| **Pony** | Bahasa yang memang **dikhususkan** untuk actor model (https://www.ponylang.io). |
| **Python** | Library pihak ketiga: **Xoscar** (actor), **tenacity** (retry), **circuit breaker**, dll. |
| **Java** | **Apache Pekko** (actor model, Supervisor–Worker), **Resilience4j** (fault tolerance: retry, circuit breaker, dll). |
| **Lainnya** | Cari library fault tolerance di repositori paket masing-masing: **npm** (JavaScript), **crates.io** (Rust), dll. |

## 8. Studi Kasus: Load Balancing Aplikasi Menggunakan nginx

Dari panduan `web-app-py-blacksheep-load-balancing-nginx.pdf`. Tujuannya mencapai **high availability** dengan men-*scale* aplikasi menjadi **2 instance** dan memasang **nginx** sebagai load balancer.

### 8.1 Komponen

| Komponen | Keterangan |
|---|---|
| **Aplikasi web** | Aplikasi **ASGI Python** dengan framework **Blacksheep** (dibuat dengan `blacksheep-cli`), dijalankan oleh **uvicorn**. Di-*scale* menjadi **2 instance**. |
| **nginx** | Proxy **load balancer** yang membagi request ke 2 instance aplikasi. |
| **Docker + Docker Compose** | Menjalankan semuanya sebagai container. |
| **uv** | *Environment manager* Python untuk membuat virtual environment (`uv venv`). |

Struktur:
```
load-balancing/
├── blacksheep_lb/      ← aplikasi + Dockerfile (bawaan Blacksheep, port 80)
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml
└── env.sh              ← alias (opsional)
```

### 8.2 Langkah Singkat

1. Buat environment Python dengan `uv`, lalu install `blacksheep` dan `blacksheep-cli`.
2. Buat aplikasi dengan CLI Blacksheep, install requirement, lalu tes: `python dev.py` → buka `http://localhost:44777`.
3. Siapkan konfigurasi nginx sebagai load balancer:
   ```nginx
   http {
     upstream backend {
       server bs_app:80;     # nama service aplikasi di docker-compose
     }
     server {
       listen 80;
       location / {
         proxy_pass http://backend;
       }
     }
   }
   ```
4. Siapkan `docker-compose.yml`:
   ```yaml
   services:
     bs_app:
       build: ./blacksheep_lb
     nginx:
       container_name: nginx
       build: ./nginx
       ports:
         - 80:80
       depends_on:
         - bs_app
   ```
5. Jalankan dengan **2 instance aplikasi**:
   ```bash
   docker-compose up --build -d --scale bs_app=2
   ```
6. Buka `http://localhost` (port **80**, bukan 44777 lagi). `docker ps` menampilkan `load-balancing-bs_app-1` dan `load-balancing-bs_app-2`.
7. Cek log: sebagian request masuk ke `bs_app-1`, sebagian ke `bs_app-2`. Ini bukti nginx membagi request.

> **Logika kenapa cukup menulis `server bs_app:80` sekali:** Saat `--scale bs_app=2`, Docker membuat 2 container dengan **nama service yang sama**. DNS internal Docker menerjemahkan nama `bs_app` menjadi **2 alamat IP**. nginx menganggap keduanya anggota `upstream backend`, lalu membagi request secara bergiliran (**round robin**, algoritma default nginx). Jika satu container mati, nginx tetap bisa meneruskan request ke container yang masih hidup.
>
> **Hubungan dengan fault tolerance:** Pengguna hanya tahu satu alamat (`localhost:80`). Di belakangnya ada 2 instance, sehingga **satu instance mati tidak membuat aplikasi mati**. Ini menghilangkan single point of failure di level aplikasi. Perlu dicatat bahwa nginx-nya sendiri masih satu; pada sistem produksi, load balancer juga dibuat redundan.

## 9. Pertanyaan 5W + 1H

**What: Apa itu fault tolerance?**
Kemampuan sistem untuk **tetap berjalan dengan benar walau ada komponen yang gagal**. Dicapai dengan teknik seperti redundansi/replikasi, load balancing, checkpointing, heartbeat, retry, circuit breaker, consensus, dan supervisi actor.

**Why: Mengapa dibutuhkan?**
- Sistem terdistribusi punya **banyak komponen**, jadi kegagalan **pasti terjadi** (lihat logika di bagian 2).
- Downtime menyebabkan **kerugian** (uang, kepercayaan pengguna), bahkan bisa **membahayakan nyawa** pada life-critical system.
- Mencegah satu kegagalan kecil **menjalar** menjadi kegagalan seluruh sistem.

**Who: Siapa yang menggunakan/terlibat?**
- **Software engineer/arsitek sistem** yang merancang aplikasi tahan gagal (retry, circuit breaker, actor).
- **DevOps/SRE** yang mengatur load balancer, replikasi, monitoring heartbeat, dan failover.
- **Perusahaan** yang layanannya harus 24/7: bank, e-commerce, ojek online, streaming, telekomunikasi (Erlang dipakai di sistem telekomunikasi dan WhatsApp), blockchain.

**When: Kapan diterapkan?**
- Sejak **tahap perancangan**, bukan setelah sistem sering error.
- Saat aplikasi bergantung pada **komponen luar** (API, database) → retry + circuit breaker.
- Saat **trafik tinggi** → load balancing.
- Saat ada **proses panjang** (batch, training AI) → checkpointing.
- Saat ada **banyak node** yang harus sepakat → consensus/leader election.

**Where: Di mana diterapkan?**
- **Level infrastruktur:** load balancer (nginx, HAProxy), cluster database, data center berlapis.
- **Level aplikasi:** library tenacity, Resilience4j, circuit breaker di microservices.
- **Level bahasa/runtime:** Erlang/Elixir (supervisor), Pony, Apache Pekko.
- **Layanan sehari-hari:** Google Drive (consensus mayoritas), blockchain, aplikasi web yang di-*scale* dengan Docker.

**How: Bagaimana cara mencapainya?**
1. **Hilangkan single point of failure:** replikasi data dan jalankan beberapa instance aplikasi.
2. **Bagi beban** dengan load balancer, dan arahkan ke instance yang sehat.
3. **Deteksi kegagalan** dengan heartbeat dan timeout.
4. **Pulihkan diri:** retry untuk gangguan sementara, circuit breaker untuk gangguan panjang, supervisor untuk me-restart proses, checkpoint/rollback untuk melanjutkan pekerjaan.
5. **Sepakati keputusan** dengan consensus (Raft) agar data tetap konsisten walau ada node yang gagal.
