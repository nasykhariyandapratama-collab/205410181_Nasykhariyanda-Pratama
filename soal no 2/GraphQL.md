## 2. Keterkaitan GraphQL dengan Komunikasi Antar Proses (IPC) pada Sistem Terdistribusi

### 2.1. Peran GraphQL di sistem terdistribusi

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

### 2.3. Diagram Arsitektur GraphQL + IPC
flowchart TD
    Client -->|GraphQL Query| Gateway
    Gateway --> AuthSvc
    Gateway --> UserSvc
    Gateway --> OrderSvc
    subgraph IPC
        AuthSvc
        UserSvc
        OrderSvc
        Broker[(Kafka / RabbitMQ / Redis)]
    end
    OrderSvc --> Broker
    Broker -->|Subscription| Gateway
    Gateway -->|WebSocket| Client
