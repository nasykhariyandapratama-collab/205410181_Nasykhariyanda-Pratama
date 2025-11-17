# Jawaban Ujian: CAP, BASE, GraphQL & PostgreSQL Streaming Replication

Repository ini berisi ringkasan jawaban dari 3 soal:

1. Penjelasan **Teorema CAP** dan **BASE** serta keterkaitannya (dengan contoh).
2. Keterkaitan **GraphQL** dengan komunikasi antar proses pada sistem terdistribusi (beserta diagram).
3. Contoh **PostgreSQL Streaming Replication** menggunakan **Docker / Docker Compose** untuk menjelaskan sinkronisasi.

---

## 1. Teorema CAP dan BASE + Keterkaitannya
- Teorema CAP adalah prinsip yang memaksa sistem terdistribusi memilih antara Konsistensi (CP) atau Ketersediaan (AP) saat terjadi partisi jaringan (P), di mana filosofi BASE (Basically Available, Soft state, Eventually consistent) adalah model yang mengimplementasikan pilihan AP dengan mengorbankan konsistensi instan demi ketersediaan.

Lokasi jawaban:
[`soal%20no%201/cap.MD`](soal%20no%201/cap.MD)
---

## 2. GraphQL & Komunikasi Antar Proses di Sistem Terdistribusi

### 2.1. GraphQL
- Sementara itu, GraphQL bukanlah mekanisme Komunikasi Antar Proses (IPC) seperti HTTP atau gRPC, melainkan sebuah query language API yang bertindak sebagai lapisan abstraksi; ia menggunakan mekanisme IPC tersebut untuk menerjemahkan satu kueri dari klien menjadi beberapa panggilan ke berbagai microservice di backend dan menggabungkan hasilnya.

Lokasi jawaban:
[`soal%20no%202/GraphQL.md`](soal%20no%202/GraphQL.md)

## 3. PostgreSQL Streaming Replication
- Untuk membuat streaming replication sinkron di PostgreSQL menggunakan Docker Compose, Anda perlu mendefinisikan dua layanan: primary dan standby. Layanan primary dikonfigurasi melalui postgresql.conf untuk mengaktifkan replikasi (wal_level = replica) dan memaksakan mode sinkron (synchronous_commit = on serta synchronous_standby_names). Layanan standby menggunakan skrip entrypoint khusus untuk menunggu primary siap, lalu menjalankan pg_basebackup untuk mengkloning data, dan terakhir memulai sebagai standby dengan application_name yang sesuai. Hasilnya adalah penyiapan di mana primary akan menahan (hang) transaksi COMMIT jika standby mati, ini membuktikan bahwa sinkronisasi data (konsistensi) diprioritaskan di atas ketersediaan (prinsip CP dari Teorema CAP).
  
Lokasi jawaban:
[`soal%20no%203/PostgreSQL%20Streaming%20Replication.md`](soal%20no%203/PostgreSQL%20Streaming%20Replication.md)
