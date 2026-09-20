# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
The five questions cover different parts of the campus-life corpus, but each
answer is stated in a short, focused document. I chose 4 of 5 because one
question could still be missed by retrieval even when the other four are easy
single-document matches.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
All five questions are answerable from named files in the corpus, and the
retrieval results already carry each file's source metadata into the answer
prompt. That makes 5 of 5 achievable; a missing source would indicate a
pipeline or generation failure rather than an ambiguity in the questions.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:**
The out-of-scope questions are about unrelated facts such as world history,
medicine, and programming, so they should normally be rejected. I allowed one
of five failures because an unrelated question could still produce an
accidentally close embedding match.

---

## 4. Something about your chunks

When I inspect five sampled chunks, at least 4 of 5 read as complete thoughts
or paragraphs, with no sentence cut in half at either edge.

**Why this target:**
Most `campus_life` documents are short posts of one to three paragraphs, so a
chunk should usually preserve a whole thought. I chose 4 of 5 because one
boundary case may be unusually long or awkward without showing that the
overall chunk size is wrong.


---

## 5. Your choice

For at least 4 of my 5 test questions, the source document named in the answer
contains the expected answer phrase listed for that question in `questions.py`.

**Why this target:**
Having a source name is not enough if that document does not support the claim.
I chose 4 of 5 because four questions have one especially direct source, while
one answer could be paraphrased or cite a nearby document even when the
retrieved information is mostly correct.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
