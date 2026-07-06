# github-contribution-log - siddhartha928

## Contribution [4]: [Install Stock Items shows all variants of part in stock list when Allow Variants is off]    

**Contribution Number:** 4  
**Student:** Siddhartha Ravilla  
**Issue:** https://github.com/inventree/InvenTree/issues/12232    
**Fork Link:** https://github.com/siddhartha928/InvenTree           
**Status:** [Phase I] [Complete]

---

## Why I Chose This Issue

I chose this issue because it sits at the intersection of backend filtering logic and frontend UX, an area I've been actively trying to strengthen. InvenTree is a mature Django + React (Mantine) codebase, and the bug occurs in the "Install Stock Item" flow, where a stock item is being installed into a parent with a BOM. The reporter shows that even with the BOM's *Allow Variants* flag disabled, selecting the parent part in the part selector still returns every variant in the stock list, and the install only fails at submit time with a "not in BOM" error. That mismatch between what the UI *shows* and what the API will *accept* is a classic filter-consistency bug, and fixing it requires tracing how BOM enforcement is applied on the stock query end-to-end, which is exactly the kind of walkthrough I want practice doing.

My skill match is reasonable: I'm comfortable with Python/Django ORM queries, and I've worked with React + TypeScript forms so that I can navigate both the backend `stock` app (the install serializer and the stock list filter) and the frontend install-stock form. My learning goal is to get real practice with a two-sided fix such as pushing a filter down into the queryset so the UI never surfaces items the backend will reject and to write a regression test against the stock API so this behaviour is pinned down. The reporter's own guidance ("filter on show as well as on submit to declutter the experience") gives a clear acceptance target to work back from.

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
