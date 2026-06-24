# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/movies and r/TrueFilm. Given a post or comment, TakeMeter predicts whether it's **Analytical**, **Opinionated**, **Reactive**, or **Social** — and compares that result to a zero-shot LLM baseline.

---

## What This Is

Online film communities produce a wide spectrum of discourse: from one-line reactions ("GOAT tier. Not even debatable.") to careful craft analysis ("Villeneuve's use of silence mirrors how Paul experiences prescience..."). TakeMeter tries to make that spectrum machine-readable by fine-tuning a small transformer model on 214 labeled examples from r/movies and r/TrueFilm.

The honest result: the fine-tuned model struggled. It learned two of four labels and collapsed the rest. The analysis of *why* is more instructive than the performance numbers themselves.

---

## Label Taxonomy

**ANALYTICAL** — Makes a specific, evidence-based argument about craft, narrative, theme, or context. If you strip all opinion language ("I think", "I loved"), a specific checkable claim about how the film works still remains.
> *"Villeneuve's use of silence in Dune isn't just aesthetic — it mirrors how Paul experiences prescience: information arriving before meaning. The quieter scenes are structurally doing the same work as the exposition."*
> *"What made Breaking Bad's finale land is that it rejected catharsis for consequence. Walt doesn't get redemption, he gets closure — and the show is careful not to conflate the two."*

**OPINIONATED** — Takes a clear stance and gives a reason, but reasoning is assertion or feeling rather than craft analysis.
> *"Hereditary scared me more than any horror film in years. The family dynamics felt genuinely realistic which made the supernatural stuff hit harder."*
> *"I think people are way too harsh on the Hobbit trilogy — it's baggy but it's still visually spectacular and the cast is great."*

**REACTIVE** — Pure emotional reaction with no supporting argument — one-liners, hyperbole, ratings without explanation.
> *"GOAT tier. Not even debatable."*
> *"Absolute garbage. Two hours of my life I'll never get back."*

**SOCIAL** — Meta-discussion, questions, recommendations, news, or off-topic content — not evaluating a film.
> *"What's everyone watching this weekend?"*
> *"Can someone explain the ending of Donnie Darko to me?"*

Full edge case decision rules are in [`planning.md`](planning.md).

**Why four labels?** REACTIVE and OPINIONATED are meaningfully different: one has reasoning, the other doesn't. Collapsing them would lose exactly the distinction the community cares about — "it was bad because the pacing was off" vs. "it was bad." SOCIAL is distinct because it's not making a take at all.

---

## Data

**Source:** Pullpush.io public Reddit archive — text posts and top comments from r/movies and r/TrueFilm, sorted by score.

**Why r/TrueFilm in addition to r/movies?** r/movies is primarily a link-sharing subreddit. Text posts and comments with real discourse are a minority. r/TrueFilm requires substantive discussion by default, which produced more ANALYTICAL examples.

**Collection tool:** `scripts/collect_data.py` — no API key required.

**Label distribution (214 examples total):**

| Label | Count | % |
|---|---|---|
| ANALYTICAL | 54 | 25.2% |
| OPINIONATED | 42 | 19.6% |
| REACTIVE | 68 | 31.8% |
| SOCIAL | 50 | 23.4% |

**Annotation method:** Groq `llama-3.3-70b-versatile` pre-labeled all examples using the full label definitions. All 214 suggestions were accepted without human override. The `groq_suggestion` and `groq_confidence` columns in `annotated.csv` record every model suggestion. This is disclosed as a limitation — see Reflection below.

**Hard cases encountered:**

1. *"In every single movie of the series, there is a horrible ratio between action scene volume and conversation volume..."* — Specific observable pattern (ANALYTICAL?) but no craft mechanism explained, stays at frustration level. **Decided: OPINIONATED.**

2. *"Inglourious Basterds was a far better film."* — Comparative claim (OPINIONATED?) but zero criterion for "better." **Decided: REACTIVE** — strip the framing, only preference remains.

3. *"I've heard Vin Diesel and Terry Crews are big nerds who play D&D..."* — Assembles specific biographical facts, never connects them to the films. **Decided: OPINIONATED** — facts without a claim about what they mean.

---

## Training

**Base model:** `distilbert-base-uncased`

**Split:** 70% train (149) / 15% validation (32) / 15% test (33), stratified by label.

**Hyperparameters:** 3 epochs, lr=2e-5, batch size=16 (notebook defaults — no changes made).

**Training behavior:** Loss moved from 1.39 → 1.36 across 3 epochs — a very small decrease indicating the model was not converging. Validation accuracy improved from 21.9% → 43.8%, which sounds significant but on a 32-example validation set represents roughly 7 additional correct predictions.

---

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` (zero-shot — no fine-tuning, no task-specific training)

**How results were collected:** Each of the 33 test examples was sent to the Groq API individually via the Colab notebook's Section 5. The model was given the system prompt below and asked to return only the label name.

**Prompt used:**

```
You are classifying posts and comments from r/movies and r/TrueFilm, Reddit's film discussion communities.

Assign each post to exactly one of the following categories.

ANALYTICAL: Makes a specific, evidence-based argument about craft, narrative, theme, or context.
If you strip out all opinion language ("I think", "I loved"), a specific checkable claim about
how the film works still remains.
Example: "Villeneuve's use of silence in Dune mirrors how Paul experiences prescience — the
quieter scenes are structurally doing the same work as the exposition."

OPINIONATED: Takes a clear stance and gives a reason, but reasoning is assertion or feeling
rather than craft analysis.
Example: "Hereditary scared me more than any horror film in years. The family dynamics felt
genuinely realistic which made the supernatural stuff hit harder."

REACTIVE: Pure emotional reaction with no supporting argument. One-liners, hyperbole, ratings
without explanation.
Example: "GOAT tier. Not even debatable."

SOCIAL: Meta-discussion, questions, recommendations, news, or off-topic. Not evaluating a film.
Example: "What's everyone watching this weekend?"

Respond with ONLY the label name — one of: ANALYTICAL, OPINIONATED, REACTIVE, SOCIAL
Your response must be a single word only. No punctuation, no explanation.
```

---

## Evaluation Report

**Test set:** 33 examples. Random-chance baseline on 4 classes = 25%.

### Overall Accuracy

| Model | Accuracy | F1 (macro) |
|---|---|---|
| Groq `llama-3.3-70b-versatile` (zero-shot) | **0.606** | **0.60** |
| Fine-tuned DistilBERT | 0.333 | 0.25 |

The zero-shot baseline outperforms fine-tuning by 27 percentage points in accuracy and 0.35 in macro F1. Fine-tuning made the model significantly worse.

### Per-class Metrics

| Label | Support | Baseline F1 | Fine-tuned F1 |
|---|---|---|---|
| ANALYTICAL | 8 | **0.93** | 0.52 |
| OPINIONATED | 6 | **0.50** | 0.00 |
| REACTIVE | 11 | 0.40 | **0.48** |
| SOCIAL | 8 | **0.58** | 0.00 |
| **macro avg** | 33 | **0.60** | 0.25 |

The fine-tuned model only beats the baseline on REACTIVE (0.48 vs 0.40) — every other class is worse, and two classes (OPINIONATED, SOCIAL) drop to 0.00.

### Confusion Matrix (Fine-tuned Model)

Rows = true label, columns = predicted label.

|  | Pred: ANALYTICAL | Pred: OPINIONATED | Pred: REACTIVE | Pred: SOCIAL |
|---|---|---|---|---|
| **True: ANALYTICAL** | 6 | 0 | 2 | 0 |
| **True: OPINIONATED** | 0 | 0 | 6 | 0 |
| **True: REACTIVE** | 4 | 0 | 7 | 0 |
| **True: SOCIAL** | 2 | 0 | 6 | 0 |

*See `results/confusion_matrix.png` for the visual version.*

**What this matrix shows:** The model never predicted OPINIONATED or SOCIAL — those columns are entirely zero. It sorted everything into ANALYTICAL or REACTIVE. The primary confusion pairs are OPINIONATED→REACTIVE (6 errors) and SOCIAL→REACTIVE (6 errors).

### Wrong Predictions — Error Analysis

**Error 1 — True: OPINIONATED → Predicted: REACTIVE (confidence: 0.27)**
> *"Inglourious Basterds was a far better film."*

This was already flagged as a hard case during annotation. The label decision was OPINIONATED (comparative claim) but "far better" with no stated criterion is functionally a reaction. The model's prediction of REACTIVE is defensible. This error reveals a genuine boundary problem: short comparative statements without explicit reasoning sit right between OPINIONATED and REACTIVE. The model drew the boundary differently than the annotator — and arguably correctly.

**Error 2 — True: ANALYTICAL → Predicted: REACTIVE (confidence: 0.26)**
> *"Channeling some Spider-Man 2 at the end there with Spidey holding that ship together."*

A short post making a specific intertextual comparison — structurally ANALYTICAL (connects two films via a specific visual callback). The model predicted REACTIVE. This reveals that the model is using post length as a proxy for label: this post is short and informal, which the model associates with REACTIVE regardless of whether a specific claim is present. A tighter training example showing that even short posts can be ANALYTICAL would help.

**Error 3 — True: SOCIAL → Predicted: REACTIVE (confidence: 0.28)**
> *"Thanks everyone for spending some time with me…it was cool to chat with you…have a good day, good days - all my best, warm regards, Keanu."*

Clearly SOCIAL (community engagement, no film evaluation). Predicted REACTIVE because the warm, expressive tone matches the model's REACTIVE pattern. The model cannot distinguish between expressing warmth and expressing an emotional reaction to a film — both use informal, first-person emotional language. This is the most common error type in the test set: 6 of 8 SOCIAL examples were misclassified as REACTIVE.

### AI-Assisted Error Pattern Analysis

Pasting the wrong predictions into Claude to identify patterns surfaced three systematic issues:

1. **OPINIONATED is a learned void.** The model predicts OPINIONATED zero times in the test set. All 6 OPINIONATED examples were misclassified as REACTIVE. This isn't random noise — it's evidence the model failed to internalize OPINIONATED as a distinct category, likely because its training examples looked too similar to REACTIVE examples at the surface text level.

2. **The model learned length as a proxy.** Short, informal posts → REACTIVE. Longer posts using film terminology → ANALYTICAL. This is a reasonable surface heuristic but a wrong one: a one-sentence specific craft observation is ANALYTICAL, not REACTIVE, and a five-paragraph emotional dump is REACTIVE, not ANALYTICAL.

3. **SOCIAL with emotional language collapses into REACTIVE.** Posts that are social in intent but expressive in tone (e.g., Keanu's goodbye, "Did we all see this in high school?") consistently get predicted as REACTIVE. The model has no surface features to separate "expressing warmth in a community" from "expressing an emotional reaction to a film."

What I verified myself: patterns 1 and 3 are definitively confirmed by the confusion matrix (zero OPINIONATED and SOCIAL predictions). Pattern 2 (length proxy) is strongly suggested by the wrong predictions list — the correctly-predicted REACTIVE examples tend to be very short, and the misclassified ANALYTICAL example ("Channeling some Spider-Man 2...") is also short.

### Sample Classifications

Posts run through the fine-tuned model with predicted label and confidence:

| Post (truncated) | True Label | Predicted | Confidence |
|---|---|---|---|
| "Villeneuve's use of silence in Dune isn't just aesthetic — it mirrors how Paul experiences prescience..." | ANALYTICAL | ANALYTICAL | ~0.42 |
| "Holy shit that ending!!! I was not prepared." | REACTIVE | REACTIVE | ~0.45 |
| "Hereditary scared me more than any horror film in years. The family dynamics felt genuinely realistic..." | OPINIONATED | REACTIVE | 0.28 |
| "What's everyone watching this weekend?" | SOCIAL | REACTIVE | 0.28 |
| "Channeling some Spider-Man 2 at the end there with Spidey holding that ship together." | ANALYTICAL | REACTIVE | 0.26 |

*Confidence values for correctly-predicted examples are approximate — exact values available in Colab output.*

**Why the first prediction is reasonable:** The post names a specific technique (use of silence), connects it to a narrative mechanism (how Paul experiences prescience), and draws a structural equivalence (silence = exposition). These are exactly the features that define ANALYTICAL — a specific claim about how a craft choice produces a particular effect. The model correctly identified this pattern.

**Confidence is low across the board (~0.25–0.45).** On a well-trained 4-class model, confident correct predictions should approach 0.80+. These scores indicate the model's softmax output is nearly uniform — it is essentially guessing between ANALYTICAL and REACTIVE on almost every input.

### Reflection: What the Model Learned vs. What Was Intended

The intended distinctions were substantive: ANALYTICAL vs. OPINIONATED captures the difference between *arguing* and *asserting*, which is a real and meaningful difference in discourse quality. SOCIAL vs. REACTIVE captures the difference between *facilitating community* and *reacting to content* — also meaningful.

What the model actually learned was a rougher approximation: **long + film-specific vocabulary = ANALYTICAL, short + emotional = REACTIVE, everything else = one of those two.** OPINIONATED and SOCIAL were never distinguished from their nearest neighbor.

Two things caused this gap. First, the dataset is too small. 37 training examples per class is not enough for DistilBERT to learn subtle distinctions — especially when OPINIONATED and REACTIVE are genuinely similar at the surface level (both are short, both express a position, both use informal language). Second, the annotation has a hidden quality risk: accepting all 214 Groq suggestions without override means the training data reflects Groq's classification heuristics, which may differ from the written definitions in ways that are invisible until you look at the model's failure modes. If Groq labeled some OPINIONATED posts as REACTIVE during annotation (because they're short), the model never had clean OPINIONATED examples to learn from.

**What would fix it:** 300+ examples per class (not total), human review of at least the low-confidence Groq suggestions, and explicit hard-case training examples — posts that are unambiguously OPINIONATED but short, to break the length proxy.

**Why the baseline wins by such a large margin:** The zero-shot baseline has seen vastly more text during pre-training and understands what "analytical" means in the abstract. It achieved F1=0.93 on ANALYTICAL — nearly perfect — without seeing a single labeled example. Fine-tuning on 149 noisy examples didn't add signal; it overwrote the model's existing knowledge with a poor approximation. This is a known failure mode of fine-tuning on very small datasets: the model forgets what it knew and replaces it with dataset-specific shortcuts. The one class where fine-tuning beat the baseline (REACTIVE: 0.48 vs 0.40) is also the majority class — consistent with the fine-tuned model having learned "when unsure, predict REACTIVE" as its dominant strategy.

---

## Spec Reflection

**Where the spec helped:** The requirement to define success criteria before training (Milestone 2) was genuinely useful. Having written "Macro F1 ≥ 0.75 = good enough for deployment" before seeing the results made the evaluation honest — the model clearly did not meet the bar, and that's documented rather than rationalized.

**Where implementation diverged from spec:** The spec assumes the annotator is a human reading and labeling posts. In practice, all 214 annotations were accepted from Groq with zero human override. This diverged from the intended process and is a real limitation — the spec's warning that "skimming without genuine review defeats the purpose" applied here in a more subtle form: accepting every LLM suggestion without pushback is a form of skimming.

---

## AI Usage

**1. Annotation pre-labeling (Groq `llama-3.3-70b-versatile`)**
I directed Groq to classify each post using a structured prompt containing the full label definitions from `planning.md`. Groq produced a label, confidence score, and one-sentence reason for all 214 examples. I accepted all suggestions without override. In retrospect, I should have treated suggestions below 0.85 confidence as requiring human review — 73 examples (34%) fell below that threshold and are the most likely source of label noise.

**2. Label stress-testing (Claude)**
Before annotation, I directed Claude to generate 8–10 posts that would sit at the boundary between ANALYTICAL and OPINIONATED using the label definitions. This surfaced that the definitions needed a concrete decision rule — the "strip the opinion language" test — before they were precise enough to apply consistently. The stress-testing changed the definitions, which is the intended use of this step.

**3. Error pattern analysis (Claude)**
After training, I pasted the 15 wrong predictions into Claude and asked it to identify systematic patterns. Claude identified three patterns: OPINIONATED as a learned void, length as a proxy, and SOCIAL/emotional-language collapse into REACTIVE. I verified all three against the confusion matrix and wrong predictions list before including them in this report. I discarded a fourth suggested pattern ("the model struggles with sarcasm") because I couldn't find evidence of it in the actual wrong predictions.

---

## Running This Project

### Prerequisites

```bash
pip install -r requirements.txt
cp .env.example .env
# Add GROQ_API_KEY for annotation assistance
```

### Collect data

```bash
python scripts/collect_data.py
```

### Annotate

```bash
python scripts/annotate.py --resume
```

### Train and evaluate

Open the Colab starter notebook. Upload `data/annotated.csv`. Set:
```python
LABEL_MAP = {"ANALYTICAL": 0, "OPINIONATED": 1, "REACTIVE": 2, "SOCIAL": 3}
```
Run Sections 1–6 in order. Download `evaluation_results.json` and `confusion_matrix.png` to `results/`.

---

## File Structure

```
.
├── planning.md              # Design decisions, label taxonomy, AI tool plan
├── README.md                # This file — final report
├── requirements.txt
├── .env.example
├── .gitignore
├── data/
│   ├── raw/                 # Collected posts (gitignored)
│   └── annotated.csv        # 214 labeled examples with Groq suggestions
├── scripts/
│   ├── collect_data.py      # Pullpush.io scraper (no auth required)
│   └── annotate.py          # Groq-assisted annotation tool
└── results/
    ├── evaluation_results.json
    └── confusion_matrix.png
```
