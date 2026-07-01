# Week 2 — Modeling (Select & Train a Model)

Course: [Machine Learning in Production](https://www.coursera.org/learn/introduction-to-machine-learning-in-production) (Andrew Ng, DeepLearning.AI)

**Through-line:** a high accuracy number is *not* the goal — a model that actually
works on the cases that matter is. And improving the **data** is often how you get
there (data-centric AI).

## 1. Why a great test score isn't enough

Modeling is iterative: **Model + Hyperparameters + Data → train → error analysis → repeat.**
But a low average test error can still be a *bad* model:

- **Disproportionately important examples** — e.g. web search must nail
  navigational queries ("Stanford", "YouTube") even if they're a small fraction.
- **Key slices / fairness** — don't discriminate by ethnicity, gender, etc.
  (loan approval, recommendations).
- **Rare classes / skewed data** — the big one (below).

The "unfortunate conversation": *MLE: "I did great on the test set!" Product owner:
"But it doesn't work for my application."*

### The skewed-data trap
If 99% of patients are healthy, a lazy model that predicts "healthy" for **everyone**
gets **99% accuracy** — a real, verified 99% — yet catches **zero** sick patients.
Useless. (Like a broken smoke detector that never beeps: "correct" 99.9% of days,
but misses the one fire.) **When the important thing is rare, "right most of the
time" ≠ "does the job," and plain accuracy can't tell them apart.**

## 2. Better metrics: precision, recall, F1

Accuracy lumps everything into one number. Split it into two sharper questions.
The **confusion matrix** has 4 outcomes:

|                | Actually sick (y=1) | Actually healthy (y=0) |
|----------------|---------------------|------------------------|
| Predicted sick | True Positive (TP)  | False Positive (FP) — false alarm |
| Predicted healthy | False Negative (FN) — missed! | True Negative (TN) |

- **Recall = TP / (TP + FN)** — *of all the ACTUALLY sick, what fraction did we catch?*
  Catches **MISSES**. The lazy model has 0% recall → exposed.
- **Precision = TP / (TP + FP)** — *of all we FLAGGED as sick, what fraction were real?*
  Catches **FALSE ALARMS**. The "flag everyone" model has ~100% recall but tiny precision.
- **F1** = combines both (harmonic mean) — **punishes imbalance**, so it's only high
  when *both* precision and recall are good. Use it to compare models with one number.

Memory hook: **Recall catches misses; Precision catches false alarms.**

For multiple classes, compute precision/recall/F1 per class (Scratch, Dent, ...).

## 3. Establish a baseline

Before judging a model, set a reference for "what's good":
- **Human Level Performance (HLP)** — how well can a human do it? (Great for
  *unstructured* data: images, audio, text.)
- Literature / state-of-the-art, or an older system.
- The baseline estimates the **irreducible error** (Bayes error) — what's *possible*
  — so you know if there's room to improve. A number without a baseline is meaningless.

Structured (tabular) vs unstructured (image/audio/text) data: HLP is a strong
baseline for unstructured, weaker for structured.

## 4. Error analysis

Don't just look at the overall score — **examine the actual mistakes.**
- Pull misclassified examples and **tag** them (e.g. "car noise", "blurry",
  "reflection", phone model).
- Per tag, ask: what fraction of *errors* have this tag? what fraction of data with
  this tag is misclassified? how much room for improvement?
- Tells you **where** the model fails, so you fix the right thing.

Sanity checks when starting: try to **overfit a tiny dataset first** (proves the
code can learn at all). And: *a reasonable algorithm with good data usually beats a
great algorithm with poor data.*

## 5. Prioritizing what to work on

Rank categories by: **room for improvement** (gap to HLP) × **how frequently it
appears** (% of data) × how **easy** × how **important**. Chase the
**frequent + big-gap** ones for the biggest payoff. Then improve them:
collect more data, improve labels, or use **data augmentation**.

## 6. Skewed datasets (recap)
Examples: manufacturing 99.7% no-defect; medical 98% no-disease; wake-word 96.7% absent.
Use precision/recall/F1, **not accuracy**.

## 7. Performance auditing
Framework: (1) brainstorm how the system might go wrong (subgroups, specific
errors like FP/FN, rare classes), (2) set metrics on the right data slices,
(3) get business/product-owner buy-in. Check accuracy, **fairness, and bias**.

## 8. Data-centric AI

- **Model-centric:** hold data fixed, improve the model/code.
- **Data-centric:** hold the model fixed, improve the **data quality** (Andrew's
  thesis — often beats tweaking the model).
- **Data augmentation:** create realistic hard examples the algorithm does poorly
  on but humans do well on (e.g. add café/car noise to audio). Checklist: sounds
  realistic? X→Y still clear? currently doing poorly on it?
- **Adding features** (structured data): e.g. restaurant rec — add "is the user
  vegetarian?", "does the restaurant have veg options?".
- Can adding data hurt? For unstructured data with a large model + clear X→Y
  mapping, adding data rarely hurts.
- **Experiment tracking:** record algorithm/code version, dataset, hyperparameters,
  results — so you can replicate and compare.
- Slogan: **"from big data to *good* data."** Good data = good coverage of important
  cases, **consistent labels**, timely feedback (covers drift), right-sized.
  **Quality > quantity.**
