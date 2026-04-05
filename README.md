<div align="center">

# Chapter 15: Sensitivity Analysis  
## How Wrong Could We Be?

**Causal Inference with LLMs** · INFO 7390  
**Hrishi Pal**

[![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)

</div>

---

## Quick links

| Resource | Link |
|:---------|:-----|
| **Published chapter (Substack)** | https://hrishipal.substack.com/p/sensitivity-analysis-how-wrong-could |
| **14-minute video walkthrough** | [Vimeo](https://vimeo.com/1180184288?share=copy&fl=sv&fe=ci) · same URL in [`video_link.txt`](https://vimeo.com/1180184288?share=copy&fl=sv&fe=ci) |
| **Chapter (Markdown)** | [`chapter/Chapter15_Sensitivity_Analysis.md`](chapter/Chapter15_Sensitivity_Analysis.md) |
| **Author’s note** | [`chapter/Authors_note.md`](chapter/Authors_note.md) |

---

> This repository is the full submission for **Chapter 15** of *Causal Inference with LLMs*.  
> It focuses on **sensitivity analysis**: quantifying how much **unmeasured confounding** a causal estimate can survive before the conclusion reverses.

**Running example:** a hospital network’s follow-up call program—14,000 discharge records, an adjusted odds ratio of **0.71**, and an unmeasured confounder (**patient motivation**) that threatens the whole finding.

---

## Repository structure

```text
causal_inference_ch15/
├── Chapter15_Sensitivity_Analysis.ipynb   ← run this (main notebook)
├── video_link.txt                         ← Vimeo URL (plain text)
├── figures/
│   ├── hero_Img.png                       ← chapter hero image
│   ├── fig1_evalue_threshold.png
│   ├── fig2_rosenbaum_curve.png
│   ├── fig3_evalue_benchmarks.png
│   └── fig4_common_support.png
├── chapter/
│   ├── Chapter15_Sensitivity_Analysis.md  ← full chapter prose
│   └── Authors_note.md                    ← tools, decisions, self-assessment
└── README.md
```

---

## Run the notebook

```bash
python3 -m pip install numpy pandas scipy scikit-learn matplotlib
cd causal_inference_ch15
jupyter notebook Chapter15_Sensitivity_Analysis.ipynb
```

Run **all cells top to bottom**. Seeds are fixed so outputs match the **verified values** below.

---

## Notebook map (16 cells)

| # | What it does |
|:--|:---------------|
| **1** | Imports and setup |
| **2** | AI scaffold + **mandatory human decision node** |
| **3** | Synthetic dataset (*n* = 2000, seed 42) |
| **4** | Naive vs adjusted logistic regression |
| **5** | **AIPW / doubly robust** estimator |
| **6** | E-value computation |
| **7** | Rosenbaum bounds |
| **8** | Positivity / overlap diagnostic |
| **9** | Falsification test |
| **10–13** | Four publication-style figures → `figures/` |
| **14** | Exercise 15.1 — *Break the Model* |
| **15** | LLM plain-language E-value interpretation |
| **16** | Automated decision-ready sensitivity report |

---

## Verified outputs

*All values below were run locally and checked before submission.*

| Analysis | Result |
|:---------|:-------|
| AIPW ATE | **0.0781** (95% CI: **0.0372**, **0.1189**) |
| E-value (point estimate) | **2.17** |
| E-value (CI bound) | **1.56** |
| Rosenbaum bound | **Γ = 1.6** |
| Outside common support | **4** units (**0.2%**) |
| Falsification test | *p* = **0.5549** (passed) |

---

## Human decision nodes

Three places the model was **overruled** with documented reasoning:

### 1. DAG edge rejection (Cell 2)

The scaffold proposed **`los → treatment`**. **Rejected:** length of stay is fixed before discharge and does not *cause* call assignment—**staffing protocol** does. **`los`** stays in the model as a confounder of **outcome**, not as a cause of treatment.

### 2. Fabricated code outputs (chapter prose)

Bookie invented numbers that did not match runnable code. Corrections after local runs:

| Claim | Wrong | Right |
|:------|:------|:------|
| E-value (point) | 2.08 | **2.17** |
| E-value (CI bound) | 1.50 | **1.56** |
| Rosenbaum | Γ = 1.9 | **Γ = 1.6** |
| Positivity | 47 pts (2.4%) | **4 pts (0.2%)** |

### 3. Tetrahedron label (chapter prose)

Bookie listed the four elements as Logic, Implementation, Outcome, and **Performance**. The correct fourth pillar is **Structure**—caught and fixed.

---

## Tools used

| Tool | Role | Human correction |
|:-----|:-----|:-----------------|
| **Bookie the Bookmaker** | Chapter prose | Tetrahedron label + four wrong numeric outputs |
| **Eddy the Editor** | Structured critique (Feynman, jargon, fluency) | Kept the third Tetrahedron close in Diagnostics (rubric needs explicit coverage) |
| **Figure Architect** | Figure plan / prompts | Took critical tiers; skipped supplementary Figure 6; hero via Ideogram; analytical figs from **Matplotlib** in the notebook |
| **AI Scaffold** | Notebook structure | Rejected **`los → treatment`**; documented in Cell 2 |

---

## Course use

Submitted for **INFO 7390** — *Causal Inference with LLMs*.  
Reuse ideas with attribution if you adapt this work elsewhere.
