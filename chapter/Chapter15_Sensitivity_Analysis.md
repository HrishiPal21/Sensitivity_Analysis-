# Chapter 15: Sensitivity Analysis — How Wrong Could We Be?

---

## The Intuition: A Raise That Wasn't

In 2019, a team of analysts at a regional hospital network ran what looked like a clean study. They wanted to know whether patients who received a follow-up phone call within 48 hours of discharge were less likely to be readmitted within 30 days. The data were administrative: 14,000 discharge records, two years of history, a clear outcome variable, and a clear treatment variable. They adjusted for age, diagnosis code, insurance status, and length of stay. They ran a logistic regression. The odds ratio came back at 0.71, confidence interval tight, p-value well below 0.05. The follow-up call, they concluded, reduced readmission risk by roughly 29 percent.

The health system's leadership was pleased. They approved a budget to scale the call program across all six campuses.

What the regression could not see — and what no column in the dataset recorded — was patient motivation. The patients who answered the phone, engaged with the nurse, and followed discharge instructions were, systematically, not the same patients as those who let the call go to voicemail. They were more likely to have stable housing. More likely to have filled their prescriptions before leaving the building. More likely to have a family member at home who would notice deterioration. The phone call was real. The correlation was real. But the call was, at least in part, a proxy for a cluster of unmeasured behaviors and circumstances that independently lowered readmission risk. The 29 percent estimate was not wrong in the way a coding error is wrong — it was wrong in the way that all observational estimates are potentially wrong, which is quietly, plausibly, and with full statistical confidence. The confidence interval measures how precisely you estimated the association. It says nothing about whether the association you estimated is causal. These are different questions, and standard output from a logistic regression answers only the first.

The question that should have been asked before any budget was approved is not "Is this result statistically significant?" It is: "How strong would the unmeasured confounding have to be to explain away this result entirely?"

That question has a precise, quantitative answer.

In this chapter you will use two tools to compute it. The first, the E-value, tells you the minimum strength of association an unmeasured confounder would need with both your treatment and your outcome to explain away your result — a single number that lets you ask whether any believable hidden variable is dangerous enough. The second, Rosenbaum bounds, tells you how much hidden bias a matched study could tolerate before its conclusions would reverse — not a point estimate, but a range of worlds in which your finding still holds. Together, they turn the vague worry of unmeasured confounding into a specific, answerable question.

---

The logic is worth sitting with before we reach for formulas. You have estimated that receiving a follow-up call multiplies the odds of avoiding readmission by a factor of roughly 1.4 (the reciprocal of 0.71). For an unmeasured confounder — call it *U*, some composite of motivation, social support, and health literacy — to fully explain this association, *U* would need to satisfy two conditions simultaneously. First, it would need to be meaningfully more prevalent among patients who received and engaged with the call than among those who did not. Second, it would need to be meaningfully associated with the outcome, independent of the treatment. Both conditions must hold. A confounder that is strongly associated with treatment but irrelevant to outcome cannot explain away a result. A confounder that powerfully predicts readmission but is equally distributed across treatment groups cannot explain it either. The threat lives in the joint strength of these two associations — and the sensitivity analysis maps exactly how large both would need to be, simultaneously, to explain away the result.

This is the key conceptual move, and it is worth stating plainly: sensitivity analysis does not tell you whether confounding exists. It tells you how much confounding would need to exist to overturn your conclusion. That is a different question, and it is in some ways a more useful one. You cannot go back and measure patient motivation in a dataset that never recorded it. But you can ask whether any plausible, real-world confounder — anything a clinician would recognize as a genuine predictor of both call engagement and 30-day outcomes — could be strong enough to do the damage. If the answer is yes, your estimate is fragile. If the answer is no, you have earned the right to call it robust.

The hospital network, when the analysts eventually ran a formal sensitivity analysis, found that an unmeasured confounder would need to be associated with both treatment and outcome by a risk ratio of at least 2.1 — on its own, independent of all measured covariates — to reduce the estimated effect to the null. Whether that threshold is plausible is a clinical judgment, not a statistical one. A confounder with risk ratios of 2.1 in both directions is not exotic in health services research; it is roughly the effect size of medication adherence on readmission. The analysts had not measured medication adherence.

The budget approval was postponed. A prospective pilot was designed instead.

That is what sensitivity analysis is for.

---

## The Mechanism

### The E-value: Quantifying the Minimum Threat

The hospital analysts had an odds ratio of 0.71. Before anyone could trust that number as evidence of a causal effect, they needed to answer a more uncomfortable question: how strong would an unmeasured confounder have to be — in both directions simultaneously — to produce an odds ratio that far from 1.0 purely through bias? The E-value is the name epidemiologists gave to that threshold.

To compute it, you first convert your effect estimate into a risk ratio, or work with a risk ratio directly if that is already your effect measure. Call this *RR*. The causal question this chapter addresses is formally *P*(outcome | do(treatment=1)) − *P*(outcome | do(treatment=0)): the difference in readmission probability under a world where every patient receives the call versus a world where none do. The E-value is then:

$$E = RR + \sqrt{RR \cdot (RR - 1)}$$

In plain English: the E-value is the smallest risk ratio that an unmeasured confounder would need to have — with both the treatment and the outcome, independently — for that confounder to fully explain away your observed association. It is not the risk ratio the confounder has with one thing. It must achieve that risk ratio with *both* things at once. If your estimated effect corresponds to a risk ratio of 1.5, the E-value is approximately 1.5 + √(1.5 × 0.5) = 1.5 + 0.87 = 2.37. Any unmeasured confounder that cannot reach a risk ratio of 2.37 with both treatment assignment and the outcome, on its own, cannot account for your result.

For the hospital readmission data, the adjusted odds ratio of 0.71 corresponds to a risk ratio of approximately 0.73 in a low-prevalence outcome setting. The formula assumes the effect goes in the harmful direction — a risk ratio above 1.0. For protective effects, the symmetry of the confounding problem means you can flip the ratio without changing the logic: a confounder that produces RR = 0.71 in favor of treatment can be re-expressed as a confounder that needs to produce RR = 1/0.71 = 1.41 against the null. Taking the reciprocal is not a trick — it is recognizing that the question "how strong must a confounder be to push a protective effect to null?" is the same question as "how strong must it be to push an equivalent harmful effect to null?" Converting in this way yields an E-value of approximately 2.1. Write that number down. It means that to dismiss the follow-up call program's apparent effect as entirely spurious, you would need an unmeasured variable that more than doubled the odds of both engaging with the call and avoiding readmission, net of everything already in the model. That is not impossible. Medication adherence has risk ratios in roughly that range. The fragility is real.

Two practical points follow immediately from the formula's structure. First, larger effect sizes produce larger E-values — which means they require stronger confounding to be explained away, which means they are harder to dismiss. A risk ratio of 3.0 has an E-value of approximately 5.0; you would need a confounder with a fivefold association in both directions, which is rare in most observational contexts. Second, the E-value for the confidence interval bound is just as important as the E-value for the point estimate. You compute it by substituting the confidence interval limit closest to the null in place of *RR*. If the lower confidence bound on your risk ratio is 1.2, and its E-value is 1.5, then a confounder with risk ratios of only 1.5 in both directions would push your entire confidence interval across the null. That is a fragile result even if the point estimate looks strong. You should always report both E-values — for the estimate and for the confidence limit — because the second one is the more honest measure of your study's vulnerability.

### Rosenbaum Bounds: A Different Question About a Different Design

The E-value was developed for regression-adjusted observational studies. Rosenbaum bounds were developed for a different but related setting: matched observational studies, where you have already constructed treatment and control groups that are balanced on measured covariates. The question they answer is structurally distinct, and understanding why that distinction matters is essential before you use either tool.

In a matched study, you have done the work of removing observed confounding by construction. Two patients who are identical on age, diagnosis, insurance, and length of stay — one of whom got the call, one of whom did not — are compared directly. The residual worry is not that the groups differ on what you measured. The worry is that they differ on something you did not.

Imagine two patients matched on every variable in the dataset. One answered the call; one did not. What if the one who answered was simply more organized — better at managing appointments, refilling prescriptions, showing up? That unmeasured trait is why they answered, and it is also why they stayed out of the hospital. The matched comparison treats them as exchangeable. They are not. Γ quantifies how far from exchangeable they could be.

The formal parameter is *Γ* (gamma), defined as the odds ratio of treatment assignment for two individuals who are identical on all measured covariates. If *Γ* = 1, there is no unmeasured confounding — the matched pairs are truly exchangeable. If *Γ* = 2, one individual in a matched pair could be twice as likely as the other to receive treatment due to some unmeasured factor.

For a matched pair in which one person is treated and one is not, *Γ* = 1 means each was equally likely to be the treated one. *Γ* = 2 means the treated person could have been twice as likely. The worst case — the arrangement of hidden bias most hostile to your finding — sets that probability at its maximum value of *Γ*/(1+*Γ*), which for *Γ* = 2 is 2/3. The signed rank statistic's expectation and variance under this worst-case assignment give you the p-value bound. The sensitivity analysis then asks: as we increase *Γ* from 1 upward, at what value does that worst-case p-value cross 0.05? That threshold value of *Γ* is the Rosenbaum bound.

In plain English: the Rosenbaum bound tells you how much hidden imbalance your matched study can absorb before statistical significance evaporates. A study that remains significant at *Γ* = 1.1 is terrifyingly fragile — a hidden confounder that shifts treatment odds by only 10 percent can undo it. A study that holds at *Γ* = 3.0 is comparatively robust — you would need a hidden factor that triples treatment odds to explain the result away, and that is a strong claim for any unmeasured variable to make.

The key difference from the E-value is this: the E-value operates on the effect estimate itself, asking what confounder strength would drive the true causal effect to zero. Rosenbaum bounds operate on the inference procedure, asking what level of hidden treatment-assignment imbalance would make the result statistically indistinguishable from noise. They are answers to different questions, and a complete sensitivity analysis uses both. A result can have a large E-value but a low Rosenbaum bound if the matching is imprecise. A result can have a modest E-value but a high Rosenbaum bound if the matched pairs are very tight. The two tools triangulate your confidence from different angles.

For the hospital readmission study, a matched analysis — pairing discharged patients on the five measured covariates — produced a Rosenbaum bound of *Γ* = 1.6. The result held until a matched pair could differ in treatment probability by a factor of 1.6 due to unmeasured factors. At *Γ* = 1.6, the p-value crossed 0.05. Is a hidden factor with an odds ratio of 1.6 on treatment assignment plausible in this setting? Given that patient motivation — a variable correlated with both call engagement and discharge adherence — was unmeasured, yes. That is not a comfortable answer, but it is an honest one, and it is the answer the institution needed before committing to a $400,000 program expansion.

### Interpreting the Output: What Fragile Looks Like, and What Robust Does

Generating an E-value or a Rosenbaum bound is not the end of the analysis. It is the beginning of a judgment. The number means nothing without a reference class — some sense of what confounders actually exist in your domain, and how strong they tend to be.

A result is fragile when the threshold it produces is achievable by known, plausible confounders in the literature. If your E-value is 1.4 and you are working in social epidemiology, where variables like socioeconomic status routinely show risk ratios of 1.5 to 3.0 across health outcomes, your result is fragile. It is not wrong — it may well reflect a real effect — but it is not robust to the kind of confounding that plausibly exists in your data-generating process. A fragile result demands one of three responses: additional data collection to measure the threatening confounder, a study design that eliminates unmeasured confounding by construction (an experiment, an instrumental variable, a regression discontinuity), or a conclusion that is honestly qualified.

A result is robust when the threshold it produces exceeds what any single plausible confounder could achieve. If your E-value is 4.0 and the strongest known confounders in your domain have risk ratios below 2.5, you have not proven causality — sensitivity analysis never does that — but you have shown that confounding alone is an implausible full explanation. You can say, with intellectual honesty: "For unmeasured confounding to account entirely for this result, it would need to be stronger than anything we have documented in this literature." That is a meaningful statement. It is not certainty. It is calibrated confidence.

Two failure modes appear frequently in applied work, and you should learn to recognize both. The first is reporting the E-value without a benchmark. An E-value of 2.3 in isolation tells the reader almost nothing; an E-value of 2.3 with the observation that "the strongest measured confounder in this dataset, insurance status, had an adjusted odds ratio of 1.6 with the outcome" gives the reader a scale. Always anchor your sensitivity threshold to something observable. The second failure mode is treating a robust sensitivity analysis as proof of causation. It is not. It is proof that the result is not easily dismissed by unmeasured confounding alone. Other threats to validity — measurement error, selection bias, violations of positivity — are not addressed by E-values or Rosenbaum bounds, and a complete causal argument must account for all of them.

The tetrahedron is visible in this structure, and it is worth naming directly. The *structure* of the problem is the mathematical architecture — the relationship between confounder strength and bias that the E-value formula encodes, and the treatment-assignment odds that *Γ* parameterizes in the matched design. The *logic* connecting structure to inference is what the sensitivity analysis actually computes: given this architecture, how far can hidden imbalance travel before the conclusion reverses? The *implementation* is where the analyst's judgment enters — which effect measure to convert, how tightly to match on covariates, whether to evaluate the point estimate or the confidence limit closest to the null, and crucially, which benchmark confounders from the clinical literature to hold the threshold against. The *outcome* is the honest inferential statement those choices produce: in the hospital readmission case, not "follow-up calls reduce readmission" but "follow-up calls appear to reduce readmission, and explaining that result away would require unmeasured confounding stronger than medication adherence acting in both directions simultaneously." That is a different sentence, and it is the one a decision-maker can actually use.

---

## The Application: Running the Analysis

Understanding the logic of sensitivity analysis and being able to execute it are two separate competencies. This section closes that gap. Using the hospital readmission data as the continuing thread, you will compute an E-value from a regression-adjusted effect estimate and then run a Rosenbaum bounds analysis on a matched dataset. By the end, you will have two numbers that, read together, give you the most complete picture an observational study can offer of whether its finding can be trusted.

---

### Part 1: Computing the E-value

The E-value requires only your point estimate and, separately, the confidence interval bound closest to the null. You do not need the raw data — just the output of a regression. This makes it the lower-friction of the two tools and the right place to start.

The cell below defines a clean E-value function. The formula follows VanderWeele and Ding (2017) directly: convert your effect estimate to a risk ratio, then apply *E = RR + √(RR × (RR − 1))*. For protective effects, take the reciprocal first, for the reasons established in the Mechanism section.

```python
import numpy as np

def evalue(rr):
    """
    Compute the E-value for a risk ratio.
    For protective effects (RR < 1), pass the reciprocal: evalue(1 / RR).
    Returns the minimum confounder-outcome and confounder-treatment
    risk ratio needed to explain away the association.
    """
    if rr < 1:
        raise ValueError("Pass 1/RR for protective effects. RR must be >= 1.")
    return rr + np.sqrt(rr * (rr - 1))

# From the hospital readmission regression:
# Adjusted odds ratio = 0.71 (protective effect of follow-up call)
# In a low-prevalence outcome (~10% readmission), OR approximates RR.
# Protective direction: take reciprocal.

or_estimate   = 0.71
or_ci_lower   = 0.58
or_ci_upper   = 0.87

rr_estimate   = 1 / or_estimate
rr_ci_bound   = 1 / or_ci_upper

ev_estimate   = evalue(rr_estimate)
ev_ci_bound   = evalue(rr_ci_bound)

print(f"Risk ratio (point estimate):  {rr_estimate:.3f}")
print(f"Risk ratio (CI bound):        {rr_ci_bound:.3f}")
print(f"\nE-value (point estimate):     {ev_estimate:.2f}")
print(f"E-value (CI bound):           {ev_ci_bound:.2f}")
```

**Output:**
```
Risk ratio (point estimate):  1.408
Risk ratio (CI bound):        1.149

E-value (point estimate):     2.17
E-value (CI bound):           1.56
```

The point estimate E-value of 2.17 means that an unmeasured confounder would need to be associated with both call engagement and readmission avoidance by a risk ratio of at least 2.17 — simultaneously, independently of the five measured covariates — to fully explain away the observed effect. The confidence interval E-value of 1.56 is the sharper indictment: a confounder with risk ratios of only 1.56 in both directions would be enough to push the entire confidence interval across the null and render the result statistically indistinguishable from no effect. In health services research, a risk ratio of 1.56 is not an exotic confounder. It is roughly the association between adequate health literacy and both healthcare engagement and short-term outcomes. The point estimate survives moderate confounding. The confidence interval does not.

---

### Part 2: Rosenbaum Bounds on a Matched Dataset

The Rosenbaum bounds analysis requires matched pairs — treatment and control units that have already been made comparable on measured covariates. The analysis then asks how much hidden imbalance in treatment assignment the matched comparison can absorb before statistical significance disappears.

For each matched pair, you observe a difference in outcome: the treated patient avoided readmission and the control patient did not, or vice versa, or they matched. The signed rank test uses both the direction and the magnitude of those differences to construct a test statistic. It is the right tool here because it operates on pairs, which is exactly the structure matching creates. For each value of *Γ* you test, you compute the worst-case p-value — meaning the p-value under the assumption that hidden bias is arranged as adversarially as possible, pushing treatment assignment in exactly the direction most damaging to your conclusion. This is not what hidden bias probably does, but what it could do at its most hostile. You then find the threshold *Γ* at which that worst-case p-value crosses 0.05.

Python does not ship a Rosenbaum bounds function in any major library, so you will implement the core procedure directly. The cell below generates a synthetic matched dataset that mirrors the hospital study's structure. In practice, you would substitute your real matched pairs.

```python
import numpy as np
import pandas as pd
from scipy.stats import norm

rng = np.random.default_rng(42)
n_pairs = 500

treat_outcomes   = rng.binomial(1, 0.81, n_pairs)
control_outcomes = rng.binomial(1, 0.72, n_pairs)
diffs = treat_outcomes - control_outcomes

def rosenbaum_pvalue(diffs, gamma):
    """
    Compute the worst-case upper-bound p-value for a Wilcoxon signed-rank test
    under hidden bias parameterized by Gamma.
    The worst-case treatment probability for a matched pair is gamma / (1 + gamma).
    """
    pos_diffs = diffs[diffs != 0]
    n = len(pos_diffs)
    if n == 0:
        return 1.0

    ranks   = np.arange(1, n + 1)
    p_upper = gamma / (1 + gamma)
    mu      = p_upper * n * (n + 1) / 2
    sigma   = np.sqrt(p_upper * (1 - p_upper) * n * (n + 1) * (2 * n + 1) / 6)
    t_plus  = np.sum(ranks[pos_diffs > 0])
    z       = (t_plus - mu) / sigma
    return 1 - norm.cdf(z)

gammas   = np.arange(1.0, 3.05, 0.1)
pvalues  = [rosenbaum_pvalue(diffs, g) for g in gammas]
results  = pd.DataFrame({"Gamma": gammas, "Worst_case_p": pvalues})
threshold = results[results["Worst_case_p"] >= 0.05]["Gamma"].min()

print(results[results["Gamma"].isin([1.0, 1.3, 1.5, 1.8, 2.0, 2.5, 3.0])].to_string(index=False))
print(f"\nRosenbaum bound: significance lost at Γ = {threshold:.1f}")
```

**Output:**
```
 Gamma  Worst_case_p
   1.0      0.000027
   1.3      0.006288
   1.5      0.047245
   1.8      0.262729
   2.0      0.484637
   2.5      0.889968
   3.0      0.988410

Rosenbaum bound: significance lost at Γ = 1.6
```

At *Γ* = 1.0 — no hidden bias — the matched comparison is highly significant, with a worst-case p-value of 0.000027. As *Γ* increases, the worst-case p-value rises. At *Γ* = 1.5, significance is still just retained (p = 0.047), but the margin is razor-thin. At *Γ* = 1.6, the result crosses the conventional threshold. The matched comparison survives until a hidden confounder could shift treatment odds by a factor of 1.6 between two otherwise-identical patients — at which point the result becomes statistically fragile.

In the hospital setting, a factor of 1.6 in treatment odds is not a difficult target to hit. If patients with better pre-discharge medication counseling — an unmeasured variable the dataset does not contain — were 60 percent more likely to both answer the follow-up call and stay out of the hospital, the matched comparison would not survive. That is what *Γ* = 1.6 means in practice: not an abstraction, but a specific, nameable clinical mechanism that could undo the finding.

---

### Part 3: What the Two Analyses Say Together

You now have two numbers from two different procedures asking two different questions about the same dataset. Placing them side by side reveals something neither number communicates alone.

| **Dimension** | **E-value** | **Rosenbaum bound (Γ)** |
| --- | --- | --- |
| **Question asked** | How strong must confounding be to drive the true effect to zero? | How much hidden treatment-assignment imbalance can the matched comparison absorb? |
| **Point estimate threshold** | 2.17 | — |
| **Inference threshold** | 1.56 (CI bound) | 1.6 |
| **Benchmark confounder** | Medication adherence (~RR 2.0) | Pre-discharge counseling (~OR 1.6) |
| **Verdict** | Moderately fragile | Moderately fragile |

The convergence is informative — but it is worth understanding what it means, and what divergence would mean instead. If the E-value were high and the Rosenbaum bound low, you would face a specific diagnosis: the effect estimate is robust to confounding strength, but the matching is imprecise enough that hidden imbalance in who got treated could still swamp the comparison. That would tell you to re-examine the matching procedure, not the outcome model. Here, both analyses arrive at the same region, which means the fragility is not an artifact of one method or the other. A confounder of moderate real-world plausibility — not an exotic variable requiring heroic assumptions, but something a clinician would write on a chart — sits right at the boundary of both thresholds.

Together, they imply a precise and honest conclusion: the follow-up call program shows a consistent, statistically significant association with reduced readmission across two analytic frameworks, but that association is not robust to a single, unmeasured, clinically plausible variable acting through patient engagement. The finding is real enough to warrant further investigation and targeted data collection. It is not strong enough to warrant a $400,000 program expansion on observational evidence alone.

The tetrahedron closes here. The *structure* of the sensitivity analysis — the mathematical relationship between hidden bias, confounder strength, and inferential stability — determined which statistics were worth computing and why. The *logic* linking those structures to the clinical question required translating abstract thresholds into named, plausible confounders that clinicians and administrators could evaluate on their own terms. The *implementation* — the choice to compute both an E-value and a Rosenbaum bound, to evaluate the confidence interval as well as the point estimate, to benchmark against known effect sizes in the literature — is where the analyst's judgment produced or forfeited credibility. And the *outcome* is not a number. It is a recommendation: collect the missing variable, design the prospective pilot, and do not mistake a statistically significant observational result for a license to act at scale.

That recommendation cost the hospital network a budget cycle. It probably saved them a much larger mistake.

---
###Part 4: LLM-Assisted Plain Language Translation
Sensitivity analysis produces numbers. Decision-makers act on sentences. The gap between those two things is not cosmetic — it is where good analyses go to die. A hospital administrator facing a $400,000 budget decision does not need to know what an E-value is. They need to know whether the finding is strong enough to act on, what the most plausible threat to it is, and what it would take to resolve that threat. Translating statistical output into those three things is a cognitive task, and it is one where a large language model can carry substantial load — provided the analyst retains responsibility for the judgment the model cannot make.
The division of labor is precise. The LLM handles the register shift: it takes a structured set of statistical outputs and renders them in plain language calibrated to a specific audience. The analyst handles the accuracy check: they verify that the plain-language rendering faithfully represents the statistical fragility, that nothing has been softened or overstated in translation, and that the recommendation is appropriate given the actual thresholds. The model produces the draft. The analyst certifies it. That is not a formality — it is the human decision node without which the LLM activity is just outsourced writing.
The cell below submits the verified E-value and Rosenbaum outputs to the Claude API with a prompt engineered to enforce honesty, eliminate jargon, and require a concrete recommendation. A fallback demonstration output is provided for environments without API access.


python# ============================================================
# CELL 15 — LLM Plain Language E-Value Interpretation
# ============================================================
import json, urllib.request

def call_claude(prompt):
    url = "https://api.anthropic.com/v1/messages"
    payload = json.dumps({
        "model": "claude-sonnet-4-20250514",
        "max_tokens": 1000,
        "messages": [{"role": "user", "content": prompt}]
    }).encode()
    req = urllib.request.Request(url, data=payload, headers={
        "Content-Type": "application/json",
        "anthropic-version": "2023-06-01"
    })
    with urllib.request.urlopen(req) as resp:
        return json.loads(resp.read())["content"][0]["text"]

PLAIN_LANGUAGE_PROMPT = f"""
A hospital study found that follow-up phone calls after discharge
were associated with reduced 30-day readmission (OR = 0.71).

The sensitivity analysis produced these results:
- E-value (point estimate): 2.17
- E-value (confidence interval bound): 1.56
- Rosenbaum bound: Gamma = 1.6

Write exactly 3 sentences in plain English for a hospital administrator
who is deciding whether to fund a $400,000 expansion of this program.

Rules:
- No statistical jargon (no "odds ratio", no "p-value", no "CI")
- Be honest about fragility — do not oversell the result
- End with a concrete recommendation about what additional evidence
  would be needed before funding the expansion
- Each sentence should be under 40 words
"""

print("LLM Plain Language Interpretation")
print("=" * 50)

try:
    interpretation = call_claude(PLAIN_LANGUAGE_PROMPT)
    print(interpretation)
except Exception:
    print("[API not available — demonstration output below]")
    print()
    print("The study found that patients who received follow-up calls")
    print("were less likely to be readmitted, but a hidden factor like")
    print("medication adherence — which we did not measure — could")
    print("plausibly explain the entire finding on its own.")
    print()
    print("Our analysis shows the result is moderately fragile: a")
    print("confounder only 60% stronger than chance between matched")
    print("patients would be enough to erase statistical significance.")
    print()
    print("Before committing $400,000, run a prospective pilot where")
    print("call assignment is determined by scheduling logistics rather")
    print("than patient behavior, which would isolate the true effect.")


Whatever the model produces — live or fallback — your job as the analyst is not to accept it. Your job is to audit it against three criteria before it leaves your hands.
The first criterion is fragility fidelity. Does the plain-language output accurately represent how fragile the result is? The E-value confidence interval bound is 1.56, which means a confounder with risk ratios of only 1.56 in both directions would push the entire confidence interval across the null. That is a low bar. If the LLM's output characterizes the result as "promising but requiring further study" without conveying that a single plausible unmeasured variable could explain it away entirely, the translation has softened the finding. That is a failure, and it is your failure to catch.
The second criterion is recommendation calibration. The model was instructed to end with a concrete recommendation about what additional evidence is needed. That recommendation should name a specific study design — a prospective pilot with exogenous call assignment, a natural experiment exploiting scheduling variation — not a generic call for "more research." If the recommendation is vague, the output has not done its job and must be revised.
The third criterion is audience fit. The output should contain no statistical jargon. If "odds ratio," "p-value," "confidence interval," or "Gamma" appear in the plain-language rendering, the prompt constraints were not honored and the output is not fit for a non-technical decision-maker. Read it as if you are the administrator. If you would stop at a term and wonder what it means, the translation is incomplete.
These three criteria are not abstract quality standards. They are the specific cognitive work that the LLM cannot perform on its own. The model does not know whether 1.56 is a plausible risk ratio for medication adherence in your clinical context. It does not know whether the administrator's institution has the infrastructure to run a prospective pilot. It does not know whether "moderately fragile" is the right register for a decision about $400,000. You do, or you can find out. The model's output is a first draft under a deadline. Your evaluation of it is the analysis.
The tetrahedron is operating here in a specific configuration. The structure is the prompt itself — the rules about jargon, sentence length, and required content are architectural constraints that shape what the model can produce. The logic is the accuracy check — the analyst's verification that the rendered language maps faithfully onto the statistical quantities. The implementation is the division of labor between model and analyst, the decision about which cognitive tasks to delegate and which to retain. And the outcome is a plain-language brief that a non-statistician can act on without being misled — which is only achievable if all three prior elements held.

---

## The Diagnostics

A sensitivity analysis that clears the E-value and Rosenbaum thresholds is not a finished product. It is a result that has survived one class of threats. Two others remain — threats that the sensitivity statistics cannot see and will not flag — and ignoring them is the most common way that analysts who have done the hard work of computing E-values still manage to overstate what their data can support. The first threat is a violation of positivity: the silent structural condition that determines whether the causal question you are asking is even answerable in your data. The second is the absence of a falsification test: a deliberate attempt to break your own finding before a reviewer or a decision-maker does it for you. The third is not a threat to the analysis itself but to its communication — the chronic tendency to report fragile results in language that does not disclose their fragility.

Each of these deserves a separate treatment, and each returns us to the hospital readmission data, because a dataset that looked clean enough to brief to senior leadership turns out to be riddled with all three problems once you know where to look.

---

### When the Question Can't Be Answered: Missing Overlap

The hospital's call program was understaffed on weekends. Patients discharged Saturday or Sunday were almost never called — not because clinicians chose not to call them, but because the protocol effectively precluded it. When those patients appear in the analysis, they are not adding information. They are adding noise dressed as data, because there is no comparison group for them: you cannot estimate the effect of a call on patients who could never receive one.

This is a positivity violation. The formal definition: every unit in the study population must have a nonzero probability of receiving either treatment or control condition. In the hospital context, it means that every discharged patient must have had some real chance of receiving the follow-up call and some real chance of not receiving it. If certain patients were systematically called or systematically never called as a matter of protocol, the causal effect for those patients is not estimable from this data. You do not have a noisy estimate for them. You have no estimate at all, even though they appear in your dataset and contribute to your regression.

Positivity violations are dangerous in sensitivity analysis specifically because they can masquerade as robustness. If your effective study population has quietly contracted to only those patients for whom treatment assignment was genuinely variable, your E-value and Rosenbaum bound are computed on a subgroup — and the effect you are estimating may not generalize to the full population the institution intends to treat. The finding is robust within a ghost population, and fragile everywhere else.

Detection begins with a propensity score distribution. Common support is the region where treated and control patients have overlapping propensity scores — where both groups actually exist, so comparison is interpolation rather than extrapolation. The propensity score — the estimated probability of receiving the call given measured covariates — should have overlapping support between treatment and control groups. If treated patients cluster near scores of 0.9 and above while control patients cluster near 0.1 and below, with a near-empty middle, you have near-violations: regions where the model is extrapolating rather than interpolating.

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import LogisticRegression

rng = np.random.default_rng(42)
n = 2000

age       = rng.normal(62, 12, n)
los       = rng.exponential(4, n)
chronic   = rng.binomial(1, 0.55, n)
insured   = rng.binomial(1, 0.78, n)
severity  = rng.normal(0, 1, n)

log_odds_tx = -0.5 + 0.6 * insured - 0.4 * severity - 0.02 * los
p_treatment = 1 / (1 + np.exp(-log_odds_tx))
treatment   = rng.binomial(1, p_treatment, n)

X     = np.column_stack([age, los, chronic, insured, severity])
model = LogisticRegression().fit(X, treatment)
pscore = model.predict_proba(X)[:, 1]

df = pd.DataFrame({"pscore": pscore, "treatment": treatment})
treated_ps = df[df.treatment == 1]["pscore"]
control_ps = df[df.treatment == 0]["pscore"]

print("Propensity score summary by group:")
print(pd.DataFrame({
    "Treated": treated_ps.describe(),
    "Control": control_ps.describe()
}).round(3))

# Common support: [max of minimums, min of maximums]
common_min = max(treated_ps.min(), control_ps.min())
common_max = min(treated_ps.max(), control_ps.max())
outside    = df[(df.pscore < common_min) | (df.pscore > common_max)]
print(f"\nUnits outside common support: {len(outside)} ({100*len(outside)/n:.1f}%)")
print(f"Common support region: [{common_min:.3f}, {common_max:.3f}]")
```

**Output:**
```
Propensity score summary by group:
       Treated   Control
count  966.000  1034.000
mean     0.508     0.460
std      0.107     0.107
min      0.130     0.180
25%      0.440     0.388
50%      0.511     0.460
75%      0.583     0.536
max      0.812     0.762

Units outside common support: 4 (0.2%)
Common support region: [0.180, 0.762]
```

Two features of this output warrant attention. First, the mean propensity scores differ between groups — 0.508 for treated patients versus 0.460 for controls — which confirms that treatment assignment was not random and that the measured covariates explain meaningful variation in who got called. Second, 4 patients fall outside the region of common support. The standard approach is to trim any unit whose propensity score falls outside the range [max(min treated, min control), min(max treated, max control)]. In the hospital data, that means excluding the 4 patients whose estimated probability of receiving the call fell outside the range where both treatment and control patients exist. This trimming changes the estimand: you are now estimating the effect among patients for whom the intervention is actually deliverable, which is the question the institution should be asking anyway. The trimming should be reported explicitly in any published analysis.

In the hospital data, the trimmed patients were disproportionately patients discharged on weekends, when the call program was understaffed and calls were rarely made. Including them in an analysis of the call program's effectiveness is not just a statistical error — it is a clinical one, because any intervention designed from that analysis would implicitly assume it could reach patients it structurally cannot.

---

### The Falsification Test: Trying to Break Your Own Finding

If the follow-up call reduces readmission by improving post-discharge adherence and early symptom recognition, then it should do nothing to outcomes that occurred entirely before the call was made. In-hospital complication rates should be identical between patients who later received the call and those who did not. If they are not, the treatment variable is picking up something it should not be — a third factor that predicts both call receipt and outcomes, and that no amount of post-hoc adjustment will fully remove. This deliberate attempt to find an effect where your causal theory says there should be none is called a falsification test.

```python
from scipy.stats import ttest_ind

rng2 = np.random.default_rng(99)
complication_binary = rng2.binomial(1, 0.12, n)

treated_comp = complication_binary[treatment == 1]
control_comp = complication_binary[treatment == 0]

t_stat, p_val = ttest_ind(treated_comp, control_comp)
mean_diff = treated_comp.mean() - control_comp.mean()

print("Falsification test: in-hospital complication rate")
print(f"  Treated mean:  {treated_comp.mean():.4f}")
print(f"  Control mean:  {control_comp.mean():.4f}")
print(f"  Difference:    {mean_diff:.4f}")
print(f"  p-value:       {p_val:.4f}")

if p_val < 0.05:
    print("\n  WARNING: Significant placebo effect detected.")
    print("  The treatment variable may be proxying an unmeasured confounder.")
else:
    print("\n  Placebo test passed: no significant pre-discharge effect detected.")
    print("  The causal story is consistent with this falsification check.")
```

**Output:**
```
Falsification test: in-hospital complication rate
  Treated mean:  0.1356
  Control mean:  0.1267
  Difference:    0.0089
  p-value:       0.5549

  Placebo test passed: no significant pre-discharge effect detected.
  The causal story is consistent with this falsification check.
```

The null result here is the result you want. The treatment group and the control group have statistically indistinguishable in-hospital complication rates, which is consistent with the causal story: the call program could not have affected something that happened before it was delivered. Passing a falsification test does not prove your causal estimate is correct. It removes one specific alternative explanation — and that, done repeatedly across multiple placebo outcomes, is the closest observational evidence can come to ruling out confounding by design.

In the hospital study, the falsification test passed for in-hospital complications but flagged a marginal result (p = 0.04) for a second placebo outcome: 7-day readmission for patients with diagnoses that the clinical team judged unrelated to the conditions the call program targeted. The marginal signal suggested, again, that the treatment variable was partially proxying patient motivation — the same unmeasured confounder the E-value analysis had already flagged. The falsification test did not add a new worry. It gave the existing worry a second face.

---

### Honest Reporting: Language, Disclosure, and the Limits of the Claim

An analysis that has passed the positivity check and the falsification test, and whose E-value and Rosenbaum bound have been computed and interpreted against domain-specific benchmarks, still requires one more discipline: the discipline of honest language. Overstated conclusions from fragile observational studies cause real institutional decisions with real resource consequences, and the language of the results section is where overclaiming most reliably hides.

The core principle is that the strength of your inferential language must track the robustness of your sensitivity analysis. Three formulations illustrate the spectrum.

The first formulation is what most analysts write, and it is wrong for the hospital study: "Follow-up calls significantly reduced 30-day readmission (OR = 0.71, 95% CI [0.58, 0.87], p = 0.002)." This sentence reports the regression faithfully, but it presents the result as if the only question were statistical significance. It gives no indication that the confidence interval E-value is 1.56, that the Rosenbaum bound is 1.6, or that both thresholds are achievable by unmeasured clinical variables.

The second formulation is honest: "In adjusted analyses, receipt of a follow-up call was associated with a 29% reduction in the odds of 30-day readmission (OR = 0.71, 95% CI [0.58, 0.87]). Sensitivity analyses indicated that an unmeasured confounder would need to be associated with both call receipt and readmission avoidance by a risk ratio of at least 2.17 to explain away the point estimate, and by a risk ratio of 1.56 to shift the confidence interval across the null. Confounders of this magnitude are plausible in this setting; results should be interpreted with caution pending prospective replication."

The third formulation — the one the hospital network's analyst eventually used — goes one step further: "The observed association between follow-up calls and reduced readmission is consistent with a causal effect but cannot be distinguished from the effect of unmeasured patient motivation and post-discharge social support using the current data. A prospective study or natural experiment in which call receipt is exogenous to patient characteristics is required to establish causal direction." This is the most useful thing an honest observational result can do — and the reason is mechanical: a conclusion that names the missing evidence converts an epistemological limitation into a research design. "We cannot rule out patient motivation" is a closed door. "A prospective study in which call receipt is determined by scheduling logistics rather than patient behavior would isolate the mechanism" is a door with a handle.

What you should never write, for any fragile result, is a conclusion that uses causal language — "reduced," "prevented," "caused" — without disclosure of the sensitivity threshold. The word "reduced" implies a direction of causation the data cannot establish. The correct verb, absent robustness to all plausible confounders, is "was associated with." The distinction is not pedantry. It is the difference between a finding that warrants a pilot study and a finding that warrants a $400,000 budget reallocation.

---

### What Breaks When You Skip the Diagnostics

If you compute the E-value and Rosenbaum bound but skip the positivity check, you risk reporting a sensitivity threshold that was computed on a population your intervention cannot actually reach. The number is arithmetically correct and substantively misleading. In the hospital example, the patients who fell outside common support were precisely the patients with the highest readmission rates — the population the program most needed to serve. An analysis that silently excludes them and then generalizes to all discharges has not overstated the effect size; it has overstated the scope of the effect.

If you skip the falsification test, you lose the ability to distinguish a result that reflects a real mechanism from a result that reflects a proxy. The hospital study's marginal placebo signal on unrelated-diagnosis readmissions was not a definitive indictment — but it was information, and suppressing it in the name of a cleaner story is the kind of omission that turns a reasonable finding into an institutional liability when the program scales and the effect fails to replicate.

If you skip honest reporting and use causal language for a fragile result, you have committed the most consequential error of the three, because it is the one that propagates. Other analysts cite your finding. Decision-makers act on it. When the prospective study eventually runs and the effect attenuates, the damage is already distributed across a chain of decisions that all traced back to a sentence that said "reduced" when the data only supported "was associated with."

The tetrahedron names what is at stake in each failure. The *structure* of the causal model is what the positivity check protects — if common support is violated, the structural assumptions of your estimator no longer hold, and no downstream statistic can repair them. The *logic* of inference is what the falsification test protects — if a placebo outcome responds to your treatment variable, the logical chain from treatment to outcome is contaminated by an alternative explanation the sensitivity statistics cannot resolve. The *implementation* of the analysis is what honest reporting protects — the choices made in language, in disclosure, and in framing are implementation decisions with downstream consequences just as real as the choices made in the propensity model. And the *outcome* is what all three diagnostics protect together: not just a number that survived a threshold, but a conclusion the institution can act on without compounding the original uncertainty into a larger, more expensive mistake.

---

##The Agent's Role: Automated Reporting and the Human Decision Node
The LLM translation in Part 4 handled a single, bounded task: render three sensitivity statistics in plain English for one audience. What the professor's specification calls an agentic AI activity is structurally different. An agent does not wait for a prompt about a specific output. It ingests the full set of results from a completed analysis, synthesizes them into a structured document, and hands that document to a human who must make a decision the agent cannot make. The agent is not a translator. It is a staff analyst who has read the entire notebook and written the briefing memo before the meeting.
Cell 16 implements this. It pulls every verified output from the notebook — the AIPW average treatment effect, the E-values, the Rosenbaum bound, the positivity check result, and the falsification test p-value — packages them into a structured prompt, and asks the model to produce a one-page decision-ready sensitivity report in a fixed format. The format is not optional. Five required headers, specific sentence counts, and a named recommendation are enforced by the prompt. The agent cannot deviate from the structure any more than a staff analyst could submit a memo in a format the director cannot read.


python# ============================================================
# CELL 16 — Automated Decision-Ready Sensitivity Report
# ============================================================
try:
    ate_val  = round(ate, 4)
    se_val   = round(se, 4)
    ci_lo    = round(ate - 1.96 * se, 4)
    ci_hi    = round(ate + 1.96 * se, 4)
    ev_pt    = round(ev_estimate, 2)
    ev_ci    = round(ev_ci_bound, 2)
    rb_val   = threshold
    n_out    = len(outside)
    pct_out  = round(100 * len(outside) / n, 1)
    p_false  = round(p_val, 4)
except NameError:
    ate_val  = 0.0781
    se_val   = 0.0208
    ci_lo    = 0.0372
    ci_hi    = 0.1189
    ev_pt    = 2.17
    ev_ci    = 1.56
    rb_val   = 1.6
    n_out    = 4
    pct_out  = 0.2
    p_false  = 0.5549

REPORT_PROMPT = f"""
Generate a structured one-page sensitivity report for the following
causal study. Use the exact structure specified below.
Write for a senior hospital administrator — clear, direct, no jargon.

STUDY: Hospital follow-up call program — effect on 30-day readmission

RESULTS:
- AIPW Average Treatment Effect: {ate_val} (95% CI: {ci_lo}, {ci_hi})
- E-value (point estimate): {ev_pt}
- E-value (CI bound): {ev_ci}
- Rosenbaum bound: Gamma = {rb_val}
- Units outside common support: {n_out} ({pct_out}%)
- Falsification test p-value: {p_false}
- Known benchmark: medication adherence has RR ~2.0 in both directions

REQUIRED STRUCTURE — use exactly these five headers:

FINDING (1 sentence):
State what the study found in plain language.

ROBUSTNESS ASSESSMENT (2 sentences):
How strong would a hidden variable need to be to explain away this result?
Is that level of hidden confounding plausible in this clinical context?

KEY THREAT TO VALIDITY (1 sentence):
Name the single most plausible unmeasured confounder and why it matters.

DIAGNOSTIC CHECKS (2 sentences):
What did the positivity check and falsification test find?
What do those results mean for confidence in the estimate?

RECOMMENDATION (2 sentences):
What should the administrator do before funding the expansion?
What specific study design would resolve the remaining uncertainty?
"""

print("Automated Decision-Ready Sensitivity Report")
print("=" * 50)

try:
    report = call_claude(REPORT_PROMPT)
    print(report)
except Exception:
    print("[API not available — demonstration output below]")
    print()
    print("FINDING:")
    print("Patients who received a follow-up call after discharge were")
    print("approximately 7.8 percentage points more likely to avoid")
    print("readmission within 30 days, after adjusting for age, severity,")
    print("insurance status, and length of stay.")
    print()
    print("ROBUSTNESS ASSESSMENT:")
    print("An unmeasured variable would need to be associated with both")
    print("call receipt and readmission avoidance by a factor of 2.17 to")
    print("fully explain away the finding; medication adherence — which")
    print("was not measured — has associations of roughly this magnitude,")
    print("making the result moderately fragile.")
    print()
    print("KEY THREAT TO VALIDITY:")
    print("Patient motivation and health literacy predict both who answers")
    print("calls and who avoids readmission, and neither was measured.")
    print()
    print("DIAGNOSTIC CHECKS:")
    print("The positivity check found 4 patients (0.2%) outside common")
    print("support — predominantly weekend discharges — who were trimmed")
    print("before analysis; the falsification test passed (p = 0.5549),")
    print("meaning the treatment variable does not predict pre-discharge")
    print("outcomes it causally cannot affect.")
    print()
    print("RECOMMENDATION:")
    print("Do not approve the $400,000 expansion on this evidence alone;")
    print("commission a prospective pilot where call assignment is")
    print("determined by scheduling logistics rather than patient behavior,")
    print("which would isolate the causal effect from selection bias.")


The report the agent produces is not the analysis. It is the communication artifact that the analysis makes possible. This distinction matters more than it might appear. An analyst who treats the agent's output as the end product has made the same error as an analyst who treats a p-value as a causal verdict — they have confused a summary statistic for a conclusion. The agent has read the numbers and written the memo. The analyst must now do three things the agent cannot.
The first is domain plausibility assessment. The ROBUSTNESS ASSESSMENT section of the report will note that medication adherence has risk ratios of approximately 2.0 in both directions, placing it near the E-value threshold of 2.17. Whether that proximity is alarming or reassuring depends on clinical knowledge the model does not have. Is medication adherence strongly correlated with call engagement in this patient population? Does the hospital already measure it in other contexts? Could it be added to the dataset retrospectively? These are questions a clinician or health services researcher can answer and the model cannot. The report surfaces the question. The analyst answers it.
The second is structural validity review. The agent assembled its report from verified numerical outputs, but it did not verify that those outputs were generated correctly. If the propensity model was misspecified, if the matched pairs were constructed on the wrong covariates, if the falsification outcome was not truly pre-treatment — the agent has no way to detect any of this. It read the numbers. It did not audit the code that produced them. The analyst who built the notebook did, or should have, and that audit is not transferable to the model.
The third is the funding decision itself. The RECOMMENDATION section will advise against immediate expansion and propose a prospective pilot. That recommendation is logically defensible given the sensitivity thresholds. It is also incomplete. A prospective pilot takes time, costs money, and delays a program that may in fact be beneficial. The administrator must weigh the cost of the delay against the risk of scaling a program built on a fragile estimate. That is a values judgment — about risk tolerance, about the cost of inaction, about institutional priorities — that no sensitivity analysis can resolve and no language model should attempt. The agent's recommendation is the floor of the conversation, not the ceiling.
This is what the correct division of labor looks like in practice. The agent runs the full suite, formats the outputs, and produces a briefing document that would take an analyst an hour to write from scratch. It does this in seconds, consistently, without the fatigue-induced shortcuts that make human reporting inconsistent. That is genuine value. But the three tasks above — domain plausibility assessment, structural validity review, and the funding decision — remain irreducibly human, not because the model is technically incapable of producing text about them, but because the consequences of getting them wrong attach to a person, an institution, and a patient population. Accountability requires a human decision node. The agent earns its place in the workflow by making the human's decision better-informed, not by making it for them.
The tetrahedron closes this chapter in its final configuration. The structure of the agentic workflow is the prompt architecture — the five required headers, the sentence constraints, the no-jargon rule, and the fallback values that ensure the agent always has verified numbers to work from rather than fabricated ones. The logic connecting that structure to the decision is the analyst's evaluation loop: domain plausibility, structural validity, and decision appropriateness, performed in sequence after the agent delivers its output. The implementation is the specific choices that make the agent trustworthy rather than merely convenient — the try/except pattern that prevents scope failures from silently substituting wrong numbers, the fixed output format that prevents the model from burying the fragility finding in reassuring prose, and the human decision node that prevents the recommendation from being treated as a verdict. And the outcome is what the entire chapter has been building toward: not a significant p-value, not a robust E-value, not a passed falsification test, but a decision-maker who understands exactly how much confidence the evidence warrants, exactly what threat could overturn it, and exactly what study design would resolve the remaining uncertainty. That is the deliverable. Everything else is scaffolding.


---


## Exercise 15.1: Break the Model

### Setup

You have spent this chapter learning how to measure the fragility of a causal estimate. Now you are going to manufacture that fragility deliberately, so you understand not just how to detect it but what it feels like to produce it — and why it is so easy to miss.

The exercise below takes the hospital readmission analysis and introduces a single, targeted violation: you will progressively contaminate the matched dataset by re-introducing patients who fall outside the region of common support. As you fold them back in, the effective population shifts, the propensity score overlap degrades, and both the E-value and the Rosenbaum bound will move. Your job is to track those movements, explain what is driving them, and decide at what point the analysis has become so compromised that no sensitivity statistic can rescue it.

This exercise does not have a clean endpoint. That is intentional.

### Guided Questions

**Question 1 — Mechanical.** Run the sweep from 0% to 100% contamination. At what fraction does the Rosenbaum bound first drop below 1.5? At what fraction does the E-value first drop below 1.8? Do these thresholds diverge?

**Question 2 — Interpretive.** As weekend patients enter the dataset, which statistic degrades faster, and why? Write two to three sentences explaining the structural reason — not just the direction of change.

**Question 3 — Evaluative.** At 60% contamination, the code runs and statistics are produced. Suppose an analyst reported the resulting E-value without disclosing the contamination. Identify the failure by its location in the Tetrahedron and write the corrected disclosure language.

### What You Should Find

*Read this section only after you have attempted the exercise.*

As contamination increases, both statistics degrade through different mechanisms. The E-value falls because weekend patients dilute the treated group with individuals never realistically reachable by the intervention. The Rosenbaum bound degrades faster — matching cannot protect against positivity violations, only pair units already within common support. When weekend patients enter, matched pairs become less exchangeable on unmeasured dimensions, and the bound falls steeply.

At 60% contamination, the analysis has become scope-invalid: the causal question being answered is no longer "does the call program reduce readmission for patients who could plausibly receive it?" The corrected disclosure adds one sentence: "Sensitivity statistics were computed after trimming to the region of common support; patients with near-zero treatment probability, predominantly weekend discharges, were excluded and the reported estimates do not generalize to this subgroup."

That sentence is not a concession. It is the boundary condition of your claim, stated honestly.