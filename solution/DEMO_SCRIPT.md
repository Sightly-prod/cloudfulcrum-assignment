# Video Demo Script — Transcript Intelligence

**Target length:** ~7–8 minutes (the brief asks for 5–10).
**Format:** Screen recording with narration. Have `notebook.ipynb` open in Jupyter, a terminal in `solution/`, and the `outputs/figures/` folder ready in Finder if you want to show artefacts.

**Pacing tip:** Don't read the markdown cells out loud — paraphrase. The cells stay on screen; let the viewer skim while you talk.

---

## 0. Pre-flight checklist (do BEFORE recording)

- [ ] Restart kernel and clear all outputs, then re-run all cells once so the notebook is clean.
- [ ] Zoom the browser to ~125% so figures are legible at recording resolution.
- [ ] Close noisy tabs / Slack / mail notifications.
- [ ] Have these files ready in tabs: `notebook.ipynb`, `src/` folder, `outputs/figures/`.
- [ ] Test mic levels with a 10-second sample.

---

## 1. Intro — who you are and the problem  (~0:30)

**On screen:** notebook top, title cell visible.

> "Hi, I'm Palkesh. This is my walkthrough of the Transcript Intelligence take-home.
>
> The dataset is 100 call transcripts — support, external, and internal. The brief
> asked for topic categorisation, sentiment trends, and at least two bonus insights.
> I'll show you the pipeline running, the four insight modules I built, and then
> the three additional ideas I didn't code up.
>
> The story is in the slide deck; this notebook is the technical reference.
> Code lives in `src/`, all figures and tables are saved to `outputs/`."

---

## 2. Pipeline & setup  (~0:45)

**On screen:** open a terminal in `solution/`, then back to the notebook.

> "Everything is reproducible. From a clean checkout, one command runs the whole pipeline:"

```bash
python -m src.run_pipeline
```

> "It takes about 30 seconds and is deterministic — same seeds, same outputs. The
> notebook itself doesn't re-run the pipeline; it just loads the cached parquets
> and CSVs from `data/` and the figures from `outputs/figures/`. That keeps the
> implementation in `src/` and the narrative in the notebook cleanly separated."

**Switch to the notebook.** Scroll through cells 1–3 quickly.

> "Section 1 covers ingestion. Each transcript ships as six JSON files — meeting
> info, transcript with per-sentence sentiment, summary with pre-labels, speakers,
> events, speaker metadata. The dataset is pre-labelled, which is convenient, but
> I built an independent pipeline anyway so I can defend every number and so it
> would work on raw transcripts where no labels exist.
>
> Call type is inferred from three signals: title prefix, attendee domains, and
> fallback heuristics. Customer linking uses external email domains. 70 of 100
> calls have a customer; 30 are pure-internal."

**Run cell 4** — show `01_call_type_breakdown.png`.

> "Quick sanity check on the split: 43 external, 32 internal, 25 support."

---

## 3. Task 1 — Topic categorisation  (~1:30)

**On screen:** scroll to section 2.

> "Task 1: categorise the transcripts. I went with a hybrid embedding-based
> clustering approach — sentence-transformers to embed each call summary, KMeans
> with silhouette-selected k, TF-IDF for top terms per cluster, then human-curated
> labels.
>
> Why not pure rule-based or keyword matching? Calls use domain language that
> doesn't share keywords — 'Detect alert latency' and 'pager fatigue' are the same
> theme but share zero tokens. Embeddings catch that.
>
> Why not pure LLM-classify? Cost and reproducibility. With 100 calls and 30k+
> sentences, embeddings give me a deterministic, defensible clustering for
> essentially zero cost. I did use Claude *optionally* for nicer cluster labels —
> there's a cache in `data/gemini_cache/` so the notebook is reproducible without
> a key."

**Run cell 6** — the cluster list.

> "Silhouette picked 9 clusters. Each shows its top TF-IDF terms, size, and a
> curated label. The biggest theme is Compliance & Audit Prep at 20 calls, which
> turns out to be a really interesting signal."

**Run cell 7** — `02_topic_clusters_by_calltype.png`.

> "Here's the cluster distribution by call type. A few things jump out: compliance
> spans all three call types, engineering syncs are obviously internal-heavy, and
> the Detect product issues cluster is concentrated in customer-facing calls —
> we'll come back to that."

**Skim cells 9–11.**

> "I also cross-validated against the pre-labelled `summary.topics`. There are
> 351 unique fine-grained tags in the data; my 9 clusters cleanly absorb subsets
> of them. So my coarser clustering is consistent with the finer pre-labels — it
> didn't invent themes out of thin air."

---

## 4. Task 2 — Sentiment trends  (~1:30)

**On screen:** scroll to section 3.

> "Task 2: sentiment. The data ships with sentiment labels, but I re-scored every
> sentence independently with VADER — a rule-based lexicon — so I'm not just
> reading back the labels someone else generated."

**Run cell 13** — show Pearson r and the per-call-type chart.

> "Pearson correlation between the provided score and my VADER mean is 0.83.
> Strong agreement at the call level, so I can read the trends as real signal,
> not artefact of one labeller's bias."

**Run cells 15 and 16** — the rollup table and stacked charts.

> "Three trends. First, support is the most negative call type — 23% negative
> sentences, about twice internal. Customers don't call support to chat; the
> percentage-negative is a leading indicator of escalation risk.
>
> Second, external calls skew most positive — 14 of 43 are 'very positive'.
> Account managers are running mostly healthy meetings.
>
> Third, internal calls have the *fewest* positive sentences but mid-range overall
> score. Engineers don't gush in syncs; they discuss problems. That's normal —
> the danger sign is when internal sentiment shifts to negative, which in this
> data correlates with live incident calls."

**Run cell 18** — `05_sentiment_by_topic.png`.

> "But here's the chart I'd take to leadership. Sentiment broken out by topic.
>
> Detect product issues scores 2.20 — whenever this theme comes up, the call goes
> negatively. What's *striking* is that 8 of the 10 calls in this theme are
> customer-facing. The Detect outage isn't just an internal engineering problem
> any more — it's leaking into renewals and competitive evals.
>
> And Compliance & Audit Prep scores 4.44 — the best-functioning workstream. The
> takeaway for product: compliance is selling; lean into it."

---

## 5. Bonus 1 — Customer journey  (~1:15)

**On screen:** scroll to section 4.

> "Now the bonus work. The first one is cross-call entity linking — building a
> per-customer journey across all their touchpoints, regardless of call type.
>
> The idea: a single customer shows up in support tickets, AM calls, sometimes
> in internal calls. Each one is a sentiment reading. Strung together, you get a
> story no individual call can tell."

**Run cell 21** — show the multi-call customers table.

> "These are the customers with the most call-type coverage in our 100-sample
> dataset."

**Run cell 22** — `10_journey_brightpathcommerce.png`.

> "This is the canonical case — Brightpath Commerce. Four touchpoints over two
> months.
>
> February 16th: Detect deployment kickoff, sentiment 3.9. New deal, good vibes.
>
> Late Feb: a support case about slow backups — first friction, 2.8.
>
> March 1: invoice discrepancy, resolved cleanly at 3.7.
>
> Late April: an external call titled *Competitive Evaluation*, score 2.6.
>
> *That* is the alarm bell. A CSM looking at only the latest AM call sees 'a
> slightly negative meeting.' A CSM with the full journey sees: 'first call
> positive, then bumps, then a competitive eval — they're shopping.'
>
> The control group of healthier accounts is shown below for contrast."

**Run cell 24** quickly — the comparison journeys.

---

## 6. Bonus 2 — Churn risk score  (~1:15)

**On screen:** scroll to section 5.

> "Bonus 2 builds on the journey work. A composite churn risk score per external
> customer, 0–100, weighted across five signals: low average sentiment, negative
> trajectory over time, support-call load, frequency of 'churn-y' key moments,
> and direct competitor mentions in transcripts.
>
> Weights are documented in the cell above — I picked them so the score is
> interpretable, not learned. With 30 customers this isn't enough to train
> anything; the goal is a defensible heuristic."

**Run cell 26** — leaderboard.

**Run cell 27** — the two churn-risk charts.

> "Brightpath comes out at 54, At-risk — driven heavily by 10 competitor mentions
> and a declining trajectory. Same story as the journey chart, now scored.
>
> Ridgeline at 53 — two support tickets, three churn key-moments, declining
> trajectory. The Detect latency issues are cascading for them.
>
> Northstar Pharma at 51 is interesting *and* a caveat — it's the lowest mean
> sentiment customer, but with only one call. I called this out explicitly: with
> n=1 the score is fragile. We'd want more touchpoints before any action.
>
> Six At-risk plus six Watch accounts is about 38% of the external base — that's
> the cohort any product launch or change-management cycle should consider first."

**Briefly mention cell 29** — the "what's missing" section.

> "I also called out the limitations honestly: no product telemetry, no recency
> weighting, small sample. The score is a triage signal, not a verdict."

---

## 7. Bonus 3 — Speaker dynamics  (~1:00)

**On screen:** scroll to section 6.

> "Bonus 3: speaker dynamics. In sales and CS, listen-to-talk ratio is one of the
> most reliable predictors of call quality — top reps talk less than 50% of the
> time and ask more questions. We have speaker-segmented transcripts, so we can
> compute it directly."

**Run cell 31** — talk-time table.

**Run cell 32** — `08_speaker_dynamics.png`.

> "On the average external call the AM talks 57% — slightly above the 50%
> 'consultative' threshold. Most reps are fine. The right-quadrant outliers
> where AM talk is above 60% *and* the customer's question ratio is high — those
> are the coaching candidates. The customer is trying to engage; the AM is
> monologuing. Classic coachable moment."

**Run cell 34** — coaching candidates list.

> "So this gives a CS leader a concrete shortlist of calls to review with their
> AMs — and a metric to track over time."

---

## 8. Additional ideas + wrap  (~0:45)

**On screen:** scroll to section 7.

> "Finally, three additional ideas I scoped but didn't code up — written for the
> panel discussion:
>
> A. **Action-item accountability tracker** — parse the action items into
> owner-verb-due-date triples; detect recurring owners. For directors who want
> load-balancing data.
>
> B. **Product-feature → call-type heatmap** — tie product names like Detect or
> Comply to topic and sentiment. Gives product leadership a roadmap-prioritisation
> view of which features drive positive vs negative calls.
>
> C. **Internal-to-external response time** — measure the lag between an internal
> call about a customer and the next external call with that customer. Hypothesis:
> faster follow-up correlates with better sentiment outcomes.
>
> D. **Cross-customer issue clustering** — flag when three or more customers raise
> variants of the same technical issue in a short window. Embedding-based near-
> duplicate detection on complaint text. An early-warning signal for engineering
> on-call.
>
> That's the walkthrough. Code is in `src/`, all figures and tables are in
> `outputs/`, slides tell the story side. Thanks for watching."

---

## Recording tips

- **Don't read markdown cells.** Glance, paraphrase, move on. Saying every word makes the demo feel slow.
- **Cursor as pointer.** When you say "this number," move the cursor to it. Cheaper than annotation.
- **One re-take per section, max.** A small stumble is fine — the panel knows it's a screencast.
- **Audio first.** Bad video is forgivable, bad audio isn't. Use a quiet room and a decent mic.
- **Export at 1080p.** Anything lower and the charts get muddy.
