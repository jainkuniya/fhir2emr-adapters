# EKA EMR Adapter

> Convert FHIR-based scribe output to EKA Care EMR input format

## Overview

This adapter enables seamless integration between any [medScribe Alliance](https://github.com/medScribeAlliance/) compliant scribe engine and EKA Care EMR system. It transforms FHIR Bundle resources into EKA Care's proprietary EMR input format.

## 🔄 Architecture & Integration Flow

```
┌─────────────────────┐
│                     │
│  Clinical Voice     │
│  Conversation       │
│                     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────────────────────────┐
│                                         │
│      medScribe Alliance Protocol        │
│      (Any Compliant Scribe Engine)      │
│                                         │
│  - Amazon HealthScribe                  │
│  - Azure Healthcare Bot                 │
│  - Google Cloud Healthcare API          │
│  - Custom AI Scribes                    │
│                                         │
└──────────┬──────────────────────────────┘
           │
           │  FHIR R4/R5 Bundle
           │  (Standard Healthcare Format)
           │
           ▼
┌─────────────────────────────────────────┐
│                                         │
│       🔌 EKA EMR ADAPTER 🔌             │
│         (This Module)                   │
│                                         │
│  Converts:                              │
│  • Symptoms                             │
│  • Diagnosis                            │
│  • Medications                          │
│  • Medical History                      │
│  • Vitals                               │
│  • Lab Tests & Results                  │
│  • Procedures                           │
│  • Follow-up                            │
│                                         │
└──────────┬──────────────────────────────┘
           │
           │  EKA Care EMR Format
           │  (Proprietary JSON)
           │
           ▼
┌─────────────────────────────────────────┐
│                                         │
│         EKA CARE EMR SYSTEM             │
│                                         │
│  • Patient Records                      │
│  • Prescription Generation              │
│  • Clinical Decision Support            │
│  • Analytics & Reporting                │
│                                         │
└─────────────────────────────────────────┘
```

## 🌟 Key Benefits

- **Universal Compatibility**: Works with any medScribe Alliance compliant scribe engine
- **Standardized Input**: Uses FHIR (Fast Healthcare Interoperability Resources) standard
- **Modular Design**: Each medical entity (symptoms, diagnosis, etc.) has its own parser
- **Extensible**: Easy to add new parsers or customize existing ones
- **Type Safety**: Well-documented interfaces and data structures

## 📦 Installation

```bash
npm install eka-emr-adapter
```

Or clone and install locally:

```bash
git clone <repository-url>
cd eka-emr-adaptar
npm install
```

## 🚀 Quick Start

### Basic Usage

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');
const fs = require('fs');

// Load FHIR Bundle from scribe output
const fhirBundle = JSON.parse(fs.readFileSync('scribe_output.json', 'utf-8'));

// Convert to EKA EMR format
const ekaEMRInput = convertFHIRToEkaEMR(fhirBundle);

// Save or send to EKA Care API
fs.writeFileSync('eka-emr-input.json', JSON.stringify(ekaEMRInput, null, 2));
console.log('✅ Conversion completed successfully!');
```

### Async Usage

```javascript
const { convertFHIRToEkaEMRAsync } = require('eka-emr-adapter');

async function processScribeOutput(fhirBundle) {
  try {
    const ekaEMRInput = await convertFHIRToEkaEMRAsync(fhirBundle);
    
    // Send to EKA Care API
    await sendToEkaCareAPI(ekaEMRInput);
    
    console.log('✅ Data synced to EKA Care EMR');
  } catch (error) {
    console.error('❌ Conversion failed:', error.message);
  }
}
```

## 📋 Supported FHIR Resources

The adapter processes the following FHIR resource types:

| FHIR Resource | EKA EMR Section | Description |
|---------------|-----------------|-------------|
| `Observation` (symptom) | `symptoms` | Patient-reported symptoms |
| `Condition` (encounter-diagnosis) | `diagnosis` | Current diagnoses |
| `Condition` (problem-list-item) | `medicalHistory.patientMedicalConditions` | Historical conditions |
| `MedicationRequest` | `medications` | Prescribed medications |
| `MedicationStatement` | `medicalHistory.currentMedications` | Current medications |
| `Observation` (vital-signs) | `medicalHistory.vitals` | Vital signs (BP, pulse, etc.) |
| `Observation` (laboratory) | `labVitals` | Lab test results |
| `ServiceRequest` | `labTests` | Ordered lab tests |
| `Procedure` | `procedures` | Medical procedures |
| `FamilyMemberHistory` | `medicalHistory.familyHistory` | Family medical history |
| `AllergyIntolerance` | `medicalHistory.foodOtherAllergy`, `drugAllergy` | Allergies |
| `Observation` (social-history) | `medicalHistory.lifestyleHabits` | Lifestyle habits |
| `Appointment` | `followup` | Follow-up appointments |

## 🏗️ Module Structure

```
eka-emr-adaptar/
├── src/
│   ├── index.js                    # Main adapter entry point
│   ├── parsers/
│   │   ├── symptoms.js             # Parse symptoms from Observations
│   │   ├── diagnosis.js            # Parse diagnosis from Conditions
│   │   ├── medications.js          # Parse medications from MedicationRequests
│   │   ├── medicalHistory.js       # Parse complete medical history
│   │   ├── vitals.js               # Parse vital signs
│   │   ├── labTests.js             # Parse lab test orders
│   │   ├── labVitals.js            # Parse lab test results
│   │   ├── procedures.js           # Parse procedures
│   │   ├── injections.js           # Parse injections
│   │   ├── followup.js             # Parse follow-up appointments
│   │   └── prescriptionNotes.js    # Parse clinical notes
│   └── utils/
│       └── helpers.js              # Utility functions
├── examples/
│   └── convert.js                  # Example usage script
├── eka-emr-input.json              # Sample EKA EMR output
├── scribe_output.json              # Sample FHIR input
├── package.json
└── README.md
```

## 🔧 Parser Modules

Each parser module is responsible for converting a specific type of FHIR resource:

### Symptoms Parser
Converts symptom observations with properties like severity, duration, and laterality.

```javascript
const { parseSymptoms } = require('./parsers/symptoms');
const symptoms = parseSymptoms(fhirEntries);
```

### Diagnosis Parser
Extracts diagnoses with ICD-10 codes, clinical status, and severity.

```javascript
const { parseDiagnosis } = require('./parsers/diagnosis');
const diagnosis = parseDiagnosis(fhirEntries);
```

### Medications Parser
Processes medication requests with dosage, frequency, timing, and duration.

```javascript
const { parseMedications } = require('./parsers/medications');
const medications = parseMedications(fhirEntries);
```

### Medical History Parser
Comprehensive parser for patient history including:
- Medical conditions
- Current medications
- Family history
- Lifestyle habits
- Allergies
- Past procedures
- Travel history

```javascript
const { parseMedicalHistory } = require('./parsers/medicalHistory');
const medicalHistory = parseMedicalHistory(fhirEntries);
```

## 🛠️ Customization

### Adding Custom Parsers

To add support for additional FHIR resources:

1. Create a new parser in `src/parsers/`:

```javascript
// src/parsers/myCustomParser.js
const { generateEkaId, getResourcesByType } = require('../utils/helpers');

function parseCustomResource(entries) {
  const resources = getResourcesByType(entries, 'CustomResource');
  
  return resources.map((resource, index) => ({
    id: generateEkaId('custom'),
    // ... your mapping logic
  }));
}

module.exports = { parseCustomResource };
```

2. Import and use in `src/index.js`:

```javascript
const { parseCustomResource } = require('./parsers/myCustomParser');

// In convertFHIRToEkaEMR function:
ekaEMRInput.customField = parseCustomResource(entries);
```

### Extending Existing Parsers

You can extend parsers by modifying the mapping logic:

```javascript
// Example: Add custom ICD-10 code lookup
function parseDiagnosisWithCustomMapping(entries) {
  const diagnosis = parseDiagnosis(entries);
  
  return diagnosis.map(diag => {
    // Add custom ICD-10 mapping
    diag.icd10_code = lookupICD10Code(diag.name);
    return diag;
  });
}
```

## 📊 Data Mapping Examples

### Example 1: Symptom Mapping

**FHIR Input:**
```json
{
  "resourceType": "Observation",
  "category": [{"coding": [{"code": "symptom"}]}],
  "code": {"text": "Headache"},
  "effectiveDateTime": "2026-01-26T18:00:42Z",
  "component": [
    {
      "code": {"coding": [{"code": "246112005", "display": "Severity"}]},
      "valueCodeableConcept": {"text": "Moderate"}
    }
  ]
}
```

**EKA EMR Output:**
```json
{
  "id": "s-5958917470",
  "name": "Headache",
  "properties": {
    "pg-1541659976": {
      "name": "Since",
      "selection": [{"value": "2", "unit": "Days"}]
    },
    "pg-2869689919": {
      "name": "Severity",
      "selection": [{"value": "Moderate"}]
    }
  }
}
```

### Example 2: Medication Mapping

**FHIR Input:**
```json
{
  "resourceType": "MedicationRequest",
  "medication": {"concept": {"text": "Dolo 650 Tablet"}},
  "dosageInstruction": [{
    "text": "Take 1 tablet three times daily after meals",
    "timing": {"repeat": {"frequency": 3, "period": 1, "periodUnit": "d", "when": ["PC"]}},
    "doseAndRate": [{"doseQuantity": {"value": 1, "unit": "tablet"}}]
  }],
  "dispenseRequest": {"expectedSupplyDuration": {"value": 7, "unit": "d"}}
}
```

**EKA EMR Output:**
```json
{
  "id": "b-4117370658",
  "name": "Dolo 650 Tablet",
  "dose": {"value": "1", "unit": "tablet"},
  "frequency": {"custom": "3 times per d"},
  "timing": "After Meal",
  "duration": {"value": "7", "unit": "Days"}
}
```

## 🧪 Testing

Run the example conversion:

```bash
node examples/convert.js
```

This will:
1. Load `scribe_output.json` (FHIR Bundle)
2. Convert to EKA EMR format
3. Save output to `eka-emr-output.json`
4. Display conversion statistics

## 🔐 Error Handling

The adapter includes comprehensive error handling:

```javascript
try {
  const ekaEMRInput = convertFHIRToEkaEMR(fhirBundle);
} catch (error) {
  if (error.message.includes('Invalid FHIR Bundle')) {
    console.error('Input is not a valid FHIR Bundle');
  } else {
    console.error('Conversion error:', error);
  }
}
```

## 📝 medScribe Alliance Protocol

This adapter follows the [medScribe Alliance](https://github.com/medScribeAlliance/) protocol, which standardizes:

- **Input Format**: FHIR R4/R5 Bundle resources
- **Resource Types**: Standard FHIR resources for clinical data
- **Interoperability**: Works with any compliant scribe engine
- **Extensibility**: Easy to extend for new FHIR resources

### Supported Scribe Engines

Any scribe engine that outputs FHIR-compliant bundles can be integrated:

- Amazon HealthScribe
- Azure Health Bot
- Google Cloud Healthcare API
- Nuance Dragon Medical
- Custom AI-powered scribes
- Open-source scribe implementations

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Add tests for new functionality
4. Submit a pull request

## 📄 License

MIT License - see LICENSE file for details

## 🆘 Support

For issues and questions:
- Open an issue on GitHub
- Check existing documentation
- Review example implementations

## 🔗 Related Links

- [medScribe Alliance GitHub](https://github.com/medScribeAlliance/)
- [FHIR Documentation](https://www.hl7.org/fhir/)
- [EKA Care](https://www.eka.care/)

---

**Made with ❤️ for better healthcare interoperability**
