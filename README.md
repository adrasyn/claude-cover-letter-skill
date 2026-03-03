# Cover Letter Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that generates tailored cover letters by cross-referencing your CVs, past cover letters, and a job description.

## What it does

When you run `/cover-letter`, the skill:

1. **Reads your source materials** — CVs, hand-written cover letters, and any previously successful generated letters
2. **Researches the company** — runs web searches for recent news, strategy, values, and culture to inform the letter
3. **Parses the job description** into structured criteria (qualifications, skills, values, responsibilities)
4. **Cross-references** each criterion against your materials, classifying matches as strong, partial, or missing
5. **Fills gaps interactively** — asks you targeted questions about partial/no matches, with concrete framing suggestions
6. **Drafts a cover letter** that matches your voice and tone (from your original letters), maps evidence to criteria, and saves as markdown for you to edit
7. **Red teams the draft** — runs a critical self-review checking argument strength, voice authenticity, and tightness before presenting findings
8. **Tracks applications** in a CSV log, with automatic promotion of successful letters to the reference pool

Three modes:
- **Thorough** — Full interactive Q&A, best for high-priority applications.
- **Fast** — Minimal questions, uses best-available material, good for bulk applications.
- **Review** — Critique an existing hand-written cover letter against the JD. Produces a coverage analysis, red team review, and targeted edits rather than a new draft.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Pandoc](https://pandoc.org/installing.html) (for docx extraction and conversion)

## Installation

From your job applications project directory:

```bash
claude skill add --name cover-letter https://github.com/adrasyn/claude-cover-letter-skill
```

Or manually copy `SKILL.md` to `.claude/skills/cover-letter/SKILL.md` in your project.

## Getting started

The first time you run `/cover-letter` in a new workspace, **Phase 0** detects the missing structure and walks you through setup:

1. Creates all required directories and the application tracker CSV
2. Asks what **sector or field** you work in (saved to `profile.md` — used to tune tone and register)
3. Tells you which files to add manually (CVs and at least one hand-written cover letter)

Once setup is complete, Phase 0 is silent on subsequent runs.

### Expected directory structure

```
source-materials/
  cv-short.docx          # Short CV (1-2 pages, submitted to employers)
  cv-long.docx           # Long CV (full detailed history)
  cover-letters/
    original/            # Your hand-written cover letters (gold standard for voice/tone)
    successful/          # Generated letters that landed interviews (auto-populated)
applications/            # Per-application output folders (auto-created)
applications-log.csv     # Application tracker (auto-created)
profile.md               # Your sector/field (auto-created during setup)
```

## Usage

```
/cover-letter
/cover-letter "Company Name" "Role Title"
```

## License

MIT
