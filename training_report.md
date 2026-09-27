# Training & Evaluation Report

## 🏆 Results Summary

| Metric | Your Score |
|--------|-----------|
| **Dev Final Score** | **0.7226** |
| Macro-F1 (Task A — classification) | 0.8948 |
| GenScore (Task B — headlines) | 0.5503 |
| Model | AfriTeVa V2 base + LoRA |
| Parameters | 429M (well under 1B) |
| Training time | 195 min (~3.25 hrs) |
| Decoder used | Beam-4 |

> [!IMPORTANT]
> Your dev Final Score of **0.7226** significantly beats the reference notebooks' ~0.59 range. That's a **+0.13 improvement** — a massive jump.

---

## Training Progression (6 Epochs)

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

### What this tells us

1. **Every epoch improved** — the model never overfit. The best checkpoint was the last epoch. Given more time, epochs 7-8 might have pushed it higher.

2. **Task A (classification) was the star.** Macro-F1 climbed steadily from 0.83 → 0.89, contributing most of the Final score improvement (+0.06 across 6 epochs).

3. **Task B (headlines) improved slowly.** GenScore went from 0.535 → 0.550 — only +0.015 over 6 epochs. This is expected: headline generation in low-resource languages is fundamentally harder, and BERTScore has known embedding collapse on Hausa/Igbo/Yoruba.

4. **Loss was still declining at epoch 6** (2.29), suggesting the model hadn't fully converged. More epochs could help, especially for headline quality.

---

## Score Breakdown: What Each Component Means

```
Final Score = 0.5 × Macro-F1 + 0.5 × GenScore
            = 0.5 × 0.8948  + 0.5 × 0.5503
            = 0.4474        + 0.2752
            = 0.7226

GenScore    = 0.5 × ROUGE-L  + 0.5 × BERTScore
```

- **Macro-F1 = 0.8948** → The model correctly classifies topics ~90% of the time across all 7 categories, weighted equally. This is excellent.
- **GenScore = 0.5503** → Headlines are reasonable but not perfect. ROUGE-L is likely around 0.20-0.25 (lexical overlap) and BERTScore around 0.85-0.90 (semantic similarity). The blend pulls them together.

---

## Submission Statistics

| Metric | Value |
|--------|-------|
| Total rows | 3,486 (1,743 × 2) ✓ |
| Validation | All checks passed ✓ |
| ID order | Matches sample_submission ✓ |
| Blanks | 0 ✓ |
| Illegal labels | None ✓ |

### Topic Predictions

| Label | Count | Share |
|-------|-------|-------|
| entertainment | 360 | 20.7% |
| politics | 342 | 19.6% |
| sports | 315 | 18.1% |
| health | 284 | 16.3% |
| religion | 197 | 11.3% |
| business | 173 | 9.9% |
| technology | 72 | 4.1% |

This distribution looks realistic — it roughly mirrors the training set proportions, and technology is correctly the rarest (concentrated in Hausa only).

### Headlines by Language

| Language | Test articles | Avg headline words |
|----------|--------------|-------------------|
| Hausa | 637 | — |
| Yoruba | 411 | — |
| Igbo | 390 | — |
| Nigerian Pidgin | 305 | — |
| **Overall** | **1,743** | **11.8 words** |

### Headline Quality Samples (from submission)

**Yoruba** — These look like genuine BBC Yoruba-style headlines:
- `Wizkid: Nkechi Blessing ní òun ti kan ẹsẹ o lẹ́yìn tí òun lọ woran awodamiẹnu`
- `Kemi Afolabi: Àìsàn Lupus ń bá mi fínra`
- `Ebenezer Obey Fabiyi: Ẹ̀kọ́ orin juju ni mo ṣe nígbà tí mo yọjú sí ilé iṣẹ́ rẹkọọdi Decca`

> [!TIP]
> The model is reproducing real BBC Yoruba house style — person names followed by a colon, then the story summary. The diacritics (ẹ, ọ, ń, etc.) are correct, which shows the AfriTeVa tokenizer is working properly.

---

## Why Your Score Beats the References (~0.59)

| Factor | Your notebook | Reference notebooks |
|--------|--------------|-------------------|
| **Backbone** | AfriTeVa V2 (African pretraining) | mT5-base / AfriTeVa V2 |
| **Fine-tuning** | LoRA → 6 epochs in 195 min | Full fine-tune → 4 epochs, slower |
| **Source length** | 320 tokens | 256 tokens |
| **Best checkpoint** | ✅ By dev Final score | Partial (mT5 only) |
| **Label balancing** | ✅ Capped 3x oversampling | Varied |

The **LoRA speed advantage** was the key enabler: training 6 epochs in the same time a full fine-tune takes for 4, and at a longer source length. The best-checkpoint selection also helped — every epoch improved, so the final epoch's weights were kept.

---

## Next Step

Upload [`submission.csv`](file:///c:/Users/USER/OneDrive/Documents/myJOB/task/DSN_LLM_/output/submission.csv) to the Kaggle competition page:

**kaggle.com/competitions/dsn-bootcamp-hackathon-2026-llm-agent-track** → **Submit Predictions** → Upload file

Good luck! 🎯
