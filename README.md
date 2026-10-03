# ClearanceRAG — Professional Secure Enterprise Research Agent
### Powered by Google Gemini & Multi-Layer Document Authorization — Challenge P14

ClearanceRAG is an enterprise research agent that answers natural-language questions over confidential internal documents while strictly enforcing document-level authorization and role-based hierarchy restrictions **before evidence reaches the LLM reasoning layer**.

> **Core Security Invariant:**  
> `User Query → Keyword Guard → Metadata Retrieval → Authorization Gate → Evidence Filtering → Conflict Resolution → LLM Synthesis → Citation Verification → Audit Logging`  
>  
> Unauthorized or restricted document content is **never** passed into synthesis, ensuring zero data leakage to reasoning models or final outputs.

---

## 🔑 1. Executive & Employee Demo Credentials

ClearanceRAG includes an authentication system supporting a **4-tier role hierarchy** (Executive, Finance, Marketing, Engineer). Use the credentials below to log into the Streamlit UI or authenticate via `/auth/login`.

| User ID | Employee Name | Role | Department | Clearance Level | Password | Access Scope & Hierarchy |
|---|---|---|---|---|---|---|
| `U001` | **Rajesh Kumar** | Executive | Executive | Restricted | `exec@2026` | 👑 **Level 1 (Highest)** — Unrestricted access across all documents & topics |
| `U002` | **Priya Sharma** | Executive | Executive | Restricted | `exec@2026` | 👑 **Level 1 (Highest)** — Unrestricted access across all documents & topics |
| `U003` | **Ankit Verma** | Finance | Finance | Confidential | `fin@2026` | 📊 **Level 2** — Financial forecasts, budgets, internal department docs |
| `U004` | **Sneha Patel** | Finance | Finance | Confidential | `fin@2026` | 📊 **Level 2** — Financial forecasts, budgets, internal department docs |
| `U005` | **Vikram Singh** | Marketing | Marketing | Internal | `mkt@2026` | 📢 **Level 3** — Campaign calendars, brand strategy, internal docs |
| `U006` | **Deepa Nair** | Marketing | Marketing | Internal | `mkt@2026` | 📢 **Level 3** — Campaign calendars, brand strategy, internal docs |
| `U007` | **Arjun Mehta** | Engineer | Engineering | Internal | `eng@2026` | 💻 **Level 4** — Technical roadmaps, system architecture, internal docs |
| `U008` | **Kavitha Rajan** | Engineer | Engineering | Internal | `eng@2026` | 💻 **Level 4** — Technical roadmaps, system architecture, internal docs |

---

## 🚫 2. Hierarchy-Based Keyword Restrictions

In addition to document clearance and role metadata checks, ClearanceRAG enforces a **Pre-Retrieval Keyword Guard**. Queries containing restricted keywords for a user's role are blocked before candidate retrieval or LLM execution:

* **Executive**: Unrestricted. Can query any topic (Revenue, M&A, Restructuring, Roadmaps, Budgets).
* **Finance**: Blocked from M&A/Acquisition strategies, Marketing Campaigns, Engineering Roadmaps.
* **Marketing**: Blocked from Revenue Forecasts, Financial Budgets, M&A/Acquisition strategies, System Architecture.
* **Engineer**: Blocked from Revenue Forecasts, Financial Budgets, M&A/Acquisition strategies, Marketing Strategy.

---

## 📥 3. Document Ingestion & Input

ClearanceRAG allows administrators and employees to dynamically upload and ingest new internal documents into the repository.

### Streamlit UI Input
Navigate to the **📄 Document Input** tab in the Streamlit application to add new documents with custom:
- Title & Classification (`Public`, `Internal`, `Confidential`, `Restricted`)
- Allowed Departments & Allowed Roles
- Required Clearance Level
- Version & Effective Date
- Text Content

### API Ingestion Endpoint
```bash
POST http://localhost:8000/documents/upload
Content-Type: application/json

{
  "title": "Q1 2027 Expansion Strategy",
  "classification": "Confidential",
  "allowed_departments": ["Finance", "Executive"],
  "allowed_roles": ["Finance", "Executive"],
  "clearance_required": "Confidential",
  "version": "1.0",
  "effective_date": "2026-10-01",
  "content": "The Q1 2027 expansion plan targets entry into 3 new regional markets."
}
```

---

## 🤖 4. Google Gemini LLM Integration

ClearanceRAG connects directly to the **Google Gemini API** (`gemini-flash-latest`) for real-time natural language synthesis over authorized evidence.

### API Configuration
Configured in `backend/.env`:
```env
GEMINI_API_KEY=AQ.Ab8RN6JFmvZvoaFu1W2A1bkbpG4hiTYAGLk2wuTdcBqp2YhLig
LLM_PROVIDER=gemini
LLM_MODEL=gemini-flash-latest
```
*(Note: If no API key is provided, ClearanceRAG automatically falls back to deterministic mock synthesis for offline testing).*

---

## ⚡ 5. Quick Start Guide

### Prerequisites
- Python 3.10+
- `pip`

### Step 1 — Start FastAPI Backend
From the repository root:
```bash
cd backend
python -m pip install -r requirements.txt
python -m uvicorn app.main:app --reload --port 8000
```
Backend Health Check: [http://localhost:8000/health](http://localhost:8000/health)

### Step 2 — Start Premium Streamlit Frontend
In a second terminal from the repository root:
```bash
streamlit run frontend/app.py
```
Access the application UI at: [http://localhost:8501](http://localhost:8501)

---

## 🧪 6. Automated Testing

ClearanceRAG includes an extensive Pytest test suite covering authentication, keyword guard policies, document authorization, version conflict resolution, prompt injection defenses, and end-to-end PS 14 test scenarios.

Run all tests:
```bash
cd backend
python -m pytest -v
```

### Verified Test Categories
- `test_ps14_scenarios.py`: Validates Test Inputs A (Authorized), B (Unauthorized), C (Conflict Resolution), and Prompt Injection.
- `test_keyword_guard.py`: Validates hierarchy restrictions across Executive, Finance, Marketing, and Engineer roles.
- `test_auth.py`: Validates JWT login, credential verification, and token revocation.
- `test_authorization.py`, `test_citations.py`, `test_conflict_resolution.py`, `test_end_to_end.py`, `test_retrieval.py`.

---

## 🏗️ 7. Architecture Overview

```text
               +-----------------------+
               | User Query + Role/JWT |
               +-----------------------+
                           |
                           v
              +-------------------------+
              | Pre-Retrieval Guard     | ---> [BLOCKED] Keyword Restricted
              +-------------------------+
                           | Passed
                           v
              +-------------------------+
              | Lexical Candidate Search| ---> Metadata Candidates Only
              +-------------------------+
                           |
                           v
              +-------------------------+
              | Authorization Gate      | ---> [DENIED] Unauthorized Content Filtered
              +-------------------------+
                           | Authorized Evidence
                           v
              +-------------------------+
              | Conflict Resolver       | ---> Latest Effective Version Selected
              +-------------------------+
                           |
                           v
              +-------------------------+
              | Google Gemini Synthesis | ---> Generates Answer from Evidence Only
              +-------------------------+
                           |
                           v
              +-------------------------+
              | Citation Verifier & Log | ---> Final Answer + Audit ID
              +-------------------------+
```

---

**Team:** TWATOSPHERE  
**Challenge:** P14 — Secure Enterprise Research Agent
