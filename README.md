# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/movies (and r/television). Given a post or comment, TakeMeter predicts whether it's **Analytical**, **Opinionated**, **Reactive**, or **Social**.

Training, baseline comparison, and evaluation all run in the [Colab starter notebook](). This repo holds the label design, labeled dataset, and Colab output files.

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
# Fill in REDDIT_CLIENT_ID and REDDIT_CLIENT_SECRET
# (create a "script" app at https://www.reddit.com/prefs/apps)
```

### Step 2 — Collect data

```bash
python scripts/collect_data.py --subreddit movies --limit 500
```

Saves posts and comments to `data/raw/posts.jsonl`. Deduplicates automatically — run multiple times if needed.

**Where data comes from:** r/movies subreddit via the Reddit API (PRAW), scraped from `hot`, `top (week)`, and `top (month)` to get a mix of visibility levels.

### Step 3 — Annotate (~2–3 hours)

```bash
python scripts/annotate.py --resume
```

Terminal tool: press `0–3` to label, `s` to skip, `q` to save and quit. Live label distribution is shown after each batch. Save the output as `data/annotated.csv`.

**Labeling process:** Apply definitions from `planning.md`. Classify by the *dominant intent* of the post. Use the documented edge case rules for hard cases.

**Target:** At least 200 labeled examples, ≥20% per label (≥40 per class).

**Label distribution:** *(fill in after annotation)*

| Label | Count | % |
|---|---|---|
| ANALYTICAL | — | — |
| OPINIONATED | — | — |
| REACTIVE | — | — |
| SOCIAL | — | — |
| **Total** | — | — |

**Hard cases:** *(document at least 3 after annotation)*

1. **Example 1:** [text snippet] — Could be X or Y. Decision: [what you chose and why].
2. **Example 2:** ...
3. **Example 3:** ...

### Step 4 — Train and evaluate in Colab

1. Open the [TakeMeter starter notebook]() and save a copy to your Drive
2. Set runtime to T4 GPU (Runtime → Change runtime type)
3. Add `GROQ_API_KEY` via the 🔑 Secrets panel
4. Upload `data/annotated.csv` when prompted
5. Fill in the label map cell and run all cells
6. Download `evaluation_results.json` and `confusion_matrix.png` → save to `results/`

---

## Evaluation Report

*(Fill in after running the Colab notebook)*

### Overall Accuracy

| Model | Accuracy | F1 (macro) |
|---|---|---|
| Groq zero-shot baseline | — | — |
| Fine-tuned DistilBERT | — | — |

### Per-class F1

| Label | Baseline F1 | Fine-tuned F1 |
|---|---|---|
| ANALYTICAL | — | — |
| OPINIONATED | — | — |
| REACTIVE | — | — |
| SOCIAL | — | — |

### Confusion matrix

*(paste or embed `results/confusion_matrix.png` here)*

### Examples the model got wrong

*(3+ examples from Colab output, with analysis)*

1. **Example 1:** True: X, Predicted: Y. Analysis: ...
2. **Example 2:** ...
3. **Example 3:** ...

### Reflection

*(1–2 paragraphs: what did the model actually learn vs. what you intended? Where did the zero-shot baseline beat fine-tuning?)*

---

## File Structure

```
.
├── planning.md              # Label taxonomy, edge cases, project plan
├── README.md
├── requirements.txt         # Local deps only (praw, python-dotenv)
├── .env.example
├── .gitignore
├── data/
│   ├── raw/                 # Collected posts (gitignored)
│   ├── annotated.csv        # Your 200+ labeled examples (committed)
│   └── processed/           # Train/val/test splits (produced by Colab)
├── scripts/
│   ├── collect_data.py      # Reddit scraper
│   └── annotate.py          # Terminal annotation tool
└── results/                 # Downloaded from Colab after training
    ├── evaluation_results.json
    └── confusion_matrix.png
```
