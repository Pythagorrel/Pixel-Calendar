# Reminder email spec — Timetable

Read at run time by the two scheduled tasks, ** timetable — morning reminder (06:30)** and ** timetable — evening recap (18:00)**. Edit this file, push it, and the next run picks the changes up — no need to touch either task.

Live at `https://raw.githubusercontent.com/Pythagorrel/Pixel-Calendar/main/reminder-email-spec.md`.

> **This repository is public.** Nothing private belongs in this file or in `Personal Dates.md`. If you want personal dates in the emails, tell Claude and it will keep them out of the repo.

---

## Precedence

When a task fetches this file successfully, it is authoritative and overrides any conflicting rule embedded in the task's own prompt. If the fetch 404s or fails, the task falls back to its embedded rules and still sends. Either way an email goes out — this file tunes the emails, it does not gate them.

## Data sources

All public, all in this repository on `main`. Download with `curl`, parse with a script — never read 466 events by eye.

| URL suffix | Use |
|---|---|
| `schedule.json` | Authoritative. All events, courses and metadata. |
| `email-preview.html` | The approved visual reference. Match its layout. |
| `Personal Dates.md` | Optional. Skip silently on 404. |

Base: `https://raw.githubusercontent.com/Pythagorrel/Pixel-Calendar/main/`

`schedule.json` shape: `meta`, `courses[]` (`id`, `name`, `colour`, `sessionCount`, `lecturers`), `events[]` (`date`, `start`, `end`, `kind`, `courseId`, `title`, `component`, `sessionNo`, `lecturer`, `note`, `provisional`), `personal[]`.

`kind` is one of `class`, `exam`, `resit`, `quiz`, `milestone`, `holiday`, `break`.

## Timezone

Rrel is in **America/La_Paz (UTC−4)**. Times in `schedule.json` are already local as printed on the timetable — never convert them.

- The morning task fires at **10:30 UTC** = 06:30 the **same** local date.
- The evening task fires at **22:00 UTC** = 18:00 the **same** local date.

Both therefore recap or report the UTC date, and "tomorrow" is the day after it. Compute this from `date -u` in a script rather than by hand — and re-check it if either task's schedule ever moves, since a fire time that crosses midnight UTC shifts the local date by a day.

## Rules that always apply

- **Merge** consecutive same-course, same-component sessions on one day into a single block, session numbers as a range: `08:00–11:00 · Sessions 1–3`.
- **Never present a provisional date as fixed.** Every Week 15/16 exam carries `provisional: true` — label those "unconfirmed — final dates from the Examinations Section".
- A `LAB` or `LEC/LAB` component is a **practical**; tag it visibly.
- Never invent a venue, lecturer or time the data does not carry. Omit rather than guess.
- **Days off** means a weekday with no `class` events. Between now and 18 December 2026 the only true one is **Monday 19 October, National Heroes Day** — everything else that looks free is re-sit or examination week, which is not a day off.
- Read `Personal Dates.md` if present; when a date matches, lead with it.
- British spelling.

## Morning email — 06:30, covers today

Subject: `<Day> <D Mon> — <n> courses, <first>–<last>`, plus ` · <the day's standout>` when one applies (a course starting, an exam, a quiz, a day off).

1. Kicker "Today", the date as a headline, then one line: week number, contact hours, first and last times.
2. A callout **only if the day earns one** — a course's first or last session, an exam or quiz, a personal date, an unusually long day, a day off. No filler callout on an ordinary day.
3. Session table: time range, course name with its colour swatch, then session numbers, lecturer and venue beneath. Free gaps of two hours or more get an italic "Free — N hours" row.
4. Footer: whether anything is assessed today; a one-line note on tomorrow if it is heavy or unusual; days until the next day off.

## Evening email — 18:00, recaps today and previews tomorrow

Subject: `<Day> <D Mon> recap — <what she had>`, plus a forward-looking clause when tomorrow is notable.

1. Kicker "How today went", the recapped date as a headline, contact hours.
2. Today's sessions as a table, with `Sessions X–Y of <course total>` so progress through each course shows.
3. A callout about tomorrow **only** when it is heavy, unusual, or carries an assessment.
4. A kicker with tomorrow's date, then tomorrow's session table in the same format.
5. Footer: the next key date; days until the next day off.

**The forward-looking half is the point.** Knowing what she had is mildly interesting; knowing tomorrow starts at 08:00 and ends at 20:00 is what gets acted on. Never drop it.

## Empty days

- **Morning, no classes:** still send, a few lines — nothing timetabled, why (weekend, holiday, break, re-sit week, exam week), and the next day that does have classes.
- **Evening, nothing today:** one line saying so and why, then tomorrow in full. If tomorrow is empty too, say when classes next resume.

## Visual style

Plain, readable HTML email. Inline styles only; no external images, no web fonts, no tracking pixels. System sans-serif stack. Ink `#3a2e28` on white, parchment `#fbf6e9` for callouts, rules `#e3d9c4`, muted text `#7a6a5e`. A `@media (prefers-color-scheme: dark)` block is welcome but must never be the only definition of a colour.

Course swatches use each course's `colour` field from `schedule.json` — a 10 px square before the name. Practical tags are small uppercase pills. The time column is tabular, left-aligned, roughly 104 px.

## Delivery and failure

Send as HTML to **jorreldesantos066@gmail.com** with the Gmail tool.

If `schedule.json` cannot be downloaded or does not parse, **send no email** and report the failure in the run output. A wrong timetable is worse than a missing one.
