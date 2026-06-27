# Contribution [#]: [bug(mat-selection-list): disabled selection list still shows cursor pointer on hover around radio]

**Contribution Number:** [1]  
**Student:** [Dipesh Pandit]  
**Issue:** [https://github.com/angular/components/issues/31937]  
**Status:** [Phase IV] [Complete]

---

## Why I Chose This Issue

I chose this issue because it directly aligns with my current technical background and academic experience. Last semester, I worked extensively on a project using the Angular framework, which allowed me to build a strong foundational knowledge of its architecture. Additionally, my programming background includes experience with JavaScript and TypeScript, making the Angular Components codebase a natural fit for my skills. Furthermore, the repository's active open-source community and exceptionally clear contribution documentation provide an ideal environment for me to learn enterprise engineering standards.

---

## Understanding the Issue

### Problem Description

In a single-selection mat-selection-list, a disabled option still shows a pointer cursor when hovering over its radio button. This makes the disabled option look like it can be clicked. The issue happens with the disabled "Stawberries" option in the list demo.

### Expected Behavior

A disabled list option should not show a pointer cursor. It should look and behave like a disabled item. Users should understand that the option cannot be selected.

### Current Behavior

The disabled option cannot be selected, but the cursor still changes to a pointer over the radio button. This gives the wrong impression that the option is clickable. The issue only happens in the single-selection list.

### Affected Components

The main affected area is the Angular Material list component, especially mat-selection-list and mat-list-option. The relevant files are src/material/list/list-option.scss, src/material/list/list.scss, and src/material/radio/_radio-common.scss. The issue likely comes from the radio indicator styles being reused in single-selection lists without applying a disabled cursor override.

---

## Reproduction Process

### Environment Setup

I set up the project locally from my fork of the `angular/components` repository and created a working branch for the issue. I followed the step from DEV_ENVIRONMENT.md file. One challenge I faced was that the project requires `pnpm`, but it was not installed or available correctly in my local environment at first. I solved this by installing/enabling `pnpm`, then ran `pnpm i` to install the project dependencies and `pnpm dev-app` to start the local Angular Material dev app.

### Steps to Reproduce

1. Run the local dev app with `pnpm dev-app`.
2. Open the dev app in the browser and navigate to `/list`.
3. Scroll down to the `Single Selection list` section.
4. Hover over the radio indicator next to the disabled `Strawberries` option.
5. Observe that the cursor changes to a pointer even though the option is disabled.

### Reproduction Evidence

- **Commit showing reproduction:** [https://github.com/dipeshpandit12/components/tree/fix-button-disabled-state]
- **Screenshots/logs:**
![Reproduction screenshot 1](IMG_8338.HEIC)
![Reproduction screenshot 2](IMG_8337.HEIC)
- **My findings:** The issue happens only in single-selection mode because it uses a radio-style indicator. The multiple-selection checkbox indicator already handles the disabled cursor correctly.

---

## Solution Approach

### Analysis

The root cause is in the shared radio "structure" styles. The `radio-structure` mixin in `src/material/radio/_radio-common.scss` unconditionally sets `cursor: pointer` on the `.mdc-radio` element. Because the selection list reuses this mixin (it is included under `.mat-mdc-list-option` in `src/material/list/list-option.scss`), every single-selection option's radio indicator inherits `cursor: pointer`.

The radio's own disabled cursor reset (`cursor: default`) only applies via the `.mdc-radio__native-control:disabled` selector. But inside a list option that native `<input>` is set to `display: none` (it is purely decorative and removed from the tab order), and the disabled state is instead represented by the `.mdc-list-item--disabled` class on the host `.mat-mdc-list-option` element. As a result, the disabled-cursor rule never matches, and the disabled radio keeps the pointer cursor — making the option look clickable.

The multiple-selection list does not show the bug because its checkbox indicator already handles the disabled cursor.

### Proposed Solution

Add a scoped override in `src/material/list/list.scss` that resets the indicator cursor to `default` for disabled list options:

```scss
.mat-mdc-list-option.mdc-list-item--disabled {
  .mdc-radio,
  .mdc-checkbox {
    cursor: default;
  }
}
```

The override is intentionally placed on the existing disabled-indicator block (which already adjusts `opacity`) and is scoped with both the `.mat-mdc-list-option` host class and the `.mdc-list-item--disabled` state class. This gives it specificity (0,3,0), which is required to win over the structure styles' `.mat-mdc-list-option .mdc-radio { cursor: pointer; }` rule at (0,2,0) — an equal-specificity rule would lose on source order. The change is limited to disabled list options, so standalone radios and enabled options are unaffected.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The issue is that a disabled `mat-list-option` in a single-selection `mat-selection-list` still shows a pointer cursor when hovering over its radio indicator. This makes the disabled option look clickable even though it cannot be selected. The disabled radio indicator should use the default cursor, matching the disabled behavior in the multiple-selection list.

**Match:** The multiple-selection list already behaves correctly because it uses a checkbox-style indicator. In `src/material/checkbox/_checkbox-common.scss`, disabled checkboxes use `.mdc-checkbox--disabled` to set `cursor: default` and remove pointer behavior. The single-selection list uses the radio styles from `src/material/radio/_radio-common.scss`, where `.mdc-radio` sets `cursor: pointer`, but the disabled list option does not override that cursor.

**Plan:** [Step-by-step implementation plan]
1. Inspect the rendered DOM for the disabled `Strawberries` option to confirm the disabled option and nested radio classes.
2. Modify `src/material/list/list-option.scss` or `src/material/list/list.scss` to override the nested `.mdc-radio` cursor when the parent `mat-list-option` is disabled.
3. Keep the change scoped to disabled selection-list radio indicators so normal radio buttons are not affected.
4. Add or update a focused test if the list test setup supports checking this style behavior.
5. Run the local dev app and focused list tests to confirm the fix.

**Implement:** Implementation will happen in Phase III on this branch:  
https://github.com/dipeshpandit12/components/tree/fix-button-disabled-state

**Review:** I will review the change against `CONTRIBUTING.md` to make sure the fix is focused, uses the project’s existing style patterns, and follows the Angular commit message format. I will also check that I do not include unrelated files such as `.devcontainer/` or temporary local files. The expected final code commit message would be similar to `fix(material/list): remove pointer cursor from disabled radio option`.

**Evaluate:** I will verify the fix manually by running `pnpm dev-app`, opening `/list`, and hovering over the disabled `Strawberries` radio indicator in the `Single Selection list`. The cursor should no longer change to a pointer for the disabled option, while enabled options should still behave normally. I will also run `pnpm test list` to confirm the focused list tests pass.

---

## Testing Strategy

### Unit Tests

Added a focused regression test in `src/material/list/selection-list.spec.ts` (describe block `with single selection and a disabled option`). It renders a single-selection `mat-selection-list` with one enabled and one disabled option and asserts the computed cursor of each option's `.mdc-radio` indicator, following the existing `getComputedStyle(...).cursor` pattern used in `slider.spec.ts` and `select.spec.ts`.

- [x] Test case 1: A disabled option's radio indicator computes `cursor: default` (the fix).
- [x] Test case 2: An enabled option's radio indicator still computes `cursor: pointer` (guards against over-broad regression — normal options unaffected).

This test fails before the fix (`Expected 'pointer' to be 'default'`) and passes after, so it is a true regression test for the bug. It also caught a real defect in my first attempt at the fix: my initial selector tied the specificity battle with the structure styles and lost, so the cursor stayed `pointer`. The failing test forced me to raise the specificity to the correct value.

### Integration Tests

- Not applicable. This is a CSS/styling fix in a single component; no cross-component integration behavior changes.

### Manual Testing

- Ran `pnpm stylelint src/material/list/list.scss` → passes (exit 0, no violations).
- Ran `pnpm test list --no-watch` → full list unit suite passes (119 tests, including the new one).
- Pending visual check via `pnpm dev-app` at `/list` (Single Selection list): confirm hovering the disabled "Strawberries" radio indicator no longer shows a pointer cursor, while enabled options still show the pointer. (The automated test already asserts this computed behavior.)

---

## Implementation Notes

### Week of 2026-06-22 Progress

- Confirmed the root cause: the `radio-structure` mixin sets `cursor: pointer` on `.mdc-radio`, and the radio's disabled-cursor reset can't apply in a list option because the native control is `display: none` and the disabled state lives on the `.mdc-list-item--disabled` host class.
- Implemented the fix in `src/material/list/list.scss` by extending the existing disabled-indicator block and committed it on its own.
- Wrote a focused regression test in `selection-list.spec.ts` and ran the list suite.
- **Challenge:** the first version of the fix didn't work — the test failed with `Expected 'pointer' to be 'default'`. The cause was a CSS specificity tie: my override `.mdc-list-item--disabled .mdc-radio` (0,2,0) matched the same specificity as the structure rule `.mat-mdc-list-option .mdc-radio` (0,2,0), so the pointer rule won on source order.
- **Resolution:** scoped the override to `.mat-mdc-list-option.mdc-list-item--disabled .mdc-radio` (0,3,0) so it reliably wins. I then amended the fix commit so the corrected selector ships in a single clean fix commit, with the test as a separate commit.
- **Decision:** kept the change in `list.scss` (not in the shared radio mixin) so standalone radio buttons are untouched and the fix stays scoped to disabled selection-list options.

### Code Changes

- **Files modified:**
  - `src/material/list/list.scss` — scoped the disabled-indicator block to `.mat-mdc-list-option.mdc-list-item--disabled` and added `cursor: default` for the nested `.mdc-radio` / `.mdc-checkbox`.
  - `src/material/list/selection-list.spec.ts` — added the `with single selection and a disabled option` regression test and its test component.
- **Key commits (branch `fix-button-disabled-state`):**
  - `fix(material/list): remove pointer cursor from disabled radio option`
  - `test(material/list): verify disabled option resets indicator cursor`
- **Approach decisions:** Reused the existing disabled-indicator block rather than adding a new rule (DRY, and behavior-neutral for the existing `opacity` since only options contain these indicators). Raised specificity via the host class instead of `!important`. Excluded the unrelated `.devcontainer/devcontainer.json` from the fix commit per the project's "only relevant changes" guidance.

---

## Pull Request

**PR Link:** Not opened upstream — issue #31937 was already fixed and closed by
[angular/components#33418](https://github.com/angular/components/pull/33418)
before submission. Opening a new PR would be a duplicate.

**What I contributed (1–2 sentences):** I independently reproduced and fixed the
disabled selection-list radio cursor bug (scoped `cursor: default` override in
`list.scss`) and added a regression test in `selection-list.spec.ts`. My
root-cause analysis matched the maintainer's merged fix (#33418).

**Maintainer Feedback / Next steps:**
- 2026-06-27: On checking the issue before submitting, found it already resolved
  upstream via #33418 (merged 2026-06-18). No PR submitted to avoid a duplicate.
- Work retained on my fork branch as the contribution artifact.

**Status:** Resolved upstream (superseded) — my equivalent fix is preserved on my
fork; no separate PR is warranted.

---

## Learnings & Reflections

### Technical Skills Gained

- Tracing a styling bug from a visible symptom to a shared SCSS mixin.
- Reasoning about CSS specificity to make an override reliably win
  (0,3,0 vs 0,2,0) instead of reaching for `!important`.
- Writing a computed-style regression test with `getComputedStyle(...).cursor`.

### Challenges Overcome

- A CSS specificity tie made my first fix silently fail; the regression test
  surfaced it and pointed me to the host-class scoping that fixed it.
- Late-stage discovery that the issue was already merged upstream — handled by
  verifying the upstream commit and documenting the duplicate honestly rather
  than submitting a conflicting PR.

### What I'd Do Differently Next Time

- **Check the issue's current status and any linked/recent PRs immediately
  before implementing**, and keep the branch rebased on the latest upstream so I
  catch an in-flight maintainer fix early.

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
