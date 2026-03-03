# Review Mode & Gap-Filling Suggestions Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a Review mode for critiquing hand-written cover letters, and add concrete suggestions during gap-filling in Thorough/Fast modes.

**Architecture:** Both features modify `SKILL.md` only. Review mode adds a third mode option in Phase 1 and a new Phase 5R that replaces Phases 5-6 when active. Gap-filling suggestions enhance the existing Phase 5 prompts.

**Tech Stack:** Markdown (skill definition file)

**Design doc:** `docs/plans/2026-03-03-review-mode-and-gap-suggestions-design.md`

---

### Task 1: Update skill description and directory layout

**Files:**
- Modify: `SKILL.md:1-18`

**Step 1: Update the frontmatter description**

Change line 2 from:
```
description: Use when the user wants to write or generate a cover letter for a job application. Triggers on requests to apply for jobs, draft cover letters, or customize application materials.
```
to:
```
description: Use when the user wants to write, review, or generate a cover letter for a job application. Triggers on requests to apply for jobs, draft cover letters, review existing cover letters, or customize application materials.
```

**Step 2: Update directory layout**

Add `cover-letter-draft.md` to the per-application folder listing. Change line 17 from:
```
- `applications/YYYY-MM-DD-company-role/` — Per-application output folder containing: `job-description.*`, `notes.md`, `research.md`, `cover-letter.md`, `cover-letter.docx`
```
to:
```
- `applications/YYYY-MM-DD-company-role/` — Per-application output folder containing: `job-description.*`, `notes.md`, `research.md`, `cover-letter-draft.md` (Review mode only — the user's original letter), `cover-letter.md`, `cover-letter.docx`
```

**Step 3: Commit**

```bash
git add SKILL.md
git commit -m "feat: update skill description and layout for review mode"
```

---

### Task 2: Update Phase 1 — add Review mode and letter input

**Files:**
- Modify: `SKILL.md:30-44`

**Step 1: Update the mode selection (step 3)**

Replace lines 30-32:
```markdown
3. Ask the user: **Thorough or Fast mode?**
   - **Thorough**: Full interactive Q&A, best for high-priority applications.
   - **Fast**: Minimal questions, uses best-available material, good for bulk applications.
```

with:
```markdown
3. Ask the user: **Thorough, Fast, or Review mode?**
   - **Thorough**: Full interactive Q&A, best for high-priority applications.
   - **Fast**: Minimal questions, uses best-available material, good for bulk applications.
   - **Review**: Critique an existing hand-written cover letter against the JD. Produces feedback and targeted edits rather than a new draft.
```

**Step 2: Add letter input step for Review mode**

After step 6 (line 44), add a new step 7:
```markdown
7. **Review mode only** — Ask for the **existing cover letter** to review. Offer the same three input options as the JD:
   - Paste the text directly
   - Provide a file path (`.html`, `.docx`, `.pdf`)
   - Provide a URL to fetch
   Save the letter into the application folder as `cover-letter-draft.md` (converting to plain text first if needed, using the same extraction commands from Phase 3).
```

**Step 3: Commit**

```bash
git add SKILL.md
git commit -m "feat: add Review mode option and letter input to Phase 1"
```

---

### Task 3: Update Phase 3 — read the user's letter in Review mode

**Files:**
- Modify: `SKILL.md:78-84`

**Step 1: Add the user's letter to the reading order**

After the existing reading list item 4 (line 82-83), add a new item 5:
```markdown
5. **Review mode only** — The user's cover letter from `applications/<folder>/cover-letter-draft.md`. This is the artifact under review — do NOT treat it as source material for voice matching.
```

**Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: read user's letter during extraction in Review mode"
```

---

### Task 4: Update Phase 5 — add suggestions during gap-filling

**Files:**
- Modify: `SKILL.md:102-115`

**Step 1: Replace the Phase 5 content**

Replace the entire Phase 5 section (lines 102-115) with:
```markdown
## Phase 5: Interactive Gap-Filling

**Applies to Thorough and Fast modes only.** Review mode skips to Phase 5R.

### Thorough Mode

1. Present the full gap analysis as a summary table (criterion, match status, evidence source).
2. For each **partial match**: present what was found in the source materials and offer a concrete suggestion for how to frame it, then ask the user to expand. Ask one gap at a time. Format:
   > **Partial match — [Criterion]**
   > Your CV mentions [specific experience from materials]. Could you expand on that? For example, you could frame [aspect of their experience] as [suggested framing] — that would align well with [company]'s emphasis on [relevant priority from research.md].
3. For each **no match**: search the long CV for adjacent experience or transferable skills and suggest a connection, then ask if it resonates. Format:
   > **No match — [Criterion]**
   > I didn't find a direct match, but your work on [X] involved [Y], which could be relevant here. Does that resonate, or do you have other experience I might have missed?
4. Save all Q&A (including the suggestions offered) to `applications/<folder>/notes.md`.

### Fast Mode

1. Only ask about **no match** items where no reasonable inference can be made from existing materials.
2. For partial matches, use best-available content from source materials without asking. Still generate the framing suggestions internally to inform the draft.
3. For no-match items that are asked about, use the same suggestion format as Thorough mode.
4. If there are no critical gaps, skip straight to drafting.
```

**Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add concrete suggestions to Phase 5 gap-filling"
```

---

### Task 5: Add Phase 5R — Review mode path

**Files:**
- Modify: `SKILL.md` (insert after Phase 5, before Phase 6)

**Step 1: Insert the new Phase 5R section**

After the Phase 5 section and before Phase 6, insert:
```markdown
## Phase 5R: Review (Review mode only)

When Review mode is selected, skip Phase 5 (gap-filling) and Phase 6 (drafting). Run this phase instead.

### 1. Coverage Analysis

Using the structured criteria from Phase 4, evaluate the user's cover letter (`cover-letter-draft.md`) against each criterion:

- **Addressed strongly** — The letter makes a clear, evidence-backed case for this criterion.
- **Addressed weakly** — It's mentioned but vague, unsupported, or undercooked.
- **Missing** — The criterion isn't addressed at all, but the user's source materials contain relevant experience that could be used.

Present as a summary table (criterion, coverage status, notes).

### 2. Red Team Review

Run the same three-part critical review as Phase 6's Red Team Review, but assessed against the user's letter rather than a generated draft:

1. **Argument strength:**
   - Are claims backed by specific evidence and examples, not just assertions?
   - Are outcomes concrete and measurable where possible?
   - Does each paragraph advance the case, or is any filler?
   - Are the strongest selling points given enough prominence?

2. **Voice authenticity:**
   - Compare the letter's tone against the user's other original hand-written letters in `source-materials/cover-letters/original/`.
   - Flag any phrases that sound generic or formulaic.
   - Check the register matches the sector.

3. **Overall tightness:**
   - Flag vague or generic phrasing that could apply to any candidate.
   - Identify sentences that could be cut or combined without losing substance.
   - Check the opening and closing are strong — not boilerplate.

Present findings as a numbered list of specific, actionable items referencing the relevant paragraph or sentence.

### 3. Targeted Edits

For each item flagged as **addressed weakly** or **missing** in the coverage analysis, and for each red team finding, offer a specific rewrite of just that section:

- Replacement paragraph or sentence, not a full rewrite of the letter.
- Draw on the user's source materials (CVs, other cover letters) and company research to provide concrete evidence and framing.
- Match the user's voice as closely as possible by referencing their original hand-written letters.

### 4. Present and Iterate

1. Present all three together: coverage table, red team findings, and targeted edits.
2. Ask the user which edits to incorporate. Iterate until approved.
3. Save the final version as `applications/<folder>/cover-letter.md`.
4. Convert to docx:
   ```bash
   pandoc -f markdown -t docx -o "applications/<folder>/cover-letter.docx" "applications/<folder>/cover-letter.md"
   ```

After this phase, skip Phase 6 and proceed directly to Phase 7 (Wrap-up).
```

**Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add Phase 5R review path for hand-written cover letters"
```

---

### Task 6: Update Phase 6 — add mode guard

**Files:**
- Modify: `SKILL.md` (Phase 6 header area)

**Step 1: Add a mode note to Phase 6**

At the start of Phase 6 (after the `## Phase 6: Drafting` heading), add:
```markdown
**Applies to Thorough and Fast modes only.** Review mode uses Phase 5R instead.
```

**Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: add mode guard note to Phase 6"
```

---

### Task 7: Update Phase 7 — handle Review mode in log entry

**Files:**
- Modify: `SKILL.md` (Phase 7, the echo command)

**Step 1: Update the CSV append command**

Replace the existing echo command:
```bash
echo "YYYY-MM-DD,\"<company>\",\"<role>\",draft,<thorough|fast>,<folder-name>," >> applications-log.csv
```

with:
```bash
echo "YYYY-MM-DD,\"<company>\",\"<role>\",draft,<thorough|fast|review>,<folder-name>," >> applications-log.csv
```

**Step 2: Commit**

```bash
git add SKILL.md
git commit -m "feat: include review mode in applications log entries"
```

---

### Task 8: Final review and verification

**Step 1: Read the full SKILL.md end-to-end**

Read the complete file and verify:
- Phase flow is coherent: Phase 1 → 2 → 3 → 4 → (5 or 5R) → (6 or skip) → 7
- No dangling references to old phase numbers
- Review mode is consistently described across all phases
- Gap-filling suggestions are clearly specified in Phase 5
- No contradictions between phases

**Step 2: Verify the design doc is satisfied**

Cross-reference `docs/plans/2026-03-03-review-mode-and-gap-suggestions-design.md` against the implemented changes. Confirm all design requirements are addressed.

**Step 3: Commit any fixes if needed, then final commit**

```bash
git add SKILL.md
git commit -m "chore: final review pass for review mode and gap suggestions"
```
