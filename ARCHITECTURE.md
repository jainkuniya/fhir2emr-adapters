# Architecture Diagrams

## System Overview

```
╔══════════════════════════════════════════════════════════════════════╗
║                    HEALTHCARE ECOSYSTEM                              ║
╚══════════════════════════════════════════════════════════════════════╝

                         ┌─────────────┐
                         │   Doctor    │
                         │  Patient    │
                         │ Conversation│
                         └──────┬──────┘
                                │
                                │ Voice/Text
                                ▼
                    ┌───────────────────────┐
                    │   SCRIBE ENGINE       │
                    │ (medScribe Alliance)  │
                    ├───────────────────────┤
                    │ • Speech Recognition  │
                    │ • NLP Processing      │
                    │ • Entity Extraction   │
                    │ • FHIR Generation     │
                    └───────────┬───────────┘
                                │
                                │ FHIR R4/R5 Bundle
                                │ (Standard Format)
                                ▼
            ╔═══════════════════════════════════════╗
            ║      EKA EMR ADAPTER (This Module)    ║
            ╠═══════════════════════════════════════╣
            ║                                       ║
            ║  ┌─────────────────────────────────┐ ║
            ║  │     Main Adapter (index.js)     │ ║
            ║  └──────────────┬──────────────────┘ ║
            ║                 │                     ║
            ║    ┌────────────┴────────────┐       ║
            ║    │                         │       ║
            ║    ▼                         ▼       ║
            ║  ┌─────────┐           ┌─────────┐  ║
            ║  │ Parsers │           │  Utils  │  ║
            ║  ├─────────┤           ├─────────┤  ║
            ║  │ • Symptoms          │ • ID Gen│  ║
            ║  │ • Diagnosis         │ • Filter│  ║
            ║  │ • Medications       │ • Map   │  ║
            ║  │ • History           │ • Extract   ║
            ║  │ • Vitals            │ • Convert   ║
            ║  │ • Labs              │         │  ║
            ║  │ • Procedures        │         │  ║
            ║  └─────────┘           └─────────┘  ║
            ║                                       ║
            ╚════════════════╦══════════════════════╝
                             │
                             │ EKA Care Format
                             │ (Proprietary JSON)
                             ▼
                    ┌─────────────────┐
                    │  EKA CARE EMR   │
                    ├─────────────────┤
                    │ • Records       │
                    │ • Prescriptions │
                    │ • Analytics     │
                    └─────────────────┘
```

## Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INPUT: FHIR BUNDLE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  {                                                                   │
│    "resourceType": "Bundle",                                         │
│    "type": "collection",                                             │
│    "entry": [                                                        │
│      { "resource": { "resourceType": "Observation", ... }},          │
│      { "resource": { "resourceType": "Condition", ... }},            │
│      { "resource": { "resourceType": "MedicationRequest", ... }},    │
│      ...                                                             │
│    ]                                                                 │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ convertFHIRToEkaEMR()
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ADAPTER PROCESSING                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. Parse Bundle Entries                                             │
│     ├─> Filter by resourceType                                       │
│     ├─> Extract resources                                            │
│     └─> Validate structure                                           │
│                                                                      │
│  2. Call Parser Modules                                              │
│     ├─> parseSymptoms(entries)        → symptoms[]                   │
│     ├─> parseDiagnosis(entries)       → diagnosis[]                  │
│     ├─> parseMedications(entries)     → medications[]                │
│     ├─> parseMedicalHistory(entries)  → medicalHistory{}             │
│     ├─> parseVitals(entries)          → vitals[]                     │
│     ├─> parseLabTests(entries)        → labTests[]                   │
│     ├─> parseLabVitals(entries)       → labVitals[]                  │
│     ├─> parseProcedures(entries)      → procedures[]                 │
│     ├─> parseFollowup(entries)        → followup{}                   │
│     └─> parsePrescriptionNotes(...)   → prescriptionNotes{}          │
│                                                                      │
│  3. Build EKA Output                                                 │
│     └─> Combine all parsed sections                                  │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   │ Return EKA EMR object
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      OUTPUT: EKA EMR FORMAT                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  {                                                                   │
│    "language": "EN",                                                 │
│    "symptoms": [...],                                                │
│    "diagnosis": [...],                                               │
│    "medications": [...],                                             │
│    "medicalHistory": {                                               │
│      "patientHistory": {                                             │
│        "patientMedicalConditions": [...],                            │
│        "currentMedications": [...],                                  │
│        "familyHistory": [...],                                       │
│        "lifestyleHabits": [...],                                     │
│        "foodOtherAllergy": [...],                                    │
│        "drugAllergy": [...],                                         │
│        "pastProcedures": [...]                                       │
│      },                                                              │
│      "vitals": [...],                                                │
│      "examinations": [...]                                           │
│    },                                                                │
│    "labTests": [...],                                                │
│    "labVitals": [...],                                               │
│    "procedures": [...],                                              │
│    "followup": {...},                                                │
│    "prescriptionNotes": {...}                                        │
│  }                                                                   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Parser Module Architecture

```
┌───────────────────────────────────────────────────────────────────┐
│                    PARSER MODULE STRUCTURE                         │
└───────────────────────────────────────────────────────────────────┘

Each parser follows a consistent pattern:

┌─────────────────────────────────────────────────────────────────┐
│  Parser Module (e.g., symptoms.js)                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Input: FHIR Bundle Entries                                      │
│     │                                                            │
│     ▼                                                            │
│  1. Filter Resources                                             │
│     └─> getResourcesByType(entries, 'Observation')              │
│     └─> Filter by category (e.g., 'symptom')                    │
│                                                                  │
│  2. Extract Data                                                 │
│     ├─> Get name/display text                                   │
│     ├─> Extract components (severity, duration, etc.)           │
│     ├─> Parse dates and calculate durations                     │
│     └─> Extract notes/annotations                               │
│                                                                  │
│  3. Map to EKA Format                                            │
│     ├─> Generate EKA ID                                          │
│     ├─> Build properties object                                  │
│     ├─> Create track metadata                                    │
│     └─> Apply EKA-specific transformations                       │
│                                                                  │
│  4. Return Array                                                 │
│     └─> Array of EKA-formatted objects                           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Integration Patterns

### Pattern 1: Real-time Webhook
```
┌─────────┐       POST       ┌──────────┐    Convert    ┌──────────┐
│ Scribe  │ ───────────────> │ Adapter  │ ───────────> │ EKA Care │
│ Engine  │   FHIR Bundle    │ Service  │  EKA Format  │   API    │
└─────────┘                  └──────────┘               └──────────┘
                                   │
                                   ▼
                             [Webhook Endpoint]
                             - Receive FHIR
                             - Convert
                             - Forward to EKA
```

### Pattern 2: Batch Processing
```
┌─────────┐    Upload    ┌────────┐    Process    ┌──────────┐
│ Scribe  │ ──────────> │ Queue  │ ───────────> │ Adapter  │
│ Engine  │  Multiple    │  (SQS) │   One by One │  Worker  │
└─────────┘  Files       └────────┘              └────┬─────┘
                                                       │
                                                       ▼
                                                  ┌──────────┐
                                                  │ EKA Care │
                                                  │   API    │
                                                  └──────────┘
```

### Pattern 3: Serverless (Lambda)
```
┌─────────┐   Upload    ┌────────┐   Trigger   ┌──────────┐
│ Scribe  │ ─────────> │   S3   │ ──────────> │  Lambda  │
│ Engine  │  to Bucket  │ Bucket │   Lambda    │ Function │
└─────────┘             └────────┘             └────┬─────┘
                                                     │
                                                     │ Convert
                                                     ▼
                                               ┌──────────┐
                                               │   S3     │
                                               │  Output  │
                                               └──────────┘
```

## Module Dependencies

```
┌────────────────────────────────────────────────────────────┐
│                   DEPENDENCY GRAPH                          │
└────────────────────────────────────────────────────────────┘

                    ┌─────────────┐
                    │  index.js   │
                    │   (Main)    │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   ┌─────────┐      ┌──────────┐      ┌──────────┐
   │ Parsers │      │  Utils   │      │ Examples │
   └────┬────┘      └────┬─────┘      └──────────┘
        │                │
        │                │
   ┌────┴────┐          │
   │         │          │
   ▼         ▼          ▼
[symptoms] [diagnosis] [helpers]
[medications]          [generateId]
[medicalHistory]       [getResources]
[vitals]               [extractText]
[labTests]             [mapSeverity]
[labVitals]            [calculateDuration]
[procedures]           [createTrack]
[injections]
[followup]
[prescriptionNotes]

All parsers depend on utils/helpers.js
```

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                 DEPLOYMENT OPTIONS                           │
└─────────────────────────────────────────────────────────────┘

Option 1: Docker Container
┌────────────────────────────────────────┐
│          Docker Container              │
├────────────────────────────────────────┤
│  ┌──────────────────────────────────┐  │
│  │   Node.js App                    │  │
│  │   - Express Server               │  │
│  │   - EKA Adapter                  │  │
│  │   - API Endpoints                │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Port 3000 exposed                     │
└────────────────────────────────────────┘

Option 2: Serverless (AWS Lambda)
┌────────────────────────────────────────┐
│          AWS Lambda Function           │
├────────────────────────────────────────┤
│  ┌──────────────────────────────────┐  │
│  │   Handler Function               │  │
│  │   - Load FHIR from S3            │  │
│  │   - Convert using Adapter        │  │
│  │   - Save to S3                   │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Triggered by S3 events                │
└────────────────────────────────────────┘

Option 3: Kubernetes Pod
┌────────────────────────────────────────┐
│          Kubernetes Pod                │
├────────────────────────────────────────┤
│  ┌──────────────────────────────────┐  │
│  │   Adapter Service                │  │
│  │   - Multiple Replicas            │  │
│  │   - Load Balanced                │  │
│  │   - Auto-scaling                 │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Service: adapter-service:3000         │
└────────────────────────────────────────┘
```

---

These diagrams illustrate the complete architecture of the EKA EMR Adapter system.
