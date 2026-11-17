Materi Minggu 6: Distributed File System

AI yang digunakan: Claude Opus 5.5 (Anthropic), melalui Claude Code

Prompt: "Berdasarkan Materi Distributed File System yang ada di lampiran pdf(06), buatkan ringkasan materi tersebut agar mudah dipahami dan berikan logika jika memang diperlukan, berikan juga pertanyaan 5w 1h pada materi ini agar tau kegunaanya." (dengan lampiran: slide `06--distributed-filesystem.pdf` dan panduan `menggunakan-hadoop-hdfs--linux.pdf`)

Bagian yang disesuaikan dari keluaran prompt ada pada: -

---

## 1. Pengertian File System (FS)

**File system** = metode yang dipakai **sistem operasi** untuk **mengelola dan mengatur file** di media penyimpanan (harddisk, flashdisk, SSD, dll). Sering ditulis *file system* atau *filesystem*, disingkat **FS**.

| Sistem Operasi | Contoh FS |
|---|---|
| **Linux** | ext4, XFS, btrfs, ReiserFS |
| **DOS/Windows** | FAT16, FAT32, NTFS |
| **Mac** | APFS (Apple File System) |

- FS diakses melalui **driver file system** yang dikelola **kernel**. Kernel menyediakan **virtual file system (VFS)** agar bisa diakses aplikasi (*user space*).
- **FS berbeda dengan hierarki FS.** Hierarki FS adalah **susunan file dan direktori** di OS (di Linux diatur oleh standar **FHS**: `/home`, `/etc`, `/var`, dst). Pengguna hanya melihat hierarkinya, tanpa peduli FS apa yang dipakai di bawahnya.

> **Analogi:** FS seperti **sistem pengarsipan** di perpustakaan (cara buku dicatat dan disusun di rak). Hierarki FS seperti **denah perpustakaan** yang dilihat pengunjung ("buku komputer di lantai 2, rak C"). Pengunjung cukup tahu denahnya, tidak perlu tahu sistem pencatatannya.
>
> **Contoh:** Di Windows kita melihat `C:\Users\...` (hierarki), padahal di bawahnya FS-nya NTFS. Flashdisk yang diformat FAT32 tetap terlihat sebagai folder biasa.

## 2. Cara Kerja FS

Alur saat aplikasi membuka/menyimpan file:

```
Aplikasi → System Call Interface → Virtual FS (VFS) → Driver FS di kernel → Hardware (media penyimpan)
```

> **Logika adanya VFS:** Aplikasi cukup memanggil perintah yang sama (`open`, `read`, `write`) untuk **semua jenis FS**. VFS yang meneruskannya ke driver yang tepat (ext4, NTFS, FAT32, dll). Karena itu aplikasi tidak perlu ditulis ulang untuk tiap jenis FS. Konsep **satu antarmuka untuk banyak media** inilah yang nanti diperluas oleh DFS ke banyak komputer.

## 3. Pengertian Distributed File System (DFS)

**DFS** bukan sekadar "FS yang tersebar di lebih dari satu mesin". Secara konseptual, DFS adalah **sistem abstraksi terkoordinasi** dari sekumpulan media penyimpanan yang **menutupi heterogenitas, kegagalan, dan konkurensi**, sehingga menghasilkan jaringan penyimpanan yang **terintegrasi, andal, dan berkinerja tinggi**.

Seperti sistem terdistribusi lainnya, DFS terdiri dari **banyak media penyimpanan**, tetapi **terlihat oleh pengguna sebagai satu media penyimpanan**.

| Yang ditutupi DFS | Artinya |
|---|---|
| **Heterogenitas** | Mesin dan disk berbeda-beda merek/ukuran/OS, tetapi terlihat seragam. |
| **Kegagalan** | Ada disk/node yang rusak, tetapi file tetap bisa diakses (karena ada replika). |
| **Konkurensi** | Banyak pengguna mengakses file bersamaan, tetapi tetap aman dan konsisten. |

> **Analogi:** **Google Drive.** Kita melihat satu "Drive" berisi folder, padahal file kita dipecah dan disimpan di ribuan disk di banyak data center.

## 4. DAS vs NAS vs SAN vs DFS

| Jenis | Cara Kerja | Contoh/Penggunaan |
|---|---|---|
| **DAS** (Direct-Attached Storage) | Disk **terpasang langsung** ke satu komputer. | Harddisk/SSD internal laptop. Terlihat dengan `df -h` (`/dev/nvme0n1p2` di-*mount* ke `/`). |
| **NAS** (Network-Attached Storage) | Penyimpanan bersama lewat **jaringan biasa**, menyajikan **file system** (berbasis file). Pengelolaan lebih mudah. | Server file kantor yang diakses banyak komputer (via NFS/SMB). |
| **SAN** (Storage Area Network) | Penyimpanan lewat **jaringan khusus** (dengan SAN switch), menyajikan **block storage**. **Cepat, tetapi mahal.** | Backend database besar, editing video/audio, virtualisasi skala besar. |
| **DFS** | **Abstraksi level software** yang mengatur data menjadi file dan direktori yang **didistribusikan ke banyak node**. | HDFS, CephFS, GlusterFS |

Poin penting dari materi:
- **SAN** menyajikan penyimpanan sebagai **block-level devices** yang dihubungkan langsung ke server, lalu server memformatnya dengan FS sendiri.
- SAN **secara konsep bisa** dipakai sebagai storage DFS, tetapi **tidak disarankan**.
- **DFS lebih mengarah pada DAS di setiap node**: tiap node memakai disk lokalnya sendiri, lalu software DFS menyatukannya.

> **Analogi:**
> - **DAS** = lemari pribadi di kamar sendiri.
> - **NAS** = lemari bersama di ruang tamu; semua penghuni bisa mengambil **barang (file)** di sana.
> - **SAN** = gudang khusus dengan jalur truk sendiri; yang dikirim berupa **kotak kosong (blok)** yang ditata sendiri oleh penerimanya. Cepat tapi mahal.
> - **DFS** = setiap rumah di satu kompleks punya lemari sendiri, tetapi ada **satu katalog bersama** sehingga semua barang terlihat seperti di satu gudang.
>
> **Logika kenapa DFS memakai DAS, bukan SAN:** DFS sudah menangani replikasi dan kegagalan **di level software**, sehingga cukup memakai disk lokal yang **murah** (commodity hardware). SAN itu mahal, dan kalau switch SAN-nya gagal semua node ikut terdampak. Ini justru bertentangan dengan tujuan DFS menghindari single point of failure.

## 5. Berbagai DFS yang Umum

| DFS | Keterangan |
|---|---|
| **Google File System (GFS)** | DFS internal Google, inspirasi HDFS. |
| **Colossus** | Pengganti GFS; dipakai di **YouTube, Gmail, Google Docs**, dll. |
| **CephFS** | File system dari **Ceph** (https://ceph.io). |
| **Gluster** | GlusterFS (https://www.gluster.org). |
| **HDFS** | Hadoop Distributed File System (https://hadoop.apache.org). |
| **Lustre** | Banyak dipakai di **superkomputer/HPC** (https://www.lustre.org). |

## 6. Cara Kerja DFS

```
                    Pengguna & Aplikasi
                            │
     ┌──────────────────────▼──────────────────────┐
     │  Global Namespace (Distributed File System) │  ← satu "pohon folder" untuk semua
     └────┬─────────┬─────────┬─────────┬─────────┬┘
          │         │         │         │          │
       NAS 1     NAS 2    Linux NFS  Windows SMB  Cloud
     (Namespace 1) (2)       (3)        (4)        (5)
     └──── Data Center ───┘ └─ Remote Location ─┘ └ Public Cloud ┘
```

- Penyimpanan bisa berasal dari **berbagai tempat**: data center (NAS), lokasi jauh (Linux NFS, Windows SMB), sampai **public cloud**.
- Masing-masing punya *namespace* sendiri (Namespace 1–5).
- DFS menyatukannya menjadi **satu Global Namespace**, sehingga pengguna dan aplikasi melihat **satu struktur folder** tanpa tahu file fisiknya ada di mana.

> **Contoh:** Pengguna membuka `/proyek/laporan.pdf`. Ia tidak tahu (dan tidak perlu tahu) bahwa file itu sebenarnya ada di NAS 2 di data center Jakarta. Inilah **transparansi lokasi** dari minggu 1.

## 7. Studi Kasus: HDFS (Hadoop Distributed File System)

### 7.1 Pengertian

**HDFS** adalah FS yang **open source, scalable, terdistribusi, dan fault-tolerant**, dirancang untuk menyimpan **dataset besar** di cluster **commodity hardware**.

- **Commodity hardware:** perangkat keras yang diproduksi **massal, murah, dan mudah dipertukarkan**, tersedia dari banyak vendor (bukan server khusus yang mahal).
- HDFS adalah **komponen utama Apache Hadoop**, dipakai untuk pemrosesan data skala besar dengan cara:
  1. **Membagi file menjadi blok-blok**,
  2. **Mendistribusikan** blok ke beberapa server,
  3. **Mereplikasi** blok agar *high availability*.
- **Apache Hadoop 3.5.0** dikompilasi dengan **JDK 17**; server memakai **Java 17**, client bisa memakai **Java 17 atau 21**.

### 7.2 Arsitektur HDFS (Master–Slave)

| Komponen | Peran |
|---|---|
| **NameNode** (master) | Menyimpan **metadata**: nama file, lokasi blok, jumlah replika (contoh: `/home/foo/data, 3, ...`). Tidak menyimpan isi file. |
| **DataNode** (slave) | Menyimpan **blok data** yang sebenarnya, dan melakukan replikasi blok ke DataNode lain. |
| **Resource Manager** (di master) / **Node Manager** (di tiap slave) | Bagian **YARN**, bukan HDFS. Resource Manager membagi sumber daya cluster, Node Manager menjalankan job pemrosesan (**Map–Reduce**) di node masing-masing. |
| **Client** | Meminta metadata ke NameNode, lalu **membaca/menulis blok langsung ke DataNode**. |

DataNode dikelompokkan dalam **rack** (Rack 1, Rack 2), dan replika blok disebar ke **rack yang berbeda**.

**Alur baca/tulis:**
1. **Write:** Client bertanya ke NameNode "simpan di mana?" → NameNode memberi daftar DataNode → Client menulis blok ke DataNode → DataNode **meneruskan replika** ke DataNode lain (termasuk di rack lain).
2. **Read:** Client bertanya ke NameNode "blok file ini ada di mana?" → Client **membaca langsung** dari DataNode terdekat.

> **Analogi:** NameNode = **katalog perpustakaan** (mencatat buku ada di rak mana). DataNode = **rak-rak buku**. Kita bertanya ke katalog dulu, lalu mengambil bukunya sendiri di rak, bukan meminta petugas katalog membawakannya.
>
> **Logika kenapa client tidak lewat NameNode untuk data:** Jika semua isi file harus lewat NameNode, NameNode akan menjadi **bottleneck**. Dengan hanya melayani metadata (data kecil), satu NameNode bisa melayani ribuan client, sementara data besar mengalir **paralel** dari banyak DataNode.
>
> **Logika membagi file jadi blok:** File 1 TB yang dipecah menjadi blok-blok (default HDFS 128 MB per blok) bisa disebar ke ratusan disk dan **dibaca/diproses paralel**. Kalau tetap utuh, file sebesar itu bahkan bisa tidak muat di satu disk.
>
> **Logika replika di rack berbeda:** Satu rack biasanya berbagi **satu switch dan satu sumber listrik**. Jika semua replika ada di rack yang sama, rack mati berarti semua salinan hilang. Menyebar ke rack lain membuat data tetap aman walau **satu rack padam total**.

### 7.3 Praktik Instalasi HDFS (dari Lecture Notes)

**Persyaratan software:** JDK 17 (dan/atau 21), Apache Hadoop 3.5.0, **pdsh** (menjalankan perintah ke banyak node sekaligus), **ssh**, dan **sshd** yang aktif.

**Langkah:**
1. **Unduh dan ekstrak** Hadoop, buat symlink `hadoop`, lalu atur `PATH` (lokasi instalasi disebut `$HADOOP_HOME`).
2. **Install pdsh**, set `PDSH_RCMD_TYPE=ssh`.
3. **SSH ke localhost tanpa password:**
   ```bash
   ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
   cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
   chmod 0600 ~/.ssh/authorized_keys
   ```
4. **Konfigurasi** di `$HADOOP_HOME/etc/hadoop/`:

   | File | Isi penting |
   |---|---|
   | `hdfs-site.xml` | `dfs.replication = 1` (jumlah replika tiap blok) |
   | `core-site.xml` | `fs.defaultFS = hdfs://localhost:9000` (alamat NameNode) |
   | `mapred-site.xml` | `mapreduce.framework.name = yarn` |
   | `yarn-site.xml` | Pengaturan memori YARN (`memory-mb = 4096`, minimal alokasi 2048 MB, dll) |

5. **Format NameNode** dengan `hdfs namenode -format`, **cukup sekali**. Jika diulang, file-file yang sudah ada di HDFS akan **terhapus**.
6. **Atur `JAVA_HOME`** di `hadoop-env.sh`.
7. **Jalankan HDFS:** `start-dfs.sh` → menyalakan NameNode, DataNode, dan Secondary NameNode.
8. **Pantau lewat web** (`http://localhost:9870`): terlihat status `localhost:9000 (active)`, versi 3.5.0, kapasitas, dan pemakaian DFS.
9. **Operasi file** (mirip perintah Linux):
   ```bash
   hdfs dfs -ls /                 # lihat isi root HDFS
   hdfs dfs -mkdir -p /user/bpdp  # buat direktori
   hdfs dfs -ls /user             # lihat isi /user
   ```
10. **Hentikan HDFS:** `stop-dfs.sh`.

> **Logika `dfs.replication = 1`:** Praktik ini memakai **satu komputer** (*pseudo-distributed*), sehingga hanya ada **1 DataNode**. Replika tidak bisa lebih banyak dari jumlah DataNode, karena menyimpan 2 salinan di disk yang sama tidak menambah keamanan. Di cluster sungguhan, nilai default-nya **3**.

## 8. Pengertian Cloud Storage

**Cloud storage** = layanan online untuk **menyimpan dan mengakses data di server jarak jauh lewat internet**, bukan di harddisk lokal.
- Data dikelola secara **remote** dalam suatu **pool (logical)** dan diakses lewat jaringan, biasanya internet.
- Penyimpanan fisiknya terdiri dari **banyak server** (kadang di beberapa lokasi), dan dimiliki serta dikelola oleh **penyedia cloud**.
- Contoh: **Amazon S3, Azure Blob Storage, Google Cloud Storage (GCS)**.

> **Analogi:** Seperti **menyewa gudang**: kita cukup menitipkan dan mengambil barang; urusan bangunan, keamanan, dan perawatan ditangani pemilik gudang.

## 9. Relasi Cloud Storage dengan DFS

- **Cloud storage = tempat penyimpanan data**, sedangkan **pengorganisasiannya dilakukan oleh DFS**.
- Contoh: **S3 bisa dipakai oleh HDFS.** S3 menjadi tempat datanya, sementara operasi file system dilakukan oleh HDFS. Pengaturannya lewat `core-site.xml`.
  - Secara teknis, Hadoop mengakses S3 lewat konektor **S3A** (alamat `s3a://nama-bucket/...`), sehingga S3 bisa dipakai dengan perintah file system Hadoop yang sama seperti HDFS.

> **Analogi:** Cloud storage = **gudang**. DFS = **sistem katalog dan pengelola** yang tahu barang mana disimpan di rak mana.

## 10. Tipe Cloud Storage

| Aspek | Block Storage | File Storage | Object Storage |
|---|---|---|---|
| **Bentuk data** | Potongan **blok** mentah | **File dan folder** (hierarki) | **Objek** = data + metadata + ID unik (struktur datar) |
| **Transport** | FC atau iSCSI | TCP/IP | TCP/IP |
| **Interface** | Direct Attached atau SAN | NFS, SMB | **HTTP, REST** (API S3) |
| **Kegunaan** | **Latensi rendah**, cocok untuk **data terstruktur** (database) | Performa baik, **berbagi file**, *global file locking* | **Scaling tanpa batas**, diakses lewat LAN dan WAN (internet) |
| **Contoh** | Disk virtual untuk VM (AWS EBS) | Folder bersama kantor (NAS) | Amazon S3, foto/video di aplikasi, backup |

> **Analogi:**
> - **Block** = **batu bata**: belum berbentuk apa-apa, kita yang menyusunnya (diformat sendiri dengan FS). Paling fleksibel dan cepat.
> - **File** = **lemari arsip** dengan map dan laci (folder bertingkat).
> - **Object** = **tempat penitipan barang dengan nomor tiket**: setiap barang diberi tiket unik (ID) dan label (metadata), tanpa susunan folder bertingkat. Cocok untuk menyimpan barang dalam jumlah tak terbatas.
>
> **Logika kenapa object storage paling mudah di-*scale*:** Object storage tidak punya **hierarki folder** yang harus dijaga konsistensinya, karena setiap objek cukup dicari dengan **ID unik**. Tanpa pohon folder yang harus dikunci saat diubah, objek bisa disebar ke server mana saja, dan kapasitas bisa ditambah hanya dengan menambah server.

## 11. Studi Kasus Cloud Storage

| Proyek | Keterangan |
|---|---|
| **Ceph** (ceph.io) | *Open-source distributed storage system*. Satu sistem menyediakan **object, block, dan file storage** (CephFS). |
| **CubeFS** (cubefs.io) | *Cloud native unstructured data storage* (proyek CNCF). Mendukung protokol **S3, HDFS, dan POSIX**. Dipakai untuk big data, AI/LLM, dan platform container. |
| **RustFS** (rustfs.com) | Object storage berbasis **Rust**, kompatibel S3, diposisikan sebagai **pengganti MinIO/S3**. |
| **SILO** (silo.pgsty.com) | Object storage **kompatibel S3**, merupakan **fork dari MinIO** yang dikelola oleh PGSTY (sesuai yang disebut di materi minggu 1: "MinIO sekarang sudah di-fork ke Silo"). |
| **Rook** (rook.io) | *Cloud-native storage* untuk **Kubernetes**; mengelola **file, block, dan object storage** (berbasis Ceph). |

**Arsitektur CubeFS** (contoh dari materi):
- **Client** (container, VM, server) mengakses lewat **POSIX/HDFS interface**, sedangkan aplikasi lewat **AWS S3 SDK**.
- **Master** mengoordinasi seluruh sistem.
- **Metadata Subsystem** (**Meta Node**): menyimpan metadata di memori (*meta partition*).
- **Data Subsystem**: **Data Node** (data partition) dan **Blob Node** (memakai **Erasure Code Stripe**).
- **Object Subsystem** (**Object Node**): melayani akses gaya S3.

> **Logika:** Polanya sama dengan HDFS: **metadata dipisahkan dari data**. Metadata yang kecil dan sering diakses disimpan di memori (cepat), sedangkan data besar disebar ke banyak node. **Erasure coding** adalah alternatif replikasi yang lebih hemat ruang: data dipecah dan ditambah potongan "paritas", sehingga data tetap bisa dibangun ulang walau beberapa potongan hilang, tanpa perlu menyimpan 3 salinan penuh.

## 12. Pertanyaan 5W + 1H

**What: Apa itu DFS dan cloud storage?**
- **DFS:** file system yang menyatukan banyak media penyimpanan di banyak komputer menjadi **satu penyimpanan** yang terlihat utuh oleh pengguna, dengan menutupi heterogenitas, kegagalan, dan konkurensi.
- **Cloud storage:** layanan penyimpanan data di server jarak jauh lewat internet. Datanya disimpan di cloud storage, dan DFS yang mengorganisasikannya.

**Why: Mengapa dibutuhkan?**
- Data modern **terlalu besar** untuk satu disk/server (big data, video, AI).
- Butuh **fault tolerance**: disk rusak, data tetap aman karena ada replika.
- Butuh **pemrosesan paralel**: file dipecah jadi blok sehingga bisa diproses banyak mesin bersamaan.
- **Lebih murah:** cukup memakai commodity hardware dan kapasitas bisa ditambah sesuai kebutuhan.

**Who: Siapa yang menggunakan?**
- **Perusahaan teknologi besar:** Google (GFS/Colossus untuk YouTube, Gmail, Docs), penyedia cloud (Amazon S3, Azure, GCS).
- **Data engineer dan data scientist** yang mengolah big data dengan Hadoop/HDFS.
- **DevOps/sysadmin** yang mengelola storage cluster (Ceph, Rook di Kubernetes).
- **Pengguna biasa:** secara tidak langsung, setiap kali memakai Google Drive, YouTube, atau backup foto di cloud.

**When: Kapan digunakan?**
- Saat ukuran data **melebihi kapasitas satu server**.
- Saat data harus **tetap tersedia** walau ada server/disk yang rusak.
- Saat data harus **diakses banyak pengguna/aplikasi** dari banyak lokasi.
- Saat data perlu **diproses paralel** (analitik big data, training AI).

**Where: Di mana diterapkan?**
- **Data center** perusahaan dan **penyedia cloud**.
- **Cluster Hadoop** untuk analitik data.
- **Kubernetes** (Rook, CubeFS) untuk aplikasi cloud-native.
- **Superkomputer/HPC** (Lustre).
- **Layanan sehari-hari:** penyimpanan foto/video media sosial, streaming, backup.

**How: Bagaimana cara kerjanya?**
1. File **dipecah menjadi blok** (HDFS: default 128 MB).
2. **Metadata** (nama file, lokasi blok, jumlah replika) dicatat oleh **NameNode/Meta Node**.
3. **Blok disimpan dan direplikasi** di beberapa **DataNode**, tersebar di rack yang berbeda.
4. Client bertanya lokasi ke NameNode, lalu **membaca/menulis langsung** ke DataNode.
5. Semua lokasi disatukan dalam **global namespace**, sehingga pengguna hanya melihat satu struktur folder.
6. Penyimpanan fisiknya bisa berupa disk lokal (DAS) atau **cloud storage** seperti S3, diatur lewat konfigurasi (`core-site.xml`).