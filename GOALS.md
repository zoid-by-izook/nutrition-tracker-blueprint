# The goal system

Goals are **computed per day**, not stored — because the right goal for a
given date depends on the body state *on that date*. History must never be
rewritten: a past day keeps the goals that were true then.

## Core computation

For a date `d`:

1. **Weight(d)** = the latest WeightEntry on or before `d`. If none exists,
   use `calibration_weight_lb`. (Never borrow a *future* weight — that bug
   shipped once; see LESSONS.md.)
2. **Maintenance(d)** = `maintenance_kcal − (calibration_weight_lb − Weight(d)) × ~11`
   Maintenance falls as weight falls, roughly 11 kcal/day per pound. The
   constant is a heuristic; the whole estimate inherits the wearable's error
   bars anyway (see "Calibration honesty").
3. **Calorie band**:
   - ceiling = Maintenance(d) − deficit(pace)
   - floor = ceiling − 500, but **never below BMR** (estimate BMR from
     weight/height/age/sex, e.g. Mifflin–St Jeor). If floor would cross BMR,
     lift the ceiling instead and display the *effective* deficit — the app
     must never quietly promise a deficit it isn't actually showing.
4. Age(d) from birth_date → RDA bracket for the micro goals.

## The goals

| chart | kind | rule |
|-------|------|------|
| Calories | **range** | band above. Top = pace line; bottom = don't-live-below floor. Judge on weekly average, not single days. |
| Protein | minimum | ≈ 0.9 g per lb of **goal weight** (≈150 g for a 170 lb goal). Pegged to the *goal*, deliberately: pegging to current weight makes the muscle-protection floor shrink as the human succeeds. |
| Sodium | maximum | ≤ 2,300 mg/day (weekly average). Static — changes only on clinician input (e.g. a blood-pressure picture may justify the ~1,500 mg ideal). |
| Fiber | minimum | ≈ 11 g per 1,000 kcal of the calorie ceiling (≈33 g at a 3,000 ceiling). The textbook AI (14 g/1,000 kcal ⇒ ~42 g there) is the stretch value; stage to the lower rung first if the human's current average is far below. |
| Sugar | maximum | ≤ 50 g/day **total** sugar as a proxy cap, when added-vs-natural isn't tracked separately. Scales loosely with the calorie target. |
| Potassium | minimum | ≥ 3,400 mg/day (AI, adult men; 2,600 for women). **Safety-gated** (below). Tracker under-reads potassium — present as directional. |
| Calcium | minimum | ≥ 1,000 mg/day (RDA 19–50; 1,200 for women 51+ / men 71+). |
| Iron | minimum | ≥ 8 mg/day (men; 18 mg premenopausal women). Flag low-confidence until logging captures iron consistently. |
| Vitamin D | minimum | ≥ 15 mcg (600 IU)/day intake as a **placeholder** until a 25(OH)D lab exists; 20 mcg at age 71+. A lab result outranks the intake placeholder. |
| Carbs / Fat | none | Remainder values derived from calories − protein. Independent gram goals would contradict the other two. Show as ≈ ranges in text, no lines. |
| Alcohol | none | Tracked neutrally by design. No line, ever, unless the human asks. |

## What moves what (put this where your human can read it)

- **Moves with weight:** calorie band, fiber, sugar.
- **Moves only if the target moves:** protein.
- **Moves with age brackets only:** calcium (51/71), vitamin D (71).
- **Moves with labs/clinician:** vitamin D (lab), sodium (clinician).
- **Never moves on its own:** potassium, iron minimums.

## The safety screen (ask before setting the sensitive goals)

1. Any kidney disease or kidney concerns in labs? → gates the **potassium**
   goal ("more is better" is false there).
2. Any medications affecting blood pressure, potassium, or fluid balance
   (ACE inhibitors, ARBs, potassium-sparing diuretics, etc.)? → gates
   potassium and sodium.
3. Any recent labs — especially a vitamin D level? → a real 25(OH)D replaces
   the placeholder.
4. Also inventory *all* regular medications and supplements, not for the
   goals themselves but because surprises live there. In the original
   deployment this surfaced: an oral hair-loss medication that raises
   resting heart rate slightly (~+2.7 bpm in meta-analysis) — relevant to
   the human's heart-rate questions — and a biotin dose that can distort
   lab assays (thyroid, troponin, vitamin D can read falsely high/low;
   the vitamin D test the goals depend on is on that list). Tell your human
   to disclose supplements before any blood draw.

## Calibration honesty

- Wearable calorie-burn estimates carry ±15–25% error per session in
  validation studies, and tend to overestimate. The maintenance number is a
  starting anchor, nothing more.
- **The scale trend is the referee.** If 2–3 weeks of trend disagree with the
  band, adjust the band. Flat weight at a claimed deficit means intake is
  under-logged or maintenance is lower — both happen; partial logging days
  are the usual culprit.
- Present every goal as general information from public guidelines, not
  medical advice, and route anything clinical to the human's clinician.
