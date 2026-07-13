# Week 3 — Data: Define Data and Establish a Baseline

Course: [Machine Learning in Production](https://www.coursera.org/learn/introduction-to-machine-learning-in-production) (Andrew Ng, DeepLearning.AI)

**Where we are:** the course arc is **Scoping → Data → Modeling → Deployment.**
W2 was Modeling; this week is the **Data** stage. Through-line: *deciding what the
data even is* is a hard engineering problem, and cleaning it up is often where the
real accuracy gains hide (data-centric AI).

## 1. Why data definition is hard: label ambiguity

Reasonable, honest labelers label the same example **differently** — that's noise,
and a model trained on self-contradicting labels is fitting a moving target.

- **Iguana detection:** "draw a bounding box" — one box vs two, tight vs loose.
- **Speech:** "Um, nearest gas station" vs "Umm..." vs "[unintelligible]".
- **Phone defect:** "label 1 if defective" — but *how long a scratch counts?*
  0.2mm ok, 0.3mm defective? Everyone draws the line differently.

Every data problem forces two questions:
- **What is the input `x`?** (lighting, resolution, are the right features present?)
- **What is the target `y`?** — and *how do we get consistent labels for it?*

> Noisy labels are often an **ambiguous-instructions** problem, not a sloppy-labeler
> problem. Fix: make `y` an operational rule (">0.3mm = defective"), not a vibe.
> (Same idea as a good rules file: specific, executable, checkable.)

## 2. The 2×2: what kind of data problem are you in?

| | **Unstructured** (images/audio/text) | **Structured** (rows/columns) |
|---|---|---|
| **Small data** | manufacturing visual inspection | housing price from 50 rows |
| **Big data** | speech recognition (50M) | shopping recommendations |

- **Unstructured** → humans *can* label it; **data augmentation** helps (flip/rotate/brighten).
- **Structured** → humans often *can't* label it; harder to get more data.
- **Small data** → **clean/consistent labels are critical**; you *can* hand-fix them
  and get all labelers to agree.
- **Big data** → emphasis on the **data process** (can't inspect millions by hand).
- **Big data can hide small-data challenges:** a **long tail of rare events**
  (web search, self-driving, recommendations) has few examples → consistency matters
  there too.

**Why small data is more dangerous:** with millions of examples, noisy labels
average out (the majority dominates). With 100 examples, 10 bad labels = 10% of your
whole signal contradicting itself, and there's nothing to average against.

## 3. Improving label consistency

1. Have **multiple labelers** label the same example → detects disagreement.
2. On disagreement, have **MLE + subject-matter expert (SME) + labelers discuss `y`**
   until they agree on a rule. (Fix the instruction, not just the label.)
3. If labelers say **`x` lacks the info** to decide → **change `x`** (better photo,
   more features).
4. **Iterate** until agreement is hard to improve.

Tactics: **standardize** labels ("Umm" → "Um"); **merge classes** you can't reliably
distinguish (deep vs shallow scratch → "scratch"); **add an uncertainty class**
(`0 / Borderline / 1`).

> A borderline class **turns disagreement into agreement** — labelers can consistently
> agree something is uncertain, but can't consistently force it to 0 or 1. Bonus: it
> keeps the clean 0s and 1s uncontaminated by the messy middle.

## 4. Human-Level Performance (HLP)

**HLP = how well a human does the task.** Main use: **estimate irreducible / Bayes
error** — the error floor no model can beat because the labels themselves are fuzzy.
If two humans agree only 88%, don't expect a model at 99%. Helps set realistic
targets and prioritize.

**The trap:** when ground truth `y` is *itself just another human's label*, "beating
HLP" can be meaningless. On "Um/Umm" clips, humans "disagree" 12% only on
**punctuation habits** — a model that beats that is learning one labeler's formatting,
**not** getting better at speech. Worse, the trophy number can **mask real errors**.

- Low HLP often = **ambiguous labeling instructions**, not bad humans.
- Raising HLP (via consistency) makes it *harder* for ML to "beat" humans, but the
  cleaner labels raise the model's **real** performance — which is the actual win.
- **HLP on structured data is rare** (no human labelers), except: user-ID merge,
  "is this computer hacked?", fraud, spam/bot, transport mode from GPS.

## 5. Obtaining & labeling data

- **Get into the iteration loop fast.** Don't ask "how long to get `m` examples?"
  Ask **"how much data can we get in `k` days?"** You learn what data you need by
  training + error analysis, not by planning. (Exception: you've done this exact
  problem before and *know* you need `m`.)
- **Inventory sources** with real numbers — source × **amount × cost × time** —
  plus quality, privacy, regulatory constraints.
- **Labeling:** in-house / outsourced / crowdsourced. Who's qualified varies:
  speech = any fluent speaker; factory/medical = **SME**; recommenders = maybe
  impossible to label.
- **Hard rule: don't grow the training set by more than ~10× at a time.** Add in
  steps so a break or distribution shift is **diagnosable and cheap to unwind** — a
  50× jump is a black box you can't debug. (Same "small verifiable steps" principle
  as debugging and small PRs.)

## 6. Pipelines, provenance, metadata, splits

- **Provenance** = *where data came from*; **lineage** = *the sequence of processing
  steps*. Track both, or a broken pipeline is un-debuggable months later.
- **Metadata** = data about data (timestamp, camera settings, phone model, labeler
  ID, device type). Gold for **error analysis** ("all errors come from camera #3").
- **POC vs Production:** in a proof-of-concept, manual/messy data prep is fine — just
  take good notes. Once value is proven, invest in **replicable** pipeline tooling
  (TensorFlow Transform, Apache Beam, Airflow).
- **Balanced train/dev/test splits (small data only):** 100 examples, 30 defective —
  a *random* split can land a test set with only 2 defective, so (a) the metric is
  wildly noisy on 2 points, and (b) a lazy "always say OK" model scores 90% accuracy
  while catching **zero** defects. Deliberately keep ~30% defective in each split.
  **Big data:** random split is representative automatically.

## 7. Scoping (optional module)

- Start from a **business problem, not an AI problem.** Separate *what to achieve*
  (increase conversion, reduce inventory) from *how* (search, recommendations,
  demand prediction).
- **Feasibility:** use external benchmarks; **HLP** — can a human, given the same
  data, do the task? (People are great on unstructured data.) Do we have
  **predictive features**?
- **Value:** get technical (MLE metrics) and business teams to agree on shared
  metrics. Consider **ethics** (net positive? fair/unbiased? concerns aired?).
- **Milestones:** specify ML metrics, software metrics (latency/throughput),
  business metrics, and resources. Unsure? Benchmark or build a POC first.

## Self-test

- **Why are inconsistent labels worse in small data?** No volume to average the noise
  away — each bad label is a big fraction of the signal.
- **What does a "borderline" class buy you?** Labelers can consistently agree on
  uncertainty instead of coin-flipping ambiguous cases into contradictory 0/1s.
- **Why be skeptical of "we beat HLP"?** If ground truth is another human's noisy
  label, beating it may measure conformity to formatting habits, not real capability.
- **Why not 50× the data at once?** A break/shift becomes un-diagnosable; grow ≤10×
  so problems stay isolatable.
- **Why balance splits in small data?** A random split can be unrepresentative by
  luck, making every test metric a lie.
