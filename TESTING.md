# OpenSpectrum — Manual Testing Guide

A step-by-step walkthrough of the features built across Phases 1–7. Work through
these in order on a fresh install (or after clearing IndexedDB) for the cleanest
results. Each section can also be run independently if you just want to spot-check
one area.

---

## Before you start

```bash
cd openspectrum-app/pwa
npm run dev          # starts dev server at http://localhost:5173
```

Open the app in Chrome or Edge. Open DevTools → Application → IndexedDB →
`openspectrum-db` if you want to inspect raw data during testing.

---

## Phase 1 — Data Foundation

These checks confirm the database schema and seed data are correct. Most are
done via DevTools rather than the UI (no Phase 1 UI was shipped).

### 1.1 Seed data present on first load

1. Open the app for the first time (fresh IndexedDB).
2. DevTools → Application → IndexedDB → `openspectrum-db`.
3. Open the `assessment_scales` table.
4. Confirm 5 rows exist:
   - **Mood** (numeric, 1–5)
   - **Sleep Quality** (numeric, 1–5)
   - **Behavior Severity** (numeric, 1–5)
   - **Food Reaction** (categorical: adverse / neutral / positive)
   - **Medication Effect** (categorical: worse / no change / better)
5. Open the `children` table — confirm 1 row ("Test Child") is present.

### 1.2 Observation schema fields

1. Log any quick-tap tag (see Phase 2 steps below).
2. Open the `observations` table in DevTools.
3. Confirm the new v2 fields are present on the row:
   - `event_time_precision` = `"exact"`
   - `data_layer` = `"raw"`
   - `entry_type` = `"quick_tap"`
   - `event_end_at`, `duration_minutes`, `severity`, `confidence` = `null`

---

## Phase 2 — Capture

### 2.1 Quick-tap logging

1. Go to **Home**.
2. Tap any tag (e.g. "Meltdown" in the behavior row).
3. A toast appears at the bottom: `"Meltdown logged"`.
4. The toast disappears after ~3 seconds.

### 2.2 Assessment prompt after logging

1. Tap a **behavior** tag (behavior scales are seeded).
2. The toast includes a **"Rate ›"** action button.
3. Tap "Rate ›" — the AssessmentSheet slides up.
4. Choose a severity score (1–5) and tap Save.
5. Go to Timeline → find today's entry → confirm the score badge appears
   on the card (e.g. `"Behavior Severity 3/5"`).

### 2.3 Time offset — backdating an entry

1. On Home, find the **TimeOffsetPicker** (the time row below the mic button).
2. Tap "1h ago" (or any offset).
3. Log a tag.
4. Go to Timeline → the entry's timestamp should reflect the offset, not "now".
5. Tap the entry's time — it should show `~` prefix for approximate precision.

### 2.4 "Earlier today" / date-only precision

1. On the TimeOffsetPicker tap **"Earlier"** (date-only option).
2. Log a tag.
3. On the Timeline card the time display shows **"Earlier"** instead of a clock time.

### 2.5 Creating a custom tag

1. On Home, tap **"+ New tag"** at the bottom of the tag grid.
2. The CreateTagSheet slides up.
3. Enter a name, pick a category, tap Save.
4. The new tag appears in the grid under the correct category.
5. Tap it — it logs successfully with a toast.

### 2.6 Tag usage ordering

1. Tap the same tag several times across a few sessions.
2. On Home, that tag should drift towards the top of its category group
   (most-used tags sort to the front).

### 2.7 Voice capture

1. Tap the **mic button** — the button should turn red and show a timer.
2. Speak a short observation: *"Had a really difficult morning, threw a toy,
   cried for about 10 minutes."*
3. Tap the mic button again to stop.
4. The VoiceReviewSheet slides up with a transcript and suggested observations.
5. Confirm the observations you want, tap **Confirm**.
6. A toast: `"2 observations logged"` (or however many were detected).
7. Go to Timeline — the entries appear with the AI summary toggle.

---

## Phase 3 — Timeline & Review

### 3.1 Day view — basic list

1. Go to **Timeline**.
2. Today's entries appear as cards, newest at the top.
3. Each card shows: time, category colour bar, category label, tag name / title.

### 3.2 Delete with undo

1. Tap **✕** on any card.
2. The card disappears and a toast appears: `"Entry deleted"` with an **Undo** action.
3. Tap **Undo** — the card reappears.
4. Tap ✕ again, do not undo — the entry is permanently gone after the toast expires.

### 3.3 Category filter row

1. The coloured pill row below the date navigator shows categories present that day.
2. Tap a category pill (e.g. "behavior 3") — the list filters to only behavior entries.
3. The pill turns filled/highlighted.
4. Tap it again — all entries return.
5. Tap two different pills — both filters apply simultaneously.

### 3.4 Week view

1. Tap the **"Week"** toggle.
2. Seven day sections appear (Mon–Sun or the surrounding 7 days).
3. Each section shows compact `TimelineRow` entries.
4. Tap any row — the view switches back to Day view at that date.

### 3.5 Date navigation

1. In Day view, tap **"‹"** (prev) — moves back one day.
2. Tap **"›"** (next) — moves forward one day.
3. Tap **"Today"** — jumps back to today.
4. In Week view, ‹ / › move by 7 days.

### 3.6 Search

1. Type in the search bar.
2. The timeline filters in real time to entries whose title, notes, or tag names
   contain the query.
3. Clear the query — all entries return.

### 3.7 Reflection widget

1. At the bottom of the day view, a **Reflection** row appears.
2. Tap one of the three mood icons (Better / Typical / Difficult).
3. The icon fills to show selection.
4. Navigate to another day and back — the rating persists.

### 3.8 Assessment badges on cards

1. Find (or create) an observation you rated in step 2.2.
2. The card shows a small pill badge with the scale name and score.

---

## Phase 4 — Focus Areas

### 4.1 Creating a focus area

1. Go to **Focus Areas** (bottom nav).
2. Tap **"+ New Focus Area"**.
3. Fill in:
   - **Title**: "Difficult mornings"
   - **Description**: "Are meltdowns happening more before school?"
   - **Categories**: tap "behavior" and "emotion" to select both
   - **Questions**: "What time do meltdowns peak?" → tap "+ Add question" →
     "Is there a food trigger?"
4. Tap **Save**.
5. The new area appears in the list with status "Active".

### 4.2 Editing a focus area

1. Tap the focus area card.
2. The FocusAreaSheet slides up pre-filled.
3. Change the status to **"Paused"**.
4. Tap Save — the card now shows "Paused" and moves down the list.
5. Edit again, change back to "Active".

### 4.3 Focus filter on Timeline

1. Go to **Timeline**.
2. If the focus area from 4.1 is active, its chip appears below the Day/Week
   toggle (e.g. "🎯 Difficult mornings").
3. Tap the chip — the timeline filters to only behavior and emotion entries.
4. The chip shows `✕` when active.
5. Tap it again — all entries return.

### 4.4 Navigate to Timeline from Focus Areas

1. On the Focus Areas page, tap the **"View in Timeline"** button (or equivalent)
   on a focus area card.
2. Timeline opens with that focus area's filter pre-applied.

---

## Phase 5 — Intelligence (Rules Engine + Insights)

### 5.1 PromptsCard on Home — time-of-day nudges

1. Open Home at different times of day to see different contextual prompts:
   - Morning (~7–9am): prompt to log the morning routine
   - Evening (~6–9pm): prompt to reflect on the day
   - Late night: gentle wind-down reminder
2. The PromptsCard appears below the ChildSelector when prompts are available.
3. Tap **✕** on the card — it dismisses for the session.

### 5.2 PromptsCard — real topCategory (Phase 7)

1. Log several entries in one category over the past few days (e.g. 5 "behavior"
   entries across today and yesterday).
2. Open Home.
3. The prompt text should reference **behavior** by name (e.g. "You've logged a
   lot of behavior events lately…") rather than generic copy.

### 5.3 PromptsCard — meltdown follow-up (Phase 7)

1. Log 2 or more **behavior** entries today or yesterday.
2. Close and reopen Home.
3. A follow-up prompt should appear: something like "You've had some difficult
   moments recently — how are you doing?"

### 5.4 Insights — range selector

1. Go to **Insights**.
2. Tap **7 Days**, **14 Days**, **30 Days** — the summary cards and charts
   re-load for each range.

### 5.5 Insights — summary cards

With some data logged:
- **Total Logs** shows the correct count.
- **Per Day** shows the average.
- **Categories** shows how many distinct categories appear.
- If a prior period exists, a `▲ X% vs prior` badge appears on Total Logs.

### 5.6 Insights — category breakdown with trends

1. Log 5 behavior entries this week and 2 last week.
2. In Insights (7-day view), the behavior row should show `↑` with a percentage
   indicating the increase vs the prior 7 days.

### 5.7 Insights — patterns & hypotheses

With a week or more of data:
1. Scroll to **"Patterns & Insights"**.
2. Look for rules-engine hypotheses such as:
   - Peak time of day for a category
   - Day-of-week concentration
   - Assessment score observations
   - Behavior spike detection

### 5.8 Insights — hourly and day-of-week charts

1. Scroll to **Time of Day** — a bar chart shows which hours have most entries.
2. Scroll to **Day of Week** — a bar chart shows which days are busiest.
3. The peak hour bar is highlighted in the primary colour.

### 5.9 Insights — assessment score averages

1. Rate several observations using the AssessmentSheet (step 2.2).
2. In Insights, scroll to **Assessment Scores**.
3. Numeric scales show an average score with a progress bar.
4. Categorical scales show a distribution of values.

---

## Phase 6 — Export

### 6.1 Settings page

1. Go to **Settings** (cog icon or bottom nav).
2. The Export section is visible with date presets and format options.

### 6.2 Export date presets

1. Tap **"Last 7 days"**, **"Last 30 days"**, **"All time"**.
2. Each updates a live preview count: `"X observations selected"`.

### 6.3 Focus scope filter

1. If a focus area exists, chips appear for each active area plus "All observations".
2. Tap a focus area chip — the count updates to reflect only observations in
   those categories.

### 6.4 CSV export

1. Select a date range and tap **Export CSV**.
2. A download triggers (or share sheet on iOS).
3. Open the CSV — confirm columns: id, date, time, category, tags, title, notes,
   assessment scores.

### 6.5 JSON export

1. Tap **Export JSON** — a `.json` file downloads.
2. Open it — confirm the structure includes observations with all fields, and
   optionally assessment and voice log data.

---

## Phase 7 — Focus Linking + Focus-specific Insights

### 7.1 Linking an observation to a focus area

1. Go to **Timeline**, Day view.
2. Make sure at least one active focus area exists (Phase 4.1).
3. Each observation card now shows a **🎯** button in the top-right (next to ✕).
4. Tap 🎯 on any card — the **FocusLinkSheet** slides up.
5. The sheet lists all active focus areas.
6. Tap a focus area — the icon changes from `○` to `🎯` (filled).
7. Tap **✕** to close the sheet.
8. The card now shows **"🎯 1 focus area"** at the bottom.

### 7.2 Unlinking

1. Tap 🎯 on the card again — the sheet opens with the area shown as linked (🎯).
2. Tap the linked area — it reverts to `○`.
3. Close the sheet.
4. The `"🎯 1 focus area"` indicator is gone from the card.

### 7.3 Multiple links

1. Link one observation to two different focus areas.
2. The card shows **"🎯 2 focus areas"**.
3. Open the sheet — both areas show as linked.

### 7.4 Empty state in FocusLinkSheet

1. Delete or complete all focus areas.
2. Tap 🎯 on a card.
3. The sheet shows: "No active focus areas yet".

### 7.5 Focus-specific Insights — chip selector

1. Make sure at least one active focus area with categories set exists.
2. Go to **Insights**.
3. Below the range selector, a row of chips appears: **"All data"** + one chip
   per active focus area.
4. Tap a focus area chip (e.g. "🎯 Difficult mornings").
5. A label appears: `"Showing: Difficult mornings"`.

### 7.6 Focus-scoped metrics

1. With the focus area chip selected (step 7.5):
   - **Total Logs** should reflect only observations in that area's categories
     (e.g. behavior + emotion), not the global count.
   - **Categories** breakdown shows only those categories.
   - **Time of Day** chart reflects only matching entries.
   - **Most Used Tags** shows only tags from those categories.
2. Tap **"All data"** — all metrics return to global values.

### 7.7 Focus filter changes with range

1. Select a focus area chip in Insights.
2. Switch the range from 7 days to 30 days.
3. The focus filter stays applied; Total Logs updates to reflect the longer
   range within those categories.

---

## Cross-cutting checks

### C.1 Data persists across page reloads

1. Log several entries.
2. Close the tab and reopen the app.
3. All entries are still present — data lives in IndexedDB, not memory.

### C.2 No entries state

1. Open the app with no data (or navigate to a day with nothing logged).
2. Timeline shows the 📝 empty state: "No entries yet for this day".
3. Insights shows the 🔍 empty state: "Not enough data yet".

### C.3 Filters clear correctly

1. In Timeline, apply a category filter and a search query.
2. Switch to Week view — the filter applies to the week view too.
3. Switch back to Day view, navigate to a different day — the filter persists
   (by design; clear it manually by tapping the active pill).

### C.4 Multiple focus areas on Timeline filter

1. With two active focus areas, tap one focus chip on Timeline.
2. The category filter pills update to only show categories in that focus area.
3. Tap a different focus chip — the category pills update again.
4. Tap the active chip to deactivate — all categories return.

---

## Notes

- All data is local-only (IndexedDB). There is no backend or cloud sync yet.
- The app is a PWA — you can install it to your home screen / taskbar and it
  will work offline.
- If something looks wrong, open DevTools Console for any JS errors, and
  DevTools → Application → IndexedDB to inspect the raw data.
- To reset all data: DevTools → Application → IndexedDB → right-click
  `openspectrum-db` → Delete database, then reload.
