# 📝 Notes – Structured Output from LLMs

## 🎯 Goal
Instead of getting random text from an LLM, force it to return **structured, validated JSON** that can be directly used in applications.

---

# 1. Why Raw LLM Output is a Problem?

### Problem
LLMs normally return plain text.

Example:
```text
Hi, I'm Priyam.
My email is xyz@gmail.com.
My issue is login failure.
```

Every response can have a different format.

### Issues
- ❌ Difficult to parse
- ❌ Not reliable
- ❌ Can break automation
- ❌ Hard to use in APIs or databases

### Trick
> **Humans read text, Machines read JSON.**

---

# 2. Pydantic

### Definition
Pydantic is a Python library used for **data validation** and **data modeling**.

It checks whether data follows the expected structure.

```python
from pydantic import BaseModel

class Ticket(BaseModel):
    name: str
    email: str
    issue: str
```

### Benefits
- Type checking
- Validation
- Error handling
- Auto JSON Schema generation

### Trick
> **Pydantic = Data Police 👮**
>
> "Only correct data is allowed."

---

# 3. BaseModel

All models inherit from `BaseModel`.

Example

```python
class Ticket(BaseModel):
    name:str
    email:str
    issue:str
```

This means every Ticket object **must** contain:

- name
- email
- issue

---

# 4. JSON Schema

Generate schema using

```python
schema = Ticket.model_json_schema()
```

Example Schema

```json
{
  "properties": {
    "name": {"type":"string"},
    "email":{"type":"string"},
    "issue":{"type":"string"}
  }
}
```

This schema tells the LLM exactly what fields are expected.

### Trick
> **Schema = Blueprint of Output**

---

# 5. response_format

```python
response_format={
    "type":"json_object"
}
```

This enables **JSON Mode**.

Now the model is forced to return JSON.

Without it

```text
Hello sir,
Name is Priyam...
```

With it

```json
{
  "name":"Priyam",
  "email":"abc@gmail.com",
  "issue":"iphone not working"
}
```

### Trick
> **response_format = JSON Lock 🔒**

---

# 6. System Prompt

System prompt controls model behaviour.

Example

```python
system_prompt=f"""
Extract the personal information
strictly based on this schema.

{schema}
"""
```

Purpose

- Gives instructions
- Shares schema
- Forces structure

### Trick
> **System Prompt = Teacher's Instructions**

---

# 7. User Prompt

Contains the actual customer message.

Example

```python
prompt=f"""
This is a customer ticket.

{text}
"""
```

System Prompt → Rules

User Prompt → Data

---

# 8. LLM Response

```python
response = client.chat.completions.create(...)
```

Output

```python
answer=response.choices[0].message.content
```

Example

```json
{
 "name":"Pratyush",
 "email":"abc@gmail.com",
 "issue":"iphone not working"
}
```

---

# 9. Convert JSON to Python

```python
import json

data=json.loads(answer)
```

Now

```python
data["name"]
```

works.

### Trick
> **loads() = JSON → Dictionary**

---

# 10. Convert Dictionary into Pydantic Object

```python
ticket=Ticket(**data)
```

Now

```python
ticket.name
ticket.email
ticket.issue
```

instead of

```python
data["name"]
```

### Trick
> **Dictionary → Smart Object**

---

# 11. Why Use Pydantic After JSON?

JSON only checks syntax.

Pydantic checks

- Required fields
- Data types
- Missing values
- Invalid values

Example

Wrong

```json
{
 "name":123
}
```

Pydantic throws validation error.

---

# 12. Validation Errors

If output doesn't match schema

```python
Ticket(**data)
```

raises

```
ValidationError
```

Always use

```python
try:
    ticket=Ticket(**data)
except ValidationError:
    print("Invalid Data")
```

### Trick
> **Never trust the LLM blindly. Validate everything.**

---

# 13. Literal Type

Restricts output to fixed values.

Example

```python
from typing import Literal

priority: Literal["Low","Medium","High"]
```

Allowed

```
Low
Medium
High
```

Not Allowed

```
Urgent
Critical
Random
```

Useful for

- Category
- Priority
- Department
- Status

### Trick
> **Literal = Only these options allowed.**

---

# 14. Temperature

For extraction tasks

```python
temperature=0
```

Why?

Because extraction should be

- Deterministic
- Accurate
- Repeatable

Not creative.

### Trick

| Task | Temperature |
|------|-------------|
| Story Writing | 0.8–1.2 |
| Chat | 0.5 |
| Extraction | **0** |

Remember

> **Extraction = Creativity ❌ Accuracy ✅**

---

# 15. Workflow

```
Customer Text
      │
      ▼
System Prompt + Schema
      │
      ▼
LLM
      │
      ▼
JSON Output
      │
      ▼
json.loads()
      │
      ▼
Pydantic Validation
      │
      ▼
Python Object
      │
      ▼
Database / API / Backend
```

---

# 16. Code Flow

```
Customer Ticket
        │
        ▼
Pydantic Model
        │
        ▼
Generate JSON Schema
        │
        ▼
Send Schema to LLM
        │
        ▼
Enable JSON Mode
        │
        ▼
Receive JSON
        │
        ▼
json.loads()
        │
        ▼
Pydantic Validation
        │
        ▼
Use Anywhere
```

---

# 17. Interview Questions

### What is Pydantic?
A Python library for data validation and parsing.

---

### Why use JSON Mode?
To force the LLM to return valid JSON instead of plain text.

---

### Why use Pydantic after JSON?
JSON ensures format; Pydantic ensures correctness and data types.

---

### What is JSON Schema?
A blueprint describing the expected JSON structure.

---

### Why use Literal?
To restrict fields to predefined values.

---

### Why use temperature = 0?
Because extraction tasks require consistency, not creativity.

---

# 18. Homework Idea

## Resume Matching AI

### Input
- Resume (PDF/DOCX)
- HR Requirements

### Extract
- Name
- Skills
- Experience
- Projects
- Education

### Compare
```
Resume Skills
        │
        ▼
HR Skills
        │
        ▼
Matching %
```

### Output

```json
{
  "match_percentage": 86,
  "matched_skills": [
    "Python",
    "SQL",
    "Machine Learning"
  ],
  "missing_skills": [
    "Docker",
    "AWS"
  ]
}
```

---

# 🚀 Quick Revision

- Raw LLM output → Unstructured text
- Pydantic → Validates data
- BaseModel → Defines structure
- JSON Schema → Blueprint of output
- response_format → Forces JSON
- System Prompt → Rules for the LLM
- User Prompt → Actual input data
- json.loads() → JSON → Python dict
- Ticket(**data) → Dict → Validated object
- Literal → Restrict allowed values
- ValidationError → Handles invalid output
- Temperature = 0 → Best for extraction
- Structured Output → Reliable automation

---

# 🧠 Memory Tricks

- 📄 **Schema = Blueprint**
- 👮 **Pydantic = Data Police**
- 🔒 **response_format = JSON Lock**
- 📥 **loads() = JSON → Dict**
- 🧱 **BaseModel = Data Structure**
- 🚦 **Literal = Fixed Choices**
- 🎯 **Temperature = 0 → Accuracy First**
- 🤖 **Humans read text, Machines read JSON**