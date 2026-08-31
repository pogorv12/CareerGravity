---
name: writing-guidelines
description: >-
  Style, tone, structure, and ATS-optimisation rules for generating tailored
  CVs, one-page résumés, and cover letters. Use when the Writer agent produces documents.
---

# Writing Guidelines Skill

## General Principles
- **Grounding & Authenticity (No Fabrication)** — NEVER invent or fabricate unspecified experience, tools, technologies, responsibilities, or metrics to match a JD. Instead, describe how the candidate's *actual, specified* experience, achievements, and transferable skills fulfill the requirements.
- **Specificity over vagueness** — "Reduced API latency by 40%" beats "Improved performance".
- **Action verbs** — Start every bullet with a strong past-tense verb: Led, Built, Designed, Reduced, Launched, Mentored, Automated…
- **Quantify wherever possible** — numbers, percentages, team sizes, time saved.
- **Mirror the JD language truthfully** — use relevant phrasing from `JDData.keywords` where it honestly reflects the candidate's work.
- **No first-person pronouns** — Never "I" or "my" in CVs/résumés; write in third-person implied (e.g. "Designed…" not "I designed…").
- **ATS-safe formatting** — no tables, columns, or graphics in the Markdown that would break text extraction.
- **Personal profile & Regional contact routing** — Read `library/candidate.json` for personal contact information and apply the defined regional contact routing rules (`contact_routing_rules`, `work_authorization`) in respect of the job location.
- **Consult relevant source CVs** — Check `library/source_cvs/` for role-aligned CVs (e.g., Data Analyst, Solutions Manager, etc.) to borrow tailored framing, strong phrasing, and relevant emphasis.

---

## Grounding & Experience Translation (Authenticity vs Framing)

- **Strict Grounding (What NOT to do):**
  - Never fabricate unverified claims, tools, projects, or metrics just to fit the JD's requirements.
  - If a skill or experience is absent from the candidate library and source CVs, do not pretend the candidate has done it.

- **Experience Translation & Framing (What TO do):**
  - **Bridge to JD Competencies:** Deconstruct the core challenges, business problems, and technical competencies the JD requires (e.g. data quality assurance, large-scale ETL, stakeholder alignment, system integration, workflow automation).
  - **Demonstrate Transferability:** Articulate how the candidate's *existing, verified* projects and achievements prove capability in those core competencies.
  - **Highlight Parallels & Equivalents:** Connect proven experience in adjacent tools or methodologies to the target stack (e.g., show how expertise in complex SQL, Python, and data pipeline design equips the candidate to excel in MDM or BI environments).
  - **Contextualize Real Impact:** Rephrase existing bullet points to accentuate the aspects of real work that matter most to the target hiring manager without distorting the truth.
  - **Address Gaps Honestly:** In cover letters and summaries, frame real experience as a solid foundation that enables rapid adaptation and domain mastery.

---

## Tailored CV (`tailored_cv.md`)

### Structure (in order)
1. **Header** — Name, email, phone, LinkedIn, GitHub, location
2. **Professional Summary** — 3–4 sentences tailored to the specific role; include 2–3 keywords from JD
3. **Key Skills** — Flat list of technical skills, tools, languages; ordered by relevance to JD
4. **Work Experience** — Reverse chronological; for each role:
   - Company | Title | Dates | Location
   - 3–6 bullet points; reorder/rephrase to emphasise JD-relevant achievements
5. **Education** — Reverse chronological
6. **Certifications** — (if any)
7. **Projects** — (if relevant to role)
8. **Languages** — (if relevant)

### Length
- No hard limit; include all relevant experience.
- Remove or condense roles older than 10 years unless highly relevant.

---

## One-Page Résumé (`resume_1page.md`)

### Rules
- Strict one-page equivalent — aim for ≤600 words.
- Keep only the **top 3 most relevant roles**.
- Summary → 2 sentences maximum.
- Skills → top 8 most relevant only.
- Education → degree name and institution only.
- Drop certifications and projects unless they directly address a JD gap.

---

## Cover Letter (`cover_letter.md`)

### Core Objective
A compelling narrative that explicitly bridges the candidate's past experience with the target job description by linking **Qualifications**, **Tools**, and **Metrics** to the JD's core requirements.

### Qualification-Tool-Metric (QTM) Alignment Principle
Every cover letter MUST explicitly articulate the direct relation between the candidate's previous experience and the JD requirements across three key dimensions:
1. **Qualifications & Competencies:** Map verified domain expertise, responsibilities, and leadership experience directly to the required qualifications and core duties in the JD.
2. **Tools & Technologies:** Explicitly mention specific tools, platforms, languages, and technical stacks used in prior roles and correlate them directly to the tech stack required in the JD (highlighting direct matches and transferable equivalents).
3. **Metrics & Measurable Impact:** Ground the tool and qualification alignments with concrete, quantified outcomes (e.g., % efficiency gain, scale handled, latency reductions, cost/time saved, revenue/data accuracy) achieved in past experience to demonstrate proven execution capability.

### Structure
```
[Date]

[Hiring Manager name — or "Hiring Team" if unknown]
[Company name]

Dear [name / Hiring Team],

PARAGRAPH 1 — Hook & Intent (2–3 sentences)
  - State the role you're applying for.
  - Lead with your most compelling strength/qualification for THIS role.
  - Show genuine enthusiasm for the company/product.

PARAGRAPH 2 — Proof of Fit & QTM Alignment (4–6 sentences)
  - Pick the 2–3 strongest matches from MatchReport.strong_points.
  - Explicitly relate prior qualifications and verified responsibilities to the JD's primary requirements.
  - Name the specific tools/technologies used in past experience that match or transfer to the JD's required tech stack.
  - Connect tools and qualifications to tangible, quantified metrics (e.g. "leveraged [Tool X] to achieve [Metric Y], directly aligning with your need for [JD Requirement Z]").

PARAGRAPH 3 — Addressing Gaps & Transferable Value (optional / conditional, 2–3 sentences)
  - If match_score < 70 or when bridging adjacent technologies, explain how proven qualifications and tool mastery enable rapid adaptation, supported by past metrics.

PARAGRAPH 4 — Call to Action (2 sentences)
  - Express enthusiasm for next steps.
  - Thank the reader.

Sincerely,
[Name]
[Email] | [Phone] | [LinkedIn]
```

### Tone
- Confident, professional, warm — never desperate.
- Match the company's culture tone (infer from `JDData.culture_notes`):
  - Startup → more casual, energetic.
  - Enterprise → more formal, structured.
  - Creative agency → personality-driven.

### Length
- 250–400 words. Never exceed one A4 page.

---

## Markdown Formatting Conventions

```markdown
# Name
**Email** · **Phone** · [LinkedIn](url) · [GitHub](url) · Location

---

## Professional Summary
...

## Key Skills
`Python` `FastAPI` `PostgreSQL` `Docker` `Kubernetes` `AWS`

## Work Experience

### Senior Software Engineer — ACME Corp
*Jan 2022 – Present · Remote*

- Led migration of monolith to microservices, reducing deploy time by 60%.
- Mentored 4 junior engineers through weekly 1-on-1 code reviews.
```
