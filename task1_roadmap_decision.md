# Task 1: Roadmap Decision

## Scoring framework

To rank candidates objectively rather than by instinct, I scored each item on five dimensions, 1–5 each:

- **Evidence Strength** — Is this backed by behavior data or just stated preference? How many independent sources report it?
- **Core Loop Impact** — Does it affect the coach-uploads → parent-opens → parent-shares → renewal cycle, or is it peripheral?
- **Reach** — How many accounts or users does this affect? One customer, or a pattern across clubs?
- **Urgency** — Is there a hard deadline, and what does delay cost?
- **Effort** (inverted, low effort scores high) — Roughly how large is the engineering lift?

**Total = Evidence Strength + Core Loop Impact + Reach + Urgency + (6 − Effort)**

## Scoring table

| Candidate | Evidence | Core Loop | Reach | Urgency | Effort (6−E) | Total |
|---|---|---|---|---|---|---|
| Roster tagging fix | 5 | 5 | 4 | 4 | 4 | **22** |
| Full livestreaming | 2 | 2 | 1 | 5 | 1 | **11** |
| Reel Editor expansion | 2 | 2 | 3 | 2 | 3 | **12** |
| Watermark bug | 3 | 2 | 1 | 1 | 5 | **12** |
| App icon change | 2 | 1 | 1 | 1 | 5 | **10** |

## Ranked priorities

### 1. Fix roster tagging accuracy (jersey number mismatches) — Score: 22

This is the top priority, and it isn't close. Three independent clubs across three different sports (Ridgeline soccer, Brightwater lacrosse, Cobblestone Little League) reported the same failure pattern this week: highlights landing in the wrong kid's reel when jersey numbers are visually similar. This isn't one unhappy customer, it's a repeating pattern across accounts, which is why it scores highest on both Evidence Strength and Reach.

The product health dashboard confirms the downstream cost: 30% of generated reels are never opened, and the analytics team's own note ties much of that to reels showing the wrong kid in the thumbnail. This bug sits directly in the core loop, coach uploads, parent opens, parent shares, and it's actively suppressing the top-of-funnel metric everything else depends on. The 92% share rate looks healthy, but it's calculated only on reels that get opened. It hides the real problem, which is why this scores a 5 on Core Loop Impact even though the headline metric looks fine.

### 2. Respond to Ridgeline with a smaller, faster commitment (not full livestreaming) — livestreaming itself scores 11

I would not put full livestreaming on the roadmap right now, but I would not go silent either. Livestreaming scores low on Evidence (2, one account, relayed secondhand through sales), Reach (1, no cross-club signal), and Effort (1, a large lift by the team's own description), but it scores high on Urgency (5) because the renewal window is real and closes in six weeks, and losing an $180k account matters.

That combination, high urgency but weak evidence, is exactly the pattern that should make a PM pause rather than build. The right move is to give Carla and the sales team something concrete before signups open, even if it isn't what she originally asked for: a firm timeline for when livestreaming will be evaluated, or a smaller step, like faster turnaround on highlight delivery during the game, that partially answers the "watch it live-ish" need without committing to full real-time streaming.

### 3. Everything else is a backlog item, not a roadmap item

Watermark bug (12) and app icon feedback (10) are real but low-stakes, both reporters flagged them as minor, and both score low on Reach and Core Loop Impact even though they're cheap to fix. They go in the backlog for a future cleanup pass, not this roadmap.

## What I would NOT build next, and why

**Full livestreaming for Ridgeline.** Score: 11, driven almost entirely by urgency, not by evidence that this is the right thing to build. The demand comes from one account and one internal team relaying pressure from that account. There's no usage data or cross-club signal suggesting this is a broader need, and the team's own read is that it's a significant engineering lift. Building a major new capability around a single customer's ask, under deadline pressure, is how roadmaps get hijacked by whoever is loudest that week. I'd rather protect the roadmap and give Ridgeline a real, smaller commitment than build the wrong big thing fast.

**Expanding the Reel Editor.** Score: 12. The quarterly survey shows 58% of parents want more editing control, but 90-day usage data shows only 6% ever open the editor, and fewer than 1% of those finish an edit. This is a classic gap between what people say they want and what they actually use, and it's why this scores low on Evidence Strength despite the strong-looking survey number. Building more into a feature nobody finishes isn't where the next unit of engineering time should go.

## Task 2: Spec — Roster Tagging Accuracy Fix

### Outcome

Reduce the rate at which highlight clips are matched to the wrong player, specifically cases where jersey numbers are visually similar (e.g., 14 vs. 4, 1 vs. 11), so that parents can trust the reel they receive actually shows their kid. Success looks like: fewer "wrong kid" support tickets, and a measurable recovery in the 30% of reels that currently go unopened, since the analytics team has tied a meaningful share of that number to thumbnail mismatches.

This is a trust fix, not a feature. The goal is not a completely new tagging system, it's closing the specific failure pattern the support tickets describe.

### In scope

- Add a confidence score to each jersey-number-to-roster match the system makes.
- Use additional available signals already captured by the system (jersey color/team, approximate player position on field, timestamp/sequence continuity) to help disambiguate numbers that are visually similar or easily confused (digit swaps, single-digit vs. double-digit).
- When a match falls below the confidence threshold, hold that specific clip out of the automated reel and flag it for coach confirmation before it's included in a kid's reel or the weekly recap.
- Build a lightweight coach-facing confirmation step for flagged clips only, not a full manual review of every clip. Critically, this prompt shows the coach more than just the system's guess: the clip itself alongside the full team roster, so the coach is disambiguating with real information, not just eyeballing the same blurry frame the system already misread. One support ticket this week came from a coach, not a parent, which tells us coaches can make the same mistake the system does if given nothing more to go on.
- Instrument and report the error rate before and after, using support tickets and a manual audit sample, so we can tell if this actually worked.

### Out of scope

- A full retrain or rebuild of the underlying jersey-number recognition model from scratch.
- Real-time tagging accuracy for any future livestreaming feature, this spec only covers the existing upload/auto-record and highlight generation flow.
- Manual review of every clip generated, only clips below the confidence threshold get flagged.
- Reprocessing or re-sending previously generated reels that may have contained errors.
- Redesigning the coach upload or reel-editing interface beyond the single confirmation step described above.
- Any customer-facing follow-up for reels that already went out with a wrong-kid error before this fix ships. That's a support/CS communication question, not a technical scope item for this spec, but it shouldn't be silently dropped either.

### Where a human stays in the loop

Any clip where the system's confidence in the jersey number match falls below the threshold is not auto-included. The coach who uploaded the footage (or manages the camera) sees a short confirmation prompt for that specific clip before the reel or recap goes out. The system does not guess and send when it isn't confident, it asks. High-confidence matches continue to flow automatically, unchanged. We don't have an exact baseline accuracy rate from the materials available, but the support tickets describe a specific, visually-triggered failure pattern (similar or transposed jersey numbers), not a systemic breakdown, so the intent is to add a check at the specific point where the system is likely to be wrong, without adding review friction to matches that were never in question.

### Acceptance criteria

- Target: on a test set of footage containing visually similar jersey numbers (built from the patterns described in the support tickets: 14/4, 1/11, and other single-vs-double-digit or transposed-digit pairs), the system correctly withholds at least 95% of low-confidence matches from auto-send, rather than sending them with a wrong name attached.
- Target: a coach can review and confirm or correct a flagged clip in under 15 seconds, with no more than a couple of taps, so the fix doesn't create a new burden that discourages upload.
- Target: following a rollout to a test group of clubs, "wrong kid" support tickets from that group drop by 50% or more over a two-week comparison window against the pre-fix baseline, which this rollout will also help establish.
- Target: the reel-open rate for the test group moves upward from its current baseline, consistent with the analytics team's hypothesis that mismatched thumbnails are suppressing opens.

### Top two failure modes after shipping, and what catches each

**1. The confidence threshold is miscalibrated.** If it's set too loose, wrong-kid errors still slip through unflagged. If it's set too strict, too many correct matches get flagged for coach confirmation, creating review fatigue.
What catches it: monitor two numbers weekly after launch, the percentage of clips going to the review queue, and the percentage of flagged clips coaches actually correct versus just approve outright. If the review queue is too large, or the correction rate is near zero, the threshold needs to be retuned.

**2. Coach confirmation becomes a rubber stamp instead of a real check.** This is a human failure mode, not an algorithm one, and it's grounded directly in the materials: one of this week's support tickets was filed by a coach, not a parent, describing the same kind of number mix-up the system makes. That means the person we're asking to catch these errors can make the same visual mistake, especially if the confirmation prompt doesn't give them more to work with than the same blurry frame the system already misread, or if confirmations become routine enough that coaches stop looking closely.
What catches it: track the correction rate on flagged clips over time (not just the queue size). A correction rate that drops toward zero and stays there, even as the underlying error rate presumably hasn't changed, is a signal that confirmations have become a formality rather than a real review, and the prompt or the roster-comparison view may need to be made clearer or more prominent.

## Task 3: Prototype

Link: https://katherinez323.github.io/sidelinereel-prototype/

## Task 4: AI Workflow Note

For Task 1 and Task 2, I did most of the actual thinking myself, and used AI more to move faster once I already had a direction, for example, in Task 1, once I'd defined the scoring dimensions, I had AI help fill in the table and total up the scores so I could see the ranking faster. But deciding what the dimensions should be and how to score each item was still on me, AI just sped up that part. The actual ranking and the call not to build livestreaming were mine. For Task 3, I used AI directly to write the prototype code, but every detail in the flow, like not pre-selecting the system's guess, was something I decided on first and then had it build.

One place AI got something wrong that stuck with me: while drafting the spec, it slipped in a line saying "90%+ of existing matches already work," and an acceptance criterion that hard-coded a 95% target. Both sounded reasonable, but neither number actually exists anywhere in the source materials, it made them up. I caught it on review and fixed both, either dropping the fabricated baseline or clearly labeling the number as a target I was setting rather than something backed by data. It was a good reminder that AI output can sound confident without actually being grounded, and that's on me to catch before it ships.
