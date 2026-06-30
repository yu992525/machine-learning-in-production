# Week 1 — ML Project Lifecycle & Deployment

Course: [Machine Learning in Production](https://www.coursera.org/learn/introduction-to-machine-learning-in-production) (Andrew Ng, DeepLearning.AI)

Big theme: training a model that scores well is only a small slice of the job.
Getting it into production — and keeping it working — is the real work.

## The ML Project Lifecycle

Four phases, and it **loops** (it's iterative, not a straight line):

```
Scoping  →  Data  →  Modeling  →  Deployment
(what to    (collect   (train &     (ship, monitor,
 build)      label)     improve)     maintain)
        ↑__________________________________|
```

- **Scoping** — decide what to build; define X→Y; pick metrics (accuracy, latency, throughput) and resources/timeline.
- **Data** — define data, label it *consistently*, establish a baseline; split into train/validation/test.
- **Modeling** — pick & train a model, do **error analysis**; highly iterative.
- **Deployment** — ship to production, then **monitor & maintain**.

## The POC-to-Production Gap

- A model working in a Jupyter notebook (a **Proof of Concept**) is the easy part.
  Making it run reliably 24/7 is a different beast.
- **ML code is only ~5–10% of a real production system.** The other 90–95% is
  infrastructure: data collection, data verification, feature extraction, serving,
  monitoring, configuration, resource management, CI/CD.
  *(From the Google paper "Hidden Technical Debt in ML Systems.")*
- Two quotes that sum it up:
  > "Reaching the milestone of doing well on a holdout test set doesn't mean you're done."
  > "When you deploy a system for the first time, you are maybe about halfway to the finish line."

## Key Challenge: Data Drift vs. Concept Drift

The world changes after you deploy, but your model stays frozen — so accuracy can
silently drop.

| | What changes | Example |
|---|---|---|
| **Data drift** | the **input** distribution (X) | speech model trained on adult voices; teenagers start using it |
| **Concept drift** | the **input→output relationship** (X→Y) | fraud: the *same* transaction pattern that was "safe" is now "fraud" |

Shortcut: **concept drift = the *answer* changes for the same input. Data drift =
the *input itself* changes.** Changes can be **gradual** (slow) or a **sudden
shock** (e.g. COVID-19 changing buying behavior overnight).

## Deployment — software engineering questions to ask

A checklist before shipping a prediction service:
- **Realtime vs. batch?** (instant response vs. processed overnight)
- **Cloud vs. edge vs. browser?** (edge = runs on-device, survives no internet)
- **Compute resources** (CPU / GPU / memory) — and can you afford them in prod?
- **Latency & throughput** (e.g. P95 < 200ms, 1000 QPS)
- **Logging** (for analysis & retraining)
- **Security & privacy** (sensitivity of data, regulations like HIPAA)

## Deployment Patterns

Deployment is **not** all-or-nothing. Roll out **gradually with monitoring**, and
keep a way to **roll back**.

| Pattern | What it's for | Does the model serve real users? |
|---|---|---|
| **Shadow mode** | measure accuracy at **zero risk** | **No** — runs in parallel, output not used; compare to human/truth |
| **Canary** | test on a **small slice** (~5%) of real traffic, then ramp up | **Yes, ~5%** — limited blast radius |
| **Blue-green** | **switch over + instant rollback** | **Yes, everyone** — old (blue) kept warm; flip router back if new (green) misbehaves |

Note: blue-green is a *cutover/rollback* mechanism, not a "more advanced" stage
than canary. The user never *sees* which version they hit — you judge the new
version by **monitoring** its live metrics.

## Degrees of Automation

A spectrum of how much the model is allowed to decide:

| Level | Who decides |
|---|---|
| 1. Human only | human does everything |
| 2. Shadow mode | human decides; ML predicts silently |
| 3. **AI assistance** | human decides; ML *helps* (e.g. highlights a scratch) |
| 4. **Partial automation** | ML decides when *confident*; unsure cases → human |
| 5. Full automation | ML decides everything |

- You **don't have to reach full automation** — stop wherever fits.
- The right level depends on **TWO dials: (1) how good the model is, AND (2) how
  costly a wrong answer is.** High stakes pull the level *down*.
  - Phone-scratch error = cheap → can automate heavily.
  - Tumor-in-X-ray error = a life → stay at **AI assistance**, human on every case.
- Levels 3 & 4 are **"human in the loop"** — humans handle hard cases, and those
  decisions become great training data.

## Monitoring

How you watch a live system and catch drift before it hurts users.

- **Method:** brainstorm everything that could go wrong → pick a **metric** to
  detect each → put them on a **dashboard**. Start with many, prune the useless ones.
- **Three families of metrics:**

| Family | Question | Examples |
|---|---|---|
| **Software** | Is the system healthy? | latency, memory, server load, throughput |
| **Input** | Has the input changed? *(→ data drift)* | avg input length, # missing values, avg image brightness/volume |
| **Output** | Has output / user behavior changed? *(→ concept drift)* | # null outputs, user redoes search, user switches to typing, CTR, **user corrections** |

- Input metrics catch **data drift**; output metrics catch **concept drift** &
  downstream breakage. Monitoring *operationalizes* the drift concept.
- **Iterative:** set thresholds → alarms; adapt metrics & thresholds over weeks.
- When an alarm trips → investigate → maybe **retrain**. *Manual* retraining
  (a human decides) is far more common than automatic.

## Pipeline Monitoring & Speed of Change

- Many ML systems are **pipelines** of multiple steps (e.g. VAD → speech
  recognition; or clickstream → user profile → recommender). A change in one stage
  **cascades** downstream — so monitor individual components too.
  - e.g. if "unknown" labels rise in the user-profile stage, recommendations degrade.
- **How fast does data drift?**
  - **Consumer/user data** usually drifts *slowly* (millions of users rarely all
    change at once).
  - **Enterprise / B2B data** can shift *fast* (one business decision — e.g. a new
    phone coating — changes the whole dataset overnight).
