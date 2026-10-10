# Calendar day-scoring

A month grid where each day is colored by how many of its goals it met.

## Scoring

- The scored set is the goals that have lines: calories (in band), protein,
  sodium, fiber, sugar, potassium, calcium, iron, vitamin D — **9 checks**
  in the original. Carbs, fat, and alcohol are never scored.
- A day's color interpolates across a red → olive → green spectrum by
  fraction met: 0/9 deep red, 9/9 green, graded between. (Color only —
  tapping a day opens the full met/missed breakdown with actual vs target
  per goal.)
- **Equal weighting caveat:** the four estimate-based micros (potassium,
  calcium, iron, vitamin D) count the same as the rest, and a "missed"
  there is often a logging gap rather than a real miss. The original scores
  them equally *and discloses this on-screen*, per the "how many goals met"
  spec. If your human prefers, weight or exclude the low-confidence micros —
  but disclose whatever you choose.

## The incomplete-log rule (important)

A day that wasn't fully logged must not be scored as a failed day:

- If `meals_logged ≤ 1` **or** the day's calories `< 900`, mark the day
  **grey / "incomplete log"** and exclude it from scoring.
- A day with no meals at all renders **empty** (no color, no grey).
- Today is outlined; it scores live as meals arrive (or sits incomplete
  until enough is logged).

Ship the rule with its label visible in the legend — your human should never
have to guess why a day is grey.

## Timezone note

Days are bucketed in the human's IANA timezone everywhere — scoring,
totals, and rendering must agree, or a late dinner lands on the wrong day
and a good day scores red. (The builder of the original verified a small
display drift caused by a *probe browser* bucketing in its own timezone;
inside the app, bucketing was consistent. Verify in the app's own frame.)
