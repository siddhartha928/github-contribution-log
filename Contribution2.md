# Contribution #2: Fix Malformed Unicode Quote in BPMH Save Button

**Contribution Number:** 2    
**Student:** Siddhartha Ravilla        
**Issue:** [#2636 — Fix Malformed Unicode Quote in BPMH Save Button](https://github.com/carlos-emr/carlos/issues/2636)      
**Status:** Phase IV — ✅ Complete

---

## Why I Chose This Issue

I chose this issue because it represented a precise, well-scoped bug with clear acceptance criteria ideal for a first open-source contribution. The problem involved understanding how browsers and JSP parsers handle non-ASCII characters in HTML attributes, which sits at the intersection of web standards and Java server-side templating.

My learning goal was to get comfortable with the full contribution lifecycle: forking a repo, identifying a real bug, verifying it locally, making a minimal targeted fix, and submitting a PR that follows the project's conventions. A focused issue like this let me focus on the process rather than getting overwhelmed by a large feature or complex logic change.

---

## Understanding the Issue

### Problem Description

The save submit button in `formBPMH.jsp` had its `value` attribute opened with a Unicode left double quotation mark (`"`, U+201C) instead of a standard ASCII double quote (`"`, U+0022). This means the HTML attribute was not properly opened, causing parsers to potentially misread the attribute boundary.

### Expected Behavior

The BPMH save button should render its label correctly, with the `value` attribute using valid ASCII double quotes so that JSP compilers and browsers parse the attribute as intended.

### Current Behavior

The opening quote of the `value` attribute was a Unicode "smart quote," which is not a valid HTML attribute delimiter. This could cause the button label to render incorrectly or the attribute to be silently dropped, depending on the parser's strictness.

### Affected Components

- `src/main/webapp/WEB-INF/jsp/form/pharmaForms/formBPMH.jsp` specifically the `<input type="submit">` element for the BPMH form save action.

---

## Reproduction Process

### Environment Setup

Set up the project locally following the standard contributing guide. The main challenge was ensuring the Java/Maven environment matched the project's requirements. Solved by carefully following the README and checking the project's issue tracker for common setup pitfalls.

### Steps to Reproduce

1. Open `src/main/webapp/WEB-INF/jsp/form/pharmaForms/formBPMH.jsp` in a hex-aware editor or run a character inspection script.
2. Locate the BPMH save submit button element.
3. Observe that the opening quote of the `value` attribute is `"` (U+201C) rather than `"` (U+0022).

### Reproduction Evidence

- **File showing issue:** [formBPMH.jsp on fix branch](https://github.com/siddhartha928/carlos/blob/fix/2636-malformed-quote-formBPMH/src/main/webapp/WEB-INF/jsp/form/pharmaForms/formBPMH.jsp)
- **Screenshots/logs:** N/A — character-level inspection confirmed the smart quote visually.
- **My findings:** The malformed quote was confirmed by comparing byte values. The fix is a one-character substitution with no logic change.

---

## Solution Approach

### Analysis

The root cause is a copy-paste or editor artifact: a "smart quote" (typographic left double quotation mark, U+201C) was introduced in place of a plain ASCII double quote. HTML parsers treat `"` as a literal text character, not an attribute delimiter, so the attribute value is either broken or absorbed into surrounding text depending on the parser.

### Proposed Solution

Replace the Unicode left double quotation mark with a standard ASCII double quote on the opening side of the `value` attribute. No other changes to the file are needed.

### Implementation Plan

Using the UMPIRE framework (adapted):

**Understand:** A single character in a JSP attribute delimiter is wrong — U+201C instead of U+0022 — causing potential misparse of the save button's label.

**Match:** This is a standard JSP/HTML quoting convention used consistently throughout the rest of the file and the codebase. No special pattern is needed; the fix mirrors every other properly-quoted attribute in the file.

**Plan:**
1. Open `formBPMH.jsp` and locate the BPMH save submit button.
2. Replace the opening `"` (U+201C) with `"` (U+0022) on the `value` attribute.
3. Verify no other attributes or lines in the file were similarly affected.
4. Confirm JSP syntax validity.

**Implement:** [View fix on branch](https://github.com/siddhartha928/carlos/blob/fix/2636-malformed-quote-formBPMH/src/main/webapp/WEB-INF/jsp/form/pharmaForms/formBPMH.jsp)

**Review:**
- ✅ Only the target character is changed.
- ✅ No logic, structure, or other attributes modified.
- ✅ Follows project's JSP conventions.
- ✅ Commit is signed off (DCO) and follows Conventional Commits format.

**Evaluate:** Visual inspection of the file confirms the correct character. JSP compile and browser rendering of the form show the save button label correctly.

---

## Testing Strategy

### Existing Test Suite Verification
Ran the full project test suite against my fix branch to verify no regressions:
Tests run: 404, Failures: 0, Errors: 0, Skipped: 0
[INFO] BUILD SUCCESS

All 404 existing tests pass on my fix branch. Since the change is a single-character Unicode correction in a JSP file with no logic change, this is the expected outcome and confirms the fix introduces no regressions.

### Unit Tests

- [x] No unit tests required — this is a markup-level character fix with no logic change.

### Integration Tests

- [x] JSP compiles without errors after the change.
- [x] BPMH form loads and the save button renders the correct label from the message key.

### Regression Test
- [x] Add regression test to verify the UI change   https://github.com/siddhartha928/carlos/commit/a42e01487e4833afa8f07e3edb9d6d9412a10437

### Manual Testing

Visually inspected the file before and after the change to confirm only the target character was modified. Verified JSP syntax remained valid. Confirmed no other lines in the file contained similar smart-quote issues.

---

## Implementation Notes

### Week 1 Progress

Identified the issue, reproduced it via file inspection, confirmed the exact Unicode codepoint of the offending character, and made the single-character fix. No unexpected challenges — the scope was tightly defined and the fix was straightforward.

### Code Changes

| Item | Detail |
|------|--------|
| **Files modified** | `src/main/webapp/WEB-INF/jsp/form/pharmaForms/formBPMH.jsp` |
| **Pull Request** | [PR #2731](https://github.com/carlos-emr/carlos/pull/2731) |
| **Approach** | Minimal one-character change to satisfy acceptance criteria without risk of unrelated regressions |

---

## Pull Request

**PR Link:** [carlos-emr/carlos #2731](https://github.com/carlos-emr/carlos/pull/2731)

### PR Description

Fixes a malformed `value` attribute on the BPMH save submit button in `formBPMH.jsp`. The opening quote was a Unicode left double quotation mark (`"`) instead of a standard ASCII double quote (`"`). This could cause the attribute to be parsed incorrectly and render the save button label incorrectly.

**Before:**
```jsp
value="<fmt:message key="colcamex.formBPMH.save"/>"
```

**After:**
```jsp
value="<fmt:message key="colcamex.formBPMH.save"/>"
```

### Maintainer Feedback

| Date | Feedback |
|------|----------|
| Jun 8, 2026 | *"Awesome work @siddhartha928! Everything looks good! (Sonar failure isn't a you issue, it's a secret token issue so dw about it). Thanks so much! Let me know if you need anything else/want another PR :)"* |

**Status:** ✅ Merged

---

## Learnings & Reflections

### Technical Skills Gained

Learned to distinguish Unicode typographic quotes from ASCII delimiter quotes at the byte level — a subtle class of bug that is easy to miss in standard code review but can silently break HTML parsing. Also practiced the full open-source PR workflow: forking, scoped commits, DCO sign-off, and Conventional Commits formatting.

### Challenges Overcome

The main challenge was recognizing that what looks like a normal quote in most editors is actually a different Unicode character. Using a hex-aware editor or running a character inspection confirmed the issue definitively.

### What I'd Do Differently Next Time

I would add a note in the PR description about whether a linter or encoding check could catch this class of issue automatically, and potentially suggest it to the maintainers as a follow-up improvement.

---

## Resources Used

- [Unicode Character U+201C — Left Double Quotation Mark](https://www.compart.com/en/unicode/U+201C)
- [HTML attribute syntax — MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes)
- [JSP syntax reference — Oracle Java EE 5 Tutorial](https://docs.oracle.com/javaee/5/tutorial/doc/bnakq.html)
- Project's `CONTRIBUTING.md` and [Conventional Commits specification](https://www.conventionalcommits.org/)
