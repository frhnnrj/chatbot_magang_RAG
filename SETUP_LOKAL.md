# Panduan Setup RAG_Anything di Laptop Lokal

Dokumen ini menjelaskan langkah-langkah untuk mengclone dan menjalankan RAG_Anything di laptop lokal dari GitHub.

---

## 📋 Prasyarat (Prerequisites)

Pastikan Anda sudah memiliki:

### 1. **Git**
   - Download: https://git-scm.com/
   - Verifikasi instalasi:
     ```powershell
     git --version
     ```

### 2. **Python 3.10+**
   - Download: https://www.python.org/downloads/
   - **Penting**: Centang "Add Python to PATH" saat instalasi
   - Verifikasi instalasi:
     ```powershell
     python --version
     ```

### 3. **PostgreSQL & Qdrant** (Database)
   - PostgreSQL: https://www.postgresql.org/download/
   - Qdrant: https://qdrant.tech/ (Docker recommended)
   
   **Atau gunakan Docker** (recommended untuk consistency):
   ```powershell
   docker run -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15
   docker run -p 6333:6333 qdrant/qdrant:latest
   ```

---

## 🚀 Langkah-Langkah Instalasi

### **Langkah 1: Clone Repository dari GitHub**

Buka PowerShell dan jalankan:

```powershell
# Navigasi ke folder tempat ingin menyimpan project
cd C:\Users\YourUsername\Documents

# Clone repository
git clone https://github.com/your-username/RAG_Anything.git

# Masuk ke folder project
cd RAG_Anything
```

**Catatan**: Ganti `https://github.com/your-username/RAG_Anything.git` dengan URL repository Anda.

---

### **Langkah 2: Buat Virtual Environment**

```powershell
# Buat virtual environment
python -m venv venv

# Activate virtual environment
# Di Windows PowerShell:
.\venv\Scripts\Activate.ps1

# Di Windows Command Prompt:
venv\Scripts\activate.bat

# Di Mac/Linux:
source venv/bin/activate
```

**Jika muncul error di PowerShell**, jalankan terlebih dahulu:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

### **Langkah 3: Install Dependencies**

Setelah virtual environment aktif, install semua package yang dibutuhkan:

```powershell
# Install dependencies dari requirements.txt
pip install -r requirements.txt

# Atau jika hanya ingin server dependencies:
pip install -r requirements_server.txt
```

**Expected output**: Semua packages terinstall tanpa error (~2-5 menit tergantung koneksi internet)

---

### **Langkah 4: Setup File .env**

#### **Opsi A: Copy dari .env.example (Recommended)**

```powershell
# Copy template env file
Copy-Item .env.example .env

# Edit file .env dengan text editor favorit (VS Code, Notepad++, dll)
# Atau gunakan PowerShell:
notepad .env
```

#### **Opsi B: Buat file .env manual**

Buat file `.env` di root folder project dengan isi:

```env
# ============================================
# OpenRouter LLM Configuration
# ============================================
OPENROUTER_API_KEY=your_openrouter_api_key_here
LLM_MODEL=qwen-2.5-72b-instruct
LLM_TEMPERATURE=0.5

# ============================================
# Embedding Model Configuration
# ============================================
EMBEDDING_MODEL=Qwen/Qwen2-1B
EMBEDDING_DEVICE=cpu

# ============================================
# OCR Configuration
# ============================================
ENABLE_OCR=true
OCR_BACKEND=easyocr
OCR_LANGUAGES=en,id

# ============================================
# DocString API Configuration (Optional)
# ============================================
DOCSTRING_API_KEY=your_docstring_api_key_here
ENABLE_DOCSTRING=false

# ============================================
# LightRAG Configuration
# ============================================
LIGHTRAG_WORK_DIR=./rag_storage
LIGHTRAG_LLM_MODEL_FUNC=gpt_model_func
LIGHTRAG_EMBEDDING_FUNC=embedding_model_func

# ============================================
# PostgreSQL Configuration
# ============================================
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=localhost
DB_PORT=5432
DB_NAME=raganything

# ============================================
# Qdrant Vector Database Configuration
# ============================================
QDRANT_HOST=localhost
QDRANT_PORT=6333
QDRANT_COLLECTION_NAME=rag_documents

# ============================================
# Processing Parameters
# ============================================
CHUNK_SIZE=1024
CHUNK_OVERLAP=100
MAX_CHUNK_DISPLAY=3
```

**Kustomisasi sesuai kebutuhan Anda**, terutama:
- `OPENROUTER_API_KEY`: Ambil dari https://openrouter.ai/
- `DB_HOST`, `DB_PORT`: Sesuaikan dengan setup database Anda
- `QDRANT_HOST`, `QDRANT_PORT`: Sesuaikan jika berbeda

---

### **Langkah 5: Setup Database**

#### **PostgreSQL**

```powershell
# Buat database (jika belum ada)
# Gunakan psql atau GUI tool (pgAdmin, Beekeper Studio)

psql -U postgres -c "CREATE DATABASE raganything;"
```

#### **Qdrant**

Jika menggunakan Docker:
```powershell
# Qdrant akan otomatis create collection saat app berjalan
# (koleksi dibuat oleh aplikasi di file db/qdrant_manager.py)
```

---

### **Langkah 6: Jalankan Server**

```powershell
# Pastikan virtual environment masih aktif
# (Jika belum, jalankan: .\venv\Scripts\Activate.ps1)

# Jalankan server
python -m uvicorn main_server:app --host 127.0.0.1 --port 8888 --reload
```

**Expected output**:
```
INFO:     Uvicorn running on http://127.0.0.1:8888
INFO:     Application startup complete
```

Buka browser: http://127.0.0.1:8888

---

### **Langkah 7: Test Aplikasi**

1. **Upload PDF**: Gunakan UI untuk upload file PDF
2. **Query**: Tanyakan sesuatu tentang dokumen yang sudah diupload
3. **Check Database**: Gunakan Beekeper Studio untuk lihat data di PostgreSQL

---

## 🔧 Troubleshooting

### **Error: ModuleNotFoundError: No module named 'xxx'**
```powershell
# Reinstall dependencies
pip install -r requirements.txt --force-reinstall
```

### **Error: Connection refused (localhost:5432)**
PostgreSQL tidak berjalan. Solusi:
```powershell
# Windows: Pastikan PostgreSQL service berjalan
# Atau gunakan Docker:
docker run -p 5432:5432 -e POSTGRES_PASSWORD=postgres postgres:15
```

### **Error: Connection refused (localhost:6333)**
Qdrant tidak berjalan. Solusi:
```powershell
# Gunakan Docker:
docker run -p 6333:6333 qdrant/qdrant:latest
```

### **Error: Virtual environment tidak aktif**
```powershell
# Activate kembali:
.\venv\Scripts\Activate.ps1
```

### **Error: "python: command not found"**
- Python belum ditambah ke PATH
- Solusi: Install ulang Python dan centang "Add Python to PATH"

### **Server berjalan tapi UI tidak load**
- Cek koneksi database di browser console (F12 → Network)
- Cek .env file apakah sudah benar
- Lihat terminal server untuk error messages

---

## 📁 Struktur Folder Setelah Setup

```
RAG_Anything/
├── venv/                    # Virtual environment (auto-created)
├── app/                     # Main application code
├── raganything/             # RAG logic
├── rag_storage/             # Storage untuk RAG embeddings
├── logs/                    # Log files
├── .env                     # Configuration file (JANGAN commit ke git!)
├── .env.example             # Template .env file (untuk reference)
├── requirements.txt         # Semua dependencies
├── requirements_server.txt  # Server-only dependencies
├── main_server.py          # Server entry point
└── ...
```

---

## ⚙️ Setup di Mesin Berbeda

Jika ingin setup di mesin lain yang berbeda (misalnya: laptop teman, server production):

1. **Laptop teman ingin pakai juga?**
   ```powershell
   # Di laptop teman, ulangi dari Langkah 1 (Clone Repository)
   ```

2. **Ingin deploy ke server cloud (AWS, GCP, Heroku)?**
   - Gunakan `requirements.txt`
   - Setup environment variables di cloud platform
   - Gunakan database cloud (Cloud SQL, AWS RDS, dll)
   - Lihat file `DEPLOYMENT_GUIDE.md` (jika ada)

---

## 📝 Checklist Sebelum Mulai

- [ ] Git sudah terinstall
- [ ] Python 3.10+ sudah terinstall
- [ ] PostgreSQL sudah berjalan
- [ ] Qdrant sudah berjalan (atau Docker siap)
- [ ] Akses ke GitHub repository
- [ ] Text editor untuk edit .env file (VS Code, Notepad++, dll)

---

## 💡 Tips Berguna

### Menggunakan Beekeper Studio (GUI Database)
```powershell
# Download: https://www.beekeeper-studio.io/
# Connect ke PostgreSQL dengan credentials di .env file
```

### Melihat Debug Logs
Debug messages akan tampil di terminal server:
```
[DEBUG] Query Type Detection: summary=False, comparison=False
[DEBUG] LLM Raw Response (245 chars): ...
```

### Update Dependencies di Masa Depan
```powershell
# Jika ada package baru
pip install -r requirements.txt

# Update single package
pip install --upgrade package_name
```

### Menyimpan Progress ke GitHub
```powershell
# Setelah membuat perubahan
git add .
git commit -m "Deskripsi perubahan"
git push origin main
```

---

## 📞 Support & Resources

- **GitHub Issues**: Laporkan bug di repository GitHub
- **Documentation**: Baca `FLOW_EXPLANATION.md` untuk memahami alur sistem
- **Architecture**: Lihat `docs/ARCHITECTURE_SERVER.md` untuk detail teknis

---

**Good luck! Happy deploying! 🚀**
