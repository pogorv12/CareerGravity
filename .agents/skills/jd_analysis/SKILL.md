---
name: jd-analysis
description: >-
  How to extract structured requirements from a job description — from a file,
  URL, or raw text. Use when parsing a job description before matching.
---

# Job Description Analysis Skill

## Goal
Produce a complete, structured `JDData` object from any job description source
so the Matcher agent can compare it against the candidate's CV.

## Input Sources

| Source | How to handle |
|--------|---------------|
| Local file (`.txt`, `.md`, `.pdf`, `.docx`) | Read file content using appropriate parser |
| URL | Fetch immediately using `read_url_content` without asking for permission if provided or implicitly specified in the request |
| Raw text (pasted) | Process directly |

### URL Scraping & Autonomous Access Strategy
- **Autonomous Access (No Permission Prompt)**: Whenever a job URL is provided, linked, or implicitly specified in the request (e.g. "Prepare an application for [URL]", "Apply for [URL]", pasting a job link), **never ask the user for permission** to access or open the web page. Fetch and read the URL immediately using `read_url_content`.
- Extract the largest `<article>` or `<main>` block; fall back to `<body>`.
- Remove boilerplate such as navigation, header/footer, cookie banners, scripts, and styles.
- Strip extra whitespace and retain the core job posting text.
- Only if the URL cannot be reached or fails to load, report the failure and prompt the user to paste the raw text.

## Extraction Rules

### Always extract
- `title` — exact job title
- `company` — employer name
- `location` — city/remote/hybrid status
- `seniority` — junior / mid / senior / lead / staff / principal / director
- `employment_type` — full-time / part-time / contract / freelance
- `required_skills[]` — must-have qualifications (years of experience, specific tools)
- `nice_to_have_skills[]` — preferred but not mandatory
- `responsibilities[]` — key duties, kept as short action phrases
- `culture_notes[]` — clues about team culture, values, ways of working
- `salary_range` — if mentioned
- `application_deadline` — if mentioned
- `keywords[]` — important nouns/phrases likely used by ATS systems

### Classification heuristics
- "Must have", "Required", "Essential", "Minimum" → `required_skills`
- "Nice to have", "Preferred", "Desirable", "Bonus", "Plus" → `nice_to_have_skills`
- Bullet points under "Responsibilities" / "What you'll do" → `responsibilities`
- Bullet points under "About us" / "Our culture" → `culture_notes`

## Output Schema (Pydantic `JDData`)

```python
class JDData(BaseModel):
    title: str
    company: str
    location: str | None
    seniority: str | None
    employment_type: str | None
    required_skills: list[str]
    nice_to_have_skills: list[str]
    responsibilities: list[str]
    culture_notes: list[str]
    keywords: list[str]
    salary_range: str | None
    application_deadline: str | None
    raw_text: str            # Always store original for reference
```

## Quality Checks
- `required_skills` must never be empty — if genuinely absent, extract from responsibilities.
- `keywords` should include the job title, company product/domain, and top 10 recurring nouns.
- Avoid duplicating items between `required_skills` and `nice_to_have_skills`.
