# github-contribution-log - siddhartha928


# Contribution [1]: Combobox Search Input and Tag Invalidation Bugs in Product Organization Form

**Contribution Number:** 1  
**Student:** Siddhartha Ravilla  
**Issue:** [https://github.com/medusajs/medusa/issues/15621]   
**Fork Link:** https://github.com/siddhartha928/medusa        
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because it touches the UI input behavior and React Query cache management of a real-world React application. Bug 1 involves a subtle prop-spreading conflict in a combobox component where `autoSelect` likely interferes with controlled input state, which is exactly the kind of tricky frontend debugging I want to get better at. Understanding how `{...field}` spreads interact with third-party component props (here `@ariakit/react`) is a practical skill that comes up constantly in production codebases.

Bug 2 appealed to me because it's a classic cache invalidation problem. The query key mismatch between `useCreateProductTag` and the combobox's hardcoded key means newly created tags silently never appear, with no obvious error to debug. Tracing that through React Query's invalidation logic and aligning it with the canonical `productTagsQueryKeys` pattern will deepen my understanding of how large codebases manage server state. Together, these two bugs offer a well-rounded challenge across component design, state management, and data fetching.

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
