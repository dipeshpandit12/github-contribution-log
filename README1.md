# Contribution 2: docs(azd): document well-known runtime hosts that `azd` consumes

**Contribution Number:** 2  
**Student:** Dipesh Pandit  
**Issue:** https://github.com/Azure/azure-dev/issues/4415  
**Status:** Phase I — Complete

---

## Problem Summary

The Azure Developer CLI (`azd`) reaches out to a number of external services at runtime — ARM management APIs, telemetry, and experimentation endpoints — but there is no single documented list of the hosts it needs. This matters for organizations running `azd` in locked-down networks behind firewalls or traffic-filtering appliances, who currently have to reverse-engineer the allowlist by trial and error. The issue asks a contributor to catalog these well-known hosts and add clear documentation for them. I chose it because it is a small, well-scoped `good first issue` / `help wanted` task with a clear definition of "done," making it a realistic first open-source contribution.

---

## Why I Chose This Issue

As an IT intern, I use Azure to implement AI solutions for process automation. This issue aligns directly with my current professional work, making it a highly relevant contribution — I regularly work inside the Azure ecosystem, so improving the documentation around `azd` is something that has immediate value both to me and to teams like mine.

Beyond the personal relevance, it is well-scoped for a first contribution and has a clear finish line. It is labeled `good first issue` and `help wanted`, is open and unassigned, and asks for a bounded deliverable — a documented list of the external hosts `azd` requires — rather than an open-ended refactor. That makes it something I can fully understand, research, and complete within the program timeline.

It also matches my learning goals. Producing this documentation means reading through the `azd` codebase to trace where network calls are made (ARM, telemetry, and experimentation endpoints), which is a great way to learn how a real production CLI is structured while making a contribution that has concrete value for teams deploying `azd` in restricted network environments. The Azure `azure-dev` repository is actively maintained and has clear contribution guidelines, which gives me a supportive environment to learn enterprise engineering standards.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
