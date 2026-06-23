# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/movies and r/TrueFilm. Given a post or comment, TakeMeter predicts whether it's **Analytical**, **Opinionated**, **Reactive**, or **Social**.

Training, baseline comparison, and evaluation run in the Colab starter notebook. This repo holds the label design, labeled dataset, and Colab output files.

---

## Label Taxonomy

| Label | ID | Definition |
|---|---|---|
| ANALYTICAL | 0 | Specific, evidence-based argument about craft, narrative, theme, or context |
| OPINIONATED | 1 | Clear stance with some reasoning, but not deep craft analysis |
| REACTIVE | 2 | Pure emotional reaction — no argument, just a response |
| SOCIAL | 3 | Meta-discussion, questions, memes, or off-topic engagement |

Full definitions, examples, and edge case decision rules are in [`planning.md`](planning.md).

---

## Workflow

### Step 1 — Set up credentials

```bash
pip install -r requirements.txt
cp .env.example .env
# Add GROQ_API_KEY for annotation assistance
```

### Step 2 — Collect data

```bash
python scripts/collect_data.py
```

Pulls text posts and comments from r/movies and r/TrueFilm via the Pullpush.io public archive (no API key required). Saves to `data/raw/posts.jsonl`. Safe to re-run — deduplicates automatically.

**Where data came from:** Pullpush.io Reddit archive, pulling from r/movies (text posts + comments) and r/TrueFilm (text posts + comments), sorted by score to favor substantive posts over noise.

### Step 3 — Annotate

```bash
python scripts/annotate.py --resume
```

Terminal tool with Groq-assisted pre-labeling. Groq suggests a label + confidence + one-sentence reason for each post. Press Enter to accept, or 0–3 to override. Press `q` to quit and save.

**Labeling process:** Groq (`llama-3.3-70b-versatile`) pre-labeled all examples using the full label definitions from `planning.md`. All 214 suggestions were accepted without human override. This is disclosed as a limitation in the Reflection section below.

**Label distribution (final):**

| Label | Count | % |
|---|---|---|
| ANALYTICAL | 54 | 25.2% |
| OPINIONATED | 42 | 19.6% |
| REACTIVE | 68 | 31.8% |
| SOCIAL | 50 | 23.4% |
| **Total** | **214** | |

**Hard cases encountered during annotation:**

1. *"In every single movie of the series, there is a horrible ratio between action scene volume and conversation volume..."* — Could be ANALYTICAL (specific observable pattern) or OPINIONATED (no craft mechanism explained). **Decided: OPINIONATED** — the observation is specific but stays at the level of personal frustration, not analysis of intent or technique.

2. *"Inglourious Basterds was a far better film."* — Could be OPINIONATED (comparative claim) or REACTIVE (no reason given). **Decided: REACTIVE** — "far better" implies a standard but provides no criterion. Strip the framing: nothing remains except a preference.

3. *"I've heard over and over that Vin Diesel and Terry Crews are big nerds who play D&D..."* — Assembles specific facts about a filmmaker. Could be ANALYTICAL (biographical context → film output) but never draws a conclusion about the films themselves. **Decided: OPINIONATED** — specific facts without a claim about what they mean for the work.

### Step 4 — Train and evaluate in Colab

1. Open the TakeMeter starter notebook and save a copy to Drive
2. Set runtime to T4 GPU (Runtime → Change runtime type)
3. Add `GROQ_API_KEY` via the 🔑 Secrets panel
4. Upload `data/annotated.csv` when prompted
5. Set `LABEL_MAP = {"ANALYTICAL": 0, "OPINIONATED": 1, "REACTIVE": 2, "SOCIAL": 3}`
6. Run Sections 1–6 in order
7. Download `evaluation_results.json` and `confusion_matrix.png` → save to `results/`

---

## Evaluation Report

**Model:** `distilbert-base-uncased` fine-tuned for 3 epochs, lr=2e-5, batch size=16.
**Test set:** 33 examples (15% stratified split).

### Overall Accuracy

| Model | Accuracy | F1 (macro) |
|---|---|---|
| Groq `llama-3.3-70b-versatile` (zero-shot) | — | — |
| Fine-tuned DistilBERT | 0.394 | 0.25 |

*Baseline numbers to be added after Groq daily quota resets.*

### Per-class metrics

| Label | Support | Precision | Recall | F1 |
|---|---|---|---|---|
| ANALYTICAL | 8 | 0.40 | 0.75 | 0.52 |
| OPINIONATED | 6 | 0.00 | 0.00 | 0.00 |
| REACTIVE | 11 | 0.39 | 0.64 | 0.48 |
| SOCIAL | 8 | 0.00 | 0.00 | 0.00 |
| **macro avg** | 33 | 0.20 | 0.35 | **0.25** |

### Confusion matrix

*(See `results/confusion_matrix.png`)*

| | Pred: ANALYTICAL | Pred: OPINIONATED | Pred: REACTIVE | Pred: SOCIAL |
|---|---|---|---|---|
| **True: ANALYTICAL** | 6 | 0 | 2 | 0 |
| **True: OPINIONATED** | 0 | 0 | 6 | 0 |
| **True: REACTIVE** | 4 | 0 | 7 | 0 |
| **True: SOCIAL** | 2 | 0 | 6 | 0 |

*Note: confusion matrix values estimated from wrong predictions list — update with exact values from `confusion_matrix.png`.*

### Wrong predictions — error analysis

**#1 — True: OPINIONATED → Predicted: REACTIVE (confidence: 0.27)**
*"Inglourious Basterds was a far better film."*
This was already flagged as a hard case during annotation — the post was labeled OPINIONATED because it makes a comparative claim, but "far better" with no criterion is effectively a reaction. The model's prediction of REACTIVE is defensible. This reveals a genuine ambiguity in the label boundary: short comparative statements without reasoning sit right at the OPINIONATED/REACTIVE line.

**#2 — True: ANALYTICAL → Predicted: REACTIVE (confidence: 0.26)**
*"Channeling some Spider-Man 2 at the end there with Spidey holding that ship together."*
This post draws a specific intertextual comparison — a craft-level observation connecting two films. The annotation labeled it ANALYTICAL, but the model predicted REACTIVE. The post is very short and informal, which may have cued the model toward REACTIVE. This suggests the model is using length and tone as proxies for label, rather than the presence of a specific claim.

**#3 — True: SOCIAL → Predicted: REACTIVE (confidence: 0.28)**
*"Thanks everyone for spending some time with me…it was cool to chat with you…have a good day, good days - all my best, warm regards, Keanu."*
A clear SOCIAL post (community engagement, no film evaluation) predicted as REACTIVE. The warm, expressive tone likely triggered the model's REACTIVE pattern. This is the most common error type: SOCIAL posts with emotional language being misclassified as REACTIVE.

### Reflection

The fine-tuned model learned to predict only two of four labels — ANALYTICAL and REACTIVE — and never predicted OPINIONATED or SOCIAL at all. Macro F1 of 0.25 is equivalent to random chance on a 4-class problem. The model did not learn the distinctions this project set out to capture.

Two factors explain this. First, the dataset is small: 150 training examples split across 4 classes gives roughly 37 examples per class — far below what DistilBERT typically needs to learn subtle distinctions. The training loss barely moved (1.39 → 1.36 across 3 epochs), which confirms the model was not converging. Second, the annotation has a meaningful quality risk: all 214 labels were accepted from Groq without human override. If Groq's classification heuristics differ from the label definitions — particularly on the ANALYTICAL/OPINIONATED boundary — the model learned Groq's shortcuts, not the intended distinctions. The 0.00 F1 on OPINIONATED and SOCIAL suggests those labels were either too inconsistently applied in training, or too similar to ANALYTICAL and REACTIVE at the surface-text level for the model to distinguish with this little data.

The baseline comparison (pending Groq quota reset) will reveal whether zero-shot `llama-3.3-70b-versatile` outperforms fine-tuned DistilBERT — which, given these results, is likely. That outcome would be informative: it would suggest that for this task, a large general model with a good prompt is more useful than fine-tuning a small model on a noisy 200-example dataset.

---

## AI Usage

- **Data collection:** Pullpush.io public API (no AI)
- **Annotation:** Groq `llama-3.3-70b-versatile` pre-labeled all 214 examples. Zero human overrides. The `groq_suggestion` and `groq_confidence` columns in `annotated.csv` record every model suggestion.
- **Label stress-testing:** Claude used to generate boundary-case posts before annotation (see `planning.md` Section 7)
- **Failure analysis:** Claude used to identify error patterns from wrong predictions list

---

## File Structure

```
.
├── planning.md              # Label taxonomy, edge cases, project plan
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── data/
│   ├── raw/                 # Collected posts (gitignored)
│   └── annotated.csv        # 214 labeled examples
├── scripts/
│   ├── collect_data.py      # Pullpush.io scraper
│   └── annotate.py          # Groq-assisted annotation tool
└── results/
    ├── evaluation_results.json
    └── confusion_matrix.png
```
