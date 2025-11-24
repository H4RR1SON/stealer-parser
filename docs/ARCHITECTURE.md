# Stealer Parser Architecture

## Overview

The Stealer Parser is a Python-based tool designed to parse information stealer (infostealer) malware logs. These logs contain sensitive data harvested from compromised systems, including credentials, system information, and other artifacts.

## Architecture Components

### 1. Entry Point (`main.py`)

The main entry point orchestrates the parsing workflow:

```
User Input → Archive Reading → Processing → JSON Output
```

**Key Functions:**
- `read_archive()`: Opens and wraps archive files (.rar, .zip, .7z)
- `main()`: Handles command-line arguments, error handling, and output generation

**Dependencies:**
- `rarfile`: For RAR archive extraction
- `zipfile`: For ZIP archive extraction
- `py7zr`: For 7z archive extraction

### 2. Processing Engine (`processing.py`)

The core processing logic that analyzes archive contents:

**Workflow:**
```
Archive → File List Generation → System Directory Grouping → Parsing → Data Collection
```

**Key Components:**

#### File Classification
Files are classified using regex patterns into:
- **PASSWORDS** (`password*.txt`): Contains credentials
- **SYSTEM** (`system*.txt`, `information*.txt`, `userinfo*.txt`): System information
- **IP** (`ip*.txt`): IP address information
- **COPYRIGHT** (`credits*.txt`, `copyright*.txt`, `read*.txt`): Stealer attribution

#### System Directory Processing
Logs are organized by compromised system:
- Each system's files are grouped together
- Stealer name detection occurs per-system
- Credentials and system data are linked together

### 3. Parsing Layer (`parsing/`)

Implements lexical analysis and parsing using PLY (Python Lex-Yacc):

#### Lexers (`lexer_passwords.py`, `lexer_system.py`)
- **Token Definition**: Define grammar tokens (WORD, NEWLINE, SPACE, prefixes)
- **Tokenization**: Convert raw text into token streams

#### Parsers (`parsing_passwords.py`, `parsing_system.py`)
- **Grammar Rules**: Implement context-free grammars (see `docs/grammar_*.txt`)
- **Data Extraction**: Extract structured data from token streams

**Example Token Flow (Passwords):**
```
Raw Text:
"Host: example.com
Username: user@example.com
Password: pass123"

Tokens:
HOST_PREFIX, SPACE, WORD(example.com), NEWLINE,
USER_PREFIX, SPACE, WORD(user@example.com), NEWLINE,
PASSWORD_PREFIX, SPACE, WORD(pass123), NEWLINE

Parsed Output:
Credential(host="example.com", username="user@example.com", password="pass123")
```

### 4. Data Models (`models/`)

Structured data representation:

#### `Credential`
Represents a single credential entry:
- `software`: Browser/application name
- `host`: URL or hostname
- `username`: Login identifier
- `password`: Password value
- `domain`: Extracted domain from host
- `local_part`, `email_domain`: Parsed email components
- `filepath`: Source file location
- `stealer_name`: Detected stealer family

#### `System`
Represents compromised system information:
- `machine_id`: Unique device identifier
- `computer_name`: Computer hostname
- `hardware_id`: HWID
- `machine_user`: Username
- `ip_address`: IP address
- `country`: Country code
- `log_date`: Infection timestamp

#### `Leak`
Container for the entire parsed dataset:
- `filename`: Source archive name
- `systems_data`: List of `SystemData` objects

#### `SystemData`
Links system information to its credentials:
- `system`: System information
- `credentials`: List of credentials from that system

### 5. Stealer Detection (`search_stealer_credits.py`)

Identifies the infostealer family using:

**Detection Methods:**
1. **Regex Matching**: Searches for keywords (redline, stealc, raccoon, lummac2)
2. **ASCII Art Banners**: Matches known ASCII art signatures
3. **Pattern Analysis**: Analyzes file structure and naming

**Supported Stealers:**
- RedLine Stealer
- StealC
- Raccoon Stealer
- Meta Stealer
- LummaC2
- DcRat

### 6. Helper Utilities (`helpers.py`)

Provides common functionality:
- `init_logger()`: Configures colored logging with verbosity levels
- `parse_options()`: Command-line argument parsing
- `dump_to_file()`: JSON serialization with custom encoding
- `EnhancedJSONEncoder`: Handles dataclasses, dates, and sets

## Data Flow

```
┌─────────────────┐
│  Archive File   │
│ (.zip/.rar/.7z) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  ArchiveWrapper │ (Unified interface)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ File List Gen   │ (Regex-based classification)
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│ System Dir Processing   │
│  ┌──────────────────┐   │
│  │ Stealer Detection│   │
│  └──────────────────┘   │
│  ┌──────────────────┐   │
│  │ Password Parsing │   │
│  └──────────────────┘   │
│  ┌──────────────────┐   │
│  │ System Parsing   │   │
│  └──────────────────┘   │
│  ┌──────────────────┐   │
│  │ IP Extraction    │   │
│  └──────────────────┘   │
└────────┬────────────────┘
         │
         ▼
┌─────────────────┐
│  Data Model     │
│  (Leak object)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  JSON Output    │
└─────────────────┘
```

## Error Handling

### Parse Errors
- Caught at the file level
- Logged with detailed error messages
- Processing continues for other files

### Archive Errors
- Bad file format → `NotImplementedError`
- Corrupted archive → `BadRarFile` or similar
- Missing password → Exception during extraction

### Recovery Strategy
- Failed files are logged to `logs/` directory
- Partial results are still output
- Error context is preserved for debugging

## Configuration

### Verbosity Levels
- **INFO** (default): Basic operation messages
- **VERBOSE** (`-v`): Detailed processing information
- **DEBUG** (`-vv`): Parser-level debugging
- **SPAM** (`-vvv`): Token-level tracing

### Archive Passwords
Protected archives can be opened with `-p/--password` flag.

### Output Customization
Default: `<archive_name>.json`
Custom: `-o/--outfile` parameter

## Performance Considerations

### Memory Management
- Archives are read into `BytesIO` buffers
- Files are processed sequentially
- Archives are properly closed after processing

### Scalability
- File list is generated once and sorted
- System directories are processed in batches
- Regex compilation is done once at module load

### Bottlenecks
1. **Archive Extraction**: Depends on compression format and password
2. **Tokenization**: PLY lexer is relatively fast but processes linearly
3. **Regex Matching**: File classification regex is evaluated for every file

## Extension Points

### Adding New Stealer Support
1. Add ASCII art signature to `search_stealer_credits.py`
2. Add keyword to regex pattern
3. Update documentation

### Supporting New File Formats
1. Add case to `read_archive()` in `main.py`
2. Add appropriate library to dependencies
3. Test with sample archives

### Custom Parsers
1. Define new lexer in `parsing/`
2. Define grammar tokens and rules
3. Add parser function
4. Update `parse_file()` in `processing.py`

### Output Formats
1. Extend `EnhancedJSONEncoder` for new types
2. Add format conversion in `dump_to_file()`
3. Update command-line options

## Dependencies

### Core
- Python 3.10+
- PLY (submodule): Lexing and parsing

### Archive Handling
- `rarfile`: RAR extraction (requires unrar binary)
- `py7zr`: 7z extraction
- `zipfile`: ZIP extraction (stdlib)

### Utilities
- `coloredlogs`: Colored console output
- `verboselogs`: Extended logging levels

### Development
- `poetry`: Dependency management
- `pre-commit`: Code quality checks
- `pycodestyle`: Style checking

## Testing Strategy

### Current State
- No automated test suite currently exists
- Testing is manual with sample archives

### Recommended Approach
1. Unit tests for parsers with known token sequences
2. Integration tests with sample log archives
3. Regression tests for each stealer family
4. Fuzzing tests for malformed inputs

## Security Considerations

### Isolated Processing
- Parser should run in sandboxed/isolated environment
- Log files may contain active malware or payloads

### Data Sensitivity
- Output contains credentials and PII
- Secure storage required for JSON outputs
- Access control recommendations for parsed data

### Input Validation
- Archives may be maliciously crafted
- Path traversal attacks possible during extraction
- Memory exhaustion via large/recursive archives

## Future Architecture Improvements

### Modularity
- Plugin system for stealer-specific parsers
- Configurable output formats (CSV, XML, database)
- Extensible data models

### Performance
- Parallel processing of system directories
- Streaming JSON output for large archives
- Incremental parsing for memory efficiency

### Reliability
- Comprehensive error recovery
- Schema validation for outputs
- Archive format detection before extraction

### Observability
- Metrics collection (parse times, success rates)
- Progress indicators for large archives
- Detailed parse statistics in output
