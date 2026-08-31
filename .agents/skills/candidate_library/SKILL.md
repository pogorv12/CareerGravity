---
name: candidate-library
description: >-
  How to read, update, merge, and save the persistent candidate library at
  candidate/candidate_library.json. The library is the single source of truth for
  everything known about the candidate. Use whenever touching the library file.
---

# Candidate Library Skill

## What the Library Is

`candidate/candidate_library.json` is a permanent, growing knowledge base for the
candidate. It accumulates content from every CV version and every gap Q&A session
across all job applications.

**Rule:** The library is NEVER overwritten from scratch — only merged into.

---

## Schema

```jsonc
{
  // ── Identity & Contact Routing ──────────────────────────────────────────
  "name": "string",
  "email": "string | null",
  "linkedin": "string | null",
  "github": "string | null",
  "default_location": "string | null",
  "contact_routing_rules": {
    "default": {
      "region_scope": "string",
      "phone": "string",
      "location": "string"
    },
    // Region-specific overrides (e.g. "ru", "by", "uk", "us", etc.)
    "<region_key>": {
      "region_scope": "string",
      "phone": "string",
      "location": "string"
    }
  },
  "work_authorization": {
    "<region_key>": "string"          // e.g. "eu_hungary": "Eligible to work in Hungary / EU"
  },

  // ── Career content ───────────────────────────────────────────────────────
  "experience": [
    {
      "company": "string",
      "title": "string",
      "dates_from": "YYYY-MM | unknown",
      "dates_to": "YYYY-MM | present | unknown",
      "location": "string | null",
      "bullets": ["string"],
      "enrichment_notes": ["string"]   // notes added from Q&A answers
    }
  ],
  "education": [
    {
      "institution": "string",
      "degree": "string",
      "field": "string | null",
      "dates_from": "YYYY-MM | unknown",
      "dates_to": "YYYY-MM | unknown",
      "grade": "string | null"
    }
  ],
  "technical_skills": ["string"],
  "soft_skills": ["string"],
  "tools": ["string"],
  "languages": ["string"] | {"<language>": "<proficiency>"},
  "certifications": [{"name": "string", "issuer": "string", "date": "string"}],
  "projects": [{"name": "string", "description": "string", "tech_stack": ["string"]}],
  "awards": ["string"],
  "summary_statements": ["string"],   // raw summaries from all CV versions
  // ── Q&A Enrichments ──────────────────────────────────────────────────────
  "enrichments": [
    {
      "topic": "string",              // e.g. "Kubernetes production experience"
      "question": "string",           // the exact gap question asked
      "answer": "string",             // candidate's full answer
      "position_context": "string | null",  // e.g. "CloudScale AI / Senior Backend Engineer"
      "created_at": "ISO-8601 string"
    }
  ],

  // ── Metadata ─────────────────────────────────────────────────────────────
  "created_at": "ISO-8601 string",
  "updated_at": "ISO-8601 string",
  "source_cvs": ["filename.md"]       // CV files that contributed to the library
}
```

## Importance of Candidate Library Information for Document Relevance

`candidate/candidate_library.json` is the **single source of truth** for all document generation and ATS matching.

> [!IMPORTANT]
> **Checking and maintaining complete, verified candidate library information is essential for high document relevance.**
> - All generated submission documents (tailored CV, 1-page résumé, cover letter) and matching scores directly draw from the library.
> - High-quality bullets with verified metrics, accurate tools, and up-to-date regional contact routing rules (`contact_routing_rules`, `work_authorization`) directly ensure the highest relevance, quality, and ATS alignment for every application.

---

## Initialising the Candidate Library from Available CVs

When setting up a new candidate library or ingesting all available CVs:

1. Locate all files in `candidate/source_cvs/` (`.md`, `.txt`, `.pdf`, `.doc`, `.docx`).
2. Load or initialize `candidate/candidate_library.json`.
3. For each CV in `candidate/source_cvs/`:
   - Parse CV data into structured format (see **cv-parsing** skill).
   - Merge the parsed data into the candidate library following the merge rules below.
   - Add the filename to `source_cvs`.
4. Save the merged library to `candidate/candidate_library.json`.
5. Advise the user to inspect and verify `candidate/candidate_library.json` (specifically contact details, location routing, work authorizations, and past experiences) before running applications.

---

## Loading the Library

1. Use `view_file` on `candidate/candidate_library.json`.
2. If the file does not exist, initialise an empty library:

```json
{
  "name": "",
  "email": null,
  "linkedin": null,
  "github": null,
  "default_location": null,
  "contact_routing_rules": {
    "default": {
      "region_scope": "Worldwide",
      "phone": "",
      "location": ""
    }
  },
  "work_authorization": {},
  "experience": [],
  "education": [],
  "technical_skills": [],
  "soft_skills": [],
  "tools": [],
  "languages": [],
  "certifications": [],
  "projects": [],
  "awards": [],
  "summary_statements": [],
  "enrichments": [],
  "created_at": "<current ISO timestamp>",
  "updated_at": "<current ISO timestamp>",
  "source_cvs": []
}
```

---

## Merging a CV into the Library

After parsing a CV (see **cv-parsing** skill), merge each section using these rules:

### Identity fields
- Overwrite library value if the CV value is **non-null** and **non-empty**.
- Never set a field to null if it already has a value.

### `experience`
- **Deduplication key:** `(company.lower(), title.lower(), dates_from)`
- If a matching entry exists:
  - Append any bullets NOT already present (exact string match).
  - Update `location` if the existing entry has none.
- If no match: append as a new entry with `enrichment_notes: []`.

### `education`
- **Deduplication key:** `(institution.lower(), degree.lower())`
- If a match exists: keep the more complete record (prefer non-null fields).
- If no match: append.

### List fields (`technical_skills`, `soft_skills`, `tools`, `languages`, `awards`)
- **Union merge:** add any item not already present.
- Case-insensitive comparison — `"Python"` and `"python"` are the same.
- Preserve original casing of the first-seen version.

### `certifications`
- **Deduplication key:** `(name, issuer)` — case-insensitive.
- If no match: append.

### `projects`
- **Deduplication key:** `name.lower()`
- If no match: append.

### `summary_statements`
- Append the CV's summary if it is not already present (exact match).

### `source_cvs`
- Append the CV filename if not already listed.

---

## Adding Enrichments (Q&A Answers)

After the interviewer collects answers, add each answered question as an enrichment:

**Deduplication rules:**
1. If `(topic.lower(), position_context)` already exists → **skip** (same question, same job).
2. If `topic.lower()` exists for a **different** `position_context` → **add** (different context = new value).
3. If `answer` is empty or whitespace → **skip**.

**Enrichment structure to append:**
```json
{
  "topic": "<topic from gap question>",
  "question": "<full question text>",
  "answer": "<candidate's answer>",
  "position_context": "<Company / Role Title>",
  "created_at": "<current ISO timestamp>"
}
```

---

## Filtering Already-Answered Gaps

Before the interview step, split gap questions into two groups:

1. **Pre-answered** — `topic.lower()` exists in any enrichment with a non-empty answer.
   → Show these to the user as informational (library already knows).
2. **New** — topic not in library.
   → Ask these in the interview.

---

## Saving the Library

1. Update `updated_at` to the current ISO timestamp.
2. Use `write_to_file` with `Overwrite: true` to save the full JSON to
   `candidate/candidate_library.json`.
3. Always pretty-print (2-space indent) for human readability.

**Save after:**
- CV merge (Step 3 of pipeline)
- Each batch of enrichments added (Step 6 of pipeline)
- End of the full pipeline run (Step 8)

---

## Do Not

- Never delete existing enrichments.
- Never replace the entire library with a single CV's content.
- Never store duplicate enrichments for the same topic+position.
- Never omit the `updated_at` timestamp on save.

