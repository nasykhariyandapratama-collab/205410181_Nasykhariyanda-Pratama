# PostgreSQL Streaming Replication (Docker Compose)

Contoh ini menunjukkan cara menyiapkan **PostgreSQL streaming replication** menggunakan **Docker Compose**, untuk mendemonstrasikan bagaimana sinkronisasi data terjadi melalui **WAL (Write-Ahead Log)** antara **primary database** dan **standby replica**.

---

## 1. Tujuan & Ringkasan

Tujuan:
- Menjalankan dua instance PostgreSQL (primary + replica)
- Mengaktifkan streaming WAL
- Menunjukkan bahwa perubahan pada primary otomatis muncul di replica

Konsep:
- **Primary**: menerima read + write
- **Replica**: read-only standby, selalu mengikuti data primary
- Replikasi dilakukan menggunakan **WAL streaming**

---

## 2. Arsitektur

```mermaid
graph LR
  Client --> Primary[(postgres-primary)]
  Primary -->|WAL stream| Replica[(postgres-replica)]
