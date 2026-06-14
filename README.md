# Contribution [#]: [bug(mat-selection-list): disabled selection list still shows cursor pointer on hover around radio]

**Contribution Number:** [1]  
**Student:** [Dipesh Pandit]  
**Issue:** [https://github.com/angular/components/issues/31937]  
**Status:** [Phase II] [Complete]

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
- **Screenshots/logs:** [If applicable]
- **My findings:** The issue happens only in single-selection mode because it uses a radio-style indicator. The multiple-selection checkbox indicator already handles the disabled cursor correctly.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

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

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
