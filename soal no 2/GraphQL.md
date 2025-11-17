## 2. Keterkaitan GraphQL dengan Komunikasi Antar Proses (IPC) pada Sistem Terdistribusi
GraphQL adalah **query language untuk API** dan runtime di server. Pada sistem mikroservis/terdistribusi, GraphQL biasanya berperan sebagai **gateway/orchestrator**:

- Client (web/mobile) mengirim **satu** GraphQL query.
- **GraphQL Gateway** (server) menerima query dan menjalankan resolver.
- Resolver tersebut komunikasi ke berbagai service lain (antar proses) menggunakan:
  - HTTP/REST
  - gRPC
  - Query langsung ke database
  - Message broker / event bus (Kafka, RabbitMQ, Redis, dsb.)
- GraphQL menggabungkan hasil dari berbagai service dan mengembalikan **satu response** ke client.

Dengan kata lain, GraphQL membantu mengatur **inter-process communication (IPC)** di belakang layar.

### 2.2. Kenapa GraphQL relevan dengan IPC?

- **Mengurangi chattiness**: client tidak perlu memanggil banyak endpoint (REST) satu per satu. Cukup satu query GraphQL → gateway yang akan memanggil banyak service.
- **Orkestrasi**: GraphQL gateway bisa melakukan pemanggilan ke beberapa service secara paralel, batching, caching, dsb.
- **Real-time**: GraphQL Subscriptions (biasanya via WebSocket) bisa menerima event dari message broker, lalu mendorong update ke client secara real-time.

Penting untuk dipahami: **GraphQL BUKAN mekanisme Komunikasi Antar Proses (IPC)**.

Sebaliknya, **GraphQL adalah *query language* (bahasa kueri) untuk API** yang seringkali bertindak sebagai **lapisan abstraksi (fasad)** yang menyembunyikan kompleksitas IPC di *backend*.

Dalam arsitektur *microservices* (sebuah bentuk sistem terdistribusi), puluhan atau ratusan layanan (proses) perlu berkomunikasi satu sama lain. Komunikasi inilah yang disebut IPC, yang bisa menggunakan berbagai mekanisme seperti **REST API, gRPC, atau *Message Queues* (RabbitMQ, Kafka)**.

#### Analogi: Asisten Pribadi vs. Departemen

* **Microservices (Proses):** Anggap sebagai departemen di perusahaan besar (misal: `UserService` adalah Dept. HRD, `PostService` adalah Dept. Operasional, `BillingService` adalah Dept. Keuangan).
* **IPC (Komunikasi Antar Proses):** Ini adalah cara *internal* departemen berkomunikasi (Telepon internal, email, memo). HRD mungkin pakai email (REST), Keuangan mungkin pakai sistem SAP (gRPC). Ini rumit dan internal.
* **Client (Aplikasi HP):** Anda adalah klien di luar perusahaan.
* **GraphQL Gateway (Asisten Pribadi):** Ini adalah asisten pribadi Anda di meja depan.

**Skenario Tanpa GraphQL:**
Anda (Klien) perlu data profil. Anda harus:
1.  Telepon HRD (IPC #1, via REST) untuk dapat nama.
2.  Telepon Operasional (IPC #2, via REST) untuk dapat daftar postingan.
3.  Telepon Keuangan (IPC #3, via gRPC) untuk dapat status tagihan.
Ini **boros** (3x panggilan) dan **rumit** (Anda harus tahu 3 protokol berbeda).

**Skenario Dengan GraphQL:**
Anda (Klien) hanya berbicara dengan **Asisten Pribadi** (GraphQL Gateway). Anda memberi **satu daftar** permintaan: "Tolong ambilkan nama, daftar postingan, dan status tagihan untuk user 123."

Asisten (GraphQL) kemudian melakukan "pekerjaan kotor" untuk Anda. Dia yang akan:
1.  Menghubungi HRD (melakukan **IPC #1**).
2.  Menghubungi Operasional (melakukan **IPC #2**).
3.  Menghubungi Keuangan (melakukan **IPC #3**).

Setelah semua data terkumpul, Asisten menggabungkannya menjadi satu laporan rapi dan memberikannya kepada Anda (satu respons JSON).

**Kesimpulan Logis:**
GraphQL adalah **jembatan** yang menyederhanakan komunikasi *client-server*, dengan cara **mengorkestrasi** dan **menyembunyikan** kerumitan komunikasi *server-server* (IPC) di belakangnya.

### Diagram

Berikut adalah diagram yang mengilustrasikan hubungan ini.

```mermaid
graph TD
    subgraph Client
        C[📱 Aplikasi Mobile/Web]
    end

    subgraph "Backend (Sistem Terdistribusi)"
        GW[🚀 GraphQL Gateway / Fasad]

        subgraph "Komunikasi Antar Proses (IPC)"
            S1[👤 UserService]
            S2[📝 PostService]
            S3[💳 BillingService]
            DB1[(Database User)]
            DB2[(Database Post)]
            DB3[(Database Billing)]
        end
    end

    %% 1. Client ke Gateway (Satu Kueri)
    C -- "1. Kueri GraphQL Tunggal\nQuery { user, posts, billing }" --> GW

    %% 2. Gateway ke Microservices (Berbagai IPC)
    GW -- "2a. IPC (misal: gRPC)" --> S1
    GW -- "2b. IPC (misal: REST)" --> S2
    GW -- "2c. IPC (misal: gRPC)" --> S3

    %% Database (Internal ke Service)
    S1 --- DB1
    S2 --- DB2
    S3 --- DB3

    %% 3. Microservices kembali ke Gateway (Respons IPC)
    S1 -- "Data User" --> GW
    S2 -- "Data Postingan" --> GW
    S3 -- "Data Tagihan" --> GW

    %% 4. Gateway kembali ke Client (Satu Respons)
    GW -- "4. Respons JSON Teragregasi" --> C
