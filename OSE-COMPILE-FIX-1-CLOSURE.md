# OSE-CANONICAL-REBALANCE-01 / COMPILE-FIX-1

Status: **READY FOR CORE RECOMPILE** · no compile pass is claimed.

| File | SHA-256 |
|---|---|
| `OSE-CORE-LIB-v1-TEST-ONLY.pine` (corrected) | `c733b29dea32c62bacd06d595804b92d78bcdf65cef970945f744b8f99c4b62f` |
| `OSE-C1-MAIN-CANDIDATE-TEST-ONLY.pine` (corrected) | `4af997623b992bbe3d269812484464e9d1937494e1dcbbbae4f54816ab5b40dc` |
| `OSE-RF1-COMPANION-LIB-v2-TEST-ONLY.pine` | unchanged — **do not republish** |

---

## A. ROOT-CAUSE SUMMARY

**One defect, eight instances.** My extraction scanner excluded every **UDT field name** from the set of "external identifiers a relocated function reads". The intent was to ignore named arguments such as `Zone.new(mask = m, …)`. The effect was that any Main global **sharing a name with a UDT field** was silently deleted from every dependency closure, so it never became a parameter and never existed inside the library.

Seven such collisions exist in this source — all of them are simultaneously an `Obs` field **and** a top-level engine global:

`roc5` · `relVol` · `relVolCum` · `adxVal` · `efficiencyRatio` · `vwapPriceTouchCount` · `vwapZoneMotionTouchCount`

(plus `s7_structText`, dropped by the same named-argument filter).

**Why the previous verifier missed it:** it was built from the *same* flawed assumption — it reused the identical exclusion list, so it agreed with the generator instead of checking it. Worse, it also matched `library("OSE_CORE_LIB", …)` as a function definition and then scanned forward to the first `=>`, swallowing the entire `const` and `type` block; that made every constant look "declared" no matter what. Two independent bugs that happened to cancel into a clean bill of health.

**The fix, structurally:** dependency detection is now **positional, not name-based**. Named arguments are removed by matching `name =` only where it directly follows `(` or `,`; nothing is ever excluded because of what it is *called*. A global can no longer be lost by colliding with a field name.

**Verifier now self-tested.** Run against the exact library TradingView rejected, it reports precisely the compiler's findings — including `roc5` at line 890:

```
OLD-CORE-BROKEN.pine: 57 functions scanned
UNRESOLVED IDENTIFIERS: 8
   line   890  in v4_matrixTruth   -> adxVal
   line   890  in v4_matrixTruth   -> relVol
   line   890  in v4_matrixTruth   -> roc5
   line  1005  in v4_liveBand      -> adxVal
   line  1006  in v4_liveBand      -> relVol
   line  1007  in v4_liveBand      -> relVolCum
   line  1053  in v4_workspace     -> efficiencyRatio
   line  1168  in v4_makeTopBar    -> s7_structText
```

A verifier that cannot fail on known-bad input proves nothing; this one now does.

---

## B. ERROR-CLOSURE MAP

| Class | Count | Detail |
|---|---:|---|
| **Root — undeclared identifier (CE10272)** | **8** | the 8 listed above, in 4 functions |
| **Cascade — `array.from` element-type inference** | ~5 | CE10123 `rf1_outRsi … "unknown" is expected`. `rf1_outRsi` **was already a correct parameter**; once `roc5` in the same `array.from` is undeclared, Pine cannot infer the element type and reports every later argument against `unknown`. |
| **Cascade — ternary operand inference (CE10122)** | ~1 | `call "operator ?:" (unknown)` vs `series string` — a ternary whose operand referenced an undeclared identifier. |
| **Cascade — array assignment (CE10173)** | ~1 | `array<unknown>` → `array<color> vp_cI`: the constructor above it had already collapsed to `unknown`. |
| **Environmental — not a code defect** | 1 | `The user 'YOUR_TV_USERNAME' does not have a published library titled 'OSE_RF1_COMPANION'` — the import placeholder was never substituted. See §G step 0. |

Consistent with ~16 reported problems from 8 real defects plus one unsubstituted placeholder. **No genuine type defect was found:** every type error dissolved once the undeclared identifiers were resolved, so nothing was cast or coerced.

---

## C/D. ARTIFACTS

Complete corrected `OSE_CORE_LIB` and C1 Main, no fragments.

---

## E. INTERFACE CHANGE MANIFEST

Only four exports changed; each gained the parameters that were wrongly dropped. Types were read from each global's own declaration, not guessed.

| Export | Added parameters | Params before → after |
|---|---|---:|
| `v4_matrixTruth` | `float adxVal`, `float relVol`, `float roc5` | 11 → **14** |
| `v4_liveBand` | `float adxVal`, `float relVol`, `float relVolCum` | 40 → **43** |
| `v4_workspace` | `float efficiencyRatio` | 41 → **42** |
| `v4_makeTopBar` | `string s7_structText` | 24 → **25** |

All 52 other exports are unchanged. Parameter **order** is deterministic (alphabetical after `Theme TH`), and the six matching C1 call sites were regenerated from the same table, so signature and call can not drift apart.

No new library global was created; no mutable canonical state was duplicated; no lifecycle authority moved.

---

## F. SEMANTIC NON-INTERFERENCE CHECK

1. **All 56 relocated bodies remain byte-identical** to C0 (automated diff, after normalising the two documented transformations: the `Theme` prologue and appended arguments). The correction added parameters only — **not one line of body text changed**.
2. The added parameters carry the **same values the functions previously read as globals**, passed from the same Main scope.
3. **Declaration-order verified**: all 6 call sites checked; no appended argument is used before its Main declaration, and `v4_TH` (line 5045) precedes its earliest use (line 6948).
4. **Scope-resolved clean** by the self-tested verifier: CORE `NONE`, C1 Main `NONE`, Companion `NONE`.
5. **Contracts unchanged**: 52 plots, 3 `request.security`, 1 `request.security_lower_tf`, 95 inputs, 1 alert, and the 52 export titles and order all identical to C0.
6. No canonical calculation, decision, observation, queue/funnel, case, ACTIVE, Structure Lost, stop, TP, trail, reversal, alert or export was touched. No further extraction was performed (§7).

C1 Main is 28,779 source tokens (+16 vs the rejected build, for the 8 restored arguments). Projection against the measured anchor is unchanged in substance: **≈83,800 compiled, ≈16,450 headroom**. Still a projection, not a result.

---

## G. NEXT USER ACTION

0. **Substitute your TradingView username first.** Both import lines in the C1 Main still read `YOUR_TV_USERNAME`; that is what produced the "does not have a published library" error. Replace it in **both** lines.
1. Open the existing `OSE_CORE_LIB` script and **replace its full source** with the corrected `OSE-CORE-LIB-v1-TEST-ONLY.pine`.
2. **Publish script → Update** the private library, then note the version number TradingView assigns.
3. Bind the exact versions in the C1 Main:
   ```
   import <your_username>/OSE_RF1_COMPANION/2 as rf1lib
   import <your_username>/OSE_CORE_LIB/<version> as core
   ```
   Companion stays at **v2, unchanged** — do not republish it.
4. Compile the C1 Main.
5. If anything still fails, return the **first** remaining compiler error verbatim with its line number.

---

## STATUS

**READY FOR CORE RECOMPILE**

TradingView compile/load NOT RUN. No compile pass claimed. No Black Box, no parity run, no further extraction.
