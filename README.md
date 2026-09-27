[onboarding_buddy_flow_readme.md](https://github.com/user-attachments/files/32695768/onboarding_buddy_flow_readme.md)
# IBM_hackton_Langflow_capston
A Retrieval-Augmented Generation (RAG) project utilizing Langflow and Astra DB Vectorize Collection to automate the processing of employee onboarding PDF documents.
# Onboarding Buddy RAG Flow 🚀

Repositori ini berisi konfigurasi *flow* **Langflow** untuk aplikasi *Retrieval-Augmented Generation (RAG)* yang dirancang khusus untuk memproses dokumen **Panduan Onboarding Karyawan**. Sistem ini memungkinkan pengguna untuk berinteraksi secara cerdas dengan dokumen kebijakan perusahaan menggunakan integrasi basis data vektor **Astra DB**.

---

## 🏗️ Penjelasan Arsitektur Flow

Flow ini dirancang menggunakan pendekatan modern menggunakan fitur *built-in embedding* (*Vectorize Collection*) dari Astra DB untuk memangkas kerumitan konfigurasi eksternal. Komponen utamanya terdiri dari:

1. **Read File Node**: Berfungsi untuk membaca dan mengekstrak teks mentah dari file PDF dokumen onboarding (`Panduan Onboarding Karyawan.pdf`).
2. **Split Text Node**: Memecah dokumen panjang menjadi potongan-potongan kecil (*chunks*) dengan ukuran *Chunk Size* (misalnya `300`) dan *Chunk Overlap* (`30`) agar relevansi konteks pencarian lebih akurat.
3. **Astra DB Node (Vectorize Collection)**: Menyimpan dan mengelola data vektor. Berbeda dengan pendekatan konvensional, *flow* ini memanfaatkan fitur *Vectorize* bawaan Astra DB di mana proses *embedding* (vektorisasi teks) ditangani secara otomatis langsung di sisi server database, sehingga tidak memerlukan node model *embedding* terpisah di Langflow.

---

## ⚙️ Cara Menjalankan (How to Run)

Ikuti langkah-langkah berikut untuk menjalankan *flow* ini di lingkungan lokal Anda:

1. **Persiapan Astra DB**:
   * Login ke [Astra Portal (DataStax)](https://astra.datastax.com/).
   * Buat Database baru (contoh: `Knowledge_agent`).
   * Buat **Collection** baru dengan mengaktifkan opsi *Vectorize* (menggunakan model integrasi bawaan seperti NVIDIA atau provider yang tersedia). Pastikan tipe koleksi diset ke *Vectorize Collection*.
2. **Setup Langflow**:
   * Pastikan Langflow sudah terinstal di komputer Anda (`pip install langflow`).
   * Jalankan Langflow melalui terminal:
     ```bash
     langflow run
     ```
   * Buka antarmuka Langflow di browser (biasanya di `http://localhost:7860`).
3. **Import & Konfigurasi Flow**:
   * Impor file JSON *Onboarding Buddy Flow* ke dalam *workspace* Langflow.
   * Masukkan **Astra DB Application Token** Anda pada node Astra DB.
   * Masukkan nama Database dan Collection yang sudah dibuat (contoh Collection: `data_onboarding`).
   * **Catatan Penting**: Untuk *Vectorize Collection*, biarkan parameter *Content Field* kosong agar sistem Astra mengaturnya secara otomatis.
4. **Run Ingestion**:
   * Unggah file PDF panduan onboarding pada node *Read File*.
   * Klik tombol *Run* pada node Astra DB untuk memulai proses *ingestion* data ke dalam database vektor.

---

## 🛠️ Catatan Troubleshooting: 2 Error Utama & Solusinya

Selama pengembangan *flow* ini, kami menghadapi dua tantangan teknis utama yang berhasil diselesaikan sebagai berikut:

### 1. Error: `zip() argument 2 is shorter than argument 1`
* **Penyebab**: Terjadi ketidakcocokan (*mismatch*) jumlah data antara potongan teks (*chunks*) dan vektor yang dihasilkan saat menggunakan model *embedding* eksternal (seperti Gemini). Beberapa *chunk* kosong atau karakter khusus di dalam PDF sering kali dilewati (*skipped*) secara diam-diam oleh API model, yang membuat fungsi `zip()` di Python mengalami *crash*.
* **Solusi**: Beralih menggunakan **Astra DB Vectorize Collections**. Dengan menyerahkan proses vektorisasi langsung ke server Astra DB, kita memotong perantara *wrapper embedding* di Langflow sehingga masalah sinkronisasi jumlah data ini bisa teratasi total.

### 2. Error: `content_field is not configurable for vectorize collections` & *Dimension Mismatch*
* **Penyebab**: 
  * Mencoba mengubah parameter `content_field` secara manual pada koleksi yang sudah diatur otomatis sebagai *vectorize*.
  * Ketidakcocokan ukuran dimensi vektor (misalnya mencoba mencampur model 768 dimensi dengan skema database yang berbeda).
* **Solusi**: 
  * Mengosongkan parameter kustom seperti *Content Field* di panel konfigurasi Astra DB Langflow agar sistem backend Astra mengaturnya secara otomatis.
  * Memastikan pembuatan koleksi baru di Astra Portal menggunakan mode *Vectorize* murni tanpa konfigurasi manual dimensi eksternal yang bentrok.

---
