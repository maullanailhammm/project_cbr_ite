# Sistem Case-Based Reasoning (CBR) — Analisis Putusan UU ITE - Pidana Khusus ITE - PN Jakarta Timur

> **Nama:** Maullana Ilham Ar Rosyiid Mariandi
> 
> **Nim:** 202310370311093I
>
> **Kelas:** Penalaran Komputer B

Sistem CBR sederhana berbasis Python untuk mendukung analisis putusan pengadilan pidana khusus **Undang-Undang Informasi dan Transaksi Elektronik (UU ITE)** dari Pengadilan Negeri Jakarta Timur. Data bersumber dari Direktori Putusan Mahkamah Agung Republik Indonesia.

---

## Daftar Isi

1. [Struktur Proyek](#struktur-proyek)
2. [Persyaratan Sistem](#persyaratan-sistem)
3. [Instalasi](#instalasi)
4. [Menjalankan Pipeline End-to-End](#menjalankan-pipeline-end-to-end)
5. [Penjelasan Tiap Tahap](#penjelasan-tiap-tahap)
6. [Output yang Dihasilkan](#output-yang-dihasilkan)
7. [Contoh Perintah](#contoh-perintah)

---

## Struktur Proyek

```
project_cbr_ite/
│
├── data/
│   ├── pdf/                        # File PDF putusan asli (≥30 dokumen)
│   ├── raw/                        # Teks putusan hasil ekstraksi & cleaning
│   │   ├── case_001.txt
│   │   ├── case_002.txt
│   │   └── ...
│   ├── processed/
│   │   ├── cases.csv               # Representasi terstruktur semua kasus
│   │   └── cases.json              # Format JSON 
│   ├── eval/
│   │   ├── queries.json            # Query uji + ground truth
│   │   ├── retrieval_metrics.csv   # Metrik evaluasi retrieval
│   │   ├── prediction_metrics.csv  # Metrik evaluasi prediksi
│   │   └── performance_chart.png   # Visualisasi perbandingan model
│   ├── models/
│   │   ├── tfidf_vectorizer.joblib # TF-IDF vectorizer (Tahap 3)
│   │   ├── svm_model.joblib        # Model SVM (Tahap 3)
│   │   ├── nb_model.joblib         # Model Naive Bayes (Tahap 3)
│   │   ├── train_embeddings.npy    # IndoBERT embeddings case base (Tahap 3)
│   │   └── train_df.csv            # Index case base (Tahap 3)
│   └── results/
│       └── predictions.csv         # Hasil prediksi semua query (Tahap 4)
│
├── logs/
│   └── cleaning.log                # Log history pembersihan teks
│
├── notebooks/
│   ├── 01_case_base.ipynb           # Tahap 1: Scraping & Preprocessing
│   ├── 02_case_representation.ipynb # Tahap 2: Metadata & Feature Engineering
│   ├── 03_retrieval.ipynb           # Tahap 3: Case Retrieval (TF-IDF + IndoBERT)
│   ├── 04_predict.ipynb             # Tahap 4: Case Solution Reuse
│   └── 05_evaluation.ipynb          # Tahap 5: Model Evaluation
│
├── requirements.txt
└── README.md
```

---

## Persyaratan Sistem

| Komponen | Versi Minimum |
|---|---|
| Python | 3.10+ |
| RAM | 8 GB (16 GB direkomendasikan untuk IndoBERT) |
| Storage | ~3 GB (termasuk model IndoBERT) |
| OS | Windows 10/11, Ubuntu 20.04+, macOS 12+ |

---

## Instalasi

# instalasi pytesseract dan poppler untuk OCR fallback OCR fallback (karena putusan saya ada yang berupa PDF hasil scan):
```bash
- pytesseract butuh Tesseract OCR terinstall di sistem:
     Windows : https://github.com/UB-Mannheim/tesseract/wiki
     Linux   : sudo apt install tesseract-ocr
     Mac     : brew install tesseract
 - pdf2image butuh poppler:
     Windows : https://github.com/oschwartz10612/poppler-windows/releases
     Linux   : sudo apt install poppler-utils
     Mac     : brew install poppler
```
### 1. Clone Repository

```bash
git clone https://github.com/<username>/project_cbr_ite.git
cd project_cbr_ite
```

### 2. Buat Virtual Environment

```bash
# Windows
python -m venv .venv
.venv\Scripts\activate

# Linux / macOS
python3 -m venv .venv
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Jalankan Jupyter Notebook

```bash
jupyter notebook
```

Buka folder `notebooks/` dan jalankan setiap notebook secara berurutan.

---

## Menjalankan Pipeline End-to-End

Jalankan notebook **secara berurutan** dari Tahap 1 hingga Tahap 5.  
Setiap notebook menghasilkan artefak yang dibutuhkan oleh tahap berikutnya.

```
01_case_base.ipynb
      ↓  (data/raw/*.txt)
02_case_representation.ipynb
      ↓  (data/processed/cases.csv)
03_retrieval.ipynb
      ↓  (data/models/*, data/eval/queries.json)
04_predict.ipynb
      ↓  (data/results/predictions.csv)
05_evaluation.ipynb
      ↓  (data/eval/retrieval_metrics.csv, prediction_metrics.csv)
```

### Opsi: Jalankan via Command Line (nbconvert)

```bash
# Jalankan semua notebook sekaligus (dari folder notebooks/)
cd notebooks

jupyter nbconvert --to notebook --execute 01_case_base.ipynb --output 01_case_base.ipynb
jupyter nbconvert --to notebook --execute 02_case_representation.ipynb --output 02_case_representation.ipynb
jupyter nbconvert --to notebook --execute 03_retrieval.ipynb --output 03_retrieval.ipynb
jupyter nbconvert --to notebook --execute 04_predict.ipynb --output 04_predict.ipynb
jupyter nbconvert --to notebook --execute 05_evaluation.ipynb --output 05_evaluation.ipynb
```

> **Catatan:** Tahap 3 mengunduh model IndoBERT (~500 MB) dari HuggingFace pada saat pertama kali dijalankan. Pastikan koneksi internet tersedia.

---

## Penjelasan Tiap Tahap

### Tahap 1 — Membangun Case Base (`01_case_base.ipynb`)

Mengunduh dan membersihkan korpus putusan pengadilan.

- **Input:** File PDF di `data/pdf/`
- **Proses:** Ekstraksi teks (PyMuPDF + OCR fallback), hapus header/footer/watermark, normalisasi karakter
- **Output:** `data/raw/case_XXX.txt` (≥30 file), `logs/cleaning.log`

### Tahap 2 — Case Representation (`02_case_representation.ipynb`)

Merepresentasikan setiap putusan dalam struktur data terorganisir.

- **Input:** `data/raw/*.txt`
- **Proses:** Ekstraksi metadata (nomor perkara, tanggal, pasal, pihak), ekstraksi vonis & label, feature engineering (BoW, word count)
- **Output:** `data/processed/cases.csv`, `data/processed/cases.json`

**Kolom utama `cases.csv`:**

| Kolom | Keterangan |
|---|---|
| `case_id` | ID unik kasus |
| `no_perkara` | Nomor perkara pengadilan |
| `tanggal` | Tanggal putusan |
| `pasal` | Semua pasal yang disebut dalam dakwaan |
| `pasal_terbukti` | Pasal yang terbukti menurut amar putusan |
| `vonis_penjara` | Lama hukuman penjara |
| `label` | Label ringkas untuk evaluasi (dari pasal terbukti) |
| `ringkasan_fakta` | Ringkasan fakta persidangan |
| `argumen_hukum` | Inti amar putusan |

### Tahap 3 — Case Retrieval (`03_retrieval.ipynb`)

Membangun sistem pencarian kasus mirip menggunakan dua pendekatan.

- **Jalur A:** TF-IDF (`TfidfVectorizer`) + cosine similarity
- **Jalur B:** IndoBERT (`indobenchmark/indobert-base-p1`) embedding + cosine similarity
- **Split data:** 80% train (case base) / 20% test
- **Output:** Model tersimpan di `data/models/`, query uji di `data/eval/queries.json`

**Fungsi utama:**
```python
retrieve_tfidf(query: str, k: int = 5)    -> list[(case_id, score)]
retrieve_embedding(query: str, k: int = 5) -> list[(case_id, score)]
```

### Tahap 4 — Case Solution Reuse (`04_predict.ipynb`)

Menggunakan kasus lama sebagai dasar prediksi untuk kasus baru.

- **Algoritma 1 — Majority Vote:** pilih pasal yang paling banyak muncul di top-k
- **Algoritma 2 — Weighted Similarity:** akumulasi skor similarity per pasal

**Fungsi utama:**
```python
predict_outcome(
    query     : str,
    k         : int = 5,
    retrieval : str = "tfidf",    # "tfidf" | "embedding"
    strategy  : str = "weighted", # "majority" | "weighted"
) -> dict
```

**Contoh penggunaan:**
```python
hasil = predict_outcome(
    query="Terdakwa mengunggah konten hoaks di media sosial...",
    k=5,
    retrieval="embedding",
    strategy="weighted"
)
print(hasil["predicted_pasal"])    # Pasal yang diprediksi
print(hasil["predicted_solution"]) # Solusi lengkap (amar putusan)
print(hasil["top_k_case_ids"])     # Kasus referensi yang dipakai
```

### Tahap 5 — Model Evaluation (`05_evaluation.ipynb`)

Mengukur dan menganalisis performa retrieval & prediksi.

- **Metrik:** Accuracy, Precision (macro), Recall (macro), F1-score (macro)
- **Library:** `sklearn.metrics`
- **Visualisasi:** Tabel perbandingan model + bar chart
- **Error analysis:** Identifikasi kasus gagal + rekomendasi perbaikan

---

## Output yang Dihasilkan

| File | Keterangan |
|---|---|
| `data/raw/*.txt` | ≥30 teks putusan bersih |
| `data/processed/cases.csv` | Representasi terstruktur semua kasus |
| `data/eval/queries.json` | 6 query uji + ground truth pasal |
| `data/models/*.joblib` | TF-IDF vectorizer, SVM, Naive Bayes |
| `data/models/train_embeddings.npy` | IndoBERT embeddings case base |
| `data/results/predictions.csv` | Prediksi 6 query × 4 kombinasi metode |
| `data/eval/retrieval_metrics.csv` | Metrik Accuracy/Precision/Recall/F1 per model |
| `data/eval/prediction_metrics.csv` | Metrik prediksi pasal per strategi |
| `data/eval/performance_chart.png` | Visualisasi perbandingan performa model |
| `logs/cleaning.log` | Log history pembersihan teks Tahap 1 |

---

## Contoh Perintah

### Cek semua file output sudah ada

```bash
# Windows PowerShell
Get-ChildItem -Recurse data\ | Select-Object FullName, Length

# Linux / macOS
find data/ -type f | sort
```

### Prediksi kasus baru secara interaktif

Buka `notebooks/04_predict.ipynb`, pastikan semua sel sudah dijalankan, lalu tambahkan sel baru:

```python
hasil = predict_outcome(
    query="""
    Terdakwa pada bulan Maret 2022 mengakses akun media sosial
    milik korban tanpa izin dan menyebarkan informasi palsu
    yang merugikan korban secara materil dan imateril.
    """,
    k=5,
    retrieval="embedding",  # atau "tfidf"
    strategy="weighted"     # atau "majority"
)

print("Pasal yang diprediksi :", hasil["predicted_pasal"])
print("Solusi (amar putusan) :", hasil["predicted_solution"])
print("Referensi kasus       :", hasil["top_k_case_ids"])
```

### Install ulang semua dependencies

```bash
pip install --upgrade -r requirements.txt
```

---

## requirements.txt

```
pandas>=2.0.0
numpy>=1.24.0
scikit-learn>=1.3.0
matplotlib>=3.7.0
seaborn>=0.12.0
jupyter>=1.0.0
nbformat>=5.9.0
PyMuPDF>=1.23.0
pytesseract>=0.3.10
Pillow>=10.0.0
transformers>=4.35.0
torch>=2.0.0
sentencepiece>=0.1.99
joblib>=1.3.0
tqdm>=4.65.0
python-Levenshtein>=0.21.0
```
