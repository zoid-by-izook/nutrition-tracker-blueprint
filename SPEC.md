# Spec — data model and action API

## Entities

### Meal

| field      | type    | notes |
|------------|---------|-------|
| id         | integer | app-assigned |
| summary    | string  | human-readable, e.g. "Chicken breakfast bowl (home)" |
| meal_type  | enum    | `breakfast` \| `lunch` \| `dinner` \| `snack`. Drinks log as `snack` unless the app grows a drink type. |
| timestamp  | string  | offset-aware ISO 8601, e.g. `2026-10-09T10:16:57-07:00` |
| timezone   | string  | IANA name, e.g. `America/Los_Angeles` |
| items      | array   | see Item |

### Item

| field        | unit | notes |
|--------------|------|-------|
| name         | —    | include brand/flavor when known |
| portion      | —    | free text, e.g. `"1 can, drained"`, `"2 tbsp"` |
| calories     | kcal | |
| protein_g, carbs_g, fat_g, fiber_g, sugar_g | g | |
| sodium_mg    | mg   | |
| potassium_mg | mg   | optional; omit when unknown rather than inventing |
| calcium_mg, iron_mg | mg | optional |
| vitamin_d_mcg | mcg | optional |
| alcohol_g    | g    | optional; 14 g = 1 US standard drink |

Micros are *capture-when-known*. Absent ≠ zero in meaning, even if the
storage treats them as 0 — the UI and any day-scoring must caveat this.

### Profile (single row)

| field | notes |
|-------|-------|
| height_in | inches (the original runs imperial for its human; pick your human's units and be consistent) |
| birth_date | used only to compute the current RDA age bracket |
| sex | RDA brackets differ |
| maintenance_kcal | **calibrated**, not formula-derived: the daily burn at the calibration weight, from the human's wearable trend (see GOALS.md) |
| calibration_weight_lb | the weight at which `maintenance_kcal` was measured |
| goal_weight_lb | target weight; protein pegs to this |
| pace | `gentle` \| `standard` \| `aggressive` → deficit 250 / 500 / 750–1000 kcal |

### WeightEntry

| field | notes |
|-------|-------|
| date | local date |
| weight_lb | one entry per weigh-in; multiple per day is fine |

Weights can arrive from a health-data sync (if your human grants their
assistant standing access to e.g. Health Connect) or manual entry. The
original's assistant logs new weigh-ins into the app whenever it sees them —
the app itself has no health-platform channel.

## Actions (the app's published API)

- `logMeal` — args: summary, meal_type, timestamp, timezone, items[].
  Returns the new meal id.
- `updateMeal` — full replacement semantics: id + the same fields as
  `logMeal`. Used to correct an entry when better data arrives (e.g. a real
  label replaces an estimate).
- `deleteMeal` — by id.
- `getMeals` — meals in a date range.
- `getDailyTotals` — args: `start`, `end`, `timezone`. Returns per-day sums
  of every tracked nutrient + `meals_logged` count (the calendar's
  completeness heuristic needs it).
- `getProfile` — profile + **the goal set computed for today** (see GOALS.md).
- `logWeight` — args: `date`, `weight_lb`.

## Views

- **Trends** — per-nutrient charts over time with that day's goal line(s)
  drawn in, and a short subtitle under each chart explaining the line
  ("why it sits where it sits" — full rationale lives in Body & goals).
- **Daily intake** — kept deliberately slim: calories against the goal band,
  plus fat. Protein/carbs live in a separate **Macro movement** view. (This
  slimming was a user-requested design correction after goal lines made the
  main view cluttered.)
- **Calendar** — see `CALENDAR.md`.
- **Body & goals** — profile editor, pace selector, weight log, and the full
  goal rationale text.
