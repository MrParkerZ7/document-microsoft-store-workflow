# Review — Iteration 1

**Date**: 2026-04-17
**Reviewers**: QA Engineer + UX/UI Reviewer
**Diagrams reviewed**: 9

## Review Methodology

Applied 3-layer verification per `PROMPT_VERIFY.md`:
- **L1 Format**: DrawIO XML validity, boilerplate, style compliance, shadow, contrast
- **L2 Content**: Matches 1-analysis/ intent
- **L3 Alignment**: Traces to 0-requirement/

## Findings

### FIND-01 [MEDIUM] — D3 (exe-workflow): Step 17 overflows End User lane

- **Location**: `03-workflow-exe-path.drawio`, cell `s17` at inner y=1560
- **Issue**: Step 17 box (h=60) at y=1560 extends to y=1620, but parent End User lane height is 1560
- **Impact**: Box visually sticks out below the lane container
- **Fix**: Move s17 from y=1560 to y=1470

### FIND-02 [MEDIUM] — D8 (trust-identity): Last steps (m8, e8) overflow containers

- **Location**: `08-trust-identity-flow.drawio`, cells `m8` and `e8` at inner y=690
- **Issue**: Boxes (h=60) at y=690 extend to y=750, but container height is 720
- **Impact**: Last trust step appears outside the model container
- **Fix**: Increase container height from 720 to 780, move summary text from y=840 to y=890

### FIND-03 [LOW] — D2 (msix-workflow): Cross-lane arrow uses forced vertical entry/exit

- **Location**: `02-workflow-msix-path.drawio`, arrow from s13 (Partner Center lane) to s14 (MS Review lane)
- **Issue**: Uses `d.arrow()` with exitY=1/entryY=0 which forces bottom-to-top routing, creating awkward orthogonal path across lanes
- **Impact**: Arrow routing looks unnatural between cross-lane items
- **Fix**: Change to `d.arrow_free()` which lets DrawIO pick optimal entry/exit

### FIND-04 [LOW] — D7 (current-state): Some text may be tight in fixed-width boxes

- **Location**: `07-current-state-autoclickkey.drawio`, several boxes with long text (e.g., "Package.appxmanifest has placeholder identity")
- **Issue**: Text wraps tightly; might render truncated on some systems
- **Impact**: Minor readability concern
- **Fix**: No change — rely on `whiteSpace=wrap` + font size 12 (already applied)

### FIND-05 [LOW] — D5 (comparison-matrix): Could highlight AutoClickKey verdict more strongly

- **Location**: `05-comparison-matrix.drawio`, last row "AutoClickKey fit"
- **Issue**: Uses same styling as other rows
- **Impact**: Conclusion may not stand out
- **Fix**: No change — final summary box already emphasizes this

## Layer Check Results

| Diagram | L1 Format | L2 Content | L3 Alignment |
|---------|-----------|------------|--------------|
| 01-c4-context | PASS | PASS | PASS |
| 02-workflow-msix | PASS* | PASS | PASS |
| 03-workflow-exe | FAIL (FIND-01) | PASS | PASS |
| 04-decision-tree | PASS | PASS | PASS |
| 05-comparison | PASS | PASS | PASS |
| 06-cicd | PASS | PASS | PASS |
| 07-current-state | PASS | PASS | PASS |
| 08-trust-identity | FAIL (FIND-02) | PASS | PASS |
| 09-update-flow | PASS | PASS | PASS |

*D2 has LOW finding FIND-03 (arrow routing) but is not failure-level.

## Fix Priority

1. FIND-01 (MEDIUM) — fix container overflow
2. FIND-02 (MEDIUM) — fix container overflow
3. FIND-03 (LOW) — auto-fix per workflow rule ("all issues fixed")

## Next Action

Apply all fixes and re-generate. Proceed to iteration 2.
