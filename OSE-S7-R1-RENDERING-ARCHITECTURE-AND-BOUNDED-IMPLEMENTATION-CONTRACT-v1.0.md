# OSE S7 — R1 RENDERING ARCHITECTURE AND BOUNDED IMPLEMENTATION CONTRACT v1.0

**Mission:** OSE-S7-CLAUDE-R1-RENDERING-ARCHITECTURE-01 · **Executor:** Claude, Development Owner · **Date:** 2026-09-08
**Scope:** read-only diagnosis + contract. No Pine edited, no compile/run, no reviewer launched, no SOURCE ACCEPTED.

## 1. Custody

`OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3R3.pine` — recomputed locally from actual bytes:
SHA-256 `d29151b6d08a9cca9d510ed73f2d9f9c9df35d126d16cfd2e95d2c59fa15118f` **MATCH** · 421,426 bytes **MATCH** ·
7,028 LF lines **MATCH** · shorttitle `OSE-S7` **MATCH**. Evidence: AAPL 1917×830, TSLA 1917×868, both inspected.
No inaccessible input. Nothing further requested.

## 2. Evidence ↔ claim reconciliation

**Same-price invariant — VIOLATED, measured.** TSLA price axis calibrated from its own labels (34.0 px per 25.00,
linear; predicted y for the 363.79 tag matches the rendered tag). Source-owned levels vs. rendered position:

| Object | Source value | Rendered price | Δ |
|---|---|---|---|
| ORH · SUPPORT / OPENING RANGE HIGH | 362.42 | ≈ 322.8 | −39.6 |
| VWAP · NEUTRAL REFERENCE | 360.42 | ≈ 290.7 | −69.7 |
| ORL · SUPPORT / OPENING RANGE LOW | 355.72 | ≈ 217.1 | −138.6 |
| PDH · RESISTANCE (box) | ≈ 364.7 (inferred) | ≈ 359.4 | −5.3 |

This is **not** a constant offset and **not** a constant ratio. A least-squares affine fit gives
`rendered ≈ 15.78 × source − 5396`, residual ≤ 0.5 on a 105-point span, and it also predicts the PDH box. Unity
occurs at source ≈ 365.1 — essentially the current price. AAPL shows the same shape with a **different slope**
(≈1.9): markers sit 7–14 points above bars whose high is ≈331.5.

**Interpretation.** A tight affine map with slope ≠ 1, unity at the current price, and a slope that differs per
chart is the signature of drawings being resolved against a **price scale that is not the candles' scale**, and one
that is auto-fitted to the drawings' own extent (≈33 price units on TSLA vs. the main scale's ≈525). It is *not*
the signature of invalid numeric y values — the authority's caution is upheld: the coordinates are internally
consistent, they are being interpreted in the wrong space.

**R-01 reconciliation — closure correctly NOT accepted.** Event markers use `x = time` with
`yloc.abovebar/belowbar`; `yloc.abovebar` is *defined* as "just above this bar's high" and therefore cannot land
14 points off — unless resolved on a foreign scale. R-01 changed y-coordinate *semantics*; a scale-assignment
fault is invisible to that change. This is why an isolated yloc patch neither closed nor could close it.

**R-02.** Session-boundary deletion confirmed (clears `s7_hist`, setup/watch labels, `s7_evtMarks`). It destroys
history rather than retaining it, so it does not meet bounded historical inspection. **Live-state reset and
historical rendering retention must become two lifecycles** — see §5/§6. Prior-session cases never stay active.

**Support rendering.** TSLA renders `ORH · SUPPORT` and `ORL · SUPPORT` in green, so "no support is rendered" is
**unsupported**. Whether *every* valid support renders at correct price/time/role is **NOT YET MEASURED**. AAPL's
registry is not inferable from its still; its box texts are clipped at the plot edge and carry no numerals.

## 3. Root-cause matrix

| # | Finding | Certainty | Basis |
|---|---|---|---|
| **C-1** | Chart objects resolved against a non-candle price scale (affine, auto-fitted, per-chart slope) | **HYPOTHESIS — highest ranked** | §2 measurements; both stills show TradingView's **`A` / `L` scale selector**, so ≥2 scales exist on the pane |
| **C-2** | Close-time used as an `xloc.bar_time` x-coordinate → object anchors one bar right of origin | **PROVEN (source)** | `s7_caseStartTime := time_close` (2×) for INITIAL STOP / TRAIL STOP / TP1 lines; WATCH label `x = time_close`; ADVANCED event label `x = time_close`; `s7_sessionRight := rthTimeClose`. On a contiguous intraday series a bar's close time equals the next bar's open time |
| **C-3** | Structure confined to a current-session rectangle; no historical extent possible | **PROVEN (source)** | every `box.new`/`line.new` uses `left/x1 = s7_sessionOpen`, `right/x2 = s7_sessionRight`, re-pointed each confirmed bar by `box.set_left/right`, `line.set_xy` (14 `line.set_xy`, 12 `box.set_*`) |
| **C-4** | No presentation archive exists; every object is a mutated live singleton | **PROVEN (source)** | fixed `var` handles reused via setters; three deletion clusters (session reset, UI-off, depth switch) and no retention path |
| **C-5** | Large text islands / clipped labels degrade readability | **PROVEN (evidence)** | AAPL box texts clipped at the plot edge; marker text blocks overlap |
| **C-6** | Exports stretch the price scale | **FALSIFIED** | all 52 `plot()` declare `display = display.data_window`; 0 declare `force_overlay`/`scale` — they never touch the scale |
| **C-7** | Multi-instance contamination | **CLOSED by PO** | not reopened |

C-2 is genuinely independent of C-1: it displaces objects **horizontally** by one bar and survives any scale fix.

## 4. Three-layer producer / coordinate / lifecycle map

| Layer | Objects | Producer | Coordinate | Lifecycle today | Required |
|---|---|---|---|---|---|
| 1 Price structure | zone bands (boxes), VWAP, ORH/ORL, NEXT BARRIER, INITIAL STOP, TRAIL STOP, TP1 | `zoneReg` `Zone.lower/upper/structuralRole`; `vwap`, `orHigh`, `orLow` (`request.security` bundle); `s7_barrier`, `s7_caseInitialStop`, `s7_trailStop`, `s7_caseTp1` | `xloc.bar_time` + `yloc.price` | live singletons, session-bounded, mutated per bar | true levels as horizontal lines, true ranges as bands between exact bounds (**already correct — no midpoint is fabricated**); live references extended over a useful horizon; historical segments terminated at recorded validity |
| 2 Candle events | WATCH, SETUP/REJECTED, management event markers | `armStore`; `rawObsSet` (`Obs.obsBarTime`, `signalClose`, `riskPass`); `s7_evtText/s7_evtPri` | `xloc.bar_time`; markers bar-relative, SETUP at frozen `signalClose` | FIFO caps 8 / 24 / 12, wiped at session boundary | frozen presentation record per event; event time kept separate from close-time text; compact shared marker + count on shared bars; archive separate from live |
| 3 Screen information | card, Evidence+SQUEEZE, SIGNAL COMPARISON, ADVANCED panel | display strings | `table` — screen-fixed, **scale-independent** | correct | unchanged in R1 |

Layer 3 is unaffected by C-1: tables are screen-anchored, which is consistent with the evidence — the cards render
correctly in both stills while chart-bound objects do not. This is corroborating evidence for C-1.

## 5. Structural provenance audit

| Field | Status |
|---|---|
| identity/mask, source lower/upper, role, eligibility gate, object identity | readable in source (`Zone.mask`, `.lower`, `.upper`, `.structuralRole`, `s7_staticMasks`) |
| role migration timing | **NOT YET MEASURED** — no recorded transition timestamp exists |
| rendered coordinates/colour | measured for 4 TSLA objects (§2); AAPL not measurable from its still |
| deletion reason | inferable from gate, not recorded on the object |
| **runtime registry values (AAPL & TSLA)** | **NOT YET MEASURED — not PASS** |

Colour is driven solely by `structuralRole` (`ROLE_SUPPORT`→green, `ROLE_RESISTANCE`→red, else gray); no
PDH/PDL-name or close-relative shortcut exists in source. **Bounded read-only inspection method for the
checkpoint:** the existing screen-fixed ADVANCED panel already renders zone identity, confluence and role — add a
temporary read-only provenance row group listing, per registry entry, `mask · lower · upper · role · rendered-y`.
Screen-fixed, no new producer, no export, no CSV. Any diagnostic build is a **separate** artifact and must never
be the candidate.

## 6. Historical retention budget — exact shortfall

Declaration (unchanged): `max_lines_count=16`, `max_labels_count=64`, `max_boxes_count=16`, `max_bars_back=300`.

**Live worst case, proven from source:** labels `7 price + 24 SETUP + 8 WATCH + 12 events = 51`; lines `7`; boxes
`4 zones + VWAP + OR = 6`. Headroom: **labels 13, lines 9, boxes 10.** Eviction precedes creation in both FIFO
paths, so no `+1` transient there; the zone loop creates before deleting, so reserve **+1 box**.

**Target = current + 2 completed RTH sessions.** Naïve replication of the live budget per session:

| Class | Needed | Cap | Result |
|---|---|---|---|
| Labels | 51 live + 2 × (12 events + 24 SETUP) = **123** | 64 | **SHORTFALL 59** |
| Boxes | 6 live + 2 × 4 = 14 (+1) = **15** | 16 | fits, zero margin for role-change segmentation |
| Lines | 7 | 16 | fits |

**Labels are the binding constraint. Do not silently reduce history or raise caps** — Product Owner decision, three
options, each within the existing caps except (d):

- **(a)** Archive management events only (drop archived SETUP): `51 + 2×6 = 63/64`. Cost: **6 archived events per session.**
- **(b)** Reduce live `S7_SETUP_LABEL_CAP` 24→12: `39 + 2×12 = 63/64`. Cost: halved live SETUP density; **12 archived events per session.**
- **(c)** One-per-bar compact merged marker (count + truthful detail) reduces live events materially where events cluster; measured benefit requires runtime counts — **NOT YET MEASURED**, so it cannot be budgeted as proven headroom.
- **(d)** Raise `max_labels_count` — a declaration change requiring explicit authorization; not assumed.

**`max_bars_back = 300` constrains the target by timeframe:** 3 RTH sessions = 234 bars at 5-minute (fits) but
≈1,170 at 1-minute (**does not fit**). The archive target is therefore only defensible at ≥5-minute unless the
declaration changes.

## 7. Collision and scale contract

- Never fabricate ATR/percentage offsets or distant price lanes to avoid collision; never silently discard a
  meaningful event. Collisions are resolved by **merging co-bar events into one compact marker with a count**,
  preserving existing priority and chronology.
- No special operator/scale setting may be used to conceal invalid coordinates. If C-1 proves to be a chart-side
  scale assignment, the correct remedy is the scale assignment — **not** compensating arithmetic in source, which
  would bake a viewer-specific transform into the analytical build.
- Structure spans horizontally; ranges stay bands between exact bounds; no midpoint collapse (already true).
- No backward extension may imply a level was known before it existed; any current-reference overlay must be
  declared as such.

## 8. Proposed source allowlist (for Main approval — nothing applied)

1. **A-1 (C-2, proven):** replace close-time x-coordinates with the originating bar's open time — `s7_caseStartTime`
   (2 sites), WATCH label, ADVANCED event label. No producer/predicate change.
2. **A-2 (C-3/C-4):** presentation-record type + archive arrays with deterministic eviction, role-change
   segmentation, duplicate handling, retention disclosure, and "incomplete" labelling; strictly no consumer in
   analytical state, case election, Risk, alerts, queue or exports.
3. **A-3:** split session reset into *live reset* (unchanged) + *archive promotion*; explicit rules for calendar
   advance, UI OFF, depth change, reload, symbol change.
4. **A-4 (C-5):** compact merged co-bar marker.
5. **A-5:** extend live structural references across the approved horizon; terminate historical segments at
   recorded validity.
6. **Exceptions requiring named authorization:** any declaration-cap change (§6d); any input display-metadata
   change (status-line cleanup is **R2**; inherited input declarations are inside the frozen boundary).

Preserved throughout: Stage-6 source, producer semantics, 52 exports/title/order, three `request.*`, three Stage-7
inputs, management and alert behaviour. No new analytical calculation, score, confidence, institutional inference
or broker execution. No R2 and no ADVANCED work.

## 9. Targeted checks

**Static, already complete:** C-2, C-3, C-4, C-6 above.
**Runtime — smallest distinguishing observation for C-1:** one still of the OSE indicator's **scale assignment** —
right-click the indicator → *Pin to scale*, with the price-scale menu visible so the `A`/`L` scales present in both
stills are identifiable. This single artifact discriminates decisively:

- pinned to a non-candle scale → **chart-side**, no source geometry change is warranted;
- pinned to the same scale as the candles → **source-side**, and the coordinate/scale ownership model must change.

Committing to either remedy before this observation would be guessing, and the two remedies are mutually exclusive.
Everything else in §8 is unaffected by the answer and can be authorized in parallel.

Remaining checkpoint outcomes (origin-bar tracking through pan/zoom, source-matched support/resistance, bounded
retention without live contamination, readable footprint) stay reserved for the separately authorized runtime
checkpoint. **Source-only proof cannot establish them, and none is claimed here.**

## 10. Preserved R2 requirements

Right Information Rail as readable, opaque, multi-card — not one monolithic spreadsheet. Preserve Entry Efficiency
categories, Signal Comparison, frozen Q1–Q5 Evidence, separate SQUEEZE, Movement/Participation, and ADVANCED-only
ADX / Efficiency Ratio / ATR% / ATR Ratio diagnostics. Modules need not map one-to-one to Pine tables; no arbitrary
pixel docking or viewport reservation is promised. Status-line cleanup is R2.

## 11. Verdict

```
HOLD — SPECIFIC RUNTIME OBSERVATION / AUTHORITY DECISION REQUIRED
```

**Smallest missing items — two, both cheap:**

1. **One still** of the OSE indicator's scale assignment (§9). Without it the top-ranked root cause C-1 stays a
   hypothesis and its remedy is unknowable.
2. **One Product Owner decision** on the §6 retention option (a) / (b) / (c) / (d), because the label cap is short
   by 59 and I will not silently cut history or raise a cap.

A-1 is proven and could be authorized immediately if Main prefers to land it ahead of the C-1 decision. Returning
to Main. Stopping before implementation.
