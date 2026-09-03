---
name: matching
description: >-
  How to compare the candidate library against a parsed job description, produce
  a match score, identify gaps, and generate targeted interview questions.
  Use after CV is merged into library and JD is parsed.
---

# Matching Skill

## Goal

Produce a MatchReport that quantifies fit, surfaces genuine non-matches and competence/qualification gaps, and drives the interviewer Q&A — using the **full candidate library** as evidence, not just the current CV parse.

---

## Input

- `CandidateLibrary` — the full library loaded from `data/bio.json`
  (includes all experience, skills, AND previous enrichments / Q&A answers)
- `JDData` — structured job description from Step 4 of the pipeline

---

## Matching Steps

### Step 1 — Skill Coverage & Non-Match Identification

For each skill, tool, technology, platform, framework, or standard in `JDData.required_skills`:
- Search across `library.technical_skills`, `library.tools`, `library.certifications`, and all `experience[].bullets`
- **Also search `library.enrichments[].answer`** — prior Q&A answers count as evidence.
- **Strict Verification (Zero Fabrication & Domain Boundaries)**:
  - Differentiate distinct industry domains (e.g. Telecom vs IT/Enterprise IT).
  - Treat all specific technologies, tools, platforms, and formal frameworks strictly: if not explicitly evidenced in the candidate library (`tools`, `technical_skills`, `experience`), mark as `missing` or `partial`. NEVER assume or fabricate tool experience (e.g. do not assume Jira Service Management or ServiceNow from generic ticketing; do not assume specific BI tools, cloud platforms, or standard frameworks).
- Mark as:
  - **covered** — directly evidenced (`gap: null`)
  - **partial** — related or adjacent skill/tool found, but lacks exact platform, depth, or specific domain (`gap: explanation of nuance/shortfall`)
  - **missing** — no evidence found (`gap: explanation of missing requirement`)

Repeat for `nice_to_have_skills`.

### Step 2 — Responsibility Relevance

For each responsibility in `JDData.responsibilities`:
- Find the best matching evidence from `library.experience[].bullets` or enrichments.
- Score: **strong** / **moderate** / **weak** / **none**
- For **moderate**, **weak**, or **none**, articulate the specific domain, tooling, or procedural gap in `gap`. Keep domain boundaries clear (do not score Telecom experience as strong for pure Enterprise IT service desk management unless verified).

### Step 3 — Seniority Alignment

- Infer candidate seniority from years of experience and titles in the library.
- Compare to `JDData.seniority`.
- Flag if more than one level apart.

### Step 4 — Keyword Density

- Count `JDData.keywords` present anywhere in the library (skills, bullets, enrichments).
- `keyword_coverage_pct = matching / total`

### Step 5 — Automated Competence & Qualification Gaps Extraction (Mandatory)

For every identified non-match (status `missing`), partial match (status `partial`), or weak responsibility (relevance `moderate`/`weak`/`none`), automatically generate a structured entry in `competence_gaps`:
- **`competence`**: Clear name of the required skill, qualification, tool, technology, domain area, or formal framework/standard.
- **`gap_type`**: Classification (`tooling_platform` | `domain_knowledge` | `formal_education` | `methodology_process` | `formal_framework`).
- **`jd_requirement`**: Exact phrase or requirement from the JD.
- **`candidate_status`**: `missing` | `partial`.
- **`candidate_reality`**: Accurate, grounded summary of what the candidate actually has in the library.
- **`gap_analysis`**: Clear, objective analysis of the shortfall, missing technology, domain distinction (e.g. Telecom vs IT), or unverified framework.
- **`mitigation_strategy`**: Verified transferable skills, analogous tooling, or adjacent experience that bridges the gap without fabricating claims.
- **`impact_level`**: `high` | `medium` | `low`.

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

*(Note: In calculations, `covered` = 1.0, `partial` = 0.5, `missing` = 0.0).*

---

## Gap Question Generation (Interview How to Fill the Gap)

For each **missing** required technology, tool, platform, formal framework/standard, or **weak/none** responsibility:

1. Generate ONE focused question per gap asking the user explicitly how they want to fill, bridge, or address the gap (e.g., whether they have unlisted hands-on exposure, adjacent tools, or specific transferable examples).
2. Questions must be clear and direct, for example:
   - "The JD requires [Technology/Framework/Competency]. Do you have hands-on experience with it, or how would you prefer to frame your transferable background from [Verified Tool/Experience] to address this requirement?"
   - "Can you describe your direct exposure to [Technology/Standard], or should we highlight [Alternative Experience] instead?"
3. NEVER assume the answer or fabricate experience. Always ask the candidate directly.
4. Do NOT ask about things already evidenced in the CV or enrichments.

### Library pre-filter (mandatory)

Before finalising gap questions:
- For each question topic, check `library.enrichments` for a matching `topic` (case-insensitive).
- If a match exists with a non-empty answer → **remove that question from the list**.
- Collect removed questions as "pre-answered" — display these to the user as informational.

---

## MatchReport Output Schema

Produce this structure and save as `match_report.json` in the submission package:

```json
{
  "job_title": "string",
  "company": "string",
  "location": "string",
  "match_score": 0.0,
  "required_skill_matches": [
    {
      "skill": "string",
      "status": "covered | partial | missing",
      "evidence": "string (quote or reference from library)",
      "gap": "string | null (explanation of shortfall if partial/missing)"
    }
  ],
  "nice_to_have_matches": [
    {
      "skill": "string",
      "status": "covered | partial | missing",
      "evidence": "string",
      "gap": "string | null"
    }
  ],
  "responsibility_matches": [
    {
      "responsibility": "string",
      "relevance": "strong | moderate | weak | none",
      "best_evidence": "string",
      "gap": "string | null"
    }
  ],
  "competence_gaps": [
    {
      "competence": "string",
      "gap_type": "tooling_platform | domain_knowledge | formal_education | methodology_process",
      "jd_requirement": "string",
      "candidate_status": "missing | partial",
      "candidate_reality": "string",
      "gap_analysis": "string",
      "mitigation_strategy": "string",
      "impact_level": "high | medium | low"
    }
  ],
  "keyword_coverage_pct": 0.0,
  "seniority_aligned": true,
  "strong_points": [
    "3–5 headline strengths, competitive advantages, and verified achievements for this specific role"
  ],
  "weak_points": [
    "3–5 candid vulnerability areas, missing specialized tools, or perceived qualification gaps from the hiring manager's perspective"
  ],
  "interview_preparation_recommendations": [
    {
      "topic": "string (specific subject, standard, tool, or methodology to study)",
      "recommendation": "string (what to read, research, or practice before the interview)",
      "key_concepts_to_master": ["string"]
    }
  ],
  "gap_questions": [
    {
      "topic": "string",
      "question": "string"
    }
  ],
  "pre_answered_enrichments": [
    {
      "topic": "string",
      "answer": "string"
    }
  ],
  "suggested_emphasis": ["string"]
}
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
1. Match score with colour interpretation.
2. A required skills & coverage table (`skill` | `status` | `evidence` | `gap`).
3. **Competence & Qualification Gaps Table**: Highlight all non-matches/partial matches with their mitigation strategies.
4. **Strong Points vs Weak Points**: Direct side-by-side or paired comparison of strengths and hiring manager vulnerability flags.
5. **Interview Preparation & Reading Recommendations**: Concrete topics, terminology, and tools to review prior to conversations with the team.
6. Count of new gap questions vs pre-answered from library.

