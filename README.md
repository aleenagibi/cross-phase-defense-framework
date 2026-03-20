# Cross-Phase Defense Framework for Adversarial Attacks on LLMs

This repository contains the prototype implementation for the paper:

> **Cross-Phase Defense Frameworks for Adversarial Attacks on Large Language Models**  
> Aleena Gibi, Indu Verma  
> Department of Science, Christ (Deemed to be University), Delhi NCR Campus

---

## Overview

This project implements a unified three-stage defense pipeline for Large Language Models (LLMs) that operates across multiple phases of the model processing pipeline:

| Phase | Mechanism | Purpose |
|---|---|---|
| Training | Adversarial training (FGSM-based) | Hardens toxicity classifier against embedding-space perturbations |
| Input | Jailbreak classifier | Blocks malicious prompts before they reach the LLM |
| Output | Toxicity filter | Suppresses unsafe model generations |

The key finding is that combining all three phases achieves complete mitigation of jailbreak attempts in our test environment, outperforming any single-phase defense.

---

## Datasets

| Dataset | Purpose | Size Used |
|---|---|---|
| [`mteb/toxic_conversations_50k`](https://huggingface.co/datasets/mteb/toxic_conversations_50k) | Train toxicity classifiers | 10,000 train / 2,000 test |
| [`jackhhao/jailbreak-classification`](https://huggingface.co/datasets/jackhhao/jailbreak-classification) | Train jailbreak input filter | 1,000 train / 300 test |

Both datasets are loaded automatically from Hugging Face during runtime.

---

## Models

- **Toxicity Classifier (baseline + adversarially trained):** `distilbert-base-uncased` fine-tuned for binary toxicity classification
- **Jailbreak Classifier (input filter):** `distilbert-base-uncased` fine-tuned on jailbreak prompt data
- **Surrogate LLM:** `gpt2` used as a lightweight generation model for jailbreak evaluation

---

## Key Results

### Toxicity Classifier Robustness

| Setting | Clean Accuracy | FGSM Robust | PGD Robust | DeepFool-style Robust |
|---|---|---|---|---|
| Baseline | 0.9390 | 0.1915 | 0.0975 | 0.8375 |
| After Adversarial Training | 0.9395 | 0.9990 | 0.1545 | 0.8445 |

### Jailbreak Evaluation (50 prompts)

| Defense Configuration | Success Rate | Blocked (Input) | Blocked (Output) |
|---|---|---|---|
| No defenses | 0.020 | 0 | 0 |
| Input filter only | 0.000 | 46 | 0 |
| Output filter only | 0.020 | 0 | 2 |
| Input + Output (cross-phase) | 0.000 | 46 | 0 |

---

## Repository Structure

```
cross-phase-defense-framework/
│
├── CrossPhaseDefense.ipynb       # Main Colab notebook (recommended entry point)
├── cross_phase_defense.py        # Standalone Python script version
├── requirements.txt              # Python dependencies
└── README.md                     # This file
```

---

## How to Run

### Option A — Google Colab (Recommended)

1. Open [Google Colab](https://colab.research.google.com)
2. Click **File → Upload notebook** and upload `CrossPhaseDefense.ipynb`
3. Set runtime to **GPU**: Runtime → Change runtime type → T4 GPU
4. Run all cells in order (Runtime → Run all)

Total runtime on a Colab T4 GPU is approximately 25–35 minutes.

### Option B — Local Python Environment

```bash
# Clone the repository
git clone https://github.com/aleenagibi/cross-phase-defense-framework.git
cd cross-phase-defense-framework

# Install dependencies
pip install -r requirements.txt

# Run the script
python cross_phase_defense.py
```

> **Note:** A CUDA-capable GPU is strongly recommended. CPU-only runs will be significantly slower, particularly during adversarial training.

---

## Requirements

See `requirements.txt` for the full list. Core dependencies:

- Python 3.9+
- PyTorch 2.0+
- Hugging Face `transformers` 4.35+
- Hugging Face `datasets` 2.14+

---

## Adversarial Attacks Implemented

All three attacks operate in the **continuous embedding space** of the DistilBERT model, since text inputs are discrete and cannot be directly perturbed.

- **FGSM** — Fast Gradient Sign Method; single-step perturbation in the gradient sign direction
- **PGD** — Projected Gradient Descent; iterative multi-step attack with epsilon-ball projection
- **DeepFool-style** — Iterative normalized gradient step approximating minimal decision-boundary perturbation

---

## Reproducibility

All results in the paper are fully reproducible by running `CrossPhaseDefense.ipynb` end-to-end on a Colab GPU runtime with the default configuration. The random seed is fixed at `seed=42` throughout.

---

## Limitations

- GPT-2 is used as a surrogate LLM and does not have safety alignment. Results on instruction-tuned models may differ.
- The DeepFool implementation is an approximation and does not constitute a full DeepFool attack.
- FGSM adversarial training does not generalize to PGD, as expected from prior literature.
- The jailbreak evaluation uses 50 prompts; broader validation on larger sets is needed.

---

## Citation

If you use this code, please cite:

```
@inproceedings{gibi2025crossphase,
  title={Cross-Phase Defense Frameworks for Adversarial Attacks on Large Language Models},
  author={Gibi, Aleena and Verma, Indu},
  booktitle={[Conference Name]},
  year={2025}
}
```

---

## License

This project is released for academic and research use.
