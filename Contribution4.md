# github-contribution-log - siddhartha928

## Contribution [4]: [Install Stock Items shows all variants of part in stock list when Allow Variants is off]    

**Contribution Number:** 4  
**Student:** Siddhartha Ravilla  
**Issue:** https://github.com/inventree/InvenTree/issues/12232    
**Fork Link:** https://github.com/siddhartha928/InvenTree           
**Status:** [Phase III] [Complete]

---

## Why I Chose This Issue

I chose this issue because it sits at the intersection of backend filtering logic and frontend UX, an area I've been actively trying to strengthen. InvenTree is a mature Django + React (Mantine) codebase, and the bug occurs in the "Install Stock Item" flow, where a stock item is being installed into a parent with a BOM. The reporter shows that even with the BOM's *Allow Variants* flag disabled, selecting the parent part in the part selector still returns every variant in the stock list, and the install only fails at submit time with a "not in BOM" error. That mismatch between what the UI *shows* and what the API will *accept* is a classic filter-consistency bug, and fixing it requires tracing how BOM enforcement is applied on the stock query end-to-end, which is exactly the kind of walkthrough I want practice doing.

My skill match is reasonable: I'm comfortable with Python/Django ORM queries, and I've worked with React + TypeScript forms so that I can navigate both the backend `stock` app (the install serializer and the stock list filter) and the frontend install-stock form. My learning goal is to get real practice with a two-sided fix such as pushing a filter down into the queryset so the UI never surfaces items the backend will reject and to write a regression test against the stock API so this behaviour is pinned down. The reporter's own guidance ("filter on show as well as on submit to declutter the experience") gives a clear acceptance target to work back from.

---

## Understanding the Issue

### Problem Description

When installing a stock item into an assembly that has a BOM, the "Install Stock Item" dialog has two selectors: a Part selector and a Stock Item selector. The Part selector correctly restricts choices to parts that are actually valid for the BOM (respecting each BOM line's "Allow Variants" flag). However, once a part is chosen, the Stock Item selector ignores that BOM restriction and queries for all stock of the selected part and all of its variants even when the matching BOM line has "Allow Variants" turned off. The user can then pick a variant's stock item that the backend will reject at submit time.

### Expected Behavior

The Stock Item selector should only list stock for the exact part chosen in the Part selector. If a BOM line disallows variants, no variant stock should ever appear as a selectable option. If a BOM line does allow variants, the Stock Item list should  only show stock for whichever specific part (parent or variant) was picked.

### Current Behavior

Selecting the parent part shows every variant's stock in the list. Attempting to install one of those variant stock items fails at submit time with "Selected part is not in the Bill of Materials", because the backend's install validation is variant-aware and correctly rejects it. The UI and the API disagree about what's valid.

### Affected Components

There is a mismatch between the front-end and back-end: the back-end works fine, but the front-end stockform shows options that should be hidden based on the user's input preference, which then blocks the back-end from submitting.

 - Frontend: src/frontend/src/forms/StockForms.tsx

---

## Reproduction Process

### Environment Setup

- I cloned my InvenTree fork and opened the repository using the included VS Code Dev Container.

- I used Dev Containers: Reopen in Container. This built the development image and started the InvenTree, PostgreSQL, and Redis containers.

- Inside the Dev Container, I started the backend server:

  invoke dev.server

- In a second Dev Container terminal, I started the frontend server:

invoke dev.frontend-server

- I opened the frontend at http://localhost:5173 and logged in with a local superuser account.

While setting up the dev container, I faced an error, "Database Migrations required," for which I did "invoke update" to update the migrations and launch the container.

### Steps to Reproduce

1. Navigate to Parts and create a new stock-enabled, trackable part with the following information:
Name: Widget
IPN: WIDGET-001
2. Create a second part to serve as a variant, with the following information:
Name: Widget (Red)
IPN: WIDGET-001-RED
Variant Of: Widget
3. Navigate to Stock → Stock Locations and create a location named Main Warehouse (if one doesn't already exist).
4. Add stock items for both parts in Main Warehouse:
One stock item for Widget, batch code WIDGET-BATCH
One stock item for Widget (Red), batch code WIDGET-RED-BATCH
5. Create a new part named Widget Assembly and configure it as an assembly that can be manufactured, with the following information:
Name: Widget Assembly
IPN: ASM-WIDGET-001
6. Open the Bill of Materials tab for Widget Assembly.
7. Add Widget as a BOM item with:
Quantity: 1
Allow Variants: unchecked
8. In Settings -> Global Settings, confirm STOCK_ENFORCE_BOM_INSTALLATION is enabled (enable it if not).
9. Create a trackable stock item for Widget Assembly in Main Warehouse.
10. Open that Widget Assembly stock item and select Install Stock Item.
11. In the Part selector, choose Widget.
12. Observed: the Stock Item selector lists both the WIDGET-BATCH stock item (for Widget) and the WIDGET-RED-BATCH stock item (for Widget (Red)), even though the BOM line for Widget has "Allow Variants" unchecked.
13. Select the WIDGET-RED-BATCH stock item and submit. The install fails with "Selected part is not in the Bill of Materials", since Widget (Red) is correctly excluded by the backend's BOM check. It should never have been offered as a selectable stock item in step 12.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/siddhartha928/InvenTree/tree/12232-install-stock-variant-filter
- **Screenshots/logs:** [If applicable]
- **My findings:** Traced the mismatch to StockForms.tsx's stock_item filter object omitting include_variants, combined with the backend filter_part (stock/api.py) defaulting that flag to True. Confirmed the Part selector's in_bom_for filter is already correct because it calls Part.get_parts_in_bom() -> BomItem.get_valid_parts_for_allocation(), which only adds variants to the valid set when self.allow_variants is True on that BOM line.

---

## Solution Approach

### Analysis

The root cause is a filter-consistency gap between two dependent form fields, not a backend validation bug. The backend's install-time BOM check was already correct. The Part selector is BOM-aware end-to-end, but the Stock Item selector, once a part ID is chosen, re-queries the stock API with only part=<id> and lets the API's own default (include_variants=True) silently re-expand the results to include variant stock the BOM line doesn't allow.

### Proposed Solution

Explicitly pass include_variants: false in the stock_item field's filters in useStockItemInstallFields(). This makes the stock list match exactly the part chosen in the Part selector

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The Stock Item selector in the Install Stock Item dialog shows stock for variants of the selected part even when the relevant BOM line disallows variants, causing selections that fail backend validation on submit.

**Match:** The Part selector already solves an analogous problem correctly via the in_bom_for filter (part/api.py's filter_in_bom, built on get_parts_in_bom()/get_valid_parts_for_allocation()), and the stock API already exposes an include_variants filter (stock/api.py) — it's just never explicitly set to false from this form.

**Plan:** 
1. Modify src/frontend/src/forms/StockForms.tsx — add include_variants: false to the stock_item field's filters object in useStockItemInstallFields()
2. No backend changes needed. filter_part/filter_include_variants in stock/api.py already support this correctly; it was just never invoked with the flag set
3. Add/verify a backend regression test on the stock list API confirming include_variants=false excludes variant stock for a given part
4. Manually verify in the running app: BOM line with variants disallowed -> only exact-part stock shown; BOM line with variants allowed -> variant appears as its own Part-selector entry, and selecting it shows only its own stock

**Implement:** https://github.com/siddhartha928/InvenTree/tree/12232-install-stock-variant-filter

**Review:** 
- [x] Diff is a single-line, scoped change to the exact field causing the bug       
- [x] No unrelated formatting; matches project code style                   
- [x] Doesn't touch backend since the needed filter already existed                     

**Evaluate:** 
- [x] Reproduce the original steps and confirm the variant no longer appears       
- [x] Add a regression test exercising StockFilter with include_variants=false                     

---

## Testing Strategy

### Unit Tests
- [x] Backend: `test_filter_by_part_include_variants` in stock/test_api.py — asserts `?part=<id>&include_variants=false` returns only exact-part stock, and confirms `include_variants=true`/default is unchanged

### Integration Tests

- [x] End-to-end: BOM line with allow_variants=False -> Install Stock Item dialog never surfaces variant stock, install succeeds only for the exact part
- [x] End-to-end: BOM line with allow_variants=True -> variant appears as a separate Part-selector option; selecting it shows only its own stock

### Manual Testing

Reproduced the original bug against `main` to establish a baseline: selecting the `Widget` parent in the Part selector returned both `WIDGET-BATCH` and `WIDGET-RED-BATCH` in the Stock Item selector, and submitting the red variant failed with *"Selected part is not in the Bill of Materials"*. After applying the bug fix in `StockForms.tsx` and reloading via the dev server, I re-ran the exact reproduction steps and additional variations:

- With `allow_variants = False`: only `WIDGET-BATCH` appeared in the Stock Item selector; `WIDGET-RED-BATCH` was correctly filtered out. Install completed successfully.
- With `allow_variants = True` on the same BOM line: both stock items appeared, and install completed successfully for either choice — matching intended behavior with no over-filtering.
- Non-variant part on a BOM (added a plain resistor part with no variants): no visible change vs. pre-fix, install works as before.

---

## Implementation Notes

### Week [1] Progress

- Environment setup and reproduction. I brought up the Dev Container, hit the "Database Migrations required" error, resolved it with `invoke update`, and confirmed I could reproduce the bug locally using a minimal fixture.
- Did an investigation trace to isolate where the two selectors diverged. 

**Investigation trace:**
- Started at the frontend: opened the browser Network tab during the Install Stock Item flow. The Stock Item selector was firing `GET /api/stock/?part=<parent_id>` with no `include_variants` parameter — which means the API's default (`True`) applied, and variant stock was included.
- Located the source of that request in `src/frontend/src/forms/StockForms.tsx`, inside `useStockItemInstallFields()`, in the `filters` object attached to the `stock_item` field.
- Cross-checked the Part selector's filter in the same hook: it uses `in_bom_for: <install_into_id>`, the correct BOM-aware filter, which is why the Part selector behaves correctly.
- Followed `in_bom_for` into `src/backend/InvenTree/part/api.py` (`filter_in_bom`), which calls `Part.get_parts_in_bom()` -> `BomItem.get_valid_parts_for_allocation()`. That method only unions in variants when `self.allow_variants` is `True` on the BOM line exactly the semantics that needed to be mirrored on the stock side.
- Confirmed in `src/backend/InvenTree/stock/api.py` that `include_variants` is already an exposed filter on the `StockFilter` FilterSet, defaulting to `True`. That global default is the right choice for the API as a whole (other callers depend on it), so the fix belonged at the specific call site, not in the API default.

**Files modified:**
Files modified: src/frontend/src/forms/StockForms.tsx (useStockItemInstallFields, stock_item field filters — added include_variants: false)

**Blockers hit and resolved:**
- Dev Container startup failed with "Database Migrations required"; fixed with `invoke update` before starting the backend server.
- Early ambiguity about where the fix belonged in backend default vs. frontend opt-in. Ruled out changing the backend default after verifying `include_variants` and finding several that intentionally rely on it.

### Code Changes

- **Files modified:** src/frontend/src/forms/StockForms.tsx
- **Key commits:** https://github.com/siddhartha928/InvenTree/commit/598b121ea01c003c3f3fdc85b10d1d170092cad5
- **Approach decisions:** Chose a frontend-only fix rather than changing the backend default, since stock/api.py already exposes include_variants as an explicit opt-in filter for exactly this purpose — the bug was that this form never set it, not that the API lacked the capability. Rejected changing the backend default to include_variants=False globally, since other call sites intentionally rely on the current default to include variant stock.

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
