---
name: matching
description: >-
  How to compare the candidate library against a parsed job description, produce
  a match score, identify gaps, and generate targeted interview questions.
  Use after CV is merged into library and JD is parsed.
---

# Matching Skill

## Goal

Produce a MatchReport that quantifies fit, surfaces genuine gaps, and drives
the interviewer Q&A — using the **full candidate library** as evidence, not just
the current CV parse.

---

## Input

- `CandidateLibrary` — the full library loaded from `data/candidate_library.json`
  (includes all experience, skills, AND previous enrichments / Q&A answers)
- `JDData` — structured job description from Step 4 of the pipeline

---

## Matching Steps

### Step 1 — Skill Coverage

For each skill in `JDData.required_skills`:
- Search across `library.technical_skills`, `library.tools`, and all `experience[].bullets`
- **Also search `library.enrichments[].answer`** — prior Q&A answers count as evidence.
- Mark as:
  - **covered** — directly evidenced
  - **partial** — related or adjacent skill found
  - **missing** — no evidence found

Repeat for `nice_to_have_skills`.

### Step 2 — Responsibility Relevance

For each responsibility in `JDData.responsibilities`:
- Find the best matching evidence from `library.experience[].bullets` or enrichments.
- Score: **strong** / **moderate** / **weak** / **none**

### Step 3 — Seniority Alignment

- Infer candidate seniority from years of experience and titles in the library.
- Compare to `JDData.seniority`.
- Flag if more than one level apart.

### Step 4 — Keyword Density

- Count `JDData.keywords` present anywhere in the library (skills, bullets, enrichments).
- `keyword_coverage_pct = matching / total`

---

## Scoring Formula

```
match_score (0–100) =
    0.45 × required_skill_coverage_pct
  + 0.25 × responsibility_relevance_score   (strong=1, moderate=0.6, weak=0.2, none=0)
  + 0.15 × keyword_coverage_pct
  + 0.10 × nice_to_have_coverage_pct
  + 0.05 × seniority_alignment_bonus        (1.0 aligned, 0.5 ±1 level, 0 otherwise)
```

---

## Gap Question Generation

For each **missing** required skill or **weak/none** responsibility:

1. Generate ONE focused question per gap.
2. Questions must start with one of:
   - "Can you describe..."
   - "Have you worked with..."
   - "Tell me about..."
3. Do NOT ask about things already evidenced in the CV or enrichments.

### Library pre-filter (mandatory)

Before finalising gap questions:
- For each question topic, check `library.enrichments` for a matching `topic` (case-insensitive).
- If a match exists with a non-empty answer → **remove that question from the list**.
- Collect removed questions as "pre-answered" — display these to the user as informational.

---

## MatchReport Output

Produce this structure and hold in context (JSON):

```
MatchReport:
  match_score: float (0–100)
  required_skill_matches:
    - skill, status (covered|partial|missing), evidence (quote from library)
  nice_to_have_matches:
    - skill, status, evidence
  responsibility_matches:
    - responsibility, relevance (strong|moderate|weak|none), best_evidence
  keyword_coverage_pct: float
  seniority_aligned: bool
  strong_points: [3–5 headline strengths for this specific role]
  gap_questions: [topic, question]   ← already filtered against library
  suggested_emphasis: [existing library items to highlight more prominently]
```

---

## Score Interpretation

| Score | Interpretation |
|---|---|
| 80–100 | Strong match — focus on keywords & tailoring tone |
| 60–79 | Good fit — address 2–3 key gaps in cover letter |
| 40–59 | Moderate — significant gaps; needs strong narrative |
| < 40 | Weak match — consider whether to apply |

---

## Display to User

After matching, show:
1. Match score with colour interpretation
2. A required skills table (skill | covered/partial/missing | evidence snippet)
3. Count of new gap questions vs pre-answered from library
