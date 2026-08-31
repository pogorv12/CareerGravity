# CareerGravity — Project Rules
# These rules apply whenever Antigravity works inside this repository.

## Project Purpose
This is a skill-driven job application system that runs entirely inside the
Antigravity IDE. No Python code. No external runtime. The IDE agent reads
skills, manages the candidate library, and produces tailored application
documents through conversation.

The **Candidate Library** (`library/candidate.json`) is the single source
of truth for everything known about the candidate. It grows with every CV parse
and every Q&A session. Documents are always generated from the library.

## How to Trigger the Pipeline
Say or provide any of the following:
- "Initialize candidate library from all source CVs" / "Process all CVs"
- "Apply for [Company] / [Role]"
- "Process my CV for [role]"
- "Prepare an application for [URL]"
- "Run the pipeline"
- **Passing a vacancy URL or job description text directly**: If just a vacancy URL or job description text is passed at the beginning of the conversation, treat it immediately as the order to create submission documents and run the full 8-step pipeline.

The agent reads the **pipeline** skill and executes all 8 steps.

## Library Initialization & Data Relevance Rule
- **Candidate Library Verification:** Checking and maintaining complete and accurate information in `library/candidate.json` is critical for ensuring maximum relevance, ATS score optimization, and quality across all generated application documents.
- **Batch Ingestion:** When asked to initialize or ingest CVs from `library/source_cvs/`, parse and merge all available CV files (`.md`, `.txt`, `.pdf`, `.doc`, `.docx`) sequentially into `library/candidate.json` following the deduplication and union merge rules in `candidate_library` skill.


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
- The candidate library lives at `library/candidate.json` and serves as the single source of truth (including personal profile, contact routing, and work authorizations).
- Source CVs go in `library/source_cvs/` — supported formats: Markdown (`.md`), plain text (`.txt`), PDF (`.pdf`), and Word (`.doc`, `.docx`).
- PDF and Word (DOC/DOCX) files are read automatically (PDF via `view_file`, DOC/DOCX via `textutil` on macOS, standard Python `zipfile/xml` or CLI parsers on Linux, and PowerShell on Windows) without requiring manual copy-pasting.
- Each submission is saved to `submissions/<company>_<role>_YYYYMMDD/` (including tailored documents, match report, and raw JD saved as `jd_source.md`).
- Output documents are always Markdown (`.md`).

## Personal Data & Contact Routing Rules
- All candidate personal information (name, email, profiles, work authorization) and regional contact routing rules are defined directly in `library/candidate.json`.
- When generating documents, consult `library/candidate.json` (`contact_routing_rules`, `work_authorization`) to select the appropriate contact details.

## Library Rules
- The library is NEVER overwritten from scratch — only merged into.
- Existing experience bullets are deduplicated before appending.
- Enrichments (Q&A answers) are deduplicated by topic+position_context.
- The Matcher must NOT generate a gap question for a topic already in the library.
- The Writer MUST include all library enrichments in its context prompt.

## CV ↔ Library Sync Rule
- Whenever a bullet, skill, tool, or any other field is edited in a CV or
  submission document, the same change MUST be applied to `library/candidate.json`
  immediately — unless the user explicitly says:
  - "only update this file"
  - "don't update the library"
  - "submission only"
- Find the matching entry in the library by `(company, title, dates_from)` for
  experience bullets, or by list membership for skills/tools.
- Also update `library.updated_at` to the current ISO timestamp on every save.

## Source CV Consultation Rule
- When creating a submission, inspect `library/source_cvs/` for role-relevant CVs (e.g., Data Analyst, Solutions Manager, Developer).
- If relevant CVs exist, consult their specific phrasing, bullet structuring, and highlighted achievements to tailor the submission documents alongside the candidate library.

## Grounding & Authenticity Rule
- Never invent or fabricate unspecified experience, credentials, tools, or metrics to match a job description.
- Describe and reframe how the candidate's *specified, verified* experience and transferable skills fulfill the requirements of the JD.

## Cover Letter Alignment Rule
- When generating cover letters, explicitly articulate the relationship between the candidate's previous experience and the job description requirements across three pillars:
  1. **Qualifications:** Map verified domain qualifications and responsibilities to the JD's requirements.
  2. **Tools:** Connect specific tools/technologies used in past work to the JD's required tech stack (including transferable parallels).
  3. **Metrics:** Anchor qualification and tool alignments in concrete, quantified outcomes achieved in prior roles.

## Security
- Never log or print any API keys.
- Never commit `.env` files.
