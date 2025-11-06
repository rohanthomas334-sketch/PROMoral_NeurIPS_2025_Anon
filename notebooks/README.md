# Notebooks

**Note**: The complete experimental code is available as a Python module in `src/promoral_bench/complete_implementation.py` (1,657 lines).

This file contains all implementations including:
- Model API wrappers (OpenAI, Anthropic, Google, Together)
- All 11 prompting strategies
- Evaluation metrics (Accuracy, F1, ECE, Brier Score, ASR, RTA)
- Data processing and result generation

To run experiments:
1. Install dependencies: `pip install -r requirements.txt`
2. Configure API keys: Copy `.env.example` to `.env` and add your keys
3. See `REPRODUCTION_GUIDE.md` for detailed instructions
