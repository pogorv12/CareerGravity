# CareerGravity

> A skill-driven job application system that runs entirely inside Antigravity IDE.
> No Python. No terminal. Just chat.

The system reads your CVs, analyses job descriptions, asks targeted questions to
fill gaps, and generates tailored application packages — CV, résumé, and cover
letter — all grounded in a **persistent candidate library** that grows smarter
with every application.

---

## 🚀 Beginner's Quick Start (No Coding or Git Required)

If you've never used **Git**, coding tools, or **Antigravity** before, don't worry! CareerGravity is designed so anyone can use it simply by chatting in plain English.

Here is the complete step-by-step guide to get up and running in minutes:

### 1. Download & Install Antigravity IDE
**Antigravity** is a next-generation AI-powered workspace (like an intelligent editor) with a built-in AI assistant that can read files, write documents, and run workflows for you.
- Download and install the **Antigravity IDE** application for your operating system (macOS / Windows / Linux).
- Launch the application once installed.

### 2. Get the CareerGravity Project Folder
You do not need to use Git or the command line:
- Download the project files as a **ZIP** archive (or copy the `CareerGravity` project folder to your computer, e.g., in your `Documents` or `Desktop` folder).
- If downloaded as a ZIP, unzip / extract it.

### 3. Open the Folder in Antigravity
- In Antigravity IDE, click **File** > **Open Folder...** (or **Open Workspace**).
- Select the `CareerGravity` folder you just extracted.
- You will see the project file list on the left side and an **AI Chat panel** on the right side.

### 4. Put Your Resume / CV in the Source Folder
- Look at the left file explorer panel and expand the `candidate/` folder.
- Drag & drop your existing resume or CV files into `candidate/source_cvs/`.
- You can drop multiple formats: **PDF (`.pdf`)**, **Word (`.docx` / `.doc`)**, **Markdown (`.md`)**, or **Text (`.txt`)**.
- *Tip: If you have different CV versions (e.g., general, technical, management), drop all of them in — the AI will combine and organize your full experience!*

### 5. Start Chatting with the AI Assistant
Click into the **Chat window** on the right side and type your request in normal English:
- **First time setup:** Type:
  > *"Please initialize my candidate library from all source CVs"*
  
  The AI will read your resumes, build your personal career profile in `candidate/candidate_library.json`, and confirm when ready.

- **To apply for a job:** Simply paste the job link or text into the chat:
  > *"I want to apply for this role: [Paste URL or Paste Job Description Text]"*

### 6. Answer Gap Questions & Get Your Tailored Documents
- The AI will analyze the job requirements and compare them against your experience.
- If there are specific skills or project details to clarify, the AI will ask you a few quick multiple-choice or short questions in the chat.
- Once answered, the AI automatically creates your tailored application package inside the `submissions/` folder:
  - 📄 **Tailored CV (`tailored_cv.md`)** — Optimized for ATS filters and the exact role
  - 📄 **1-Page Résumé (`resume_1page.md`)** — Concise, high-impact one-page version
  - ✉️ **Targeted Cover Letter (`cover_letter.md`)** — Compelling story connecting your past achievements to the job's needs
  - 📊 **Match Report (`match_report.json`)** — Match percentage, score breakdown, and strengths

---

## 💬 How to Use & Example Prompts

**Just chat with the Antigravity IDE agent.** Say things like:

```
"Initialize candidate library from all source CVs"
"Apply for CloudScale AI / Senior Backend Engineer — here's the JD: [URL]"
"Process my CV for the BP Planning Analyst role"
"Prepare an application — I'll paste the job description"
"Run the pipeline"
Or simply paste a vacancy URL or job description text directly
```

The agent automatically reads the `pipeline` skill and walks you through the entire workflow.

---

## Initiating Your Candidate Library

Before generating job applications, initialize and verify your persistent career knowledge base:

### Step 1: Drop Your Available CVs
Place all your existing CV versions into `candidate/source_cvs/` (supported: `.md`, `.txt`, `.pdf`, `.doc`, `.docx`). If you have role-specific CVs (e.g., Data Analyst, Solutions Manager, Software Engineer), include all of them so the library captures your full career history, tools, and achievements.

### Step 2: Ingest CVs into the Library
Tell the agent:
> *"Initialise my candidate library from all source CVs in `candidate/source_cvs/`"*

The agent parses every CV file and cleanly merges experience bullets, skills, education, tools, and certifications into `candidate/candidate_library.json`.

### Step 3: Review & Verify Library Information (Crucial!)
> [!IMPORTANT]
> **Checking and maintaining your candidate library information is essential for high document relevance.**
> `candidate/candidate_library.json` is the **single source of truth** for all document generation and ATS matching. The quality, precision, and truthfulness of your tailored CV, one-page résumé, and cover letter directly depend on the library's data.
>
> Open `candidate/candidate_library.json` to verify:
> 1. **Identity & Contact Routing:** Ensure `contact_routing_rules` has correct phone numbers and locations for your target countries/regions.
> 2. **Work Authorization:** Confirm `work_authorization` accurately reflects your legal eligibility for target jurisdictions (e.g., EU, UK, US).
> 3. **Experience Bullets & Metrics:** Check that past roles contain quantified outcomes, verified technologies, and clear responsibilities.

---

## How the Candidate Library Works

`candidate/candidate_library.json` is your personal career knowledge base:

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
candidate/
├── candidate_library.json   ← Persistent career knowledge base, profile & contact routing
└── source_cvs/              ← Drop your CVs here (.md, .txt, .pdf, .doc, or .docx)
    └── my_cv.md

submissions/                 ← Generated application packages
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
1. Locate CV (candidate/source_cvs/ or paste text)
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

Drop your CV into `candidate/source_cvs/`. Supported formats:
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

