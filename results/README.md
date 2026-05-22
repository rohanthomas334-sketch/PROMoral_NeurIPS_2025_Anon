# Results Directory

This directory contains all experimental results from the ProMoral-Bench study.

## 📁 Directory Structure

```
results/
├── ETHICS/                     All ETHICS dataset results
│   ├── GPT-4.1/               Results by strategy
│   ├── Sonnet-4/              Results by strategy  
│   ├── Gemini 2.5/            Results by strategy
│   └── Deepseek/              Results by strategy
│   └── Llama/                 Results by strategy
├── SCRUPLES/                   All Scruples dataset results
│   ├── GPT-4.1/               Results by strategy
│   ├── Sonnet-4/              Results by strategy  
│   ├── Gemini 2.5/            Results by strategy
│   └── Deepseek/              Results by strategy
│   └── Llama/                 Results by strategy
├── ETHICS CONTRAST/            All ETHICS-Contrast results
│   ├── GPT-4.1/               Results by strategy
│   ├── Sonnet-4/              Results by strategy  
│   ├── Gemini 2.5/            Results by strategy
│   └── Deepseek/              Results by strategy
│   └── Llama/                 Results by strategy
└── WildJailbreak/              All WildJailbreak results
│   ├── GPT-4.1/               Results by strategy
│   ├── Sonnet-4/              Results by strategy  
│   ├── Gemini 2.5/            Results by strategy
│   └── Deepseek/              Results by strategy
│   └── Llama/                 Results by strategy
└── Ablations/                  All WildJailbreak results
    ├── GPT-4.1/               Results by strategy
    ├── Sonnet-4/              Results by strategy
    ├── Gemini 2.5/            Results by strategy
    └── Deepseek/              Results by strategy
```

## 📊 What's Inside

Each model/strategy folder contains CSV files with:
- `*_results.csv` - Main results with predictions and metrics
- `*_metrics_summary.txt` - Summary statistics
- `*_manual_review.csv` - (Some strategies) Manual review annotations

## 🔍 File Format

### Main Results CSV Columns:
- `id` - Example identifier
- `prompt` / `scenario` - Input text
- `prediction` - Model's prediction
- `label` - Ground truth label
- `confidence` - Model's confidence score (0-1)
- Additional metrics vary by dataset

### Metrics Summary Files:
- Accuracy, Precision, Recall, F1
- ECE (Expected Calibration Error)
- Brier Score
- Token statistics

## 📈 Strategies Evaluated

1. Zero-Shot
2. Zero-Shot CoT
3. Few-Shot
4. Few-Shot CoT
5. Role Prompting (with confirmation)
6. Thought Experiment
7. Plan-and-Solve
8. Self-Correct
9. Value-Grounded
10. First Principles

## 🔢 Dataset Sizes

- **ETHICS**: 1,700 examples
- **ETHICS-Contrast**: 200 example pairs (400 total)
- **Scruples**: 1,466 examples
- **WildJailbreak**: 430 examples (230 harmful + 200 benign)

## 📝 Usage

To analyze results:
1. Load CSV files with pandas: `pd.read_csv('path/to/results.csv')`
2. See RESULTS_FORMAT.md for detailed column specifications
3. See REPRODUCTION_GUIDE.md for analysis scripts

## ⚙️ Processing Results

If you want to create cleaned/aggregated CSVs:
1. See REPOSITORY_CHECKLIST.md for instructions
2. Or use the data as-is (completely acceptable!)

## ❓ Questions?

- Format questions → See RESULTS_FORMAT.md
- Reproduction questions → See REPRODUCTION_GUIDE.md
- General questions → See README.md

---

**Total Results**: 176 configurations (11 strategies × 4 models × 4 datasets)
**Total Compute**: ~$800-1,200 USD, 50-80 hours runtime
**All experiments**: Deterministic (temperature=0), single run per configuration
