# 🛡️ ScamShield ID

**AI-Powered Digital Fraud Triage Agent built with Langflow and Gemini**

ScamShield ID is an experimental Generative AI system designed to perform structured initial triage of suspected digital fraud cases based on evidence provided by the user.

Instead of using an LLM as a general-purpose chatbot, ScamShield organizes the analysis into a multi-step workflow that transforms raw evidence into a structured risk assessment and a human-readable report.

The project was developed as part of the **IBM x Hacktiv8 Generative AI Capstone Project**.

---

## 🎯 Project Goal

Digital fraud reports often contain fragmented information spread across chat messages, transaction details, phone numbers, URLs, bank accounts, and other evidence.

A typical LLM can summarize this information, but free-form responses introduce several problems:

- inconsistent output format,
- unsupported assumptions,
- difficulty integrating responses into other systems,
- and limited control over what information is extracted.

ScamShield ID explores a different approach:

> Treat the LLM as one component inside a structured AI workflow rather than as a standalone chatbot.

The system analyzes only the evidence supplied by the user and returns a predefined structured assessment containing the case stage, incident category, risk level, extracted entities, confidence level, and recommended actions.

---

## ✨ Key Features

- 🤖 Multi-step Generative AI workflow using Langflow
- 🧩 Schema-constrained structured LLM outputs
- 🔍 Evidence-grounded fraud analysis
- 🏷️ Incident category classification
- 🚦 Risk-level assessment
- 📍 Case-stage classification
- 🧾 Structured entity extraction
- 📝 Human-readable report generation
- ⚠️ Safety notes and explicit system limitations
- 🔄 Separation between machine-readable analysis and presentation output

---

## 🏗️ System Architecture

The current workflow follows this general architecture:

```text
User Evidence
     │
     ▼
Evidence Analysis
     │
     ▼
Structured Risk Assessment
     │
     ▼
Evidence / Consistency Checking
     │
     ▼
Human-Readable Report Formatter
     │
     ▼
Final Fraud Triage Report
```

### Workflow Overview

![ScamShield Langflow Workflow](docs/workflow.png)

The workflow separates analysis from presentation.

The structured risk assessment acts as the primary machine-readable representation, while the final formatting stage converts that assessment into a report that is easier for users to understand.

---

## 🧠 How It Works

### 1. User Evidence

The system receives evidence related to a suspected digital fraud case.

Evidence may contain information such as:

- chat messages,
- suspicious recruitment offers,
- payment requests,
- URLs,
- phone numbers,
- bank account information,
- organization names,
- timestamps,
- transaction amounts,
- or other relevant case details.

The system is instructed to avoid treating missing information as factual.

---

### 2. Evidence Analysis

Gemini analyzes the supplied evidence to identify:

- suspicious indicators,
- the current stage of the incident,
- the likely incident category,
- exposed sensitive information,
- important entities,
- and appropriate next actions.

The analysis is designed to remain grounded in the submitted evidence.

---

### 3. Structured Risk Assessment

Instead of generating only natural-language text, the model returns a structured object.

A simplified representation looks like this:

```json
{
  "case_id": "CASE-001",
  "case_stage": "PRE_TRANSFER",
  "incident_category": "FAKE_RECRUITMENT",
  "risk_level": "HIGH",
  "overall_confidence": 0.95,
  "summary": "The evidence contains multiple indicators associated with a suspicious recruitment scheme.",
  "final_recommendation": "Do not transfer money and independently verify the organization through official channels.",
  "safety_note": "This assessment is based only on the submitted evidence and does not independently verify the identities involved.",
  "extracted_entities": {
    "claimed_organization_names": [],
    "person_names": [],
    "platforms": [],
    "bank_names": [],
    "bank_accounts": [],
    "account_holder_names": [],
    "transaction_amounts": [],
    "phone_numbers": [],
    "urls": [],
    "key_timestamps": [],
    "disclosed_sensitive_information": []
  }
}
```

This structure allows the output to be:

- validated,
- consumed programmatically,
- rendered in a frontend,
- stored in a database,
- or passed into another workflow stage.

---

## 🚦 Case Stages

ScamShield currently defines three primary incident stages.

| Case Stage         | Description                                                              |
| ------------------ | ------------------------------------------------------------------------ |
| `PRE_TRANSFER`     | The user has not transferred money or disclosed sensitive credentials    |
| `POST_TRANSFER`    | The user has already transferred money or experienced financial loss     |
| `POST_CREDENTIALS` | Sensitive information or account credentials have already been disclosed |

Case stage is separated from incident type because the same scam category may require very different recommendations depending on what has already happened.

---

## 🏷️ Incident Categories

The current prototype uses a constrained set of incident categories:

```text
FAKE_RECRUITMENT
PHISHING
INVESTMENT
FAKE_SHOP
ACCOUNT_TAKEOVER
OTHERS
```

Using a predefined enum prevents the model from generating arbitrary category names and makes downstream aggregation and evaluation easier.

---

## 🔎 Entity Extraction

The workflow also extracts important entities from the evidence.

Supported fields include:

```text
claimed_organization_names
person_names
platforms
bank_names
bank_accounts
account_holder_names
transaction_amounts
phone_numbers
urls
key_timestamps
disclosed_sensitive_information
```

For example, a synthetic recruitment-fraud scenario may produce:

```json
{
  "claimed_organization_names": ["PT Nusantara Digital"],
  "person_names": ["Budi Santoso"],
  "platforms": ["WhatsApp"],
  "bank_names": ["Bank Nusantara"],
  "bank_accounts": ["123456789012"],
  "account_holder_names": ["Budi Santoso"],
  "transaction_amounts": ["Rp750.000"],
  "key_timestamps": ["02/08/2026 09:10", "02/08/2026 09:16"]
}
```

All example identities and transaction information in this repository are synthetic and are used only for demonstration.

---

## 🧪 Example Scenario

One test scenario simulates a suspicious recruitment message.

The alleged recruiter:

- claims to represent a company,
- requests an administrative payment,
- provides a bank account under an individual's name,
- creates urgency using a short payment deadline,
- and contacts the user through an informal communication channel.

The user has not transferred any money.

The expected high-level assessment is:

```json
{
  "case_stage": "PRE_TRANSFER",
  "incident_category": "FAKE_RECRUITMENT",
  "risk_level": "HIGH",
  "overall_confidence": 0.95
}
```

Example files can be found in:

```text
examples/
├── fake-recruitment-input.md
└── fake-recruitment-output.json
```

---

## 📊 Example Structured Output

![ScamShield Structured Output](docs/structured-output.png)

The structured response becomes the source of truth for downstream processing.

A separate formatting stage then converts the machine-readable object into a human-readable fraud triage report.

---

## 📝 Human-Readable Report

![ScamShield Final Report](docs/final-report.png)

Keeping structured analysis separate from presentation provides several benefits:

- the UI can change without modifying the core analysis,
- structured fields can be stored or analyzed independently,
- downstream systems do not need to parse natural-language responses,
- and validation can happen before information reaches the user.

---

## 🧩 Why Structured Outputs?

A normal chatbot may produce something like:

```text
This message appears suspicious. You should avoid sending money
and verify the organization through official channels.
```

The response may be useful to a human but difficult for software to consume reliably.

ScamShield instead produces explicit fields such as:

```text
risk_level          = HIGH
case_stage          = PRE_TRANSFER
incident_category   = FAKE_RECRUITMENT
overall_confidence  = 0.95
```

This makes the LLM easier to integrate as a component inside a larger application.

---

## 🛡️ Evidence-Grounded Prompting

One of the main design goals of ScamShield is reducing unsupported conclusions.

For example, if the evidence only says:

```text
Please transfer the administrative fee to the account below.
```

the system should not invent:

- a bank name,
- an account holder,
- a transaction amount,
- the identity of the sender,
- or the legal status of the organization.

The workflow therefore instructs the model to perform analysis only from evidence available in the case.

This does not eliminate hallucination completely, but it reduces the space available for unsupported claims.

---

## 🧰 Tech Stack

### AI & Workflow

- Langflow
- Google Gemini
- Generative AI
- Large Language Models
- Prompt Engineering
- Structured Outputs
- AI Agent Workflows

### Infrastructure

- Docker
- Docker Compose

### Data Representation

- JSON
- Structured schemas
- Template-based report generation

---

## 📁 Repository Structure

```text
scamshield-id/
│
├── README.md
├── docker-compose.yml
├── .env.example
├── .gitignore
│
├── flows/
│   └── scamshield-id.json
│
├── schemas/
│   └── fraud-triage-schema.json
│
├── examples/
│   ├── fake-recruitment-input.md
│   └── fake-recruitment-output.json
│
└── docs/
    ├── workflow.png
    ├── structured-output.png
    └── final-report.png
```

---

# 🚀 Running the Project

## Prerequisites

Make sure the following tools are installed:

- Docker
- Docker Compose

---

## 1. Clone the Repository

```bash
git clone https://github.com/ardavv/scamshield-id.git
cd scamshield-id
```

---

## 2. Configure Langflow

Copy the environment template:

```bash
cp .env.example .env
```

Configure the Langflow environment variables inside `.env`.

The local Docker configuration uses variables including:

```env
LANGFLOW_AUTO_LOGIN=
LANGFLOW_SUPERUSER=
LANGFLOW_SUPERUSER_PASSWORD=
```

Do not commit your real credentials to Git.

---

## 3. Start Langflow

Run:

```bash
docker compose up -d
```

Langflow will be available at:

```text
http://localhost:7860
```

---

## 4. Import the ScamShield Flow

Inside Langflow:

1. Open the Langflow interface.
2. Import the flow from:

```text
flows/scamshield-id.json
```

3. Open the imported workflow.
4. Configure your own Gemini credentials in the relevant model component.
5. Save the flow.

API credentials are intentionally excluded from the public repository.

---

## 5. Run a Test Case

Use the synthetic example located at:

```text
examples/fake-recruitment-input.md
```

Provide the evidence through the workflow input and execute the flow.

Compare the generated structured assessment with:

```text
examples/fake-recruitment-output.json
```

Because Gemini is a generative model, exact wording and confidence values may differ between executions.

---

## 🛑 Stopping the Environment

Stop the containers with:

```bash
docker compose down
```

To also remove Docker volumes:

```bash
docker compose down -v
```

---

# 🐛 Engineering Challenges

Several important lessons came from debugging the workflow.

## 1. Structured Output Reliability

Early versions of the workflow occasionally produced:

```text
No structured output returned
```

Understanding the case correctly was not sufficient.

The model also needed to satisfy the output contract expected by downstream components.

This led to a more explicit structured-output schema.

---

## 2. Schema Alignment

One iteration successfully produced the analysis but failed to render the final report.

The underlying issue was not the model response itself.

The problem was a schema mismatch between the structured-analysis output and the fields expected by the report-generation stage.

This highlighted an important engineering principle:

> AI workflows still depend on traditional software-engineering concepts such as interface contracts and consistent data structures.

---

## 3. Output Rendering

Different output sinks were explored during development.

The final workflow separates the structured model response from a template-based formatting stage used to generate the human-readable report.

---

## 4. External API Dependencies

The project also encountered API quota limitations during experimentation.

This demonstrated that production AI systems must consider more than model quality.

Operational reliability also depends on:

- provider availability,
- quotas,
- latency,
- retries,
- fallback strategies,
- monitoring,
- and cost control.

These areas are not yet fully implemented in the current prototype.

---

# 📚 What I Learned

## LLM Applications Are More Than Prompts

A useful AI application depends on more than prompt engineering.

For ScamShield, system behavior is influenced by:

```text
Prompt
+
Schema
+
Workflow
+
Data Flow
+
Validation
+
Presentation
```

A strong prompt cannot compensate for broken interfaces between components.

---

## Structured Output Bridges LLMs and Software Systems

Natural language is ideal for human communication but unreliable as a software interface.

Structured outputs allow the LLM to participate in a conventional application pipeline with a more explicit contract.

---

## LLM Outputs Should Be Treated as Untrusted Data

Even when a response sounds convincing, its contents still need validation.

This becomes especially important in sensitive domains such as fraud analysis.

---

## Grounding Matters More Than Confidence

High confidence does not automatically mean that an answer is supported by evidence.

For this reason, ScamShield prioritizes evidence-grounded conclusions rather than maximizing model confidence.

---

## Agentic Does Not Mean Fully Autonomous

The workflow intentionally gives each component a narrow responsibility.

Rather than allowing one unrestricted agent to control the entire process, the system uses a more constrained multi-stage pipeline.

The goal is controllability, not maximum autonomy.

---

# ⚠️ Current Limitations

ScamShield ID is an experimental prototype and is **not** a production fraud-detection or law-enforcement system.

Current limitations include:

### No Independent Fact Verification

The system does not independently verify:

- bank accounts,
- organizations,
- individuals,
- domains,
- URLs,
- phone numbers,
- or legal claims.

The assessment is based on the evidence supplied to the system.

### No Production Evaluation Dataset

The current prototype does not yet have a sufficiently large labeled evaluation dataset for measuring metrics such as:

- classification precision,
- recall,
- false-positive rate,
- entity-extraction accuracy,
- hallucination rate,
- or confidence calibration.

### Generative Output Is Non-Deterministic

Gemini may produce slightly different results across executions even when using the same input.

### Limited Multimodal Evidence Support

The current workflow primarily focuses on evidence that can be represented through the configured Langflow input.

A complete fraud-analysis system would benefit from dedicated processing for:

- screenshots,
- PDFs,
- documents,
- transaction receipts,
- image metadata,
- and OCR-derived evidence.

### No Persistent Case Management

The prototype performs individual fraud triage but does not yet include:

- user authentication,
- persistent case histories,
- reviewer assignment,
- audit logs,
- investigation dashboards,
- or case lifecycle management.

---

# 🔮 Future Development

Future experimentation may include:

## Multimodal Evidence Processing

Support evidence from:

```text
Text
Screenshots
Images
PDF Documents
URLs
Transaction Receipts
```

---

## Retrieval-Augmented Verification

Integrate trusted external information sources to distinguish between:

```text
User-Provided Evidence
```

and:

```text
Externally Retrieved Evidence
```

This could support additional verification while preserving clear evidence provenance.

---

## LLM Evaluation Framework

Create a dedicated evaluation dataset for measuring:

- incident classification accuracy,
- case-stage classification,
- entity extraction precision and recall,
- structured-output compliance,
- hallucination rate,
- recommendation consistency,
- and confidence calibration.

---

## Human-in-the-Loop Review

Route ambiguous or high-impact cases to a human reviewer instead of treating AI output as a final decision.

---

## Case Management Platform

The structured output could support a future dashboard containing:

```text
Case Queue
Risk Level
Incident Category
Evidence
Extracted Entities
Recommended Actions
Review Status
Audit History
```

---

# 🔐 Security

Never commit:

```text
.env
API keys
Gemini credentials
Langflow passwords
Access tokens
```

Before publishing an exported Langflow flow, verify that no secret credentials are embedded in the JSON file.

---

# 📌 Disclaimer

ScamShield ID is an educational and experimental AI prototype.

It is intended for initial digital-fraud triage and AI engineering experimentation only.

Its output should not be treated as:

- legal advice,
- confirmation that an individual or organization committed fraud,
- an authoritative fraud report,
- or a replacement for professional investigation.

Important decisions should be verified using appropriate official channels.

---

# 👨‍💻 Author

**Arya Davi Sulaiman**

Informatics Student
Universitas Pembangunan Nasional “Veteran” Jakarta

- GitHub: https://github.com/ardavv
- LinkedIn: https://linkedin.com/in/ardav26
- Portfolio: https://arya.gokiltech.com

---

## ⭐ Project Status

```text
Prototype / Research Project
```

The core Langflow fraud-triage workflow is functional.

Current development focuses on documenting the architecture, improving reproducibility, and exploring more rigorous evaluation and verification approaches.
