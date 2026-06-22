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

# The issue is closed now, as someone submitted a PR

