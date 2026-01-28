# Quick Reference Guide

## Installation & Setup

```bash
# Install dependencies
npm install

# Run example conversion
npm run example
```

## Basic API

### Convert FHIR to EKA EMR

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');

// Synchronous
const ekaInput = convertFHIRToEkaEMR(fhirBundle);

// Asynchronous
const ekaInput = await convertFHIRToEkaEMRAsync(fhirBundle);
```

## FHIR Resource → EKA EMR Mapping

### Symptoms
- **FHIR**: `Observation` with `category: symptom`
- **EKA**: `symptoms[]`
- **Includes**: Name, severity, duration, laterality

### Diagnosis
- **FHIR**: `Condition` with `category: encounter-diagnosis`
- **EKA**: `diagnosis[]`
- **Includes**: Name, ICD-10 code, clinical status, severity

### Medications
- **FHIR**: `MedicationRequest`
- **EKA**: `medications[]`
- **Includes**: Name, dose, frequency, timing, duration

### Medical History
- **FHIR**: Various (Condition, MedicationStatement, FamilyMemberHistory, etc.)
- **EKA**: `medicalHistory.patientHistory`
- **Includes**: 
  - Patient medical conditions
  - Current medications
  - Family history
  - Lifestyle habits
  - Allergies (food and drug)
  - Past procedures
  - Travel history

### Vitals
- **FHIR**: `Observation` with `category: vital-signs`
- **EKA**: `medicalHistory.vitals[]`
- **Includes**: Blood pressure, pulse, temperature, etc.

### Lab Tests
- **FHIR**: `ServiceRequest` with `category: laboratory`
- **EKA**: `labTests[]`
- **Includes**: Test name, booking status

### Lab Results
- **FHIR**: `Observation` with `category: laboratory`
- **EKA**: `labVitals[]`
- **Includes**: Test name, value, unit, interpretation

### Procedures
- **FHIR**: `Procedure`
- **EKA**: `procedures[]` and `medicalHistory.patientHistory.pastProcedures[]`
- **Includes**: Name, date performed

## Utility Functions

```javascript
const helpers = require('./src/utils/helpers');

// Generate EKA ID
const id = helpers.generateEkaId('s'); // s-1234567890

// Get resources by type
const observations = helpers.getResourcesByType(entries, 'Observation');

// Extract display text
const text = helpers.getDisplayFromConcept(codeableConcept);

// Calculate duration from date
const duration = helpers.calculateDuration(dateString);
// Returns: { value: "3", unit: "Days" }

// Map FHIR severity to EKA
const severity = helpers.mapSeverity(severityConcept);
// Returns: "Mild" | "Moderate" | "Severe"
```

## Custom Parser Example

```javascript
// Create custom parser in src/parsers/customParser.js
const { generateEkaId, getResourcesByType } = require('../utils/helpers');

function parseCustomData(entries) {
  const resources = getResourcesByType(entries, 'CustomResource');
  
  return resources.map((resource, index) => ({
    id: generateEkaId('custom'),
    name: resource.name || 'Unknown',
    // Add your custom mapping
  }));
}

module.exports = { parseCustomData };
```

## Integration with medScribe Alliance

The adapter follows the [medScribe Alliance protocol](https://github.com/medScribeAlliance/):

1. **Scribe Engine** outputs FHIR Bundle
2. **Adapter** converts FHIR → EKA format
3. **EKA Care EMR** receives formatted data

### Compatible Scribe Engines
- Amazon HealthScribe
- Azure Health Bot
- Google Cloud Healthcare API
- Any FHIR R4/R5 compliant scribe

## Troubleshooting

### "Invalid FHIR Bundle" Error
- Verify `resourceType` is "Bundle"
- Check bundle structure is valid FHIR

### Missing Data
- Ensure FHIR resources have required fields
- Check category codes match expected values

### Incorrect Mapping
- Review parser logic in `src/parsers/`
- Add custom mapping logic for specific fields

## Performance Tips

- Process large bundles in chunks
- Use async version for file I/O operations
- Cache repeated lookups (ICD-10 codes, etc.)

## File Structure

```
eka-emr-adaptar/
├── src/
│   ├── index.js              # Main entry point
│   ├── parsers/              # Resource-specific parsers
│   │   ├── symptoms.js
│   │   ├── diagnosis.js
│   │   ├── medications.js
│   │   ├── medicalHistory.js
│   │   ├── vitals.js
│   │   ├── labTests.js
│   │   ├── labVitals.js
│   │   ├── procedures.js
│   │   ├── injections.js
│   │   ├── followup.js
│   │   └── prescriptionNotes.js
│   └── utils/
│       └── helpers.js        # Utility functions
├── examples/
│   └── convert.js            # Example usage
├── scribe_output.json        # Sample FHIR input
├── eka-emr-input.json        # Sample EKA output
├── package.json
└── README.md
```

## Common Use Cases

### Use Case 1: Real-time Scribe Integration

```javascript
const express = require('express');
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');

app.post('/scribe-webhook', async (req, res) => {
  const fhirBundle = req.body;
  const ekaInput = convertFHIRToEkaEMR(fhirBundle);
  
  // Send to EKA Care API
  await sendToEkaCare(ekaInput);
  
  res.json({ success: true });
});
```

### Use Case 2: Batch Processing

```javascript
const fs = require('fs');
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');

async function processBatch(files) {
  for (const file of files) {
    const fhirBundle = JSON.parse(fs.readFileSync(file));
    const ekaInput = convertFHIRToEkaEMR(fhirBundle);
    
    // Process each converted file
    await processEkaInput(ekaInput);
  }
}
```

### Use Case 3: Validation Pipeline

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');

function validateAndConvert(fhirBundle) {
  // Pre-conversion validation
  if (!fhirBundle || fhirBundle.resourceType !== 'Bundle') {
    throw new Error('Invalid input');
  }
  
  // Convert
  const ekaInput = convertFHIRToEkaEMR(fhirBundle);
  
  // Post-conversion validation
  validateEkaFormat(ekaInput);
  
  return ekaInput;
}
```

## Support & Contributing

- **Issues**: Report on GitHub
- **Documentation**: See README.md
- **Contributing**: Fork, modify, submit PR

---

**Quick Links**:
- [Full Documentation](README.md)
- [medScribe Alliance](https://github.com/medScribeAlliance/)
- [FHIR Spec](https://www.hl7.org/fhir/)
