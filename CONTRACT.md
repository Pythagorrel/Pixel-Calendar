# CONTRACT.md — data contract for the Pixel Art Semester Calendar

**Status: FROZEN.** Every downstream agent reads this file. Nobody edits it without
telling the orchestrator. If two agents disagree, the orchestrator decides and
updates this file.

British spelling throughout code, comments and UI copy (`colour`, `centre`,
`organise`, `behaviour`).

---

## 1. Sources and authority

Vault: `C:\Users\jorre\Documents\Cowork Sandbox\Cowork Vault\Schedule\`

| File | Authoritative for |
|---|---|
| `Semester 1 — Week Index.md` | **Every teaching session and the Thursday quiz.** The `## Week N — …` tables are the authoritative session list. |
| `Semester 1 — Overview.md` | **Examinations, re-sits, public holidays, breaks, and key-date milestones.** Use the *Examinations*, *Public holidays and breaks*, and *Key dates* tables. |
| `Courses/<Course>.md` | **Per-course lecturer(s), components, session totals, first/last session** (frontmatter + *At a glance*). Used to build the `courses[]` array and to cross-check counts. |

Rules:

- **Never invent a value the source does not print.** Missing → `null` plus a `note`.
- The Week Index wins for anything about a class session (date, time, course,
  component, session number, lecturer).
- The Overview wins for exams/holidays/milestones. The Week 14–16 rows *also* appear
  in the Week Index; the parser takes exams from the **Overview** table because it
  carries the fuller "Notes" text, and must not double-emit the Week Index copies.
- Course notes win for `lecturers`, `components`, `sessionCount`, `firstSession`,
  `lastSession`.

Course names appear as Obsidian wikilinks (`[[Diagnostic Imaging]]`). Strip `[[` `]]`
when parsing. Times use an EN DASH `–` (U+2013): `08:00–09:00`, never a hyphen.
Em dash `—` (U+2014) and the curly apostrophe `’` (U+2019) appear in filenames and
cells; a lone `—` in a Component / No. / Lecturer cell means "not stated" → `null`.

---

## 2. `schedule.json` — exact shape

UTF-8, LF line endings, **2-space indent**, keys in the order shown below, a single
trailing newline. Non-ASCII characters emitted **literally** (no `\uXXXX`), `/` not
escaped. Julia and Python outputs must be **byte-identical** — Agent A should write a
small shared-spec serialiser in each language rather than rely on a library's
default. Sort order for `events` and `personal`: by `date`, then `start` (nulls
first), then `kind`, then `courseId` (nulls last), then `sessionNo` (numeric where
possible, else string, nulls last), then `id`.

```json
{
  "meta": {
    "programme": "Undergraduate Dental Programme",
    "year": "Year 3",
    "cohort": "Class of 2029",
    "semester": "Semester 1, 2026-2027",
    "termStart": "2026-09-01",
    "termEnd": "2026-12-18",
    "generated": "2026-09-07T12:00:00Z",
    "source": "Draft 6 — not a final timetable",
    "counts": { "sessions": 410, "courses": 8, "weeks": 16 }
  },
  "courses": [
    {
      "id": "diagnostic-imaging",
      "name": "Diagnostic Imaging",
      "lecturers": ["Dr. Osagbemiro", "Dr. Osagbemiro/Staff Dental Assistants"],
      "components": ["LEC", "LAB"],
      "sessionCount": 114,
      "firstSession": "2026-09-09",
      "lastSession": "2026-11-27",
      "colour": "#8ecae6"
    }
  ],
  "events": [
    {
      "id": "2026-09-08-dental-public-health-1",
      "date": "2026-09-08",
      "start": "08:00",
      "end": "09:00",
      "kind": "class",
      "courseId": "dental-public-health",
      "title": "Dental Public Health",
      "component": null,
      "sessionNo": "1",
      "lecturer": "Dr. Balkaran",
      "venue": null,
      "note": null,
      "provisional": false,
      "endDate": null
    }
  ],
  "personal": []
}
```

### 2.1 `meta`

`counts` is the **hard-coded expectation**
`{ "sessions": 410, "courses": 8, "weeks": 16 }`, where `sessions` counts
**`class` + `quiz` events together** (teaching sessions). The parser must
**exit non-zero** if its tally of `kind == "class"` events ≠ **409**, or
`kind == "quiz"` ≠ **1**, or `class + quiz` ≠ 410, or course count ≠ 8.

`generated` = ISO-8601 UTC timestamp. To keep the Julia and Python outputs
**byte-identical and re-runnable**, `generated` defaults to
`2026-09-07T12:00:00Z` (the Week Index frontmatter `updated` date at noon UTC);
an environment variable `PIXELCAL_GENERATED` overrides it with a real timestamp
when a caller wants one.

### 2.2 `courses[]` — exactly 8, in this order and with these frozen values

| id | name | lecturers | components | sessionCount | firstSession | lastSession | colour |
|---|---|---|---|---:|---|---|---|
| `diagnostic-imaging` | Diagnostic Imaging | `["Dr. Osagbemiro", "Dr. Osagbemiro/Staff Dental Assistants"]` | `["LEC","LAB"]` | 114 | 2026-09-09 | 2026-11-27 | `#8ecae6` |
| `composite-restorations` | Composite Restorations | `["Dr. McDonald"]` | `["LEC/LAB"]` | 78 | 2026-09-07 | 2026-11-20 | `#ffb4c6` |
| `beginners-sign-language` | Beginner’s Sign Language | `["Dr. Cumberbatch"]` | `[]` | 50 | 2026-09-04 | 2026-11-27 | `#b7e4a0` |
| `dental-jurisprudence` | Dental Jurisprudence | `["Dr. Moore"]` | `["LEC"]` | 41 | 2026-09-10 | 2026-11-20 | `#ffd98e` |
| `pain-management` | Pain Management | `["Dr. Ramirez", "Dr. Ramirez/Dr. Jampany"]` | `["LEC","LAB"]` | 39 | 2026-09-11 | 2026-11-24 | `#ffab73` |
| `orthodontics` | Orthodontics | `["Dr. Meeks", "Dr. Mitchell"]` | `[]` | 36 | 2026-09-09 | 2026-11-25 | `#8fdec9` |
| `clinical-dental-pharmacology` | Clinical Dental Pharmacology | `["Dr. Moncrieffe"]` | `[]` | 26 | 2026-09-07 | 2026-11-16 | `#e6c79c` |
| `dental-public-health` | Dental Public Health | `["Dr. Balkaran"]` | `[]` | 26 | 2026-09-08 | 2026-11-10 | `#cdb4f0` |

- `name` keeps the curly apostrophe in "Beginner’s".
- `id` = course name, lower-cased, curly apostrophe dropped, spaces → `-`.
- `lecturers` = the distinct lecturer strings the course frontmatter lists, in
  frontmatter order. `Dr. Moore, Mr.` in the Jurisprudence frontmatter → keep only
  `["Dr. Moore"]` (the bare `Mr.` is not a name; see §4).
- `components` = course frontmatter `components` (empty array where the source
  records none). `QUIZ` is **not** a component here.
- `sessionCount` is the **course-note total** and must equal the parser's own count
  of `kind == "class"` events for that `courseId` (the quiz is `kind == "quiz"`, so
  Jurisprudence has 40 class + 1 quiz = the note's 41; see §2.4).
- `colour` values are **frozen** (agreed with STYLE.md). Agent B may tune shades for
  the dark theme in CSS but the JSON carries exactly these hex strings.

### 2.3 `events[]` — union of classes, the quiz, exams, re-sits, holidays, breaks, milestones

Event object, keys in this order:

| key | type | notes |
|---|---|---|
| `id` | string | stable slug, unique. See §2.5. |
| `date` | `"YYYY-MM-DD"` | start date. For multi-day breaks, the first day. |
| `start` | `"HH:MM"` \| null | 24h. null for all-day events (holidays, breaks, most milestones). |
| `end` | `"HH:MM"` \| null | null when the source prints no finish time *and* none can be carried without inventing — but see the specific carried values in §3. |
| `kind` | enum | `class` \| `exam` \| `resit` \| `quiz` \| `milestone` \| `holiday` \| `break` |
| `courseId` | string \| null | one of the 8 ids, or null when the row is not one of the 8 courses (e.g. `DENT1103`, `H&N Anatomy`, `National Heroes Day`, `Last day of teaching`). |
| `title` | string | display title. For `class`/`quiz`/course exams: the course `name`. For non-course exams: the source "Paper/Course" text verbatim, wikilink brackets stripped, minus a trailing ` EXAM` (moved to `component`). For holidays/breaks/milestones: the occasion/event text verbatim. |
| `component` | string \| null | `LEC` \| `LAB` \| `LEC/LAB` for classes; `EXAM` \| `LAB EXAM` \| `LAB ICA` for exams/re-sits; null for the quiz, holidays, breaks, milestones, and unlabelled classes (BSL, Ortho, DPH, CDP). |
| `sessionNo` | string \| null | the source's per-component number **as a string** (`"1"`, `"33"`, `"A"`, `"J"`). null where the source prints `—` (BSL rows, quiz, exams, holidays, milestones). Never merged by the parser (see §2.6). |
| `lecturer` | string \| null | verbatim single string from the Lecturer cell, incl. slash strings like `Dr. Osagbemiro/Staff Dental Assistants`. null for `—`, for `-Wk. N` artefacts, for a bare `Mr.`, and for `BSL` in the lecturer position (see §4). |
| `venue` | null | always null — the source records no per-session room (blanket venue note only). |
| `note` | string \| null | free text carried from the source: the `-Wk. N` artefact text, the "surname missing" flag, the reconstructed-time flag, the provisional-exam caveat, the split-half-column flag, etc. See §3 and §4. null when there is nothing to say. |
| `provisional` | bool | `true` **only** for Week 15 and Week 16 exams. `false` for classes, the quiz, Week 14 re-sits, holidays, breaks, milestones. |
| `endDate` | `"YYYY-MM-DD"` \| null | **orchestrator extension to the brief's shape.** Non-null for multi-day spans: the two "No classes scheduled" `break` weeks, and the three exam-week `milestone` span rows (Weeks 14/15/16). null for everything else. |

### 2.4 What becomes which `kind`

- **`class`** — every Week Index `## Week N` row for Weeks 1–13 that resolves to one
  of the 8 courses and is not the quiz. Expected total **409** (the 8 course
  `sessionCount` values sum to 410, of which the Jurisprudence quiz is 1 → 409
  `class` + 1 `quiz`).
- **`quiz`** — the single Week Index row `2026-11-19 (Thu) | 18:00–19:00 |
  [[Dental Jurisprudence]] | QUIZ | — | —`. `courseId: "dental-jurisprudence"`,
  `component: null`, `sessionNo: null`, `lecturer: null`,
  `note: "Evening quiz. Component label \"QUIZ\" is ours, not the source's; no lecturer or venue recorded."`
- **`resit`** — the 8 rows of the Overview *Examinations* table dated 2026-11-30 to
  2026-12-04 (all marked "Re-sit examination"). `provisional: false`.
  `courseId: null` (none are among the 8). `title` = paper text minus trailing
  ` EXAM`, `component` = `EXAM` / `LAB EXAM` / `LAB ICA`.
  `note: "Re-sit examination."`
- **`exam`** — the 20 rows of the Overview *Examinations* table dated 2026-12-07 to
  2026-12-18. `provisional: true`. `courseId` set when the paper is a wikilink to one
  of the 8 courses, else null. `note` **must begin**
  `"Subject to change — final dates from the Examinations Section."` and then append,
  space-separated, any extra source detail from the Notes cell (e.g.
  `"Written & Data Interpretation."`, the DENT1103 finish-time flag, the DENT2106
  unbalanced-parenthesis text reproduced as printed, the 2026-12-15 split-half-column
  flag).
- **`holiday`** — from the Overview *Public holidays and breaks* table, the
  single-date rows: `2026-10-19` National Heroes Day, `2026-12-25` Christmas Day,
  `2026-12-26` Boxing Day, `2027-01-01` New Year’s Day. `start`/`end` null,
  `courseId` null, `endDate` null.
- **`break`** — the two ranged rows `2026-12-21 to 2026-12-25` and
  `2026-12-28 to 2027-01-01` ("No classes scheduled"). `date` = range start,
  `endDate` = range end, `start`/`end` null, `title: "No classes scheduled"`.
- **`milestone`** — from the Overview *Key dates* table, every row that is not
  already covered by a holiday/break above. This includes "Teaching starts"
  (2026-09-01), each `… begins` / `… ends` course marker, "Last day of teaching"
  (2026-11-27), the three exam-week span rows (Weeks 14/15/16 — `date` = span start,
  `endDate` = span end), and "Semester 1 ends" (2026-12-18). `start`/`end` null
  (except the quiz key-date row, which duplicates the quiz and is **skipped** to
  avoid a double). `courseId` set when a course is named. Keep the parenthetical
  ("Dr. Meeks strand", "red end-marker in source") in `note`.

*Do* emit the course-marker milestones even though they restate a
`firstSession`/`lastSession` (they drive the UI's key-date list); just make the
`id` distinct (`…-milestone-…`) so they never collide with the class rows. The
Overview *Key dates* table has 25 rows; after dropping the 2026-11-19 quiz row
(duplicate of the `quiz` event) and the two "No classes scheduled" rows
(emitted as `break`), **22 `milestone` events** remain. Emit exactly those 22.

### 2.4a `id` formula, read precisely

In `{date}-{courseId}-{component or kind}-{sessionNo or startHHMM}` the
`component or kind` segment is: the `component` when non-null; otherwise the
`kind` **only when it is not `"class"`**. So an unlabelled class slugs to
`{date}-{courseId}-{sessionNo}` (e.g. `2026-09-08-dental-public-health-1`), the
quiz to `{date}-dental-jurisprudence-quiz-1800`.

### 2.5 `id` algorithm (stable, unique)

```
slug(s) = lower-case; replace each run of [^a-z0-9] with "-"; trim leading/trailing "-"
base:
  class / quiz : "{date}-{courseId}-{component or kind}-{sessionNo or startHHMM}"
  exam / resit : "{date}-{courseId or slug(title)}-{slug(component)}-{startHHMM}"
  holiday      : "{date}-holiday-{slug(title)}"
  break        : "{date}-break"
  milestone    : "{date}-milestone-{slug(title)}"
```

`startHHMM` = `start` with the colon removed (`"0800"`). If two ids still collide
(shouldn't, given sessionNo), append `-2`, `-3`. The parser must assert global
uniqueness and fail otherwise.

### 2.6 Merging is the app's job, not the parser's

The parser emits **one event per source row, unmerged**. The app merges for display
only: consecutive events with the same `date`, `courseId`, `component`, `kind`,
`lecturer` and contiguous times (`end` of one == `start` of the next) collapse into
one block whose displayed session number is a range (`"1–3"`) built from the first
and last `sessionNo`. The underlying events and their count are untouched, so totals
still reconcile. Rows with `sessionNo == null` (BSL) still merge on the time
contiguity rule but show no number.

### 2.7 `personal[]`

Same object shape as `events` (with `endDate`). Read from an optional
`VAULT\Personal Dates.md` containing a `| Date | Occasion |` table. Each row →
`{ kind: "milestone", courseId: null, title: <Occasion>, date: <Date>, start: null,
end: null, component: null, sessionNo: null, lecturer: null, venue: null,
note: "Personal date.", provisional: false, endDate: null }`, `id` =
`"{date}-personal-{slug(title)}"`. If the file is absent or has no data rows, emit
`[]`. The parser must not fail when the file is missing.

---

## 3. Irregular rows the parser must handle (from the source's own caveat lists)

| Where | Handling |
|---|---|
| **Week 1**: only `2026-09-04` Fri, two BSL slots at `16:00–16:59` / `17:00–17:59`. No 1–3 Sept sessions. | Emit the two BSL classes with `end` `"16:59"` / `"17:59"` exactly as printed. `sessionNo: null`. |
| **`2026-09-04` teaching-start header** "TUES- Teaching Starts" | Ignored for sessions; the "Teaching starts" milestone comes from the Overview Key dates row dated `2026-09-01`. |
| **Week 6 `2026-10-06` (Tue)**: blank TUES column header in source | Day is `2026-10-06`; emit its DPH (13–15) and Composite (29–32) classes normally. `note: null` on the rows; the blank-header fact is a source caveat, not per-row data. |
| **Orthodontics 20, `2026-11-11` (Wed)**: blank time label | Week Index prints `19:00–20:00`. Emit `start:"19:00" end:"20:00"`, `note: "Time label blank in source; 19:00–20:00 reconstructed from the preceding row and the Wednesday 18:00–20:00 pattern."` |
| **Jurisprudence `LEC 24`, `2026-10-16`**: lecturer cell reads `Mr.` | `lecturer: null`, `note: "Source gives the lecturer as \"Mr.\" only — surname missing."` |
| **Jurisprudence Friday `-Wk. N` artefacts** (`LEC` 15, 19, 23, 27, 31, 37, 39 carry `-Wk. 1`…`-Wk. 6`, `-Wk. 4` twice on 10-23 and 10-30) | `lecturer: null`, `note: "Source text in the lecturer position: \"-Wk. N\" — a rotation-style artefact the source does not explain."` (substitute the actual `-Wk. N`). The paired even-numbered slot on the same day has an empty lecturer cell → `lecturer: null`, `note: null`. |
| **Jurisprudence 13 lecturer-less `LEC` slots** (15,16,19,20,23,27,28,31,32,37,38,39,40) | `lecturer: null`. Add the `-Wk. N` note only to the seven that carry it; the rest get `note: null`. |
| **Jurisprudence quiz `2026-11-19` 18:00–19:00** | `kind: "quiz"` per §2.4. |
| **Composite Restorations 55, `2026-11-02` 15:00–16:00**: cell reads `… 55 BSL` | `lecturer: null`, `note: "Source text \"BSL\" in the lecturer position — flagged as non-lecturer trailing text, not resolved to a name."` |
| **Orthodontics Dr. Mitchell block**: `sessionNo` values `A`–`J` | Keep as strings `"A"`…`"J"`. `lecturer: "Dr. Mitchell"`. |
| **`2026-10-19` National Heroes Day** — appears as a row inside the Week 8 table *and* in the Overview holidays table | Emit **one** `holiday` event (from the Overview). Do not emit a class for it. |
| **Week 15 Mon `2026-12-07` DENT1103 (Embryology and Histology) EXAM** — no finish time | `start:"09:00"`, `end:"10:00"`, `note` appends `"Finish time not stated in source; recorded as 09:00–10:00."` `courseId: null`, `title: "DENT1103 (Embryology and Histology)"`, `component: "EXAM"`, `provisional: true`. |
| **Week 15 Mon `2026-12-07` CDP EXAM 09:00–12:00** — shares the split column | Emit normally, `courseId: "clinical-dental-pharmacology"`, `start:"09:00" end:"12:00"`. |
| **Week 15 Tue `2026-12-08` DENT2106** — Notes cell has an unbalanced parenthesis | Reproduce the Notes text exactly as printed in `note` (after the standard caveat sentence). |
| **Week 16 Tue `2026-12-15` Composite LAB EXAM** — drawn across split half-columns | Two events: `08:00–12:00` and `13:00–17:00`, each `note` appends `"Drawn across split half-columns in the source; morning finish time approximate."` |
| **`2026-11-27`** — last teaching day; DI LAB 79/80 sit `10:00–12:00` not the usual 13:00 | Emit from the Week Index times as printed. Also emit the "Last day of teaching" milestone (Overview). |
| **`2026-12-18`** — DI EXAM + PM LAB EXAM, and "Semester 1 ends" | Emit both exams (`provisional: true`) and the "Semester 1 ends" milestone. |
| **Grey shaded blocks** (mostly Thursday afternoons) | Carry no text; not sessions; not emitted. |
| **Semester 2 stub grid** | Not in range; not emitted. |

`end` is **carried, not invented**, only where the Week Index / Overview themselves
print it (which is every class row and every exam row except DENT1103, handled
above).

---

## 4. Facts that appear in more than one note (conflict map)

| Fact | Notes it appears in | Winner |
|---|---|---|
| Session date/time/component/No./lecturer | Week Index + the course note's Schedule table | **Week Index** |
| Per-course session total | Course frontmatter + Overview *Courses* table + "weeks at a glance" | **Course frontmatter** (all three agree: 114/78/50/41/39/36/26/26) |
| `firstSession` / `lastSession` | Course frontmatter + Overview *Key dates* | **Course frontmatter** |
| Exam date/time/paper | Overview *Examinations* + Week Index Weeks 14–16 + course note *Assessment* | **Overview *Examinations*** |
| Holiday / break dates | Overview *Public holidays and breaks* + Week Index Week 8 row | **Overview** |
| Lecturer identity for a course | Course frontmatter + Overview *Courses* + per-row cells | Course frontmatter for `courses[].lecturers`; per-row Week Index cell for each event's `lecturer` |
| National Heroes Day | Week Index Week 8 table row + Overview holidays table | **Overview** (emit once, as `holiday`) |
| The `QUIZ` label | Week Index + Jurisprudence note (both say it is *our* label) | Either — both agree; keep the disclaimer in `note` |

All eight courses' totals are internally consistent across the three places they
appear, so there is no live numeric conflict to resolve — but the parser must still
**verify** rather than assume.

---

## 5. Parser output contract (stdout + exit code)

`parse_schedule.jl` / `parse_schedule.py` both:

1. Write `schedule.json` (byte-identical between the two) next to the script, or to a
   path given as `argv[1]`.
2. Print a reconciliation summary to stdout: total `class` events, total events by
   `kind`, per-course `class` counts vs the expected `sessionCount`, and a list of
   any source row that could not be parsed.
3. **Exit non-zero** if: total `class` ≠ 409; total `quiz` ≠ 1;
   `class + quiz` ≠ 410; any per-course `class` count ≠ its `sessionCount` minus
   (1 if that course owns the quiz else 0) — i.e. Jurisprudence expects 40 `class`
   + 1 `quiz`; course count ≠ 8; milestone count ≠ 22; any unparsed row; any
   duplicate `id`; `events` not in sorted order.
4. Be re-runnable with no hand editing when a later draft lands.

Expected `kind` tally for Draft 6:

| kind | count |
|---|---|
| class | 409 |
| quiz | 1 |
| exam | 20 |
| resit | 8 |
| holiday | 4 |
| break | 2 |
| milestone | 22 |

`class + quiz` = 410 = `meta.counts.sessions`. The `milestone` count is the
Overview *Key dates* table (25 rows) minus the quiz row minus the two "No classes
scheduled" rows = 22. The parser prints every tally; the verifier re-derives them
from the sources.
