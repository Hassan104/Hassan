# OSE — STAGE 7 / MAIN CONTROL ROOM
## A49 — CLAUDE IC3R1 BOUNDED CORRECTION AND SOURCE-ACCEPTANCE RETURN v1.0

**Executing authority:** OSE-S7-CLAUDE-IC3R1-CORRECT-ACCEPT-01
**Governing artifact:** OSE-S7-MAIN-A48R v1.1 (supersedes A48 v1.0 in full)
**Role:** Claude — Development Owner and source-acceptance authority
**Date:** 2026-09-08

---

## 0. VERDICT

```
CLAUDE SOURCE ACCEPTED — IC3R1 READY FOR MAIN COMPILE/LOAD GATE
```

All four findings survived independent falsification, all four are corrected inside the
C-01…C-05 allowlist, and every §5 verification item passes. No bounded source defect remains
and no correction required authority beyond the allowlist.

Acceptance authorizes no compile or load by itself.

---

## 1. EXACT SOURCE IDENTITY

### Correction parent — authenticated from actual bytes before analysis or editing

| Property | Required by A48R §1 | Recomputed | Result |
|---|---|---|---|
| SHA-256 | `47f9a73b83883309ede9dba343700e2d58c43f2d9aa3beec7583f84e34f72da5` | `47f9a73b83883309ede9dba343700e2d58c43f2d9aa3beec7583f84e34f72da5` | **MATCH** |
| Bytes | 410,437 | 410,437 | **MATCH** |
| LF lines | 6,835 | 6,835 | **MATCH** |

**IC3 PARENT CUSTODY: PROVEN.** IC3 was opened read-only and its post-run hash is unchanged.

### Frozen lineage parent — not edited

`OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC2.pine` — `a7ab34af…7564730`, 376,892 bytes, 6,328 LF
lines. Re-verified unchanged after all edits.

### Child returned

| Property | Value |
|---|---|
| File | `OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3R1.pine` |
| SHA-256 | `415153c6eba135224908d12b89619b120cfda72a648b63018f93d85a885cd519` |
| Bytes | 417,424 |
| LF lines | 6,952 |
| Terminates with newline | yes (`wc -l` 6,952 == `awk END{NR}` 6,952) |
| CR bytes | 0 |

---

## 2. INDEPENDENT FALSIFICATION — F-01…F-04

Each finding was treated as a hypothesis and attacked against exact IC3 using actual dominating
controls and lifecycle reachability. None could be defeated.

### F-01 — REVERSAL CARD TUPLE / HANDOFF TRUTH → **CONFIRMED — CORRECTED**

Attempted defeat: does any dominating control stop the card from formatting the reversal from
post-handoff fields?

Reachability proof, all inside one execution pass on a single confirmed bar:

1. **S7.5.6** records the reversal with the correct tuple —
   `array.push(s7_evtText, "DIRECTION REVERSED · " + s7_dirText(s7_caseDir) + " → " + s7_dirText(s7_candDir))`
   at `S7_PRI_REVERSED` — then sets `s7_endedThisBar`.
2. **S7.5.8** ends the case: `s7_caseActive := false`.
3. **S7.5.9** re-opens the opposite case: `s7_caseActive := true`, and
   `s7_caseDir := s7_c2.direction` — the **new** direction. Immediately after, the candidate
   block clears `s7_candDir := 0`, `s7_candLive := false`.
4. **S7.6a** then evaluated
   `s7_cardWarnSub := s7_dirText(s7_caseDir) + " → " + s7_dirText(s7_candDir) + " · case ended"`
   → `s7_caseDir` = SHORT (new), `s7_candDir` = 0 → `s7_dirText(0)` = `"NEUTRAL"`.

**Rendered result: `SHORT → NEUTRAL · case ended`** — precisely the string A48R §3 C-01 names
as forbidden, and "case ended" is additionally false for the new case that is active.

No control closes this path: the guard was `s7_caseActive`, which S7.5.9 has just re-asserted
for the *new* case.

**Second reachable facet:** when the opposite case does *not* open — `s7_idOk` false, or
`s7_terminalBar` — `s7_caseActive` stays false, so the same `s7_caseActive` guard suppressed the
reversal band **entirely** on a genuine confirmed reversal.

### F-02 — REJECTED OBSERVATION MISLABELED SETUP → **CONFIRMED — CORRECTED**

Attempted defeat: is the strip source ever filtered by `riskPass` before reaching the EVENT cell?

It is not. The strip's Observation branch takes `array.get(rawObsSet, 0)` with **no** `riskPass`
filter, so `s7_stripHas` is true for a risk-rejected Observation. The EVENT cell then read
`s7_r5a = s7_stripHas ? "SETUP · " + …` — an unconditional literal. Meanwhile `s7_lifeMain`
independently produced `"REJECTED · "`. Card and table therefore contradicted each other on the
same snapshot.

### F-03 — ENDED-CASE AT-SIGNAL CONTEXT SUBSTITUTION → **CONFIRMED — CORRECTED**

Attempted defeat: does anything prevent an unrelated Observation from supplying an ended case's
AT SIGNAL context?

It does not. The strip source was a two-branch fallthrough — `if s7_caseActive … else if
rawObsSetClosed and s7_obsCount > 0 …`. On case end, S7.5.8 sets `s7_caseActive := false` while
retaining every anchor field (they clear only at the S7.5.0 session boundary). The card's
unconditional `if s7_caseEndedFlag and (not s7_caseActive)` override then titled the surface
`CASE ENDED`, while the comparison table fell through to `rawObsSet[0]` — an unrelated
current-bar Observation — and presented it as that ended case's AT SIGNAL.

Independently reachable and additionally corrected under the Product Owner's C-03 lifecycle:
**SIGNAL STATUS populated live values on the elected opportunity's own anchor bar**, presenting
same-pass live facts as post-signal state before any later bar existed.

### F-04 — ADVANCED INITIAL-STOP STYLE REGRESSION → **CONFIRMED — CORRECTED**

Attempted defeat: is the IC2 dotted style restored anywhere on the ADVANCED path?

It is not. IC2 created the line with `style = line.style_dotted` and never re-set it. IC3 added
an unconditional `line.set_style(s7_lineInit, line.style_dashed)` on the shared path, so
ADVANCED lost the inherited dotted style on every confirmed bar. Colour and width were already
depth-scoped correctly (`s7_simple ? s7c_dim : color.new(color.silver, 40)`, width 1); only
`style` regressed. TRAIL STOP and TP1 were checked on the same path and are **not** regressed.

**No finding was rejected. No rejected finding was corrected.**

---

## 3. CHANGED-HUNK LEDGER — 13 hunks, every one classified

| # | Site | Class | Change |
|---|---|---|---|
| 1 | `indicator()` line 395 | **C-05** | title/shorttitle IC3→IC3R1. All four caps unchanged |
| 2 | `S7_BUILD` | **C-05** | build string IC3→IC3R1 |
| 3 | Stage-7 firewall header | **C-05** | A49 correction manifest (comment only) |
| 4 | Strip source header | **C-02 / C-03** | election doctrine comment |
| 5 | Strip source decls | **C-02 / C-03** | `s7_stripFam`, `s7_stripDir`, `s7_stripPass`, `s7_stripEnded` |
| 6 | Strip branch 1 (ACTIVE) | **C-02 / C-03** | carries family, direction, riskPass |
| 7 | Strip branch 2 (Observation) | **C-02 / C-03** | carries family, direction, riskPass |
| 8 | Strip branch 3 (**new**) | **C-03** | retained ended case elects its **own** frozen anchor |
| 9 | Anchor-bar timing | **C-03** | `s7_atAnchorBar`, `s7_cmpLive` |
| 10 | CASE ENDED card override | **C-03** | gated on `s7_stripEnded` so card and table transition together |
| 11 | `s7_lifeMain` / `s7_lifeSide` | **C-02** | classification from the elected snapshot's own `riskPass` |
| 12 | `s7_revEvtText` + card warn band | **C-01** | reversal bound to the recorded event tuple |
| 13 | `line.set_style(s7_lineInit, …)` | **C-04** | depth-scoped dashed / dotted |
| — | comparison table cells (within hunk 11 region) | **C-02 / C-03** | columns drawn from one identity; CHANGES/STATUS withheld on the anchor bar |
| — | END marker | **C-05** | IC3→IC3R1 |

**No hunk falls outside C-01…C-05. No unrelated source change.**

### Note on the card-side element of C-02

Correcting only the comparison cell would have left a *new* contradiction: the card read
`"SETUP"` whenever **any** Observation on the bar passed Risk (`s7_setupPass > 0`), while its
identity fields already came from Observation 0 — so a rejected displayed snapshot could still
be titled SETUP by the card and REJECTED by the table. C-02's own invariant — *"primary card and
comparison table must not contradict each other for the same snapshot"* — and C-03's *"the
primary card and SIGNAL COMPARISON must transition together to that same context"* both require
the card to follow the elected snapshot. `riskPass` itself, Observation production, election and
every predicate are unchanged; only the source of the pass/fail read moved to the displayed
snapshot. This is inside C-02, not an extension of it.

---

## 4. CORRECTION-CLOSURE MATRIX

| Path | Elected context | AT SIGNAL | CHANGES | STATUS | Card | Closed |
|---|---|---|---|---|---|---|
| New SETUP, anchor bar | Observation 0 | its frozen snapshot | `—` | `—` | `SETUP · FAM · DIR` | C-03 §3.3/3.5 |
| New case ACTIVE, anchor bar | ACTIVE case | its frozen anchor | `—` | `—` | `ACTIVE · DIR · FAM · PRE-TP1` | C-03 §3.3/3.5 |
| REJECTED Observation, anchor bar | Observation 0 | its frozen snapshot | `—` | `—` | `REJECTED · FAM · DIR` + reason | F-02, C-02 |
| Next confirmed bar | unchanged | unchanged, immutable | delta vs anchor | live accepted state | unchanged | C-03 §3.4 |
| Subsequent bars | unchanged | unchanged, immutable | updates | updates | unchanged | C-03 §3.4 |
| Case ended, **no** new opportunity | retained ended case | **its own** frozen anchor | populated | populated | `CASE ENDED · <reason>` | F-03, C-03 §3.1 |
| Case ended, **new** opportunity appears | new Observation | new snapshot | `—` (anchor bar) | `—` (anchor bar) | `SETUP`/`REJECTED` — transitions with the table | F-03, C-03 §3.2 |
| Reversal bar, opposite case opens | new ACTIVE case | new frozen anchor | `—` (anchor bar) | `—` (anchor bar) | new case head + recorded reversal band | F-01, C-01 |
| Reversal bar, opposite case does **not** open | ended case or new Observation | per election | per timing | per timing | recorded reversal band still shown | F-01 second facet |
| ACTIVE + same-direction new Observation | ACTIVE case retains context | case anchor | live | live | `ACTIVE …` | no new contradiction (Final UX §33 preserved) |
| Session reset | none | `—` | `—` | `—` | per readiness | branch 3 guarded by `not na(s7_caseSignalTime)` |

**Sibling-path regression check (§5.4):** an ACTIVE case still outranks a same-bar Observation,
so case election and Final UX §33 are untouched; SETUP chart markers still render every
Observation regardless of election. No path produces a card/table disagreement.

---

## 5. §5 VERIFICATION RESULTS — 17 of 17

| # | Item | Result |
|---|---|---|
| 1 | Exact IC3 parent custody | **PASS** — three values recomputed and matched |
| 2 | Complete diff classified under C-01…C-05 | **PASS** — 13 of 13 |
| 3 | Surviving failure paths closed | **PASS** — §4 matrix |
| 4 | No sibling lifecycle contradiction | **PASS** — §4 last three rows |
| 5 | C-03 timing on every named path | **PASS** — anchor bar, next bar, subsequent bars, ended/no-new, SETUP, REJECTED |
| 6 | IC3 behavior retained outside the four corrections | **PASS** — card, Evidence, SQUEEZE, comparison, geometry, markers otherwise untouched |
| 7 | AT SIGNAL immutable and context-coherent | **PASS** — all columns from one elected identity; `s7_strip*` are per-bar locals, never `var`; zero new anchor writes |
| 8 | SIMPLE / ADVANCED depth-exclusive | **PASS** — S7.8.0 lifecycle unchanged |
| 9 | ADVANCED otherwise preserved | **PASS** — all 39 `s7_pushRow` rows byte-identical; C-04 restores IC2 dotted |
| 10 | Stage-6 inherited body byte-identical | **PASS** — lines 1–5021 excluding the declaration identity line, `49a3a701…d5813` |
| 11 | 52 exports, identical title and order | **PASS** |
| 12 | Three inherited `request.*` calls unchanged | **PASS** |
| 13 | Stage-7 inputs exactly three | **PASS** |
| 14 | Alert eligibility, message construction, single `alert()` stream | **PASS** — identical to IC3; `alertcondition` 0, `alert()` 1 |
| 15 | Caps unchanged, worst case within them | **PASS** — §6 |
| 16 | No reverse write, new predicate/threshold, unbounded growth, invalid index, runtime-fatal path | **PASS** — see below |
| 17 | A47 BI-6 / BI-7 / BI-8 remain resolved, not re-raised | **PASS** — zero Arabic in strip cells; strip still `bottom_left`; SQUEEZE still its own row; FD-1…FD-5 untouched |

**Item 16 detail.** The four new booleans are `s7_atAnchorBar` (an integer *equality*),
`s7_cmpLive` (its negation), `s7_stripEnded`, and `s7_stripPass` (a read of the existing
`riskPass`) — **no magnitude comparison, so no new threshold**. The C-01 loop is bounded by
`s7_domN = array.size(s7_evtPri)` and guarded by `if s7_domN > 0`; all four event ledgers take
**21 parallel pushes** each, so `array.get(s7_evtText, s7_i)` is index-safe, and it reuses the
exact pairing IC3's own dominant-event loop already relies on. Table dimensions and every cell
coordinate are unchanged. No `var` state was added, so no unbounded growth is possible.

**Static Pine-v6 syntax / type / scope review:** declaration-before-use confirmed for all seven
new identifiers; zero empty block bodies; the only five non-multiple-of-4 indents are the
pre-existing Stage-6 tuple continuations at lines 612–616, unchanged from IC2. **Compiler
acceptance is not claimed.**

---

## 6. PRESERVATION AND CONSTRUCT / RESOURCE COUNTS

Executable construct counts are **identical** IC3 → IC3R1 — the corrections changed expressions
and election, never object creation.

| Construct | IC3 | IC3R1 |
|---|---|---|
| `table.new` / `table.cell` | 4 / 48 | 4 / 48 |
| `label.new` / `line.new` / `box.new` | 11 / 7 / 3 | 11 / 7 / 3 |
| `plot()` / `input.*` | 52 / 43 | 52 / 43 |
| `alertcondition` / `alert()` | 0 / 1 | 0 / 1 |

| Resource | Cap | SIMPLE worst case | ADVANCED worst case |
|---|---|---|---|
| `max_lines_count` | 16 | 7 | 3 |
| `max_labels_count` | 64 | 51 | 33 |
| `max_boxes_count` | 16 | 4 | 6 |
| `max_bars_back` | 300 | 300 | 300 |

Caps unchanged; worst case unchanged from accepted IC3 and inside every cap.

**Disclosed shared-read consequence (in-scope, C-03).** The strip source feeds both depths. With
election branch 3 added, a retained ended case with no new Observation now shows **its own**
frozen Evidence in ADVANCED as well, where IC3 showed `—`. This is the same truth correction
F-03 requires, applied consistently; it is not an ADVANCED redesign, and the ADVANCED panel's own
AT SIGNAL rows read `s7_case*` directly and are byte-identical.

---

## 7. RESIDUALS RESERVED FOR COMPILE / LOAD / VISUAL EVIDENCE

Not repaired here, and explicitly outside A48R:

1. **Pine compiler acceptance** — static review only; no compile, load or run performed.
2. **Right-edge label overlap (FD-5)** — Pine labels carry no measurable height, so
   near-identical prices may still crowd. Legibility, not truth. Needs visual evidence.
3. **H5 / runtime / viewport / pixel-overlap residuals** — A48R §4 forbids repairing these.
4. **`s7_setupPass`** — the multi-Observation pass accumulator is now assigned but not read after
   the C-02 correction. It remains a self-referencing `+= 1` so it is not an unused-variable
   diagnostic, and removing it is not in the allowlist. Flagged so it is not mistaken for an
   oversight; a future tidy-up may drop it.
5. **A47 BI-6 (Arabic in the Evidence strip)** — still awaiting Product Owner disposition. A48R
   §4 forbids changing the A47 Evidence-language resolution, so IC3R1 leaves it exactly as
   accepted. Not re-raised here as a defect.
6. **A47 BI-7** — the Optimized Execution Packet still has not been supplied as a document;
   IC3R1 continues to treat IC2's inline packet references as the binding invariants.

---

## 8. MISSION-BOUNDARY COMPLIANCE

Not performed: compile, load, TradingView execution, screenshot/CSV/alert collection, runtime
evidence claims, additional reviewer launch, SIMPLE rewrite or redesign, ADVANCED redesign, any
change to the A47 Evidence-language or lower-left-placement resolutions, any change to FD-1…FD-5,
any Stage-6 / B3 / UX Gate / Stage-8 / Production reopening. No score, confidence, probability,
Trade Health, Q6, TP2/TP3/TP4, institutional inference, options-chain fact or broker/execution
behavior added. No input, `request.*`, plot export, `alertcondition`, resource cap or alert
stream added or changed. IC2 and IC3 both left byte-identical.

---

## 9. FINAL VERDICT

```
CLAUDE SOURCE ACCEPTED — IC3R1 READY FOR MAIN COMPILE/LOAD GATE
```

| | SHA-256 | Bytes | LF lines |
|---|---|---|---|
| Parent IC3 | `47f9a73b83883309ede9dba343700e2d58c43f2d9aa3beec7583f84e34f72da5` | 410,437 | 6,835 |
| **Child IC3R1** | `415153c6eba135224908d12b89619b120cfda72a648b63018f93d85a885cd519` | **417,424** | **6,952** |

Returning to Stage-7 Main. Acceptance authorizes no compile or load by itself.

---

**END — OSE-S7-A49-CLAUDE-IC3R1-BOUNDED-CORRECTION-AND-SOURCE-ACCEPTANCE-RETURN-v1.0**
