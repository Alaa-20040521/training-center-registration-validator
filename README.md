# 📝 Training Center — Registration Validation Agent

An [n8n](https://n8n.io) workflow that validates student registration submissions for a training center using an AI-powered LLM Chain, then automatically logs accepted registrations to Google Sheets.

---

## ✨ Features

- **Web form intake** — collects registrant data through an n8n Form Trigger (Name, Email, Age, Course selection, Additional notes).
- **AI-powered validation** — uses an OpenAI Chat Model (via Basic LLM Chain) to validate the submitted data against business rules and return a structured JSON verdict.
- **Conditional routing** — an `If` node checks the validation status and routes accordingly.
- **Automated logging** — accepted registrations are appended to a Google Sheet with full submission details, validation result, and timestamp.

---

## 🧩 Workflow structure

```
On form submission
      │
Basic LLM Chain   (OpenAI validates Name & Age against rules → returns JSON)
      │
     If            (checks: status == "مقبول" / accepted)
      │
Append row in sheet   (logs the registration + validation result to Google Sheets)
```

### Nodes

| Node | Purpose |
|---|---|
| **On form submission** | Form trigger that collects: `Name`, `Email Address`, `Age`, `Selected course`, `Additional notes`. |
| **Basic LLM Chain** | Sends the submitted data to the OpenAI model with a strict validation prompt and expects a structured JSON response. |
| **OpenAI Chat Model** | The language model used by the Basic LLM Chain. |
| **If** | Parses the model's JSON output and checks whether `status` equals the accepted value. |
| **Append row in sheet** | Google Sheets node — appends a row with the registrant's data, validation status, message, and submission date. |

---

## 📋 Form fields

| Field | Type |
|---|---|
| Name | Text Input |
| Email Address | Email |
| Age | Number |
| Selected course | Checkboxes |
| Additional notes | Textarea |

---

## 🧠 Validation rules (AI prompt)

The AI acts as a **registration validation assistant** for the training center, enforcing:

1. **Name** — must be between 3 and 15 characters.
2. **Age** — must be between 10 and 18 years.

**Output format** — the model must return **only** a JSON object, with no extra text:

```json
{
  "status": "مقبول" | "غير مقبول",
  "validation_message": "...",
  "submitted_at": "current date"
}
```

- `"مقبول"` (accepted) → `"تم قبول التسجيل بنجاح"`
- `"غير مقبول"` (rejected) → one of the specific rejection reasons:
  - الاسم يجب أن يكون بين 3 و 15 حرف
  - العمر يجب أن يكون بين 10 و 18 سنة
  - الاسم والعمر غير متوافقين مع الشروط

---

## 📊 Google Sheet columns (`Append row in sheet`)

| Column | Source |
|---|---|
| Name | Form submission |
| Email Address | Form submission |
| Age | Form submission |
| Selected course | Form submission |
| Additional notes | Form submission |
| Status | AI validation result |
| Validation Message | AI validation result |
| Submitted At | Form submission timestamp (formatted `dd/MM/yyyy`) |

---

## ⚙️ Setup

### Prerequisites
- An [n8n](https://n8n.io) instance (cloud or self-hosted).
- An OpenAI API key.
- A Google account with access to Google Sheets.
- A Google Sheet (e.g. `Exam` → `course` tab) with the columns listed above.

### Steps
1. **Import the workflow** into n8n (`Import from File` → select the exported `.json`).
2. **Add credentials**:
   - OpenAI API credential.
   - Google Sheets OAuth2 credential.
3. **Connect your Google Sheet** in the `Append row in sheet` node (select your own Document and Sheet).
4. **Review the form fields** in `On form submission` and adjust labels/options as needed (e.g. course list).
5. **Activate the workflow**, then share the generated form URL with registrants.

---

## 🔐 Security notes

- The exported workflow JSON does **not** include credentials — reconnect OpenAI and Google Sheets credentials after importing.
- Do not commit real registrant data (names, emails, ages) or API keys to this repository.

---

## License
This project is licensed under the MIT License — see the LICENSE file.
If you use or adapt this workflow, please credit Ala'a Yahia Almousa with a link back to this repository.
