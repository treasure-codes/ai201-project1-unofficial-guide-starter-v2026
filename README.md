# The Unofficial Guide

The Unofficial Guide uses the campus_life corpus to answer student questions from real campus advice posts and administrative notes, with each answer tied back to the source file that said it.

---

# Unit 1

## What This Does

I picked the campus_life corpus because the documents are short student-written posts and administrative notes, which makes it a good fit for a retrieval system built around quick facts and direct answers. The system loads those documents, splits them into meaningful chunks, embeds each chunk, retrieves the closest matches to a question, and then answers using only those retrieved excerpts. It is designed for questions like whether the housing lottery is random, whether a meal plan can be changed after the first week, or how declaring a major actually works. This is the kind of “real student knowledge” search the project is built to support.

## Chunking Strategy

**Chunk size:** 320 characters
**Overlap:** 80 characters

I chose this chunk size for the campus_life corpus because most documents are one to three short paragraphs and the useful fact usually sits in a single sentence, not across multiple pages. A larger fixed window would bury the answer in surrounding text; a much smaller one would cut useful sentences into fragments. I kept 80 characters of overlap so a sentence split near the border still keeps enough context to be understandable without carrying the whole post in one chunk. I revised the initial generic strategy away from a raw 800-character window because that style is a poor fit for short posts, where a full thought can be only a few sentences long.

## Sample Chunks

**Chunk 1** — source: admin_housing_lottery.txt — produced by: chunker.py::split_documents

```
On the housing lottery

The housing lottery is not random in the way most people assume. Rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, and only tie-break randomly.
```

**Chunk 2** — source: admin_housing_lottery.txt — produced by: chunker.py::split_documents

```
That means a senior who took summer courses reliably beats a senior who didn't. Numbers come out the second week of March and selection runs over four evenings.
```

**Chunk 3** — source: admin_meal_plan_changes.txt — produced by: chunker.py::split_documents

```
On the meal plan changes

You can change your meal plan tier once, in the first ten days of the semester. After that it's locked. Downgrading refunds the difference to your student account; upgrading bills you immediately.
```

**Chunk 4** — source: admin_declaring_a_major.txt — produced by: chunker.py::split_documents

```
On the declaring a major

You declare at the end of your second semester, or later if you need to. There's no penalty for declaring late and no advantage to declaring early except that it assigns you a departmental adviser, who is generally more useful than the general one.
```

**Chunk 5** — source: admin_housing_lottery.txt — produced by: chunker.py::split_documents

```
Rising sophomores get a number drawn at random, but juniors and seniors are ordered by accumulated credit hours first, and only tie-break randomly. That means a senior who took summer courses reliably beats a senior who didn't.
```

## Sample Answer

**Question:** Is the housing lottery actually random?

**Answer:**

```
According to admin_housing_lottery.txt, the housing lottery is not random in the way most people assume. Rising sophomores get a random number, but juniors and seniors are ordered by accumulated credit hours first and only tie-break randomly, which means summer coursework can affect who gets priority.
```

**My relevance cutoff:** 0.6

I compared five in-corpus questions against five out-of-scope questions and looked for a gap between the best-distance groups. The idea was to place the cutoff between questions whose best retrieved chunk clearly matched the topic and questions that were unrelated. In this corpus, the in-corpus questions clustered in the lower, more relevant range while the off-topic questions sat much higher, so 0.6 was a reasonable cut point for the first pass.

| Question | In corpus? | Best distance |
|---|---|---|
| Is the housing lottery random? | Yes | 0.42 |
| Can I change my meal plan after ten days? | Yes | 0.46 |
| When do students declare a major? | Yes | 0.48 |
| Is there a penalty for late declaration? | Yes | 0.51 |
| Does summer coursework affect housing priority? | Yes | 0.55 |
| What is the capital of Mongolia? | No | 0.82 |
| How do I change oil in a diesel engine? | No | 0.89 |
| Who won the 1994 World Cup? | No | 0.93 |
| What is the recommended dosage of ibuprofen? | No | 0.87 |
| How do I write a for loop in Rust? | No | 0.91 |

## How I Used AI

**1.** I asked an AI assistant to help me pressure-test my chunking idea after reading the campus_life posts. The first draft suggested a simple fixed-size window without overlap, which was exactly the problem I had already identified: the system would cut through sentences and lose the fact the post was trying to communicate. I changed the approach to sentence-aware chunking with overlap so each chunk still read like a complete thought.

**2.** I asked for help tightening the acceptance-criteria language so each target was measurable instead of vague. The first version of some criteria said things like “retrieval works,” which is not testable. I rewrote them around counts and observable outcomes such as “4 of 5 questions,” “every answer names a source,” and “the gate refuses at least 4 of 5 unrelated questions,” so the standard is something another person could check without me explaining the intent.

<!-- ── Stretch features ─────────────────────────────────────────────────────
     No stretch features were added in this unit.
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
