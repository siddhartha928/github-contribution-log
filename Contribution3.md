8# github-contribution-log - siddhartha928

8# Contribution [#3]: Initializer function re-rendering every time

**Contribution Number:** [3]  
**Student:** [Siddhartha Ravilla]  
**Issue:** [https://github.com/wso2/product-is/issues/27955]  
**Fork Link:** [https://github.com/siddhartha928/identity-apps]
**Status:** Phase II Complete

---

## Why I Chose This Issue

This issue caught my attention because it targets a subtle but impactful React performance anti-pattern that's easy to overlook in large codebases. 
Passing a function call directly into useState like useState(getExcludedStoresFromClaim()) means the initializer runs on every render, even though useState only 
uses the value once on mount. 

Beyond the fix itself, this issue gave me a chance to deepen my understanding of how React handles state initialization during the render cycle. Working across 
multiple packages in a real-world enterprise codebase, such as identity apps, also aligns with my learning goal of confidently navigating and contributing to large monorepo projects.

---

## Understanding the Issue

### Problem Description

In React, when you pass a function call directly as the initial value to useState — e.g. useState(getExcludedStoresFromClaim()) — the function is invoked on 
every render, even though useState only uses the returned value during the very first mount. This is a performance anti-pattern known as eager initialization. 
The fix is to change the function to a lazy initializer and call it only once.

### Expected Behavior

The initializer function passed to useState should execute only on the initial render (mount). On subsequent re-renders, React should skip the 
computation entirely and use the already-stored state value, avoiding unnecessary recalculations.

### Current Behavior

The initializer function is called on every render of the component. If getExcludedStoresFromClaim() (or similar functions) involve any non-trivial computation 
such as filtering arrays, accessing stores, or parsing data, this results in redundant work on every re-render, degrading performance especially in 
frequently-updating components.

### Affected Components

The violation exists across 6 files in the identity-apps monorepo:

admin.claims.v1/components/edit/local-claim/edit-mapped-attributes-local-claims.tsx — line 170
admin.console-settings.v1/components/console-settings-tabs.tsx — line 202
admin.login-flow-builder.v1/providers/authentication-flow-provider.tsx — line 147
admin.logs.v1/components/time-range-selector.tsx — line 57
admin.org-insights.v1/pages/org-insights.tsx — line 36
admin.tenants.v1/components/system-settings/system-settings-tabs.tsx — line 116

---

## Reproduction Process

### Environment Setup

The project is a large pnpm monorepo using Node.js. Setup steps followed from the repository README:

Installed dependencies via pnpm install. Encountered peer dependency warnings due to mismatched React versions across packages; resolved by using --shamefully-hoist flag as suggested in the contributing docs.
The repo uses a .nvmrc file; switched to the correct Node version using nvm use to avoid build failures.
Used VS Code with the recommended extensions (ESLint, Prettier) to surface the react-doctor lint warnings directly in the editor.
Running pnpm lint from the affected package directories confirmed the react-doctor/rerender-lazy-state-init warnings at the exact lines listed in the issue.

### Steps to Reproduce

1. Clone the repository and check out the master branch at commit b2a8f16f154cf4ada332b7d624ba4b0ad7629518.
2. Navigate to any affected package, e.g., cd features/admin.logs.v1.
3. Run the linter: pnpm lint.
4. Observe the warning: react-doctor/rerender-lazy-state-init — "useState(getExcludedStoresFromClaim()) calls initializer on every render" at the reported line numbers.
5. Open admin.logs.v1/components/time-range-selector.tsx at line 57 and confirm useState(someFunction()) pattern without an arrow function wrapper.
6. Observed result: Linter flags the eager initialization; the function is being called on every render cycle instead of only once on mount.


### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/siddhartha928/identity-apps/blob/master/features/admin.claims.v1/components/edit/local-claim/edit-mapped-attributes-local-claims.tsx#L169
- **Screenshots/logs:** functions are rerendering
- **My findings:** All 6 violations follow the identical pattern. A function returning computed/derived state is passed directly into useState() without a lazy wrapper.
- The functions involved (e.g., getExcludedStoresFromClaim()) perform non-trivial lookups, making this a real performance concern rather than a purely stylistic one.
  
---

## Solution Approach

### Analysis

The root cause is eager evaluation of the useState initializer. JavaScript evaluates function arguments before passing them, so useState(fn()) runs fn() immediately on every render call. React cannot intercept this. The solution is to pass a function reference instead: useState(() => fn()). React's useState implementation checks if the initial value argument is a function, and if so, calls it only on the first render. This is a React design feature specifically for expensive initializations, and the fix requires only a syntactic wrapping.

### Proposed Solution

Wrap each eager useState initializer in an arrow function across all 6 affected files. The change per file is a single-line modification to achieve render-cycle performance improvement.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** useState(fn()) eagerly calls fn on every render. React only uses the initial value once, so all subsequent calls are wasted computation. The fix is lazy initialization via useState(() => fn()).

**Match:** This is a well-documented React performance pattern. The official React docs explicitly recommend lazy initializers for expensive computations. Within the codebase, other useState calls already use this pattern correctly.

**Plan:** 
1. Create a working branch from master named after the issue (e.g. fix/rerender-lazy-state-init).
2. Open admin.claims.v1/components/edit/local-claim/edit-mapped-attributes-local-claims.tsx at line 170 — wrap the initializer in an arrow function.
3. Open admin.console-settings.v1/components/console-settings-tabs.tsx at line 202 — apply the same fix.
4. Open admin.login-flow-builder.v1/providers/authentication-flow-provider.tsx at line 147 — apply fix.
5. Open admin.logs.v1/components/time-range-selector.tsx at line 57 — apply fix.
6. Open admin.org-insights.v1/pages/org-insights.tsx at line 36 — apply the fix.
7. Open admin.tenants.v1/components/system-settings/system-settings-tabs.tsx at line 116 — apply fix.
8. Re-run pnpm lint across all affected packages to confirm all 6 warnings are resolved.
9. Manually verify components render correctly, and state initializes as expected.

**Implement:** https://github.com/siddhartha928/identity-apps/tree/fix/rerender-lazy-state-init

**Review:** 

[] All 6 files modified with arrow function wrappers
[] Linter passes with zero react-doctor/rerender-lazy-state-init warnings
[] Code follows the project's existing formatting and style conventions
[] Commit message follows the repo's conventional commits format

**Evaluate:** 

Run pnpm lint in each affected package and check for 0 warnings for this rule.
Manually open each affected UI section in a local dev build and confirm components mount and behave identically to before.

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
