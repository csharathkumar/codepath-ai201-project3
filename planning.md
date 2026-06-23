# TakeMeter — Planning Document

---

## 1. Community

**Chosen community:** r/movies and r/TrueFilm (Reddit film discussion communities)

r/movies is one of Reddit's largest subreddits (~35M members), where people post reactions to trailers, news, reviews, and discussion threads about films. r/TrueFilm is a smaller, more focused companion sub that explicitly requires substantive discussion — no memes, no low-effort posts. Together they create a spectrum: r/movies runs from "this movie SLAPPED" to thoughtful craft analysis, while r/TrueFilm skews analytical.

This community is a strong fit for classification because discourse quality varies enormously and in identifiable ways. A regular participant in either community immediately recognizes the difference between a one-line hot take and a paragraph analyzing a director's use of negative space — and they have opinions about which is more valuable. That shared intuition is what makes the distinction learnable. The community is also text-heavy and public, which makes data collection straightforward.

---

## 2. Labels

**Four labels:** ANALYTICAL, OPINIONATED, REACTIVE, SOCIAL

### ANALYTICAL (label 0)
A post that makes a specific, evidence-based argument about craft, narrative, theme, or historical/genre context — one where the claim would still stand if you stripped out all the opinion language.

**Examples:**
- *"Villeneuve's use of silence in Dune isn't just aesthetic — it mirrors how Paul experiences prescience: information arriving before meaning. The quieter scenes are structurally doing the same work as the exposition."*
- *"What made Breaking Bad's finale land is that it rejected catharsis for consequence. Walt doesn't get redemption, he gets closure — and the show is careful not to conflate the two."*

**Key signal:** Remove "I think / I loved / this was amazing" — does a specific checkable claim remain? If yes: ANALYTICAL.

---

### OPINIONATED (label 1)
A post that takes a clear stance and gives a reason, but the reason is assertion or feeling rather than craft analysis — the author tells you what they thought and why, without unpacking how the film produces that effect.

**Examples:**
- *"Hereditary scared me more than any horror film in years. The family dynamics felt genuinely realistic which made the supernatural stuff hit harder."*
- *"I think people are way too harsh on the Hobbit trilogy — it's baggy but it's still visually spectacular and the cast is great."*

**Key signal:** Has a position and a reason, but the reason is "I felt X because Y" rather than "X works because of how Z is constructed."

---

### REACTIVE (label 2)
A post that expresses an emotional response — excitement, disgust, hype, disappointment — with no supporting argument. Includes hyperbole, one-liners, ratings without explanation, and emotional dumps.

**Examples:**
- *"GOAT tier. Not even debatable."*
- *"Absolute garbage. Two hours of my life I'll never get back."*

**Key signal:** Nothing in the post could be argued with — it is pure reaction.

---

### SOCIAL (label 3)
A post that is not primarily about evaluating a film or show — instead it facilitates discussion, asks questions, shares news, makes recommendations, or engages socially with the community.

**Examples:**
- *"What's everyone watching this weekend?"*
- *"Can someone explain the ending of Donnie Darko to me?"*

**Key signal:** Not making a take — organizing conversation or sharing information.

---

### Why four and not three?
REACTIVE and OPINIONATED are meaningfully different: one has reasoning, one doesn't. Collapsing them into "opinion" would lose the distinction between "I hated it because the pacing was off" (Opinionated — has a reason) and "Absolute garbage" (Reactive — no reason). That distinction is exactly what the community cares about.

---

## 3. Hard Edge Cases

**The critical boundary: ANALYTICAL vs. OPINIONATED**

This is the hardest call. The test: *strip the opinion language — does a specific, checkable claim about how the film works remain?*

- If yes → ANALYTICAL
- If no → OPINIONATED

The deciding factor is **specificity**, not sophistication. A long eloquent post that asserts without analyzing is still OPINIONATED. A short post that names a specific technique and explains its effect is ANALYTICAL.

**Pre-annotation worked example:**
*"The third act is where most blockbusters fall apart, and this one is no exception."*
- Could be ANALYTICAL (craft-level claim about structure) or OPINIONATED (no specifics about this film).
- **Decision: OPINIONATED** — the craft framing is borrowed, not applied. No specific scene or mechanism is named. Strip the opinion: nothing checkable remains.

**Pre-annotation worked example:**
*"Does anyone else feel like the color grading in this film is doing a lot of heavy lifting emotionally?"*
- Could be SOCIAL (it's a question) or ANALYTICAL (craft observation).
- **Decision: ANALYTICAL** — the question is rhetorical. Strip it: a specific craft claim (color grading, emotional weight) remains.

**Hard case from actual annotation (OPINIONATED vs. ANALYTICAL):**
*"In every single movie of the series, there is a horrible ratio between action scene volume and conversation volume. Every time, I need to up the volume so I can at least understand what's being said..."*
- The post identifies a specific, observable pattern across a franchise (volume mixing). That sounds like a craft observation.
- But it doesn't explain *why* this happens or connect it to any intentional filmmaking choice — it's describing a frustration, not analyzing a technique.
- **Decision: OPINIONATED** (Groq confidence: 0.80 — the lowest in the dataset). The observation is specific but the reasoning stays at the level of personal reaction. No mechanism or intent is analyzed.

**Hard case from actual annotation (REACTIVE vs. OPINIONATED):**
*"Inglourious Basterds was a far better film."*
- Four words with a comparative claim — "far better" implies a standard of comparison.
- But there's no reason given, no context, no criterion for "better."
- **Decision: REACTIVE** — the comparative framing gives the illusion of an argument, but strip it: nothing remains except a preference. A reason would need to accompany the comparison to reach OPINIONATED.

**Hard case from actual annotation (OPINIONATED vs. ANALYTICAL):**
*"I've heard over and over that Vin Diesel and Terry Crews are big nerds who play D&D. I've even heard they play together. Mr. Diesel also seems to have a lot of input into his movies, even having director approval..."*
- Assembles specific facts about a filmmaker's background and creative control.
- Could be ANALYTICAL (connecting biographical context to film output) but never makes the connection — the post stops at observation without drawing a conclusion about the films.
- **Decision: OPINIONATED** — specific facts without a claim about what those facts mean for the work. Strip the framing: no checkable argument about craft remains.

**Other decision rules:**

| Case | Rule |
|---|---|
| Analytical post with emotional language | ANALYTICAL — tone doesn't override a specific craft argument |
| Reactive post with one vague "because" | REACTIVE — a single undeveloped assertion isn't reasoning |
| Social post with an embedded opinion | SOCIAL — classify by dominant intent |
| Mixed posts | Dominant intent wins |

---

## 4. Data Collection Plan

**Sources:**
- r/movies text posts (filtered `is_self=true` — skips link/news posts)
- r/movies comments (top-scored — better discourse quality signal than new/hot)
- r/TrueFilm text posts and comments (skews ANALYTICAL, balances the dataset)

**Tool:** Pullpush.io public Reddit archive (no API key required), via `scripts/collect_data.py`.

**Target per label:** 50 examples minimum, ideally 55–60 for buffer.

**Actual results (completed):** 214 total examples — ANALYTICAL: 54, OPINIONATED: 42, REACTIVE: 68, SOCIAL: 50. OPINIONATED is the thinnest class at 42 (19.6%), still above the 20% floor. If it had dropped below 40, the plan was to re-run collection targeting r/TrueFilm comments specifically, which generate more reasoning-based posts.

**Annotation method:** Groq (`llama-3.3-70b-versatile`) pre-labeled each post; human reviewer accepted or overrode each suggestion. All 214 were accepted without override (see AI Tool Plan for disclosure implications).

**If a label is underrepresented after 200 examples:** Run collection again filtering to the source most likely to produce that label — r/TrueFilm for ANALYTICAL, r/movies hot comments for REACTIVE. Do not rebalance by duplicating examples.

---

## 5. Evaluation Metrics

**Primary metrics:** F1 (macro) and per-class F1.

**Why not accuracy alone:**
Accuracy rewards a model that learns to predict the majority class. In this dataset REACTIVE is 31.8% of examples — a model that predicted REACTIVE for everything would be ~32% accurate, which sounds reasonable but is useless. Macro F1 averages F1 across all four classes equally, which means a class with 42 examples counts as much as one with 68. That's the right measure when all four labels matter equally.

**Per-class F1** is also essential because failure modes are class-specific. I expect the model to struggle most on ANALYTICAL vs. OPINIONATED — macro F1 would hide that if the other three classes are easy. Per-class F1 surfaces it.

**Confusion matrix:** Required to see exactly which pairs of labels the model conflates. The ANALYTICAL/OPINIONATED cell is the one to watch.

**Baseline comparison:** Zero-shot `llama-3.3-70b-versatile` via Groq on the same test set. This tells us whether fine-tuning actually helped — if the fine-tuned model doesn't beat the baseline by a meaningful margin, it learned something wrong.

---

## 6. Definition of Success

A classifier is genuinely useful for a community moderation or recommendation tool if it can reliably distinguish substantive posts from noise. That means:

| Threshold | Meaning |
|---|---|
| Macro F1 ≥ 0.70 | Minimum acceptable — model is better than guessing on all classes |
| Macro F1 ≥ 0.75 | "Good enough" — useful as a filter, acceptable for deployment in a low-stakes tool (e.g., sorting posts in a feed) |
| Macro F1 ≥ 0.80 | Strong — I would trust it to pre-label new examples with human spot-checking |
| ANALYTICAL F1 ≥ 0.65 | Minimum for that class — it's the hardest and most subjective |

**What would I not accept:** A model that achieves high overall accuracy by mislabeling most ANALYTICAL posts as OPINIONATED. That's the failure mode that matters most, because ANALYTICAL is the label the community cares about surfacing.

**Comparison to baseline:** Fine-tuning should improve macro F1 by at least 5 points over zero-shot Groq. If it doesn't, that's a signal that the dataset is too small or the labels too ambiguous for fine-tuning to add signal.

---

## 7. AI Tool Plan

### Label stress-testing
Before annotating, I used Claude to generate boundary posts — examples designed to sit exactly between ANALYTICAL and OPINIONATED. Prompt: *"Generate 8 r/movies comments that would be genuinely hard to classify as ANALYTICAL vs. OPINIONATED using these definitions: [definitions]."* This revealed that the "strip the opinion language" test was necessary — without it, sophisticated-sounding OPINIONATED posts were hard to distinguish from weak ANALYTICAL ones.

### Annotation assistance
Groq (`llama-3.3-70b-versatile`) pre-labeled all 214 examples using a structured prompt with the full label definitions. The human reviewer (me) saw the suggestion and confidence before deciding. All 214 were accepted without override.

**Disclosure:** Because there were zero overrides, the annotations reflect Groq's judgment applied to my label definitions, not independent human judgment. This is a real limitation — the model may have learned Groq's classification heuristics rather than the definitions themselves. This will be noted in the evaluation reflection.

**Tracking:** The `annotated.csv` file includes `groq_suggestion` and `groq_confidence` columns for every row, so it is always clear which label came from the model vs. a human decision.

### Failure analysis
After training, I will export the wrong predictions from the Colab notebook and prompt Claude: *"Here are N posts the model misclassified. What patterns do you see in the errors? Focus on systematic patterns, not individual cases."* I will then verify each proposed pattern by reading the examples myself before including it in the evaluation write-up.

---

## Stretch Features

- [ ] Inter-annotator reliability — have someone else label 30+ examples and compute Cohen's kappa
- [ ] Confidence calibration — does 90% confidence actually predict correctness better than 60%?
- [ ] Error pattern analysis (planned — see AI Tool Plan above)
- [ ] Deployed Gradio interface

---

## Timeline

| Step | Status | Notes |
|---|---|---|
| Label taxonomy + pilot read | ✅ Done | This document |
| Data collection (200+ records) | ✅ Done | 214 examples via Pullpush.io |
| Annotation | ✅ Done | Groq-assisted, 214 labeled |
| Fine-tuning + baseline + eval | ⬜ Next | Colab starter notebook (T4 GPU) |
| README evaluation section | ⬜ Pending | Fill in after Colab |
