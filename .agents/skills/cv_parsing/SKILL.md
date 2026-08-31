---
name: cv-parsing
description: >-
  Rules and techniques for extracting structured data from CV documents
  (Markdown, plain text, PDF, DOCX). Use when reading or parsing a candidate's CV.
---

# CV Parsing Skill

## Goal

Extract a complete, structured representation of a CV so it can be merged into
the candidate library and matched against job descriptions.

## Reading the CV

Read the CV file directly from `library/source_cvs/` based on its format:

- **Markdown (`.md`) / Plain text (`.txt`):** Use `view_file`.
- **PDF (`.pdf`):** Use `view_file` directly (built-in binary viewer parses PDF pages and OCR/text).
- **Word (`.docx`):**
  - **macOS:** `run_command` with `textutil -convert txt "<path>" -stdout`
  - **Linux / Cross-Platform (Universal Python stdlib, zero extra dependencies):**
    ```bash
    python3 -c "import zipfile, xml.etree.ElementTree as ET, sys; tree=ET.fromstring(zipfile.ZipFile(sys.argv[1]).read('word/document.xml')); print('\n'.join(''.join(p.itertext()) for p in tree.iter('{http://schemas.openxmlformats.org/wordprocessingml/2006/main}p')))" "<file_path>"
    ```
  - **Linux / Unix CLI tools (if available):** `pandoc -f docx -t plain "<file_path>"`, `docx2txt "<file_path>"`, or `unzip -p "<file_path>" word/document.xml | sed -e 's/<[^>]*>/ /g'`
  - **Windows (PowerShell):**
    ```powershell
    powershell -Command "[System.IO.Compression.ZipFile]::OpenRead('<file_path>').Entries | Where-Object { $_.FullName -eq 'word/document.xml' } | ForEach-Object { (New-Object System.IO.StreamReader($_.Open())).ReadToEnd() -replace '<[^>]+>', ' ' }"
    ```
- **Legacy Word (`.doc`):**
  - **macOS:** `run_command` with `textutil -convert txt "<path>" -stdout`
  - **Linux / Unix:** `catdoc "<file_path>"` or `antiword "<file_path>"` or `soffice --headless --convert-to txt:Text "<file_path>" --stdout`
  - **Windows (PowerShell via Word COM):**
    ```powershell
    powershell -Command "$w = New-Object -ComObject Word.Application; $w.Visible = $false; $d = $w.Documents.Open((Resolve-Path '<file_path>').Path); Write-Output $d.Content.Text; $d.Close(); $w.Quit()"
    ```

No manual copy-pasting is required from the user.

---

## Always Extract

| Section | Fields |
|---|---|
| **Identity** | Full name, email, phone, LinkedIn URL, GitHub URL, location |
| **Summary** | Professional summary or objective paragraph (if present) |
| **Experience** | For each role: `company`, `title`, `dates_from`, `dates_to`, `location`, `bullets[]` |
| **Education** | `institution`, `degree`, `field`, `dates_from`, `dates_to`, `grade` |
| **Skills** | Split into: `technical[]`, `soft[]`, `languages[]`, `tools[]` |
| **Certifications** | `name`, `issuer`, `date` |
| **Projects** | `name`, `description`, `tech_stack[]`, `url` |
| **Awards** | List of strings |

---

## Normalisation Rules

1. **Dates** — normalise to `YYYY-MM`. Use `present` for current roles. Use `unknown` if missing — never guess.
2. **Skills** — deduplicate within each category; keep original casing.
3. **Bullets** — preserve verbatim. Do NOT rephrase, summarise, or omit.
4. **Empty sections** — omit rather than including null/empty values.
5. **Ambiguous headings** — infer from content (e.g. "Background" → experience or education based on content).
6. **Duplicate skills** — if the same skill appears as e.g. "JS" and "JavaScript", keep the more explicit form.

---

## Output Structure

Produce this structure in your reasoning (JSON-like, no code needed):

```
CVData:
  name: string
  email: string | null
  phone: string | null
  linkedin: string | null
  github: string | null
  location: string | null
  summary: string | null
  experience:
    - company, title, dates_from, dates_to, location, bullets[]
  education:
    - institution, degree, field, dates_from, dates_to, grade
  technical_skills: []
  soft_skills: []
  languages: []
  tools: []
  certifications: [{name, issuer, date}]
  projects: [{name, description, tech_stack[], url}]
  awards: []
```

---

## After Parsing

Immediately follow the **candidate-library** skill to merge this CVData into
`library/candidate.json`.

Do not skip the merge step — the library is the source of truth for all
downstream matching and writing.
