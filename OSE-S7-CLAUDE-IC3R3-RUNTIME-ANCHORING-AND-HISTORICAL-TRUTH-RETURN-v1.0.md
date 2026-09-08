# OSE — STAGE 7 / MAIN CONTROL ROOM
## CLAUDE IC3R3 — RUNTIME-ANCHORING AND HISTORICAL-TRUTH CORRECTION RETURN v1.0

**Mission:** OSE-S7-CLAUDE-IC3R3-RUNTIME-ANCHORING-AND-HISTORICAL-TRUTH-CORRECTION-01
**Role:** Claude — Development Owner
**Date:** 2026-09-08

---

## 0. CUSTODY GATE — LIFTED

The prior `INPUT CUSTODY HOLD` is lifted. All four required properties recomputed from the
attached file's actual bytes:

| Property | Required | Recomputed | Result |
|---|---|---|---|
| SHA-256 | `05e7c863090b121a5b60c73ab79dcdec580ddaf50ebc16976f9070d0605547b6` | identical | **MATCH** |
| Bytes | 417,399 | 417,399 | **MATCH** |
| LF lines | 6,972 | 6,972 | **MATCH** |
| Declared shorttitle | `OSE-S7` | `OSE-S7` | **MATCH** |

`awk END{NR}` 6,972 confirms termination with a newline; zero CR bytes.

**The shorttitle also closes the evidence gap I raised while holding.** The runtime
screenshot's chart legend renders the bold shorttitle `OSE-S7`, which matches IC3R2 and matches
neither IC3 (`OSE-S7 UI IC3`) nor IC3R1 (`OSE-S7 UI IC3R1`). The observed behaviour is therefore
bound to this exact parent, and the correction is being made against the source that produced it.

IC3R2 is the sole writable parent. IC3R1, IC3 and IC2 were read for lineage only and are all
byte-identical after this run.

### Parent delta actually inherited

IC3R2 = IC3R1 + two compile-load fixes only: shorttitle shortened to ≤10 characters, and the
SIMPLE/ADVANCED table-render bodies extracted into `s7_renderSimple` / `s7_renderAdvanced` to
reduce main-body compilation size. Visual content, producers and lifecycle are unchanged, so the
accepted IC3R1 behaviour carries forward intact.

---

## 1. SCOPE DERIVATION — STATED PLAINLY

The A50-class mission packet defining findings, a correction allowlist and a verdict set **was
never delivered to this session**; only the mission ID arrived. Rather than hold a second time on
a paperwork gap when both substantive inputs are in custody, scope was derived from:

1. the mission ID's own two named classes — **runtime anchoring** and **historical truth**;
2. the controlling runtime screenshot;
3. the standing A48R §4 prohibitions and the Stage-7 product firewall, which remain in force.

**Exactly two defect classes were corrected, one per named class.** If your packet enumerates
findings beyond these two, this return under-covers it and I will extend on receipt — but nothing
here exceeds the standing firewall, and no finding was invented to fill the gap.

---

## 2. FINDINGS FROM THE RUNTIME EVIDENCE

### R-01 — RUNTIME ANCHORING: bar-relative markers detach from their bars

**Observed.** Calibrating the price axis from its own labels (2.00 price = 58 px; 346.00 at
y=135) and measuring the marker anchors against the candles at the same x:

| Marker | Anchor price | Bar high at that x | Detachment |
|---|---|---|---|
| `STRUCTURE LOST · 11:00` | ≈ 336.2 | ≈ 330.8 | ≈ +5.4 |
| `ACTIVE · LONG · LRT +1 10:50` | ≈ 335.4 | ≈ 330.8 | ≈ +4.6 |

Both sit above **every price the instrument traded in the visible window** (chart maximum
≈ 330.7). The detachment is also *inconsistent*: in the later region `STRUCTURE LOST 10:05` and
`ACTIVE · SHORT · LBO 09:50` sit on their bars, `TRAIL ON +3 10:10` sits correctly below its bar,
while `DIRECTION WARNING 15:30` and `WATCH · VWR · SHORT` sit below bars they should sit on or
above.

**Cause in source.** The bar-relative SIMPLE markers were anchored by price:

```
label.new(x = time, y = s7_above ? rthH : rthL, …, yloc = yloc.price, …)   // event markers
label.new(x = time_close, y = s7_simple ? rthL : rthC, …, yloc = yloc.price, …)   // WATCH
```

`rthH` / `rthL` / `rthC` are not the chart's own bars. They come from
`request.security(rthTicker, timeframe.period, coreBundle(), gaps = barmerge.gaps_on)` where
`rthTicker = ticker.new(prefix, ticker, session.regular, adjustment.splits)` — a separately
requested, RTH-sessioned, split-adjusted series. Its price basis and its `gaps_on` alignment need
not agree bar-for-bar with the series TradingView renders. A marker positioned by that price
therefore lands wherever the two series disagree.

**I could not isolate which of the candidate mechanisms (adjustment basis vs. gap alignment)
dominates, and I am not going to assert one from a screenshot.** The correction below is chosen
precisely because it is robust to all of them: it removes the dependency rather than compensating
for it.

**Style mapping was verified correct and is unchanged.** `TRAIL ON` renders below its bar and
`STRUCTURE LOST` / `STOP BREAK` above theirs, confirming the IC3R1 placement fix
(`s7_above = not (domPri == TRAILON or domPri == ACTIVE)`) behaves as intended.

### R-02 — HISTORICAL TRUTH: event markers survive the RTH session boundary

**Observed.** The screenshot shows management event markers from **three different sessions**
coexisting at full live salience — `ACTIVE · LONG · LRT 10:50` / `STRUCTURE LOST 11:00` /
`STOP BREAK · CASE ENDED 13:25` from one session, `ACTIVE · SHORT · LBO 09:50` /
`STRUCTURE LOST 10:05` / `TRAIL ON 10:10` from another, and `DIRECTION WARNING 15:30` /
`WATCH · VWR · SHORT` in the current one.

**Cause in source, proven.** The S7.5.0 RTH session-boundary reset clears the history ring, the
static zone boxes, the SETUP labels, the WATCH labels and the WATCH keys — but **not**
`s7_evtMarks` / `s7_evtMarkKeys`. Before this fix those were the *only* Stage-7 drawing lifecycle
that survived a session boundary; their sole clearing sites were the UI-disabled teardown and the
depth switch.

Two consequences, both visible:

1. markers from ended prior sessions keep full live salience over a new session's geometry, while
   the SETUP context that produced them has already been erased beside them — Final UX §35 scopes
   the case to the RTH session and §16 requires historical cases to be de-emphasised;
2. the bounded FIFO (`S7_EVENT_MARK_CAP = 12`) is occupied by stale events, so the current
   session can display fewer of its own than its budget allows.

---

## 3. CORRECTIONS APPLIED

### R-01 — chart-native bar-relative anchoring

```
// event markers
yloc = s7_above ? yloc.abovebar : yloc.belowbar      (y passed as na — ignored under these yloc)
// WATCH, SIMPLE only
yloc = s7_simple ? yloc.belowbar : yloc.price        (y left as rthC; ignored in SIMPLE)
```

TradingView resolves the vertical position from the rendered bar itself, so the mismatch class
cannot recur regardless of its cause. The `rthH` / `rthL` readiness gate on `s7_markable` is kept
byte-for-byte, so **marker eligibility is unchanged** — only placement is.

**Deliberately not changed**, with reasons:

| Anchor | Kept at `yloc.price` because |
|---|---|
| SETUP marker at `s7_o.signalClose` | the frozen signal close is the analytic fact the marker exists to mark; bar-relative anchoring would destroy that meaning |
| INITIAL STOP, TRAIL STOP, TP1, NEXT BARRIER | each pairs with a line at the same price and must stay on it |
| VWAP, opening range high/low | same — label and line are one object |
| ADVANCED current-event label at `rthC` | ADVANCED is preserved verbatim |

### R-02 — session-boundary clearing

`s7_clearLabels(s7_evtMarks)` and `array.clear(s7_evtMarkKeys)` added to the S7.5.0 reset,
immediately beside the SETUP/WATCH label clears already there. This applies the convention the
reset already establishes rather than inventing a policy. Canonical chronology remains owned by
the history ring and by ADVANCED; **no event production, alert, history custody or case rule is
touched.**

---

## 4. CHANGED-HUNK LEDGER — 7 hunks, all classified

| # | Site | Class | Change |
|---|---|---|---|
| 1 | `indicator()` line 395 | identity | title IC3R2→IC3R3. Shorttitle stays `OSE-S7` (IC3R2 C-05 ≤10-char compile constraint). Caps unchanged |
| 2 | Stage-7 firewall header | identity | IC3R3 correction manifest, comment only |
| 3 | `S7_BUILD` | identity | build string IC3R2→IC3R3 |
| 4 | S7.5.0 session reset | **R-02** | clear `s7_evtMarks` / `s7_evtMarkKeys` |
| 5 | WATCH marker `label.new` | **R-01** | `yloc.belowbar` in SIMPLE; ADVANCED unchanged |
| 6 | Event marker `label.new` | **R-01** | `yloc.abovebar` / `yloc.belowbar` |
| 7 | END marker | identity | IC3R2→IC3R3 |

No hunk touches anything outside the two named classes plus identity.

---

## 5. VERIFICATION

| Check | Result |
|---|---|
| IC3R2 parent custody (4 properties) | **PASS** |
| Stage-6 inherited body, lines 1–5021 excl. declaration identity | **PASS** — byte-identical vs IC2 |
| 52 exports, identical title and order | **PASS** |
| `request.*` — executable lines | **PASS** — 3 calls identical to IC2 |
| Stage-7 inputs exactly three | **PASS** — 43 total, unchanged |
| `alertcondition` 0 / `alert()` 1, message construction | **PASS** — unchanged |
| Declaration caps | **PASS** — 300 / 16 / 64 / 16, unchanged |
| Construct counts | **PASS** — `table.new` 4, `table.cell` 48, `label.new` 11, `line.new` 7, `box.new` 3 — all identical to IC3R2 |
| New assignment targets / new `var` state | **PASS** — none |
| ADVANCED panel rows | **PASS** — all 39 `s7_pushRow` identical |
| Structural lint | **PASS** — 0 empty block bodies; the only 5 non-multiple-of-4 indents are the pre-existing Stage-6 tuple continuations at lines 612–616 |
| A47 BI-6 / BI-7 / BI-8, FD-1…FD-5, A49 C-01…C-04 | **PASS** — untouched, not re-raised |

Resource bounds are unchanged from accepted IC3R1/IC3R2 because no drawing object was added:
SIMPLE worst case 7 lines / 51 labels / 4 boxes, ADVANCED 3 / 33 / 6, all inside caps.

**Static Pine-v6 review only. Compiler acceptance is not claimed.** Note that `yloc.abovebar` /
`yloc.belowbar` ignore the `y` argument — this is the documented behaviour these corrections rely
on, and it is the one point a compile/load gate should confirm first.

---

## 6. EXACT SOURCE IDENTITY

| | SHA-256 | Bytes | LF lines |
|---|---|---|---|
| Parent IC3R2 | `05e7c863090b121a5b60c73ab79dcdec580ddaf50ebc16976f9070d0605547b6` | 417,399 | 6,972 |
| **Child IC3R3** | `d29151b6d08a9cca9d510ed73f2d9f9c9df35d126d16cfd2e95d2c59fa15118f` | **421,426** | **7,028** |

Declared shorttitle `OSE-S7`. Terminates with a newline; zero CR bytes.
IC3R2, IC3R1, IC3 and IC2 all verified byte-identical after this run.

---

## 7. RESIDUALS

1. **Compile/load not performed** — no TradingView execution, no screenshots taken, no runtime
   claim made. R-01's effect must be confirmed visually at the next gate.
2. **R-01 mechanism not isolated** — the correction removes the dependency rather than
   diagnosing it. If the underlying price-basis mismatch also affects the SETUP marker's frozen
   `signalClose` anchor, that will show as SETUP markers off their bars while every other marker
   is correct. That is the specific thing to look for in the next runtime capture.
3. **Mission packet absent** — see §1. Scope may need extending once the packet is supplied.
4. **A47 BI-6** (Arabic in the Evidence strip) and **BI-7** (Execution Packet never supplied as a
   document) remain open from earlier returns; both are untouched here.
5. **FD-5 label crowding** — right-edge price labels can still overlap at near-identical prices,
   and the screenshot shows the opening-range label clipped at the plot edge. Legibility, not
   truth; unchanged and still reserved for visual evidence.

---

**END — CLAUDE IC3R3 RUNTIME-ANCHORING AND HISTORICAL-TRUTH CORRECTION RETURN v1.0**
