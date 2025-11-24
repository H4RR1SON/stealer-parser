# Documentation Index

Welcome to the Stealer Parser documentation. This index provides an overview of all available documentation.

## Quick Start

New to Stealer Parser? Start here:

1. **[Installation](#installation)** - Get up and running
2. **[Quick Start Guide](USAGE.md#getting-started)** - Basic usage examples
3. **[Basic Usage](USAGE.md#basic-usage)** - Common commands
4. **[File Structures](FILE_STRUCTURES.md)** - Understanding stealer logs

## Core Documentation

### User Guides

- **[Usage Guide](USAGE.md)** - Comprehensive usage documentation
  - Installation and setup
  - Basic and advanced usage
  - Output analysis
  - Use cases and examples
  - Troubleshooting
  - Best practices

### Technical Documentation

- **[Architecture](ARCHITECTURE.md)** - System architecture and design
  - Component overview
  - Data flow
  - Processing pipeline
  - Extension points
  - Technical specifications

- **[File Structures](FILE_STRUCTURES.md)** - Stealer log format reference
  - Common structure patterns
  - Stealer-specific formats (RedLine, Raccoon, StealC, etc.)
  - Field naming variations
  - Parsing challenges
  - Detection methods

### Reference

- **[Limitations](LIMITATIONS.md)** - Current limitations and known issues
  - Archive format limitations
  - Parsing limitations
  - Performance constraints
  - Security considerations
  - Tracking priority issues

- **[Improvements](IMPROVEMENTS.md)** - Recommended improvements
  - Priority 1: Critical reliability improvements
  - Priority 2: Rich CLI integration
  - Priority 3: Enhanced features
  - Priority 4: Advanced features
  - Implementation roadmap

- **[Roadmap](ROADMAP.md)** - Project development roadmap
  - Version history
  - Planned releases
  - Feature backlog
  - Long-term vision

### Development

- **[Contributing Guide](../CONTRIBUTING.md)** - How to contribute
  - Code standards
  - Development workflow
  - Testing requirements
  - Pull request process

- **[Grammar Files](grammar_passwords.txt)** - Parsing grammars
  - Password file grammar
  - System information grammar

## Installation

### Quick Install

```bash
# Clone repository with submodules
git clone --recurse-submodules https://github.com/H4RR1SON/stealer-parser
cd stealer-parser

# Install dependencies
pip install poetry
poetry install

# Activate environment
poetry shell

# Verify installation
stealer_parser --help
```

See [Usage Guide](USAGE.md#installation) for detailed installation instructions.

## Quick Examples

### Parse an Archive

```bash
# Basic parsing
stealer_parser logs.zip

# With password
stealer_parser protected.zip --password infected

# Verbose output
stealer_parser logs.zip -vv

# Custom output file
stealer_parser logs.zip -o results/analysis.json
```

### Analyze Results

```bash
# Count credentials
jq '[.systems_data[].credentials | length] | add' results.json

# Find specific domain
jq '.systems_data[].credentials[] | select(.domain == "gmail.com")' results.json

# List stealers found
jq -r '.systems_data[].credentials[0].stealer_name' results.json | sort -u
```

## Documentation by Role

### For Security Analysts

**Essential Reading:**
1. [Usage Guide](USAGE.md) - Day-to-day usage
2. [File Structures](FILE_STRUCTURES.md) - Understanding logs
3. [Use Cases](USAGE.md#use-cases) - Practical examples

**Recommended:**
- [Output Analysis](USAGE.md#output-analysis) - Working with results
- [Troubleshooting](USAGE.md#troubleshooting) - Solving common issues

### For Threat Intelligence Researchers

**Essential Reading:**
1. [File Structures](FILE_STRUCTURES.md) - Log format details
2. [Architecture](ARCHITECTURE.md) - How parsing works
3. [Use Cases](USAGE.md#use-cases) - CTI workflows

**Recommended:**
- [Limitations](LIMITATIONS.md) - Parser constraints
- [Improvements](IMPROVEMENTS.md) - Planned enhancements

### For Developers

**Essential Reading:**
1. [Architecture](ARCHITECTURE.md) - System design
2. [Contributing Guide](../CONTRIBUTING.md) - Development process
3. [Improvements](IMPROVEMENTS.md) - Enhancement opportunities

**Recommended:**
- [Roadmap](ROADMAP.md) - Development plan
- [Limitations](LIMITATIONS.md) - Known issues
- [Grammar Files](grammar_passwords.txt) - Parser specifications

### For System Administrators

**Essential Reading:**
1. [Installation](USAGE.md#installation) - Setup guide
2. [Best Practices](USAGE.md#best-practices) - Secure deployment
3. [Troubleshooting](USAGE.md#troubleshooting) - Problem solving

**Recommended:**
- [Advanced Usage](USAGE.md#advanced-usage) - Automation
- [Performance](LIMITATIONS.md#performance-limitations) - Optimization

## Topic Index

### By Feature

**Archive Support:**
- [Supported Formats](ARCHITECTURE.md#archive-handling)
- [Password Protection](USAGE.md#password-protected-archives)
- [Multi-part Archives](LIMITATIONS.md#multi-part-archives)

**Parsing:**
- [Grammar Specification](grammar_passwords.txt)
- [Parsing Engine](ARCHITECTURE.md#parsing-layer)
- [Error Handling](ARCHITECTURE.md#error-handling)

**Stealer Detection:**
- [Supported Stealers](FILE_STRUCTURES.md#overview)
- [Detection Methods](ARCHITECTURE.md#stealer-detection)
- [Adding New Stealers](IMPROVEMENTS.md#enhanced-stealer-detection)

**Output:**
- [JSON Format](USAGE.md#json-structure)
- [Analysis with jq](USAGE.md#query-results-with-jq)
- [Database Import](USAGE.md#import-into-database)
- [Multi-format Export](IMPROVEMENTS.md#multi-format-output)

**Performance:**
- [Current Performance](LIMITATIONS.md#performance-limitations)
- [Optimization Tips](USAGE.md#performance)
- [Parallel Processing](IMPROVEMENTS.md#parallel-processing)

### By Stealer Family

**RedLine Stealer:**
- [File Structure](FILE_STRUCTURES.md#redline-stealer)
- [Detection Signature](FILE_STRUCTURES.md#redline-stealer)
- [Field Format](FILE_STRUCTURES.md#password-format)

**Raccoon Stealer:**
- [File Structure](FILE_STRUCTURES.md#raccoon-stealer)
- [ASCII Art](FILE_STRUCTURES.md#ascii-art-banner)
- [Information Format](FILE_STRUCTURES.md#information-format)

**StealC:**
- [File Structure](FILE_STRUCTURES.md#stealc)
- [Banner](FILE_STRUCTURES.md#banner)
- [System Info](FILE_STRUCTURES.md#system-info-format)

**Other Stealers:**
- [Meta Stealer](FILE_STRUCTURES.md#meta-stealer)
- [LummaC2](FILE_STRUCTURES.md#lummac2)
- [Vidar Stealer](FILE_STRUCTURES.md#vidar-stealer)

### By Task

**Installation:**
- [Quick Install](#installation)
- [Detailed Setup](USAGE.md#installation)
- [Troubleshooting Install](USAGE.md#troubleshooting)

**Basic Operations:**
- [Parse Archive](USAGE.md#parse-a-single-archive)
- [View Results](USAGE.md#output-analysis)
- [Search Credentials](USAGE.md#query-results-with-jq)

**Analysis:**
- [Threat Intelligence](USAGE.md#1-threat-intelligence-analysis)
- [Credential Intelligence](USAGE.md#2-credential-intelligence)
- [Password Analysis](USAGE.md#3-password-analysis)
- [OSINT Pivoting](USAGE.md#4-osint-pivoting)

**Advanced:**
- [Batch Processing](USAGE.md#batch-processing)
- [Automation](USAGE.md#automation)
- [SIEM Integration](USAGE.md#integration-with-siem)
- [Custom Parsers](USAGE.md#custom-parsers)

**Troubleshooting:**
- [Common Issues](USAGE.md#common-issues)
- [Performance Issues](USAGE.md#performance-issues)
- [Getting Help](USAGE.md#getting-help)

## External Resources

### Academic & Research

- [RedLine Stealer Analysis](https://securityscorecard.com/research/redline-stealer)
- [Haris Qazi - Stealer Logs](https://www.harisqazi.com/open-source-intelligence/breach-data/stealer-logs/)
- [Intel Techniques - Breach Data](https://inteltechniques.com/blog/2022/07/06/new-breach-data-lesson-ii-stealer-logs/)
- [OSINT Team Newsletter](https://www.osintteam.com/the-practical-osint-newsletter/issue-4/)

### Industry Reports

- [Lexfo - Infostealer Parser](https://blog.lexfo.fr/infostealer-parser.html)
- [ZeroFox - Stealer Logs Intro](https://www.zerofox.com/blog/an-introduction-to-stealer-logs/)
- [Accenture - Info Stealer Malware](https://www.accenture.com/us-en/blogs/security/information-stealer-malware-on-dark-web)

### Related Tools

- [Lexfo stealer-parser](https://github.com/lexfo/stealer-parser) - Original inspiration
- [thredb sysinfo-parser](https://github.com/thredb/sysinfo-parser) - System info parser
- [nak0823 RParseX](https://github.com/nak0823/RParseX) - RedLine parser
- [milxss universal_stealer_log_parser](https://github.com/milxss/universal_stealer_log_parser) - Universal parser
- [This Repository](https://github.com/H4RR1SON/stealer-parser) - Fork with enhancements

## Getting Help

### Documentation

Can't find what you're looking for? Try:
1. Use the search function in your browser (Ctrl+F / Cmd+F)
2. Check the [Usage Guide](USAGE.md) - Most comprehensive
3. Review [Troubleshooting](USAGE.md#troubleshooting)

### Support Channels

- **GitHub Issues:** [Report bugs or request features](https://github.com/H4RR1SON/stealer-parser/issues)
- **Discussions:** [Community discussions](https://github.com/H4RR1SON/stealer-parser/discussions)
- **Contributing:** [Contribute code or docs](../CONTRIBUTING.md)

### Feedback

Help us improve the documentation:
- Report unclear sections
- Suggest additional examples
- Contribute corrections
- Add use cases

## Documentation Status

### Coverage

| Area | Status | Last Updated |
|------|--------|--------------|
| Installation | ✅ Complete | Nov 2024 |
| Basic Usage | ✅ Complete | Nov 2024 |
| Advanced Usage | ✅ Complete | Nov 2024 |
| Architecture | ✅ Complete | Nov 2024 |
| File Formats | ✅ Complete | Nov 2024 |
| Limitations | ✅ Complete | Nov 2024 |
| Improvements | ✅ Complete | Nov 2024 |
| Roadmap | ✅ Complete | Nov 2024 |
| API Reference | 🔄 Planned | TBD |
| Video Tutorials | 🔄 Planned | TBD |

### Legend
- ✅ Complete
- 🔄 In Progress / Planned
- ⚠️ Needs Update
- ❌ Missing

## Contributing to Documentation

See [Contributing Guide](../CONTRIBUTING.md) for:
- Documentation standards
- Style guide
- Review process
- How to submit changes

**Quick tips:**
- Use clear, concise language
- Include code examples
- Add screenshots where helpful
- Link to related sections
- Test all commands

## License

This documentation is part of the Stealer Parser project, licensed under [Apache License 2.0](../LICENSE.md).

---

**Last Updated:** November 2024  
**Version:** 1.0.0  
**Maintainer:** H4RR1SON

For the latest documentation, visit: https://github.com/H4RR1SON/stealer-parser
