# Author’s note

**Chapter 15:** *Sensitivity Analysis — How Wrong Could We Be?*  
**Context:** Causal Inference with LLMs · INFO 7390 · Hrishi Pal

---

## Design choices

I chose Chapter 15 because sensitivity analysis is the part of causal inference where intellectual honesty lives. Every other chapter in this book teaches you how to estimate something. This one teaches you how to know whether your estimate can be trusted—and what to say when it cannot. The mechanism is precise and the mistake it prevents is consequential: analysts routinely report statistically significant observational results using causal language without ever asking how much confounding their finding could survive. That mistake costs institutions money and patients’ health. I wanted to write the chapter that makes that mistake visible.

The hospital readmission example was chosen deliberately over a business or marketing case. Healthcare decisions have direct human consequences, which raises the ethical stakes of honest reporting in a way that a pricing model does not. It also gave me a domain where the benchmark confounders—medication adherence, health literacy, social support—are well-documented in the literature and have known effect sizes, which is essential for interpreting the E-value and Rosenbaum bound as more than abstract numbers. The clinical judgment calls at the end of the sensitivity analysis—whether a risk ratio of 2.17 is achievable by a real-world confounder—cannot be made by a statistician alone. That non-transferability was the point.

I made two deliberate omissions and one documented design outcome.

1. I did not cover the omitted-variable bias formula or partial *R*² sensitivity approaches (Cinelli and Hazlett, 2020). They are important but would have doubled the chapter’s length and introduced a separate inferential framework.
2. I did not include instrumental-variable or regression-discontinuity designs as alternatives to sensitivity analysis—those are upstream design choices, not diagnostic tools, and belong in a different chapter.
3. The exercise produces an unexpected result rather than an omission: common-support trimming partially protects the Rosenbaum bound in the current DGP, so contamination increases rather than decreases Γ. I documented this as a known limitation rather than hiding it, because reporting fragility honestly—even about your own pedagogical materials—is the chapter’s core argument.

The methods in this chapter are domain-agnostic. The professor’s specified example—social media use and adolescent mental health—would follow exactly the same procedure: estimate the effect, compute the E-value, identify benchmark confounders from the developmental psychology literature, and ask whether any of them could achieve the required risk ratio in both directions. I chose the hospital case instead because the benchmark confounders are quantified in the clinical literature with known effect sizes, and because the $400,000 budget decision makes the stakes of honest reporting concrete rather than abstract. A finding that is fragile in a healthcare context has a face—a patient readmitted, a program scaled prematurely. That stakes structure makes the chapter’s argument harder to dismiss.

---

## Tool usage

### Bookie the Bookmaker (prose)

Bookie drafted all five prose sections. I used it for generation, not for validation.

The most important correction I made was in the Mechanism section, where Bookie silently named the four Tetrahedron elements as “Logic, Implementation, Outcome, and Performance.” The correct fourth element is **Structure**. I caught this by cross-checking against the framework definition and sent Bookie a targeted correction prompt. That correction is documented and is the **Human Decision Node** for the prose section.

Bookie also hallucinated all four code output blocks. The original E-value output showed 2.08 and 1.50; the Rosenbaum bound showed significance lost at Γ = 1.9; the positivity output showed 47 patients (2.4%) outside common support. None of these matched the actual code. I ran every cell locally, obtained the real outputs (E-value 2.17 / 1.56, Γ = 1.6, 4 patients / 0.2%), and corrected the chapter text to match. I corrected Bookie via targeted prompts four times before giving up and updating the chapter artifact directly for the positivity output block, which Bookie repeatedly failed to fix. This is the **Human Decision Node** for the code section.

### Eddy the Editor (critique)

Eddy, a structured critique tool, flagged 18 issues across four categories: four Feynman Standard violations (terms named before intuition was built), four jargon-before-intuition gaps, five fluency issues, and five line-level mechanical errors. I accepted and applied all fixes with one exception: Eddy recommended cutting the third Tetrahedron closing paragraph in the Diagnostics section as repetitive ritual. I kept it because the rubric explicitly rewards Tetrahedron coverage in every section, and removing it would leave the Diagnostics section without an explicit structural close. That disagreement is the **Human Decision Node** for the editing phase.

### Figure Architect (figures)

Figure Architect returned a prioritized suite of six figure prompts with correct priority rankings. I accepted the critical and important tiers and skipped Figure 6 (Disclosure Language Spectrum) as supplementary. For the hero image I used the Figure Architect prompt through Ideogram. For the four analytical figures I built verified Matplotlib code cells rather than using Midjourney or external tools, because runnable code cells in the notebook are more pedagogically valuable than static images.

### AI Scaffold (notebook)

The AI Scaffold, used for notebook structure planning, proposed **los → treatment** as a DAG edge. I rejected this edge with the documented reason that length of stay is determined before discharge and does not cause call assignment—staffing protocol does. `los` is retained as a confounder of outcome, not a cause of treatment. This is the **Human Decision Node** for the notebook.

---

## Self-assessment

### Causal rigor (35 pts) — Self-score: 33/35

The chapter includes a formal do-calculus statement in the Mechanism section: *P*(outcome | do(treatment=1)) − *P*(outcome | do(treatment=0)), with plain-English interpretation. The backdoor criterion is applied through propensity score adjustment but not named as such in the chapter text. The E-value and Rosenbaum bounds are correctly implemented and verified against local execution. I lose points because the DAG was never drawn as a visual figure.

### Technical implementation (25 pts) — Self-score: 21/25

The notebook includes a production AIPW / doubly robust estimator, positivity checks, overlap plots, a falsification test, and both E-value and Rosenbaum bounds sensitivity analyses. The known limitation in Exercise 15.1—where Γ increases rather than decreases with contamination—costs points here. The exercise works mechanically but does not reproduce the directional story the chapter describes. With more time I would redesign the DGP so weekend patients are assigned near-zero treatment probability more aggressively, which would force trimming to fail and produce the expected degradation. The sensitivity analysis for the AIPW estimator itself (as opposed to the chapter’s illustrative OR) is also absent.

### Pedagogical clarity (20 pts) — Self-score: 18/20

The Feynman Standard was enforced by Eddy, and all flagged violations were corrected. The intuition section opens with a concrete scenario, names the outcome, the treatment, and the hidden variable before any formula appears. The three-tier honest reporting formulations in the Diagnostics section are the clearest pedagogical contribution of the chapter—they give students language they can use immediately. I lose minor points because the chapter does not include a worked example of how to benchmark an E-value against the literature from scratch; it names the benchmark (medication adherence) but does not show the student how to find or evaluate such benchmarks in a new domain.
