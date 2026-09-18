# OSE-CANONICAL-REBALANCE-01 — C1 CANDIDATE

Status: **READY FOR LIBRARY PUBLISH + C1 COMPILE** · C1 is a **candidate only**, not an approved baseline.

---

## A. SOURCE AUTHENTICATION (C0)

| Role | File | SHA-256 |
|---|---|---|
| C0 Main | `OSE-PRE8-RF1-R6-R7-INTEGRATED-v0.1.4-LIB-TEST-ONLY.pine` | `65c1250c0563b2d8493dbe85feb7fe93db8abb924c02e5dd1b213e07cdad5e03` |
| C0 Companion | `OSE-RF1-COMPANION-LIB-v2-TEST-ONLY.pine` | `f651d4815d9b6d36e03135ef7d3d87377d497e1eb34613f998e02193c213929b` |

Both re-hashed before mutation and **left byte-untouched**. No Black-Box / Track-A / Track-B artifact was read or used as source authority.

**Outputs**

| File | SHA-256 |
|---|---|
| `OSE-CORE-LIB-v1-TEST-ONLY.pine` | `97cf051d48e46ddbe61dc05fd9d09ca38335aa33af927e2f581ab5741d589f5e` |
| `OSE-C1-MAIN-CANDIDATE-TEST-ONLY.pine` | `3b76481f7362111b8835e7fa67a58cc3fd04fde3f29e67c33a67c65e99777289` |

---

## B. FINAL PARTITION ARCHITECTURE

```
CANONICAL MAIN (C1)
├── OSE_RF1_COMPANION  v2   UNCHANGED   — Profile/Delta compute core
└── OSE_CORE_LIB       v1   NEW         — canonical types + deterministic compute + presentation
```

**Two libraries, not three.** One additional library was sufficient: the extracted material forms a single dependency chain (types → kernels → vocabulary → salience → renderers) where the renderers consume every layer below. Splitting it would have forced a library-to-library import and version coupling for no token benefit. Companion stays untouched per §8; where CORE needed `vp_zoneText`, it carries a **private copy** (library tokens are free) rather than importing Companion.

---

## C. EXTRACTION SUMMARY

| Category | Count | Examples |
|---|---:|---|
| Canonical record types | 10 | `Zone`, `Arm`, `Obs`, `V4Event`, `V4Range`, `V4Level`, `V4Candle`, `V4Watch`, `V4Coverage`, `V4PriceRef` |
| Pure scalar/geometry kernels | 20 | `hasBit`, `dirDistOf`, `distToZone`, `sideOf`, `zoneFarOf/NearOf`, `closeLocOf`, `dirWorstStrict/Accum`, `armIdxOfKey`, `zoneIdxOfMask`, `flagSetOf`, `dnumFromStr`, `fillDateList`, **`buildZones`** |
| Vocabulary / text helpers | 15 | `s7_dirText`, `s7_famText`, `s7_priceText`, `s7_hhmm`, `vp_dir`, `vp_fam`, `vp_entry`, `vp_riskReason` |
| VP-07 salience bands | 13 | `vp_mix`, `vp_a14Of`, `vp_adxTint/Bg`, `vp_c19Tint/Bg`, `vp_c60Tint/Bg`, `vp_atrTint/Bg`, `vp_erTint/Bg` |
| Presentation computation | 8 | `v4_workspace`, `v4_liveBand`, `v4_matrix`, `v4_matrixTruth`, `v4_makeTopBar`, `v4_panelCell`, `opt_cell`, `v4_eventSummary` |

**Deliberately NOT moved:** `coreBundle()` and `adxErBundle()` (they are `request.security` callees — §17); the whole record/capture layer (`v4_capture*`, `v4_store*`, `v4_trim*`, `v4_prune*`); the whole draw layer (`v4_draw*`, `v4_marker`, `opt_line`, `opt_label`); all engine lifecycle, Queue/Funnel, election, case/ACTIVE/stop/trail/reversal mutation; all exports and the alert.

---

## D. MIGRATION MANIFEST (compact)

Every row: old Main definition → `OSE_CORE_LIB` export of the same name. All bodies byte-identical (see §I).

| ID | Old block | New export | Extra inputs | Relief |
|---|---|---|---|---|
| T01–T10 | 10 `type` declarations | `export type` ×10 | — | MEDIUM (enabler) |
| K01–K20 | §13.5/§19 pure helpers + `buildZones` | same names | **none** | HIGH |
| V01–V15 | vocabulary helpers | same names | none | MEDIUM |
| S01–S13 | VP-07 band helpers | same names | `Theme` | MEDIUM |
| R01 | `v4_panelCell` | `v4_panelCell` | `v4_font` | MEDIUM |
| R02 | `opt_cell` | `opt_cell` | `v4_font` | LOW |
| R03 | `v4_matrix` | `v4_matrix` | `Theme` + 6 | MEDIUM |
| R04 | `v4_matrixTruth` | `v4_matrixTruth` | `Theme` + 10 | HIGH |
| R05 | `v4_liveBand` | `v4_liveBand` | `Theme` + 39 | HIGH |
| R06 | `v4_workspace` | `v4_workspace` | `Theme` + 40 | HIGH |
| R07 | `v4_makeTopBar` | `v4_makeTopBar` | `Theme` + 23 | HIGH |

- **Mutable-state dependencies:** none. Statically verified that **no relocated function writes any global** (see §I).
- **Request dependencies:** none relocated, none duplicated.
- **Parity surface:** every table cell text/colour/tooltip, the Top Bar strip, the signal matrix, the live band, the workspace cards, zone geometry from `buildZones`, and the RVOL median.

---

## E/F. ARTIFACTS

- `OSE-CORE-LIB-v1-TEST-ONLY.pine` — complete, publish-ready, 46 exports.
- `OSE-C1-MAIN-CANDIDATE-TEST-ONLY.pine` — complete, no diagnostics, compile-ready after import binding.

---

## G. IMPORT / PUBLICATION INSTRUCTIONS

1. **Publish `OSE-CORE-LIB-v1-TEST-ONLY.pine` FIRST.** Pine Editor → Publish script → **Publish new library** → **Private**. Declaration is `library("OSE_CORE_LIB", overlay = false)`.
2. **Companion: do nothing.** `OSE_RF1_COMPANION` **v2 is unchanged** and is still the dependency.
3. In the C1 Main, bind both imports (replace `YOUR_TV_USERNAME`, and the version number if TradingView assigns something other than 1):
   ```
   import YOUR_TV_USERNAME/OSE_RF1_COMPANION/2 as rf1lib
   import YOUR_TV_USERNAME/OSE_CORE_LIB/1 as core
   ```
4. Compile `OSE-C1-MAIN-CANDIDATE-TEST-ONLY.pine`.

---

## H. RESOURCE COMPARISON

| Metric | C0 Main | C1 Main | Δ | CORE_LIB | COMPANION |
|---|---:|---:|---:|---:|---:|
| bytes | 462,266 | 424,309 | −37,957 | 61,867 | 20,761 |
| lines | 7,791 | 7,048 | −743 | 1,222 | 468 |
| executable lines | 3,903 | 3,246 | −657 | 997 | 374 |
| source tokens | 35,657 | 28,763 | **−6,894** | 9,271 | 3,127 |
| plots | 52 | 52 | 0 | 0 | 0 |
| `request.security` | 3 | 3 | 0 | 0 | 0 |
| `request.security_lower_tf` | 1 | 1 | 0 | 0 | 0 |
| inputs | 88 | 88 | 0 | 0 | 0 |
| alerts | 1 | 1 | 0 | 0 | 0 |
| `table.new` | 3 | 2 | −1 | **+1** | 0 |
| box/line/label `.new` | 3/1/1 | 3/1/1 | 0 | 0 | 0 |
| local functions | 108 | 52 | −56 | 57 | 11 |
| local types | 10 | 0 | −10 | 11 | 3 |
| library count | 1 | **2** | +1 | | |

`table.new` totals 3 across Main+CORE, unchanged — one moved with `v4_makeTopBar`.

**Compiled-token expectation (projection, NOT a result).** Anchored on the only measured datapoint: TradingView reported **100,417** for v0.1.3-LIB at 36,163 source tokens, and the measured marginal ratio on this codebase is **2.25 compiled per source token**. C1 Main is 28,763 source tokens, i.e. −7,400 from that anchor:

> **C1 projected ≈ 83,800** against the 100,256 ceiling → **≈16,500 headroom**, and **≈9,700 still spare after a ~6,800-token Black Box**.
> At ratio 2.0 the spare after Black Box is still ≈7,900.

No successful compiled-token count is invented; TradingView does not report one on success.

---

## I. SEMANTIC NON-INTERFERENCE REPORT

1. **All 56 relocated bodies are byte-identical** to C0, verified by automated diff after normalising exactly two documented transformations:
   - theme colours arrive as one `Theme` object and a prologue re-binds them to their **original names** (`color s7c_mut = TH.s7c_mut`), so body text is untouched;
   - calls to relocated helpers that gained parameters have those arguments appended.
2. **No relocated function writes any global** — statically verified across the whole move set. Everything moved is read-only with respect to canonical state.
3. **No canonical state, Observation/queue/funnel mutation, election, case/ACTIVE/Structure-Lost/stop/TP/trail/reversal transition, alert or export was relocated.** Lifecycle ordering in Main is unchanged; the relocated calls sit at exactly their former positions.
4. **Types keep one identity**: Main now uses `core.Obs`, `core.Zone`, … so the engine and library share the same declarations, with field names, order and defaults copied verbatim.
5. **Request topology untouched**: 3 + 1 requests, same expressions, same tuple positions; `coreBundle()`/`adxErBundle()` stayed in Main precisely because they are request callees.
6. **Constants** are a verbatim copy of the Main `const` block, so every literal a relocated function reads is value-identical.
7. `buildZones` sorts caller-owned arrays in place; Pine arrays are references, so relocation preserves that mutation exactly.
8. **Static checks all pass**: no un-prefixed call or type usage left in Main, no leftover definitions, no unresolved identifier in the library, every library function defined before use, 52 export titles and order identical, `indicator()` byte-identical.

Static checks do **not** establish compile or runtime equivalence.

---

## J. PARITY TEST PLAN (smallest sufficient C0 ↔ C1 sequence)

Same symbol, same timeframe, same inputs, same bar range, both charts loaded simultaneously.

1. **Cold load, historical only** — compare all 52 data-window exports bar-for-bar on 3 sampled RTH sessions (one NORMAL, one early-close, one with missing slots).
2. **Direction / setup / observation** — `obsId`, `obsBarTime`, `classCode`, `decisionCode`, `riskPass`, `rejectReason` on every drained bar.
3. **Queue / funnel** — all counters and flags at session end.
4. **Election / case** — Case ID, ACTIVE transitions, Structure Lost, planned stop, TP1, trail version and reason, reversal, case ending reason.
5. **Alerts** — fire the same ALERT MODE on both; compare payload text and bar timing.
6. **Presentation** — Top Bar 14 cells, live band 11 cells, signal matrix 5×4, workspace 15 cards: text, colour, tooltip, FULL vs A-ONLY, overlay OFF/GHOST/FOCUS, HISTORY on/off.
7. **Geometry** — zone bands, price references, markers, VWAP, R7 histogram POC/VAH/VAL.
8. **Realtime** — run both live for one session; confirm identical confirmed-close refresh behaviour and no drift after reload.

Any single mismatch ⇒ C1 rejected, return to C0.

---

## K. CHANGE REGISTER (recorded, NOT fixed)

| # | Finding |
|---|---|
| CR-01 | `v4_panelCell` deliberately omits cell tooltips (comment introduced in v0.1.2). Pre-existing UI-information loss relative to its parent; not reinstated here. |
| CR-02 | R5 surface values declared but never read anywhere: `rf1_outDeltaResidual`, `rf1_outDeltaAvailable`, `rf1_outRsiAvailable`, `rf1_outMacdAvailable`, `rf1_outDiAvailable`, `rf1_outDeltaCommitted`, `rf1_outSessionPositive/Negative/Unclassified`, `rf1_outSignalQuality`, `rf1_outParentTime`, `rf1_outSessionDate`. |
| CR-03 | Six colour constants declared and never read: `s7c_trail`, `s7c_warnBd`, `s7c_panel`, `s7c_hair`, `s7c_supFil`, `s7c_resFil`. |
| CR-04 | `s7_qAppl` is unpacked from the CHECKS count but never read downstream. |
| CR-05 | **C0 Main (v0.1.4) was never compile-confirmed.** Its 99,278 figure was a projection. The C1 projection above deliberately avoids that link by anchoring on the measured v0.1.3 result instead. |
| CR-06 | `v4_workspace` reaches ~41 parameters. Within Pine's practical range as far as is known, but it is the most likely place for a compiler objection; if rejected, the fix is to bundle the engine readouts into a second UDT exactly as `Theme` does. |

---

## L. RECOVERY POINT (untouched)

- `OSE-PRE8-RF1-R6-R7-INTEGRATED-v0.1.4-LIB-TEST-ONLY.pine` + published `OSE_RF1_COMPANION` **v2** — the C0 pair, byte-unchanged in the repo.
- `OSE-PRE8-RF1-R6-R7-INTEGRATED-v0.1.3-LIB-TEST-ONLY.pine` + Companion **v1** — the earlier measured pair, also retained.
- Nothing was republished over an existing accepted library version. `OSE_CORE_LIB` is a **new** library; Companion v2 is not modified.

---

## M. NEXT USER ACTION

**Publish `OSE-CORE-LIB-v1-TEST-ONLY.pine` as a new PRIVATE library.** Then bind the two imports in the C1 Main and compile it. If CE10117 returns, send the exact token count and I will run another bounded extraction pass (the record layer and the draw layer are measured and reserved: ≈2,038 and ≈2,704 source tokens respectively).

---

## N. FINAL STATUS

**READY FOR LIBRARY PUBLISH + C1 COMPILE**

C1 is a candidate. TradingView compile/load **NOT RUN**; C0 ↔ C1 parity **NOT RUN**. No approval is claimed and the migration is not closed.
