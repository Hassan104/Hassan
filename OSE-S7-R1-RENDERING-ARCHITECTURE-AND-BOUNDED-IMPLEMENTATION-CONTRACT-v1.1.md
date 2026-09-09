# OSE S7 — R1 RENDERING ARCHITECTURE AND BOUNDED IMPLEMENTATION CONTRACT v1.1

**Mission:** OSE-S7-CLAUDE-R1-CONTRACT-COMPLETION-02 · **Executor:** Claude, Development Owner · **Date:** 2026-09-08
**Supersedes v1.0.** No Pine edited this round. No compile/load, reviewer, evidence collection or source acceptance.

## 0. Change summary vs v1.0

| # | v1.0 position | v1.1 |
|---|---|---|
| 1 | C-1 scale mismatch, top-ranked hypothesis | **Resolved.** Operator found *No scale (fullscreen)*; pinning to the right (candle) scale fixed motion and alignment with **no source edit**. Scale arithmetic, source compensation and further yloc repair are removed from scope. The A/L-button argument is withdrawn as proof of multiple scales |
| 2 | A-1: replace close-time x at 3 site classes | **Narrowed to 1 site** after the required reader audit. `s7_caseStartTime` is read *only* by the three protection lines (no analytical consumer) — but a protection level legitimately becomes valid at the case-open bar's **close**, so it must **not** change. v1.0 over-reached; corrected |
| 3 | `max_bars_back` bounds the archive horizon | **Withdrawn.** It sizes historical series buffers; it does not cap the age of timestamped records in a persistent archive. The 1-minute objection is dropped; scope stays 5-minute |
| 4 | 123 labels ⇒ shortfall 59; four options incl. cap raise / SETUP cut | **Withdrawn as a minimum.** It priced full replication. A no-replication design fits **inside current caps with margin** and needs no cap increase, no live-quota cut and no loss of a live fact |
| 5 | Verdict HOLD (2 items) | **READY** — both items closed |

Unchanged and still controlling: three-layer separation; R2 (cards/right rail, status line, ADVANCED redesign) deferred;
Stage-6/B3 closure; management/alert semantics, 52 exports, 3 requests, 3 inputs preserved.

**Source (unchanged, reused, re-verified locally):** `OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3R3.pine` — SHA-256
`d29151b6…5118f`, 421,426 bytes, 7,028 LF lines, shorttitle `OSE-S7`. Nothing requested; nothing inaccessible.

## 1. Runtime disposition recorded

C-1 **resolved for the observed failure**: attachment to the candle price scale is the verified configuration and is
recorded as a precondition for the later checkpoint. It cannot excuse a future coordinate mismatch. Green
`ORL · SUPPORT` renders, so universal absence of support is disproved; **registry-to-render completeness remains
unverified**. Role-based colours preserved; no support fabricated. IC3R3 compile/load PASS stands; R1 runtime
presentation acceptance remains OPEN.

Remaining R1 scope is exactly three items: **(B)** horizontal extent of levels/bands within validity, **(A)** bounded
prior-session history separate from live state, **(C)** event markers fixed to their originating candle with reduced
crowding.

## 2. (A) Retention design and complete budget

### 2.1 Measured live baseline (from source, not estimated)

Price-reference labels/lines are SIMPLE-gated (`s7_planLbl`, `s7_vwapLine`, `s7_orLines`, `s7_barLine` all require
`s7_simple`); the VWAP/OR **bands** are ADVANCED-only.

| Pool | SIMPLE live | ADVANCED live | Cap |
|---|---|---|---|
| Labels | 7 price + 24 SETUP + 8 WATCH + 12 events = **51** | 24 + 8 + 1 event label = **33** | 64 |
| Lines | IS, TS, TP1, BARRIER, VWAP, ORH, ORL = **7** | IS, TS, TP1 = **3** | 16 |
| Boxes | 4 zone bands = **4** | 4 zones + VWAP band + OR band = **6** | 16 |

**Transition worst case = 51 labels**, not a sum: a depth switch deletes the outgoing depth's objects in the same
pass that creates the incoming depth's, and SETUP/WATCH persist across it. **Eviction precedes creation in every
path** — both label FIFOs shift-and-delete before `label.new`, and the zone loop removes stale boxes before the
creation loop — so no `+1` allocation transient exists. A 1 label / 1 line / 1 box reserve is nevertheless declared.

Free after live, SIMPLE (binding depth): **labels 13, lines 9, boxes 12.**

### 2.2 The insight that removes the shortfall

Replication was never required. Two facts change the arithmetic:

1. **Archived structure needs no labels.** A subdued historical segment is a box (range) or line (level) whose text
   is omitted — subdued historical treatment is exactly what is wanted, and it costs zero from the scarce pool.
2. **Lines and boxes are the abundant pools** (9 and 12 free) while labels are scarce (13). Historical structure is
   therefore paid for out of the abundant pools, and the label budget is spent solely on archived event markers.

### 2.3 Recommended allocation — **Option A**

Horizon: current RTH session + 2 completed recent RTH sessions, 5-minute.

| Class | Per completed session | Total archived | Pool | Final | Cap | Spare |
|---|---|---|---|---|---|---|
| Management event markers | **6** | 12 | labels | 51 + 12 = **63** | 64 | 1 |
| Structure range segments (bands) | **4** | 8 | boxes | 4 + 8 = **12** | 16 | 4 |
| Structure level segments (ORH/ORL) | **2** | 4 | lines | 7 + 4 = **11** | 16 | 5 |

No cap increase, no live SETUP/WATCH/event quota reduction, no live rendering primitive changed, margin in all
three pools. **Recommended.**

### 2.4 What is inspectable and what is omitted — stated precisely

- Retained per completed session: the **6 highest-priority** management events by the existing priority rank, in
  chronological order; up to 4 structure range segments; up to 2 level segments. Ties break by earliest bar.
- **Omitted** in completed sessions: SETUP/REJECTED markers, WATCH markers, price-reference labels, and any event
  beyond the 6th. A typical single-case session (ACTIVE, TP1 REACHED, TRAIL ON, STRUCTURE LOST, STOP BREAK,
  CASE ENDED) fits; a two-case session **will** truncate.
- Favourable same-bar clustering is **not** counted as headroom. Same-bar grouping keeps the existing `+N` count and
  truthful detail; **events from different originating bars are never merged** into one falsely anchored marker.
- **Truncation is disclosed, never labelled complete.** Zero-cost site: `s7_evTbl` row 1 is currently an all-blank
  spacer row; it carries `HISTORY · 2 SESSIONS · n/m RETAINED` (and `INCOMPLETE` when evicted). Screen-fixed, no
  drawing object. *This is a display-only addition to the SIMPLE composition and is listed for approval in §5.*
- Eviction is deterministic: oldest completed session first, then lowest priority then oldest within a session. The
  archive never evicts a live object; live never evicts archived without the disclosure updating.

### 2.5 Quantified alternative — **Option B**, only if 6 events/session is judged insufficient

Render the 7 live price references as zero-height boxes carrying right-aligned text
(`box.new(top == bottom, border_style = …, text = …, text_halign = text.align_right)`), replacing 7 lines + 7 labels
with 7 boxes. Live labels drop to 44.

| Class | Per session | Pool | Final | Cap | Spare |
|---|---|---|---|---|---|
| Event markers | **9** | labels | 44 + 18 = **62** | 64 | 2 |
| Range segments | **3** | lines (2 each) | 0 + 12 = **12** | 16 | 4 |
| Level segments | **2** | boxes | 4 + 7 refs + 4 = **15** | 16 | 1 |

**Trade-off, exactly:** +3 archived events per session, −1 archived range per session, price-reference typography
moves from a label beside the line to text inside a zero-height box, and the box pool falls to 1 spare. Higher risk,
higher history. **Not recommended for the first implementation.**

No shortfall remains under either option, so no Product Owner exception is required.

### 2.6 `max_bars_back`

Corrected per Main: it sizes historical series buffers and does not cap the age of timestamped records held in a
persistent archive. Archived objects are handles plus frozen scalar records in arrays; rendering them requires no
historical series access. `max_bars_back = 300` is therefore **not** a horizon constraint here. The real bounds are
the three drawing caps above and array memory, both budgeted. Scope stays 5-minute.

## 3. (B) Price structure — extent and validity

- **Live segment:** `known-from → most recent confirmed bar`, where *known-from* is the bar at which the entry
  entered `zoneReg` with its **current** role. No projection into future bars.
- **Historical segment:** `known-from → valid-until`, closed at role migration, removal from the registry, or session
  end — whichever occurs first. Historical segments are never recoloured with a later role and never extended.
- **Role migration opens a new segment** and closes the previous one at the migration bar. Later knowledge is never
  projected backward; a current-reference overlay, if ever used, must be declared as such and is not proposed here.
- **Ranges keep exact bounds** (`Zone.lower` / `Zone.upper`) as bands; **levels stay levels**. No midpoint collapse —
  already true in IC3R3 and preserved.
- Colour remains solely `structuralRole` → SUPPORT green / RESISTANCE red / NEUTRAL gray. No PDH/PDL-name or
  close-relative shortcut exists in source and none is added. Accepted prices, roles and missingness preserved.
- VWAP, OR and case-plan geometry keep their own producer/readiness/lifecycle contracts. **No historical VWAP
  reconstruction.**
- Segment budget is §2.3: 4 range + 2 level segments per archived session. A session in which several zones migrate
  role exceeds that; the excess is evicted oldest-first and disclosed per §2.4.

## 4. (C) Event anchoring — audited, and narrower than v1.0

**Reader audit (required before proposing any change).** `s7_caseStartTime` is written twice (`:= time_close`) and
read **only** by the three protection lines' `x1` plus its own `na` guard. No analytical, alert, export or state
consumer exists. Despite that, it is **not changed**: INITIAL STOP / TRAIL STOP / TP1 become valid at the case-open
bar's close, so close-time is the correct validity origin for a protection level. v1.0's blanket
`time_close → time` proposal is withdrawn.

**Originating time vs confirmation time.** The frozen presentation record stores both: `originBarTime` (the bar the
marker attaches to) and `confirmTime` (when the event was committed/activated). Marker *anchoring* uses
`originBarTime`; marker *text* uses `confirmTime`. Today the marker appends the render bar's `time_close`, which
coincides only because creation is same-bar — the record makes that explicit rather than incidental. No event is
made to appear available earlier than it was.

**Exact display-only correction sites — one:**

| Site | Now | Correct to | Why |
|---|---|---|---|
| SIMPLE WATCH marker | `x = time_close` | `x = originBarTime` (bar open of the arm's first-seen bar) | a WATCH marker exists to mark its originating candle; `xloc.bar_time` resolves a close time onto the **next** bar |

Not changed: management event markers (`x = time`, already correct); SETUP marker (`x = s7_o.obsBarTime`, already the
RTH bar-open time); ADVANCED current-event label (ADVANCED preserved); `s7_sessionRight` (a session boundary, not an
event anchor).

**Marker treatment and collision policy.** Concise markers, no large text islands. Co-bar events share one compact
marker with the existing `+N` count and truthful retained detail, preserving current priority and chronology. No
fabricated ATR/percentage offsets, no distant price lanes, no silent discarding of a meaningful event — anything
dropped is disclosed per §2.4.

## 5. (D) Lifecycles, provenance, and the exact allowlist

**Lifecycles.** Session/calendar rollover: live objects are **promoted** to the archive, not deleted; live case state
resets exactly as today — a retained drawing never keeps a prior case live. Symbol change, `s7_uiEnabled` toggle and
depth switch each change an input, which makes TradingView re-execute the script from the first bar, so the archive
**rebuilds deterministically** from replayed bars; the same holds on reload. The only irrecoverable case is data
absent from the chart's loaded history, which is disclosed as incomplete rather than shown as complete. Archived
display records have **no consumer** in analytical state, case election, Risk, alerts, queue or exports.

**Provenance check (design complete, no diagnostic build authorized).** A read-only row group in the existing
screen-fixed ADVANCED panel listing, per registry entry: `mask · lower · upper · role · known-from · segment count ·
rendered`. Source-proven today: identity/mask, bounds, role, eligibility gate, object identity, colour mapping.
**Runtime-unmeasured (NOT PASS):** role-migration timing, and registry-to-render completeness on both symbols.

**Proposed source allowlist — display-only, for Main's authorization:**

| # | Change | Class |
|---|---|---|
| A-1 | SIMPLE WATCH marker anchored to its originating bar (§4) | display-only, 1 site |
| A-2 | Frozen presentation record (`originBarTime`, `confirmTime`, identity, kind, price/attachment, case identity, live/historical) | new presentation state, no consumer |
| A-3 | Archive pools + deterministic eviction + promotion at session rollover, per §2.3 | presentation lifecycle |
| A-4 | Live segments extended to `known-from → last confirmed bar`; historical segments closed at `valid-until`; role migration segmentation (§3) | presentation geometry |
| A-5 | Subdued unlabelled treatment for archived segments | presentation style |
| A-6 | Retention/incompleteness disclosure in the existing blank `s7_evTbl` spacer row (§2.4) | display-only addition — **needs explicit nod**, it is the one item that alters SIMPLE composition |

Preserved: Stage-6 source, producer semantics, 52 exports/title/order, 3 `request.*`, 3 inputs and their defaults,
management and alert behaviour, all unrelated source. No cap change, no new analytical logic, threshold, score or
decision authority. No R2, no ADVANCED redesign, no status-line work.

## 6. Later runtime checkpoint (already bounded — unchanged)

Same AAPL and TSLA 5-minute setups, **with the indicator pinned to the candle price scale** (now a recorded
precondition). One pan/zoom recording per symbol ≤ 60 s, or paired before/after stills plus operator observation if
video is unavailable — not both. Maximum four stills total covering live, two prior-session inspection and
structural provenance. Zero CSVs, zero live-alert captures; reuse existing captures. Outcomes to establish:
same-price alignment; origin-bar tracking through pan/zoom; horizontal structure spans within validity;
source-matched support/resistance completeness; bounded retention without live-context contamination; readable
event footprint. **Source-only proof cannot establish these and none is claimed.**

## 7. Verdict

```
R1 CONTRACT READY FOR MAIN IMPLEMENTATION AUTHORIZATION
```

Both v1.0 blockers are closed: C-1 was resolved operationally, and the retention budget now fits inside 16 lines /
64 labels / 16 boxes with margin under the recommended Option A — no cap increase, no live-quota reduction, no lost
live fact, no manufactured PASS. The single item needing an explicit word from Main is **A-6** (the disclosure row);
if it is declined, name the alternative disclosure surface and the contract is otherwise unchanged. Option B remains
available if 6 archived events per session is judged too few.

This verdict authorizes no source edit. Returning to Main and stopping.
