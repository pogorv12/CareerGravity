# CareerGravity

> A skill-driven job application system that runs entirely inside Antigravity IDE.
> No Python. No terminal. Just chat.

The system reads your CVs, analyses job descriptions, asks targeted questions to
fill gaps, and generates tailored application packages — CV, résumé, and cover
letter — all grounded in a **persistent candidate library** that grows smarter
with every application.

---

## How to Use

**Just chat with the Antigravity IDE agent.** Say things like:

```
"Apply for CloudScale AI / Senior Backend Engineer — here's the JD: [URL]"
"Process my CV for the BP Planning Analyst role"
"Prepare an application — I'll paste the job description"
"Run the pipeline"
```

The agent reads the `pipeline` skill and walks you through all 8 steps.

---

## How the Candidate Library Works

`data/candidate_library.json` is your personal career knowledge base:

| Event | What happens |
|---|---|
| CV parsed | Experience, skills, education merged in (deduplicated) |
| Gap question answered | Answer stored as enrichment keyed by topic |
| 2nd application | Already-answered gaps shown from library — not re-asked |
| Document generated | Full library is source of truth — all prior answers inform output |

By your 3rd–4th application the library is rich and Q&A gets shorter every time.

---

## Data Structure

```
data/
├── personal_profile.json    ← Contact info & regional routing rules
├── candidate_library.json   ← Persistent career knowledge base
├── source_cvs/              ← Drop your CVs here (.md, .txt, .pdf, .doc, or .docx)
│   └── my_cv.md
└── submissions/             ← Generated application packages
    └── cloudscale_ai_backend_engineer_20260828/
        ├── README.md            ← Checklist + summary
        ├── tailored_cv.md       ← Full CV tailored for this role
        ├── resume_1page.md      ← One-page résumé
        ├── cover_letter.md      ← Personalised cover letter
        ├── match_report.json    ← Match score & gap analysis
        └── jd_source.md         ← Original JD text
```

---

## Skill Architecture

```
pipeline/           ← Master orchestrator (8 steps)
candidate_library/  ← Library schema & merge rules
cv_parsing/         ← CV extraction instructions
jd_analysis/        ← JD structured extraction
matching/           ← Scoring algorithm & gap logic
writing_guidelines/ ← Style, ATS, tone, structure
folder_management/  ← Naming conventions
```

---

## Pipeline Steps

```
0. Load candidate library
1. Locate CV (data/source_cvs/ or paste text)
2. Get JD (file / URL / paste)
3. Parse CV → merge into library → save
4. Parse JD → extract structured data
5. Match library vs JD → score + gap questions
   (skips topics already answered in library)
6. Interview for new gaps → save to library
7. Generate tailored CV, résumé, cover letter
8. Package submission folder
```

---

## CV Formats

Drop your CV into `data/source_cvs/`. Supported formats:
- **Markdown (`.md`)** & **Plain Text (`.txt`)**: Read directly.
- **PDF (`.pdf`)**: Read directly via built-in viewer and text/OCR.
- **Word (`.docx` & `.doc`)**: Extracted automatically across operating systems:
  - **macOS**: Native `textutil` utility.
  - **Linux / Cross-Platform**: Universal Python stdlib extractor (`zipfile`/`xml.etree.ElementTree`, zero dependencies) or CLI tools (`pandoc`, `docx2txt`, `catdoc`, `antiword`, LibreOffice).
  - **Windows**: PowerShell or Word automation.

No manual copy-pasting or format conversion is required.

---

## No Setup Required

No `pip install`. No virtual environment. No API keys to configure.
Open the project in Antigravity IDE and start chatting.

---

## License

Free to use for **personal, non-commercial use** at your own risk and responsibility. See [LICENSE.md](file:///Users/pogorv/Dev/CareerGravity/LICENSE.md) for full terms, conditions, and disclaimers.

