# CareerGravity — Project Rules
# These rules apply whenever Antigravity works inside this repository.

## Project Purpose
This is a skill-driven job application system that runs entirely inside the
Antigravity IDE. No Python code. No external runtime. The IDE agent reads
skills, manages the candidate library, and produces tailored application
documents through conversation.

The **Candidate Library** (`data/candidate_library.json`) is the single source
of truth for everything known about the candidate. It grows with every CV parse
and every Q&A session. Documents are always generated from the library.

## How to Trigger the Pipeline
Say any of the following:
- "Apply for [Company] / [Role]"
- "Process my CV for [role]"
- "Prepare an application for [URL]"
- "Run the pipeline"

The agent reads the **pipeline** skill and executes all 8 steps.

## Skill Files
All skills live in `.agents/skills/`:

| Skill | Purpose |
|---|---|
| `pipeline` | Master 8-step pipeline orchestrator |
| `candidate_library` | Library schema, merge rules, enrichment storage |
| `cv_parsing` | CV text extraction rules |
| `jd_analysis` | JD structured extraction rules |
| `matching` | Match scoring algorithm & gap question generation |
| `writing_guidelines` | Style, ATS, tone, document structure |
| `folder_management` | Submission folder naming conventions |

## File & Folder Conventions
- Personal profile & contact routing configuration lives at `data/personal_profile.json`.
- Source CVs go in `data/source_cvs/` — Markdown or plain text.
- The candidate library lives at `data/candidate_library.json`.
- Each submission is saved to `data/submissions/<company>_<role>_YYYYMMDD/` (including tailored documents, match report, and raw JD saved as `jd_source.md`).
- Output documents are always Markdown (`.md`).

## Personal Data & Contact Routing Rules
- All candidate personal information (name, email, profiles, work authorization) and regional contact routing rules are defined in `data/personal_profile.json`.
- When generating documents, consult `data/personal_profile.json` to select the appropriate contact details.

## Library Rules
- The library is NEVER overwritten from scratch — only merged into.
- Existing experience bullets are deduplicated before appending.
- Enrichments (Q&A answers) are deduplicated by topic+position_context.
- The Matcher must NOT generate a gap question for a topic already in the library.
- The Writer MUST include all library enrichments in its context prompt.

## CV ↔ Library Sync Rule
- Whenever a bullet, skill, tool, or any other field is edited in a CV or
  submission document, the same change MUST be applied to `data/candidate_library.json`
  immediately — unless the user explicitly says:
  - "only update this file"
  - "don't update the library"
  - "submission only"
- Find the matching entry in the library by `(company, title, dates_from)` for
  experience bullets, or by list membership for skills/tools.
- Also update `library.updated_at` to the current ISO timestamp on every save.

## Source CV Consultation Rule
- When creating a submission, always inspect `data/source_cvs/` for any role-relevant CVs (e.g., Data Analyst, Solutions Manager, Developer).
- If relevant CVs exist, consult their specific phrasing, bullet structuring, and highlighted achievements to tailor the submission documents alongside the candidate library.

## Grounding & Authenticity Rule
- Never invent or fabricate unspecified experience, credentials, tools, or metrics to match a job description.
- Describe and reframe how the candidate's *specified, verified* experience and transferable skills fulfill the requirements of the JD.

## Security
- Never log or print any API keys.
- Never commit `.env` files.
