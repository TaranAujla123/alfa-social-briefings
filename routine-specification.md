# ALFA Weekly Briefing Routine — Specification

**Purpose:** Every Monday 7:00 AM local, Claude picks 3 quotes for the week, rotates pillars, and drops a ready-to-paste briefing in `briefings/`. You open Canva + Meta Business Suite and ship.

**Runs:** Every Monday 7:00 AM (local timezone)
**Creates:** `MARKETING/IG Posts/briefings/week-YYYY-MM-DD.md`
**Reads:** `daily-quotes-bank.md` + `quotes-used.json`
**Writes:** Updates `quotes-used.json` to track what's been used

---

## Files the routine touches

| File | Purpose |
|---|---|
| `daily-quotes-bank.md` | 90-quote source library (never modified by routine) |
| `quotes-used.json` | State tracker — which quote IDs have been used |
| `briefings/week-YYYY-MM-DD.md` | Output doc for the week (one per run) |

---

## The weekly pillar rotation (locked)

Three posts per week, always in this order:

| Day | Pillar | Quote # range | Canva variant |
|---|---|---|---|
| **Monday** | Pattern (mechanism) | #1–22 | Variant 1 or 2 |
| **Wednesday** | Stoic layer | #49–70 | Variant 1 or 2 |
| **Saturday** | Brother **OR** Founder Proof (alternates biweekly) | #71–90 or #23–32 | Variant 3 (Brother) or Variant 5 (Founder Proof) |

**Saturday rotation:** alternates every week.
- Odd weeks: Brother pillar
- Even weeks: Founder Proof
- Tracked in `quotes-used.json` under `saturday_alternate`

---

## What the briefing doc contains

For each of the 3 posts:
1. **Day + post time** (Mon 7am, Wed 7am, Sat 9am)
2. **Pillar + quote ID**
3. **Canva variant to use** (1, 2, 3, 4, or 5)
4. **Headline** with gold accent word clearly marked
5. **Subhead** (if Variant 2)
6. **Caption** — ready to paste into IG caption field
7. **Hashtags** — ready to paste (already baked into each quote in the bank)

Plus a "How to ship this week" footer with the step-by-step.

---

## How to use the briefing (every Monday)

1. Open the week's briefing in `MARKETING/IG Posts/briefings/week-YYYY-MM-DD.md`
2. Open Canva template
3. For each of the 3 posts:
   - Duplicate template page
   - Paste headline → mark gold word → update DAY number
   - Export PNG
4. Open Meta Business Suite → Planner
5. Upload 3 PNGs → paste captions + hashtags → schedule for Mon 7am, Wed 7am, Sat 9am
6. Close laptop

**Total time:** ~20 minutes, once per week.

---

## How to manually trigger the routine (outside the schedule)

If you want to generate a briefing right now without waiting for Monday:

```bash
# From any terminal
claude-code --trigger "weekly-briefing"
```

Or just ask Claude in chat: "Run the weekly briefing routine."

The routine is deterministic — if you run it twice in the same week, it regenerates the same week's briefing (doesn't pick new quotes).

---

## How to modify the routine

### Change the rotation
Edit `MARKETING/IG Posts/routine-specification.md` → "Weekly pillar rotation" section. The next routine run reads the current spec.

### Force a specific quote
Edit `briefings/week-YYYY-MM-DD.md` after generation and paste manually — the routine won't re-pick for the same week once the briefing exists.

### Reset used-quote tracking
Delete or edit `quotes-used.json` → next routine run will treat all quotes as unused again.

### Pause the routine
Tell Claude: "Pause the weekly briefing routine." (uses the `schedule` skill to delete the trigger — doesn't delete the quote bank or state).

### Resume
"Resume the weekly briefing routine."

---

## Troubleshooting

### "The briefing didn't generate on Monday"
- Check: was Claude Code running / your machine on at 7am Monday?
- Workaround: manually trigger ("Run the weekly briefing routine")
- Permanent fix: move the scheduled agent to a cloud runner (future upgrade)

### "The routine picked a quote I already used manually"
- The routine only knows what it put in `quotes-used.json`.
- If you use a quote manually from the bank, append its ID to `used_quote_ids` in the JSON so the routine skips it next time.

### "Same pillar twice in one week"
- Shouldn't happen — pillars are locked to days. If it does, the rotation logic got corrupted. Reset `quotes-used.json` and re-run.

### "I want 5 posts/week instead of 3"
- Edit this spec: add Tue + Thu rows to the rotation
- Re-run — the routine will pick 5 quotes next Monday

---

## When to upgrade from Tier A → Tier B or C

Review these signals after 30 days of running:

**Upgrade to Tier B (auto-generate PNGs) if:**
- Canva step is consistently the bottleneck
- You've found 1-2 template variants that always work
- You're OK switching PNG generation to `post-pipeline` (one-time style match needed)

**Upgrade to Tier C (full auto-post) if:**
- You're confident in the voice (no tweaks needed per post)
- You have time to set up Meta Developer App (~1 hr)
- You want posts going out while traveling / asleep

Ask Claude: "Upgrade my weekly briefing routine to Tier B" (or Tier C) and I'll build the next layer.

---

## The routine prompt (what Claude executes on trigger)

This is the prompt the scheduled agent runs. Do not edit unless you know what you're doing — the routine logic depends on it.

```
You are running the ALFA weekly briefing routine.

1. Read C:/Users/Owner/ALFA REBUILD/MARKETING/IG Posts/quotes-used.json
2. Read C:/Users/Owner/ALFA REBUILD/MARKETING/IG Posts/daily-quotes-bank.md
3. Compute this week's Monday date (YYYY-MM-DD).
4. If briefings/week-YYYY-MM-DD.md already exists, stop (don't regenerate).
5. Pick 3 unused quotes:
   - Monday: Pattern pillar (quote IDs 1-22) not in used_quote_ids
   - Wednesday: Stoic pillar (49-70) not in used_quote_ids
   - Saturday: use saturday_alternate field.
     - If "brother" → pick from 71-90 not in used_quote_ids
     - If "founder" → pick from 23-32 not in used_quote_ids
     - Then flip saturday_alternate for next week.
6. Write briefings/week-YYYY-MM-DD.md in the standard briefing format (see example below).
7. Update quotes-used.json:
   - Append the 3 quote IDs to used_quote_ids
   - Increment week_count
   - Update last_run to today's ISO date
   - Push to history: { week: "YYYY-MM-DD", quotes: [id1, id2, id3] }
   - Flip saturday_alternate if needed
8. Report: "Weekly briefing ready at briefings/week-YYYY-MM-DD.md. Monday post: [headline]. Wednesday: [headline]. Saturday: [headline]."

Standard briefing format:
# ALFA Weekly Briefing — Week of [Monday date]

## Monday 7:00 AM — Pattern pillar
**Quote #[N]** | Canva variant [V]
**Headline:** [headline with **GOLD WORD** bolded]
**Subhead:** [if variant 2]

**Caption:**
[full caption]

**Hashtags:**
`#hashtags`

---
(repeat for Wednesday, Saturday)

---

## Ship checklist
- [ ] Open Canva template
- [ ] Duplicate page 3 times, paste each headline
- [ ] Export 3 PNGs
- [ ] Upload to Meta Business Suite
- [ ] Paste captions + hashtags
- [ ] Schedule: Mon 7am, Wed 7am, Sat 9am
- [ ] Done for the week.
```

---

*Routine spec v1.0 — April 2026. Works with Canva + Meta Business Suite (Tier A).*
