# ProMoral-Bench: Evaluating Prompting Strategies for Moral Reasoning and Safety in LLMs

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Official repository for **ProMoral-Bench**, a unified benchmark for evaluating prompting strategies across ethical reasoning and safety tasks in large language models.

## Overview

ProMoral-Bench provides a standardized evaluation protocol for comparing 11 prompting paradigms across 4 model families on moral reasoning and safety tasks. This benchmark enables systematic comparison of prompt engineering approaches for ethical alignment.

### Key Features

- **11 Prompting Strategies**: Zero-Shot, Few-Shot, Chain-of-Thought variants, Role Prompting, Plan-and-Solve, Self-Correct, Thought Experiment, Value-Grounded, and First-Principles reasoning
- **5 Model Families**: GPT-4.1 (OpenAI), Claude Sonnet-4 (Anthropic), Gemini 2.5 Pro (Google), DeepSeek-V3 (Together), llama 3.3 70b instruct (Meta)
- **4 Evaluation Tasks**: ETHICS, ETHICS-Contrast (new), Scruples, WildJailbreak
- **Unified Metrics**: Unified Moral Safety Score (UMSS) combining competence and safety
- **Reproducible Protocol**: Fixed templates, deterministic decoding (temperature=0)

## Installation

```bash
# Clone the repository
git clone https://github.com/[ANONYMIZED]/promoral-bench.git
cd promoral-bench

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Requirements

```
openai>=1.0.0
anthropic>=0.18.0
google-generativeai>=0.3.0
together>=0.2.0
numpy>=1.24.0
pandas>=2.0.0
scikit-learn>=1.3.0
jupyter>=1.0.0
tqdm>=4.65.0
python-dotenv>=1.0.0
```

## Repository Structure

```
promoral-bench/
├── data/
│   └── ethics_contrast/     # ETHICS-Contrast dataset (our contribution)
├── src/
│   └── promoral_bench/      # Source code and model wrappers
├── results/                 # Pre-computed experimental results
│   ├── ethics/              # ETHICS results by model and strategy
│   ├── ethics_contrast/     # ETHICS-Contrast results
│   ├── scruples/            # Scruples results
│   └── wildjailbreak/       # WildJailbreak results
├── Annotations/              
│   └── annotations_eval.ipynb       # The results of the annotation of the ETHICS-Contrast set
│   └── ETHICS_Contrast_annotation_notation.ipynb       # The notation given to all three coauthors who participated in the annotation
├── Significance Tests/              
│   └── Statistical_Significance_and_UMSS_Ablation.ipynb       # The code cells for all significance tests and UMSS ablations
├── requirements.txt
├── .env.example
├── CONTRIBUTING.md
├── LICENSE
└── README.md
```

## Dataset Access

### ETHICS-Contrast Dataset (Our Contribution)

Our newly contributed dataset containing 200 minimal-edit pairs:
- 100 label-flipping edits (should change moral judgment)
- 100 label-preserving edits (should maintain moral judgment)
- Human-audited through two-stage validation process
- Tests robustness to controlled perturbations

Located in `data/ethics_contrast/`.

**License**: CC-BY 4.0

**Limitations**: 
- English-only
- Reflects predominantly Western moral norms
- Perturbations generated with researcher priors
- See paper Section 8 (Limitations) for full discussion

### External Datasets

The following datasets are accessed via HuggingFace during evaluation:
- **ETHICS**: `hendrycks/ethics` - Subset of ETHICS dataset (1,700 instances)
- **Scruples**: `allenai/scruples` - AITA subset (1,466 instances)
- **WildJailbreak**: `allenai/wildjailbreak` - Harmful/benign prompt pairs (430 total)

## API Configuration

Create a `.env` file with your API keys:

```bash
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here
GOOGLE_API_KEY=your_key_here
TOGETHER_API_KEY=your_key_here
```

**Note**: Running all experiments requires API access to all four model providers. See [Compute Resources](#compute-resources) for cost estimates.

## Usage

### Reproducing Results

All experiments can be reproduced using the Jupyter notebook in `notebooks/`:

```bash
# Configure API keys
cp .env.example .env
# Edit .env with your API keys

# Launch Jupyter
jupyter notebook notebooks/analysis.ipynb
```

The notebook contains:
- Complete implementation of all 11 prompting strategies
- Model API wrappers for all 4 providers (OpenAI, Anthropic, Google, Together)
- All evaluation metrics (accuracy, F1, ECE, Brier, ASR, RTA, UMSS)
- Code to reproduce all paper tables and figures

### Pre-computed Results

All experimental results are available in `results/` with per-model, per-dataset CSV files containing detailed metrics. See `results/metadata.json` for experiment configuration details.

## Compute Resources

### Hardware Requirements

- **Compute Type**: Mostly API-based with the addition of open-sourced Llama
- **Memory**: Minimum 8GB RAM recommended for data processing and result aggregation
- **Storage**: ~500MB for complete results and intermediate outputs

### Runtime and Cost Estimates

#### Per Dataset Average Runtime

**ETHICS Dataset** (1,700 examples):
- Efficient strategies (Zero-Shot, Few-Shot): 15-20 minutes per model
- Moderate strategies (Plan-and-Solve, CoT variants): 25-40 minutes per model
- Expensive strategies (Thought Experiment, Self-Correct): 1.5-2 hours per model

**Scruples Dataset** (1,466 examples):
- Efficient strategies: 20-30 minutes per model
- Moderate strategies: 35-60 minutes per model
- Expensive strategies: 1.5-3 hours per model

**ETHICS-Contrast Dataset** (200 examples):
- Efficient strategies: 2-5 minutes per model
- Moderate strategies: 15-25 minutes per model
- Expensive strategies: 40-75 minutes per model

**WildJailbreak Dataset** (230 harmful + 200 benign = 430 total):
- Efficient strategies: 10-25 minutes per model
- Moderate strategies: 30-45 minutes per model
- Expensive strategies: 1-1.5 hours per model

#### Token Usage by Strategy

| Strategy | Avg Tokens/Example | ETHICS Total | Scruples Total | WildJailbreak Avg |
|----------|-------------------|--------------|----------------|-------------------|
| Zero-Shot | 127-630 | 217K-337K | 554K-923K | 507-654 |
| Zero-Shot-CoT | 149-216 | 134K-194K | 515K-601K | 488-659 |
| Few-Shot | 305-955 | 519K-681K | 841K-1.4M | 2,235-2,454 |
| Few-Shot-CoT | 513-1,000 | 462K-522K | 792K-900K | 1,869-2,064 |
| Role Prompting | 207-927 | 266K-505K | 749K-1.1M | 539-710 |
| Thought Experiment | 12,840-18,556 | 642K-1.5M | 798K-1.9M | 6,532-7,498 |
| Plan-and-Solve | 642-1,573 | 183K-405K | 318K-630K | 1,465-1,681 |
| Self-Correct | 2,759-8,726 | 466K-1.2M | 872K-1M | 4,238-8,648 |
| Value-Grounded | 1,148-2,072 | 220K-421K | 310K-605K | 1,148-1,249 |
| First-Principles | 494-1,738 | 97K-246K | 521K-667K | 805-880 |

#### Estimated API Costs (USD)

Based on pricing as of January 2025:
- **OpenAI GPT-4.1**: $5/M input tokens, $15/M output tokens
- **Anthropic Claude Sonnet-4**: $3/M input tokens, $15/M output tokens
- **Google Gemini 2.5 Pro**: $1.25/M input tokens, $5/M output tokens
- **DeepSeek-V3**: $0.27/M input tokens, $1.10/M output tokens

**Per Model-Dataset Configuration:**
- Efficient strategies: $0.10-$2.00
- Moderate strategies: $0.50-$5.00
- Expensive strategies: $5.00-$30.00

**Full Benchmark Reproduction (all 176 configurations):**
- **Estimated Total**: $800-$1,200 USD
  - OpenAI (GPT-4.1): $300-$450
  - Anthropic (Claude Sonnet-4): $250-$400
  - Google (Gemini 2.5 Pro): $100-$150
  - Together (DeepSeek-V3): $30-$50

**Cost Breakdown by Dataset:**
- ETHICS (largest dataset): ~40% of total cost
- Scruples: ~35% of total cost
- WildJailbreak: ~15% of total cost
- ETHICS-Contrast: ~10% of total cost

### Compute Beyond Paper

**Preliminary Experiments Not Reported:**
1. **Max Token Tuning**: Initial runs used default max tokens (512) which caused truncation issues for longer strategies. We reran the following with adjusted limits:
   - Plan-and-Solve: 500 (plan) + 20 (solve) → 600 + 20
   - Thought Experiment: Various stages with 500/700/600/50/300 tokens
   - Self-Correct: 900 tokens (to accommodate multiple revision iterations)
   - Value-Grounded: 800-1,500 tokens depending on model
   - Scruples Role Prompting: 900 tokens (narratives are longer)

2. **Rate Limiting**: Several Gemini runs required retry logic with 0.15s sleep intervals due to rate limits.

3. **Failed Extractions**: Some runs had output parsing failures (noted in "Total Failed Extractions" column), requiring format adjustments to prompts.

4. **Total compute used**: Approximately 1.5× the reported experiments due to the preliminary tuning runs above.

### Computational Efficiency Notes

- All experiments use `temperature=0` for deterministic sampling (single run per configuration)
- No gradient computation or fine-tuning required
- Parallelizable across model-strategy-dataset configurations
- Rate limits encountered primarily with Gemini (implemented retry logic)
- Parsing failures occurred in <2% of runs across all configurations

## Evaluation Metrics

### Classification Tasks (ETHICS, ETHICS-Contrast, Scruples)
- **Accuracy**: Overall classification accuracy
- **Precision, Recall, F1**: For positive class (Unacceptable for ETHICS, fault attribution for Scruples)
- **Expected Calibration Error (ECE)**: 10-bin calibration metric
- **Brier Score**: Probabilistic calibration score
- **Multi-class metrics** (Scruples only): Overall accuracy and weighted F1 across 5 labels

### Safety Task (WildJailbreak)
- **Attack Success Rate (ASR)**: Fraction of harmful prompts that received compliant responses (lower is better)
- **Refusal to Answer (RTA)**: Fraction of benign prompts incorrectly refused (lower is better)
- **Hybrid judgment system**: 
  1. Regex-based heuristic judge
  2. LLM judge (Cohere Command A) for borderline cases
  3. Human audit for disagreements and flagged keywords

### Unified Metric

**UMSS (Unified Moral Safety Score)**: Harmonic mean of two components:

1. **Moral Competence Score (MCS)**: 
   - Min-max normalize all classification metrics across models
   - Average normalized scores across ETHICS, ETHICS-Contrast, and Scruples
   - Captures ethical reasoning accuracy and calibration

2. **Safety Robustness Score (SRS)**:
   - Compute safe rates: (1-ASR) and (1-RTA)
   - Min-max normalize across models
   - Average the two normalized safe rates
   - Captures both attack resistance and benign helpfulness

3. **UMSS Formula**:
   ```
   UMSS = (2 × MCS × SRS) / (MCS + SRS)
   ```
   where β=1 (equal weighting)

The harmonic mean penalizes strategies with imbalanced competence-safety tradeoffs.

See paper Section 3.4 for complete mathematical definitions.

## Prompting Strategies

All strategies enforce standardized output formats for comparability:

1. **Zero-Shot**: Minimal task instruction with output schema
2. **Zero-Shot-CoT**: Adds "think step by step" reasoning cue
3. **Few-Shot**: 5 fixed demonstrations spanning diverse scenarios
4. **Few-Shot-CoT**: 5 demonstrations with 1-3 sentence rationales
5. **Role Prompting**: Expert framing with model-generated confirmation
6. **Thought Experiment**: 5-turn Socratic pipeline (Probe → Answer → Summarize → Select → Decide)
7. **Plan-and-Solve**: 2-stage decomposition (Plan criteria → Solve with constraints)
8. **Self-Correct**: Iterative critique-revision loop (max 3 iterations, early stopping on convergence)
9. **Value-Grounded**: Explicit reasoning through Schwartz values + care ethics frameworks
10. **First-Principles**: 4-step deductive reasoning (Facts → Values → Logic → Conclusion)

See Appendix A of paper for complete prompt specifications.

### Role Confirmation Process

For **Role Prompting**, we use a confirmation selection procedure:
1. Generate 10 acknowledgment candidates from each model
2. Evaluate each on a held-out development set (200 examples)
3. Select the confirmation with highest accuracy + F1
4. Use this fixed confirmation for all test evaluations

**Selected Confirmations by Model:**
- **GPT-4.1**: "Understood! I will use my knowledge of current social norms to assess whether specific behaviors or scenarios are generally considered socially acceptable or unacceptable..."
- **Claude Sonnet-4**: "I understand my role as a reasonable adult evaluating scenarios based on contemporary social norms and standards of acceptable behavior..."
- **Gemini 2.5 Pro**: "Understood. I will act in the role of a typical, reasonable adult with a strong understanding of contemporary social norms..."
- **DeepSeek-V3**: "Understood. I am a typical reasonable adult with a firm grasp of contemporary social norms and will evaluate scenarios as either socially acceptable or unacceptable..."

## Results Summary

### Key Findings

1. **Compact scaffolds dominate**: Few-Shot and Few-Shot-CoT achieve highest UMSS (0.802) at ~1,200 tokens/example, outperforming verbose pipelines by 3-4× on efficiency with superior performance.

2. **Verbose pipelines underperform**: Thought Experiment and Self-Correct consume 10-20× more tokens (12K-18K vs 500-1,500) while achieving UMSS < 0.25, demonstrating that "thinking more" does not improve moral reasoning.

3. **Model-specific patterns**:
   - **GPT-4.1**: Best overall UMSS (0.675), excels with Plan-and-Solve and compact strategies
   - **Claude Sonnet-4**: Top performer on Scruples (0.656 acc) with Few-Shot-CoT, but exhibits higher over-refusal (RTA up to 0.34)
   - **Gemini 2.5 Pro**: Largest gains from exemplars (ΔASR: 0.65 reduction with Few-Shot-CoT)
   - **DeepSeek-V3**: Competitive with First-Principles (0.590 acc on Scruples), strong cost-performance ratio

4. **Safety-competence tradeoff**: Exemplar-based strategies achieve best balance:
   - Few-Shot-CoT: Lowest ASR (0.09) with acceptable RTA (0.19) on Gemini
   - Role Prompting: Efficient safety default (557-710 tokens, ASR 0.21-0.30)

5. **Robustness to perturbations** (ETHICS-Contrast):
   - Compact strategies show smallest degradation (Δ accuracy: -0.015 to -0.045)
   - Verbose strategies exhibit brittleness (Δ accuracy: -0.087 to -0.137)
   - Role Prompting and Zero-Shot-CoT most stable across models

See paper for complete results, statistical analysis, and figures.

## License

This code is released under the MIT License (see [LICENSE](LICENSE)).

### Dataset Licenses

- **ETHICS**: MIT License (original dataset by Hendrycks et al., 2021)
- **ETHICS-Contrast**: CC-BY 4.0 (our contribution, included with this repository)
- **Scruples**: Apache 2.0 (original dataset by Lourie et al., 2020)
- **WildJailbreak**: Apache 2.0 (original dataset by Wei et al., 2023)

All datasets used with permission and proper attribution.

## Contributing

We welcome contributions! Areas where contributions would be particularly valuable:

- Additional prompting strategies
- Support for additional models (e.g., Llama, Mistral)
- Extended evaluation metrics
- Multi-lingual adaptations
- New contrast sets for robustness testing

Please see [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## Limitations and Ethical Considerations

### Statistical Rigor
- **Single-run protocol**: Results are deterministic (temperature=0) but lack variance quantification
- **No significance testing**: Computational constraints prevented multiple runs with error bars
- **Reproducibility**: Results are fully reproducible given fixed random seeds and API versions

### Model and Temporal Specificity
- **API versions**: Results tied to specific model versions accessed January-February 2025
- **Model updates**: Future API updates may affect reproducibility
- **Closed-source models**: Cannot control for training data or alignment procedures
- **Version information**: All model identifiers and API endpoints logged in results metadata

### Linguistic and Cultural Scope
- **English-only**: All prompts and datasets are in English
- **Western moral norms**: Judgments reflect predominantly WEIRD (Western, Educated, Industrialized, Rich, Democratic) moral frameworks
- **Limited generalization**: Findings may not transfer to non-English languages or non-Western cultural contexts
- **Future work**: Multi-lingual and cross-cultural validation needed

### Dataset Biases
- **Scruples (AITA subset)**: 
  - Reddit demographic skew (younger, male-majority, Western users)
  - Self-reported narratives with potential social desirability bias
  - Specific sub-community norms may not generalize
  
- **ETHICS-Contrast**: 
  - Perturbations reflect researcher priors and intuitions
  - Human annotators were US-based researchers
  - May not capture all edge cases or cultural variations

- **WildJailbreak**:
  - Adversarial prompts may not reflect real-world safety scenarios
  - Binary safe/unsafe classification simplifies nuanced safety concerns

### Annotation and Compliance Details

**ETHICS-Contrast Annotation**: Created and validated by three co-authors (computer science researchers based in the United States). No external annotators or crowdworkers were used. No IRB approval was required as this did not constitute human subjects research. Annotation results and notation can be found in the Annotations section of the repository.

**API Terms**: All model outputs obtained in compliance with provider terms of service.

### Prompt-Evaluation Coupling
- **In-distribution demonstrations**: Few-shot examples drawn from same datasets under evaluation
- **Potential inflation**: Within-distribution sampling may inflate few-shot performance
- **Mitigation**: Demonstrations are fixed and non-overlapping with test instances
- **Future work**: Cross-dataset few-shot transfer evaluation needed

### Responsible Use
This benchmark is intended for research purposes to improve LLM alignment and safety. It should not be used to:
- Develop methods to bypass safety guardrails
- Generate harmful content at scale
- Make high-stakes moral decisions without human oversight

The WildJailbreak dataset contains examples of harmful prompts for evaluation purposes only. Access is restricted and should be handled responsibly.

## Contact

For questions, issues, or collaboration inquiries:
- **GitHub Issues**: [Open an issue](https://github.com/[ANONYMIZED]/promoral-bench/issues)
- **Email**: [ANONYMIZED for submission]

## Acknowledgments

We thank:
- The creators of ETHICS (Hendrycks et al.), Scruples (Lourie et al.), and WildJailbreak (Wei et al.) for making their datasets publicly available
- Our human annotators for their careful work on ETHICS-Contrast validation
- [Additional acknowledgments to be added after de-anonymization]

---
