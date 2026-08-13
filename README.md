## Chatbot LLM RAG Database PostgreSQL dengan Open WebUI (Text-to-SQL Pipeline)

Pipeline chatbot dengan **Open WebUI** yang mengintegrasikan LlamaIndex, LLM (via Groq), dan PostgreSQL untuk mengubah pertanyaan bahasa natural menjadi *query* SQL (Text-to-SQL). Pipeline ini dirancang khusus untuk membaca `dataset_pembangunan` dalam PostgreSQL dan dilengkapi dengan agen validator untuk memastikan jawaban bebas dari halusinasi.

## ✨ Fitur Utama

- **Smart Column Filtering**: Mampu memilih beberapa kolom yang relevan (`SELECT a, b, c FROM d`) secara otomatis berdasarkan konteks pertanyaan.
- **Entity Disambiguation**: Memisahkan dan mengelompokkan data secara akurat jika terdapat nama entitas yang memiliki awalan mirip (misal: membedakan berbagai "Dinas" atau "Balai") menggunakan pengelompokan pada kolom `unor`.
- **Flexible Text Matching**: Secara otomatis menggunakan klausa `ILIKE` alih-alih `=` pada SQL untuk pencarian teks yang lebih fleksibel dan tidak *case-sensitive*.
- **Auto-Validation System**: Dilengkapi dengan sistem validasi *looping* (hingga 3 percobaan) menggunakan model `gemma2-9b-it` untuk memastikan jawaban akhir benar-benar bersumber dari *database*, bukan karangan LLM.
- **Transparansi Query (Sitasi)**: Menampilkan *query* SQL asli yang dieksekusi secara langsung di antarmuka Open WebUI sebagai bentuk referensi/sitasi bagi pengguna.

## 🛠️ Prasyarat (Prerequisites)

Sebelum menjalankan pipeline ini, pastikan sistem operasi Windows Anda sudah terinstal:
1. **Docker Desktop** (Pastikan WSL2 backend sudah aktif).
2. **PostgreSQL** (Jika ingin menjalankan *database* di luar Docker, atau gunakan PostgreSQL di dalam Docker sesuai langkah di bawah).
3. Akun dan API Key aktif dari **Groq** (menggunakan model `llama3-70b-8192` dan `gemma2-9b-it`).

## 🚀 Proses Instalasi & Konfigurasi

### 1. Menjalankan Environment dengan Docker
Jalankan Open WebUI di dalam Docker agar berada di satu lingkungan jaringan yang sama dengan *host* (menggunakan `host.docker.internal` untuk menjembatani koneksi ke PostgreSQL lokal).

Buka Command Prompt atau PowerShell, lalu jalankan perintah berikut:

```bat
docker run -d -p 3030:8080 ^
  --add-host=host.docker.internal:host-gateway ^
  -v open-webui:/app/backend/data ^
  --name open-webui1 --restart always ^
  -e DATABASE_URL="postgresql://postgres:1234@host.docker.internal:5432/film" ^
  ghcr.io/open-webui/open-webui:main
```

### 2. Update Database PostgreSQL
Pastikan *database* PostgreSQL Anda sudah berjalan dan tabel `dataset_pembangunan` sudah terisi dengan data yang relevan. 

* **Host**: `host.docker.internal` (jika menggunakan docker compose di atas) atau `localhost`
* **Port**: `5432`
* **User**: `postgres`
* **Password**: `1234`
* **Database Target**: `bangun` (Catatan: pastikan nama *database* di dalam script sesuai dengan *database* Anda).

### 3. Pemasangan Script di Open WebUI
1. Buka Open WebUI di *browser* Anda (`http://localhost:3030`).
2. Masuk ke menu **Workspace** > **Functions**
3. Buat *Function* baru dengan mengeklik tombol **+ (Add)**.
4. Salin (`Copy`) seluruh kode Python pada file Main, lalu tempel (`Paste`) ke dalam editor teks yang disediakan.
5. Simpan dan aktifkan pipeline.
6. Pipeline `Magang_Chatbot` kini siap digunakan di antarmuka obrolan utama.

## 🧩 Teknologi yang Digunakan
- **LlamaIndex**: Sebagai *framework* utama untuk orkestrasi Text-to-SQL.
- **SQLAlchemy**: Untuk manajemen koneksi *database* ke PostgreSQL.
- **HuggingFace Embeddings**: Menggunakan `sentence-transformers/all-MiniLM-L6-v2` untuk *vector store/retrieval*.
- **Groq API**: Mesin inferensi berkecepatan tinggi untuk model Llama 3 dan Gemma 2.
- **OpenAI Python SDK**: Digunakan sebagai klien untuk memanggil endpoint Groq.
