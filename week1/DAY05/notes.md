# Episode 06 — AI Resume Evaluator

## 🎯 Project Goal

Build an AI-powered **Resume Evaluator** that:

1. Takes a Job Description (JD)
2. Converts the JD into structured JSON
3. Reads PDF / DOCX resumes
4. Converts resumes into clean text
5. Converts resume text into structured data
6. Uses an LLM to compare JD + Resume
7. Generates:

   * Match score
   * Matching skills
   * Missing skills
   * Experience requirement status
   * Final verdict
8. Ranks all candidates
9. Displays **Top 2** and **Lowest 2** candidates

### Overall Pipeline

```text
Job Description
       ↓
LLM
       ↓
Structured Job JSON
       ↓
                    Resume Folder
                         ↓
                 PDF / DOCX files
                         ↓
                  Extract clean text
                         ↓
                       LLM
                         ↓
                Structured Resume JSON
                         ↓
             Compare JD + Resume
                         ↓
                    LLM Scoring
                         ↓
              Score + Details/Verdict
                         ↓
                    Sort Candidates
                    ↙             ↘
                TOP 2          WORST 2
```

---

# SECTION 1 — JOB DESCRIPTION → JSON

## 1.1 Why convert JD into JSON?

A Job Description is normally unstructured text.

Example:

```text
Looking for an SDE with Python/C++,
knowledge of data structures,
AWS experience...
```

Instead of sending raw text everywhere, convert it into a predictable structure:

```json
{
  "role": "...",
  "required_skills": [],
  "preferred_skills": [],
  "minimum_experience": null,
  "education_requirements": [],
  "responsibilities": []
}
```

This makes the JD easier for the program and LLM to process.

---

## 1.2 Pydantic Schema for Job Description

```python
class JobD(BaseModel):
    role: str
    required_skills: list[str]
    preferred_skills: list[str]
    minimum_experience: float | None
    education_requirements: list[str]
    responsibilities: list[str]
```

### Important

`BaseModel` comes from Pydantic:

```python
from pydantic import BaseModel
```

Pydantic is being used to define the **expected structure of data**.

---

## 1.3 Generate JSON Schema

```python
jobd_schema = JobD.model_json_schema()
```

This converts the Pydantic model into a JSON-schema-like description that can be shown to the LLM.

---

## 1.4 Prompt Engineering

The LLM is told:

```text
You are an expert HR assistant.

Extract structured information from the JD.

Return ONLY valid JSON matching this schema.
```

Important rules:

```text
Do not return the schema itself.
Do not invent information.
If minimum experience is missing → null.
If a list is missing → [].
```

### Key Idea

**Pydantic defines the structure.**

**Prompt tells the LLM how to fill the structure.**

---

## 1.5 Groq LLM Call

```python
response = client.chat.completions.create(
    model=model,
    messages=messages,
    response_format={"type": "json_object"}
)
```

The important part:

```python
response_format={"type": "json_object"}
```

This asks the model to return JSON rather than normal conversational text.

---

## 1.6 Convert JSON → Pydantic Object

First:

```python
job_data = json.loads(raw_json)
```

Then:

```python
job = JobD(**job_data)
```

### Flow

```text
LLM response
     ↓
JSON string
     ↓
json.loads()
     ↓
Python dictionary
     ↓
JobD(**job_data)
     ↓
Validated Pydantic object
```

---

# SECTION 2 — RESUME → STRUCTURED DATA

## 2.1 Resume Schema

The resume needs a different structure.

### Experience

```python
class Experience(BaseModel):
    company: str | None = None
    role: str | None = None
    duration: str | None = None
    description: str | None = None
    skills_used: list[str] = []
```

### Resume

```python
class Resume(BaseModel):
    name: str | None = None
    email: str | None = None
    phone: str | None = None

    total_experience_years: float | None = None

    skills: list[str] = []
    experiences: list[Experience] = []
    education: list[str] = []
    projects: list[str] = []
    certifications: list[str] = []
```

---

## 2.2 Why `| None`?

Example:

```python
name: str | None = None
```

Means:

```text
name can be a string
OR
name can be None
```

Useful because a resume may not contain every field.

Example:

```json
{
  "name": "Rahul",
  "email": null
}
```

---

## 2.3 Why `list[str]`?

```python
skills: list[str]
```

Means a list containing strings.

Example:

```python
["Python", "C++", "SQL", "AWS"]
```

---

## 2.4 Resume Parsing with LLM

Function:

```python
def parse_resume(resume_text):
```

The clean resume text is sent to the LLM.

The LLM is instructed to:

* Extract information based on meaning
* Handle different section names
* Include internships as experience
* Extract skills from the entire resume
* Never invent information
* Return `null` when a value is unavailable
* Return `[]` when a list is unavailable

### Important Concept

Resume headings can differ:

```text
Experience
Professional Experience
Work History
Employment
Internships
```

The LLM should understand that these can represent the same concept.

---

## 2.5 Resume JSON Conversion

```python
raw_output = response.choices[0].message.content

data = json.loads(raw_output)

resume = Resume(**data)
```

Same pattern as JD:

```text
LLM
 ↓
JSON
 ↓
json.loads()
 ↓
Dictionary
 ↓
Pydantic Model
```

---

# SECTION 3 — READING PDF / DOCX FILES

This section converts actual resume files into **plain text**.

## 3.1 Libraries

### PDF

```python
from pypdf import PdfReader
```

### Word

```python
from docx import Document
```

### Important Installation

Install:

```bash
uv add pypdf python-docx
```

⚠️ Install **`python-docx`**, not the old `docx` package.

---

# 3.2 Reading PDF

```python
def read_pdf(file_path):
    reader = PdfReader(file_path)

    text = ""

    for page in reader.pages:
        page_text = page.extract_text()

        if page_text:
            text += page_text + "\n"

    return text
```

### Logic

```text
PDF
 ↓
PdfReader
 ↓
Pages
 ↓
extract_text()
 ↓
Combine text
 ↓
Return string
```

---

# 3.3 Reading DOCX

```python
def read_docx(file_path):
    document = Document(file_path)

    text = ""

    for paragraph in document.paragraphs:
        if paragraph.text.strip():
            text += paragraph.text + "\n"

    for table in document.tables:
        for row in table.rows:
            for cell in row.cells:
                if cell.text.strip():
                    text += cell.text + "\n"

    return text
```

### Two things are extracted:

#### Paragraphs

```python
document.paragraphs
```

#### Tables

```python
document.tables
```

This is important because resumes may store information inside tables.

---

# 3.4 Automatically Detect File Type

```python
def read_resume(file_path):

    if file_path.suffix.lower() == ".pdf":
        return read_pdf(file_path)

    elif file_path.suffix.lower() == ".docx":
        return read_docx(file_path)

    else:
        return None
```

### Why `.suffix.lower()`?

For:

```text
resume.pdf
resume.PDF
resume.docx
resume.DOCX
```

all can be handled.

---

# SECTION 4 — COMPARISON, SCORE & VERDICT

Now we have:

```text
JobD object
+
Resume object
```

The final step is to compare them.

---

## 4.1 MatchResult Schema

```python
class MatchResult(BaseModel):
    score: float
    details: dict
```

This stores:

```text
score
details
```

---

## 4.2 final_score()

```python
def final_score(job, resume):
```

The function sends both structured objects to the LLM.

### Job

```python
job.model_dump_json(indent=2)
```

### Resume

```python
resume.model_dump_json(indent=2)
```

Then the LLM receives:

```text
JOB DESCRIPTION
       +
CANDIDATE RESUME
       ↓
     LLM
       ↓
MATCH RESULT
```

---

## 4.3 What Does the LLM Evaluate?

The prompt asks for:

### 1. Candidate name

Who is the candidate?

### 2. Matching skills

Skills appearing in both JD and resume.

Example:

```text
Python
SQL
AWS
```

### 3. Missing important skills

Important JD requirements that the candidate lacks.

Example:

```text
AWS
Docker
Kubernetes
```

### 4. Experience requirement

Does the candidate meet the required experience?

```text
YES / NO
```

### 5. Overall match percentage

```text
0 → 100
```

Example:

```text
87%
```

### 6. Final verdict

Short hiring-style conclusion.

Example:

```text
Strong Match
```

---

# 4.4 Parse Match Result

```python
data = json.loads(
    response.choices[0].message.content
)

return MatchResult(**data)
```

Again:

```text
LLM
 ↓
JSON
 ↓
json.loads()
 ↓
Pydantic validation
 ↓
MatchResult
```

---

# SECTION 5 — PROCESSING MULTIPLE RESUMES

Resume directory:

```text
resumes/
    resume1.pdf
    resume2.pdf
    resume3.docx
    resume4.pdf
```

---

## 5.1 Get Resume Folder

```python
resume_folder = Path("resumes")
```

---

## 5.2 Iterate Through Files

```python
for file_path in resume_folder.iterdir():
```

This loops through every file in the folder.

---

## 5.3 Ignore Unsupported Files

```python
if file_path.suffix.lower() not in [".pdf", ".docx"]:
    continue
```

Only:

```text
.pdf
.docx
```

are processed.

---

## 5.4 Complete Processing Pipeline

```python
resume_text = read_resume(file_path)

parsed_resume = parse_resume(resume_text)

result = final_score(job, parsed_resume)
```

### Three major stages

```text
FILE
 ↓
read_resume()
 ↓
CLEAN TEXT
 ↓
parse_resume()
 ↓
STRUCTURED RESUME
 ↓
final_score()
 ↓
MATCH RESULT
```

---

# SECTION 6 — WHY `time.sleep(5)`?

The code contains:

```python
time.sleep(5)
```

between LLM calls.

The project may process many resumes.

For each resume:

```text
Resume → LLM Call
Resume → LLM Call
```

So the sleep introduces a delay between requests.

### Important

It is **not part of resume parsing logic**.

It is simply a delay between API requests.

---

# SECTION 7 — STORE ALL RESULTS

```python
all_results = []
```

For every candidate:

```python
all_results.append({
    "name": parsed_resume.name,
    "score": result.score,
    "details": result.details
})
```

Example:

```python
[
    {
        "name": "Rahul",
        "score": 91,
        "details": {...}
    },
    {
        "name": "Aman",
        "score": 76,
        "details": {...}
    }
]
```

---

# SECTION 8 — RANK CANDIDATES

```python
all_results.sort(
    key=lambda candidate: candidate["score"],
    reverse=True
)
```

### Meaning

Sort candidates using:

```python
candidate["score"]
```

and:

```python
reverse=True
```

means:

```text
Highest → Lowest
```

Example:

```text
Rahul    91
Priya    87
Aman     76
Ravi     64
```

---

# SECTION 9 — TOP 2

```python
top_2 = all_results[:2]
```

Python slicing:

```text
[:2]
```

means:

```text
first 2 elements
```

Result:

```text
TOP 2 CANDIDATES
```

---

# SECTION 10 — WORST 2

```python
worst_2 = all_results[-2:]
```

Python slicing:

```text
[-2:]
```

means:

```text
last 2 elements
```

Because the list is already sorted descending, these are the lowest-scoring candidates.

---

# SECTION 11 — FINAL OUTPUT

### Top candidates

```python
print("TOP 2 CANDIDATES")

for candidate in top_2:
    print(
        candidate["name"],
        "-",
        candidate["score"],
        "%"
    )

    print(candidate["details"])
```

### Lowest candidates

```python
print("LOWEST 2 CANDIDATES")

for candidate in worst_2:
    print(
        candidate["name"],
        "-",
        candidate["score"],
        "%"
    )

    print(candidate["details"])
```

---

# 🧠 COMPLETE CONCEPT IN ONE PAGE

```text
                    JOB DESCRIPTION
                           │
                           ▼
                         LLM
                           │
                           ▼
                    JobD Pydantic
                           │
                           │
                           │
        ┌──────────────────┴──────────────────┐
        │                                     │
        ▼                                     ▼
   resume.pdf                            resume.docx
        │                                     │
        ▼                                     ▼
   PdfReader                             Document()
        │                                     │
        └──────────────────┬──────────────────┘
                           ▼
                     CLEAN TEXT
                           │
                           ▼
                          LLM
                           │
                           ▼
                  Resume Pydantic
                           │
                           ▼
                  ┌────────────────┐
                  │   JOB + RESUME │
                  └───────┬────────┘
                          ▼
                         LLM
                          │
                          ▼
                    MatchResult
                    ┌─────┴─────┐
                    │           │
                  Score       Details
                    │
                    ▼
              Store Results
                    │
                    ▼
                   Sort
                    │
             ┌──────┴──────┐
             ▼             ▼
           TOP 2        WORST 2
```

---

# 🔑 IMPORTANT FUNCTIONS TO REMEMBER

| Function              | Purpose                         |
| --------------------- | ------------------------------- |
| `JobD()`              | JD structure                    |
| `Resume()`            | Resume structure                |
| `Experience()`        | Experience structure            |
| `MatchResult()`       | Final matching result           |
| `model_json_schema()` | Pydantic → JSON schema          |
| `json.loads()`        | JSON string → Python dict       |
| `model_dump_json()`   | Pydantic object → JSON          |
| `read_pdf()`          | Extract text from PDF           |
| `read_docx()`         | Extract text from Word          |
| `read_resume()`       | Detect PDF/DOCX automatically   |
| `parse_resume()`      | Resume text → structured resume |
| `final_score()`       | JD + Resume → score/details     |
| `iterdir()`           | Iterate folder files            |
| `sort()`              | Rank candidates                 |

---

# ⚡ 10-MINUTE REVISION

### Q1. Why Pydantic?

To define and validate structured data.

### Q2. Why `model_json_schema()`?

To generate the schema that can be given to the LLM.

### Q3. Why `json.loads()`?

To convert JSON text into a Python dictionary.

### Q4. Why `model_dump_json()`?

To convert a Pydantic object into JSON for the LLM prompt.

### Q5. How is PDF read?

```python
PdfReader()
→ pages
→ extract_text()
```

### Q6. How is DOCX read?

```python
Document()
→ paragraphs
→ tables
→ text
```

### Q7. How does the program know PDF vs DOCX?

```python
file_path.suffix.lower()
```

### Q8. What does `parse_resume()` do?

```text
Resume text → LLM → structured Resume object
```

### Q9. What does `final_score()` do?

```text
JobD + Resume → LLM → MatchResult
```

### Q10. How are top candidates found?

```python
sort(reverse=True)
top_2 = all_results[:2]
```

### Q11. How are worst candidates found?

```python
worst_2 = all_results[-2:]
```

---

# 🧩 KEY ARCHITECTURE

Remember these **4 transformations**:

```text
1. TEXT → STRUCTURED JD
        ↓
     Pydantic

2. FILE → CLEAN TEXT
        ↓
     pypdf / python-docx

3. TEXT → STRUCTURED RESUME
        ↓
     Pydantic + LLM

4. JD + RESUME → SCORE
        ↓
     LLM + MatchResult
```

### Final Project Formula

```text
Pydantic
   +
LLM
   +
PDF/DOCX Parsing
   +
JSON
   +
Ranking
   =
AI Resume Evaluator
```
