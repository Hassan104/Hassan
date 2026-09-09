# OSE S7 — IC3R5 SURGICAL COMPILE-CORRECTION RETURN v1.0

**Mission:** continuation from IC3R4, correcting three observed TradingView compile defects.
**Executor:** Claude, Development Owner · **Date:** 2026-09-09

## Parent authenticated

`OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3R4.pine` — SHA-256
`79ee0fa053dd5e8495696e36ae31f62540e221e370bcc35a7bb50e6ba94464b9`, 446,668 bytes, 7,455 LF lines.
Preserved unchanged throughout this correction.

---

## C1 — duplicate identifier `s7_candTime`

**Root cause.** Two unrelated declarations shared the name:
- `var int s7_candTime = na` (pre-existing, part of the DIRECTION CRITICAL reversal-candidate
  tuple alongside `s7_candLive`, `s7_candObsId`, `s7_candDir`, `s7_candFamily`, `s7_candOrd`,
  `s7_candObsRef`) — 6 sites total.
- `var array<int> s7_candTime = array.new_int()` — the IC3R4 R1 candidate-pool array, one of the
  parallel arrays (`s7_candPri`, `s7_candTime`, `s7_candConf`, `s7_candTxt`, `s7_candCasA`,
  `s7_candCasB`) backing the bounded top-K event archive.

**Correction.** Renamed **only** the new array and its exact 3 usage sites to `s7_candBarTimes`:
declaration, the `array.get` read in the rollover promotion loop, and the `s7_candPush` call.
The pre-existing scalar and all 6 of its sites (declaration, three session/candidate resets, the
`s7_idOk` reversal-identity check, and the reversal-clear) are byte-for-byte untouched. No global
textual replacement was used — each of the 4 renamed lines was located and edited individually,
verified against the pre-existing scalar's sites before and after.

**Disposition: PASS.** `grep -c "s7_candTime\b"` returns exactly 6 code sites (plus 2 descriptive
mentions in this correction's own manifest comment) — none is a duplicate declaration.

## C2 — dependent array-type error

**Root cause.** `array.get(s7_candTime, s7_i)` resolved against the wrong (scalar) declaration
once C1's collision existed — a direct consequence of C1, not an independent defect.

**Correction.** Closed by the same rename: the call now reads `array.get(s7_candBarTimes, s7_i)`,
which type-checks against the array declaration. Every array operation on the candidate-pool
arrays (`array.get/push/set/clear/size` across `s7_candPri/BarTimes/Conf/Txt/CasA/CasB`) was
re-audited after the rename; none references a scalar, and the scalar `s7_candTime` is never
passed to an `array.*` call anywhere in the file.

**Disposition: PASS.**

## C3 — main body of script too long

**Root cause.** IC3R4 added roughly 700 executable lines (R1 archive/rollover/segment tracking
and chart-object management) directly in the top-level script body, on top of IC3R2's earlier
`s7_renderSimple`/`s7_renderAdvanced` extraction. That pushed the compiled main-body size back
over TradingView's limit.

**Correction — bounded function extraction, two functions, chosen for zero semantic risk:**

| Function | Extracted from | Lines moved | Touches |
|---|---|---|---|
| `s7_rolloverArchive()` | S7.5.0 session-boundary reset — the R1 rollover/archive/promotion sequence only | 102 | archive arrays, archive drawing handles (labels/boxes/lines) |
| `s7_renderChartObjects()` | S7.8.1 chart-object rendering (zones, VWAP, OR, WATCH/SETUP/event markers, protection lines) | 412 | drawing handles only (tables untouched — still rendered by the existing `s7_renderSimple`/`s7_renderAdvanced`) |

Both are presentation/object-management blocks, matching the preferred extraction category.
**Neither touches case election, Direction, Risk, trail-stop, structure, or any other
decision-bearing management state** — confirmed by diffing the complete set of
`s7_case*`/`s7_trail*`/`s7_struct*`/`s7_warn*`/`s7_cand*` assignment statements between IC3R4 and
IC3R5: **identical, zero difference**. The actual S7.5 management engine (case election,
DIRECTION WARNING/CRITICAL/REVERSED, TRAIL ON/ratchet, STOP TEST/BREAK, reversal, session-end) was
left exactly where it was, per the mission's own preference for leaving decision-bearing mutation
in the main body.

**Pine safety, checked, not assumed:**
- Every value either function writes is a pre-existing `var`-declared drawing handle, archive
  array, or archive counter — Pine permits a function to reassign a `var` global. Verified by
  cross-referencing every `:=` target inside both bodies against its `var` declaration (three
  apparent exceptions — `s7_bx`, `s7_li`, `s7_still` — are declared and reassigned entirely
  within the same loop iteration, i.e. genuinely local, unaffected by extraction).
- Neither function contains a `plot*`, `hline`, `fill`, or `alertcondition` call (none existed in
  either source block).
- Each function is called **exactly once**, at the identical point in program order its body
  previously executed inline — confirmed by grep count and by checking the one distinctive line
  from each block appears exactly once in the file (no duplication, no loss).
- Every external identifier either function references was checked to be declared earlier in the
  script than the function definition (129 identifiers audited for the chart-objects function, 57
  for the rollover function — all resolved; three initial "not found" hits were a regex
  limitation against tuple-destructuring syntax, confirmed by direct lookup to be declared at
  line 622).
- Function definitions are placed as top-level statements immediately before their single call
  site (`s7_rolloverArchive()` defined right after `int s7_caseIdPre = s7_caseId`, before the S7.5
  banner; `s7_renderChartObjects()` defined right after `bool s7_simple = not s7_advanced`, before
  the S7.8 dispatch) — matching the codebase's existing convention for `s7_renderSimple` /
  `s7_renderAdvanced`.

One indentation bug was caught and fixed during self-verification: the rollover extraction's
initial dedent left its body at 8 spaces instead of the correct 4 (a double-indent artifact of
the extraction script). Found by inspection immediately after the mechanical edit, corrected
before any other work, and re-verified with a full structural lint (zero indent anomalies outside
the five pre-existing Stage-6 tuple-continuation lines at 612–616, zero empty block bodies).

**Disposition: HOLD — PRODUCT OWNER TRADINGVIEW CHECK REQUIRED.** Two self-contained blocks
totalling 514 lines were moved out of the main body; whether this is *sufficient* to clear
TradingView's specific size threshold cannot be verified without the compiler. If insufficient,
the next-largest self-contained candidate is the SIMPLE-surface string computation in S7.6/S7.6a,
not yet extracted here because it produces ~30 values consumed later by `s7_renderSimple`, which
would need a return-tuple — higher risk, deliberately not attempted in this bounded pass.

---

## Newly created functions (C3)

1. `s7_rolloverArchive()`
2. `s7_renderChartObjects()`

## No approved product/UI semantics were changed

Confirmed by direct diff: Stage-6 inherited body byte-identical to IC2 (excluding the declaration
identity line); 52 exports identical in title and order; all 3 `request.*` calls identical; 43
inputs / `alertcondition` 0 / `alert()` 1 unchanged; all four declaration caps unchanged; all 39
ADVANCED panel rows byte-identical; the complete management-state assignment set identical to
IC3R4. The only content differences between IC3R4 and IC3R5, verified by a full sorted-line diff,
are: the 4 renamed identifier sites, the 2 new function-definition headers and their 2 call sites,
this correction's manifest comment, and the identity strings (title/build/END marker).

## Corrected source

| Property | Value |
|---|---|
| Filename | `OSE-S7-DECISION-SUPPORT-UI-v1.0.0-IC3R5.pine` |
| SHA-256 | `704f58fa51b74156c7c791c47c54d5609a23f4f6dd078f4208a8686dda06a51f` |
| Bytes | 448,448 |
| LF lines | 7,493 |
| Terminates with newline | yes (`wc -l` 7,493 == `awk END{NR}` 7,493) |
| CR bytes | 0 |
| Declared shorttitle | `OSE-S7` |

## Compiler status

```
COMPILE NOT EXECUTED — PRODUCT OWNER TRADINGVIEW CHECK REQUIRED
```

This environment has no Pine compiler. Every claim above is static source inspection and
mechanical verification (grep-based cross-reference, structural lint, sorted-diff comparison,
declaration-before-use audit) — none of it is a compile result, and none is presented as one.

---

## Final disposition

```
IC3R4 SURGICAL COMPILE CORRECTION COMPLETE — RETURNED FOR PRODUCT OWNER COMPILE CHECK
```
