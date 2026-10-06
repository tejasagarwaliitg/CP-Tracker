# CP Tracker — UI Review

**Audited:** 2026-10-06
**Baseline:** abstract 6-pillar standards (no UI-SPEC.md, standalone non-GSD project)
**Screenshots:** not captured — no Playwright-MCP in session; `npx playwright` CLI present but no browser binary installed (`chrome-headless-shell` missing), so no captures. Dev server detected alive (`curl http://localhost:8137 → 200`). App renders empty/placeholder states until a CF handle is entered, so this is a code-only audit of `index.html` (397 lines, single-file app). Empty-state strings audited as evidence.
**Registry audit:** skipped — no `components.json`, no shadcn, no third-party registries.

---

## Pillar Scores

| Pillar | Score | Key Finding |
|--------|-------|-------------|
| 1. Copywriting | 3/4 | Empty/error states specific and actionable; placeholder "User 1/User 2" tab labels ship as default UI |
| 2. Visuals | 2/4 | Icon-only buttons (☰ ✕ ★) have zero aria-labels; all data viz is canvas-only with no DOM fallback |
| 3. Color | 3/4 | Token discipline good, accent used sparingly; canvas/JS colors hardcoded instead of reusing vars |
| 4. Typography | 3/4 | Single system stack, minimal weights; 9px heatmap labels too small, heading scale unstyled |
| 5. Spacing | 2/4 | Consistent padding/gap scale on desktop; data tables have no overflow wrapper — crush risk at 375px |
| 6. Experience Design | 3/4 | Loading/error/empty coverage present; async Sync/Fetch buttons never disable — double-fire possible |

**Overall: 16/24**

---

## Top 3 Priority Fixes

1. **Add `aria-label`s to icon-only controls + `:focus-visible` styles** — ☰/✕/★ buttons are invisible to screen readers and keyboard users get no focus ring (only `.hgrid i:hover` exists, `index.html:35`) — add `aria-label="Open sidebar"` / `"Close"` / `"Star problem"` and a 2px `var(--acc)` focus outline.
2. **Wrap data tables in a horizontal-scroll container** — heatmap already has `.hwrap{overflow-x:auto}` (`index.html:30`) but the solved table (`index.html:285`) and compare table (`index.html:356`) don't; at 375px the 4–5 column tables (tags + dates) will squeeze/overflow — reuse the `.hwrap` pattern with a `.twrap` div.
3. **Disable async buttons while pending + Esc-close for sidebar** — `syncAll`/`fetchCF` leave `#syncAll`/`#fetchCF` clickable mid-flight (`index.html:382`), so rapid clicks fire parallel CF paged syncs; set `disabled` during the `await` and close `#side` on `Escape` alongside the existing scrim click (`index.html:386`).

---

## Detailed Findings

### Pillar 1: Copywriting (3/4) — WARNING

**What's good (evidence):**
- Empty states are specific and tell the user what to do next, not generic "No data":
  - `index.html:207` — `No matches / not synced yet — open ☰ and Save & Sync.` vs `Open ☰, enter your CF handle, then Save & Sync.`
  - `index.html:278` — `Nothing today yet.` / `index.html:361` — `No contests yet.` / `index.html:220` — `no rated solves in period`
- Error strings name the cause and the remedy: `index.html:160` `CF sync failed — check handles.`, `index.html:371` `CF API failed.`, `index.html:116` `Firebase failed — local fallback.`
- CTAs are verb-led: `Save & Sync all` (`:51`), `↻ Sync all from CF` (`:63`), `Fetch ratings` (`:65`), search placeholder `Search 123-D, dp…` (`:284`). No `Submit`/`Click Here`/`OK` generics (only `OK` is the CF API `verdict`, `:144–151`).

**Findings:**
- [WARNING] Placeholder identity ships as UI: `USERS` defaults are `User 1` / `User 2` (`index.html:73–76`), rendered verbatim as tab labels (`paintTabs`, `:260`). First-run tabs read "User 1 · User 2 · Compare ⚖️" — generic, not real content. Fix: default tabs to `Add handle +` affordance or `Unnamed tracker 1` with an inline prompt.
- [WARNING, minor] Terse/transient status strings: `Connecting…` (`:57`), `Syncing from Codeforces…` → `Synced from CF.` (`:158–159`), `Updated 12:00:00` (`:370`). Acceptable, but `Synced from CF.` gives no scope (how many users/problems). Fix: `Synced 2 users · 340 problems from CF.`
- [Minor] `Track users` sidebar heading (`:49`) + `Compare ⚖️` tab (`:260`): heading is vague ("Track whose?"), emoji in tab label is decorative noise. Fix: `Codeforces handles`, `Compare`.

### Pillar 2: Visuals (2/4) — WARNING

**What's good:** clear focal point (today-strip stat block `0 solved today · streak · total · starred`, `:273–278`); card-based hierarchy with size/weight differentiation (22px stat numerals vs 12px muted labels, `:28`); heatmap has Less→More legend (`:189`); grouped-bar charts draw legends (`:331`); sidebar collapsed by default keeps first paint focused.

**Findings:**
- [WARNING] Every icon-only control lacks an accessible name — zero `aria-label`/`role` attributes in the file (only `title=` on heatmap cells, `:184`). Affected: `#menuBtn` ☰ (`:56`), `#sideClose` ✕ (`:49`), all star toggles ★/☆ (`:201`, `:278`), `↻` sync (`:63`). Screen-reader users hear "button" with no name. Fix per Top Fix #1.
- [WARNING] All five data visualizations are `<canvas>` with no DOM/text fallback (`:66`, `:280`, `:281`, `:341`, `:342`). Pre-sync the page shows blank dark boxes; canvas-painted "Nothing solved in this period." (`:223`, `:247`) and "No data — sync first." (`:312`) are pixels, not selectable/AT-readable text. Fix: pair each canvas with a visually-hidden (or visible) DOM summary, e.g. `<p class="mut" id="rchartAlt">`.
- [WARNING] No keyboard focus styling anywhere — the only interaction outline is `.hgrid i:hover` (`:35`). Tab order exists (real `<button>`s) but focus is invisible. Fix: `button:focus-visible, input:focus-visible, select:focus-visible { outline: 2px solid var(--acc); outline-offset: 2px; }`.
- [Minor] Stat strip mixes interactive star buttons inline with links (`:278` today-list) — dense tap targets on mobile, no spacing between `·`-joined items.

### Pillar 3: Color (3/4) — WARNING

**What's good:** centralized `:root` tokens (`:8` — `--bg #0f1115`, `--card #171b22`, `--line #2a303b`, `--txt #e8eaf0`, `--mut #9aa3b2`, `--acc #4c8dff`, `--star #ffc531`, heatmap `--g0–g4` GitHub greens). Accent `var(--acc)` used only 4× (links, active tab, heatmap hover, tag bars) — disciplined ~60/30/10 split (dark bg dominant, card surfaces, blue accent sparse). Sync status dots use semantic green/amber (`:102`, `:107`).

**Findings:**
- [WARNING] JS-painted colors bypass the token system with ~20 hardcoded hexes: `cfColor` rank palette (`:192` — `#555c68 #9aa3b2 #39d353 #2dd4bf #5b8cff #c366ff #ff9a3c #ff5c5c`), `UCOLS` (`:307`), chart `cols` (`:364`, duplicate of `UCOLS`), canvas text `#9aa3b2`/`#e8eaf0` repeated at `:223, :232, :234, :247, :252, :255, :312, :316, :324, :326, :331`. Visual output is coherent, but a theme change to `:root` won't propagate to any chart. Fix: read tokens via `getComputedStyle(document.documentElement).getPropertyValue('--acc')` or define a `PALETTE` const mirroring `:root`.
- [Minor] CF rank colors (`cfColor`) vs user-series colors (`UCOLS`) are two unrelated palettes on the same Compare screen — a user's bar color (`#ff5c5c`) can coincide with CF "red / grandmaster" semantics. Justified (CF rank colors are conventional), but document or separate usage. No change required unless confusion reported.
- [Minor] `#fff` active-tab text (`:14`) and `#0008` scrim (`:40`), `#0c0e12` canvas/input bg (`:17`, `:22`) are un-tokenized one-offs. Fix: `--input:#0c0e12; --on-acc:#fff;`.

### Pillar 4: Typography (3/4) — WARNING

**What's good:** one system stack (`system-ui`, `:9`, `:249`, `:315` — zero webfont cost); restrained scale — base 15px, table 14px, muted/label 12–13px, stat numerals 22px, star glyph 18px; weights minimal (default + `<b>`/headings only). No weight soup.

**Findings:**
- [WARNING] Heatmap month labels at 9px (`:32` — `.hlabels span{...font-size:9px}`) are below readable minimum and will render as blurs on low-dpi screens. Fix: 10–11px minimum, drop alternate labels if space is tight.
- [WARNING] Heading scale is entirely browser-default (`h1` `:56`, `h3` `:280–282, :341–342` have margins zeroed in places but no sizes) while stat numerals are fixed 22px (`:28`) — on some UA stylesheets section `h3`s (~18–19px) sit too close to stat numerals, weakening hierarchy. Fix: explicit `h1{font-size:22px} h3{font-size:15px}` (or 16px) in the stylesheet.
- [Minor] Canvas axis/tag text at 11–12px (`:249`, `:315`) is small but acceptable inside charts; ensure canvas `width=900` backing store vs CSS `width:100%` doesn't blur it on hidpi (no `devicePixelRatio` scaling anywhere — charts will look soft on retina; consider scaling backing store by `dpr`).

### Pillar 5: Spacing (2/4) — WARNING

**What's good:** genuinely consistent scale — paddings 6/8/12/14/16, gaps 3/6/8/10/20, radii 8/10 (pill 20), container `max-width:960px`, table cells `6px 8px`. No arbitrary one-off values; `flex-wrap` on nav/row/strip (`:12`, `:16`, `:27`) plus `min-width:140px` on row children gives a sane narrow-screen fallback.

**Findings:**
- [WARNING] Tables have no scroll container: heatmap got `.hwrap{overflow-x:auto}` (`:30`) but the solved-problem table (`:285`, 4 cols incl. wrapping tag lists) and the compare table (`:356`, 5 cols) sit directly in `.card`. At 375×812 the tag column forces squeeze/overflow with no scroll affordance. Fix per Top Fix #2: `<div class="hwrap"><table>…` (rename or add `.twrap`).
- [WARNING] Mixed container paddings: `header,main` 12px/16px (`:10`), `.card` 12px (`:15`), `#side` 14px (`:38`) — 12 vs 14 vs 16 gutters misalign sidebar content edge against main column when open. Fix: unify on 16px page gutter / 12px card inset (or 16/16).
- [Minor] Today-strip `gap:20px` (`:27`) vs everywhere-else 6–10px gaps is a scale jump; fine as section separation but tighten to 16px to match the 4px-grid feel of the rest.

### Pillar 6: Experience Design (3/4) — WARNING

**What's good (state coverage):**
- Loading: `Connecting…` (`:57`) → `Syncing from Codeforces…` (`:158`) → `Synced…`/`Updated …` (`:159`, `:370`); compare fetch shows `Loading…` (`:350`).
- Errors: `try/catch` on `initFB` (`:116`), `syncAll` (`:160`), `fetchCF` (`:371`) — all surface user-facing strings, none throw raw to console-only.
- Empty: every view has a designed empty state (Pillar 1 evidence); table capped at 500 rows with `Showing first 500 — use search.` overflow note (`:209`).
- Reversible-by-design: star/unstar writes instantly and updates the count optimistically (`:296–300`) — correctly no destructive-confirm friction.
- Persistence fallback: Firestore → localStorage (`LS_M`, `LS_H`, `:95–100`) so single-device use never loses stars/handles.

**Findings:**
- [WARNING] No disabled/pending guard on async actions: `#syncAll` (`:382`), `#fetchCF` (`:381`), `#sideSave` (`:387`) stay enabled during `await`. Double-clicking Sync fires parallel `Promise.all(USERS.map(syncUser))` paged `user.status` loops (up to 10k-sub pages each, `:141–148`) — wasted CF API load and last-write-wins on `solved`. Fix per Top Fix #3: `btn.disabled = true` + `aria-busy` around the awaits.
- [WARNING] Sidebar has no `Escape` handling and no focus management (`:383–386` — only scrim-click and ✕ close). Keyboard users tab into a 290px overlay they can't dismiss without reaching ✕. Fix: `Escape` → `closeSide()`, move focus to `#menuBtn` on close.
- [Minor] `syncAll` failure message (`:160` `CF sync failed — check handles.`) doesn't say *which* handle failed when 2+ users sync via `Promise.all` — one bad handle fails the whole batch message. Fix: `Promise.allSettled` + per-user error naming the handle.

---

## Files Audited

- `index.html` (397 lines — entire frontend: CSS `:root` tokens + layout, sidebar/tabs/strip/charts/heatmap/table markup, vanilla JS: CF API sync, Firestore/localStorage store, canvas renderers, compare view)
- No other frontend assets exist in project root (verified: root contains only `index.html` + `.git/`; no `src/`, no `components.json`, no UI-SPEC.md)
- Server: `http://localhost:8137` → HTTP 200 (left running, untouched)
- Screenshots: none (Playwright browser binary absent; CLI errored `chrome-headless-shell` missing — no browser install attempted)
