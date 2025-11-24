# Stealer Log File Structures

This document provides detailed information about the file structures used by various infostealer malware families. Understanding these structures is crucial for developing robust parsers and conducting effective threat intelligence analysis.

## Overview

Infostealer malware harvests sensitive information from compromised systems and packages it into "logs" - structured collections of files containing credentials, system information, and other data. Despite variations between malware families, most follow similar organizational patterns.

## Common Structure Patterns

### Typical Log Archive Structure

```
log_archive.zip/
├── [Machine_ID_or_Username]/
│   ├── System Information.txt
│   ├── Passwords.txt
│   ├── Cookies.txt
│   ├── Autofills.txt
│   ├── Credit Cards.txt
│   ├── [Browser_Name]/
│   │   ├── Passwords.txt
│   │   ├── Cookies.txt
│   │   └── History.txt
│   ├── Desktop/
│   │   └── [grabbed_files]
│   ├── Documents/
│   │   └── [grabbed_files]
│   └── Wallets/
│       └── [wallet_files]
└── Copyright.txt (or similar attribution file)
```

### Directory Naming Conventions

**System Identifiers:**
- Machine ID (GUID): `{12345678-1234-1234-1234-123456789012}`
- Username: `DESKTOP-ABC123_JohnDoe`
- Computer name: `WORKSTATION-01`
- Combination: `JohnDoe_192.168.1.100_2024-01-15`

## RedLine Stealer

### Characteristics
- **Most Common:** Accounts for 50-60% of stealer logs in circulation
- **Format:** Well-structured text files with clear prefixes
- **Attribution:** ASCII art banner or "RedLine" text

### File Structure

```
[SystemID]/
├── System Information.txt
├── Passwords.txt
├── Cookies/
│   ├── [Browser]_Cookies.txt
├── AutoFills/
│   └── [Browser]_Autofills.txt
├── CreditCards/
│   └── [Browser]_CreditCards.txt
└── UserInformation.txt
```

### System Information Format

```
*   ____  _____ ____  _     ___ _   _ _____   *
*  |  _ \| ____|  _ \| |   |_ _| \ | | ____|  *
*  | |_) |  _| | | | | |    | ||  \| |  _|    *
*  |  _ <| |___| |_| | |___ | || |\  | |___   *
*  |_| \_|_____|____/|_____|___|_| \_|_____|  *

MachineID: {GUID}
Computer Name: DESKTOP-ABC123
User Name: John Doe
OS: Windows 10 Pro
IP Address: 192.168.1.100
Country: US
Log date: 2024-01-15 10:30:45
```

### Password Format

```
Soft: Google Chrome

Host: https://www.example.com
Username: user@example.com
Password: SecurePassword123

---

Soft: Mozilla Firefox

URL: https://www.site.com/login
Login: testuser
Pass: mypassword456
```

**Key Prefixes:**
- `Soft:` or `Browser:` - Application name
- `Host:` or `URL:` or `Hostname:` - Target website
- `Username:` or `Login:` or `User:` - User identifier
- `Password:` or `Pass:` - Credential

## Raccoon Stealer

### Characteristics
- **Format:** Structured directories per browser
- **Attribution:** Raccoon ASCII art or "Raccoon" text
- **Special Feature:** Detailed cookie extraction

### File Structure

```
[ComputerName]_[IP]_[Date]/
├── Information.txt
├── [Browser] Default/
│   ├── Passwords.txt
│   ├── Cookies.txt
│   └── CC.txt
└── System Info.txt
```

### ASCII Art Banner

```
░░░░░░░░░░░░░░░▄▄▄▄▄▄▄▄░░░░░░░░░░░░░░
░▄█▀███▄▄████████████████████▄▄███▀█░
░█░░▀████████████████████████████░░█░
...
```

### Information Format

```
IP: 192.168.1.100
Country Code: US
Computer Name: HOME-PC
User Name: User
Install Date: 2024-01-15 10:30:00

Installed Apps:
- Google Chrome 120.0.6099.129
- Mozilla Firefox 121.0
- Discord 1.0.9015
```

## StealC

### Characteristics
- **Format:** Modular organization with subfolders
- **Attribution:** StealC ASCII art banner
- **Special Feature:** Comprehensive system profiling

### File Structure

```
[HWID]/
├── Information.txt
├── Passwords/
│   ├── Chrome_Default.txt
│   ├── Edge_Default.txt
├── Cookies/
│   ├── Chrome_Cookies.txt
├── Files/
│   ├── Desktop/
│   └── Documents/
└── Wallets/
```

### Banner

```
 ______     ______   ______     ______     __         ______
/\  ___\   /\__  _\ /\  ___\   /\  __ \   /\ \       /\  ___\
\ \___  \  \/_/\ \/ \ \  __\   \ \  __ \  \ \ \____  \ \ \____
 \/\_____\    \ \_\  \ \_____\  \ \_\ \_\  \ \_____\  \ \_____\
  \/_____/     \/_/   \/_____/   \/_/\/_/   \/_____/   \/_____/
```

### System Info Format

```
User Name: Administrator
Computer Name: WORK-PC
OS: Windows 10 Enterprise
HWID: 12345-67890-ABCDE-FGHIJ
IP Address: 10.0.0.50
Country: United Kingdom
Log date: 2024-01-15T10:30:45Z

Process List:
chrome.exe
firefox.exe
discord.exe
...
```

## Meta Stealer

### Characteristics
- **Format:** Simplified structure
- **Attribution:** "META" ASCII art
- **Special Feature:** Lightweight logs

### Banner

```
*              / \ / \ / \ / \                *
*             ( M | E | T | A )               *
*              \_/ \_/ \_/ \_/                *
```

### File Structure

```
Meta_[Date]_[ID]/
├── info.txt
├── passwords.txt
└── cookies.txt
```

### Password Format (Simplified)

```
Application: Chrome
[Browser = "Chrome"]

Host: example.com
User: user@example.com
Pass: password123
```

## LummaC2

### Characteristics
- **Format:** Compact with minimal structure
- **Attribution:** "LummaC2" or "Lummac2" text
- **Special Feature:** Focus on cryptocurrency

### File Structure

```
[ID]/
├── UserInformation.txt
├── passwords.txt
└── Wallets/
    ├── Metamask/
    ├── Coinbase/
    └── TrustWallet/
```

## Vidar Stealer

### Characteristics
- **Format:** Detailed browser separation
- **Attribution:** Often lacks clear banner
- **Special Feature:** Screenshot capture

### File Structure

```
[Date]_[IP]/
├── information.txt
├── [Browser]_[Profile]/
│   ├── passwords.txt
│   ├── cookies.txt
│   └── autofills.txt
└── screenshot.jpg
```

## Field Naming Variations

### System Information Fields

Different stealers use various names for the same fields:

| Field | Variants |
|-------|----------|
| Machine ID | `MachineID`, `UID`, `GUID`, `Device ID`, `Machine GUID` |
| Computer Name | `Computer Name`, `ComputerName`, `PC Name`, `Hostname`, `MachineName` |
| Hardware ID | `HWID`, `HardwareID`, `Hardware ID` |
| Username | `User Name`, `UserName`, `User`, `Current User`, `Username` |
| IP Address | `IP`, `Ip`, `IPAddress`, `IP Address`, `LANIP` |
| Country | `Country`, `Country Code`, `Location` |
| Date | `Log date`, `Last seen`, `Install Date`, `Date`, `Timestamp` |

### Credential Fields

| Field | Variants |
|-------|----------|
| Application | `Soft`, `SOFT`, `Browser`, `Application`, `Storage`, `["Browser" = "Profile"]` |
| Website | `Host`, `Hostname`, `URL`, `UR1` (leet speak), `Site` |
| Username | `Username`, `User`, `Login`, `USER LOGIN`, `USER`, `U53RN4M3` (leet) |
| Password | `Password`, `Pass`, `USER PASSWORD`, `PASS`, `P455W0RD` (leet) |

### Leet Speak Obfuscation

Some stealers use leet speak to evade basic string detection:
- `UR1` instead of `URL`
- `U53RN4M3` instead of `USERNAME`
- `P455W0RD` instead of `PASSWORD`

## File Content Patterns

### Empty Fields

Stealers handle missing data differently:

```
# Explicit empty
Host: 
Username: 
Password: 

# Implicit empty (field omitted)
Host: example.com
Password: pass123

# Placeholder
Host: N/A
Username: (none)
Password: [empty]
```

### Multiline Entries

Some password fields span multiple lines, often base64-encoded:

```
Password:
aGVsbG8gd29ybGQK
dGhpcyBpcyBhIHRlc3QK

(Decoded: long password or encrypted data)
```

### Special Characters

Logs may contain:
- Unicode characters (emoji, foreign languages)
- Control characters (tabs, carriage returns)
- HTML entities
- URL encoding
- Escape sequences

## Compressed Archive Formats

### Common Formats
- **ZIP**: Most common, sometimes password-protected
- **RAR**: Popular for multi-part archives
- **7z**: Used for higher compression ratios
- **TAR.GZ**: Less common, mostly Linux-targeted stealers

### Password Protection

Common passwords for protected archives:
- `infected`
- `malware`
- `virus`
- Date-based: `2024`, `01-15-2024`
- Simple: `123`, `password`

### Multi-part Archives

Large log collections may be split:
```
logs.zip
logs.z01
logs.z02
...
```

Or:
```
logs.part1.rar
logs.part2.rar
...
```

## Metadata and Attribution

### Copyright/Credits Files

Many logs include attribution files:

**Filenames:**
- `Copyright.txt`
- `Credits.txt`
- `ReadMe.txt`
- `Info.txt`
- `Read.txt`

**Content:**
- ASCII art logo
- Malware name and version
- Telegram channel for sales
- Build configuration
- Compilation date

**Example:**
```
╔═══════════════════════════════╗
║     RedLine Stealer v1.2      ║
║                               ║
║   @RedLineStealer (Telegram)  ║
║   Built: 2024-01-15           ║
╚═══════════════════════════════╝
```

### Seller Information

Sometimes includes:
```
Seller: @darkvendor
Log Tools: Professional Edition
Free Logs: t.me/freestealerlogs
```

## Special Files

### Desktop and Document Grabs

File types commonly grabbed:
- `.txt` - Text files
- `.pdf` - Documents
- `.doc`, `.docx` - Word documents
- `.xls`, `.xlsx` - Excel spreadsheets
- `.wallet`, `.dat` - Wallet files
- `.key`, `.pem` - Key files
- `.kdbx` - KeePass databases

### Screenshots

Some stealers capture screenshots:
- Filename: `screenshot.jpg` or `screen.png`
- Taken at moment of infection
- May reveal sensitive desktop information

### Session Tokens

Discord, Telegram, and other app tokens:
```
[Discord]
Token: MTIzNDU2Nzg5MDEyMzQ1Njc4OQ.GaBcDe.FgHiJkLmNoPqRsTuVwXyZ
Email: user@example.com
Phone: +1234567890

[Telegram]
Session: dc1:xxxxx:xxxxx:xxxxx
Phone: +1234567890
```

## Parsing Challenges

### Inconsistent Formatting
- Whitespace variations (tabs vs spaces)
- Missing newlines or extra blank lines
- Inconsistent field ordering
- Mixed case in field names

### Encoding Issues
- UTF-8 vs Latin-1 vs Windows-1252
- Byte order marks (BOM)
- Invalid characters
- Mixed encodings within single file

### Obfuscation
- Intentional misspellings
- Leet speak variations
- Base64 or hex encoding
- Custom encryption

### Corruption
- Incomplete exfiltration
- Transmission errors
- Partial archives
- Truncated files

## Detection and Identification

### Fingerprinting Stealers

**By ASCII Art:**
Most reliable method for attribution.

**By File Structure:**
- Directory naming patterns
- File organization
- Specific file names

**By Field Names:**
- Unique field prefixes
- Specific terminology
- Leet speak patterns

**By Content:**
- Telegram channels mentioned
- Version strings
- Build artifacts

## OSINT Applications

### Credential Pivoting
1. Extract email addresses
2. Search across other breaches
3. Identify password reuse patterns
4. Map user accounts

### Geolocation
1. Extract IP addresses
2. Resolve to geographic locations
3. Identify targeted regions
4. Map attack campaigns

### Victimology
1. Analyze system information
2. Identify organization types
3. Profile user behavior
4. Assess impact scope

### Attribution
1. Identify stealer family
2. Link to threat actors
3. Track distribution channels
4. Monitor campaign evolution

## References and Resources

### Academic Research
- [RedLine Stealer Analysis](https://securityscorecard.com/research/redline-stealer)
- [Raccoon Stealer Technical Report](https://cloudsek.com/raccoon-stealer)
- [StealC Analysis](https://anyrun.com/stealc-analysis)

### OSINT Guides
- [Haris Qazi - Stealer Logs](https://www.harisqazi.com/open-source-intelligence/breach-data/stealer-logs/)
- [Intel Techniques - Breach Data Lesson II](https://inteltechniques.com/blog/2022/07/06/new-breach-data-lesson-ii-stealer-logs/)
- [OSINT Team - Practical Newsletter #4](https://www.osintteam.com/the-practical-osint-newsletter/issue-4/)

### Threat Intelligence
- [Accenture - Info Stealer Malware](https://www.accenture.com/us-en/blogs/security/information-stealer-malware-on-dark-web)
- [ZeroFox - Introduction to Stealer Logs](https://www.zerofox.com/blog/an-introduction-to-stealer-logs/)
- [Lexfo - Infostealer Parser](https://blog.lexfo.fr/infostealer-parser.html)

### Related Parsers
- [Lexfo stealer-parser](https://github.com/lexfo/stealer-parser)
- [thredb sysinfo-parser](https://github.com/thredb/sysinfo-parser)
- [nak0823 RParseX](https://github.com/nak0823/RParseX)
- [milxss universal_stealer_log_parser](https://github.com/milxss/universal_stealer_log_parser)

## Updates and Evolution

Stealer log formats continuously evolve:
- New malware families emerge regularly
- Existing stealers receive updates
- Obfuscation techniques improve
- File structures adapt to evade detection

**Recommendation:** Regularly update parser rules and test against new samples.
