# STYLE.md — visual contract for the Pixel Art Semester Calendar

**Status: FROZEN.** Agent B implements this in `pixel.css` + `preview.html`.
Agent C consumes the tokens and class names. British spelling in all UI copy.

Mood: cosy pastel, Stardew-adjacent. A wooden almanac pinned to a dorm wall — warm,
readable, a little hand-cut. Not neon, not "retro terminal", not brutalist.

---

## 1. The palette

**8 pastel course colours + 2 neutrals + 1 ink**, all as CSS custom properties on
`:root`. Every course fill listed here is **frozen** and matches `CONTRACT.md`
`courses[].colour`. Text placed on any fill is always `--px-ink` and must reach
**contrast ratio ≥ 4.5:1** against that fill (verify each pair; nudge the *fill*
lighter if a pair fails, but keep the hue and keep it equal to the JSON value —
if a nudge is unavoidable, tell the orchestrator so the JSON changes too).

### Light theme (`:root`)

| Token | Hex | Role |
|---|---|---|
| `--px-ink` | `#3a2e28` | all text, borders, sprite outlines |
| `--px-parchment` | `#f4ecd8` | page background |
| `--px-panel` | `#fbf6e9` | raised panel / cell background |
| `--px-lecture` | `#8ecae6` | Diagnostic Imaging |
| `--px-composite` | `#ffb4c6` | Composite Restorations |
| `--px-sign` | `#b7e4a0` | Beginner’s Sign Language |
| `--px-juris` | `#ffd98e` | Dental Jurisprudence |
| `--px-pain` | `#ffab73` | Pain Management |
| `--px-ortho` | `#8fdec9` | Orthodontics |
| `--px-pharm` | `#e6c79c` | Clinical Dental Pharmacology |
| `--px-public` | `#cdb4f0` | Dental Public Health |

Derived, non-course tokens (define explicitly, do not compute at runtime):

| Token | Light | Role |
|---|---|---|
| `--px-exam` | `#e5544e` | exam accent (star sprite, chip border) — used with hatching, never as a large flat fill behind ink text unless it also passes 4.5:1 (it does not at body size, so exam chips use a pale wash `--px-exam-wash` with an ink label) |
| `--px-exam-wash` | `#f6c6c0` | pale exam chip fill (ink text passes) |
| `--px-holiday` | `#f2b8d0` | holiday / peach-legend accent |
| `--px-break` | `#e9e2cd` | "no classes" wash |
| `--px-today` | `#fff3b0` | today-cell glow fill |
| `--px-grid` | `#d8ccae` | 1-px internal grid lines on the parchment |
| `--px-focus` | `#2b6cb0` | focus ring (survives the pixel border; 3px solid, offset 2px) |

### Dark theme — `@media (prefers-color-scheme: dark)`, redefined explicitly

| Token | Hex |
|---|---|
| `--px-ink` | `#f3e9d2` |
| `--px-parchment` | `#2a2320` |
| `--px-panel` | `#352c27` |
| `--px-lecture` | `#356f86` |
| `--px-composite` | `#8f4a5c` |
| `--px-sign` | `#4c7a3c` |
| `--px-juris` | `#8a6a24` |
| `--px-pain` | `#9c5a30` |
| `--px-ortho` | `#3a7d6b` |
| `--px-pharm` | `#7d6039` |
| `--px-public` | `#5e4c86` |
| `--px-exam` | `#ff6b63` |
| `--px-exam-wash` | `#5b3230` |
| `--px-holiday` | `#7a3a55` |
| `--px-break` | `#3a332c` |
| `--px-today` | `#5c4d1e` |
| `--px-grid` | `#4a3f37` |
| `--px-focus` | `#7cc0ff` |

In dark theme, course chips carry `--px-ink` (`#f3e9d2`) text on the darker course
fills — check each of the 8 pairs reaches 4.5:1; deepen the fill if not.

Both themes are **defined in full** (no token gets its only value inside a media
block — `:root` carries the complete light set, the media query overrides).

---

## 2. Grid, borders, corners

- **Everything on an 8-px grid.** Spacing, padding, margins, panel sizes, sprite
  boxes are multiples of 8 (4 is allowed only for hairline insets). Expose
  `--px-u: 8px` and build with `calc(var(--px-u) * n)`.
- **Chunky borders: 3–4 px**, colour `--px-ink`, `border-style: solid`.
- **Hand-cut corners**, not `border-radius`. Achieve the stepped-pixel corner with a
  layered `box-shadow` "pixel staircase" (e.g. four 4-px `box-shadow` steps at each
  corner) or an SVG border-image with integer coords. **`border-radius` is banned**
  everywhere.
- Panels get a 2-tone drop shadow suggesting a 4-px lip:
  `box-shadow: 4px 4px 0 0 var(--px-ink)` plus an inner `2px 2px 0 0` panel-tint
  highlight. No blur.
- Dividers inside panels: 2-px `--px-grid`, hard.

---

## 3. No anti-aliasing

- `image-rendering: pixelated` on any `<img>`, `<canvas>` or CSS background bitmap.
- **All decorative art is inline SVG with integer coordinates** on a 16×16 viewBox
  (see §5), or a CSS `box-shadow` pixel grid. No PNGs, no gradients, no `filter:
  blur`, no sub-pixel transforms. `transform` may only translate by whole pixels.
- Diagonal lines only at 45° or as deliberate stair-steps.
- No `opacity` on text. Layer tints as solid colours.

---

## 4. Type

- **Pixel display face**: an OFL bitmap font — **Press Start 2P** — embedded as a
  **base64 WOFF2 subset** (`@font-face { font-family: "PxDisplay"; src:
  url(data:font/woff2;base64,…) }`). Subset to the glyphs the UI actually renders
  (Latin letters, digits, space, `–—:/().,'&%°`, arrows). Keep the base64 under
  ~40 KB; if Press Start 2P won't subset that small, **Pixelify Sans** is the
  approved fallback OFL face. Ship the licence text in `src/` (`OFL.txt`).
- **Fallback stack**: `"PxDisplay", ui-monospace, "Cascadia Mono", Menlo, Consolas,
  monospace`.
- **Where the pixel face is used**: headings, month/week titles, day numbers,
  weekday initials, legend labels, chip session numbers, countdown numerals, buttons.
  Never below **10px** computed size — day numbers at 10–12px, headings 12–16px,
  the countdown "hero" numerals up to 24px (multiples of the 8-grid where possible).
- **Body / long strings** (lecturer names, exam caveats, notes, the "subject to
  change" tooltip): the **fallback monospace only**, `font-family: ui-monospace,
  …, monospace`, 12–13px, `line-height: 1.5`. Set these runs with a `.px-prose`
  class so the pixel face never touches a long word.
- `letter-spacing: 0`. `text-rendering: optimizeSpeed`. No ligatures.

---

## 5. Sprites (inline SVG, 16×16, integer coords, 2-colour: `--px-ink` outline +
one fill token)

Agent B delivers these four as reusable `<symbol id="px-…">` in a hidden `<svg>`:

| id | drawing | default fill |
|---|---|---|
| `px-tooth` | a molar: 8-px-wide crown, two roots, one occlusal groove pixel | `--px-panel` |
| `px-book` | a little closed book, spine on the left, 2-px pages edge | `--px-juris` |
| `px-star` | 5-point star for exams, chunky, 12 px across | `--px-exam` |
| `px-sprig` | holiday sprig: a stem + three leaf pixels + one berry | `--px-sign` |

Each sprite: `shape-rendering: crispEdges`, `stroke: var(--px-ink)`,
`stroke-width: 1`, no partial-pixel points. They scale only by integer multiples
(16, 32, 48).

---

## 6. Components Agent B must ship in `pixel.css` (and demo in `preview.html`)

Class names are the integration contract — Agent C targets these exact names.

| Component | Class | States / modifiers |
|---|---|---|
| App frame | `.px-frame` | — |
| Raised panel | `.px-panel` | `.px-panel--inset` |
| Button | `.px-btn` | `:hover`, `:focus-visible`, `.px-btn--active`, `[disabled]` |
| Month grid | `.px-month` | — |
| Day cell | `.px-day` | `.px-day--weekend`, `.px-day--today`, `.px-day--holiday`, `.px-day--break`, `.px-day--has-exam`, `.px-day--outside` (not in term / adjacent month), `.px-day--empty` |
| Day number | `.px-day__num` | — |
| Event chip | `.px-chip` | `.px-chip--class`, `.px-chip--exam`, `.px-chip--resit`, `.px-chip--quiz`, `.px-chip--milestone`, `.px-chip--holiday`, `.px-chip--provisional` (hatched fill), `.px-chip--dim` (course filtered off) |
| Chip course colour | `.px-chip[data-course="diagnostic-imaging"]` … | drive fill from `--px-<course>` via a `[data-course]` attribute selector for all 8 |
| Week time-grid | `.px-week` | rows `08:00`–`20:00`; `.px-week__slot`, `.px-week__now` |
| Day detail panel | `.px-daypanel` | `.px-daypanel__row`, `.px-daypanel__time`, `.px-daypanel__exam` |
| Legend | `.px-legend` | `.px-legend__chip` (toggle), `[aria-pressed]` |
| Countdown strip | `.px-countdown` | `.px-countdown__num`, `.px-countdown__label` |
| Tooltip | `.px-tip` | shown on `:hover`/`:focus-within`; the provisional caveat lives here |
| Prose run | `.px-prose` | forces the monospace fallback face |
| Visually-hidden | `.px-sr-only` | standard clip pattern |

**Provisional hatch**: a 45° 2-px stripe of `--px-exam` over `--px-exam-wash`,
done with `repeating-linear-gradient` at hard stops (`0 2px` steps — hard stops are
not anti-aliased) **or** an inline SVG `<pattern>`. Must remain legible with ink
text on top; if not, drop the hatch behind and keep a 3-px dashed `--px-exam`
border + the star sprite instead.

---

## 7. Motion & accessibility

- Focus ring: `outline: 3px solid var(--px-focus); outline-offset: 2px;` on every
  interactive element, and it must sit **outside** the chunky border (never
  clipped). Do not remove default outlines without replacing them.
- Any sprite idle-animation (e.g. the tooth "blinks") is **≤ 1 step/2s, 2 frames**,
  and is fully removed — not just slowed — under
  `@media (prefers-reduced-motion: reduce)`.
- Hit targets ≥ `calc(var(--px-u) * 5)` (40px) in at least one dimension.
- Colour is never the only signal: exams also carry the star sprite, holidays the
  sprig, provisional the hatch + dashed border, filtered-off courses the `--dim`
  desaturation *and* reduced size.
- Respect `prefers-contrast: more` by thickening borders to 4px and dropping washes
  to solid.

---

## 8. Layout targets

- Works from a 320-px phone up to a wide desktop. The month grid is 7 columns down
  to ~480px, then may switch to a scrollable agenda list below that (Agent C's
  call) — but the pixel look holds.
- Wide content (the week time-grid) scrolls inside its own
  `overflow-x: auto` container; the page body never scrolls sideways.
- Print: not required, but don't actively break it.
