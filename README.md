# The Unofficial Guide

<!-- Replace this line with your name. -->

**Corpus:** `campus_life`

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

This project is a retrieval-augmented question-answering system for the
`campus_life` corpus, which contains short student-written posts and campus
administrative documents. It retrieves relevant chunks about topics such as
dining, housing, courses, transit, and university policies. A relevance gate
refuses questions that are not supported by the corpus, and the grounded
answer prompt requires the response to use only retrieved documents and name
the source file.

## Chunking Strategy

**Chunk size:** 450 characters
**Overlap:** 0 characters

The starter's 800-character windows produced 88 chunks for 88 documents,
because nearly every campus-life post was shorter than 800 characters. I kept
each post's paragraph structure and packed adjacent paragraphs up to 450
characters. This keeps the short factual posts together while preventing a
longer multi-topic post from becoming one oversized retrieval result. I used
no overlap because the useful facts are normally complete within a paragraph,
so duplicating text across chunks would add noise.

<!-- What about YOUR documents made you pick these numbers? Short posts and
     long sectioned guides don't want the same chunking, and "800 seemed
     reasonable" earns nothing. Point at something you noticed when you read
     the documents in Milestone 1.

     If you changed your mind partway through, say so and say why. That's worth
     more than pretending you got it right first time.

     Milestone 3. -->

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

**Chunk 2** — source: `course_biol_160_exams.txt#0` — produced by: `chunker.py::split_documents`

```
BIOL 160 Cell Biology — assessment

Four unit tests and a cumulative final. Not curved.

The unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.
```

**Chunk 3** — source: `course_math_220_exams.txt#0` — produced by: `chunker.py::split_documents`

```
MATH 220 Linear Algebra — assessment

Two midterms and a cumulative final. Curved to a b- median.

The problem sets are the course; the lectures make sense afterwards rather than during.
```

**Chunk 4** — source: `dining_the_ridgeway_cafe.txt#0` — produced by: `chunker.py::split_documents`

```
The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The thing worth going for is the only place on campus with real espresso. The thing to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal swipes.
```

**Chunk 5** — source: `housing_morrow_house.txt#0` — produced by: `chunker.py::split_documents`

```
Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.
```

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:**

How often does the campus shuttle run on weekdays, and how often does it run
on weekends?

**Answer:**

According to `transit_shuttle.txt`, the campus shuttle runs every 20 minutes on
weekdays and every 40 minutes on weekends. It is free with a student ID.

**Source:** `transit_shuttle.txt`

**My relevance cutoff:**

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

| Question | In corpus? | Best distance |
|---|---|---|
| How long are the wait times at Kestrel Commons between 12:15 and 1:00, and before 11:45? | Yes | 0.2331 |
| What are the rules for taking a course pass/fail outside my major? | Yes | 0.3004 |
| How much do washing and drying cost in Aldridge Hall, and when is the best time to do laundry? | Yes | 0.1854 |
| How often does the campus shuttle run on weekdays, and how often does it run on weekends? | Yes | 0.4748 |
| How many hours per week should I expect to spend outside class for CS 210, and when is the workload heaviest? | Yes | 0.1769 |
| What is the capital of Mongolia? | No | 0.8246 |
| How do I change the oil in a diesel engine? | No | 0.9340 |
| Who won the 1994 World Cup? | No | 0.8859 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.8442 |
| How do I write a for loop in Rust? | No | 0.8960 |

I chose a cutoff of **0.6** because the best in-scope distance was 0.4748 and
the closest out-of-scope distance was 0.8246, leaving a clear gap between the
two groups. With `TOP_K = 5`, the three questions I inspected retrieved
on-topic chunks near the top; the shuttle question's first result contained
both weekday and weekend frequencies. I reviewed `GROUNDING_INSTRUCTION` in
`generate.py` and kept it unchanged because it already requires document-only
answers, refusal when the documents do not cover the question, and the source
filename.

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked an AI assistant to pressure-test whether my test questions had
specific, checkable answers and to identify the exact phrase each answer
should contain. It helped identify direct facts such as the Kestrel Commons
wait time and the CS 210 workload; I checked each phrase against the source
documents and removed the spring-tide question because it belonged to the
`city_guides` corpus rather than my selected `campus_life` corpus.

**2.** I asked an AI assistant to help choose a chunking strategy for the
short campus posts and interpret the retrieval distances. It suggested
paragraph-aware chunks, and my measurements produced 91 chunks averaging 306
characters; I implemented the 450-character, zero-overlap strategy, kept
`TOP_K = 5`, and set the cutoff at `0.6` after comparing the in-scope and
out-of-scope distance groups.

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

The three answer passes were run with caching off. The raw evidence is in
[`results/run_2026-09-24_2202_before.md`](results/run_2026-09-24_2202_before.md),
produced by `run_eval.py::main` (with each answer produced by
`run_eval.py::run_once`).

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Sampled chunks are complete thoughts | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Named source contains the expected answer phrase | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |

### Verbatim output for criteria 1, 2, and 5

The following is the run-1 answer text and retrieved-source metadata copied from
`results/run_2026-09-24_2202_before.md`, produced by
`run_eval.py::main` → `run_eval.py::run_once` → `generate.py::answer_from_chunks`.
Each answer contains the expected phrase and names a source whose retrieved
chunk contains that phrase.

**Question:** How long are the wait times at Kestrel Commons between 12:15 and
1:00, and before 11:45?

**Sources retrieved:** `dining_halden_hall_followup.txt`,
`dining_kestrel_commons.txt`, `dining_kestrel_commons_followup.txt`,
`dining_the_ridgeway_cafe_followup.txt`, `transit_walking.txt`

```
Based on the documents, the wait times at Kestrel Commons are 20 to 25 minutes between 12:15 and 1:00, and under 5 minutes before 11:45 (*dining_kestrel_commons.txt*).
```

**Question:** What are the rules for taking a course pass/fail outside my
major?

**Sources retrieved:** `admin_add_drop_deadline.txt`,
`admin_declaring_a_major.txt`, `admin_graduation_requirements.txt`,
`admin_pass_fail_option.txt`, `course_stat_150.txt`

```
Any course outside your major can be taken pass/fail, and you can declare it as late as week eight. A pass requires a C- or better, with a maximum of two per year and eight across a degree.

Source: admin_pass_fail_option.txt
```

**Question:** How much do washing and drying cost in Aldridge Hall, and when
is the best time to do laundry?

**Sources retrieved:** `housing_aldridge_hall_laundry.txt`,
`housing_innisfree_hall.txt`, `housing_innisfree_hall_laundry.txt`,
`housing_morrow_house.txt`, `housing_old_brewhouse_laundry.txt`

```
In Aldridge Hall, a wash costs $1.75 and a dry costs $1.50. The best time to do laundry is Tuesday or Wednesday morning.

Source: housing_aldridge_hall_laundry.txt
```

**Question:** How often does the campus shuttle run on weekdays, and how often
does it run on weekends?

**Sources retrieved:** `course_cs_210_workload.txt`,
`course_econ_101_workload.txt`, `course_stat_150_workload.txt`,
`dining_verrill_street_grill.txt`, `transit_shuttle.txt`

```
Based on the provided documents, the campus shuttle runs a loop every 20 minutes on weekdays and every 40 minutes on weekends (transit_shuttle.txt).
```

**Question:** How many hours per week should I expect to spend outside class
for CS 210, and when is the workload heaviest?

**Sources retrieved:** `course_cs_210_workload.txt`,
`course_cs_340_workload.txt`, `course_econ_101_workload.txt`,
`course_engl_205_workload.txt`, `course_stat_150_workload.txt`

```
For CS 210, you should expect to spend 8 to 10 hours a week outside of class (course_cs_210_workload.txt). The workload is front-loaded, meaning the first month is heavier than the rest (course_cs_210_workload.txt).
```

### Verbatim output for criterion 3

Copied from the `The relevance gate on out-of-corpus questions` section of
`results/run_2026-09-24_2202_before.md`, produced by
`run_eval.py::check_out_of_scope`.

```
Produced by `run_eval.py::check_out_of_scope`, cutoff 0.6. Refused 5 of 5.

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.825 | refused |
| How do I change the oil in a diesel engine? | 0.934 | refused |
| Who won the 1994 World Cup? | 0.886 | refused |
| What is the recommended dosage of ibuprofen for a headache? | 0.844 | refused |
| How do I write a for loop in Rust? | 0.896 | refused |
```

### Verbatim output for criterion 4

This is the output of `app.py::cmd_chunks` for `app.py chunks -n 5`; each
chunk was produced by `chunker.py::split_documents`.

```
Chunk 1 | source: admin_add_drop_deadline.txt#0 | produced by: chunker.py::split_documents
On the add/drop deadline

You can add a course through the end of the second week. Dropping is a longer window — through the end of week six — but a drop after week two shows as a W on your transcript. Nothing anywhere on the registrar's site says this plainly, and students find out from each other.

Chunk 2 | source: course_biol_160_exams.txt#0 | produced by: chunker.py::split_documents
BIOL 160 Cell Biology — assessment

Four unit tests and a cumulative final. Not curved.

The unit tests come fast, roughly every three weeks; falling behind once is very hard to recover from.

Chunk 3 | source: course_math_220_exams.txt#0 | produced by: chunker.py::split_documents
MATH 220 Linear Algebra — assessment

Two midterms and a cumulative final. Curved to a b- median.

The problem sets are the course; the lectures make sense afterwards rather than during.

Chunk 4 | source: dining_the_ridgeway_cafe.txt#0 | produced by: chunker.py::split_documents
The Ridgeway Café

Second-year here. Wait times: 10 to 15 minutes at 12:30, none after 2:00. The thing worth going for is the only place on campus with real espresso. The thing to know is that seating is tight; about 40 seats for a building of 900.

Hours are 7:00am to 4:00pm weekdays only. Costs declining balance only, no meal swipes.

Chunk 5 | source: housing_morrow_house.txt#0 | produced by: chunker.py::split_documents
Morrow House — what it's actually like

Just finished a year in this building. Built 1954, partially renovated 2008. Rooms are singles and doubles, hall bathrooms.

The good: cheapest housing tier by about $900 a year, and the singles are real singles.

The bad: known damp problem on the ground floor; two rooms were taken offline in 2024.
```

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

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
