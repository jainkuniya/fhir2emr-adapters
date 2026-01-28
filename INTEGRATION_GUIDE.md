# Integration Guide - EKA EMR Adapter

## Overview

This guide explains how to integrate the EKA EMR Adapter into your healthcare workflow using the medScribe Alliance protocol.

## Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    CLINICAL WORKFLOW                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Doctor-Patient                                               │
│  Conversation    ──────────┐                                  │
│                           │                                  │
│                           ▼                                  │
│                 ┌─────────────────┐                          │
│                 │  Voice/Text     │                          │
│                 │  Capture        │                          │
│                 └────────┬────────┘                          │
│                          │                                   │
│                          ▼                                   │
│         ╔════════════════════════════════╗                   │
│         ║  SCRIBE ENGINE                 ║                   │
│         ║  (medScribe Alliance Protocol) ║                   │
│         ╠════════════════════════════════╣                   │
│         ║  - Speech Recognition          ║                   │
│         ║  - NLP Processing              ║                   │
│         ║  - Clinical Entity Extraction  ║                   │
│         ║  - FHIR Bundle Generation      ║                   │
│         ╚═════════════┬══════════════════╝                   │
│                       │                                      │
│                       │ FHIR R4/R5 Bundle                    │
│                       │ (JSON)                               │
│                       ▼                                      │
│         ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓                      │
│         ┃  EKA EMR ADAPTER             ┃                      │
│         ┃  (This Module)               ┃                      │
│         ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫                      │
│         ┃  Input:  FHIR Bundle         ┃                      │
│         ┃  Output: EKA EMR Format      ┃                      │
│         ┃                              ┃                      │
│         ┃  Parsers:                    ┃                      │
│         ┃  • Symptoms                  ┃                      │
│         ┃  • Diagnosis                 ┃                      │
│         ┃  • Medications               ┃                      │
│         ┃  • Medical History           ┃                      │
│         ┃  • Vitals                    ┃                      │
│         ┃  • Lab Tests                 ┃                      │
│         ┃  • Procedures                ┃                      │
│         ┗━━━━━━━━━━━┯━━━━━━━━━━━━━━━━━┛                      │
│                     │                                        │
│                     │ EKA Care Format (JSON)                 │
│                     ▼                                        │
│         ╔═══════════════════════════════╗                    │
│         ║  EKA CARE EMR SYSTEM          ║                    │
│         ╠═══════════════════════════════╣                    │
│         ║  • Patient Records            ║                    │
│         ║  • Prescription Management    ║                    │
│         ║  • Clinical Workflows         ║                    │
│         ║  • Analytics & Reporting      ║                    │
│         ╚═══════════════════════════════╝                    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

## Integration Scenarios

### Scenario 1: Real-time Integration

**Use Case**: Convert scribe output immediately after consultation

```javascript
const express = require('express');
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');

const app = express();
app.use(express.json());

// Webhook endpoint for scribe engine
app.post('/api/scribe/webhook', async (req, res) => {
  try {
    // Receive FHIR Bundle from scribe
    const fhirBundle = req.body;
    
    // Convert to EKA format
    const ekaInput = convertFHIRToEkaEMR(fhirBundle);
    
    // Send to EKA Care API
    const response = await fetch('https://api.eka.care/v1/emr/records', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.EKA_API_KEY}`
      },
      body: JSON.stringify(ekaInput)
    });
    
    if (response.ok) {
      res.json({ success: true, message: 'Data synced to EKA Care' });
    } else {
      res.status(500).json({ success: false, error: 'EKA API error' });
    }
  } catch (error) {
    console.error('Conversion error:', error);
    res.status(500).json({ success: false, error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Adapter service running on port 3000');
});
```

### Scenario 2: Batch Processing

**Use Case**: Process multiple scribe outputs from a queue

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');
const AWS = require('aws-sdk');

const sqs = new AWS.SQS();

async function processQueue() {
  while (true) {
    // Receive messages from SQS
    const messages = await sqs.receiveMessage({
      QueueUrl: process.env.SQS_QUEUE_URL,
      MaxNumberOfMessages: 10,
      WaitTimeSeconds: 20
    }).promise();
    
    if (messages.Messages) {
      for (const message of messages.Messages) {
        try {
          const fhirBundle = JSON.parse(message.Body);
          
          // Convert
          const ekaInput = convertFHIRToEkaEMR(fhirBundle);
          
          // Process
          await sendToEkaCare(ekaInput);
          
          // Delete message
          await sqs.deleteMessage({
            QueueUrl: process.env.SQS_QUEUE_URL,
            ReceiptHandle: message.ReceiptHandle
          }).promise();
          
          console.log('Processed message:', message.MessageId);
        } catch (error) {
          console.error('Processing error:', error);
          // Handle error (DLQ, retry, etc.)
        }
      }
    }
  }
}

processQueue();
```

### Scenario 3: Serverless Function (AWS Lambda)

**Use Case**: Trigger conversion on S3 upload

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');
const AWS = require('aws-sdk');

const s3 = new AWS.S3();

exports.handler = async (event) => {
  for (const record of event.Records) {
    const bucket = record.s3.bucket.name;
    const key = record.s3.object.key;
    
    try {
      // Get FHIR Bundle from S3
      const s3Object = await s3.getObject({ Bucket: bucket, Key: key }).promise();
      const fhirBundle = JSON.parse(s3Object.Body.toString());
      
      // Convert
      const ekaInput = convertFHIRToEkaEMR(fhirBundle);
      
      // Save converted output
      await s3.putObject({
        Bucket: process.env.OUTPUT_BUCKET,
        Key: key.replace('.json', '-eka.json'),
        Body: JSON.stringify(ekaInput),
        ContentType: 'application/json'
      }).promise();
      
      console.log('Converted:', key);
    } catch (error) {
      console.error('Lambda error:', error);
      throw error;
    }
  }
  
  return { statusCode: 200, body: 'Success' };
};
```

## Configuration

### Environment Variables

```bash
# EKA Care API
EKA_API_KEY=your_api_key_here
EKA_API_URL=https://api.eka.care/v1

# Scribe Engine
SCRIBE_WEBHOOK_SECRET=your_webhook_secret

# AWS (if using)
AWS_REGION=us-east-1
SQS_QUEUE_URL=https://sqs.us-east-1.amazonaws.com/...
OUTPUT_BUCKET=eka-emr-outputs
```

### Docker Deployment

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY src/ ./src/
COPY examples/ ./examples/

EXPOSE 3000

CMD ["node", "examples/server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  eka-adapter:
    build: .
    ports:
      - "3000:3000"
    environment:
      - EKA_API_KEY=${EKA_API_KEY}
      - EKA_API_URL=${EKA_API_URL}
    restart: unless-stopped
```

## Testing

### Unit Testing

```javascript
const { convertFHIRToEkaEMR } = require('eka-emr-adapter');
const assert = require('assert');

describe('EKA EMR Adapter', () => {
  it('should convert FHIR Bundle to EKA format', () => {
    const fhirBundle = {
      resourceType: 'Bundle',
      type: 'collection',
      entry: [
        {
          resource: {
            resourceType: 'Observation',
            category: [{ coding: [{ code: 'symptom' }] }],
            code: { text: 'Fever' }
          }
        }
      ]
    };
    
    const result = convertFHIRToEkaEMR(fhirBundle);
    
    assert.equal(result.symptoms.length, 1);
    assert.equal(result.symptoms[0].name, 'Fever');
  });
  
  it('should throw error for invalid bundle', () => {
    assert.throws(() => {
      convertFHIRToEkaEMR({ invalid: 'data' });
    }, /Invalid FHIR Bundle/);
  });
});
```

### Integration Testing

```javascript
const request = require('supertest');
const app = require('./server');

describe('API Integration', () => {
  it('should accept webhook and convert', async () => {
    const fhirBundle = require('./test/fixtures/sample-bundle.json');
    
    const response = await request(app)
      .post('/api/scribe/webhook')
      .send(fhirBundle)
      .expect(200);
    
    assert.equal(response.body.success, true);
  });
});
```

## Monitoring & Logging

### Structured Logging

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// Log conversion
logger.info('Conversion started', {
  bundleId: fhirBundle.id,
  entryCount: fhirBundle.entry.length
});

const ekaInput = convertFHIRToEkaEMR(fhirBundle);

logger.info('Conversion completed', {
  bundleId: fhirBundle.id,
  symptomsCount: ekaInput.symptoms.length,
  diagnosisCount: ekaInput.diagnosis.length
});
```

### Metrics

```javascript
const prometheus = require('prom-client');

const conversionDuration = new prometheus.Histogram({
  name: 'eka_conversion_duration_seconds',
  help: 'Duration of FHIR to EKA conversion',
  buckets: [0.1, 0.5, 1, 2, 5]
});

const conversionErrors = new prometheus.Counter({
  name: 'eka_conversion_errors_total',
  help: 'Total number of conversion errors'
});

// Usage
const end = conversionDuration.startTimer();
try {
  const ekaInput = convertFHIRToEkaEMR(fhirBundle);
  end();
} catch (error) {
  conversionErrors.inc();
  throw error;
}
```

## Error Handling

### Graceful Error Handling

```javascript
function convertWithErrorHandling(fhirBundle) {
  try {
    return convertFHIRToEkaEMR(fhirBundle);
  } catch (error) {
    if (error.message.includes('Invalid FHIR Bundle')) {
      // Handle invalid bundle
      logger.error('Invalid FHIR Bundle received', { error: error.message });
      return null;
    } else if (error.message.includes('parsing')) {
      // Handle parsing errors
      logger.error('Parsing error', { error: error.message });
      return createPartialOutput(fhirBundle);
    } else {
      // Unknown error
      logger.error('Unexpected error', { error: error.stack });
      throw error;
    }
  }
}
```

## Best Practices

1. **Validation**: Always validate FHIR Bundle before conversion
2. **Logging**: Log conversions for audit trails
3. **Monitoring**: Track conversion success/failure rates
4. **Versioning**: Version your adapter to track changes
5. **Testing**: Test with real scribe outputs before production
6. **Security**: Use HTTPS and API keys for EKA Care API
7. **Rate Limiting**: Implement rate limiting for API calls
8. **Caching**: Cache ICD-10 lookups and other mappings
9. **Async Processing**: Use queues for high-volume scenarios
10. **Error Handling**: Implement retry logic with exponential backoff

## Support & Troubleshooting

### Common Issues

**Issue**: Conversion takes too long
- **Solution**: Process large bundles asynchronously, use worker threads

**Issue**: Missing data in output
- **Solution**: Check FHIR resource categories, review parser logic

**Issue**: API authentication fails
- **Solution**: Verify EKA_API_KEY is correct and not expired

**Issue**: Memory issues with large bundles
- **Solution**: Implement streaming parsing for very large bundles

## Resources

- [medScribe Alliance Protocol](https://github.com/medScribeAlliance/)
- [FHIR R4 Specification](https://www.hl7.org/fhir/R4/)
- [EKA Care API Documentation](https://www.eka.care/developers)
- [Adapter Documentation](README.md)
- [Quick Reference](QUICK_REFERENCE.md)

---

**Questions or Issues?** Open an issue on GitHub or contact support.
