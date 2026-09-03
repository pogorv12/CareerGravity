---
name: pipeline
description: >-
  Master pipeline for the CareerGravity job application system.
  Runs the full end-to-end workflow: CV parsing → library merge → JD analysis
  → matching → gap Q&A → document generation → submission packaging.
  Activate when the user says "apply for [role]", "process my CV", or simply
  provides a vacancy URL or job description text.
---

# CareerGravity Pipeline Skill

## Overview

You are the CareerGravity AI. When triggered (including when the user passes just a vacancy URL or job description text at the start of a conversation), you immediately treat it as an order to create submission documents and execute the following 8-step pipeline using only your built-in IDE tools. No Python, no terminal.

```
Step 0 → Load candidate library
Step 1 → Locate CV
Step 2 → Get job description
Step 3 → Parse CV → merge into library
Step 4 → Parse job description
Step 5 → Match library vs JD → gap analysis
Step 6 → Interview user for new gaps
Step 7 → Write 3 documents from library
Step 8 → Package submission folder
```

Always read the **candidate-library** skill before touching `data/candidate.json`.
Always read the **writing-guidelines** skill before generating any document.
Always read the **folder-management** skill before creating the submission folder.

---

## Step 0 — Load Candidate Library

Read `data/candidate.json` using `view_file`.

- If `data/candidate.json` does not exist, start with an empty library structure (see the
  **candidate-library** skill for the schema).
- `data/candidate.json` is the single source of truth defining career experience, enrichments, personal contact details, work authorizations, and regional routing rules. Checking and verifying the candidate library file is essential for high relevance and quality across all generated submission documents.
- If the user asks to initialise the library or ingest all CVs, scan `data/source_cvs/`, parse each CV file, and merge them sequentially into `data/candidate.json`.
- Hold the library in context for the entire pipeline run.
- Print: `📚 Library loaded — [N] experiences, [N] enrichments`

---

## Step 1 — Locate CV

Check if the user specified a CV file. If not:

1. List files in `data/source_cvs/` using `list_dir`.
2. Pick the most relevant or recently modified file.
3. Confirm the selected file with the user.
4. Read it directly according to format:
   - **`.md` / `.txt` / `.pdf`:** Read with `view_file` (built-in binary viewer handles PDF pages and text/OCR).
   - **`.docx` / `.doc`:** Read automatically using `run_command`:
     - *macOS:* `textutil -convert txt "<file_path>" -stdout`
     - *Linux / Cross-platform (`.docx`):* Zero-dependency Python one-liner (`python3 -c "import zipfile, xml.etree.ElementTree as ET, sys; tree=ET.fromstring(zipfile.ZipFile(sys.argv[1]).read('word/document.xml')); print('\n'.join(''.join(p.itertext()) for p in tree.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}p')))" "<file_path>"`), `pandoc`, or `docx2txt`.
     - *Linux (`.doc`):* `catdoc`, `antiword`, or `soffice --headless --convert-to txt:Text "<file_path>" --stdout`.
     - *Windows:* PowerShell extraction or Word COM automation.

**Supported formats:** `.md`, `.txt`, `.pdf`, `.doc`, `.docx` (all read automatically without requiring copy-paste).

---

## Step 2 — Get Job Description

If the user did not supply a JD, ask:
> "Please provide the job description — paste the text or give me a URL."

- **URL** → use `read_url_content` to fetch the page text immediately. **Never ask for permission** to access the web page if the URL was provided or implicitly specified in the user's prompt.
- **File** → use `view_file` (if user points to a specific file).
- **Pasted text** → use as-is.

Hold the raw JD text in context. It will be saved directly into the submission folder as `jd_source.md` during Step 8.

---

## Step 3 — Parse CV & Merge into Library

Read the **cv-parsing** skill for extraction rules.

1. Extract the full CVData structure from the raw CV text.
2. Read the **candidate-library** skill for merge rules.
3. Merge CVData into the current library:
   - Identity fields: last-seen non-null value wins.
   - Experience: deduplicate by `(company, title, dates_from)` — merge bullets.
   - Skills/tools/languages: union, case-insensitive dedup.
   - Education: deduplicate by `(institution, degree)`.
   - Track source filename in `library.source_cvs`.
4. Save updated library to `data/candidate.json` with `write_to_file`.
5. Print: `✓ CV merged — [N] roles, [N] skills in library`

---

## Step 4 — Parse Job Description

Read the **jd-analysis** skill for extraction rules.

Extract a structured JD object:
```
title, company, location, seniority, employment_type,
required_skills[], nice_to_have_skills[],
responsibilities[], culture_notes[], keywords[]
```

Hold this structured object in context — the original JD text is saved to `jd_source.md` in the submission folder during Step 8.

---

## Step 5 — Match Library vs JD

Read the **matching** skill for the scoring algorithm and competence gap schema.

1. Match the **full library** (not just one CV parse) against the JD:
   - Experience, skills, tools, awards, projects
   - **Also check `library.enrichments`** — prior Q&A answers count as evidence.
   - **Enforce Zero Fabrication & Domain Boundaries**: Differentiate between IT and Telecom (or other adjacent domains). Do not assume familiarity with any unverified technologies, platforms, tools, or formal frameworks (e.g. Jira Service Management, ServiceNow, ITIL, TOGAF, ISO, COBIT, eTOM) unless explicitly verified.
2. Produce a MatchReport:
   - Calculate match score according to formula.
   - For all items (required skills, nice-to-have, responsibilities), track `gap` nuances.
   - **Automatically generate structured `competence_gaps`** for every non-match (`missing`), partial match (`partial`), or weak responsibility with mitigation strategies.
   - **Extract `strong_points` and `weak_points`**: Explicitly contrast key competitive advantages against candid vulnerability areas.
   - **Generate `interview_preparation_recommendations`**: Provide actionable topics, industry concepts, tools, or frameworks to read/improve before interviews.
3. **Filter gap questions:** for each gap topic (including missing technologies, platforms, formal frameworks, or unverified domain experience), check if `library.enrichments` already contains an answer for that topic. If yes → skip that question.
4. Print the match score, skills coverage table, competence gaps table, strong vs weak points, and interview preparation recommendations.
5. Save `match_report.json` to the session — you will write it to the submission
   folder in Step 8.

---

## Step 6 — Interview User for New Gaps

If there are pre-answered gaps (from library):

> "📚 Your library already covers these topics from previous applications:"
> [list topic + short answer for each]

For each **new** gap question (missing technologies, tools, platforms, formal frameworks, or domain experience):

1. Show the gap topic and ask clearly how the user wants to address or fill that gap (e.g. whether they have direct unrecorded exposure, or how they prefer to frame verified transferable experience).
2. Use `ask_question` if a multiple-choice answer helps; otherwise ask free-form.
3. If the answer seems vague, ask one gentle follow-up:
   > "Can you add a specific example or metric to strengthen that answer?"
4. Accept the final answer (max 2 follow-ups per question).
5. Append each answered enrichment to the library immediately.
6. Save library to `data/candidate.json` after all questions.

Print: `✓ [N] new answers saved to library`

---

## Step 7 — Generate Documents

Read the **writing-guidelines** skill in full before generating any document.

Generate three documents in order. For each, use the **full library** as context
(not just the CV parse from this session). Include:
- All `library.experience` entries (ordered by recency)
- All `library.enrichments` (especially those relevant to this JD)
- **Strict Zero Fabrication & Domain Integrity**: Never invent, assume, or fabricate any technologies, tools, platforms, frameworks, standards, certifications, or metrics that are not explicitly present in `data/candidate.json` or answered in Q&A enrichments. Keep IT and Telecom domains distinct.
- **Relevant CVs in `data/source_cvs/` (if gaps exist)**: If `data/candidate.json` has gaps matching the JD, check `data/source_cvs/` for role-specific or domain-aligned CVs (e.g., Data Analyst, Solutions Manager, Software Engineer CVs) to adopt their specific phrasing, bullet structures, and emphasized achievements to address those gaps without adding fictitious tools.
- The MatchReport's `strong_points`, `weak_points`, `competence_gaps`, and `suggested_emphasis`
- The JD's `keywords` (mirror exact phrasing ONLY where truthful and verified)

### 7a — Tailored CV (`tailored_cv.md`)
Full CV, restructured and reworded for this specific role.
Structure: Header → Summary → Key Skills → Experience → Education → Certs → Projects.

### 7b — One-Page Résumé (`resume_1page.md`)
≤600 words. Top 3 experiences, top 8 skills, degree only (no dates).

### 7c — Cover Letter (`cover_letter.md`)
250–400 words. Opening → Proof & QTM Alignment → (Gap) → Closing.
- Infer tone from `culture_notes`.
- **Qualification-Tool-Metric (QTM) Mapping**: Explicitly relate previous qualifications, specific tools/technologies used, and quantified metrics achieved to the requirements in the job description.

### 7d — Compensation & Salary Benchmark Report (`compensation_benchmark.md`)
Conduct market salary benchmarking based on the position, seniority, location, and employer profile:
- **Base Gross Salary Range (Monthly & Annual)**: Market baseline, realistic target, and upper band.
- **Net Salary Estimations**: Net take-home calculations reflecting local tax regulations.
- **Variable Pay & Benefits**: Bonus structures (10–20%+), cafeteria/fringe allowances, and corporate perks listed in the JD.
- **Total Compensation (OTE)**: Comprehensive package scenarios (Moderate, Target, Upper Range).
- **Negotiation Recommendations**: Target anchor phrasing and experience-backed justifications for HR/recruiter screening calls.

**Contact details & Regional Routing (mandatory):**
- Consult `data/candidate.json` for name, email, links, and contact routing rules (`contact_routing_rules`, `work_authorization`).
- Select the phone number, location, and work authorization notes matching the position's target geography as specified in `data/candidate.json`.

---

## Step 8 — Package Submission Folder

Read the **folder-management** skill for naming conventions.

Create folder: `data/submissions/<company>_<role>_<YYYYMMDD>/`

Write these files:
| File | Content |
|---|---|
| `tailored_cv.md` | From Step 7a |
| `resume_1page.md` | From Step 7b |
| `cover_letter.md` | From Step 7c |
| `compensation_benchmark.md` | From Step 7d (Salary benchmarks, total package OTE, negotiation strategy) |
| `match_report.json` | The MatchReport from Step 5 as JSON (including `competence_gaps`, `weak_points`, `interview_preparation_recommendations`) |
| `jd_source.md` | Original JD text |
| `README.md` | Checklist (see template below) |

### README.md template

```markdown
# Application: [Job Title] @ [Company]
**Date:** YYYY-MM-DD  **Match Score:** XX/100

## Submission Checklist
- [ ] Review `tailored_cv.md` — check all facts are accurate
- [ ] Review `cover_letter.md` — personalise opening line
- [ ] Review `resume_1page.md` — confirm it fits one page
- [ ] Review `compensation_benchmark.md` — benchmark salary expectations & negotiation targets
- [ ] Upload to ATS / email as instructed in the JD
- [ ] Log application in your tracker

## Match Summary
**Strong points:** [from report]
**Weak points (Hiring Manager Perspective):** [from report]
**Competence Gaps & Non-Matches Highlighted:** [list from competence_gaps]
**Interview Preparation & Reading Recommendations:** [from report]
**Gaps addressed:** [topics answered in interview]
**Suggested emphasis:** [from report]

## Library Enrichments Used
[List enrichments included, with topics]
```

After packaging, print a summary:
> ✅ Submission saved to `data/submissions/<folder>/`
> 📚 Library updated: [N] total enrichments

---

## Error Handling

| Situation | Action |
|---|---|
| No CV in `source_cvs/` | Ask user to paste CV text |
| JD URL unreachable | Ask user to paste JD text |
| User skips a gap question | Store empty answer, continue |
| Library file corrupted | Warn user, start fresh library |
| Ambiguous company/role name | Ask user to confirm slug |
