# Income Bottom Sheet Alignment Fix – Bugfix Design

## Overview

The income screen's OTP bottom sheet renders its heading ("Enter OTP") and description subtitle with incorrect alignment and spacing. Both elements should be left-aligned and separated by exactly 4px of vertical space.

The root causes are two CSS property values in two separate files:

1. `BottomSheet.svelte` — `.sheet-title` has no explicit `text-align`, and the `.sheet-inner` container applies a `gap: 24px` between the title row and the content slot, making the 4px spacing impossible without a local override.
2. `income/+page.svelte` — `.otp-subtitle` explicitly sets `text-align: center`.

The fix is minimal and scoped: change `text-align` on `.sheet-title` in the generic component and on `.otp-subtitle` in the income page, and add a negative `margin-top` on `.otp-subtitle` to override the `24px` gap from the sheet's layout so the net space between heading and subtitle is exactly 4px.

## Glossary

- **Bug_Condition (C)**: The condition where the income screen's OTP bottom sheet is rendered — specifically, when `showOtpSheet` is `true` in `income/+page.svelte`.
- **Property (P)**: The desired visual state — heading left-aligned, description left-aligned, 4px vertical gap between them.
- **Preservation**: All other behaviors, layouts, and styles in the income screen and all other bottom sheet usages across the application that must remain unchanged.
- **`BottomSheet.svelte`**: The reusable bottom sheet component at `src/lib/components/BottomSheet.svelte` that renders `.sheet-title` and the content slot.
- **`.otp-subtitle`**: The `<p>` element in `src/routes/income/+page.svelte` that renders the description text inside the OTP bottom sheet.
- **`sheet-inner` gap**: The `gap: 24px` in `BottomSheet.svelte`'s `.sheet-inner` flex container, which controls spacing between the title row and the content slot.

## Bug Details

### Bug Condition

The bug manifests when a user presses "Continue" on the income screen (with all steps complete and the consent checkbox checked), which sets `showOtpSheet = true` and renders the `BottomSheet` component with `title="Enter OTP"`. Inside this sheet, the `.sheet-title` heading and `.otp-subtitle` description are misaligned and incorrectly spaced.

**Formal Specification:**
```
FUNCTION isBugCondition(state)
  INPUT: state — application render state
  OUTPUT: boolean

  RETURN state.route == '/income'
         AND state.showOtpSheet == true
         AND (
           textAlign(sheet-title) != 'left'
           OR textAlign(otp-subtitle) != 'left'
           OR verticalGap(sheet-title, otp-subtitle) != 4
         )
END FUNCTION
```

### Examples

- **Heading alignment**: When the OTP bottom sheet opens, `.sheet-title` renders "Enter OTP" with no explicit `text-align` (inherits or defaults to center). Expected: `text-align: left`.
- **Description alignment**: `.otp-subtitle` renders "Enter the OTP sent to your Aadhaar-linked mobile number" with `text-align: center`. Expected: `text-align: left`.
- **Spacing**: The `.sheet-inner` flex container has `gap: 24px`, placing 24px between the title row and the content slot where `.otp-subtitle` lives. Expected: 4px between heading bottom edge and description top edge.
- **Edge case**: If the title is empty (other usages of `BottomSheet`), the title row is not rendered; this fix must not affect those cases.

## Expected Behavior

### Preservation Requirements

**Unchanged Behaviors:**
- Opening the OTP bottom sheet when the "Continue" CTA is clicked must continue to work correctly.
- All other content inside the OTP bottom sheet (OTP input boxes, resend row, verify button) must remain visually and functionally unaffected.
- All other screens that use `BottomSheet.svelte` (e.g., option selection sheets in `ProgressiveInputStepper`) must continue to render with their existing alignment and spacing.
- Mouse interactions, form validation, and navigation behavior on the income screen must remain unchanged.

**Scope:**
All inputs that do NOT result in rendering the income screen's OTP bottom sheet should be completely unaffected by this fix. This includes:
- Any other route using `BottomSheet.svelte`
- Non-bottom-sheet elements on the income screen
- All other routes in the application

## Hypothesized Root Cause

Based on code inspection of `src/lib/components/BottomSheet.svelte` and `src/routes/income/+page.svelte`:

1. **Missing `text-align: left` on `.sheet-title`** (`BottomSheet.svelte`, line ~93): The `.sheet-title` rule defines `font-family`, `font-weight`, `font-size`, `line-height`, and `color` but omits `text-align`. Without an explicit value, the heading can appear centered depending on context.

2. **Explicit `text-align: center` on `.otp-subtitle`** (`income/+page.svelte`, line ~333): The `.otp-subtitle` rule explicitly sets `text-align: center`. This must be changed to `left`.

3. **`gap: 24px` on `.sheet-inner` creates 24px spacing, not 4px** (`BottomSheet.svelte`, line ~78): The `.sheet-inner` flex container uses `gap: 24px` to space the title row from the content slot. Since the description is the first element inside the content slot (`.sheet-content`), it inherits this 24px gap. The fix must reduce this to 4px for the income OTP sheet context without disrupting other sheets. The cleanest approach is a scoped `margin-top: -20px` override on `.otp-subtitle` (24px gap − 20px = 4px net), keeping the generic BottomSheet component untouched for the spacing dimension.

4. **Alternative spacing approach** — changing `gap: 24px` on `.sheet-inner` globally would affect all bottom sheet usages and is a risky regression surface. The per-page override is safer and more targeted.

## Correctness Properties

Property 1: Bug Condition – Income OTP Bottom Sheet Text Alignment and Spacing

_For any_ render state where `isBugCondition` returns true (the income screen's OTP bottom sheet is open), the fixed code SHALL display the heading (`sheet-title`) with `text-align: left`, the description (`.otp-subtitle`) with `text-align: left`, and a vertical gap of exactly 4px between the bottom edge of the heading and the top edge of the description.

**Validates: Requirements 2.1, 2.2, 2.3**

Property 2: Preservation – All Other Bottom Sheet and Screen Behavior

_For any_ render state where `isBugCondition` returns false (any other screen, or the income screen when the OTP sheet is not open), the fixed code SHALL produce exactly the same visual output and behavior as the original code, preserving all existing alignment, spacing, and interaction styles.

**Validates: Requirements 3.1, 3.2, 3.3**

## Fix Implementation

### Changes Required

**File 1**: `src/lib/components/BottomSheet.svelte`

**Target Rule**: `.sheet-title`

**Specific Change**:
Add `text-align: left` to the `.sheet-title` CSS rule.

```css
/* Before */
.sheet-title {
  font-family: 'Nunito Sans', sans-serif;
  font-weight: 600;
  font-size: 20px;
  line-height: 24px;
  color: #111827;
}

/* After */
.sheet-title {
  font-family: 'Nunito Sans', sans-serif;
  font-weight: 600;
  font-size: 20px;
  line-height: 24px;
  color: #111827;
  text-align: left;
}
```

**Rationale**: Left-aligning the heading in the shared component is safe — all existing usages either have no visual dependency on the heading's alignment or would benefit from consistent left alignment. This is a low-risk global fix.

---

**File 2**: `src/routes/income/+page.svelte`

**Target Rule**: `.otp-subtitle`

**Specific Changes**:
1. Change `text-align: center` to `text-align: left`.
2. Add `margin-top: -20px` to reduce the effective vertical gap from the `BottomSheet` component's `gap: 24px` down to 4px (24 − 20 = 4px).

```css
/* Before */
.otp-subtitle {
  font-family: 'Nunito Sans', sans-serif;
  font-weight: 400;
  font-size: 13px;
  color: #6B7280;
  line-height: 1.5;
  text-align: center;
  margin-top: 4px;
  margin-bottom: 24px;
}

/* After */
.otp-subtitle {
  font-family: 'Nunito Sans', sans-serif;
  font-weight: 400;
  font-size: 13px;
  color: #6B7280;
  line-height: 1.5;
  text-align: left;
  margin-top: -20px;
  margin-bottom: 24px;
}
```

**Rationale**: The `margin-top: -20px` cancels out 20px of the `.sheet-inner`'s `gap: 24px`, yielding a net 4px space between heading and description. This keeps the generic BottomSheet component's layout intact for all other usages.

---

### Summary of Changes

| File | Rule | Property | Old Value | New Value |
|------|------|----------|-----------|-----------|
| `BottomSheet.svelte` | `.sheet-title` | `text-align` | _(unset)_ | `left` |
| `income/+page.svelte` | `.otp-subtitle` | `text-align` | `center` | `left` |
| `income/+page.svelte` | `.otp-subtitle` | `margin-top` | `4px` | `-20px` |

## Testing Strategy

### Validation Approach

The testing strategy follows a two-phase approach: first, confirm the bug exists on the unfixed code by writing tests that assert the expected (correct) state and observing them fail; then apply the fix and verify all tests pass including preservation tests.

### Exploratory Bug Condition Checking

**Goal**: Surface counterexamples that demonstrate the bug BEFORE implementing the fix. Confirm the root cause analysis by observing which assertions fail and why.

**Test Plan**: Render the income screen's OTP bottom sheet (set `showOtpSheet = true`) and assert the correct CSS property values on `.sheet-title` and `.otp-subtitle`. Run on the unfixed code to observe failures.

**Test Cases**:
1. **Heading alignment test**: Assert `getComputedStyle(sheetTitle).textAlign === 'left'` — will fail on unfixed code (no explicit value set).
2. **Description alignment test**: Assert `getComputedStyle(otpSubtitle).textAlign === 'left'` — will fail on unfixed code (`text-align: center` is set).
3. **Vertical spacing test**: Assert that the vertical gap between heading bottom and description top equals 4px — will fail on unfixed code (gap is 24px from `.sheet-inner`).
4. **Edge case — empty title**: Assert that a `BottomSheet` with no title prop still renders its content slot correctly — should pass on both unfixed and fixed code.

**Expected Counterexamples**:
- `textAlign` of `.sheet-title` is not `'left'` (missing property defaults or centers).
- `textAlign` of `.otp-subtitle` is `'center'` instead of `'left'`.
- Vertical offset between elements is 24px (from gap) instead of 4px.

### Fix Checking

**Goal**: Verify that for all states where the bug condition holds, the fixed code produces the correct visual output.

**Pseudocode:**
```
FOR ALL state WHERE isBugCondition(state) DO
  render income screen with showOtpSheet = true
  ASSERT getComputedStyle(sheet-title).textAlign == 'left'
  ASSERT getComputedStyle(otp-subtitle).textAlign == 'left'
  ASSERT verticalGap(sheet-title, otp-subtitle) == 4
END FOR
```

### Preservation Checking

**Goal**: Verify that for all states where the bug condition does NOT hold, the fixed code produces the same visual output as the original code.

**Pseudocode:**
```
FOR ALL state WHERE NOT isBugCondition(state) DO
  ASSERT render(fixed_code, state) == render(original_code, state)
END FOR
```

**Testing Approach**: Property-based testing is recommended for preservation because:
- It can generate varied content (different titles, different sheet content) to confirm the generic BottomSheet is unaffected.
- It provides confidence that the `text-align: left` change to `.sheet-title` doesn't break any other sheet usage.
- It catches edge cases like empty titles, long text wrapping, or no-footer variants.

**Test Cases**:
1. **Other BottomSheet usages**: Verify that bottom sheets rendered outside the income OTP context (e.g., option selection sheets) continue to display with unchanged styles.
2. **Income screen CTA continues to open sheet**: Verify `showOtpSheet` is set to `true` when `openOtpSheet()` is called with valid state.
3. **OTP input and verify flow unchanged**: Verify that the OTP boxes, resend row, and verify button within the sheet remain unaffected after the style fix.
4. **Other screens unaffected**: Render a sample of other routes (e.g., `/pan`, `/email`) and verify their bottom sheets, if any, are visually unchanged.

### Unit Tests

- Test that `.sheet-title` has `text-align: left` after the fix.
- Test that `.otp-subtitle` has `text-align: left` after the fix.
- Test that the computed vertical distance between `.sheet-title` and `.otp-subtitle` is 4px.
- Test that `BottomSheet` with `title=""` (no title) still renders the content slot without layout regressions.

### Property-Based Tests

- For any string value passed as `title` to `BottomSheet`, the rendered `.sheet-title` SHALL always have `text-align: left`.
- For any content rendered inside `BottomSheet` from routes other than the income OTP sheet, the layout gap SHALL remain unchanged (24px between title row and content).
- For any valid income form state where `allValid && agreed`, calling `openOtpSheet()` SHALL always result in `showOtpSheet === true`.

### Integration Tests

- Full flow: complete all income steps, tick the consent checkbox, tap "Continue", verify the OTP bottom sheet opens with left-aligned heading and description separated by 4px.
- Regression: complete the same flow on the pan or email screens and verify their bottom sheets are visually identical to the pre-fix baseline.
- Dismissal: open and dismiss the income OTP bottom sheet and verify the income screen returns to its normal state correctly.
