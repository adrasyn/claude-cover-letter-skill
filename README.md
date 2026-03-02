# Cover Letter Skill for Claude Code

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that generates tailored cover letters by cross-referencing your CVs, past cover letters, and a job description.

## What it does

When you run `/cover-letter`, the skill:

1. **Reads your source materials** — CVs, hand-written cover letters, and any previously successful generated letters
2. **Parses the job description** into structured criteria (qualifications, skills, values, responsibilities)
3. **Cross-references** each criterion against your materials, classifying matches as strong, partial, or missing
4. **Fills gaps interactively** — asks you targeted questions about partial/no matches
5. **Drafts a cover letter** that matches your voice and tone (from your original letters), maps evidence to criteria, and outputs markdown + docx
6. **Tracks applications** in a CSV log, with automatic promotion of successful letters to the reference pool

Two modes: **Thorough** (full Q&A, best for high-priority applications) and **Fast** (minimal questions, good for bulk applications).

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Pandoc](https://pandoc.org/installing.html) (for docx extraction and conversion)

## Installation

From your job applications project directory:

```bash
claude skill add --name cover-letter https://github.com/adrasyn/claude-cover-letter-skill
```

Or manually copy `SKILL.md` to `.claude/skills/cover-letter/SKILL.md` in your project.

## Directory setup

The skill expects this structure in your project root:

```
source-materials/
  cv-short.docx          # Short CV (1-2 pages, submitted to employers)
  cv-long.docx           # Long CV (full detailed history)
  cover-letters/
    original/            # Your hand-written cover letters (gold standard for voice/tone)
    successful/          # Generated letters that landed interviews (auto-populated)
applications/            # Per-application output folders (auto-created)
applications-log.csv     # Application tracker (auto-created)
```

Create the CSV with this header:

```csv
date,company,role,status,priority,folder,notes
```

## Usage

```
/cover-letter
/cover-letter "Company Name" "Role Title"
```

## Customisation

The skill is designed to be adapted to your sector. Edit the **Sector awareness** line in `SKILL.md` to match your field — the default is tuned for government, public sector, consulting, and public policy roles.

## License

MIT
