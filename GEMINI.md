# CareerGravity — Project Rules
# These rules apply whenever Antigravity works inside this repository.

## Project Purpose
This is a skill-driven job application system that runs entirely inside the
Antigravity IDE. No Python code. No external runtime. The IDE agent reads
skills, manages the candidate library, and produces tailored application
documents through conversation.

The **Candidate Library** (`data/bio.json`) is the single source
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
- **Candidate Library Verification:** Checking and maintaining complete and accurate information in `data/bio.json` is critical for ensuring maximum relevance, ATS score optimization, and quality across all generated application documents.
- **Batch Ingestion:** When asked to initialize or ingest CVs from `data/source_cvs/`, parse and merge all available CV files (`.md`, `.txt`, `.pdf`, `.doc`, `.docx`) sequentially into `data/bio.json` following the deduplication and union merge rules in `candidate_library` skill.


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
- The candidate library lives at `data/bio.json` and serves as the single source of truth (including personal profile, contact routing, and work authorizations).
- Source CVs go in `data/source_cvs/` — supported formats: Markdown (`.md`), plain text (`.txt`), PDF (`.pdf`), and Word (`.doc`, `.docx`).
- PDF and Word (DOC/DOCX) files are read automatically (PDF via `view_file`, DOC/DOCX via `textutil` on macOS, standard Python `zipfile/xml` or CLI parsers on Linux, and PowerShell on Windows) without requiring manual copy-pasting.
- Each submission is saved to `data/submissions/<company>_<role>_YYYYMMDD/` (including tailored documents, match report, compensation benchmark report, and raw JD saved as `jd_source.md`).
- Output documents are always Markdown (`.md`).

## Personal Data & Contact Routing Rules
- All candidate personal information (name, email, profiles, work authorization) and regional contact routing rules are defined directly in `data/bio.json`.
- When generating documents, consult `data/bio.json` (`contact_routing_rules`, `work_authorization`) to select the appropriate contact details.

## Library Rules
- The library is NEVER overwritten from scratch — only merged into.
- Existing experience bullets are deduplicated before appending.
- Enrichments (Q&A answers) are deduplicated by topic+position_context.
- The Matcher must NOT generate a gap question for a topic already in the library.
- The Writer MUST include all library enrichments in its context prompt.

## CV ↔ Library Sync Rule
- Whenever a bullet, skill, tool, or any other field is edited in a CV or
  submission document, the same change MUST be applied to `data/bio.json`
  immediately — unless the user explicitly says:
  - "only update this file"
  - "don't update the library"
  - "submission only"
- Find the matching entry in the library by `(company, title, dates_from)` for
  experience bullets, or by list membership for skills/tools.
- Also update `library.updated_at` to the current ISO timestamp on every save.

## Source CV Consultation Rule
- When creating a submission, inspect `data/source_cvs/` for role-relevant CVs (e.g., Data Analyst, Solutions Manager, Developer).
- If relevant CVs exist, consult their specific phrasing, bullet structuring, and highlighted achievements to tailor the submission documents alongside the candidate library.

## Strict Grounding, Domain Boundaries & Zero Fabrication Rules
- **Zero Fabrication Principle:** NEVER fabricate, invent, or assume any experience, technologies, tools, programming languages, platforms, formal frameworks, standards, certifications, or metrics (e.g. ITIL, TOGAF, ISO, Jira Service Management, specific cloud services, unverified software). If a technology, tool, standard, or qualification is not explicitly recorded in `data/bio.json`, it does not exist for the candidate.
- **No Domain Mixing:** Never confuse, conflate, or mix up distinct or adjacent domains (e.g., IT/Enterprise IT vs Telecommunications, Software Development vs Infrastructure/Network Engineering, Data Analytics vs Data Engineering). Ground each role and achievement accurately within its true industry and functional domain.
- **Mandatory Gap Questioning for Missing Requirements:** Whenever a job description requires a technology, tool, platform, standard, framework, or domain background not present in the candidate library, treat it as a hard gap. Actively interview the user during the Gap Q&A step (Step 6) to ask how to address or bridge the gap before generating any application documents.
- **Authentic Grounding:** Describe and frame ONLY the candidate's *specified, verified* experience, real tools, and genuine transferable competencies. Never auto-fill unverified skills or tools.

## Cover Letter Alignment Rule
- When generating cover letters, explicitly articulate the relationship between the candidate's previous experience and the job description requirements across three pillars:
  1. **Qualifications:** Map verified domain qualifications and responsibilities to the JD's requirements.
  2. **Tools:** Connect specific tools/technologies actually used in past work to the JD's required tech stack (including verified transferable parallels).
  3. **Metrics:** Anchor qualification and tool alignments in concrete, verified quantified outcomes achieved in prior roles.

## Example Data Maintenance Rule
- `data_example/` contains the version-controlled reference example of the entire `data/` folder (including `data_example/bio.json`, `data_example/source_cvs/`, and `data_example/submissions/`).
- Whenever agent changes affect the structure, schemas, formatting, or document templates of the candidate library (`data/bio.json`), source CVs, or submission packages (`data/submissions/`), you MUST update `data_example/` accordingly to keep the example content valid, synchronized, and up to date.
- **Strict Anonymization & Privacy in `data_example/`:** NEVER include real personal contact data (real email addresses, real personal phone numbers, or private direct contact details) in `data_example/`. Always use placeholder contact information (such as `@example.com` emails, example phone numbers like `+36 20 123 4567`, and generic profile links) across all example files.

## Security
- Never log or print any API keys.
- Never commit `.env` files.
