## 1. Teorema CAP dan BASE serta keterkaitannya

### 1.1. Teorema CAP

CAP theorem (Eric Brewer) menyatakan bahwa dalam sistem terdistribusi, ketika terjadi **network partition (P)**, kita tidak bisa mendapatkan **Consistency (C)** dan **Availability (A)** secara bersamaan. Kita harus memilih:

- **Consistency (C)**  
  Semua client melihat data yang sama pada waktu yang sama. Setelah sebuah write sukses, semua read berikutnya mengembalikan nilai terbaru.

- **Availability (A)**  
  Setiap request ke sistem akan mendapatkan response (tidak “diam”/timeout terus), meskipun isinya bisa jadi data lama.

- **Partition Tolerance (P)**  
  Sistem tetap beroperasi meski ada pemisahan jaringan antar node (sebagian node tidak bisa saling komunikasi).

Saat terjadi partition, pilihan praktisnya:

- **CP (Consistency + Partition tolerance)**  
  Sistem lebih memilih konsistensi yang kuat, tapi mungkin mengorbankan ketersediaan (beberapa request akan gagal/ditolak).

- **AP (Availability + Partition tolerance)**  
  Sistem tetap melayani request (lebih available), tapi mengizinkan inkonsistensi sementara (eventual consistency).

**Contoh:**

- Sistem **perbankan / saldo rekening** → cenderung **CP**  
  Jika jaringan bermasalah, lebih baik transaksi ditolak daripada saldo nasabah jadi berantakan.

- **Feed sosial / katalog produk** → sering **AP**  
  User tetap bisa membuka feed meskipun beberapa data (jumlah like, stok) belum paling up-to-date.

---

### 1.2. BASE (Basically Available, Soft state, Eventual consistency)

BASE adalah prinsip yang sering dipakai di sistem yang memilih **AP** pada CAP.

- **Basically Available**  
  Sistem berusaha selalu memberikan respons, walaupun data mungkin belum sepenuhnya konsisten.

- **Soft State**  
  State bisa berubah seiring waktu meskipun tidak ada input baru dari client, misalnya karena replikasi/propagasi di background.

- **Eventual Consistency**  
  Jika tidak ada lagi write baru, pada akhirnya semua node akan konvergen ke nilai data yang sama.

**Contoh:**

- **Sistem jumlah like** di media sosial:  
  Setelah user menekan like, beberapa node mungkin belum menerima update, jadi jumlah like bisa berbeda antar user untuk sementara. Setelah replikasi selesai, nilainya akan sama.

---

### 1.3. Keterkaitan CAP dan BASE

- CAP menjelaskan **batas teoretis**: dalam kondisi partition, kita tidak bisa punya C dan A sekaligus.
- BASE adalah **pendekatan praktis** saat kita memilih **Availability (A)** dan **Partition Tolerance (P)** → artinya kita *mengorbankan konsistensi kuat* dan menggantinya dengan **eventual consistency**.
- Jadi, BASE pada dasarnya adalah cara implementasi untuk sistem **AP** menurut CAP.

**Contoh kombinasi di dunia nyata:**

#### 🛒 Contoh: Sistem Stok E-commerce saat Flash Sale

Contoh yang pernah saya gunakan (atau sering ditemui) adalah **sistem manajemen stok pada platform *e-commerce*** saat *flash sale*.

* **Skenario:** Ada 10 unit "Produk A" yang dijual. Ribuan pengguna mencoba membelinya secara bersamaan. Server-server *database replica* tersebar di berbagai wilayah.
* **Pilihan Sistem (AP/BASE):**
    * Platform *e-commerce* tidak ingin *website*-nya *down* atau menampilkan *error* "server sibuk" saat *flash sale*. Mereka harus memilih **Availability (A)**.
    * Mereka tahu jaringan bisa lambat atau putus antar *data center* (*Partition Tolerance*).
    * Oleh karena itu, mereka terpaksa mengorbankan **Consistency (C)**.
* **Implementasi BASE dalam Aksi:**
    1.  **Basically Available:** *Website* tetap merespons. 100 pengguna mungkin berhasil memasukkan "Produk A" ke keranjang mereka, meskipun stok aslinya hanya 10. Sistem selalu tersedia untuk menerima permintaan "masukkan ke keranjang".
    2.  **Soft State:** Status "Produk A di keranjang Anda" adalah *soft state*. Sistem belum benar-benar mengalokasikan stok itu untuk Anda. Itu hanya catatan *intent* (niat) Anda untuk membeli.
    3.  **Eventually Consistent:** Saat pengguna mencoba *checkout*, sistem akan melakukan pemeriksaan ulang ke *database* utama (sumber kebenaran). Pada titik ini, hanya 10 pengguna pertama yang akan berhasil, sementara 90 lainnya akan mendapatkan pesan "Maaf, stok habis" di halaman *checkout*. Sistem pada akhirnya menjadi konsisten (stok 0) setelah semua transaksi diproses.

---
