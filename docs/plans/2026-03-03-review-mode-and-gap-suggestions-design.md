# Design: Review Mode & Gap-Filling Suggestions

## Summary

Two enhancements to the cover-letter skill:

1. **Review mode** — A third mode (alongside Thorough/Fast) for reviewing hand-written cover letters against a job description, producing feedback and targeted edits.
2. **Gap-filling suggestions** — During Phase 5 (Thorough/Fast modes), offer concrete suggestions for how to frame existing experience when addressing gaps, rather than just asking the user to provide more info.

## Feature 1: Review Mode

### Phase 1 Changes

The mode selection prompt becomes three options: **Thorough, Fast, or Review**.

When Review is selected:
- Steps 4-6 proceed as normal (get JD, company name, role title, create folder).
- An additional step asks the user for their existing cover letter — same three input options as the JD (paste text, file path, or URL).
- The letter is saved as `cover-letter-draft.md` in the application folder (distinct from the final `cover-letter.md`).

### Phases 2-4: Unchanged

Company research, reading & extraction, and analysis all run exactly as today. In Review mode, the user's submitted letter is read during Phase 3, but treated as the artifact under review — not as source material for voice matching.

### New Phase 5R: Review (replaces Phases 5 and 6 in Review mode)

When Review mode is selected, skip normal Phase 5 (gap-filling) and Phase 6 (drafting). Instead:

**1. Coverage analysis**

Using the structured criteria from Phase 4, evaluate the user's letter against each criterion:
- **Addressed strongly** — Clear, evidence-backed case for this criterion.
- **Addressed weakly** — Mentioned but vague, unsupported, or undercooked.
- **Missing** — Not addressed, but the user's source materials contain relevant experience.

Present as a summary table.

**2. Red team review**

Same three-part review as current Phase 6 (argument strength, voice authenticity, overall tightness) — but assessed against the user's own writing style from their original letters, not against a generated draft.

**3. Targeted edits**

For each weak or missing area, offer a specific rewrite of just that section — a replacement paragraph or sentence — drawing on the user's source materials and company research. Edits match the user's voice as closely as possible.

Present all three together: coverage table, red team findings, and targeted edits. Iterate on user feedback until approved. Save final version as `cover-letter.md` / `cover-letter.docx` as normal.

### Phase 7: Unchanged

Wrap-up runs the same regardless of mode.

## Feature 2: Gap-Filling Suggestions

### Applies to: Phase 5 (Thorough and Fast modes only)

When presenting a partial or no match during gap-filling, the skill also offers a concrete suggestion drawing on the user's source materials and company research.

**Partial match format:**

> **Partial match — [Criterion]**
> Your CV mentions [specific experience from materials]. Could you expand on that? For example, you could frame [aspect] as [suggested framing] — that would align well with [company]'s emphasis on [relevant company priority from research].

**No match format:**

For no-match items, search the long CV more creatively for adjacent experience and transferable skills:

> **No match — [Criterion]**
> I didn't find a direct match, but your work on [X] involved [Y], which could be relevant here. Does that resonate?

The user can take the suggestion, ignore it, or go a different direction. All responses still saved to `notes.md` as today.
