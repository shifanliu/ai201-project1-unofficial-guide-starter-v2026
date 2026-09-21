# The Unofficial Guide

**[shifanliu]** — corpus: `city_guides`

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

<!-- Three or four sentences. Which corpus you picked, and the kinds of
     questions your system answers. Write it for someone who has never seen
     this repo.

     Milestone 5. -->

This is a retrieval-augmented question-answering system built on the `city_guides` corpus: fourteen long-form travel guides covering nine towns and villages in a region, plus five documents that cut across all of them (eating, walking, regional transport, seasons, and accessibility). Ask it
a specific question, for example, how long a drive takes, where to find cheaper food, and it finds
the relevant section of the right guide, and answers using only what's written there, naming the source file. If you ask something the guides don't cover, it says so honestly instead of guessing.

## Chunking Strategy

**Chunk size:** No fixed character size — one chunk per `##` heading (a full section)
**Overlap:** None in the traditional sense; instead, the document's top-level title (e.g. `# Corry Vale`) is prepended to every chunk

city_guides documents are long, structured travel guides (14 documents, ~2,068
characters each on average) organized by labelled `##` sections — Getting there, Getting around, Eat and drink, and so on. The useful information for
a given topic lives entirely within its own section, not scattered across sentences, so I replaced the starter's fixed 800-character window with a
split on `##` headings: each chunk is one complete section.

I set no upper size cap, since a section stays a single coherent topic even when it runs long — capping it would just reintroduce the mid-thought cuts
I'm trying to avoid. Instead of overlap in the usual sense (repeating trailing characters from the previous chunk), I prepend the document's
top-level title to every chunk, so a single retrieved chunk still carries which town it's about even without the surrounding document.

Result: 94 chunks (up from the starter's 51), averaging 322 characters (down from 650), ranging from 174 to 762 characters — no more 24-character
fragments like the starter produced.

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
**Chunk 1** — source: `guide_accessibility.md#0` — produced by: `chunker.py::split_documents`

```
# Getting around the region with limited mobility

An honest assessment rather than a promotional one. Some of these places are
difficult and it is better to know in advance.
```

This one is weaker than the rest — it's an intro/disclaimer with no concrete
information, so on its own it can't answer a specific question. That's a
property of this document (it opens with a framing paragraph before any
real content), not a flaw in the splitting logic.

**Chunk 2** — source: `guide_corry_vale.md#5` — produced by: `chunker.py::split_documents`

```
# Corry Vale

## Where to stay

Perhaps thirty beds in the entire valley, spread across two pubs and a
handful of farmhouse rooms. In summer these are booked months ahead.
Camping is permitted on two marked fields and nowhere else.
```

Stands on its own — answers "where can I stay in Corry Vale" completely.

**Chunk 3** — source: `guide_givens_mill.md#2` — produced by: `chunker.py::split_documents`

```
# Givens Mill

## Getting around

Everything is on one street along the river. The mill is at one end and the
church at the other, eight minutes apart. The riverside path continues in
both directions for as far as you want to walk.
```

Stands on its own — answers "how do I get around Givens Mill" completely.

**Chunk 4** — source: `guide_kestrelford.md#4` — produced by: `chunker.py::split_documents`

```
# Kestrelford

## What to see

The market square on a Saturday morning is the main event and has run
continuously since the 1400s. The parish church has a 13th-century tower
you can climb for £2. The old trackbed walk runs six miles to the next
village along an easy gradient and is the best half-day here.
```

Stands on its own — answers "what should I see in Kestrelford" completely,
including the specific figures (1400s, £2, six miles) intact.

**Chunk 5** — source: `guide_pellew_sands.md#6` — produced by: `chunker.py::split_documents`

```
# Pellew Sands

## When to go

June and September for the beach without the crowds. July and August are
busy and the town is at its most itself, for better and worse. Winter is
bleak, largely closed, and has a following among people who like that sort
of thing.
```

Stands on its own — answers "when should I visit Pellew Sands" completely.

**Summary:** 4 of 5 sampled chunks read as a complete thought, matching the
target set in `criteria.md` #4. The one exception (Chunk 1) is an
intro-paragraph chunk with no factual content — a document-structure issue
rather than a chunking-logic one.

## Sample Answer

<!-- One complete question and answer, pasted as text, with the source line
     visible. Milestone 4. -->

**Question:** How long does it take to drive from Brightwater to Corry Vale?

**Answer:**

```
Driving from Brightwater to Corry Vale takes 35 minutes on a good road as far as the valley mouth and then 20 more on a poor one (guide_corry_vale.md).

Sources retrieved: guide_brightwater.md, guide_corry_vale.md, guide_marchwood.md, guide_thornby_wells.md, guide_walking.md
```

**My relevance cutoff:** 0.6 (kept the starter's default)

<!-- The number you set in config.py, and how you got there.

     You ran five questions your corpus covers and the five in OUT_OF_SCOPE
     that it clearly doesn't, and wrote down the best distance for each. What
     did those two groups look like? Where was the gap? Put the actual numbers
     here — the table below wants all ten rows.

     Milestone 4. -->

Ran all 5 in-corpus questions and all 5 OUT_OF_SCOPE questions. The best
distance for in-corpus questions ranged from 0.196 to 0.411; the best
distance for out-of-scope questions ranged from 0.803 to 0.975. That's a
gap of nearly 0.4 with no overlap at all — the default 0.6 sits cleanly in
the middle and correctly separates both groups completely (5/5 each).

| Question | In corpus? | Best distance |
|---|---|---|
| Which pub in Corry Vale serves food every day of the week? | Yes | 0.196 |
| How long does it take to drive from Brightwater to Corry Vale? | Yes | 0.251 |
| Where in Brightwater can you find cheaper food than the riverside strip? | Yes | 0.274 |
| What months should you visit Corry Vale? | Yes | 0.342 |
| How far in advance do you need to book Sunday lunch at Thornby Wells? | Yes | 0.411 |
| What is the capital of Mongolia? | No | 0.803 |
| What is the recommended dosage of ibuprofen for a headache? | No | 0.835 |
| How do I write a for loop in Rust? | No | 0.836 |
| How do I change the oil in a diesel engine? | No | 0.888 |
| Who won the 1994 World Cup? | No | 0.975 |

## How I Used AI

<!-- Two specific moments. For each: what you asked for, what came back, and
     what you changed about it.

     "I asked Claude to write the chunking function from my notes. It ignored
     the overlap, so I added that myself" is the level of detail we're after.
     "I used AI to help me code" is not.

     Milestone 5. -->

**1.** I asked Claude to write the chunking function, giving it precise constraints upfront rather than a vague request: split on `##` headings so
each chunk is one complete section, no upper size cap, and prepend the document's top-level title to every chunk instead of using traditional
character-overlap. It returned a `split_documents` function matching those constraints exactly, including handling the edge case of an intro paragraph
before the first heading. I ran it, checked the actual output (94 chunks, 174–762 characters, no more 24-character fragments like the starter's
version), and confirmed the sample chunks read as complete thoughts before keeping the function as given.

**2.** I pasted my five acceptance criteria and asked Claude to say exactly how it would test each one using only the sentence, with no suggestions. It
flagged that my chunk-size criterion listed three question types as *examples* rather than naming which specific questions counted, so someone
grading it could reasonably disagree on which 3 of my 5 questions the "2 of 3" target applied to. I rewrote it to say "the 3 of my 5 test questions
whose expected answer is a specific number, date, or duration" instead of just listing examples, so the set of questions is fixed rather than open to
interpretation.

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

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

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
