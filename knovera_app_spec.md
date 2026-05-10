# Knovera Simulation — App Spec for AI Agent

**Project:** SDG 14 fishery simulation (Schaefer model + bio-economics)
**Hackathon deadline:** May 31, 2026
**Token budget:** $300 (Google AI Studio)
**Base file:** `SDG_fishing_formula_animated.html` (working — do not break)

---

## PROJECT CONTEXT (read first, every task)

This is a single-file HTML simulation that teaches sustainable fisheries management using the Gordon-Schaefer bio-economic model. Students manipulate fishing effort (E), price (P), cost (c), and discount rate (δ) to discover the tradeoff between short-term profit and long-term sustainability.

**What already works (DO NOT BREAK):**
- Schaefer population dynamics formula
- Bio-economic NPV calculation
- Live population chart with MSY target line
- Reflection modal at end of run
- LocalStorage run history table
- Six parameter sliders with live labels

**Tech stack — fixed, no additions:**
- Single HTML file, no build process
- Tailwind CSS via CDN (already loaded)
- Chart.js via CDN (already loaded)
- MathJax via CDN (already loaded)
- Vanilla JavaScript only

---

## GLOBAL CONSTRAINTS (apply to every task)

**DO:**
- Work in the existing `SDG_fishing_formula_animated.html` file only
- Preserve the Schaefer formula exactly as written: `growth_t = r * currentN * (1 - (currentN / K))` and `yield_t = q * E * currentN`
- Rescale all annual parameters to monthly on conversion: `r_monthly = r / 12`, `q_monthly = q / 12`, `SHIP_MAINT_monthly = SHIP_MAINT / 12`
- Display time as `Tahun X Bulan Y` derived from a single tick counter (e.g. tick 14 = Tahun 2 Bulan 2)
- Preserve the bio-economic formula exactly: `profit_t = P * yield_t - c * E` and `discounted_t = profit_t / Math.pow(1 + delta, currentYear)`
- Match existing Tailwind color palette and styling conventions
- Test in a browser before declaring a task done
- Report which lines/sections were touched and which were not

**DO NOT:**
- Add new npm packages, CDN libraries, or build tools
- Refactor working functions unless the task explicitly requires it
- Change any constants (`q = 0.00003`, `maxMonths = 60`, `tickSpeedMs = 400`) without explicit instruction
- Change the economic constants: `UMR_MONTHLY = 2500000` (Rp), `MGMT_CUT = 0.10`, `J_FISHERMEN = 50`
- Combine multiple tasks into one commit or session
- Touch the localStorage key name `tunaRunHistory` (data continuity matters)
- Generate placeholder lorem ipsum text — use specific Indonesian fishery context

---

## VISUAL DESIGN LANGUAGE

**Existing palette (reuse, do not invent new colors):**
- Primary blue: `#2563eb` (population, primary CTAs)
- Header navy: `#1e3a8a` (`bg-blue-900`)
- Sustainability green: `#10b981` (MSY line, healthy state)
- Warning amber: `#f59e0b` / `text-amber-500` (depleted state, profit metric)
- Crash red: `#dc2626` / `text-red-600` (effort slider, crash state)
- NPV purple: `#9333ea` (NPV line, NPV metric)
- Slate neutrals: `slate-50`, `slate-200`, `slate-600`, `slate-800`

**Typography:**
- System font stack via Tailwind default
- Numbers: `font-bold` with `tabular-nums` if displayed in tables
- Labels: `text-xs uppercase tracking-wider font-semibold`

**Boat sprites (PLACEHOLDER PRIMITIVES — assets will be replaced post-hackathon):**

The product owner will draw and swap in detailed SVG assets later. The agent's job is to build a *pluggable visual system* using simple geometric primitives. Do not attempt artistic representation.

- **Small boat placeholder**: Simple trapezoid hull (4-point polygon) with a thin vertical mast line. ViewBox: `0 0 40 25`. Hull color: `#92400e`. Mast color: `#1e3a8a`. The "waterline" anchor sits at y=20 within the viewBox.
- **Large boat placeholder**: Larger trapezoid hull with a rectangular cabin on top. ViewBox: `0 0 80 35`. Hull color: `#475569`. Cabin color: `#dc2626`. Waterline anchor at y=28.
- Both rendered as `<symbol>` definitions inside `<defs>`, instantiated via `<use href="#smallBoat" x="..." y="...">` for positioning.

**Fish placeholder:**
- Simple oval body with a triangular tail (use a `<path>` or composite of `<ellipse>` + `<polygon>`)
- ViewBox: `0 0 12 6`. Body color depends on population state (set via CSS class on the `<use>` element):
  - `.fish-healthy`: fill `#2563eb`
  - `.fish-pale`: fill `#93c5fd`
  - `.fish-sparse`: fill `#94a3b8`
- Density logic stays the same: ~20 fish at N > 0.7K, ~10 at 0.3K-0.7K, 2-3 at < 0.3K, none at 0.

**Ocean scene background:**
- Sky gradient top: `#dbeafe` → `#93c5fd`
- Water gradient: `#3b82f6` → `#1e40af`
- Subtle CSS wave animation (translateX, 6s infinite, no JS)
- Coastal silhouette on left edge (Indonesia coastline hint)

---

## TASK 1: Add Cumulative Profit & Fisher Welfare Metric Cards

**Why:** Currently NPV is shown in metrics but cumulative (raw) profit is only displayed in the run history table. The divergence between cumulative and NPV is the headline pedagogical lesson. The Fisher Welfare card translates aggregate profit into per-person income, making overfishing consequences human-scale for students.

**Token estimate:** Low (~2-3k tokens of code changes)
**Time estimate:** 45 minutes

### DO
- Add a fifth metric card to the existing 2-column grid in the left column (around line 117 in current file). Convert grid to `grid-cols-2` already exists — make it span properly with the new card.
- Card label: "Cumulative Profit"
- Card value source: existing `cumulativeProfit` variable
- Card color: pink-600 (`text-pink-600`) to differentiate from NPV's purple
- Subtitle: "Raw earnings (no discount)"
- Update the value inside `stepYear()` next to where `valNPV.textContent` is set (around line 386)
- Add `valCumProfit` to the DOM element list at top of script
- Reset to "Rp 0" in `resetSimulation()` next to other metric resets

### DO NOT
- Modify the cumulative profit calculation itself
- Reshape the grid layout in a way that breaks responsive behavior
- Add a new chart line — that's a separate future task

### FISHER WELFARE INDICATOR (add together with Cumulative Profit card)

Add a sixth metric card derived from `cumulativeProfit`. This represents the simplified port model — the human-scale consequence of the ecological simulation.

**Constants (define at top of script, do not hardcode inline):**
```js
const UMR_MONTHLY = 2500000;   // Rp — Upah Minimum Regional
const MGMT_CUT    = 0.10;      // 10% for management/owner (the user)
const J_FISHERMEN = 50;        // estimated fishing community size (hardcoded)
```

**Formula:**
```
mgmtIncome      = cumulativeProfit × MGMT_CUT
communityPool   = cumulativeProfit × (1 - MGMT_CUT)
perFisherAnnual = communityPool / J_FISHERMEN
perFisherMonthly = perFisherAnnual / (simulationYear × 12)
```

**Card spec:**
- Label: "Pendapatan Nelayan/Bulan"
- Value: `perFisherMonthly` formatted as `Rp X.XXX.XXX`
- Color: `text-green-600` if `perFisherMonthly ≥ UMR_MONTHLY`, else `text-red-600`
- Subtitle: "≥ UMR Rp 2.500.000 ✓" or "< UMR — kesejahteraan terancam ✗"
- Update inside `stepYear()` alongside other metric updates
- Reset to "Rp 0" in `resetSimulation()`

**Constraint:** If `cumulativeProfit ≤ 0`, display "Rp 0 — Defisit" in red. Do not show negative values.

### VERIFY (agent self-check)
- Run simulation at default values for 60 months (5 years)
- Cumulative profit should display a value greater than NPV
- Fisher Welfare card shows a value ≥ UMR in green at moderate effort
- Crank E to max — Fisher Welfare card turns red as fishery collapses
- Reset returns all cards to Rp 0
- No console errors

### TEAM CHECK (human verifies)
1. Open file in Chrome and Firefox
2. Click Play, let run to month 60
3. Confirm both NPV and Cumulative Profit cards show values
4. Confirm Cumulative Profit > NPV (because discounting reduces NPV)
5. Click Reset, confirm both return to Rp 0
6. Try with discount rate at 0% — they should match exactly
7. At default E, confirm Fisher Welfare card shows green value ≥ Rp 2.500.000
8. Set E to maximum, run to year 50 — confirm Fisher Welfare card turns red

### DONE WHEN
- All eight checks above pass
- The two profit metrics visibly diverge during a normal run
- Fisher Welfare card correctly reflects green/red based on UMR threshold
- No existing functionality is broken

---

## TASK 2: Build Visual Ocean Scene Above Chart

**Why:** The simulation is currently graphs-only. For a hackathon demo, judges need to see the *story* of overfishing visually — boats appearing, fish disappearing — not just a line going down. This is the single biggest demo impact change.

**Token estimate:** Medium (~5-8k tokens — reduced from original because placeholder primitives are simpler than detailed artwork)
**Time estimate:** 2-3 hours

### DO
- Insert a new section in the right column, **above** the existing "Live Population Tracker" card (around line 142)
- Container: `<section id="oceanScene">` with locked height of `200px`, full width of column
- Build inline SVG ocean scene with:
  - Sky gradient (top 30%)
  - Water gradient (bottom 70%)
  - Static Indonesian coastline silhouette on left edge (simple flat shape)
  - Layer for fish school (positioned middle-water depth)
  - Layer for boats (positioned at water surface)
- Three SVG `<symbol>` definitions inside `<defs>`, each as **simple geometric primitives only** (see Visual Design Language section). The product owner will replace these with detailed artwork post-hackathon — your job is the system, not the art.
  - `#smallBoat` — trapezoid hull + mast line (viewBox `0 0 40 25`)
  - `#largeBoat` — larger hull + cabin rectangle (viewBox `0 0 80 35`)
  - `#fish` — oval body + triangle tail (viewBox `0 0 12 6`)
- New JavaScript function `updateOceanScene()` called from inside `stepYear()` after population update
- Function logic:
  - Determine number of boats: derive from E value. Map: `effortToBoats(E)` returns `{small: int, large: int}`. Use simple thresholds (e.g., E < 10000 = 2 small/0 large, E < 25000 = 4 small/1 large, etc.)
  - Determine fish density: `populationToFishCount(N, K)` returns int between 0-20
  - Determine fish color state class based on N/K ratio
  - Position boats spread horizontally with small random offset
  - Render fish in a clustered school formation
- Wave animation: CSS-only via keyframes, applied to a translucent overlay rect

### ASSET REPLACEMENT CONTRACT (critical)

The product owner needs to swap in detailed artwork later without touching positioning logic. To make this possible:

- Each `<symbol>` must declare its `viewBox` exactly as specified above. Future asset replacements will respect the same viewBox dimensions.
- All positioning math must reference the symbol via `<use href="#id" x="..." y="...">`. Do NOT inline boat or fish shapes directly in the scene.
- The "waterline anchor" inside each boat symbol is fixed (y=20 for small, y=28 for large) — replacement assets will align to the same anchor.
- Color application for fish must be via CSS class on the `<use>` element, NOT hardcoded `fill` inside the symbol. This way replacement fish artwork can also accept dynamic coloring.
- Document the contract in a comment block above the `<defs>` section so the product owner knows what to preserve.

### DO NOT
- Use canvas — use inline SVG for crispness and ease of debugging
- Animate via requestAnimationFrame — keep it state-driven, only update when `stepYear()` runs
- Add any external image assets — everything inline
- Make the scene taller than 220px — must not push chart below fold

### VERIFY (agent self-check)
- Default state (year 0, full population, E=20000): healthy fish school visible, moderate boats
- Crank E to 50000 and play: more boats appear, fish school thins
- Let population crash to 0: scene shows empty water with boats
- Reset: scene returns to healthy state

### TEAM CHECK (human verifies)
1. Open file, ocean scene visible above chart on first load
2. Press Play with default E — scene shows reasonable boat count and fish density
3. Drag E slider to maximum during play — boats visibly increase
4. Watch population decline on chart while fish thin in scene (visual coherence)
5. Trigger collapse — scene shows empty water, no console errors
6. Reset — fish school returns to healthy
7. Resize browser window — scene scales without breaking
8. Test in Chrome, Firefox, Safari if possible

### DONE WHEN
- Scene loads on first paint
- Boat counts and fish density update each simulation tick
- No flicker, jank, or layout shift
- Existing chart still works
- Run history still works
- Reflection modal still works

---

## TASK 3: Add Scenario Challenge Mode

**Why:** Free-play simulation is good for exploration but weak for assessment. Pre-defined scenarios with target conditions force students to apply learning to specific problems. Three scenarios cover the core PISA competencies.

**Token estimate:** Medium (~5-7k tokens)
**Time estimate:** 2-3 hours

### DO
- Add a new card in the left column, below "Mission Controls" and above the metrics grid
- Card title: "Scenario Challenge"
- Dropdown selector with four options:
  1. **Free Play** (default) — current behavior, no constraints
  2. **Sustain MSY** — keep population within ±10% of K/2 for 60 months (5 years)
  3. **Recover the Fishery** — start with N = 0.15K, recover to N ≥ 0.4K by month 36 (year 3)
  4. **Fuel Crisis** — cost `c` is locked at Rp 150, find profitable strategy
- Each non-Free scenario:
  - Sets specific starting parameters when selected (override slider defaults)
  - Locks specific sliders relevant to the constraint (use `disabled` attribute)
  - Shows a small instruction banner at top of the chart card with the goal
  - At end of run, the reflection modal shows "Challenge Result: SUCCESS" or "FAILED" before the textarea
- Add scenario data to a JS constant `SCENARIOS = { ... }` containing `id`, `name`, `description`, `setup()`, `evaluate()`
- Save scenario id and result to run history along with reflection

### DO NOT
- Hardcode scenario logic inside `stepYear()` — keep it in the SCENARIOS object
- Add scenarios beyond these four
- Force the user to pick a scenario — Free Play remains default
- Block sliders that aren't part of the scenario constraint

### VERIFY (agent self-check)
- Switch between scenarios — slider locks update correctly
- Select "Recover the Fishery" — initial N is set to 0.15K
- Select "Fuel Crisis" — cost slider locked at 150
- Complete a run — reflection modal shows pass/fail status
- Run history shows scenario name in a new column

### TEAM CHECK (human verifies)
1. Default load shows "Free Play" — no constraints visible
2. Select "Sustain MSY" — instruction banner appears above chart
3. Select "Recover the Fishery" — population starts low, banner explains target
4. Select "Fuel Crisis" — cost slider greyed out at Rp 150
5. Complete each scenario at least once — confirm pass/fail logic works
6. Run history table now has a "Scenario" column showing which one was played

### DONE WHEN
- All four scenarios selectable and produce different starting states
- Reflection modal shows scenario result
- Run history logs scenario per run
- Switching scenarios mid-run resets the simulation (or warns first)

---

## TASK 4: Onboarding Tutorial Overlay

**Why:** A judge or first-time user needs to understand the simulation in under 60 seconds. A 4-step pointer-based tour eliminates confusion and improves demo quality.

**Token estimate:** Low-Medium (~3-5k tokens)
**Time estimate:** 1.5 hours

### DO
- On first page load (check localStorage flag `tunaTutorialSeen`), show modal with welcome text
- After dismissal, run a 4-step pointer tour highlighting:
  1. Population chart and MSY target line ("This green dashed line is your sustainability target")
  2. Fishing Effort slider ("This is your main lever — change it during play to see impact")
  3. Ocean scene ("Watch the boats and fish change as you adjust effort")
  4. Reflection modal preview ("After each run, you'll record what you learned")
- Each step: dimmed backdrop, spotlight on target element, tooltip with text and "Next" button
- Skip button always visible
- Set `tunaTutorialSeen = "true"` in localStorage on completion or skip
- Add a small "Restart Tutorial" link in the page footer

### DO NOT
- Use a third-party tour library — build minimally with vanilla JS and absolute positioning
- Block the simulation from running if tutorial isn't completed
- Auto-replay on every visit

### VERIFY (agent self-check)
- Clear localStorage and reload — tutorial appears
- Complete tutorial — does not appear on next reload
- Click "Restart Tutorial" — appears again

### TEAM CHECK (human verifies)
1. Open in incognito window — tutorial appears
2. Each step's spotlight aligns with the correct UI element
3. Tooltips are readable, no overflow off-screen
4. Skip button works at every step
5. After completion, refreshing does not re-trigger
6. Restart Tutorial link works

### DONE WHEN
- Tutorial appears once for new users
- All four spotlights target correct elements
- Skip and complete both set the flag
- Restart Tutorial works

---

## TASK 5: Demo Polish & Hackathon Branding

**Why:** Final pass. Hackathon judges form first impressions in 5 seconds. Strong header, clear branding, and a "demo mode" reset button matter.

**Token estimate:** Low (~2-3k tokens)
**Time estimate:** 1 hour

### DO
- Update header: keep "Knovera Time-lapse Simulator" but add tagline: "Belajar Pengelolaan Perikanan Berkelanjutan untuk SDG 14"
- Add team credit line in footer: configurable string at top of script (Firman fills in)
- Add "Demo Mode" button next to Reset that:
  - Clears run history
  - Resets tutorial flag
  - Sets parameters to interesting demo values (E = 35000 to show overfishing pattern fast)
  - Sets `tickSpeedMs` to 200 for faster demo (revert to 400 on Reset)
- Add favicon (inline SVG data URI, fish silhouette, blue)
- Add page meta description for SEO if hosted

### DO NOT
- Change the core color scheme
- Add Knovera logo image — keep text-only branding for now
- Add analytics, tracking, or external scripts

### VERIFY (agent self-check)
- Demo Mode button resets state correctly
- Tutorial reappears after Demo Mode click
- Tab title and favicon visible

### TEAM CHECK (human verifies)
1. Tab favicon visible
2. Header reads correctly with tagline
3. Demo Mode produces interesting first-30-second visual story
4. Footer shows team credit

### DONE WHEN
- Page presents professionally on first load
- Demo Mode produces a compelling 60-second judge experience
- All previous tasks still functional

---

## TASK EXECUTION ORDER (REQUIRED)

Execute strictly in numerical order. **Do not start Task N+1 until Task N passes both VERIFY and TEAM CHECK.**

1. Task 1 — Cumulative Profit Card (~1 day)
2. Task 2 — Ocean Scene (~3 days, biggest token spend)
3. Task 3 — Scenario Challenges (~2 days)
4. Task 4 — Tutorial Overlay (~1 day)
5. Task 5 — Demo Polish (~1 day)

Total estimated calendar time: 7-10 days of agent work + team review cycles.
Reserve final 5 days before May 31 for bug fixes, deployment, and pitch prep.

---

## TEAM REVIEW PROTOCOL

After the agent completes each task and submits work:

1. **Pull the file** to a clean folder. Do not overwrite the previous version — keep `task1_done.html`, `task2_done.html` etc. as version checkpoints.
2. **Open in browser** without any cache (Ctrl+Shift+R).
3. **Run the TEAM CHECK steps** for that task verbatim.
4. **Open browser console** (F12) — confirm zero errors during normal operation.
5. **Test the previous tasks** still work — regressions are common.
6. **If any check fails:** report back to the agent with: which step failed, what was expected, what actually happened, browser used.
7. **If all pass:** mark task complete in shared tracker, proceed to next task.

**Red flags that mean stop and review:**
- Agent reports "I also refactored X for cleaner code" when X was not in the task
- File line count grows by more than 2x what the task should require
- Agent adds new CDN scripts or imports
- Existing simulation behavior changes (different population curves, different NPV values for same inputs)

---

## DEPLOYMENT NOTES

- This is a single HTML file. To deploy: upload to any static host (GitHub Pages, Netlify drop, Cloudflare Pages).
- For hackathon submission: ensure file works opened directly via `file://` protocol as well as via HTTPS.
- Test on a fresh device and browser before final submission.
- Localization to Bahasa Indonesia is a post-hackathon task unless otherwise specified.

---

## OPEN QUESTIONS FOR PRODUCT OWNER (Firman)

Mark these resolved before agent starts:

1. Are the four scenarios in Task 3 the right ones, or should one be replaced?
2. Should tutorial text be in English, Bahasa Indonesia, or bilingual?

**Resolved:**
- ✅ Boat/fish visuals: agent builds placeholder geometric primitives. Product owner will replace with hand-drawn artwork post-hackathon following the Asset Replacement Contract in Task 2.
- ✅ Simulation length: 5 years (60 monthly ticks). Each tick = 1 month. Annual parameters rescaled to monthly.
