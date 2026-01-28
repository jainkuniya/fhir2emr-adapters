# 📚 EKA EMR Adapter - Documentation Index

Welcome to the EKA EMR Adapter documentation! This adapter converts FHIR-based scribe output to EKA Care EMR format, following the [medScribe Alliance protocol](https://github.com/medScribeAlliance/).

## 🚀 Quick Start

```bash
# Install
npm install

# Run example
npm run example
```

## 📖 Documentation Structure

### 🎯 Getting Started

1. **[README.md](README.md)** - Start here!
   - Overview and benefits
   - Architecture diagram
   - Installation guide
   - Basic usage examples
   - Supported FHIR resources
   - Module structure
   - Data mapping examples
   - Customization guide

### 📘 Reference Documentation

2. **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - API Reference
   - Installation & setup
   - Basic API usage
   - FHIR → EKA mapping table
   - Utility functions reference
   - Custom parser examples
   - Troubleshooting
   - Common use cases

3. **[ARCHITECTURE.md](ARCHITECTURE.md)** - System Design
   - System overview diagrams
   - Data flow visualization
   - Parser module architecture
   - Integration patterns
   - Module dependencies
   - Deployment architectures

### 🔧 Integration Guides

4. **[INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)** - Production Integration
   - Real-time webhook integration
   - Batch processing setup
   - Serverless (AWS Lambda) deployment
   - Docker deployment
   - Configuration management
   - Error handling strategies
   - Monitoring & logging
   - Testing approaches
   - Best practices

### 📊 Project Information

5. **[PROJECT_SUMMARY.md](PROJECT_SUMMARY.md)** - Project Overview
   - What was built
   - Project structure
   - Key features
   - Test results
   - Technical highlights
   - Use cases covered

6. **[LICENSE](LICENSE)** - MIT License
   - Open source license terms

## 🎓 Learning Path

### For First-Time Users
1. Read [README.md](README.md) → Overview and basic usage
2. Run example: `npm run example`
3. Check [QUICK_REFERENCE.md](QUICK_REFERENCE.md) → API usage
4. Review sample files: `scribe_output.json` and `eka-emr-output.json`

### For Developers Integrating
1. Read [README.md](README.md) → Understanding the adapter
2. Study [ARCHITECTURE.md](ARCHITECTURE.md) → System design
3. Follow [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) → Implementation
4. Reference [QUICK_REFERENCE.md](QUICK_REFERENCE.md) → API details

### For Advanced Users
1. Review parser modules in `src/parsers/`
2. Study utility functions in `src/utils/helpers.js`
3. Explore [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) → Advanced scenarios
4. Customize parsers for specific needs

## 📂 Code Organization

```
eka-emr-adaptar/
├── 📄 Documentation
│   ├── README.md                 # Main documentation
│   ├── QUICK_REFERENCE.md        # API reference
│   ├── INTEGRATION_GUIDE.md      # Integration scenarios
│   ├── ARCHITECTURE.md           # System diagrams
│   ├── PROJECT_SUMMARY.md        # Project overview
│   └── INDEX.md                  # This file
│
├── 🔧 Source Code
│   └── src/
│       ├── index.js              # Main entry point
│       ├── parsers/              # Parser modules (11 files)
│       │   ├── symptoms.js
│       │   ├── diagnosis.js
│       │   ├── medications.js
│       │   ├── medicalHistory.js
│       │   ├── vitals.js
│       │   ├── labTests.js
│       │   ├── labVitals.js
│       │   ├── procedures.js
│       │   ├── injections.js
│       │   ├── followup.js
│       │   └── prescriptionNotes.js
│       └── utils/
│           └── helpers.js        # Utility functions
│
├── 📝 Examples
│   └── examples/
│       └── convert.js            # Example script with stats
│
├── 📊 Sample Data
│   ├── scribe_output.json        # Sample FHIR input
│   ├── eka-emr-input.json        # Reference EKA output
│   └── eka-emr-output.json       # Generated output
│
└── 📦 Configuration
    ├── package.json              # NPM configuration
    ├── .gitignore                # Git ignore rules
    └── LICENSE                   # MIT license
```

## 🔍 Find What You Need

### "I want to..."

#### ...understand what this does
→ Read [README.md](README.md) - Overview section

#### ...see how it works
→ Check [ARCHITECTURE.md](ARCHITECTURE.md) - Data flow diagram

#### ...use the adapter in my code
→ See [QUICK_REFERENCE.md](QUICK_REFERENCE.md) - Basic API section

#### ...integrate with my scribe engine
→ Follow [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) - Real-time integration

#### ...deploy to production
→ Read [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) - Deployment section

#### ...customize the parsers
→ See [README.md](README.md) - Customization section

#### ...understand FHIR mapping
→ Check [QUICK_REFERENCE.md](QUICK_REFERENCE.md) - Mapping table

#### ...troubleshoot an issue
→ See [QUICK_REFERENCE.md](QUICK_REFERENCE.md) - Troubleshooting section

#### ...contribute to the project
→ Read [README.md](README.md) - Contributing section

## 📚 External Resources

- **[medScribe Alliance Protocol](https://github.com/medScribeAlliance/)**
  - Standard protocol for scribe engines
  - FHIR-based output specification

- **[FHIR R4 Documentation](https://www.hl7.org/fhir/R4/)**
  - Official FHIR specification
  - Resource definitions

- **[EKA Care](https://www.eka.care/)**
  - EKA Care EMR system

## 🎯 Key Concepts

### FHIR Bundle
A FHIR Bundle is a collection of FHIR resources (like Observations, Conditions, etc.) grouped together. The scribe engine outputs this standard format.

### Parser Module
Each parser module handles a specific type of medical data (symptoms, diagnosis, etc.) and converts it from FHIR to EKA format.

### medScribe Alliance
A protocol/standard that ensures different scribe engines can work with different EMR systems through a common interface (FHIR).

### EKA EMR Format
The proprietary JSON format used by EKA Care EMR system for accepting clinical data.

## 🤝 Support

- **GitHub Issues**: Report bugs or request features
- **Documentation**: Check this index for relevant guides
- **Examples**: Review `examples/convert.js` for working code

## ✅ Quick Checklist

Before using the adapter in production:

- [ ] Read the main [README.md](README.md)
- [ ] Run the example: `npm run example`
- [ ] Review [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md)
- [ ] Test with your actual FHIR data
- [ ] Implement error handling
- [ ] Set up monitoring/logging
- [ ] Configure environment variables
- [ ] Review security considerations

## 📞 Getting Help

1. **Check Documentation**: Use this index to find relevant docs
2. **Run Examples**: Test the example script
3. **Review Code**: Look at parser modules in `src/parsers/`
4. **Search Issues**: Check if your question was answered
5. **Ask Questions**: Open a GitHub issue

---

## Navigation Quick Links

| Document | Purpose | Audience |
|----------|---------|----------|
| [README.md](README.md) | Main documentation, getting started | Everyone |
| [QUICK_REFERENCE.md](QUICK_REFERENCE.md) | API reference, quick lookup | Developers |
| [ARCHITECTURE.md](ARCHITECTURE.md) | System design, diagrams | Architects, Developers |
| [INTEGRATION_GUIDE.md](INTEGRATION_GUIDE.md) | Production integration | DevOps, Developers |
| [PROJECT_SUMMARY.md](PROJECT_SUMMARY.md) | Project overview | Stakeholders, Managers |

---

**Happy Integrating! 🚀**

For the latest updates, visit the [GitHub repository](https://github.com/medScribeAlliance/eka-emr-adapter).
