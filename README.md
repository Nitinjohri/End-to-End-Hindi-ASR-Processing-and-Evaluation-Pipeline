# 🎙️ End-to-End Hindi ASR Processing and Evaluation Pipeline

[![Python 3.8+](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-Framework-EE4C2C?logo=pytorch&logoColor=white)](https://pytorch.org/)
[![HuggingFace](https://img.shields.io/badge/HuggingFace-Transformers-FFD21E?logo=huggingface&logoColor=black)](https://huggingface.co/)
[![Whisper](https://img.shields.io/badge/OpenAI-Whisper--small-412991?logo=openai&logoColor=white)](https://github.com/openai/whisper)
[![Colab Ready](https://img.shields.io/badge/Google%20Colab-Ready-F9AB00?logo=googlecolab&logoColor=white)](https://colab.research.google.com/)

A comprehensive suite of **four Jupyter notebooks** for Hindi Automatic Speech Recognition (ASR) — covering **fine-tuning**, **evaluation**, **disfluency detection**, **spelling classification**, and **lattice-based WER analysis**.

> 🚀 **Designed for Google Colab** (GPU runtime recommended for fine-tuning)

---

## 📑 Table of Contents

- [Notebooks Overview](#-notebooks)
- [Results & Outputs](#-results--outputs)
- [Quick Start](#-quick-start)
- [Dataset Format](#-dataset-format)
- [Tech Stack](#️-tech-stack)
- [Project Structure](#-project-structure)
- [License](#-license)
- [Contributing](#-contributing)

---

## 📓 Notebooks

### 1. Whisper Hindi Fine-Tuning (`whisper_hindi_finetuning.ipynb`)

End-to-end pipeline for fine-tuning **OpenAI's Whisper-small** on Hindi speech data.

| Step | Description |
|------|-------------|
| 🔧 Setup | Install dependencies & configure model/data paths |
| 📥 Data Ingestion | Load Excel dataset, auto-fix broken GCS URLs |
| 🔊 Audio Preprocessing | Resample to 16 kHz mono WAV, filter by duration (1–30s) |
| ✍️ Text Normalization | Unicode NFC, punctuation cleanup, Devanagari preservation |
| ✅ Quality Checks | Remove corrupted audio, empty transcriptions, wrong language |
| 📦 Dataset Formatting | HuggingFace `Dataset` with log-mel spectrograms & tokenized labels |
| 📏 Baseline Evaluation | Pre-training WER on validation split |
| 🏋️ Fine-Tuning | Seq2Seq training with AdamW, FP16, early stopping |
| 📊 Post-Training | WER comparison & model export to HuggingFace Hub |

---

### 2. Disfluency Detection & Audio Clipping (`disfluency_detection.ipynb`)

Detects disfluencies in Hindi/English transcriptions and clips the corresponding audio segments.

**Disfluency Categories:**

| Category | Detection Method | Examples |
|----------|-----------------|----------|
| Fillers | Dictionary lookup | `uh`, `umm`, `मतलब`, `तो` |
| Repetitions | Regex pattern matching | `मैं मैं`, `वो वो` |
| False Starts | Sentence-level pattern analysis | `मैं कल… मतलब परसों गया` |
| Prolongations | Elongated character detection | `सोऽऽऽ`, `आआआ`, `soooo` |
| Hesitations | Duration anomaly detection | Short/abnormal duration segments |

**Pipeline:** `Load Excel` → `Fix URLs` → `Download Transcription JSONs` → `Text Preprocessing` → `Disfluency Detection` → `Audio Clipping` → `CSV Export`

---

### 3. Hindi Spelling Classification (`spelling_classification_pipeline.ipynb`)

Multi-layer pipeline to classify **~175,000 Hindi words** as correct or incorrect spelling.

<details>
<summary><strong>📋 13-Step Pipeline (click to expand)</strong></summary>

| Step | Description |
|------|-------------|
| 1 | Data Collection & URL Fixing |
| 2 | Rebuild Vocabulary from FT Dataset |
| 3 | Text Normalization (Unicode NFC, invisible chars, nukta variants) |
| 4 | High-Frequency Confidence Rule |
| 5 | Protected Core Vocabulary (~200+ words) |
| 6 | Dictionary-Based Validation (~400+ words) |
| 7 | Orthographic Rule Checking (invalid matra patterns, repetitions) |
| 8 | Edit-Distance Typo Detection (Levenshtein ≤ 2) |
| 9 | English-in-Devanagari Loanword Handling (~100+ words) |
| 10 | Conservative Default Policy |
| 11 | Final Classification Logic |
| 12 | Output Generation (Excel export) |
| 13 | Final Count Reporting |

</details>

---

### 4. Lattice-Based WER Pipeline (`lattice_wer_pipeline.ipynb`)

Builds a **confusion lattice** from multiple ASR model outputs to compute a fairer WER metric.

**Pipeline:**
1. Load & preprocess transcriptions from **6 ASR models** + **1 human reference**
2. Word-level multi-sequence alignment (Needleman-Wunsch)
3. Build confusion lattice per audio segment
4. Model consensus logic (majority vote ≥ 3/6 models)
5. Reference correction (override human errors via model agreement)
6. Dual WER computation (Standard vs Lattice-corrected)
7. Structured reporting with charts

---

## � Results & Outputs

### Whisper Hindi Fine-Tuning Results

The fine-tuned Whisper-small model shows significant improvement over the baseline:

![Whisper Hindi Fine-Tuning Output](outputs/whisper_hindi_finetuning_output.png)

---

### Lattice-Based WER Pipeline Results

The lattice-based approach produces fairer WER metrics by leveraging multi-model consensus to correct human reference errors:

![Lattice WER Pipeline Output](outputs/lattice_wer_pipeline_output.png)

---

### Spelling Classification Results

The spelling classification pipeline processes ~175,000 Hindi words and exports detailed results:

📄 **Output file:** [`spelling_classification_output.xlsx`](outputs/spelling_classification_output.xlsx)

---

### Disfluency Detection Results

The disfluency detection pipeline identifies and categorizes speech disfluencies across all audio segments:

📄 **Output file:** [`disfluency_results.csv`](outputs/disfluency_results%20(1).csv)

---

## �🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/Nitinjohri/End-to-End-Hindi-ASR-Processing-and-Evaluation-Pipeline.git
cd End-to-End-Hindi-ASR-Processing-and-Evaluation-Pipeline
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Run on Google Colab

1. Upload the desired `.ipynb` notebook to [Google Colab](https://colab.research.google.com/)
2. Set runtime to **GPU** (`Runtime → Change runtime type → T4 GPU`) for fine-tuning
3. Upload your Excel dataset when prompted
4. Run all cells sequentially

---

## 📊 Dataset Format

All notebooks expect an **Excel file (`.xlsx`)** with columns such as:

| Column | Description |
|--------|-------------|
| `recording_id` | Unique identifier for each recording |
| `audio_url` / `audio_path` | URL or path to the audio file |
| `transcription_url` | URL to the transcription JSON |
| `transcription` | Hindi text transcription |

> **Note:** Broken JoshTalks GCS URLs are automatically fixed by each notebook.

---

## 🛠️ Tech Stack

| Category | Technologies |
|----------|-------------|
| **Language** | Python 3.8+ |
| **Framework** | PyTorch, HuggingFace Transformers |
| **ASR Model** | OpenAI Whisper-small |
| **Audio Processing** | librosa, pydub, torchaudio, soundfile |
| **NLP** | jiwer (WER), python-Levenshtein (edit distance) |
| **Data** | pandas, openpyxl |
| **Visualization** | matplotlib |

---

## 📁 Project Structure

```
End-to-End-Hindi-ASR-Processing-and-Evaluation-Pipeline/
├── 📄 README.md
├── 📄 requirements.txt
├── 📄 .gitignore
│
├── 📁 codes/
│   ├── 📓 whisper_hindi_finetuning.ipynb          # Whisper fine-tuning pipeline
│   ├── 📓 disfluency_detection.ipynb              # Disfluency detection & audio clipping
│   ├── 📓 spelling_classification_pipeline.ipynb   # Hindi spelling classification
│   └── 📓 lattice_wer_pipeline.ipynb              # Lattice-based WER computation
│
└── 📁 outputs/
    ├── 🖼️ whisper_hindi_finetuning_output.png     # Fine-tuning results visualization
    ├── 🖼️ lattice_wer_pipeline_output.png         # Lattice WER comparison chart
    ├── 📊 spelling_classification_output.xlsx      # Spelling classification results
    └── 📊 disfluency_results.csv                  # Disfluency detection results
```

---

## 📝 License

This project is for educational and research purposes.

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome! Feel free to open an issue or submit a pull request.

---

<p align="center">
  Made with ❤️ for Hindi ASR Research
</p>
