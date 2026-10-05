# The Unofficial Guide

Tanisha Jain — corpus: `campus_life`

> Note: unit 1 was completed late, at the start of unit 2. The questions and
> criteria were committed (34c8d9d) before any eval run, so the order in the
> history is still criteria → results.

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

A question-answering system over `campus_life`: 88 short student-written posts
about a fictional university's dining halls, dorms, courses, transit and admin
rules. You ask a plain-English question ("How late is the library open during
reading week?"), it retrieves the most relevant posts with a local embedding
model, refuses if nothing is close enough, and otherwise has Gemini answer
using only those posts, naming the file it used.

## Chunking Strategy

**Chunk size:** one whole post per chunk (posts over 800 characters would be split on paragraph breaks — none are)
**Overlap:** none

Every post is one short thought: they average 317 characters and the longest
(`housing_old_brewhouse.txt`) is 554. Splitting them would only separate a
building's name from its facts — "Laundry in Morrow House" from "$1.50 wash,
$1.25 dry" — and since seven laundry posts are otherwise word-for-word
identical, a chunk without the building name is useless. So
`chunker.py::split_documents` keeps each post whole (88 documents → 88
chunks), with a paragraph-boundary split as a safety net for anything over
`CHUNK_SIZE`. Overlap only matters when you cut, so there is none.

## Sample Chunks

<!-- Five chunks, pasted as text. Label each one and name the file it came from
     AND the function that produced it — the grader checks your code against
     what you claim here.

     `python app.py chunks -n 5` prints all three for you. Copy them straight
     across.

     Milestone 3. -->

**Chunk 1** — source: `admin_add_drop_deadline.txt#0` — produced by: `chunker.py::split_documents`

```
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.
```

**Chunk 2** — source: `course_biol_160.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology

I lived here my sophomore year. Format is lecture three times a week with a weekly lab. Assessment: four unit tests and a cumulative final. Not curved.

Expect 9 to 11 hours a week, the heaviest first-year course by reputation.

The one piece of advice: the unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_hist_118_workload.txt#0` — produced by: `chunker.py::split_documents`

```
Workload for HIST 118 Modern World History

People keep asking so: a lot of reading, about 120 pages a week, but no problem sets. That's real time, not optimistic time.

It's front-loaded — the first month is heavier than the rest, partly because you're learning the format.
```

**Chunk 4** — source: `dining_pellew_dining_hall_followup.txt#0` — produced by: `chunker.py::split_documents`

```
Re: Pellew Dining Hall

Adding to what people have said about Pellew Dining Hall. The wait figure of 12 to 18 minutes at peak matches what I've seen. If you're trying to eat between classes, go before 11:45 and it's a different building entirely.

Also worth saying: the furthest hall from anywhere, next to the athletics centre. Nobody tells you this at orientation.
```

**Chunk 5** — source: `housing_innisfree_hall.txt#0` — produced by: `chunker.py::split_documents`

```
Innisfree Hall — what it's actually like

Transferred in last year, so take this with a grain of salt. Built 1991, renovated 2022. Rooms are doubles arranged as pairs sharing one bathroom between two rooms.

The good: the shared-bathroom-between-two-rooms arrangement is the best compromise on campus.

The bad: no air conditioning, which matters for the first three weeks of September.

Laundry costs $1.75 wash, $1.75 dry, app-based. On noise: moderate; the building is L-shaped and the short wing is much quieter.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** How much does it cost to use a dryer in Morrow House?

**Answer:** (`python app.py ask ...`, best distance 0.320, cutoff 0.6)

```
It costs $1.25 to use a dryer in Morrow House.

Source: housing_morrow_house.txt (also found in housing_morrow_house_laundry.txt)

Sources retrieved: housing_aldridge_hall_laundry.txt, housing_innisfree_hall_laundry.txt, housing_morrow_house.txt, housing_morrow_house_laundry.txt, housing_old_brewhouse_laundry.txt
```

**My relevance cutoff:** 0.6 (kept the default, because my own measurements put it in the middle of a clean gap)

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

The five in-corpus questions landed between 0.320 and 0.463; the five
out-of-corpus ones between 0.825 and 0.934. Nothing falls between 0.463 and
0.825, so any cutoff in that range separates them; 0.6 leaves ~0.14 of margin
above the worst in-corpus question and ~0.22 below the closest out-of-corpus one.

| Question | In corpus? | Best distance |
|---|---|---|
| How often does the campus shuttle run on weekends? | yes | 0.411 |
| How much does it cost to use a dryer in Morrow House? | yes | 0.320 |
| What is the last week you can withdraw from a course? | yes | 0.406 |
| How late is the library open during reading week? | yes | 0.427 |
| Is a cash meal cheaper at Halden Hall or Kestrel Commons? | yes | 0.463 |
| What is the capital of Mongolia? | no | 0.825 |
| How do I change the oil in a diesel engine? | no | 0.934 |
| Who won the 1994 World Cup? | no | 0.886 |
| What is the recommended dosage of ibuprofen for a headache? | no | 0.844 |
| How do I write a for loop in Rust? | no | 0.896 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1. Writing the questions and criteria.** I asked Claude (Claude Code) to
draft my five test questions and criteria 4 and 5 from the `campus_life`
documents. It read the whole corpus and deliberately picked questions with
traps — seven near-identical laundry posts, the drop vs. withdrawal deadlines,
seven noise posts that all say "library open until 2am". It also added an
`answer_in` field to each question so criterion 1 could be checked
mechanically instead of by my judgment. I reviewed them and committed them
before running anything.

**2. Chunking and the cutoff.** Claude suggested keeping each post as one
chunk rather than tuning the 800/120 numbers, because no post reaches 800
characters, and measured the ten best distances that set the cutoff. I kept
0.6 because the measured gap (0.463 → 0.825) showed it was already safe.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

Source file: [`results/run_2026-10-04_2112_before.md`](results/run_2026-10-04_2112_before.md),
produced by `run_eval.py::main` (3 runs, cache off, 15 real model calls).
Per-criterion counts aggregated from it by `criteria_report.py::main`; answers
scored by `scorer.py::judge`. Corpus `campus_life`, whole-post chunks
(`chunker.py::split_documents`), top-k 5, cutoff 0.6.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunks contain the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Rank-1 chunk is from the answer document | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 5. Answer states the correct fact | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

Criteria 3 and 4 are identical across runs by construction: both depend only on
retrieval, which is deterministic. Criteria 2 and 5 depend on the generated
answer and *could* have moved; they didn't. The runs were real — the wording
differs between runs (e.g. Morrow House run 1 cites
"`housing_morrow_house_laundry.txt` (also mentioned in
`housing_morrow_house.txt`)", run 2 "Sources: housing_morrow_house_laundry.txt
and housing_morrow_house.txt") — the facts just didn't.

**Real output, run 1** (from `results/run_2026-10-04_2112_before.md`):

*Criterion 1* — `store.py::search`, sources retrieved for the comparison question (both `answer_in` documents present):
```
- Best distance: 0.4630 (passed the gate)
- Sources retrieved: admin_dining_dollars.txt, dining_halden_hall.txt, dining_kestrel_commons.txt, dining_north_kitchen.txt, dining_pellew_dining_hall.txt
```

*Criterion 2* — `generate.py::answer_from_chunks`:
```
You can withdraw from a course through week ten, according to admin_withdrawal_deadline.txt.
```

*Criterion 3* — `run_eval.py::check_out_of_scope`, cutoff 0.6:
```
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.934 | refused |
| Who won the 1994 World Cup? | 0.886 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused |
| How do I write a for loop in Rust? | 0.896 | refused |
```

*Criterion 4* — `store.py::search` ranking, the one question where rank 1 was wrong:
```
How late is the library open during reading week?
  1. admin_library_holds.txt          0.427   <- not the answer document
  2. study_library_hours.txt          0.450   <- the answer
  3. housing_morrow_house_noise.txt   0.485
  4. housing_calder_annexe_noise.txt  0.497
  5. dining_north_kitchen_followup.txt 0.523
```

*Criterion 5* — `generate.py::answer_from_chunks`, scored by `scorer.py::judge`:
```
The library is open until 10pm during reading week (from study_library_hours.txt).
A cash meal is cheaper at Halden Hall, where it costs $10.00 cash compared to $12.50 cash at Kestrel Commons (dining_halden_hall.txt and dining_kestrel_commons.txt).
```

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunks contain the answer (4 of 5) | MET | 5/5 in all three runs; every `answer_in` document was in the top 5 every time, so there's no generous reading involved. |
| 2 | Every answer names a source (5 of 5) | MET | All 15 answers contain a corpus filename; the strictest target I set, and it held in every run. |
| 3 | Gate stops out-of-corpus questions (4 of 5) | MET | 5/5 refused; the closest out-of-scope question (Mongolia, 0.825) is still 0.225 over the cutoff. |
| 4 | Rank-1 chunk is from the answer document (4 of 5) | MET — barely | 4/5 in all three runs, exactly on the target. The opposite case: one more wrong rank-1 misses it, and the library question shows the mechanism that would cause it. But the target was 4 of 5, it held 4 of 5 in every run, and the miss is the question I predicted would be hard. |
| 5 | Answer states the correct fact (4 of 5) | MET | 15/15 answers contained the expected fact. I also read the Halden/Kestrel answers by hand since the scorer only checks "Halden" appears; all three state Halden $10.00 vs Kestrel $12.50 correctly. |

## Diagnoses

**Nothing was missed.** That mostly says my targets were safe, not that the
system is excellent: four of five criteria cleared their target with room to
spare, on questions I chose knowing the corpus. If I were setting them again
I'd tighten criterion 1 and criterion 5 to **5 of 5** (both came out 15/15) and
criterion 4 to **5 of 5**, since it's the only one with real signal.

The two near-misses are still worth diagnosing, because they share a cause and
they're the first things that would break with harder questions:

1. **Criterion 4, the library question — stage: embedding/retrieval.** The
   answer document `study_library_hours.txt` ranks 2nd (0.450) behind
   `admin_library_holds.txt` (0.427). The whole post is embedded as a single
   384-dim vector, and both posts are overwhelmingly "about the library", so
   they land close together. The words that actually separate them — "reading
   week", "10pm" — are a handful of tokens in a 300-character post and barely
   move its vector. Generation recovered (it read the 2nd chunk), so the
   answer was right, but retrieval was wrong.

2. **The comparison question — stage: retrieval.** `dining_kestrel_commons.txt`
   came in **5th of 5** (0.598), behind North Kitchen (0.493) and Pellew
   (0.571), which the question never mentions. Every dining post has the same
   template ("Wait times: ... Hours are ... Costs one meal swipe, or $X cash"),
   so to the embedding they are all "a dining hall post about cost". The
   question's exact name, "Kestrel Commons", is the strongest signal and
   semantic search treats it as just more dining vocabulary. At top-k 4 this
   question would have failed criterion 1 and criterion 5.

**Pattern:** both are the same problem. The corpus is full of near-template
posts (7 dining, 7 laundry, 7 noise, 7 course triples) that differ only in
proper names and specific terms, and meaning-only retrieval underweights
exactly those exact terms.

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
