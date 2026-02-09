# CC312 Logs CLI Tool

A lightweight command-line utility for parsing, validating, and filtering cloud application logs with support for severity level and service-based filtering. Learning about monitoring, scripting and how to deal with large amounts of data while working on these small projects. 

## Overview

This project implements a log filter tool designed for CC312 course requirements. It processes large log files, validates log entries against a strict format, and outputs filtered results based on user-specified criteria. The tool handles malformed or invalid log entries gracefully by skipping them.

## Features

- **Format Validation**: Automatically validates logs against the required format (timestamp | level | service | message)
- **Smart Filtering**: Filter logs by severity level (INFO, WARN, ERROR) and/or service name
- **Case-Insensitive Input**: Accepts log levels in any case (error, ERROR, Error, etc.)
- **Invalid Line Handling**: Silently skips malformed entries and reports only valid processed logs
- **Performance Metrics**: Displays summary statistics after processing
- **Configurable Output**: Specify custom output filenames or use default

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.x |
| CLI Framework | `argparse` (Python Standard Library) |
| File I/O | Python `pathlib.Path` |
| Encoding | UTF-8 |

**Dependencies**: None (uses only Python Standard Library)

## Log Format

Valid log entries must follow this structure:

```
timestamp | level | service | message
```

### Field Specifications
- **timestamp**: ISO 8601 format (e.g., `2026-02-01 00:00:00`)
- **level**: INFO, WARN, or ERROR (case-insensitive in processing)
- **service**: Service identifier (e.g., auth, api, payments, search)
- **message**: Log message with optional structured metadata

### Example Valid Log
```
2026-02-01 00:00:15 | INFO | auth | Cache miss key=rate:218.224.64.212 (env=staging, region=me-south-1, ip=218.224.64.212)
```

### Invalid Log Examples
```
2026-02-01 00:00:00 | INFO | auth                    # Only 3 fields
2026-02-01 | INFO | auth | Missing seconds          # Invalid timestamp format

                                                     # Empty line
```

## Installation

1. Clone the repository:
```bash
git clone https://github.com/Ali-AlQassab08/CC312_Logs_CLI_Tool.git
cd CC312_Logs_CLI_Tool
```

2. Ensure Python 3.x is installed:
```bash
python --version
```

3. Place `logs.txt` in the project directory (if not already present)

## Usage

### Basic Usage

Process all valid logs:
```bash
python log_tool.py
```
Output: `filtered_logs.txt` (default)

### Filter by Log Level

Extract ERROR logs only:
```bash
python log_tool.py --level ERROR
```

Extract WARNING logs:
```bash
python log_tool.py --level WARN
```

### Filter by Service

Extract logs from a specific service:
```bash
python log_tool.py --service auth
```

Extract logs from the auth service and save to custom file:
```bash
python log_tool.py --service auth --out auth_logs.txt
```

### Combined Filters

Extract ERROR logs from the auth service:
```bash
python log_tool.py --level ERROR --service auth --out error_auth.txt
```

### All Command-Line Options

```
positional arguments:
  (none)

optional arguments:
  --level LEVEL       Filter by log level (INFO, WARN, ERROR) - case insensitive
  --service SERVICE   Filter by service name
  --out OUT           Output filename (default: filtered_logs.txt)
  -h, --help          Show help message and exit
```

## Output

The tool generates:

1. **Output File**: Contains filtered log lines (one per line), cleaned of leading/trailing whitespace
2. **Console Summary**: Three-line report showing processing statistics

Example output:
```
Valid lines scanned: 12847
Lines written: 342
Output file: auth_logs.txt
```

## How It Works

### Processing Pipeline

```
┌─ Read logs.txt
│
├─ For each line:
│  ├─ Parse line (split by |, trim whitespace)
│  ├─ Check format validity (exactly 4 fields)
│  ├─ Validate level (INFO|WARN|ERROR)
│  ├─ Apply user filters (level, service)
│  └─ If match: add to output buffer
│
├─ Write output buffer to file
│
└─ Print summary (scanned count, written count, filename)
```

### Key Functions

- **`parse_line(line: str)`**: Parses a log line into components or returns None if invalid
- **`is_valid_level(level: str) -> bool`**: Checks if level is INFO, WARN, or ERROR
- **`matches_filters(level, service, level_filter, service_filter) -> bool`**: Determines if a log matches applied filters
- **`build_arg_parser()`**: Constructs the argument parser for CLI
- **`main()`**: Orchestrates the entire pipeline

## Example Scenarios

### Scenario 1: Find all authentication errors
```bash
python log_tool.py --level ERROR --service auth --out auth_errors.txt
```

### Scenario 2: Audit warnings across all services
```bash
python log_tool.py --level WARN --out all_warnings.txt
```

### Scenario 3: Get all logs from database service
```bash
python log_tool.py --service db --out db_logs.txt
```

## Performance Characteristics

- **Time Complexity**: O(n) where n = number of log lines
- **Space Complexity**: O(m) where m = number of matching lines (output buffer)
- **Suitable For**: Log files up to several million lines
- **Note**: Entire output is buffered in memory before writing

## Error Handling

| Scenario | Behavior |
|----------|----------|
| `logs.txt` not found | Prints error message and exits gracefully |
| Invalid log format | Skipped silently (not counted in output) |
| Invalid log level | Skipped silently (only INFO/WARN/ERROR accepted) |
| No matches found | Creates empty output file with 0 lines written |
| Invalid CLI arguments | Displays usage help and exits |

## Validation Rules

✓ **Valid lines must have**:
- Exactly 4 pipe-separated fields
- Non-empty fields (after trimming)
- Level value matching INFO, WARN, or ERROR (case-insensitive)

✗ **Invalid lines**:
- Empty or whitespace-only lines
- Wrong number of fields (< 4 or > 4)
- Unknown log level value
- Malformed field content

## Example Workflow

```bash
# Process 66,942 lines in logs.txt
$ python log_tool.py --level WARN --out warnings.txt

# Output shown
Valid lines scanned: 8932
Lines written: 1247  
Output file: warnings.txt

# Verify results
$ head -5 warnings.txt
2026-02-01 00:00:07 | WARN | search | Rate limit nearing threshold remaining=17
2026-02-01 00:00:10 | WARN | orders | Temporary network jitter detected rtt_ms=338
2026-02-01 00:00:12 | WARN | frontend | High memory usage warning mem=60%
...
```

## Project Structure

```
CC312_Logs_CLI_Tool/
├── log_tool.py              # Main application
├── logs.txt                 # Input log file
├── filtered_logs.txt        # Default output (all valid logs)
├── auth_logs.txt            # Example: filtered by service=auth
├── db_logs.txt              # Example: filtered by service=db
├── warn_logs.txt            # Example: filtered by level=WARN
├── run_proof.txt            # Execution proof/documentation
├── .gitignore               # Git ignore rules
└── README.md                # This file
```

## Testing

Sample output files are included for validation:
- `auth_logs.txt`: Contains 3,238 logs from auth service (all levels)
- `warn_logs.txt`: Contains WARNING level logs from all services
- `db_logs.txt`: Contains logs from db service
- `filtered_logs.txt`: Contains all valid logs with no filters applied

## Limitations

1. **Single Input File**: Tool only processes `logs.txt` (not configurable)
2. **Basic Filtering**: Only supports level and service filters; no timestamp ranges or regex
3. **No Multi-File Input**: Cannot merge logs from multiple sources
4. **Memory Buffer**: All matching lines held in memory before writing
5. **Fixed Level Set**: Only three severity levels (INFO, WARN, ERROR)
6. **No Log Reformatting**: Output logs maintain original format

## Future Enhancements

- [ ] Configurable input file path
- [ ] Timestamp-based filtering (--after, --before)
- [ ] Regex pattern matching for messages
- [ ] Streaming output (avoid full buffer in memory)
- [ ] Support for additional log levels
- [ ] Multi-file processing with aggregation
- [ ] Output format options (JSON, CSV)
- [ ] Performance optimization for gigabyte-scale files

## Requirements Met

✅ Reads logs.txt  
✅ Ignores invalid lines gracefully  
✅ Filters by --level and --service (optional)  
✅ Writes to output file (configurable --out)  
✅ Prints concise summary statistics  
✅ Handles case-insensitive level input  
✅ Validates log format strictly  

## License

Educational project for CC312 course

## Author

Ali AlQassab

---

**Last Updated**: February 2026
