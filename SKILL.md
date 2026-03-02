---
name: cover-letter
description: Use when the user wants to write or generate a cover letter for a job application. Triggers on requests to apply for jobs, draft cover letters, or customize application materials.
argument-hint: [company-name] [role-title]
---

# Cover Letter Generator

Generate a tailored cover letter by cross-referencing CVs, past cover letters, and a job description.

## Directory Layout

- `source-materials/cv-short.docx` — Short CV (1-2 pages, the version submitted to employers)
- `source-materials/cv-long.docx` — Long CV (full academic/detailed history)
- `source-materials/cover-letters/original/` — Hand-written cover letters (gold standard for voice and tone)
- `source-materials/cover-letters/successful/` — Generated letters that landed interviews (supplementary reference)
- `applications/YYYY-MM-DD-company-role/` — Per-application output folder
- `applications-log.csv` — Tracker: date, company, role, status, priority, folder, notes

## Phase 1: Startup

1. Read `applications-log.csv`. Present the current contents to the user and ask: **"Is your applications log up to date? Any statuses to update before we begin?"** Wait for a response before continuing.

2. **Promotion scan** — Scan CSV for rows where `status` = `successful`. For each, check if `source-materials/cover-letters/successful/` already contains a copy. If not, verify the source file exists and copy it:
   ```bash
   cp "applications/<folder>/cover-letter.md" "source-materials/cover-letters/successful/<folder>-cover-letter.md"
   ```
   If `cover-letter.md` is not found in an application folder, skip that entry and inform the user.

3. Ask the user: **Thorough or Fast mode?**
   - **Thorough**: Full interactive Q&A, best for high-priority applications.
   - **Fast**: Minimal questions, uses best-available material, good for bulk applications.

4. Ask for the **job description**. Offer three options:
   - Paste the text directly
   - Provide a file path (`.html`, `.docx`, `.pdf`)
   - Provide a URL to fetch (if fetch fails, ask the user to paste the text or provide a downloaded file instead)

5. Get the **company name** and **role title**. Use `$1` and `$2` from arguments if provided; otherwise ask.

6. Create the application folder: `applications/YYYY-MM-DD-<company>-<role>/` (slugified, lowercase, hyphens). Save the JD into it:
   - Pasted text: save as `job-description.md`
   - File: copy the original into the folder
   - URL: fetch with WebFetch and save as `job-description.html`

## Phase 2: Reading & Extraction

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

Do NOT read cover letters from individual `applications/` folders as source material — only use `original/` and `successful/` to prevent AI voice drift.

## Phase 3: Analysis

Parse the job description into structured criteria grouped as:
- **Required Qualifications**
- **Desired Skills**
- **Values / Culture Fit**
- **Key Responsibilities**

Cross-reference each criterion against ALL source materials (both CVs and all cover letters). Classify each as:

- **Strong match** — Clear, specific evidence in the user's materials. Note the source.
- **Partial match** — Relevant experience exists but needs expansion. Note what's there and what's missing.
- **No match** — No evidence found. Flag for user input.

## Phase 4: Interactive Gap-Filling

### Thorough Mode

1. Present the full gap analysis as a summary table (criterion, match status, evidence source).
2. For each **partial match**: ask the user to write a few sentences expanding on their relevant experience. Ask one at a time.
3. For each **no match**: ask if they have relevant experience the analysis might have missed.
4. Save all Q&A to `applications/<folder>/notes.md`.

### Fast Mode

1. Only ask about **no match** items where no reasonable inference can be made from existing materials.
2. For partial matches, use best-available content from source materials without asking.
3. If there are no critical gaps, skip straight to drafting.

## Phase 5: Drafting

Write the cover letter with these constraints:

- **Voice and tone**: Match the user's original hand-written cover letters. These are the gold standard. Do NOT default to generic AI cover letter style.
- **Structure**: Follow conventions from existing letters (greeting style, paragraph structure, sign-off).
- **Content**: Map strong matches directly. Incorporate user's answers for partial/no matches. Weave in specific examples from the long CV where they strengthen the case.
- **Length**: Match the length of the user's existing cover letters unless the job explicitly requests otherwise.
- **Sector awareness**: These are primarily government and public sector applications. Also handles consulting, public policy, and government relations roles. Use appropriate register -- professional but not corporate-buzzwordy.

Steps:
1. Present the full draft to the user.
2. Ask if they want changes. Iterate until approved.
3. Save approved letter as `applications/<folder>/cover-letter.md`.
4. Convert to docx:
   ```bash
   pandoc -f markdown -t docx -o "applications/<folder>/cover-letter.docx" "applications/<folder>/cover-letter.md"
   ```

## Phase 6: Wrap-up

1. Append a row to `applications-log.csv`:
   ```bash
   echo "YYYY-MM-DD,\"<company>\",\"<role>\",draft,<thorough|fast>,<folder-name>," >> applications-log.csv
   ```
   Use today's actual date, the real company/role names, and the folder name created earlier.

2. Print: **"Don't forget to update the status in applications-log.csv when you hear back!"**

## Future Features

- [ ] **Company research step** — After receiving the company name and JD, do a web search of the company (recent news, culture, strategic priorities) and search for advice on writing successful applications to that specific organisation. Use findings to inform the analysis and drafting phases.
- [ ] **Red team review** — After the draft is complete and before presenting to the user, run a critical review pass: identify areas for improvement, flag unclear language, suggest ways to make outcomes more concrete and impactful, check for vague or generic phrasing, and tighten the overall argument. Present the review findings alongside the draft.
