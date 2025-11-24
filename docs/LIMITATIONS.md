# Current Limitations and Known Issues

This document details the current limitations of the stealer parser, known issues, and planned improvements.

## Archive Format Limitations

### Multi-part Archives
**Status:** ❌ Not Supported

**Description:** Multi-part ZIP archives (e.g., `archive.zip`, `archive.z01`, `archive.z02`) are not currently handled.

**Impact:** Large log collections distributed across multiple archive files cannot be processed.

**Workaround:** Manually combine archive parts before processing.

**Recommended Fix:** Implement multi-part archive detection and assembly before extraction.

### Encrypted Archives
**Status:** ⚠️ Partial Support

**Description:** Password-protected archives are supported via `-p` flag, but only single-password archives work.

**Limitations:**
- No support for multiple passwords per archive
- No interactive password prompting
- No password list/dictionary support

**Workaround:** Extract manually if password is unknown or multiple passwords needed.

### Nested Archives
**Status:** ❌ Not Supported

**Description:** Archives within archives are not automatically extracted and processed.

**Impact:** Some log distributions use nested compression (e.g., `.zip` inside a `.rar`).

**Workaround:** Extract outer archive first, then process inner archives separately.

**Recommended Fix:** Recursive archive processing with depth limits.

## Parsing Limitations

### Grammar Coverage
**Status:** ⚠️ Incomplete

**Description:** Current grammars cover common patterns but not all variants.

**Known Issues:**
- Non-standard field names may not be recognized
- Heavily obfuscated or encoded data may fail to parse
- Some stealer families use proprietary formats not yet supported

**Impact:** Some valid log files are skipped or partially parsed.

**Metrics:** Based on testing, estimated 85-90% coverage of common stealer formats.

### Character Encoding
**Status:** ⚠️ Limited

**Description:** Parser assumes UTF-8 encoding with fallback to Latin-1.

**Known Issues:**
- Files with mixed encodings may have corrupted characters
- Some Eastern European/Cyrillic text may not display correctly
- Emoji and special Unicode characters may cause issues

**Error Example:**
```
UnicodeDecodeError: 'utf-8' codec can't decode byte 0xff in position 123
```

**Recommended Fix:** Implement charset detection (e.g., using `chardet` library).

### Base64 Decoding
**Status:** ⚠️ Best Effort

**Description:** Multiline password entries are assumed to be base64-encoded.

**Limitations:**
- Invalid base64 is silently ignored
- Non-base64 multiline passwords may be corrupted
- No detection of other encoding schemes (hex, custom)

### File Size Limits
**Status:** ⚠️ Implicit

**Description:** Very large files (>100MB) may cause memory issues.

**Impact:** Parser loads entire file contents into memory for processing.

**Recommended Fix:** Implement streaming parser for large files.

## Stealer Detection Limitations

### Coverage
**Status:** ⚠️ Limited to Known Families

**Current Support:**
- RedLine Stealer
- StealC
- Raccoon Stealer
- Meta Stealer
- LummaC2
- DcRat

**Missing:**
- Vidar Stealer
- Arkei Stealer
- Azorult
- Predator
- Mars Stealer
- Titan Stealer
- Nexus Stealer
- Rhadamanthys
- Atomic (AMOS)
- Many others...

**Impact:** Logs from unsupported stealers may not be properly attributed.

**Workaround:** Generic parsing still works; only stealer attribution is missing.

### Detection Accuracy
**Status:** ⚠️ Heuristic-Based

**Method:** Regex and ASCII art matching.

**Limitations:**
- Modified/customized stealers may not be detected
- Aggregators often strip ASCII art banners
- Generic logs with no attribution remain unidentified

**False Positives:** Low (high specificity)
**False Negatives:** Moderate (may miss customized/stripped logs)

## Output Format Limitations

### JSON Only
**Status:** ⚠️ Single Format

**Description:** Output is currently JSON only.

**Missing Formats:**
- CSV (for spreadsheet analysis)
- XML
- SQLite database
- STIX/TAXII (threat intelligence formats)
- Custom templates

**Impact:** Additional post-processing needed for some workflows.

**Recommended Fix:** Add format plugins or conversion utilities.

### No Incremental Output
**Status:** ❌ Batch Only

**Description:** All results are written at the end of processing.

**Impact:** 
- No progress visibility during long operations
- Memory usage grows with archive size
- Crash/interruption = complete data loss

**Recommended Fix:** Implement streaming JSON output with progress indicators.

## Performance Limitations

### Single-Threaded Processing
**Status:** ⚠️ No Parallelization

**Description:** All processing happens in a single thread.

**Impact:** Multi-core systems are underutilized.

**Typical Speed:** ~100-500 files/second (varies by file size and complexity).

**Recommended Fix:** Parallel processing of system directories.

### Memory Usage
**Status:** ⚠️ Linear Growth

**Description:** Memory usage grows with number of credentials and systems.

**Estimate:** ~1MB per 1000 credentials (varies by data complexity).

**Risk:** Large archives (10k+ systems) may require significant RAM.

**Recommended Fix:** Streaming output and data model optimization.

### Archive Extraction
**Status:** ⚠️ No Caching

**Description:** Archives are fully extracted into memory on each run.

**Impact:** Re-processing same archive is not optimized.

**Recommended Fix:** Optional cache directory for extracted files.

## User Experience Limitations

### CLI Only
**Status:** ⚠️ No GUI

**Description:** Command-line interface only.

**Impact:** Less accessible to non-technical users.

**Workaround:** Use command-line wrappers or scripts.

**Recommended Fix:** 
- Add Rich TUI (Text User Interface)
- Consider web interface for remote analysis
- Desktop GUI option

### Limited Progress Feedback
**Status:** ⚠️ Minimal Output

**Description:** Progress information is limited even with verbose mode.

**Missing:**
- Progress bar for large archives
- Estimated time remaining
- Real-time statistics (files processed, credentials found)
- Visual indicators

**Recommended Fix:** Integrate Rich library for better console output.

### Error Messages
**Status:** ⚠️ Technical

**Description:** Error messages are developer-oriented, not user-friendly.

**Example:**
```
LexError: Illegal character '�' at line 42
```

**Better Example:**
```
❌ Parse Error in file 'passwords.txt' (line 42)
   Unable to recognize character. File may be corrupted or in unsupported format.
   
   Suggestion: Try opening the file manually to check encoding.
```

**Recommended Fix:** User-friendly error messages with actionable suggestions.

### No Configuration File
**Status:** ❌ Not Supported

**Description:** All options must be specified via command-line flags.

**Missing:**
- Configuration file support (.toml, .yaml, .json)
- Profiles for different use cases
- Default settings persistence

**Recommended Fix:** Add config file support with `--config` flag.

## Data Extraction Limitations

### Incomplete Credential Metadata
**Status:** ⚠️ Basic Fields Only

**Description:** Only standard fields are extracted.

**Missing:**
- Browser profile names
- Password creation/modification dates
- Form field names (beyond username/password)
- Cookie expiration dates
- Session token metadata
- Credit card information
- Autofill data (addresses, phone numbers)

**Impact:** Some OSINT use cases require manual extraction.

### System Information Gaps
**Status:** ⚠️ Variable Coverage

**Description:** System information extraction depends on log format.

**Commonly Missing:**
- Installed software list
- Running processes
- Browser extensions
- Network adapters
- Display resolution
- Keyboard layout
- Time zone
- Anti-virus software
- Cryptocurrency wallet addresses

**Recommended Fix:** Extend system parser grammar for additional fields.

### No Relationship Mapping
**Status:** ❌ Flat Structure

**Description:** Cross-references between data are not automatically created.

**Missing:**
- Same password used across sites
- Related email addresses
- Username patterns
- Temporal correlation
- Geographic clustering

**Impact:** Analysts must manually correlate data for pivot operations.

**Recommended Fix:** Add optional post-processing analysis module.

## Compatibility Limitations

### Operating System
**Status:** ✅ Cross-Platform (mostly)

**Notes:**
- RAR extraction requires `unrar` binary (platform-specific)
- Path separators may cause issues on Windows
- Some dependencies may have OS-specific requirements

### Python Version
**Status:** ⚠️ 3.10+ Required

**Description:** Uses modern Python features introduced in Python 3.10:
- Structural pattern matching (`match` statements) - PEP 634
- Union type syntax with `|` - PEP 604
- Enhanced type hints

**Impact:** Cannot run on older Python installations.

**Distribution:** Many systems still use Python 3.8 or 3.9.

**Note:** If upgrading dependencies (e.g., for tomllib), may require Python 3.11+ in future versions.

### Binary Dependencies
**Status:** ⚠️ External Tools Required

**Required:**
- `unrar` or `unar` or `bsdtar` for RAR archives

**Installation Complexity:** Varies by platform; may require manual installation.

**Recommended Fix:** Consider pure-Python alternatives or bundle binaries.

## Security Limitations

### No Malware Scanning
**Status:** ❌ Not Implemented

**Description:** Parser does not scan for active malware in archives.

**Risk:** Archives may contain live malware samples.

**Recommendation:** Always run in isolated environment (VM, container, sandbox).

### No Input Validation
**Status:** ⚠️ Minimal

**Description:** Limited validation of archive contents.

**Risks:**
- Path traversal during extraction
- Zip bombs (decompression bombs)
- Memory exhaustion attacks
- Malicious filenames

**Recommended Fix:** Add comprehensive input validation and sanitization.

### Credential Handling
**Status:** ⚠️ Plain Text

**Description:** Parsed credentials are stored in plain text JSON.

**Risk:** Output files are highly sensitive.

**Recommendations:**
- Encrypted output option
- Secure deletion of temporary files
- Access control documentation
- Data retention policies

## Documentation Limitations

### Missing Content
**Status:** ⚠️ Incomplete

**Current:**
- Basic README
- Grammar documentation
- Contributing guide

**Missing:**
- Comprehensive API documentation
- Stealer-specific parsing guides
- Troubleshooting guide
- Video tutorials
- Use case examples
- Integration guides (SIEM, TIP)

### Code Comments
**Status:** ⚠️ Inconsistent

**Description:** Code is generally well-structured but lacks comprehensive inline documentation.

**Impact:** Steeper learning curve for contributors.

## Testing Limitations

### No Test Suite
**Status:** ❌ Not Implemented

**Description:** No automated tests exist.

**Impact:**
- Regression risk with changes
- Difficult to validate new features
- Cannot verify edge cases
- Quality assurance is manual

**Recommended Fix:** Implement comprehensive test suite:
- Unit tests for parsers
- Integration tests with sample archives
- Regression tests
- Fuzzing tests

### No CI/CD
**Status:** ⚠️ Pre-commit Only

**Description:** Only basic pre-commit hooks exist.

**Missing:**
- Automated test execution
- Build verification
- Release automation
- Security scanning
- Coverage reporting

## Dependency Management

### Submodule Complexity
**Status:** ⚠️ PLY as Submodule

**Description:** PLY is included as a Git submodule.

**Issues:**
- Requires `--recurse-submodules` during clone
- Complicates deployment
- Version pinning is manual

**Recommended Fix:** Consider using PLY as a PyPI package dependency.

### Version Pinning
**Status:** ⚠️ Relaxed Constraints

**Description:** Dependencies use caret (`^`) version constraints.

**Risk:** Updates may introduce breaking changes.

**Recommendation:** Stricter version pinning for production use.

## Summary of Priority Issues

### Critical (Should fix before 2.0)
1. ❌ No automated tests
2. ❌ No multi-part archive support
3. ❌ No progress feedback for large operations
4. ❌ Security: No input validation

### High (Impactful improvements)
1. ⚠️ Limited stealer coverage
2. ⚠️ Single output format (JSON only)
3. ⚠️ No parallel processing
4. ⚠️ CLI UX needs improvement (Rich integration)

### Medium (Nice to have)
1. ⚠️ Nested archive support
2. ⚠️ Charset detection
3. ⚠️ Configuration file support
4. ⚠️ Extended field extraction

### Low (Future enhancements)
1. GUI/TUI interface
2. Relationship mapping
3. Additional output formats
4. Built-in OSINT pivoting

## Tracking

For implementation status of fixes, see:
- [GitHub Issues](https://github.com/H4RR1SON/stealer-parser/issues)
- [Project Roadmap](ROADMAP.md)
- [Contributing Guide](../CONTRIBUTING.md)
