---
name: cover-letter
description: Use when the user wants to write, review, or generate a cover letter for a job application. Triggers on requests to apply for jobs, draft cover letters, review existing cover letters, or customize application materials.
argument-hint: [company-name] [role-title]
---

# Cover Letter Generator

Generate a tailored cover letter by cross-referencing CVs, past cover letters, and a job description.

## Directory Layout

- `source-materials/cv-short.docx` — Short CV (1-2 pages, the version submitted to employers)
- `source-materials/cv-long.docx` — Long CV (full academic/detailed history)
- `source-materials/cover-letters/original/` — Hand-written cover letters (gold standard for voice and tone)
- `source-materials/cover-letters/successful/` — Generated letters that landed interviews (supplementary reference)
- `applications/YYYY-MM-DD-company-role/` — Per-application output folder containing: `job-description.*`, `notes.md`, `research.md`, `cover-letter-draft.md` (Review mode only — the user's original letter), `cover-letter.md`, `cover-letter.docx`
- `applications-log.csv` — Tracker: date, company, role, status, priority, folder, notes
- `profile.md` — User profile (sector/field, created during first-time setup)

## Phase 0: First-Time Setup

This phase runs silently before every session. If the workspace is already set up, it adds no output or delay.

1. **Check for required structure** — verify all of the following exist:
   - `source-materials/cv-short.docx`
   - `source-materials/cv-long.docx`
   - `source-materials/cover-letters/original/` (containing at least one file)
   - `source-materials/cover-letters/successful/`
   - `applications/`
   - `applications-log.csv`
   - `profile.md`

2. **If everything exists** — skip to Phase 1 immediately (no output).

3. **If anything is missing** — inform the user this looks like a fresh workspace and offer to set it up. Then:
   - Create all missing directories: `source-materials/`, `source-materials/cover-letters/original/`, `source-materials/cover-letters/successful/`, `applications/`
   - Create `applications-log.csv` with the header row if it doesn't exist:
     ```
     date,company,role,status,mode,folder,notes
     ```
   - Ask the user: **"What sector or field do you work in?"** (e.g. government & public sector, consulting, tech, finance, academia, non-profit). Use their answer to create `profile.md`:
     ```markdown
     # Profile
     - **Sector**: <their answer>
     ```
   - Tell the user which files they still need to add manually:
     - `source-materials/cv-short.docx` — Short CV (1-2 pages, the version submitted to employers)
     - `source-materials/cv-long.docx` — Long CV (full academic/detailed history)
     - At least one hand-written cover letter in `source-materials/cover-letters/original/`
   - **Wait** for the user to confirm they've added their files before continuing to Phase 1.

## Phase 1: Startup

1. Read `applications-log.csv`. Present the current contents to the user and ask: **"Is your applications log up to date? Any statuses to update before we begin?"** Wait for a response before continuing.

2. **Promotion scan** — Scan CSV for rows where `status` = `successful`. For each, check if `source-materials/cover-letters/successful/` already contains a copy. If not, verify the source file exists and copy it:
   ```bash
   cp "applications/<folder>/cover-letter.md" "source-materials/cover-letters/successful/<folder>-cover-letter.md"
   ```
   If `cover-letter.md` is not found in an application folder, skip that entry and inform the user.

3. Ask the user: **Thorough, Fast, or Review mode?**
   - **Thorough**: Full interactive Q&A, best for high-priority applications.
   - **Fast**: Minimal questions, uses best-available material, good for bulk applications.
   - **Review**: Critique an existing hand-written cover letter against the JD. Produces feedback and targeted edits rather than a new draft.

4. Ask for the **job description**. Offer three options:
   - Paste the text directly
   - Provide a file path (`.html`, `.docx`, `.pdf`)
   - Provide a URL to fetch (if fetch fails, ask the user to paste the text or provide a downloaded file instead)

5. Get the **company name** and **role title**. Use `$1` and `$2` from arguments if provided; otherwise ask.

6. Create the application folder: `applications/YYYY-MM-DD-<company>-<role>/` (slugified, lowercase, hyphens). Save the JD into it:
   - Pasted text: save as `job-description.md`
   - File: copy the original into the folder
   - URL: fetch with WebFetch and save as `job-description.html`

7. **Review mode only** — Ask for the **existing cover letter** to review. Offer the same three input options as the JD:
   - Paste the text directly
   - Provide a file path (`.html`, `.docx`, `.pdf`)
   - Provide a URL to fetch
   Save the letter into the application folder as `cover-letter-draft.md` (converting to plain text first if needed, using the same extraction commands from Phase 3).

## Phase 2: Company Research

After the application folder is created and the JD is saved, research the company to inform later analysis.

1. Run 2-3 `WebSearch` queries:
   - `"<company name>" recent news strategy 2025 2026`
   - `"<company name>" values culture mission`
   - If the role mentions a specific department or programme, one targeted search for that

2. Synthesise findings into a brief research summary covering:
   - **Recent news** — What has the organisation been doing lately?
   - **Strategic priorities** — What are they focused on?
   - **Values and culture** — How do they describe themselves?

3. Save the summary to `applications/<folder>/research.md`.

4. If WebSearch fails or returns thin results, note this in `research.md` and continue. The skill should never block on research.

This research feeds into Phase 4 (Analysis) — it helps identify which aspects of the user's experience to emphasise and what language and framing to use.

## Phase 3: Reading & Extraction

Before extraction, verify that `cv-short.docx` and `cv-long.docx` exist and that `source-materials/cover-letters/original/` contains at least one file. If any are missing, inform the user and halt.

Extract text from all source materials using these commands:

| Format | Command |
|--------|---------|
| `.docx` | `pandoc -t plain "<file>"` |
| `.pdf` | `pdftotext "<file>" -` |
| `.html` | `pandoc -f html -t plain "<file>"` |

Read, in order:
1. Both CVs (`cv-short.docx`, `cv-long.docx`)
2. All files in `source-materials/cover-letters/original/`
3. All files in `source-materials/cover-letters/successful/`
4. The job description (using the appropriate extraction method)
5. **Review mode only** — The user's cover letter from `applications/<folder>/cover-letter-draft.md`. This is the artifact under review — do NOT treat it as source material for voice matching.

Do NOT read cover letters from individual `applications/` folders as source material — only use `original/` and `successful/` to prevent AI voice drift.

## Phase 4: Analysis

Parse the job description into structured criteria grouped as:
- **Required Qualifications**
- **Desired Skills**
- **Values / Culture Fit**
- **Key Responsibilities**

Cross-reference each criterion against ALL source materials (both CVs and all cover letters). Classify each as:

- **Strong match** — Clear, specific evidence in the user's materials. Note the source.
- **Partial match** — Relevant experience exists but needs expansion. Note what's there and what's missing.
- **No match** — No evidence found. Flag for user input.

Also use the company research from Phase 2 (`research.md`) to inform which criteria to weight most heavily and what framing/language will resonate with this organisation.

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

## Phase 6: Drafting

**Applies to Thorough and Fast modes only.** Review mode uses Phase 5R instead.

Write the cover letter with these constraints:

- **Voice and tone**: Match the user's original hand-written cover letters. These are the gold standard. Do NOT default to generic AI cover letter style.
- **Structure**: Follow conventions from existing letters (greeting style, paragraph structure, sign-off).
- **Content**: Map strong matches directly. Incorporate user's answers for partial/no matches. Weave in specific examples from the long CV where they strengthen the case.
- **Length**: Match the length of the user's existing cover letters unless the job explicitly requests otherwise.
- **Sector awareness**: Read the user's sector from `profile.md` and use the appropriate register. Match the tone to the sector (e.g. professional but not corporate-buzzwordy for government; concise and results-oriented for consulting; technical precision for tech roles).

### Red Team Review

After writing the draft and before presenting it to the user, run a critical self-review:

1. **Argument strength:**
   - Are claims backed by specific evidence and examples, not just assertions?
   - Are outcomes concrete and measurable where possible? (e.g. "improved X by Y%" not "made significant improvements")
   - Does each paragraph advance the case, or is any filler?
   - Are the strongest selling points given enough prominence?

2. **Voice authenticity:**
   - Compare the draft's tone against the user's original hand-written letters.
   - Flag any phrases that sound like generic AI cover letter language (e.g. "I am excited to apply", "I believe I would be a valuable asset", "leveraging my expertise").
   - Check the register matches the user's sector (from `profile.md`).

3. **Overall tightness:**
   - Flag vague or generic phrasing that could apply to any candidate.
   - Identify sentences that could be cut or combined without losing substance.
   - Check the opening and closing are strong — not boilerplate.

Present the review as a numbered list of specific, actionable findings. Each finding should reference the relevant paragraph or sentence and suggest a concrete improvement.

Steps:
1. Save the draft as `applications/<folder>/cover-letter.md`.
2. Present the draft to the user and ask: **"Here's the draft — I've saved it to `cover-letter.md`. Feel free to open the file and make any edits you'd like before I run the red team analysis. Let me know when you're ready."** Wait for the user to respond before continuing.
3. Re-read `applications/<folder>/cover-letter.md` to pick up any changes the user made.
4. Run the red team review (above) against the current version of the file.
5. Present the red team findings to the user.
6. Ask if they want further changes. Iterate until approved.
7. Save the final approved letter as `applications/<folder>/cover-letter.md`.
8. Convert to docx:
   ```bash
   pandoc -f markdown -t docx -o "applications/<folder>/cover-letter.docx" "applications/<folder>/cover-letter.md"
   ```

## Phase 7: Wrap-up

1. Append a row to `applications-log.csv`:
   ```bash
   echo "YYYY-MM-DD,\"<company>\",\"<role>\",draft,<thorough|fast|review>,<folder-name>," >> applications-log.csv
   ```
   Use today's actual date, the real company/role names, and the folder name created earlier.

2. Print: **"Don't forget to update the status in applications-log.csv when you hear back!"**