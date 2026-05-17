# Anansi-
Clinical workflow intelligence for urgent care: risk stratification, documentation support, EHR integration, and operational guardrails.


Anansi is a physician-built clinical decision support prototype designed around a simple premise: urgent care software should help clinicians see risk earlier, document more cleanly, and avoid predictable workflow failures before they become patient-safety, billing, or operational problems.

Anansi is not a chatbot pasted onto a clinical workflow. It is a structured clinical workflow system with explicit logic modules, EHR data mapping, pre-room readiness checks, risk-tiering, documentation generation, discharge instruction support, and audit-style supervision logic.

> **Status:** Prototype / portfolio system. Not intended for clinical use without formal validation, security review, regulatory review, and institutional governance.

---

## Why Anansi Exists

Urgent care clinicians operate inside a high-friction environment: incomplete histories, variable rooming quality, abnormal vitals that may or may not be rechecked, EHR documentation burden, billing complexity, and patients who often present with vague symptoms that can still hide high-risk disease.

Anansi was built to demonstrate how clinical AI systems can be designed around **workflow reliability**, not novelty.

The system focuses on four recurring failure modes:

1. **Missed risk escalation** — abnormal vitals, high-risk complaints, or risk modifiers not surfaced early enough.
2. **Incomplete pre-rooming** — missing chief complaint timing, vitals, allergies, medications, PMH, or surgical history before provider handoff.
3. **Documentation drift** — notes that fail to reflect clinical reasoning, risk level, workup, return precautions, or billing logic.
4. **Operational supervision gaps** — APP supervision, incident-to billing traps, abnormal discharge vitals, and high-risk cases without documented physician involvement.

Anansi addresses these through deterministic clinical logic, structured data capture, OpenMRS integration, OCR-assisted extraction, and transparent audit outputs.

---

## Core Capabilities

### 1. Pre-Room Readiness Check

Anansi includes an MA-facing pre-room module that checks whether required intake elements are complete before the patient is marked ready for provider review.

It evaluates:

* Chief complaint and timing
* Vital signs
* Allergies
* Medications
* Past medical history
* Past surgical history
* “Unable to obtain” documentation with reason capture

Output states include:

* **Green** — ready for provider
* **Yellow** — usable but flagged
* **Red** — hard stop requiring completion or documented inability to obtain

This creates a structured handoff layer between rooming staff and clinicians.

---

### 2. Weighted Risk Stratification

The clinical logic engine assigns a risk tier based on chief complaint, vitals, age, and clinical risk modifiers.

Risk tiers:

* **Green** — low complexity / routine presentation
* **Orange** — moderate risk / needs structured workup or caution
* **Red** — high risk / possible threat to life or bodily function

The engine can surface:

* Hard interrupt alerts
* Vital sign alerts
* Can’t-miss diagnoses
* Suggested workup
* Risk modifiers
* MDM complexity estimate

Risk modifiers include factors such as:

* Immunocompromised status
* Diabetes
* Cardiac history
* Anticoagulation
* Pregnancy
* Biologic therapy
* Active cancer
* COPD/asthma

The design goal is not to replace clinical judgment. The goal is to make risk harder to ignore.

---

### 3. Clinical Note Generation

Anansi generates structured clinical notes using a clinic-style format:

* Chief Complaint
* History of Present Illness
* Physical Exam
* Testing
* Procedures
* Assessment
* Plan
* Medical Decision Making
* ICD-10 / CPT support

The note generator combines clinical inputs with risk-tiering and billing logic to produce documentation that reflects both clinical reasoning and operational requirements.

---

### 4. Discharge Instruction Generation

Anansi generates patient-facing discharge instructions based on diagnosis and complaint type.

Supported categories include:

* Respiratory complaints
* UTI
* Abdominal complaints
* Cardiac complaints
* General/default presentations

Instructions include:

* Diagnosis summary
* Care instructions
* Medication instructions
* Follow-up guidance
* Return precautions

The goal is to standardize safety-netting while preserving clinician review.

---

### 5. OpenMRS Integration

Anansi includes an OpenMRS client that connects to the OpenMRS REST API and maps EHR patient data into an Anansi-compatible schema.

The integration supports:

* Session-based OpenMRS authentication
* Patient search by name or identifier
* Full patient demographic retrieval
* Observation retrieval
* Vitals mapping
* Diagnosis/problem-list mapping
* Clinical note mapping
* Attachment metadata mapping
* Import of vitals into OpenMRS
* Import of diagnoses into OpenMRS problem list

Mapped vitals include:

* Temperature
* Heart rate
* Respiratory rate
* Blood pressure
* SpO2
* Height
* Weight

Anansi converts OpenMRS patient records into a simplified clinical context object used by the risk engine and UI.

---

### 6. OCR-Assisted Data Extraction

Anansi includes an OCR processor designed to extract clinical data from attached documents or images.

The extraction layer summarizes:

* Vitals
* Lab values
* Diagnoses
* Medications
* Procedures
* Source metadata
* Raw extracted text for review

The current prototype distinguishes between import-ready values and values that are noted but not yet mapped into OpenMRS concepts.

---

### 7. Billing and MDM Support

The billing module supports MDM-level estimation and ICD-10 suggestion logic.

It includes:

* Problem complexity
* Data complexity
* Risk level
* MDM templates
* CPT support
* ICD-10 suggestions
* Documentation tips
* Audit-risk notes

This is designed to help align documentation with clinical complexity and reduce mismatch between what happened clinically and what the chart supports.

---

### 8. APP Supervision / Chart Audit Logic

Anansi includes an APP supervision module that flags predictable oversight and compliance risks.

Examples include:

* Incident-to billing traps
* Incident-to billing without supervising physician availability
* Abnormal vitals at discharge without documented reassessment
* High-risk patient without documented physician consult
* Diagnostic anchoring concerns

The supervision module is framed as operational risk detection, not punitive review.

---

## System Architecture

Anansi is built as a modular Flask application with discrete engines for clinical logic, billing, note generation, pre-room readiness, EHR integration, OCR extraction, and supervision review.

```text
                      ┌────────────────────────┐
                      │        Flask UI         │
                      │  Forms + JSON API       │
                      └───────────┬────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌────────────────┐       ┌────────────────┐       ┌────────────────┐
│ Pre-Room Check │       │ Risk Engine    │       │ Note Generator │
│ intake quality │       │ WRS + alerts   │       │ note + DC docs │
└───────┬────────┘       └───────┬────────┘       └───────┬────────┘
        │                        │                        │
        │                        ▼                        ▼
        │                ┌────────────────┐       ┌────────────────┐
        │                │ Billing Engine │       │ Discharge Logic│
        │                │ MDM + ICD/CPT  │       │ precautions    │
        │                └────────────────┘       └────────────────┘
        │
        ▼
┌────────────────┐       ┌────────────────┐       ┌────────────────┐
│ OpenMRS Client │◄─────►│ OCR Processor  │──────►│ Import Helpers │
│ EHR mapping    │       │ docs/images    │       │ vitals/dx      │
└────────────────┘       └────────────────┘       └────────────────┘
        │
        ▼
┌────────────────┐
│ APP Supervision│
│ audit flags    │
└────────────────┘
```

---

## Major Modules

### `openmrs_client.py`

Responsible for OpenMRS authentication, patient search, patient retrieval, observation retrieval, concept lookup, vitals import, diagnosis import, and conversion to the Anansi patient schema.

Key objects:

* `OpenMRSConfig`
* `OpenMRSVitals`
* `OpenMRSDiagnosis`
* `OpenMRSNote`
* `OpenMRSAttachment`
* `OpenMRSPatient`
* `OpenMRSClient`

Important method:

```python
OpenMRSPatient.to_anansi_dict()
```

This converts an OpenMRS patient record into the simplified structure used by the Anansi clinical logic layer.

---

### `logic_engine.py`

The clinical logic engine performs risk stratification and complaint-based clinical reasoning support.

Responsibilities:

* Assign risk tier
* Detect hard interrupts
* Evaluate vital sign abnormalities
* Identify can’t-miss diagnoses
* Suggest workup
* Estimate MDM complexity

---

### `preroom.py`

The pre-room module checks intake completeness and determines whether a patient is ready for provider evaluation.

Responsibilities:

* Required field checks
* Missing-data flags
* Hard stops
* Unable-to-obtain documentation
* Alert level assignment

---

### `note_generator.py`

Generates structured clinic documentation and patient discharge instructions.

Responsibilities:

* Clinical note generation
* MDM text generation
* Discharge instruction generation
* Complaint-specific return precautions
* Billing-aware documentation support

---

### `billing.py`

Handles billing support logic.

Responsibilities:

* MDM level calculation
* CPT support
* ICD-10 suggestion
* Documentation tips
* Audit-risk notes

---

### `ocr_processor.py`

Extracts structured clinical information from documents and images.

Responsibilities:

* Text extraction
* Vitals extraction
* Labs extraction
* Diagnoses extraction
* Medications extraction
* Procedure extraction
* Source confidence summaries

---

### APP Supervision Module

Audits chart-level supervision risks for APP workflows.

Responsibilities:

* Incident-to billing validation
* Supervising physician presence checks
* Abnormal discharge vitals checks
* Scope escalation checks
* Diagnostic anchoring checks

---

## Web Routes

### User-facing routes

| Route                 | Purpose                             |
| --------------------- | ----------------------------------- |
| `/openmrs`            | Search or load OpenMRS patient data |
| `/preroom`            | Run MA pre-room readiness check     |
| `/`                   | Manual risk assessment form         |
| `/assess`             | Submit risk assessment              |
| `/note`               | Clinical note form                  |
| `/note/generate`      | Generate structured clinical note   |
| `/discharge`          | Discharge instruction form          |
| `/discharge/generate` | Generate patient instructions       |

### API routes

| Route                         | Method | Purpose                                |
| ----------------------------- | -----: | -------------------------------------- |
| `/api/openmrs/search?q=`      |    GET | Search OpenMRS patients                |
| `/api/openmrs/patient/<uuid>` |    GET | Return patient mapped to Anansi schema |
| `/api/preroom/check`          |   POST | Run pre-room check from JSON           |

---

## Data Flow

### OpenMRS-to-risk-assessment path

```text
OpenMRS REST API
      ↓
OpenMRSClient.fetch_patient + fetch_observations
      ↓
OpenMRSPatient object
      ↓
OpenMRSPatient.to_anansi_dict()
      ↓
PatientContext + VitalSigns
      ↓
LogicEngine.assess()
      ↓
RiskAssessment output
      ↓
UI display / note generation / documentation support
```

### OCR-to-import path

```text
Attached document or image
      ↓
OCRProcessor extraction
      ↓
Structured summary of vitals/labs/diagnoses/medications/procedures
      ↓
Review screen
      ↓
OpenMRS import helpers
      ↓
Updated patient chart
```

### Documentation path

```text
Clinical inputs + exam + testing + assessment + plan
      ↓
PatientContext
      ↓
LogicEngine risk tier
      ↓
BillingEngine MDM support
      ↓
NoteGenerator
      ↓
Structured clinical note + CPT/ICD support
```

---

## Design Principles

### 1. Structured logic over vibes

Anansi uses explicit clinical and operational rules rather than relying only on free-form model output. If the system flags something, the reason should be visible.

### 2. Human-in-the-loop by default

Anansi is designed to support clinicians, not replace them. Generated notes, risk tiers, discharge instructions, and billing suggestions require human review.

### 3. Workflow-first architecture

The system follows the clinical workflow: rooming, data ingestion, risk assessment, documentation, discharge, and supervision review.

### 4. Safety before elegance

Hard stops, warning tiers, and audit flags are intentionally blunt. Clinical software should be less interested in seeming clever than in preventing predictable failure.

### 5. Explainability as usability

Anansi surfaces rationale, missing fields, risk modifiers, and documentation gaps so users can understand what the system is doing and why.

---

## Current Prototype Limitations

Anansi is an early prototype and should be treated accordingly.

Known limitations:

* Not validated for clinical deployment
* Not a medical device
* Not HIPAA-ready in its current form
* Uses demo OpenMRS configuration in current prototype state
* Inline Flask templates should be moved into a production template structure
* Authentication, authorization, audit logs, and PHI safeguards need production hardening
* OCR extraction requires validation and confidence thresholds before clinical use
* Billing suggestions require compliance review
* Risk logic requires formal clinical validation and version-controlled rulesets
* No production database layer is included in the current public-facing prototype description

---

## Future Roadmap

Potential next steps:

1. Externalize clinical rules into versioned configuration files.
2. Add formal test suites for risk-tiering, vitals thresholds, and complaint bundles.
3. Add role-based access control.
4. Add audit logging for every generated recommendation and user override.
5. Move inline templates into a maintainable frontend structure.
6. Add FHIR-compatible integration pathway.
7. Add model-assisted summarization only behind deterministic validation gates.
8. Add formal evaluation harness comparing Anansi outputs to clinician-reviewed cases.
9. Add deployment hardening: secrets management, environment config, database persistence, monitoring, and secure hosting.
10. Add documentation for institutional review, compliance, and validation workflows.

---

## Portfolio Significance

Anansi demonstrates the kind of AI healthcare architecture that sits between clinical judgment and operational infrastructure.

It shows:

* End-to-end clinical workflow modeling
* EHR data ingestion and schema mapping
* Deterministic safety logic
* Human-in-the-loop documentation support
* AI-assisted but reviewable clinical workflow augmentation
* Billing and supervision risk awareness
* Practical translation of frontline clinical knowledge into software architecture

This is not a demo about “AI can write a note.”

It is a demo about how a clinician-engineer thinks when building systems for messy, high-stakes environments where the software has to earn trust.

---

## Disclaimer

Anansi is a prototype built for demonstration, research, and portfolio purposes. It is not approved, validated, or intended for use in patient care. Any clinical deployment would require formal validation, security review, compliance review, regulatory assessment, institutional governance, and clinician oversight.
