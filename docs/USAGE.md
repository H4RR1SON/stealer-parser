# Usage Guide

Comprehensive guide for using the Stealer Parser for CTI (Cyber Threat Intelligence) and OSINT analysis.

## Table of Contents

- [Getting Started](#getting-started)
- [Basic Usage](#basic-usage)
- [Advanced Usage](#advanced-usage)
- [Output Analysis](#output-analysis)
- [Use Cases](#use-cases)
- [Troubleshooting](#troubleshooting)
- [Best Practices](#best-practices)

## Getting Started

### Installation

1. **Clone the repository:**
```bash
git clone --recurse-submodules https://github.com/H4RR1SON/stealer-parser
cd stealer-parser
```

2. **Install dependencies:**
```bash
# Install Poetry if not already installed
pip install poetry

# Install project dependencies
poetry install
```

3. **Install external tools:**

For RAR archive support, install one of:
- **Linux:** `sudo apt install unrar` or `sudo apt install unar`
- **macOS:** `brew install unrar`
- **Windows:** Download UnRAR from [rarlab.com](https://www.rarlab.com/)

4. **Activate environment:**
```bash
poetry shell
```

### Verify Installation

```bash
stealer_parser --help
```

You should see the help message with available options.

## Basic Usage

### Parse a Single Archive

```bash
stealer_parser logs.zip
```

Output: `logs.json`

### Specify Output File

```bash
stealer_parser logs.zip -o results/analysis.json
```

### Password-Protected Archives

```bash
stealer_parser protected.zip --password infected
```

Common passwords for stealer archives:
- `infected`
- `malware`
- `virus`
- Date-based: `2024`, `01-15-2024`

### Verbose Output

```bash
# Basic verbose
stealer_parser logs.zip -v

# Debug mode
stealer_parser logs.zip -vv

# Maximum verbosity (spam)
stealer_parser logs.zip -vvv
```

**Verbosity Levels:**
- **None (default):** INFO messages only
- **`-v`:** VERBOSE - detailed processing info
- **`-vv`:** DEBUG - parser-level details
- **`-vvv`:** SPAM - token-level tracing

## Advanced Usage

### Batch Processing

Process multiple archives:

```bash
#!/bin/bash
# batch_process.sh

for archive in archives/*.zip; do
    echo "Processing $archive"
    stealer_parser "$archive" -o "results/$(basename "$archive" .zip).json"
done
```

### Processing with Timeout

```bash
# Linux/macOS - use timeout command
timeout 300s stealer_parser large_archive.zip
```

### Error Logging

Redirect errors to a file:

```bash
stealer_parser logs.zip 2> errors.log
```

Or capture both stdout and stderr:

```bash
stealer_parser logs.zip &> full_output.log
```

### Using with Python Scripts

```python
from stealer_parser.main import read_archive, main
from stealer_parser.processing import process_archive
from stealer_parser.models import Leak
from io import BytesIO

# Programmatic usage
def parse_archive(filepath: str, password: str = None) -> Leak:
    """Parse archive programmatically."""
    leak = Leak(filename=filepath)
    
    with open(filepath, 'rb') as f:
        with BytesIO(f.read()) as buffer:
            archive = read_archive(buffer, filepath, password)
            process_archive(None, leak, archive)  # Pass logger if needed
            archive.close()
    
    return leak

# Usage
leak = parse_archive('logs.zip')
print(f"Found {len(leak.systems_data)} systems")
```

## Output Analysis

### JSON Structure

```json
{
  "filename": "logs.zip",
  "systems_data": [
    {
      "system": {
        "machine_id": "{12345678-1234-1234-1234-123456789012}",
        "computer_name": "DESKTOP-ABC123",
        "hardware_id": "HWID-12345",
        "machine_user": "JohnDoe",
        "ip_address": "192.168.1.100",
        "country": "US",
        "log_date": "2024-01-15T10:30:45"
      },
      "credentials": [
        {
          "software": "google chrome",
          "host": "https://example.com",
          "username": "user@example.com",
          "password": "password123",
          "domain": "example.com",
          "local_part": "user",
          "email_domain": "example.com",
          "filepath": "logs.zip/System_001/Passwords.txt",
          "stealer_name": "redline"
        }
      ]
    }
  ]
}
```

### Query Results with jq

Extract specific information using `jq`:

```bash
# Count total credentials
jq '[.systems_data[].credentials | length] | add' results.json

# Find all Gmail credentials
jq '.systems_data[].credentials[] | select(.domain == "gmail.com")' results.json

# List unique domains
jq -r '.systems_data[].credentials[].domain' results.json | sort -u

# Filter by stealer type
jq '.systems_data[] | select(.credentials[0].stealer_name == "redline")' results.json

# Get systems by country
jq '.systems_data[] | select(.system.country == "US")' results.json

# Find credentials with specific username
jq '.systems_data[].credentials[] | select(.username | contains("admin"))' results.json

# Extract all emails
jq -r '.systems_data[].credentials[].username | select(contains("@"))' results.json | sort -u

# Count systems by stealer
jq -r '.systems_data[].credentials[0].stealer_name' results.json | sort | uniq -c
```

### Import into Database

**SQLite:**
```python
import json
import sqlite3

# Load JSON
with open('results.json') as f:
    data = json.load(f)

# Create database
conn = sqlite3.connect('stealer_logs.db')
cursor = conn.cursor()

# Create tables
cursor.execute('''
    CREATE TABLE systems (
        id INTEGER PRIMARY KEY,
        machine_id TEXT,
        computer_name TEXT,
        ip_address TEXT,
        country TEXT
    )
''')

cursor.execute('''
    CREATE TABLE credentials (
        id INTEGER PRIMARY KEY,
        system_id INTEGER,
        software TEXT,
        host TEXT,
        username TEXT,
        password TEXT,
        domain TEXT,
        stealer_name TEXT,
        FOREIGN KEY(system_id) REFERENCES systems(id)
    )
''')

# Insert data
for system_data in data['systems_data']:
    system = system_data.get('system')
    if system:
        cursor.execute(
            'INSERT INTO systems (machine_id, computer_name, ip_address, country) VALUES (?, ?, ?, ?)',
            (system.get('machine_id'), system.get('computer_name'), 
             system.get('ip_address'), system.get('country'))
        )
        system_id = cursor.lastrowid
    else:
        system_id = None
    
    for cred in system_data['credentials']:
        cursor.execute(
            'INSERT INTO credentials (system_id, software, host, username, password, domain, stealer_name) VALUES (?, ?, ?, ?, ?, ?, ?)',
            (system_id, cred.get('software'), cred.get('host'), 
             cred.get('username'), cred.get('password'), 
             cred.get('domain'), cred.get('stealer_name'))
        )

conn.commit()
conn.close()
```

**PostgreSQL:**
```python
import json
import psycopg2

# Connect to PostgreSQL
conn = psycopg2.connect(
    dbname="stealer_logs",
    user="username",
    password="password",
    host="localhost"
)
cursor = conn.cursor()

# Create tables
cursor.execute('''
    CREATE TABLE IF NOT EXISTS systems (
        id SERIAL PRIMARY KEY,
        machine_id TEXT,
        computer_name TEXT,
        ip_address INET,
        country VARCHAR(2),
        log_date TIMESTAMP
    )
''')

cursor.execute('''
    CREATE TABLE IF NOT EXISTS credentials (
        id SERIAL PRIMARY KEY,
        system_id INTEGER REFERENCES systems(id),
        software TEXT,
        host TEXT,
        username TEXT,
        password TEXT,
        domain TEXT,
        stealer_name TEXT
    )
''')

# Similar insert logic as SQLite...
```

### Convert to CSV

```python
import json
import csv

try:
    # Load JSON
    with open('results.json', encoding='utf-8') as f:
        data = json.load(f)

    # Export to CSV
    with open('credentials.csv', 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.writer(csvfile)
        
        # Header
        writer.writerow([
            'Software', 'Host', 'Domain', 'Username', 'Password',
            'Stealer', 'IP', 'Country', 'Date'
        ])
        
        # Data
        for system_data in data['systems_data']:
            system = system_data.get('system', {})
            for cred in system_data['credentials']:
                try:
                    writer.writerow([
                        cred.get('software', ''),
                        cred.get('host', ''),
                        cred.get('domain', ''),
                        cred.get('username', ''),
                        cred.get('password', ''),
                        cred.get('stealer_name', ''),
                        system.get('ip_address', ''),
                        system.get('country', ''),
                        system.get('log_date', '')
                    ])
                except Exception as e:
                    print(f"Warning: Failed to write credential row: {e}")
                    continue
    
    print(f"Successfully exported to credentials.csv")

except FileNotFoundError:
    print("Error: results.json not found")
except json.JSONDecodeError as e:
    print(f"Error: Invalid JSON format - {e}")
except Exception as e:
    print(f"Error: Failed to export CSV - {e}")
```

## Use Cases

### 1. Threat Intelligence Analysis

**Objective:** Identify attack patterns and stealer distribution.

```bash
# Parse the archive
stealer_parser campaign_2024.zip -o campaign.json

# Analyze with jq
# Count systems by stealer family
jq -r '.systems_data[].credentials[0].stealer_name' campaign.json | sort | uniq -c | sort -rn

# Geographic distribution
jq -r '.systems_data[].system.country' campaign.json | sort | uniq -c | sort -rn

# Timeline analysis
jq -r '.systems_data[].system.log_date' campaign.json | sort | head -1  # First infection
jq -r '.systems_data[].system.log_date' campaign.json | sort | tail -1  # Last infection
```

### 2. Credential Intelligence

**Objective:** Identify compromised credentials for your organization.

```bash
# Parse logs
stealer_parser leaked_logs.zip -o analysis.json

# Find your domain credentials
jq '.systems_data[].credentials[] | select(.email_domain == "yourcompany.com")' analysis.json > company_compromised.json

# Extract unique usernames
jq -r '.username' company_compromised.json | sort -u > compromised_accounts.txt

# Check for admin accounts
jq 'select(.username | contains("admin"))' company_compromised.json
```

### 3. Password Analysis

**Objective:** Analyze password patterns and reuse.

```python
import json
from collections import Counter

# Load data
with open('results.json') as f:
    data = json.load(f)

# Find password reuse
password_map = {}
for system_data in data['systems_data']:
    for cred in system_data['credentials']:
        pwd = cred.get('password')
        if pwd:
            if pwd not in password_map:
                password_map[pwd] = []
            password_map[pwd].append({
                'username': cred.get('username'),
                'host': cred.get('host')
            })

# Passwords used multiple times
reused = {pwd: accounts for pwd, accounts in password_map.items() if len(accounts) > 1}

print(f"Found {len(reused)} reused passwords")
for pwd, accounts in list(reused.items())[:10]:
    print(f"\nPassword: {pwd}")
    print(f"Used in {len(accounts)} accounts:")
    for account in accounts[:5]:  # Show first 5
        print(f"  - {account['username']} @ {account['host']}")
```

### 4. OSINT Pivoting

**Objective:** Pivot on email addresses across multiple breaches.

```bash
# Search for specific email
EMAIL="target@example.com"

# Parse multiple archives
for archive in breaches/*.zip; do
    stealer_parser "$archive" -o "results/$(basename "$archive" .zip).json"
done

# Search across all results
grep -r "$EMAIL" results/*.json

# Or with jq
for file in results/*.json; do
    echo "Checking $file:"
    jq --arg email "$EMAIL" '.systems_data[].credentials[] | select(.username == $email)' "$file"
done
```

### 5. Network Infrastructure Mapping

**Objective:** Map compromised systems by network.

```python
import json
from collections import defaultdict
import ipaddress

# Load data
with open('results.json') as f:
    data = json.load(f)

# Group by subnet
subnets = defaultdict(list)

for system_data in data['systems_data']:
    system = system_data.get('system')
    if system and system.get('ip_address'):
        try:
            ip = ipaddress.ip_address(system['ip_address'])
            # Group by /24 subnet
            subnet = str(ipaddress.ip_network(f"{ip}/24", strict=False))
            subnets[subnet].append(system)
        except ValueError:
            pass  # Invalid IP

# Display results
print(f"Found {len(subnets)} distinct subnets")
for subnet, systems in sorted(subnets.items(), key=lambda x: len(x[1]), reverse=True)[:10]:
    print(f"\n{subnet}: {len(systems)} systems")
    for system in systems[:3]:
        print(f"  - {system.get('computer_name')} ({system.get('ip_address')})")
```

### 6. Temporal Correlation

**Objective:** Identify infection waves and campaigns.

```python
import json
from datetime import datetime
from collections import Counter

# Load data
with open('results.json') as f:
    data = json.load(f)

# Extract dates
dates = []
for system_data in data['systems_data']:
    system = system_data.get('system')
    if system and system.get('log_date'):
        try:
            date = datetime.fromisoformat(system['log_date'])
            dates.append(date.date())
        except ValueError:
            pass

# Count by date
date_counts = Counter(dates)

# Display timeline
print("Infection timeline:")
for date in sorted(date_counts.keys()):
    count = date_counts[date]
    bar = '█' * (count // 10)  # Simple bar chart
    print(f"{date}: {count:4d} {bar}")
```

## Troubleshooting

### Common Issues

#### 1. "ModuleNotFoundError: No module named 'stealer_parser.ply.src'"

**Problem:** PLY submodule not initialized.

**Solution:**
```bash
git submodule update --init --recursive
```

#### 2. "unrar binary not found"

**Problem:** RAR extraction tool not installed.

**Solution:**
```bash
# Ubuntu/Debian
sudo apt install unrar

# macOS
brew install unrar

# Or use unar as alternative
sudo apt install unar  # Ubuntu
brew install unar      # macOS
```

#### 3. "UnicodeDecodeError"

**Problem:** File encoding not supported.

**Workaround:** Files with encoding issues are skipped. Check `logs/` directory for error details.

**Potential Fix:** Manual conversion with `iconv`:
```bash
iconv -f WINDOWS-1252 -t UTF-8 problem_file.txt > fixed_file.txt
```

#### 4. "Memory Error" on Large Archives

**Problem:** Archive too large for available RAM.

**Solutions:**
- Increase swap space
- Extract archive manually and process in batches
- Use a machine with more RAM
- Wait for streaming implementation (see IMPROVEMENTS.md)

#### 5. "BadRarFile" or Corrupted Archive

**Problem:** Archive is damaged or uses unsupported compression.

**Solutions:**
- Try different extraction tool
- Repair archive with `rar r archive.rar`
- Check if file downloaded completely
- Verify archive integrity with `unrar t archive.rar`

#### 6. No Output Generated

**Problem:** Processing completed but no JSON file created.

**Possible Causes:**
- No valid log files found in archive
- All files failed to parse (check logs/)
- Permissions issue writing output

**Debug Steps:**
```bash
# Run with maximum verbosity
stealer_parser archive.zip -vvv

# Check logs directory
ls -la logs/

# Try explicit output path
stealer_parser archive.zip -o /tmp/output.json
```

### Performance Issues

#### Slow Processing

**If processing is very slow:**

1. **Check file count:**
```bash
# For ZIP
unzip -l archive.zip | wc -l

# For RAR
unrar l archive.rar | wc -l
```

Large archives (10k+ files) will take longer.

2. **Monitor resources:**
```bash
# Linux/macOS
top
htop  # If installed

# Watch progress
stealer_parser large.zip -v  # Verbose mode shows progress
```

3. **Consider splitting:**
If archive is very large, extract and process subdirectories separately.

### Getting Help

If issues persist:

1. **Check existing issues:** [GitHub Issues](https://github.com/H4RR1SON/stealer-parser/issues)
2. **Create new issue:** Include:
   - Full error message
   - Archive type and size
   - Command used
   - Verbose output (`-vvv`)
3. **Discord/Community:** Check README for community channels

## Best Practices

### Security

1. **Isolated Environment:**
   - Run parser in VM or container
   - Never run on production systems
   - Logs may contain malware

2. **Secure Storage:**
   - Encrypt output JSON files
   - Use access controls
   - Implement data retention policies

3. **Sanitize Data:**
   - Redact sensitive info before sharing
   - Hash passwords if not needed in plaintext
   - Remove PII when possible

### Performance

1. **Process Archives Locally:**
   - Download before processing
   - Don't process from network shares
   - Use SSD for better I/O

2. **Batch Processing:**
   - Process multiple archives in parallel
   - Use job queues for large datasets
   - Monitor system resources

3. **Output Management:**
   - Use meaningful output names
   - Organize results by date/campaign
   - Clean up old results regularly

### Data Quality

1. **Validate Results:**
   - Spot-check random samples
   - Verify credential counts
   - Check for obvious parsing errors

2. **Handle Errors:**
   - Review failed files in `logs/`
   - Attempt manual parsing if needed
   - Report systematic issues

3. **Document Findings:**
   - Track archive sources
   - Note stealer families found
   - Record campaign indicators

### Compliance

1. **Legal Considerations:**
   - Only analyze authorized data
   - Follow applicable laws (GDPR, CCPA, etc.)
   - Document chain of custody

2. **Ethical Use:**
   - Don't misuse extracted credentials
   - Notify affected parties when appropriate
   - Follow responsible disclosure

3. **Data Protection:**
   - Minimize data retention
   - Secure deletion when done
   - Audit access logs

## Advanced Techniques

### Custom Parsers

For specialized log formats, create custom parsers:

```python
# my_custom_parser.py
from stealer_parser.parsing import LogsParser
from stealer_parser.models import Credential

def parse_custom_format(logger, filename, text):
    """Parse custom stealer format."""
    credentials = []
    
    # Custom parsing logic
    lines = text.split('\n')
    for i in range(0, len(lines), 3):
        if i+2 < len(lines):
            cred = Credential(
                host=lines[i].strip(),
                username=lines[i+1].strip(),
                password=lines[i+2].strip(),
                filepath=filename
            )
            credentials.append(cred)
    
    return credentials
```

### Automation

Integrate with automation tools:

**Cron job:**
```bash
# /etc/cron.d/stealer-parser
# Run daily at 2 AM
0 2 * * * user cd /path/to/stealer-parser && ./process_new_archives.sh
```

**Systemd timer:**
```ini
# /etc/systemd/system/stealer-parser.timer
[Unit]
Description=Process stealer logs daily

[Timer]
OnCalendar=daily
OnCalendar=02:00

[Install]
WantedBy=timers.target
```

### Integration with SIEM

Send results to SIEM for correlation:

```python
import json
import requests

# Load results
with open('results.json') as f:
    data = json.load(f)

# Send to SIEM (example: Splunk HEC)
for system_data in data['systems_data']:
    event = {
        'sourcetype': 'stealer_log',
        'event': system_data
    }
    
    requests.post(
        'https://splunk:8088/services/collector/event',
        headers={'Authorization': 'Splunk YOUR_TOKEN'},
        json=event
    )
```

## Reference

- [Architecture Documentation](ARCHITECTURE.md)
- [File Structure Guide](FILE_STRUCTURES.md)
- [Known Limitations](LIMITATIONS.md)
- [Improvement Recommendations](IMPROVEMENTS.md)
- [Contributing Guide](../CONTRIBUTING.md)
- [GitHub Repository](https://github.com/H4RR1SON/stealer-parser)
