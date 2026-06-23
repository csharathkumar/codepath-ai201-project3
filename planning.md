# TakeMeter — Planning Document

## Community: r/movies (and r/television)

Movie and TV subreddits are dense with discourse: hot takes on new releases, retrospective reappraisals, actor/director deep dives, and a lot of noise in between. The quality of a "take" in these communities varies widely — from a paragraph of thoughtful craft analysis to "this movie sucked lol." This project tries to make that distinction machine-readable.

---

## Label Taxonomy

### Why 4 labels?

Pilot reading of ~40 r/movies posts revealed a clear four-way split. Two labels ("analytical" vs. "reactive") were obvious; the middle ground needed subdividing because "has an opinion" covers wildly different post quality.

---

### Label Definitions

| Label | Name | One-line definition |
|---|---|---|
| 0 | **ANALYTICAL** | Makes a specific, evidence-based argument about craft, narrative, theme, or context |
| 1 | **OPINIONATED** | States a clear stance with some reasoning, but doesn't go deep on craft |
| 2 | **REACTIVE** | Pure emotional reaction — no argument, no reasoning, just a response |
| 3 | **SOCIAL** | Meta-discussion, questions, memes, or off-topic engagement |

---

### Detailed Definitions + Examples

#### 0 — ANALYTICAL
The post makes a *specific claim* and *supports it* with evidence drawn from the film/show itself or relevant context (director's filmography, genre conventions, cinematographic technique, narrative structure, etc.). The take would be informative to someone who hadn't thought about it that way.

**Examples:**
- "Villeneuve's use of silence in Dune isn't just aesthetic — it mirrors how Paul experiences prescience: information arriving before meaning. The quieter scenes are structurally doing the same work as the exposition."
- "The first season of The Bear works because every episode is formally mimicking the chaos it depicts. The single-take ep isn't a flex — it's the argument."
- "What made Breaking Bad's finale land is that it rejected catharsis for consequence. Walt doesn't get redemption, he gets closure — and the show is careful not to conflate the two."

**Key signal:** Would pass the "so what?" test — there's a claim + a reason.

---

#### 1 — OPINIONATED
The post has a discernible position *and* some reasoning, but the reasoning is more assertion than analysis. It might reference plot, performances, or vibes, but doesn't unpack *why* at the craft level.

**Examples:**
- "Hereditary scared me more than any horror film in years. The family dynamics felt genuinely realistic which made the supernatural stuff hit harder."
- "I think people are way too harsh on the Hobbit trilogy — it's baggy but it's still visually spectacular and the cast is great."
- "The ending of Severance season 2 felt rushed. Too many threads, not enough payoff for what the show set up."

**Key signal:** Has an opinion + a reason, but the reason is "I felt X because Y" rather than "X works because of how Z is constructed."

---

#### 2 — REACTIVE
A pure response — excitement, disgust, hype, disappointment — with no supporting argument. This includes hyperbolic takes, one-liners, and emotional dumps.

**Examples:**
- "GOAT tier. Not even debatable."
- "Absolute garbage. Two hours of my life I'll never get back."
- "Holy shit that ending!!! I was not prepared."
- "Ugh. Another superhero movie."
- "10/10 no notes"

**Key signal:** No claim that could be argued with — just a reaction.

---

#### 3 — SOCIAL
Anything that isn't primarily about evaluating a film/show: meta-questions, recommendation requests, meme-adjacent posts, news links without commentary, "is anyone else watching X?" posts.

**Examples:**
- "What's everyone watching this weekend?"
- "Can someone explain the ending of Donnie Darko to me?"
- "RIP to a legend 🙏" (on a director's death)
- "Reminder: new episode drops tonight"
- "Is the show worth picking up if I haven't seen the original?"

**Key signal:** Not making a take — facilitating discussion or sharing information.

---

### Exhaustiveness Check

From pilot reading of 40 posts:
- ANALYTICAL: ~20%
- OPINIONATED: ~30%
- REACTIVE: ~35%
- SOCIAL: ~15%

Total coverage: ~100% — no post required an "other" bucket in the pilot.

---

### Mutual Exclusivity Rules

**The critical boundary: ANALYTICAL vs. OPINIONATED**

This is the hardest call in the taxonomy. The test: *if you removed all the opinion language ("I think", "I loved", "this was amazing"), would the remaining text still contain a specific, checkable claim about how the film works?*

- If yes → ANALYTICAL
- If no → OPINIONATED

Example: *"Nolan's intercutting in Oppenheimer creates a false sense of simultaneity that flatters the audience — the Trinity test and the Senate hearing aren't morally equivalent, but the editing implies they are."* Strip the opinion framing: there's still a specific structural claim (intercutting, false simultaneity, implied moral equivalence). → ANALYTICAL.

Example: *"The pacing in the second act is where it falls apart — it just loses momentum."* Strip the opinion: "loses momentum" is a feeling, not a structural observation. No specific scene, technique, or mechanism named. → OPINIONATED.

The deciding factor is specificity, not sophistication. A long, eloquent post that asserts without analyzing is still OPINIONATED. A short post that names a specific technique and explains its effect is ANALYTICAL.

Other cases:

| Case | Decision rule |
|---|---|
| Analytical with emotional language | If there's a specific craft argument, it's ANALYTICAL regardless of tone |
| Reactive with a brief "because" | If the "because" is a single undeveloped assertion ("because the plot sucked"), it's REACTIVE |
| Social post with embedded opinion | If the post is primarily asking/facilitating and the opinion is incidental, it's SOCIAL |
| Mixed posts | Classify by the *dominant* intent |

---

### Edge Cases (Genuinely Difficult to Label)

These are the cases I expect to encounter during annotation. Document yours in the README.

**Case A:** *"The third act is where most blockbusters fall apart, and this one is no exception."*
- Could be ANALYTICAL (craft-level claim) or OPINIONATED (no specifics about *this* film's third act).
- **Decision:** OPINIONATED — the craft framing is borrowed, not applied. No specific scene or technique is analyzed.

**Case B:** *"I've seen this movie four times and it gets better every time. Fincher just operates on a different level."*
- REACTIVE vs. OPINIONATED — there's a re-watch claim but no argument.
- **Decision:** REACTIVE — "operates on a different level" is assertion without content.

**Case C:** *"Does anyone else feel like the color grading in this film is doing a lot of heavy lifting emotionally?"*
- SOCIAL (it's a question) vs. ANALYTICAL (craft-level observation).
- **Decision:** ANALYTICAL — the question is rhetorical; it contains a specific craft observation. The question form is just an invitation to discuss.

---

## Stretch Features

- [ ] Inter-annotator reliability (get a friend to label 30+ examples)
- [ ] Confidence calibration analysis
- [ ] Error pattern analysis
- [ ] Deployed Gradio interface

---

## Timeline

| Step | Est. time | Notes |
|---|---|---|
| Label taxonomy + pilot read | 1h | Done — this doc |
| Data collection (200+ posts) | 1–2h | `scripts/collect_data.py` locally |
| Annotation | 2–3h | `scripts/annotate.py`, aim for ≥40 per label |
| Fine-tuning + baseline + eval | 1–2h | Colab starter notebook (T4 GPU) |
| Write-up (README eval section) | 1h | Pull numbers from Colab output files |
