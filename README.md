# Jawaban Ujian: CAP, BASE, GraphQL & PostgreSQL Streaming Replication

Repository ini berisi ringkasan jawaban dari 3 soal:

1. Penjelasan **Teorema CAP** dan **BASE** serta keterkaitannya (dengan contoh).
2. Keterkaitan **GraphQL** dengan komunikasi antar proses pada sistem terdistribusi (beserta diagram).
3. Contoh **PostgreSQL Streaming Replication** menggunakan **Docker / Docker Compose** untuk menjelaskan sinkronisasi.

---

## 1. Teorema CAP dan BASE + Keterkaitannya

### 1.1. Teorema CAP

**CAP theorem** (Eric Brewer) menyatakan bahwa dalam sistem terdistribusi, terutama saat terjadi **network partition (P)**, kita **tidak bisa mendapatkan Consistency (C) dan Availability (A) secara bersamaan**. Kita dipaksa memilih:

- **C – Consistency**  
  Semua node memberikan data yang **sama dan terbaru**. Setiap `read` setelah `write` akan melihat hasil terbaru (strong consistency).

- **A – Availability**  
  Setiap request ke sistem selalu mendapat **response** (tidak error/timeout), walaupun mungkin datanya tidak paling baru.

- **P – Partition tolerance**  
  Sistem tetap berjalan meskipun ada pemisahan jaringan antar node (node-node tidak bisa saling berkomunikasi).

Saat **partition terjadi**, pilihan praktisnya:

- **CP (Consistency + Partition tolerance)**  
  Prioritaskan konsistensi; sistem boleh mengorbankan availability (misalnya menolak request sampai cluster sehat).

- **AP (Availability + Partition tolerance)**  
  Prioritaskan availability; sistem boleh mengorbankan konsistensi kuat dan menerima **eventual consistency**.

#### Contoh singkat

- **Sistem perbankan (transfer uang)**  
  Biasanya memilih **CP**:  
  Lebih baik menolak transaksi ketika jaringan bermasalah daripada saldo nasabah jadi salah.

- **Feed media sosial / katalog produk**  
  Biasanya memilih **AP**:  
  Lebih penting pengguna bisa lihat konten meskipun kadang data sedikit telat (stale).

---

### 1.2. BASE (Basically Available, Soft state, Eventual consistency)

**BASE** adalah pendekatan praktis untuk sistem yang memilih **AP** pada CAP.

Kepanjangannya:

- **Basically Available**  
  Sistem berusaha **selalu merespons**, walaupun mungkin datanya belum konsisten.

- **Soft state**  
  State (data) bisa berubah seiring waktu, bahkan tanpa interaksi langsung dari client, karena adanya **replication** atau **background process**.

- **Eventual consistency**  
  Jika tidak ada lagi update, **pada akhirnya** semua node akan **konvergen ke nilai yang sama**.

#### Contoh singkat

- **Jumlah like pada postingan sosmed**  
  Ketika user menekan like:
  - Update dikirim ke beberapa node.
  - Sementara, node lain mungkin belum menerima update → nilai like yang ditampilkan bisa berbeda antar user.
  - Setelah beberapa saat (replication selesai), semua node menampilkan nilai yang sama.

---

### 1.3. Keterkaitan CAP dan BASE

- **CAP**: teori batasan yang mengatakan bahwa ketika ada partition, Anda harus memilih antara **C** atau **A** (dengan tetap mendukung P).
- **BASE**: filosofi desain ketika kita memilih **Availability + Partition tolerance (AP)** dan **melonggarkan Consistency menjadi Eventual consistency**.

Jadi:

- Sistem **AP** pada CAP biasanya diimplementasikan dengan prinsip **BASE**.
- Sistem **CP** cenderung menjaga **strong consistency**, tetapi mungkin mengorbankan availability saat terjadi masalah jaringan.

---

### 1.4. Contoh Kombinasi dalam Sistem Nyata

- **E-commerce**:
  - **Katalog produk** (nama, deskripsi, image) → bisa AP/BASE: tidak masalah kalau update terlambat sedikit.
  - **Stock barang untuk checkout** → cenderung CP: tidak boleh oversell.

- **Arsitektur hibrid**:
  - Gunakan database AP (mis. NoSQL) untuk read-heavy dan non-kritis.
  - Gunakan database CP (mis. PostgreSQL) untuk transaksi yang sangat penting (uang, stok, dsb).

---

## 2. GraphQL & Komunikasi Antar Proses di Sistem Terdistribusi

### 2.1. Peran GraphQL

**GraphQL** adalah:

- **Query language** untuk API.
- **Runtime** untuk mengeksekusi query tersebut terhadap data Anda.

Dalam sistem terdistribusi (misalnya microservices), GraphQL sering dipakai sebagai **gateway** atau **API aggregator**:

- Client mengirim **satu** query GraphQL.
- GraphQL server (gateway) akan:
  - Memanggil **banyak service** di backend (REST/gRPC/database/message broker).
  - Menggabungkan hasilnya.
  - Mengembalikan satu response ke client.

Ini secara langsung berkaitan dengan **komunikasi antar proses (IPC)**:

- Antar proses dalam sistem terdistribusi (microservices) saling komunikasi melalui:
  - HTTP/REST
  - gRPC
  - Message broker (Kafka, RabbitMQ, dsb)
- GraphQL berada di **lapisan depan**, bertindak sebagai **orchestrator** untuk calls tersebut.

### 2.2. Alur Sederhana

1. Client mengirim query GraphQL:
   ```graphql
   query {
     user(id: "123") {
       name
       orders {
         id
         total
       }
     }
   }
