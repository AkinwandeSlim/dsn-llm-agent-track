# DSN AI Bootcamp 2026 — LLM / Agent Track

**Multilingual Topic Classification & Headline Generation for Nigerian Languages**

[![Kaggle Competition](https://img.shields.io/badge/Kaggle-Competition-blue?logo=kaggle)](https://www.kaggle.com/competitions/dsn-bootcamp-hackathon-2026-llm-agent-track)
[![Python](https://img.shields.io/badge/Python-3.11-green?logo=python)](https://python.org)
[![License](https://img.shields.io/badge/License-Competition%20Rules-orange)]()

One model, two tasks, four languages: given a news article in **Hausa**, **Igbo**, **Yoruba**, or **Nigerian Pidgin**, assign a topic label and write a headline in the article's own language.

---

## 🏆 Results

| Metric | Score |
|--------|-------|
| **Dev Final Score** | **0.7226** |
| Macro-F1 (Task A — Topic Classification) | 0.8948 |
| GenScore (Task B — Headline Generation) | 0.5503 |
| Parameters | 429M (under 1B limit) |
| Training time | 195 min on single T4 GPU |

### Scoring Formula

```
Final Score = 0.5 × Macro-F1 + 0.5 × GenScore
GenScore    = 0.5 × ROUGE-L  + 0.5 × BERTScore (multilingual)
```

---

## 📋 Model Card

| | |
|---|---|
| **Backbone** | [`castorini/afriteva_v2_base`](https://huggingface.co/castorini/afriteva_v2_base) — 429M params |
| **Pretraining data** | Wura, a curated multilingual African-language corpus |
| **Fine-tuning** | LoRA adapter (r=16, alpha=32) on q, v attention projections |
| **Task format** | Multitask seq2seq via task prefixes (`classify`/`headline`) |
| **Optimizer** | Adafactor with linear warmup + decay schedule |
| **Precision** | fp32 (T5 overflows in fp16 — non-negotiable) |
| **Decoding** | Beam-4 (MBR decoding implemented but beam used for submission) |
| **Hardware** | Single T4 GPU (Google Colab free tier) |

---

## 📁 Repository Structure

```
├── DSN_AI_Bootcamp_2026,..._ALEXDATA.ipynb   # Main submission notebook
├── submission.csv                             # Final submission (3,486 rows)
├── sample_submission.csv                      # Competition template
├── training_report.md                         # Detailed training analysis
└── README.md                                  # This file
```

---

## 🔬 Approach

### Why AfriTeVa V2

AfriTeVa V2 is pretrained on **Wura**, a curated African-language corpus covering all four target languages. [AfriHG](https://arxiv.org/abs/2404.18434) (Ogunremi et al., 2024) showed Africa-centric seq2seq models outperform mT5-base on African headline generation. The tokenizer advantage alone — measured at 1.4–1.7 F1 points by Ndomba et al. (2025) — makes this a material choice.

### Why LoRA

The competition brief names LoRA as the expected route. With r=16 and alpha=32 targeting q/v attention projections, only ~2M of 429M parameters are trainable. This trains **~3x faster** than a full fine-tune, enabling:
- **6 epochs** (vs ~4 with full fine-tune in the same time)
- **320-token source inputs** (vs 256 with full fine-tune's memory pressure)
- Zero OOM risk on a single T4

### Multitask Training

Both tasks are cast as text-to-text with task + language prefixes:
```
classify hau: <article>   →  politics
headline yor: <article>   →  Kemi Afolabi: Àìsàn Lupus ń bá mi fínra
```

Rare topic labels are **oversampled up to 3x** to address the severe class imbalance (technology: 178 rows vs entertainment: 1,278 rows).

### Label Masking

Only Hausa carries all 7 labels. Technology exists in Hausa alone. At inference, predictions are **masked to language-valid labels** to avoid wasting guesses on categories a language has zero training examples for.

---

## 📈 Training Progression

```
Epoch │ Avg Loss │ Macro-F1 │ GenScore │ Final  │ Best?
──────┼──────────┼──────────┼──────────┼────────┼──────
  1   │  3.9588  │  0.8324  │  0.5353  │ 0.6838 │  ✓
  2   │  2.6752  │  0.8695  │  0.5433  │ 0.7064 │  ✓
  3   │  2.5085  │  0.8829  │  0.5412  │ 0.7121 │  ✓
  4   │  2.4301  │  0.8820  │  0.5484  │ 0.7152 │  ✓
  5   │  2.3547  │  0.8843  │  0.5511  │ 0.7177 │  ✓
  6   │  2.2852  │  0.8948  │  0.5503  │ 0.7226 │  ✓ ← saved
```

**Key observations:**
- Every epoch improved — the model never overfit
- Task A (classification) drove most of the gain: Macro-F1 rose +0.06 over 6 epochs
- Task B (headlines) improved more slowly: GenScore rose +0.015 — expected for low-resource generation
- Loss was still declining at epoch 6, suggesting more epochs would help

---

## 📊 Submission Statistics

### Topic Distribution

| Label | Count | Share |
|-------|-------|-------|
| entertainment | 360 | 20.7% |
| politics | 342 | 19.6% |
| sports | 315 | 18.1% |
| health | 284 | 16.3% |
| religion | 197 | 11.3% |
| business | 173 | 9.9% |
| technology | 72 | 4.1% |

### Languages

| Language | Code | Test Articles |
|----------|------|--------------|
| Hausa | `hau` | 637 |
| Yoruba | `yor` | 411 |
| Igbo | `ibo` | 390 |
| Nigerian Pidgin | `pcm` | 305 |

### Generated Headline Samples

**Yoruba** (BBC Yoruba house style — name: summary):
```
Wizkid: Nkechi Blessing ní òun ti kan ẹsẹ o lẹ́yìn tí òun lọ woran awodamiẹnu
Kemi Afolabi: Àìsàn Lupus ń bá mi fínra
```

**Hausa**:
```
WhatsApp ya sanar da fara amfani da harshen Hausa a wayoyin komai da ruwanka
Yadda injiya ya ƙirƙira injin ban-ruwa a lokacin noman rani
```

---

## 🔧 How to Reproduce

### Requirements
- Google Colab (free tier with T4 GPU) or Kaggle (2x T4)
- Python 3.11+
- `transformers`, `peft`, `bert-score`, `rouge-score`, `sentencepiece`

### Steps
1. Upload the notebook to Google Colab
2. Set runtime to **T4 GPU**
3. Download competition data via Kaggle API or manual upload
4. **Run All** — training takes ~195 min, inference ~30 min
5. Download `submission.csv` and submit to Kaggle

### Seed & Reproducibility
Seed 42, single T4, all training inside the notebook, no external checkpoints. Running end-to-end regenerates the exact submission.

---

## 📚 References

- Adelani et al. (2023). [MasakhaNEWS: News Topic Classification for African Languages](https://arxiv.org/abs/2304.09972).
- Ogunremi et al. (2024). [AfriHG: News Headline Generation for African Languages](https://arxiv.org/abs/2404.18434).
- Oladipo et al. (2023). Better Quality Pretraining Data and T5 Models for African Languages.
- Ndomba et al. (2025). Tokenizers for African Languages. *IEEE Access*.
- Suzgun et al. (2022). Follow the Wisdom of the Crowd: Effective Text Generation via Minimum Bayes Risk Decoding.
- Adjovi et al. (2026). Evaluating LLMs for Hausa and Fongbe Machine Translation.

---

## 👤 Author

**Fakorede Akinwande (AlexData)**

DSN AI Bootcamp 2026 — LLM / Agent Track

---

## 📜 Citation

```bibtex
@misc{dsn2026llm,
  title     = {DSN Bootcamp Hackathon 2026 LLM/Agent Track},
  author    = {Anjuwon Ololade and DSN Community},
  year      = {2026},
  url       = {https://www.kaggle.com/competitions/dsn-bootcamp-hackathon-2026-llm-agent-track}
}
```
