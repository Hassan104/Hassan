# OSE S7 — CLAUDE R1 IC3R4 IMPLEMENTATION RETURN v1.0

**Mission:** OSE-S7-CLAUDE-R1-IC3R4-IMPLEMENT-01 · **Executor:** Claude, Development Owner · **Date:** 2026-09-08
**Authority:** OSE-S7-MAIN-R1-IC3R4-BOUNDED-IMPLEMENTATION-AUTHORITY v1.0, over contract v1.1 as amended there.

## 0. Verdict

```
CLAUDE R1 IMPLEMENTATION COMPLETE — IC3R4 RETURNED FOR INDEPENDENT BOUNDED STATIC VERIFICATION
```

No SOURCE ACCEPTED, no runtime acceptance, no compile/load, no reviewer, no evidence collection.

## 1. Custody

| Object | SHA-256 | Bytes | LF lines | Result |
|---|---|---|---|---|
| IC3R3 parent (required) | `d29151b6…5118f` | 421,426 | 7,028 | **MATCH**, recomputed before editing |
| Contract v1.1 (required) | `59f43ff3…f2f943` | 15,383 | 213 | **MATCH** |
| **IC3R4 child** | `79ee0fa053dd5e8495696e36ae31f62540e221e370bcc35a7bb50e6ba94464b9` | **446,668** | **7,455** | — |

IC3R4 terminates with a newline (`wc -l` 7,455 == `awk END{NR}` 7,455), zero CR bytes, shorttitle `OSE-S7`
preserved. **IC3R3 was opened read-only and its post-run hash is unchanged.**

## 2. Changed-hunk ledger — 18 hunks, all classified

| # | Site | Class |
|---|---|---|
| 1, 2, 18 | `indicator()` title, `S7_BUILD`, END marker | identity (shorttitle unchanged, caps unchanged) |
| 3 | `S7_ARC_SESSIONS/EVENTS/RANGES/LEVELS`, `S7_EXT_BARS` | Option-A constants |
| 4 | S7.2b helpers: `s7_candPush`, `s7_segPush`, `s7_lvlPush`, `s7_arcDropSlot`, `s7_arcAgeSlots`, `s7_arcDropAll` | A-2/A-3 pure presentation |
| 5 | S7.3.6 frozen records + archive drawing pools + per-session counters | A-2 |
| 6 | `s7_caseIdPre` snapshot before the engine | R-2 |
| 7 | Session rollover: close-open-segments → seal → expire → age → promote → clear | R-4 / A-3 |
| 8 | Candidate collection as events occur | R-2 |
| 9 | S7.6b disclosure computation | A-6 |
| 10 | Disclosure rendered in the existing `s7_evTbl` spacer row | A-6 |
| 11 | UI-OFF teardown drops the archive | R-4 |
| 12 | `s7_barMs` / `s7_extRight` / `s7_liveRight`; registry-unusable closes open segments | R-1 |
| 13 | Registry removal freezes the segment at its endpoint | R-1 |
| 14 | Segment opened on box creation; `known-from` left edge; `s7_liveRight` right edge; role/bounds migration closes and reopens | R-1 |
| 15 | Opening-range level segments + live extent | R-1 |
| 16 | SIMPLE WATCH marker anchored to its originating bar | R-5 / A-1 |
| 17 | Concise live marker text + tooltip; depth-switch drops the archive; corrected the obsolete scale attribution comment | R-5 / R-4 / authorized comment removal |

No hunk falls outside the amended allowlist.

## 3. Selection and omission policy — as implemented

**Events.** Eligibility is the existing SIMPLE management-marker policy (`s7_domPri >= S7_PRI_TRAILON`, non-empty
dominant text); nothing new is detected, scored or re-prioritised. One candidate per **originating bar group**,
carrying the existing same-bar `+N` count and dominant priority. Candidates are frozen **as events occur**, so the
live 12-slot FIFO can never erase a qualified one. Selection is a streaming top-K over the existing priority: a
strictly higher priority displaces the weakest held group, where weakest = lowest priority and, on a tie, the
**latest** originating time. An equal-priority later group therefore never evicts an earlier one — this is exactly
the R-3 rule, and it removes v1.1's earlier-first/oldest-first contradiction. Groups render at their own bar times,
so display is chronological by construction. Different bars are never merged.

**Case context.** `s7_caseIdPre` (pre-engine) and `s7_caseId` (post-engine) are both stored. On a reversal bar the
tooltip reads `#7 → #8`; on any other bar `#7`. Same-bar events are never all re-attributed to a newly opened case.

**Structure.** A range segment closes on role migration, bounds change, registry removal, registry unusability or
session end, and is frozen at its real endpoint; a new segment opens with `known-from = time`. Live boxes start at
`known-from`, never earlier. OR level segments follow the same rule on value change or unavailability. Completed
segments keep the **most recent**, evicting oldest.

**Omitted in completed sessions, disclosed, never implied complete:** SETUP/REJECTED markers, WATCH markers,
price-reference labels, VWAP history, protection/target history, and any event group beyond the 6th, range beyond
the 4th, level beyond the 2nd. The disclosure always reads `SELECTED`, adds `INCOMPLETE` on quota truncation and
`PARTIAL` when fewer than two completed sessions are loaded — two distinct conditions, never conflated. The tooltip
carries three **separate** ratios (events, ranges, levels) and states explicitly that the event ratio does not
describe structural coverage.

## 4. Peak resource accounting

Live quotas unchanged (7 price labels, 24 SETUP, 8 WATCH, 12 event markers, 7 lines, 4 boxes).

| Pool | SIMPLE live | Archive (2 × quota) | **Peak** | Cap | Spare |
|---|---|---|---|---|---|
| Labels | 51 | 12 | **63** | 64 | 1 |
| Boxes | 4 | 8 | **12** | 16 | 4 |
| Lines | 7 | 4 | **11** | 16 | 5 |
| ADVANCED labels | 33 | 0 (SIMPLE-only) | **33** | 64 | 31 |

**Transients proven, not assumed.** (a) *Rollover*: live pools are cleared by the existing reset **before** the
archive block runs, and the oldest slot is expired **before** any replacement is created, so the instantaneous peak
is 7 price labels + 6 aged + 6 promoted = **19**. Promoting before expiry would peak at 25 — still legal, but
expire-first is implemented and simulated. (b) *Depth switch*: SIMPLE→ADVANCED deletes archive and marker pools in
the same pass that creates the single ADVANCED label; ADVANCED→SIMPLE creates at most 7 price labels while
SETUP/WATCH persist — neither exceeds 51. (c) *FIFOs*: eviction precedes creation in every path, including the zone
loop, which removes stale boxes before the creation loop. (d) *Records*: every array is hard-capped
(6 candidates, 4 ranges, 2 levels, 4 live segments, 2 counter slots ×6) — **no unbounded archive and no reliance on
garbage collection**. Caps 64/16/16/300 unchanged.

## 5. Bounded verification results

**Preservation (mechanical).** Stage-6 body lines 1–5021 excluding the declaration identity line **byte-identical
to IC2**; 52 exports identical in title and order; 3 `request.*` identical; 3 Stage-7 inputs (43 total) unchanged;
`alertcondition` 0 / `alert()` 1 with identical construction and eligibility; **management-state write set
identical to IC3R3** (diff of all `s7_case*`/`s7_trail*`/`s7_struct*`/`s7_warn*`/`s7_cand*` assignments is empty);
all 39 ADVANCED `s7_pushRow` rows identical; declaration caps unchanged.

**Static review.** Declaration-before-use confirmed for every new identifier, including the disclosure globals
(declared at S7.6b, line 6629) relative to `s7_renderSimple` (line 6708). Zero empty block bodies. The only five
non-multiple-of-4 indents are the pre-existing Stage-6 tuple continuations at 612–616. Tooltip newlines are proper
`\n` escapes. `s7_liveRight` declared before all ten uses. `s7_staticMasks` and the four `s7_segLive*` arrays are
pushed, removed and cleared together at every site, so index alignment holds. The rollover is behind a single
boundary `if`, and `s7_sessionOpen` is reset such that `s7_calendarAdvance` cannot re-fire — **no duplicate
promotion**.

**Bounded simulation** (algorithms transcribed to Python; presentation logic only, no Pine/runtime claim) —
**25 of 25 checks pass**: >6 eligible groups keeps exactly the 6 highest priorities with a truthful eligible count;
equal-priority later group rejected while a strictly higher one is admitted and displaces the *latest* of the tied
weakest; an early `REVERSED` group evicted from the live 12-FIFO survives in the archive candidates; one group per
originating bar with both case ids retained through handoff; 7 range changes bound to 4 keeping the most recent with
exact bounds and endpoints; rollover → PARTIAL at one session, clears at two, third rollover evicts the oldest and
holds 12 marks across slots {0,1}; an empty session fabricates nothing; INCOMPLETE raised on truncation and the text
never claims full history; and the peak-resource arithmetic above.

**Not established here:** Pine compiler acceptance, and anything runtime. `yloc.abovebar/belowbar` ignoring the `y`
argument, and label/box/line creation at historical `xloc.bar_time` coordinates, are the two documented Pine
behaviours this implementation rests on.

## 6. Remaining runtime checks (for the separately released checkpoint)

Indicator pinned to the candle price scale (recorded precondition). Then: same-price alignment; live bands and OR
levels visibly continuing to the right of the last bar with historical segments stopping at their endpoints; archived
markers sitting on their own originating bars across two prior sessions; the disclosure row reading `SELECTED` with
correct `PARTIAL`/`INCOMPLETE`; unchanged ADVANCED composition; and no duplicate drawings after rollover, depth
change, UI toggle or reload.

**Honest note on rebuild:** archive drawings are dropped on UI OFF and on a switch to ADVANCED. Both are input
changes, which make TradingView re-execute from the first bar, so the archive rebuilds from replayed bars. That
reasoning is sound but **is itself a runtime behaviour and is not proven here** — it belongs in the checkpoint list
above.

---

**END — OSE-S7-CLAUDE-R1-IC3R4-IMPLEMENTATION-RETURN-v1.0**
