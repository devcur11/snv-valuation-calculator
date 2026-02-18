# Val Calc — Handover Doc for Next Session

## File
`/Users/devonriche/Desktop/Val Calc/index.html` — single-file React app (~760 lines)

## Context
StartUpNV valuation calculator. Two modes: "forward" (investor evaluating deal) and "reverse" (founder planning terms). Goal is to hand this file off to someone to replace a page on their website. Keeping it as a portable single HTML file with CDN dependencies (React, Babel, Tailwind, Google Fonts).

---

## COMPLETED WORK

### Pass 1: UX/Visual Audit (16 changes) — ALL DONE
- Output panel gradient + shadow differentiation
- Hero metric bumped to text-4xl (NOTE: Finding 14 of Pass 2 says to reduce to text-3xl — this is NOT yet done)
- Hero label weight/color bump
- Input touch targets py-2 → py-2.5 (all 4 input components)
- M/B toggle buttons enlarged
- Responsive grid breakpoints (grid-cols-1 sm:grid-cols-2 lg:grid-cols-3)
- Label contrast slate-500 → slate-600 (WCAG AA)
- Waterfall text sizes bumped
- Amber badges → yellow for differentiation (Badge + realityBg/realityDot/realityText)
- Dilution loss bars opacity 40% → 70%
- Dilution section spacing mb-1 → mb-3
- Leader lines slate-200 → slate-100 (NOTE: Finding 13 of Pass 2 says to revert to slate-200 — this IS done in MetricRow)
- Sticky output panel (lg:sticky lg:top-8 lg:self-start)
- Mode toggle max-w-md → max-w-lg
- Active states on buttons (scale-[0.97], scale-[0.98])
- Collapsible header hover state
- Hero metric transition-all duration-300

### Pass 2: Math Audit — ALL DONE
- All formulas verified correct against VC standards
- Forward/reverse are true algebraic inverses
- All edge cases guarded
- Added >100% ownership warning (line ~580-582)

### Pass 3: Output Panel Audit (16 findings) — PARTIALLY DONE

#### COMPLETED (2 of 16):
1. **getReality function restructured** (Findings 7, 8, 15)
   - Now returns `{ worst, summary, exitA, samA, exitVal, salesVal, hint }` instead of `{ worst, lines, exitA, samA }`
   - `summary` is a neutral statement (not a question)
   - `hint` is populated for orange/red severity levels
   - **CRITICAL: The Reality Check card rendering (lines ~723-742) still uses the OLD `reality.lines` API. This will break. Must update the card to use new structured data.**

2. **MetricRow component updated** (Findings 13, 14)
   - Added `indent` prop (adds `pl-3 border-l border-slate-200`)
   - Added `large` prop (switches value from `text-sm font-semibold` to `text-base font-bold`)
   - Leader line now has `min-w-[2rem]` and uses `border-slate-200`

#### REMAINING (14 of 16):

**Finding 1 — Fix exit badge in reverse mode** (HIGH)
- Problem: In reverse mode, the badge next to "Maximum Valuation" hero rates the TARGET EXIT (input), not the valuation (output). Semantically wrong.
- Fix: Remove exit badge from reverse hero. Add secondary line: "For a [fmtResult(targetExit)] exit" with the exit badge attached there.
- Location: Lines ~668-681 (hero metric section)

**Finding 2 — Replace "Expected Dilution" with "Initial Ownership"** (HIGH)
- Problem: "Expected Dilution" is triple-redundant (Ownership at Exit row + dilution waterfall on left)
- Fix: Replace with "Initial Ownership" row showing `forward?.ownership` (forward) or `reverse?.impliedOwnership` (reverse)
- The `initialOwnership` useMemo at line ~492 already computes this value
- Location: Lines 692 and 709

**Finding 3 — Mobile sticky summary bar** (HIGH)
- Problem: On mobile, results are 3-4 scrolls below inputs. No feedback when changing inputs.
- Fix: Add `fixed bottom-0 left-0 right-0 lg:hidden z-50` bar showing hero metric + reality dot. Uses IntersectionObserver on the results card.
- Needs: `resultsRef` (useRef), `showMobileBar` (useState), IntersectionObserver in useEffect
- Location: New code in main component + new JSX at end of return

**Finding 4 — Reorder metric rows** (HIGH)
- New order: Initial Ownership → Ownership at Exit → Required Return → Revenue at Exit → [subheading] → % of TAM → % of SAM
- Location: Lines ~684-720

**Finding 5 — Rename "Req. Annual Sales" → "Revenue at Exit"** (HIGH)
- Also add tooltip in BOTH modes: "Annual revenue needed at exit, based on the industry sales multiple."
- Location: Lines 698, 715

**Finding 6 — Visually group TAM/SAM** (MEDIUM)
- Add `indent={true}` to TAM/SAM MetricRows
- Add tiny subheading between Revenue at Exit and TAM: `text-[10px] uppercase tracking-wider text-slate-400 pt-1` reading "Market Share Required"
- Location: Between revenue row and TAM row in both mode branches

**Finding 8 — Structured Reality Check card** (MEDIUM)
- Replace prose rendering with two structured assessment rows:
  - Row 1: "Exit Difficulty" | fmtResult(reality.exitVal) | Badge(reality.exitA)
  - Row 2: "Market Capture" | fmtPct(samPct) + " of SAM" | Badge(reality.samA)
- Then one summary sentence using `reality.summary`
- Then hint using `reality.hint` (only for orange/red)
- **CRITICAL: The card currently renders `reality.lines.map(...)` which no longer exists. This MUST be updated or the app will crash.**
- Need to get `samPct` in the card — compute from forward/reverse data or add to getReality return
- Location: Lines ~723-742

**Finding 9 — Rename reverse hero label** (MEDIUM)
- Change "Maximum Valuation" → "Max Valuation Cap"
- Location: Line ~671

**Finding 10 — Add TAM/SAM tooltips** (MEDIUM)
- % of TAM: "Share of Total Addressable Market the company must capture in annual revenue. Lower is more achievable."
- % of SAM: "Share of Serviceable Addressable Market required. Below 2% is generally achievable."
- Location: Lines 699-700, 716-717

**Finding 11 — Reverse mode label changes** (MEDIUM)
- Rename "Ownership at Exit" → "Required Ownership" in reverse mode
- Change tooltip to: "The ownership stake the investor needs at exit to hit their target return."
- Add subtitle under "Your Optimal Terms": "Based on a [fmtResult(targetExit)] target exit"
- Location: Lines ~664-666, 704-708

**Finding 12 — Rename section title** (LOW)
- "What Investors Need" / "Your Optimal Terms" → "Deal Analysis" (both modes)
- Location: Line ~665

**Finding 14 — Three-tier type scale** (LOW)
- Reduce hero from text-4xl → text-3xl
- Top 3 metric rows get `large={true}` (Initial Ownership, Ownership at Exit, Required Return)
- TAM/SAM rows stay text-sm (default)
- Location: Line ~674 (hero), and MetricRow props throughout

**Finding 16 — Reverse tooltip parity** (LOW)
- Covered by Finding 5 (Revenue at Exit tooltip) and Finding 10 (TAM/SAM tooltips)

---

## CRITICAL WARNING

**The app is currently in a BROKEN state.** The `getReality` function was updated to return `{ summary, hint }` instead of `{ lines }`, but the Reality Check card rendering (lines ~723-742) still references `reality.lines`. This will cause a runtime error.

**First thing to do in next session:** Fix the Reality Check card rendering to use the new structured data format. See Finding 8 above for the target design.

---

## Architecture Notes
- All state is in `ValuationCalculator` component via `useState(DEFAULTS)`
- `set(key)` is a curried setter helper
- `calcForward`, `calcReverse`, `calcDilution`, `getReality` are pure functions
- `initialOwnership` is already computed via useMemo (line ~492) and available for the new "Initial Ownership" row
- Components: Tip, SectionCard, CurrencyInput, PctInput, NumInput, MarketInput, Badge, MetricRow, DilutionWaterfall
- MetricRow now supports `indent` and `large` props (just added)

## Plan File
Full audit details at `/Users/devonriche/.claude/plans/encapsulated-riding-clover.md`
