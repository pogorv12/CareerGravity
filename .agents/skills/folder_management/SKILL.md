---
name: folder-management
description: >-
  Naming conventions and folder structure for CareerGravity submission packages.
  Use when creating or saving submission folders.
---

# Folder Management Skill

## Directory Layout

```
library/
├── candidate.json            # Persistent career knowledge base, profile & routing rules
└── source_cvs/               # Drop raw CVs here; one file per format
    ├── john_doe_cv.pdf
    ├── john_doe_cv.docx
    └── john_doe_cv.md

submissions/                  # Generated output; one folder per application
└── <company>_<role>_YYYYMMDD/
    ├── README.md         # Application checklist & summary
    ├── tailored_cv.md
    ├── resume_1page.md
    ├── cover_letter.md
    ├── match_report.json # Raw MatchReport for reference
    ├── jd_source.md      # Original job description
    └── session.json      # Gap Q&A answers (for re-generation)
```

## Naming Conventions

### Submission folder names
- Format: `<company>_<role>_YYYYMMDD`
- Use the date the submission package is generated
- Truncate role to 3 significant words if long
- If multiple submissions for the same role on the same day, append `_v2`, `_v3`
- Examples:
  - `google_senior_software_engineer_20260828`
  - `stripe_product_manager_20260828_v2`

### File naming inside submission folders
| File | Name (fixed) |
|------|-------------|
| Application checklist | `README.md` |
| Full tailored CV | `tailored_cv.md` |
| One-page résumé | `resume_1page.md` |
| Cover letter | `cover_letter.md` |
| Match report | `match_report.json` |
| Original job description | `jd_source.md` |
| Session (gap answers) | `session.json` |

## README.md Checklist Template

```markdown
# Application: {role} at {company}
**Date generated:** {date}
**Match score:** {score}/100

## Submission Checklist
- [ ] Review `tailored_cv.md` for accuracy
- [ ] Review `cover_letter.md` — personalise greeting if hiring manager name is known
- [ ] Convert to PDF if required (e.g. `pandoc tailored_cv.md -o tailored_cv.pdf`)
- [ ] Attach correct documents to application form
- [ ] Log application in your tracker

## Key Strengths for This Role
{strong_points as bullet list}

## Gaps Addressed
{gap_questions topics and answers summary}

## Notes
{any additional notes from the user}
```

## Slug Generation (Python utility)

```python
import re
from datetime import date

def make_slug(company: str, role: str) -> str:
    text = f"{company}_{role}"
    text = text.lower().strip()
    text = re.sub(r"[^a-z0-9_]+", "_", text)
    text = re.sub(r"_+", "_", text).strip("_")
    return text[:60]   # Max 60 chars

def make_submission_dir_name(company: str, role: str, version: int = 1) -> str:
    slug = make_slug(company, role)
    today = date.today().strftime("%Y%m%d")
    suffix = f"_v{version}" if version > 1 else ""
    return f"{slug}_{today}{suffix}"
```
