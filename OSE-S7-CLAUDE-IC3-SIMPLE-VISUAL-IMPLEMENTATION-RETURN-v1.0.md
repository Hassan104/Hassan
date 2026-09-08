# OSE — STAGE 7 / SIMPLE UI IMPLEMENTATION PILOT
## CLAUDE IC3 SIMPLE VISUAL IMPLEMENTATION — RETURN v1.0

**Mission ID:** OSE-S7-CLAUDE-IC3-SIMPLE-VISUAL-IMPLEMENTATION-01
**Role:** Claude — Development Owner
**Return date:** 2026-09-08

---

## 0. RETURN VERDICT

**B. CLAUDE PARTIAL SIMPLE IMPLEMENTATION — BOUNDED FEASIBILITY OR AUTHORITY ISSUE REQUIRES PRODUCT OWNER DISPOSITION**

Every element of the approved SIMPLE design is implemented; nothing was left unbuilt. Verdict B
is issued **solely** because the enumerated bounded items in §7 require Product Owner
disposition — one of which (BI-6) is a material conflict between two controlling authorities
that Mission §2 forbids resolving silently. Read B here as *"complete and returnable, pending
disposition of §7"*, not as work abandoned mid-way.

**No SOURCE ACCEPTED is issued.** No compile, load, TradingView execution, screenshot, CSV or
runtime claim is made anywhere in this return.

---

## 1. EXACT SOURCE IDENTITY

### Parent (unchanged, re-verified after all edits)

| Property | Value |
|---|---|
| File | `OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC2.pine` |
| SHA-256 | `a7ab34af13c456cf6e7dd4b9b3c9f59aa5018d326f3c779096ca430fa7564730` |
| Bytes | 376,892 |
| LF lines | 6,328 |

All three recomputed independently before editing and re-confirmed after. **IC2 was opened
read-only and is byte-identical; it was never overwritten or modified.**

### Child (returned)

| Property | Value |
|---|---|
| File | `OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3.pine` |
| SHA-256 | `47f9a73b83883309ede9dba343700e2d58c43f2d9aa3beec7583f84e34f72da5` |
| Bytes | 410,437 |
| LF lines | 6,835 |
| Terminates with newline | yes (`wc -l` 6,835 == `awk END{NR}` 6,835) |
| CR bytes | 0 |

### Visual authority

| Property | Value |
|---|---|
| Artifact | `OSE_S7_SIMPLE_Prototype_v3_standalone.html` |
| SHA-256 | `85d386b3e6ecad49147aa1a41671975778c56d1ed29a4bdf7b6bd6f0f6bf39be` |
| Bytes | 871,949 |
| Design identity | `OSE-S7-CLAUDE-UI-DESIGN-01 · D1 · REVISION 3` |

Decoded from its bundler manifest (45,804-byte design template + `support.js`, confirmed to be
the generated `dc-runtime` framework carrying no design content), then rendered in headless
Chromium and visually inspected across all three artboards. The inline SVG was identified as a
loading thumbnail and **not** used as design authority.

---

## 2. VISUAL-ELEMENT → SOURCE-PRODUCER MAPPING

Every implemented element reads an already-accepted producer. No producer was added, renamed
in substance, or redefined.

| Approved element | Producer read | Implementation |
|---|---|---|
| Case head `ACTIVE · LONG · LBO · POST-TP1` | `s7_lifeMain` + `s7_lifeSide` | card row 0 |
| DIRECTION WARNING / CRITICAL / REVERSED band | `s7_warnActive`, `s7_candLive`, `s7_domPri`, `structBias` | card row 1 |
| `STRUCTURE ● INTACT` + Arabic | `s7_structText` / `s7_structAr` (Gate-4 premise) | card row 2 |
| `CURRENT EVENT` + `+N` + time | `s7_dirEventText`, `s7_domText` | card row 2 |
| Main support / resistance bands | `zoneReg` `Zone.structuralRole` | `box` bands, green/red/gray family |
| VWAP thin neutral reference | `vwap` (existing `request.security` bundle) | line + right-edge label |
| Opening range high / low | `orHigh`, `orLow`, `orReady` | thin lines + labels |
| INITIAL STOP | `s7_caseInitialStop` (original R-INV) | dashed line + label |
| TRAIL STOP | `s7_trailStop`, `s7_trailVersion` | 2px solid line + label |
| TP1 (+ REACHED) | `s7_caseTp1`, `s7_tp1Reached` | 2px dashed line + label |
| NEXT BARRIER · ROOM | `s7_barrier`, `s7_room` (C14/R9) | line + one combined label |
| WATCH marker | `armStore` Arm registry | hollow ▵ amber, keyed to Arm, removed on expiry |
| SETUP / REJECTED marker | `rawObsSet` `Obs.riskPass` | filled ▴ chip at the signal bar |
| Event markers | `s7_domPri` / `s7_domText` per committed bar | one-time marker at the transition bar |
| Evidence glyphs Q1–Q5 | `s7_qGlyph(s7_stripFlag, s7_stripAppl, QB_Q1..Q5)` | strip row, exact order |
| `EVIDENCE n/m` | `s7_qPass` / `s7_qAppl` | strip lead cell |
| RVOL frozen value | `Obs.relVol` → `s7_caseSigRelVol` | `1.32x` |
| MOM frozen value | `Obs.roc5` → `s7_caseSigRoc5` | raw ROC5, `0.0000`, no `%` |
| LOC frozen result | `Obs.zoneIdMask` | zone identity (`PDL+ORL`) |
| FRESH frozen result | `Obs.touchCount` | `TOUCH n` |
| SLOPE frozen state | Q5 evidence result | `ALIGNED` / `NOT ALIGNED` — see FD-2 |
| Comparison DIRECTION | `s7_caseDir` → `structBias` (C17) | transition or UNCHANGED |
| Comparison RVOL / MOM | frozen vs live, same unit | signed numeric delta |
| Comparison VWAP SLOPE | Q5 frozen; `vwapSlopeState` live | CHANGES = N/A — see FD-2 |
| Comparison EVENT | `s7_famText`, `s7_tp1Reached`, `s7_trailUnlocked` | milestone chronology |
| SQUEEZE five states | `s7_sqzText` precedence block (packet §13.2) | own row, own treatment |

---

## 3. CHANGED-HUNK LEDGER — 15 hunks, all classified

| # | Location | Classification |
|---|---|---|
| 1 | `indicator()` line 395 | **Identity** — title/shorttitle IC2→IC3. All four resource caps unchanged. |
| 2 | `S7_BUILD` | **Identity** — build string IC2→IC3. |
| 3 | S7.1a (new) | **Presentation constants** — design colour tokens, `S7_EVENT_MARK_CAP`, `S7_CMP_ROWS`. |
| 4 | S7.2a (new) | **PURE helpers** — `s7_roleText`, `s7_deltaText`, `s7_transText`, `s7_qVerdict`, `s7_ratioText`, `s7_slopeText`. No predicate. |
| 5 | S7.3.5 (new) | **Drawing handles** — 3 tables, 4 lines, 7 labels, 2 marker arrays. Presentation state only. |
| 6 | Evidence strip source | **Frozen-read extension** — captures `relVol`/`roc5`/`zoneIdMask`/`touchCount`/time from the *same frozen record* already read. No new producer. |
| 7 | S7.6a (new) | **Display strings** — card head, warning band, structure glyph, comparison note, Evidence values. |
| 8 | S7.8 header + teardown | **Render lifecycle** — `s7_simple`; teardown extended to the new handles. |
| 9 | S7.8.0 (new) | **Depth-scoped lifecycle** — exactly one depth's surfaces exist at a time. |
| 10 | Zone bands | **Presentation** — SIMPLE uses the approved palette and `PDH · RESISTANCE` naming; ADVANCED keeps IC2 palette and confluence text. |
| 11 | VWAP | **Presentation** — SIMPLE thin line + label; ADVANCED keeps the IC2 band. |
| 12 | Opening range | **Presentation** — SIMPLE thin ORH/ORL lines + labels; ADVANCED keeps the IC2 band. |
| 13 | WATCH markers | **Presentation** — hollow ▵ amber below bar in SIMPLE; ADVANCED unchanged. Guard strengthened for `rthL`. |
| 14 | SETUP markers | **Presentation** — filled ▴ chip in SIMPLE; ADVANCED unchanged. |
| 15 | S7.8.1–S7.8.3 | **Composition** — price geometry + right-edge labels + event markers + the three SIMPLE tables; ADVANCED panel re-nested verbatim. |

**No unrelated source change.** Hunk 1 is the only edit outside the Stage-7 region.

---

## 4. SELF-VERIFICATION RESULTS (mission §7)

| # | Check | Result |
|---|---|---|
| 1 | IC2 parent authenticity | **PASS** — SHA-256, bytes, LF lines recomputed and matched |
| 2 | Stage-6 inherited body preservation | **PASS** — lines 1–5021 excluding the `indicator()` identity line are **byte-identical** (`49a3a701…d5813` both sides) |
| 3 | Exactly 52 exports, title and order | **PASS** — `diff` of extracted titles is empty; count 52 |
| 4 | `request.*` preserved | **PASS** — all 3 calls byte-identical |
| 5 | Zero new decision-bearing write / reverse dependency | **PASS** — all 31 new `:=` targets are drawing handles or display strings; no write to `s7_case*`, `s7_trail*`, `s7_struct*`, `s7_warn*`, `s7_cand*`, `s7_hist`, `s7_evt*`. All new code sits after S7.5, so no management path reads it |
| 6 | Zero new analytical threshold or score | **PASS** — no new numeric comparison against a magnitude; the only new constants are a label-FIFO cap and a table row count |
| 7 | Evidence frozen/live separation | **PASS** — strip values read the frozen record exclusively; live `relVol`/`roc5`/`vwapSlopeState` appear only in SIGNAL CHANGES / SIGNAL STATUS |
| 8 | AT SIGNAL immutability | **PASS** — anchor writes remain exactly the three IC2 sites (reset, case open, Gate-4 reversal hand-off). No new anchor write |
| 9 | No fabricated WATCH plan values | **PASS** — all plan geometry gated on `s7_caseActive`; WATCH draws no INITIAL STOP / TRAIL STOP / TP1, and AT SIGNAL/CHANGES render `—` under `NO SIGNAL SNAPSHOT YET` |
| 10 | No TP2/TP3/TP4 | **PASS** — single occurrence is a comment asserting NEXT BARRIER *is never* TP2 |
| 11 | SQUEEZE preserved as a separate box | **PASS** — own row, own three-level treatment, blank spacer row separating it from the strip; never a Q cell. ADVANCED SQUEEZE row unchanged |
| 12 | ADVANCED not silently removed or redesigned | **PASS** — the 102-line ADVANCED block is identical modulo its +4 nesting indent; all 39 `s7_pushRow` calls byte-identical |
| 13 | Resource counts and declaration caps | **PASS** — see §5; all four caps unchanged, no expansion required |
| 14 | Complete changed-hunk classification | **PASS** — §3, 15 of 15 |
| 15 | No unrelated source change | **PASS** |

Structural lint: zero non-multiple-of-4 indents, zero empty block bodies, zero table-cell
writes outside declared dimensions.

Two defects were found and fixed during self-verification:

- `s7_transText` guarded on `!= 0`, but `DIR_NEUTRAL == 0` — a truthful `LONG → NEUTRAL`
  change would have been suppressed as `—`. Re-guarded on `na()`.
- Event-marker placement keyed off numeric rank, which is non-monotonic (`ACTIVE` 50 sits
  between `WARNING` 45 and `TP1` 55). `ACTIVE` would have rendered above the bar; the approved
  design places it below. The two below-bar milestones are now named explicitly.

---

## 5. BEFORE / AFTER RESOURCE COUNTS

Concurrent runtime objects, worst case, against the **unchanged** declaration caps.

| Resource | Cap | IC2 | IC3 SIMPLE | IC3 ADVANCED |
|---|---|---|---|---|
| `max_lines_count` | 16 | 3 | **7** (IS, TS, TP1, BARRIER, VWAP, ORH, ORL) | 3 |
| `max_labels_count` | 64 | 33 | **51** (watch ≤8 + setup ≤24 + markers ≤12 + price 7) | 33 |
| `max_boxes_count` | 16 | 6 | **4** (zone bands; VWAP/OR bands deleted in SIMPLE) | 6 |
| `max_bars_back` | 300 | 300 | 300 | 300 |
| Tables | — | 1 | 3 | 1 |

Every count is inside its existing cap. **No cap was raised** — the open item BI-3 carried into
implementation is closed.

Static call-site counts: `table.new` 1→4, `table.cell` 2→48, `label.new` 3→11, `line.new` 3→7,
`box.new` 3→3, `plotshape`/`plotchar`/`barcolor`/`hline` 0→0, `bgcolor()` 0→0,
`alertcondition` 0→0, `alert()` 1→1, `input.*` 43→43, `plot()` 52→52.

**Stage-7 input surface remains exactly three** (packet §17.1/§20.2). No new input.

---

## 6. PINE FEASIBILITY DEVIATIONS

Each states the requirement, the limitation, the substitute, and whether meaning changes.

### FD-1 — SQUEEZE and Evidence share one viewport anchor
1. **Requirement:** two separate lower-left components, SQUEEZE above the Evidence strip.
2. **Limitation:** Pine offers exactly 9 table positions; two tables at `bottom_left` overlap.
3. **Substitute:** one `bottom_left` table — SQUEEZE on row 0 with its own three-level
   treatment, a transparent spacer row, then the strip on rows 2–3.
4. **Meaning:** unchanged. SQUEEZE keeps a distinct surface, is never a sixth cell and never
   becomes Q6. Reading order, spatial region and separation are preserved. Resource counts
   unaffected (one table instead of two).

### FD-2 — No frozen signal-time VWAP-slope value exists
1. **Requirement:** the design shows `AT SIGNAL · VWAP SLOPE = RISING` and `UNCHANGED`.
2. **Limitation:** the `Obs` record freezes `relVol`, `relVolCum`, `roc5`, `atrPct`,
   `atrRatio`, `adxVal`, `efficiencyRatio` — **no slope field**. `vwapSlopeState` is live only.
   Rendering Q5 as `RISING` would be an invention (Q5 pass on a SHORT case means falling).
3. **Substitute:** AT SIGNAL shows the frozen Q5 evidence **state** (`ALIGNED` / `NOT ALIGNED`
   / `—`); CHANGES stays `—` because no same-unit pair exists; STATUS shows live
   `RISING`/`FALLING`/`FLAT`.
4. **Meaning:** narrower than the mock, but truthful. This is the branch mission §4.5 already
   prescribes. Creating the field would be a new producer, prohibited by §5.

### FD-3 — LOC and FRESH cell wording
1. **Requirement:** the design mock shows `IN ZONE` and `LATE`.
2. **Limitation:** no accepted producer owns those words; `LATE` in particular implies an age
   or threshold the source does not define.
3. **Substitute:** the accepted underlying results — zone identity (`PDL+ORL`) and `TOUCH n`.
4. **Meaning:** same fact, source-owned wording. No numeric location score and no freshness age
   is invented.

### FD-4 — No inline mixed styling inside a card row
1. **Requirement:** the card head pairs a 20px/700 title with a smaller tag chip; the meta row
   mixes weights and colours inline.
2. **Limitation:** a Pine table cell carries one text colour and one text size.
3. **Substitute:** three single-column rows — head at `size.large`, warning band, meta row at
   `size.small` with `│` separators; per-row background and colour carry the hierarchy.
4. **Meaning:** unchanged; salience hierarchy preserved, inline typographic contrast lost.

### FD-5 — No right-edge label de-collision, no label borders
1. **Requirement:** labels sort to ≥16px separation and jog inward past the comparison table;
   chips are white with 1px borders.
2. **Limitation:** Pine labels are price-anchored with no measurable height and no border.
3. **Substitute:** one label per reference at its own price, anchored at the session's right
   edge; borderless with a light chip background for the marker chips.
4. **Meaning:** unchanged. Labels at near-identical prices may visually crowd — a legibility
   deviation, not a truth deviation. Also: the current-price rail tag is drawn natively by
   TradingView and was **not** re-implemented.

None of FD-1…FD-5 changes visibility of any accepted fact or raises a resource cap.

---

## 7. UNRESOLVED BOUNDED ISSUES — DISPOSITION REQUIRED

### BI-6 — Material authority conflict: Arabic in the Evidence Strip *(this is why the verdict is B)*

Surfaced unresolved per Mission §2: *"Do not silently resolve a material conflict between these
authorities."*

- **Final UX v1.2 §37:** English first, Arabic immediately below — *"This rule is **mandatory**
  for the five Evidence cells."* §42.4 makes it a binding acceptance criterion: *"Arabic meaning
  is immediately available under each label."* §42 opens: *"An implementation conforms to this
  design **only if all** of the following are true."*
- **Mission §4.5 delta and the approved canvas:** both remove Arabic from inside the compact
  strip. The rendered design carries English cell names and frozen values only.

**As implemented:** the delta and the canvas were followed — no Arabic inside the strip. Arabic
is retained in the card (`الهيكل قائم`, `غير متاح`), the SQUEEZE box, and the ADVANCED panel.

**The conflict stands:** by the controlling semantic document's own acceptance test, IC3 does
not satisfy §42.4. Claude will not silently narrow a criterion Final UX calls mandatory.

**Select one:**
- **(a)** Arabic is satisfied elsewhere (tooltip or ADVANCED); §37/§42.4 read as *available*,
  not *inline*. *(Name the carrying surface and Claude will add it.)*
- **(b)** Amend §37/§42.4 under change control to match the approved design. *(No code change.)*
- **(c)** Restore inline Arabic in the strip — **contradicts the approved canvas** and requires
  a design revision.

### BI-7 — Optimized Execution Packet not supplied as a document

Authority 3 was never provided. IC2's Stage-7 section is written against it by number (§3, §5,
§6.1/6.2/6.4, §11.1, §13.2, §14, §15A/§9.3, §16, §17.1/§20.2, §21). Implementation treated
those inline references as the binding invariants and preserved every one it names — notably
the three-input surface and the product firewall. **If the standalone packet carries
constraints not reproduced in those comments, supply it for re-verification.**

### BI-8 — Wording discrepancy in the PROCEED instruction *(no action taken)*

The instruction lists *"Evidence strip in the approved upper-center position"*. In the approved
design the strip is at **lower-left** (`left:56px; bottom:8px`), beneath the SQUEEZE box; the
top-centre position is occupied by the primary case card, so an upper-centre strip would
collide with it. Since the same list is framed as *"preserve the approved visual architecture"*
and the design artifact is the named visual authority, the strip was implemented **lower-left
per the artifact**. Flagged rather than silently reconciled. Confirm, or state the intended
change and it will be applied.

FD-2 (no frozen slope) is disclosed rather than escalated — mission §4.5 already prescribes
that fallback.

---

## 8. PRESERVATION RESULTS

- **Stage-6:** lines 1–5021 byte-identical excluding the `indicator()` identity line. No
  calculation, predicate, threshold, Direction authority, Risk rule, Observation, Evidence
  applicability, TRAIL/STOP rule, reversal rule or Forward Research path touched.
- **52 exports:** identical in count, title and order.
- **`request.*`:** 3 calls, byte-identical. No new call; `vwap`, `rthH`, `rthL`, `rthC` come
  from the existing `coreBundle` destructuring.
- **Alerts:** `alertcondition` 0→0, `alert()` 1→1, message body and eligibility untouched.
- **ADVANCED:** full IC2 panel rebuilt verbatim; all 39 `s7_pushRow` calls identical; no
  ADVANCED producer, state, diagnostic or display path removed. The only ADVANCED-visible
  change is that it now renders exclusively when the depth input selects it.
- **Underlying-only disclosure:** `Underlying only · لا بيانات عقود` semantics untouched.

---

## 9. MISSION-BOUNDARY COMPLIANCE

Not performed: compile, load, TradingView execution, runtime/evidence/screenshot/CSV
collection, source acceptance, ADVANCED redesign, Stage-6 change, Stage-8 work, Gate 1–5
reopening, additional reviewer launch, design reconstruction from memory. No new decision
logic, threshold, score, confidence, Trade Health, Q6, TP2/TP3/TP4, institutional inference,
broker/execution semantics, input, `request.*` call, plot export, `alertcondition`, or
drawing/resource-cap expansion.

The headless-browser render was performed on the **design artifact only**, to establish that
the visual authority was genuinely readable. No Pine was compiled, loaded or executed.

---

## 10. RETURN VERDICT — FINAL

```
B.  CLAUDE PARTIAL SIMPLE IMPLEMENTATION —
    BOUNDED FEASIBILITY OR AUTHORITY ISSUE REQUIRES PRODUCT OWNER DISPOSITION
```

All approved SIMPLE elements are implemented and statically verified. Outstanding:
**BI-6** (authority conflict — needs a decision), **BI-7** (packet never supplied),
**BI-8** (instruction wording vs artifact). FD-1…FD-5 are disclosed feasibility substitutions,
none of which changes meaning, visibility or resource counts.

IC3 is returned for **independent static verification**.

---

**END OF RETURN — OSE-S7-CLAUDE-IC3-SIMPLE-VISUAL-IMPLEMENTATION-01**
