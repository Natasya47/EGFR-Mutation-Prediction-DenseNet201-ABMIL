# EGFR Mutation Prediction: DenseNet201 + ABMIL

Prediksi status mutasi EGFR (Mutant atau Wild-Type) pada lung adenocarcinoma (TCGA-LUAD) dari citra H&E Whole Slide Image (WSI), menggunakan DenseNet201 (frozen) sebagai ekstraktor fitur dan Attention-Based Multiple Instance Learning (ABMIL) sebagai agregator tingkat slide.

> Program ini untuk keperluan penelitian dan dokumentasi, bukan alat diagnosis atau keputusan klinis.

## Isi Repositori

| Berkas | Fungsi |
|---|---|
| `TAHAP_PRAPROSES.ipynb` | Unduh dan seleksi WSI dari GDC, pembagian data, deteksi jaringan, ekstraksi patch, dan normalisasi warna Macenko |
| `FINALL_DENSENET201+ABMIL.ipynb` | Ekstraksi fitur DenseNet201, validasi silang, pengujian hold-out, dan analisis tambahan |

## Data

- 200 WSI diagnostik TCGA-LUAD (50 Mutant dan 150 Wild-Type, satu slide per pasien) dari [GDC Data Portal](https://portal.gdc.cancer.gov/).
- Label mutasi EGFR dari cBioPortal (TCGA PanCancer Atlas 2018), disimpan dalam berkas `EGFR.xlsx` (tidak disertakan di repositori).
- Pembagian data per pasien: 160 kasus latih/validasi dan 40 kasus hold-out (seed 42).

## Alur Singkat

1. **Praproses** (`TAHAP_PRAPROSES.ipynb`, server dengan CPU multi-core): Download WSI, Split Dataset, Tissue Detection, Patch Extraction (224 × 224 piksel, 40×), Macenko.
2. **Pemodelan** (`FINALL_DENSENET201+ABMIL.ipynb`, Google Colab dengan GPU): ekstraksi fitur 1.920 dimensi per patch, validasi silang lima lipatan, pelatihan model final, dan pengujian hold-out satu kali.

## Cara Menjalankan

1. Unduh repositori:

   ```
   git clone https://github.com/Natasya47/EGFR-Mutation-Prediction-DenseNet201-ABMIL.git
   ```

2. **Praproses**: pasang OpenSlide (Ubuntu: `sudo apt install openslide-tools`) dan pustaka Python:

   ```
   pip install notebook numpy pandas openpyxl requests opencv-python-headless openslide-python scikit-image matplotlib
   ```

   Letakkan `EGFR.xlsx` di folder kerja, buka `TAHAP_PRAPROSES.ipynb`, lalu jalankan lima sel secara berurutan. Unggah folder hasil Macenko ke Google Drive.

3. **Pemodelan**: unggah `FINALL_DENSENET201+ABMIL.ipynb` ke Google Drive dan buka dengan Google Colab (GPU A100 dengan High-RAM disarankan). Bila pustaka belum tersedia:

   ```
   !pip install timm statsmodels huggingface_hub
   ```

   Isi `DATA_ROOT` pada Sel 2, lalu jalankan sel dari atas ke bawah. Biarkan `KONFIRMASI_HOLDOUT = False` sampai validasi silang selesai; ubah menjadi `True` hanya untuk menjalankan hold-out (Sel 12), yang cukup dijalankan sekali.

4. **Analisis tambahan** (Sel 14 sampai 35) bersifat opsional. Pembanding UNI memerlukan token `HF_TOKEN` dan persetujuan lisensi model di HuggingFace.

## Hasil Utama

| Ukuran | Nilai |
|---|---|
| AUROC validasi silang 5 lipatan | 0,827 ± 0,089 |
| AUROC hold-out 40 WSI | 0,713 (CI 95% bootstrap 0,502–0,914) |
| Sensitivitas / spesifisitas (ambang 0,5) | 60,00% (6 dari 10) / 80,00% (24 dari 30) |

Hold-out hanya memuat 10 kasus Mutant sehingga ketidakpastiannya besar; hasil dibaca sebagai sinyal awal.

Hasil lengkap program yang sudah dijalankan: [Google Drive](https://drive.google.com/drive/folders/1GCs4Temrh8msLWRrR-2YODpAjYisohRs?usp=sharing)

## Penulis

Natasya Haidar Romadhoni, Program Studi Sistem Informasi, Universitas Trunojoyo Madura.
