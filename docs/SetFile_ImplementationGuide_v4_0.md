# Set File Implementation Guide
**Version 4.0**  
**Updated: November 2025**

---

## License

[![CC BY 4.0][cc-by-shield]][cc-by]

This guide is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

**Copyright (c) 2025 Kirk Siqveland**

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose, even commercially

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made

Full license text: https://creativecommons.org/licenses/by/4.0/

Implementations of this specification may use any license of the implementer's choosing.

[cc-by]: https://creativecommons.org/licenses/by/4.0/
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg

---

## About This Guide

This document provides comprehensive guidance for implementing Set file parsers, query languages, and extended functionality. These are **recommendations and patterns** for building Set-based systems, not requirements of the core specification.

**For the core format specification, see:**  
[Set File Format Specification v4.0](SetFile_Spec_v4_0.md)

---

## Table of Contents

1. [Query Language (SetQL)](#1-query-language-setql)
2. [Implementation Patterns & Conventions](#2-implementation-patterns--conventions)
3. [SetTag Extensions](#3-settag-extensions)
4. [CRUD Operations](#4-crud-operations)
5. [Validation & Error Handling](#5-validation--error-handling)
6. [Programming Interface Guidelines](#6-programming-interface-guidelines)
7. [Complete Examples](#7-complete-examples)
8. [Version History & Migration](#8-version-history--migration)

---

## 1. Query Language (SetQL)

SetQL provides a simple query syntax for filtering and selecting data from Set files. This is an **optional feature** - implementations may choose whether to support it.

### 1.1 Basic Syntax

```
FROM [GroupName] WHERE field=value
FROM [GroupName] SELECT field1,field2
FROM [GroupName] WHERE field>value ORDER BY field
```

### 1.2 Supported Operations

**Comparison Operators:**
- `=` Equal to
- `!=` Not equal to
- `>` Greater than
- `<` Less than
- `>=` Greater than or equal
- `<=` Less than or equal

**Pattern Matching:**
- `LIKE` Pattern matching (use `%` as wildcard)

**List Operations:**
- `IN` Value in list

**Logical Operators:**
- `AND` Combine conditions
- `OR` Alternative conditions

### 1.3 Query Components

**FROM clause** - Specifies the group to query
```
FROM [USERS]
FROM [DATABASE_CONFIG]
```

**WHERE clause** - Filters records
```
WHERE role='admin'
WHERE age>18 AND status='active'
WHERE email LIKE '%@example.com'
```

**SELECT clause** - Chooses specific fields
```
SELECT username,email
SELECT *
```

**ORDER BY clause** - Sorts results
```
ORDER BY last_name
ORDER BY age DESC
```

### 1.4 Examples

**Simple queries:**
```
FROM [USERS] WHERE role='admin'
FROM [PRODUCTS] WHERE price>100
FROM [SETTINGS] WHERE key LIKE 'Email%'
```

**Complex queries:**
```
FROM [EMPLOYEES] WHERE department='Engineering' AND salary>75000 ORDER BY hire_date
FROM [ORDERS] WHERE status IN ('pending','processing') AND total>500
FROM [CONTACTS] WHERE (city='Seattle' OR city='Portland') AND active=true
```

**With SELECT:**
```
FROM [USERS] SELECT username,email WHERE role='user'
FROM [PRODUCTS] SELECT name,price WHERE category='Electronics' ORDER BY price DESC
```

### 1.5 Text Block References

Text blocks are not directly queryable, but references are resolved in query results:

```
FROM [ARTICLES] SELECT title,body
```

If the `body` field contains `[{ARTICLE_1_BODY}]`, the query result includes the resolved text content.

### 1.6 Implementation Notes

- Query language is case-sensitive for field names and values
- String values should be quoted with single quotes
- Numeric values do not need quotes
- Boolean values: `true` / `false` (lowercase)
- NULL values: use empty string or special null handling
- Implementations may extend with additional operators or functions

---

## 2. Implementation Patterns & Conventions

The Set file format's simple structure enables many powerful patterns through creative use of existing features. These are **conventions, not requirements** - implementations choose what makes sense for their use cases.

### 2.1 Hierarchical Data via Dot Notation

Use dots in key names to represent nested structures.

```
[DATABASE_CONFIG]
host|localhost
port|5432
connection.pool.min|5
connection.pool.max|20
connection.timeout|30
ssl.enabled|true
ssl.cert|/path/to/cert.pem
[EOG]
```

**Simple parser:** Treats `connection.pool.min` as a single key  
**Advanced parser:** Builds nested structure:
```javascript
{
  host: "localhost",
  connection: {
    pool: { min: 5, max: 20 },
    timeout: 30
  },
  ssl: {
    enabled: true,
    cert: "/path/to/cert.pem"
  }
}
```

### 2.2 Runtime Calculation Fields (Implementation Pattern)

Some implementations may choose to support calculated fields that are generated at parse time rather than stored in the file.

**Convention:** Use `::` prefix to mark calculated fields in field definitions.

**Example:**
```
[SALES]
{date|product|amount|::tax|::total}
2025-01-01|Widget A|100.00
2025-01-02|Widget B|150.00
2025-01-03|Widget C|200.00
```

**Parser behavior:**
When the parser encounters `::tax` and `::total`, it:
1. Calculates tax (e.g., `amount * 0.08`)
2. Calculates total (e.g., `amount + tax`)
3. Adds these fields to the returned data structure

**Returned data might look like:**
```javascript
[
  {date: "2025-01-01", product: "Widget A", amount: "100.00", tax: "8.00", total: "108.00"},
  {date: "2025-01-02", product: "Widget B", amount: "150.00", tax: "12.00", total: "162.00"},
  {date: "2025-01-03", product: "Widget C", amount: "200.00", tax: "16.00", total: "216.00"}
]
```

**Notes:**
- This is **not part of the Set file format specification**
- It's a convention some implementations may choose to support
- The calculation logic is entirely implementation-specific
- The `::` prefix is just a convention to distinguish calculated from stored fields
- Simple parsers can ignore `::` fields or treat them as documentation

**Use cases:**
- Computed totals, subtotals, running totals
- Calculated dates (e.g., expiration date from creation date)
- Derived values (e.g., full name from first + last name)
- Display formatting (e.g., formatted currency from raw numbers)

### 2.3 Type Hints via Key Conventions

Add type information through naming conventions.

**Suffix notation:**
```
[SETTINGS]
maxUsers_int|50
timeout_float|30.5
debugMode_bool|true
database_null|
tags_array|development,testing,production
[EOG]
```

**Colon notation:**
```
[SETTINGS]
maxUsers:int|50
timeout:float|30.5
debugMode:bool|true
[EOG]
```

**Value prefixes:**
```
[SETTINGS]
maxUsers|i:50
timeout|f:30.5
debugMode|b:true
[EOG]
```

Choose whatever convention fits your implementation.

### 2.4 Arrays and Lists

**Horizontal arrays (positional fields):**
```
[COLORS]
{name|red|green|blue}
Primary Red|255|0|0
Sky Blue|135|206|235
Forest Green|34|139|34
[EOG]
```

**Vertical arrays (repeated keys):**
```
[ALLOWED_IPS]
ip|192.168.1.1
ip|192.168.1.2
ip|192.168.1.3
ip|10.0.0.5
[EOG]
```

**Comma-separated lists in values:**
```
[USER_ROLES]
admin|create,read,update,delete
editor|read,update
viewer|read
[EOG]
```

### 2.5 Version Suffixes

Use version numbers in group names for managing configuration versions:

```
[DATABASE_V1]
host|localhost
port|3306
[EOG]

[DATABASE_V2]
host|db.example.com
port|5432
pool_size|20
[EOG]
```

### 2.6 Environment-Specific Configurations

```
[DATABASE_PRODUCTION]
host|prod.db.example.com
port|5432
[EOG]

[DATABASE_STAGING]
host|staging.db.example.com
port|5432
[EOG]

[DATABASE_DEVELOPMENT]
host|localhost
port|5432
[EOG]
```

### 2.7 Base64 Encoding for Binary Data

Store binary data as Base64-encoded text in text blocks:

```
[APP_CONFIG]
icon|[{APP_ICON_BASE64}]
certificate|[{SSL_CERT_BASE64}]
[EOG]

[{APP_ICON_BASE64}]
iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==
[EOG]

[{SSL_CERT_BASE64}]
MIIDXTCCAkWgAwIBAgIJAKL0UG+mRfQNMA0GCSqGSIb3DQEBCwUAMEUxCzAJBgNV
BAYTAkFVMRMwEQYDVQQIDApTb21lLVN0YXRlMSEwHwYDVQQKDBhJbnRlcm5ldCBX
[EOG]
```

### 2.8 External File References

Reference external files for large binary data or shared resources:

```
[MEDIA_FILES]
logo|file://assets/logo.png
manual|file://docs/user-manual.pdf
dataset|file://data/large-dataset.csv
video|https://cdn.example.com/intro-video.mp4
[EOG]
```

Parser can fetch these references as needed.

### 2.9 Schema and Validation Patterns

Define expected structure in comments or dedicated groups:

```
This file follows the User schema v2.0
Expected fields: id, username, email, created_date

[USERS]
{id|username|email|created_date}
1|alice|alice@example.com|2025-01-15
2|bob|bob@example.com|2025-02-20
[EOG]
```

Or use a dedicated schema group:

```
[{SCHEMA_USERS}]
Fields: id (int), username (string), email (string), created_date (date)
Required: id, username, email
Unique: username, email
[EOG]

[USERS]
{id|username|email|created_date}
1|alice|alice@example.com|2025-01-15
[EOG]
```

### 2.10 Multi-Language Content

```
[APP_INFO]
name_en|My Application
name_es|Mi Aplicación
name_fr|Mon Application
description_en|[{DESC_EN}]
description_es|[{DESC_ES}]
description_fr|[{DESC_FR}]
[EOG]

[{DESC_EN}]
A powerful tool for managing your workflow.
[EOG]

[{DESC_ES}]
Una herramienta poderosa para gestionar tu flujo de trabajo.
[EOG]

[{DESC_FR}]
Un outil puissant pour gérer votre flux de travail.
[EOG]
```

### 2.11 Composed Structures

Reference other groups to build composite configurations:

```
[APPLICATION]
name|My Application
database_config|see:[DATABASE]
cache_config|see:[CACHE]
[EOG]

[DATABASE]
host|localhost
port|5432
database|myapp
[EOG]

[CACHE]
provider|redis
host|cache.example.com
ttl|3600
[EOG]
```

Implementations can choose how to resolve these references.

### 2.12 Special Character Representation (Implementation Pattern)

Since Set files use UTF-8 encoding by default, most Unicode characters can be included directly as literals. However, some implementations may choose to support escape-style character representation methods for special cases.

**UTF-8 Direct Input (Recommended):**

The preferred method is to use UTF-8 characters directly:

```
[MESSAGES]
Welcome|Café ☕
Greeting|你好世界
Symbols|★ ♥ ✓
Math|π ≈ 3.14159
Arrows|← → ↑ ↓
Emoji|😀 🎉 ✨
[EOG]
```

**Optional Character Representation Methods:**

Some implementations may choose to support these **optional** representation patterns:

**Line Break Representations:**
- `\n` - Line feed (LF)
- `\r` - Carriage return (CR)
- `\t` - Tab character

```
[DATA]
MultiLine|Line 1\nLine 2\nLine 3
Tabbed|Column1\tColumn2\tColumn3
[EOG]
```

**Note:** Text blocks are the **recommended** way to handle multi-line content. These representations are only useful if you need to embed line breaks in single-field values.

**Unicode Code Point Representations:**
- `\uXXXX` - Unicode character (4 hex digits)
- `\UXXXXXXXX` - Unicode character (8 hex digits)

```
[SYMBOLS]
CheckMark|\u2713
Copyright|\u00A9
Emoji|\U0001F600
[EOG]
```

**When to Use These Representations:**

1. **Rarely needed** - UTF-8 supports direct input of these characters
2. **Legacy systems** - Interfacing with systems that expect escaped characters
3. **Restricted input** - When your text editor doesn't support UTF-8 input
4. **Documentation** - Making character codes explicit in examples

**Implementation Notes:**

- These are **not part of the core specification**
- Simple parsers can ignore these entirely
- Advanced parsers may choose to support them
- There is no requirement to support these
- If supporting, document which patterns you recognize
- Unrecognized patterns should be treated as literal text

**Recommendation:**

For new implementations:
1. Rely on UTF-8 direct input (works for 99% of cases)
2. Use text blocks for multi-line content
3. Only add character representation support if needed for your specific use case

---

## 3. SetTag Extensions

SetTags allow Set file data to be embedded within other file formats (HTML, XML, source code comments, etc.). This is an **optional feature** for specialized use cases.

### 3.1 Syntax

```
{SETTAG:NAME}
[Set file content here]
{/SETTAG:NAME}
```

SetTags can contain any valid Set file structure, including configuration groups, data groups, and text blocks.

### 3.2 SetTag Naming Rules

SetTag names must:
- Match pattern: `[A-Z][A-Z0-9_]*`
- Be unique within the document
- Cannot be empty
- Start with uppercase letter

Valid: `CONFIG`, `USER_DATA`, `REPORT_2025`  
Invalid: `config`, `user-data`, `123DATA`

### 3.3 Example in HTML

```html
<!DOCTYPE html>
<html>
<head><title>Application Report</title></head>
<body>

<!--
{SETTAG:CONFIG}
config.set

[THIS-FILE]
Version|4.0
[EOG]

[SETTINGS]
Theme|Dark
Language|en-US
[EOG]

[{WELCOME_MESSAGE}]
Welcome to our application!
This is a multi-line welcome message.
[EOG]

[EOF]
{/SETTAG:CONFIG}
-->

<h1>Application Report</h1>

<script>
// Configuration can be extracted from SetTag
const config = parseSetTag('CONFIG');
</script>

</body>
</html>
```

### 3.4 Example in Source Code

```python
# Application configuration

"""
{SETTAG:APP_CONFIG}
app_config.set

[DATABASE]
host|localhost
port|5432
database|myapp
[EOG]

[CACHE]
provider|redis
ttl|3600
[EOG]
{/SETTAG:APP_CONFIG}
"""

def load_config():
    settag_data = extract_settag('APP_CONFIG')
    return parse_set_file(settag_data)
```

### 3.5 Example in XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<report>
  <metadata>
    <![CDATA[
{SETTAG:REPORT_DATA}
report.set

[REPORT_INFO]
Title|Q4 2025 Financial Summary
Author|Finance Team
Date|2025-11-27
[EOG]

[{EXECUTIVE_SUMMARY}]
Q4 2025 showed strong performance across all regions.
Revenue up 15% year-over-year.
[EOG]
{/SETTAG:REPORT_DATA}
    ]]>
  </metadata>
  <content>...</content>
</report>
```

### 3.6 Use Cases

**Configuration embedding:** Store application settings within HTML/XML documents

**Self-documenting code:** Embed structured metadata within source code

**Report data:** Include both data and presentation in single HTML file

**Version control friendly:** SetTags in comments preserve file functionality while adding structured data

**Template systems:** Embed configuration data alongside templates

### 3.7 Implementation Notes

- SetTag delimiters `{SETTAG:NAME}` and `{/SETTAG:NAME}` should not appear within the Set file content
- Multiple SetTags can exist in the same document
- SetTags should be properly closed
- Parsing SetTags is independent of parsing the host file format
- Consider namespace collisions if host format uses similar brace syntax

---

## 4. CRUD Operations

Set files support standard Create, Read, Update, and Delete operations. These are **implementation guidelines** - the specific API design is up to each implementation.

### 4.1 Create Operations

**Add new group:**
- Append to file before `[EOF]` (if present)
- Ensure unique group name
- Use appropriate group type syntax

**Add record to regular group:**
- Append line matching field structure
- Ensure field count matches definition
- Escape special characters as needed

**Add key-value pair:**
- Append `Key|Value` line to group
- Check for duplicate keys (implementation choice: error, warning, or allow)
- Handle text block references if needed

**Add text block:**
- Create new `[{NAME}]` group with content
- Ensure unique name
- No escaping needed for content

**Example:**
```python
# Pseudocode
file.add_group("[USERS]", type="regular")
file.add_record("[USERS]", ["3", "charlie", "charlie@example.com"])

file.add_group("[SETTINGS]", type="keyvalue")
file.add_keyvalue("[SETTINGS]", "Theme", "Dark")

file.add_textblock("[{LICENSE}]", "MIT License\nCopyright (c) 2025...")
```

### 4.2 Read Operations

**Parse file:**
- Respect group type syntax
- Process delimiters and escapes appropriately
- Resolve text block references when encountered

**Return data structures:**
- Regular groups: Arrays of arrays/objects with field names
- Key-value groups: Objects/maps/dictionaries
- Text blocks: Strings with raw content

**Example:**
```python
# Pseudocode
data = parse_set_file("config.set")

# Regular group
users = data["USERS"]  # [{id: "1", name: "alice", email: "..."}, ...]

# Key-value group  
settings = data["SETTINGS"]  # {Theme: "Dark", Language: "en-US"}

# Text block
license = data["{LICENSE}"]  # "MIT License\nCopyright..."
```

### 4.3 Update Operations

**Modify existing records:**
- Update in-place or create new version
- Maintain field structure
- Preserve escape sequences

**Update key-value pairs:**
- Match by key name
- Replace value
- Handle text block references

**Replace text block content:**
- Replace entire content between group markers
- No escaping needed

**Example:**
```python
# Pseudocode
file.update_record("[USERS]", where={"id": "1"}, data={"email": "newemail@example.com"})
file.update_keyvalue("[SETTINGS]", "Theme", "Light")
file.update_textblock("[{LICENSE}]", "New license text here...")
```

### 4.4 Delete Operations

**Remove records from groups:**
- Identify by field values or position
- Maintain group structure

**Remove entire groups:**
- Delete group and all contents
- Warn if text block is referenced elsewhere

**Delete key-value pairs:**
- Match by key name
- Remove entire line

**Remove text block groups:**
- Check for references first
- Warn or error if referenced

**Example:**
```python
# Pseudocode
file.delete_record("[USERS]", where={"id": "3"})
file.delete_keyvalue("[SETTINGS]", "DebugMode")
file.delete_group("[OLD_CONFIG]")
file.delete_textblock("[{UNUSED_TEXT}]")
```

### 4.5 Special Considerations

**Text block references:**
- When deleting a text block, check if it's referenced elsewhere
- Warn or prevent deletion if references exist
- Alternatively, offer to delete references or replace with literal text

**Validation:**
- Validate unique group names on create
- Validate field count on record create/update
- Validate text block references exist

**Atomic operations:**
- Consider implementing transactions for multi-operation changes
- Provide rollback capability for failed operations
- Maintain file integrity during writes

**Circular references:**
- Detect before write operations
- Prevent creation of circular references
- Text blocks cannot reference other text blocks

**File locking:**
- Implement appropriate locking for concurrent access
- Consider read locks vs write locks
- Handle lock failures gracefully

---

## 5. Validation & Error Handling

### 5.1 Group Name Validation

**Rules:**
- Must be unique within file
- Only letters, numbers, hyphens, underscores
- No spaces
- Cannot be empty
- Cannot use reserved names: `EOG`, `EOF`

**Validation:**
```
Valid: USERS, User_Data, CONFIG-V2, Database123
Invalid: User Data (space), [USERS] (contains brackets), "" (empty), EOG (reserved)
```

### 5.2 Regular Group Validation

**Field definitions:**
- Field names must not be empty
- Field names follow same rules as group names
- Recommend using field definition line `{field1|field2|...}`

**Data records:**
- Field count must match field definition (if present)
- Empty fields are valid (represented as `||`)
- Single-use fields `:::` can appear at end of data lines

**Errors to detect:**
```
- Mismatched field count
- Invalid field names
- Malformed single-use field syntax (:::)
```

### 5.3 Key-Value Group Validation

**Keys:**
- Must not be empty
- Must follow field naming rules (letters, numbers, hyphens, underscores)
- Case-sensitive

**Values:**
- Can be empty
- Can contain any characters (with proper escaping)
- Can reference text blocks: `[{NAME}]`

**Warnings:**
- Duplicate keys within same group (implementation choice: allow, warn, or error)

### 5.4 Text Block Group Validation

**Group names:**
- Follow standard naming rules
- Must be unique
- Use `[{NAME}]` syntax

**Content:**
- Cannot contain field definitions
- Cannot reference other text blocks (no nesting)
- All content is literal - no validation needed
- Should not contain lines that look like group markers (flag as warning)

**References:**
- Text block references `[{NAME}]` must point to existing text block groups
- No circular references allowed
- No nested references allowed

### 5.5 Delimiter and Escape Validation

**Delimiter definition:**
- Must follow format: `preamble:component:component:...`
- All components must be defined
- Preamble delimiter must be single character (or consistent)

**Escape sequences:**
- `\|` for delimiter escaping
- `\\` for backslash escaping
- Unrecognized escape sequences should warn or error

**Common errors:**
```
- Unescaped delimiter in data
- Unclosed escape sequence at end of line
- Invalid escape character combinations
```

### 5.6 File Structure Validation

**Preamble/configuration:**
- `[THIS-FILE]` group should appear before data groups (convention, not requirement)
- Delimiter definition must be parseable
- Encoding must be recognized

**Group markers:**
- Must be complete: `[NAME]` not `[NAME`
- Must be on own line
- Must not appear mid-line in data

**End markers:**
- `[EOG]` is optional but recommended
- `[EOF]` is optional
- Multiple consecutive empty lines are allowed

### 5.7 Error Reporting

**Structured error information should include:**
- Error type (validation, parse, reference, etc.)
- Line number where error occurred
- Column/position if relevant
- Expected vs actual values
- Suggested fix

**Error types:**

**Validation Errors:**
```
- Invalid group name
- Duplicate group name
- Invalid field name
- Field count mismatch
- Invalid key name
- Duplicate key (if configured as error)
```

**Parse Errors:**
```
- Malformed group marker
- Unclosed text block
- Invalid delimiter definition
- Unrecognized escape sequence
```

**Reference Errors:**
```
- Text block reference to non-existent group
- Circular reference detected
- Nested text block reference
```

**Operation Errors:**
```
- CRUD operation on non-existent group
- Invalid data type for operation
- File access/permission errors
```

**Example error format:**
```
Error: Field count mismatch
Line: 42
Expected: 4 fields (id, name, email, role)
Actual: 3 fields
Data: 5|Alice|alice@example.com
Suggestion: Add missing 'role' field or update field definition
```

### 5.8 Warning Conditions

**Non-fatal issues that should generate warnings:**
- Duplicate keys in key-value groups (if warnings mode)
- Text block content contains group marker patterns
- Trailing whitespace in values (unless intentional with `\_`)
- Very long lines (potential performance issue)
- Unreferenced text blocks (potential unused data)
- Missing field definition in regular groups
- Use of deprecated features (implementation-specific)

### 5.9 Validation Levels

**Implementations may offer different validation levels:**

**Strict mode:**
- All violations are errors
- Duplicate keys not allowed
- Field definitions required
- All references must resolve

**Standard mode:**
- Core violations are errors
- Some issues are warnings
- Duplicate keys allowed with warning
- Missing field definitions generate warnings

**Lenient mode:**
- Minimal validation
- Accept files with warnings
- Best-effort parsing
- Useful for importing from other formats

---

## 6. Programming Interface Guidelines

### 6.1 Minimal Parser Design (Q-Set Approach)

A minimal Set file parser can be implemented in approximately 50 lines of code. Here's the conceptual approach:

**Core functionality:**
1. Read file line by line
2. Detect group markers: `[NAME]`, `[{NAME}]`
3. For regular groups: split lines on delimiter
4. For text blocks: collect raw content
5. Store in appropriate data structure

**Pseudocode:**
```python
def parse_set_file(filename):
    groups = {}
    current_group = None
    current_type = None
    content = []
    
    for line in read_file(filename):
        if line.startswith('[{') and line.endswith('}]'):
            # Text block group
            save_previous_group()
            current_group = extract_group_name(line)
            current_type = 'textblock'
            content = []
        elif line.startswith('[') and line.endswith(']'):
            # Regular group
            save_previous_group()
            current_group = extract_group_name(line)
            current_type = 'regular'
            content = []
        elif is_empty(line):
            # End of group (implicit EOG)
            save_previous_group()
        else:
            # Data line
            content.append(line)
    
    return groups
```

**This handles:**
- ✓ Group detection
- ✓ Text blocks
- ✓ Implicit EOG
- ✓ Basic parsing

**Not included (can be added incrementally):**
- Field definitions
- Escape sequences
- Text block references
- Validation
- Special functions

### 6.2 Full-Featured Parser Design

**Components:**

**1. Lexer/Tokenizer:**
- Tokenize input into groups, markers, data lines
- Handle escape sequences
- Process delimiters

**2. Parser:**
- Build data structures from tokens
- Detect group types
- Validate syntax

**3. Resolver:**
- Resolve text block references
- Detect circular references
- Cache resolved content

**4. Validator:**
- Validate group names
- Check field counts
- Verify references exist

**5. Serializer:**
- Convert data structures back to Set file format
- Apply escape sequences
- Format output

### 6.3 API Design Patterns

**Object-oriented approach:**
```python
class SetFile:
    def __init__(self, filename):
        self.filename = filename
        self.groups = {}
    
    def load(self):
        # Parse file
        
    def save(self):
        # Write file
        
    def get_group(self, name):
        # Retrieve group data
        
    def add_group(self, name, type):
        # Create new group
        
    def delete_group(self, name):
        # Remove group
        
class Group:
    def __init__(self, name, type):
        self.name = name
        self.type = type  # 'regular', 'textblock'
        self.data = []
        
    def add_record(self, record):
        # Add data
        
    def update_record(self, index, record):
        # Modify data
```

**Functional approach:**
```javascript
// Functional API
const file = parseSetFile('config.set');
const users = getGroup(file, 'USERS');
const settings = getGroup(file, 'SETTINGS');

const updated = addRecord(file, 'USERS', {id: 3, name: 'charlie'});
const saved = saveSetFile(updated, 'config.set');
```

**Fluent/chaining approach:**
```javascript
SetFile.load('config.set')
  .addGroup('USERS')
  .addRecord('USERS', {id: 1, name: 'alice'})
  .addKeyValue('SETTINGS', 'Theme', 'Dark')
  .save();
```

### 6.4 Data Structure Recommendations

**Regular groups:**
```python
# Array of objects
[
  {id: "1", name: "alice", email: "alice@example.com"},
  {id: "2", name: "bob", email: "bob@example.com"}
]

# Or array of arrays (if field definition exists)
[
  ["1", "alice", "alice@example.com"],
  ["2", "bob", "bob@example.com"]
]
```

**Key-value groups:**
```python
# Object/Map/Dictionary
{
  "Theme": "Dark",
  "Language": "en-US",
  "MaxUsers": "50"
}
```

**Text blocks:**
```python
# String
"MIT License\n\nCopyright (c) 2025..."
```

### 6.5 Memory Management

**For small files (< 1MB):**
- Read entire file into memory
- Parse completely
- Return full data structure

**For large files (> 1MB):**
- Stream parsing line by line
- Lazy load groups on demand
- Cache frequently accessed groups
- Provide iterator interface for large groups

**Example streaming approach:**
```python
class SetFileStream:
    def iter_group(self, group_name):
        # Generator that yields records one at a time
        for record in self._stream_group(group_name):
            yield record
```

### 6.6 Caching Strategies

**Text block reference caching:**
```python
# Cache resolved text blocks
text_block_cache = {}

def resolve_reference(ref_name):
    if ref_name not in text_block_cache:
        text_block_cache[ref_name] = load_text_block(ref_name)
    return text_block_cache[ref_name]
```

**Group caching:**
- Cache parsed groups to avoid re-parsing
- Invalidate cache on file modification
- Use LRU cache for large file sets

### 6.7 Concurrency Considerations

**Read operations:**
- Multiple concurrent readers are safe
- No locking required for read-only access

**Write operations:**
- Implement file locking for writes
- Use atomic write patterns (write to temp, then rename)
- Queue write operations if needed

**Example atomic write:**
```python
def save_set_file(data, filename):
    temp_file = filename + '.tmp'
    write_to_file(data, temp_file)
    atomic_rename(temp_file, filename)
```

### 6.8 Error Handling Patterns

**Return error codes:**
```c
int parse_set_file(const char* filename, SetFile** result) {
    if (file_not_found(filename)) return ERROR_FILE_NOT_FOUND;
    if (parse_failed()) return ERROR_PARSE_FAILED;
    return SUCCESS;
}
```

**Exceptions:**
```python
try:
    file = SetFile.load('config.set')
except SetFileNotFoundError:
    # Handle missing file
except SetFileParseError as e:
    # Handle parse error, e.line_number available
```

**Result types:**
```rust
fn parse_set_file(filename: &str) -> Result<SetFile, SetFileError> {
    // Returns Ok(SetFile) or Err(SetFileError)
}
```

### 6.9 Testing Recommendations

**Unit tests should cover:**
- Empty files
- Files with only groups, no data
- Files with text blocks only
- Mixed group types
- Escape sequences in data
- Text block references
- Missing references (error case)
- Malformed group markers
- Edge cases (very long lines, unusual delimiters)

**Integration tests:**
- Read-modify-write cycles
- Concurrent access scenarios
- Large file handling
- Different encodings (UTF-8, UTF-16)

### 6.10 Performance Optimization

**Parsing:**
- Use buffered I/O for file reading
- Minimize string allocations
- Pre-compile regex patterns
- Use efficient data structures (hash maps for groups)

**Serialization:**
- Buffer writes
- Minimize string concatenation
- Use string builders
- Batch write operations

**Benchmarking targets:**
- Small files (< 100 KB): < 10ms parse time
- Medium files (1-10 MB): < 100ms parse time
- Large files (> 10 MB): Consider streaming instead

---

## 7. Complete Examples

### 7.1 Minimal Configuration File

```
myapp.set

[DATABASE]
Host|localhost
Port|5432
Database|myapp
User|admin
Password|secret123
[EOG]

[APP_SETTINGS]
Theme|dark
Language|en-US
MaxUsers|50
DebugMode|false
[EOG]

[EOF]
```

### 7.2 Configuration with Text Blocks

```
application.set

[THIS-FILE]
Version|4.0
Created|2025-11-27
Author|Kirk Siqveland
[EOG]

[APP_INFO]
Name|My Application
Version|2.0.0
Description|[{APP_DESCRIPTION}]
License|[{LICENSE_TEXT}]
[EOG]

[DATABASE]
Host|prod.db.example.com
Port|5432
ConnectionPool|20
Timeout|30
[EOG]

[{APP_DESCRIPTION}]
My Application is a comprehensive workflow management tool.

Features:
- Task tracking and assignment
- Team collaboration
- Real-time synchronization
- Customizable workflows
- Advanced reporting

Perfect for teams of any size!
[EOG]

[{LICENSE_TEXT}]
MIT License

Copyright (c) 2025 Kirk Siqveland

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
[EOG]

[EOF]
```

### 7.3 Structured Data with Regular Groups

```
employees.set

[THIS-FILE]
Version|4.0
Created|2025-11-27
[EOG]

[EMPLOYEES]
{id|first_name|last_name|department|hire_date|salary|email}
101|Alice|Smith|Engineering|2023-01-15|95000|alice.smith@example.com
102|Bob|Jones|Marketing|2023-02-20|75000|bob.jones@example.com
103|Carol|White|Engineering|2023-03-10|98000|carol.white@example.com
104|David|Brown|Sales|2023-04-05|82000|david.brown@example.com
105|Eve|Davis|Engineering|2023-05-12|92000|eve.davis@example.com
[EOG]

[DEPARTMENTS]
{id|name|manager|budget}
1|Engineering|Alice Smith|500000
2|Marketing|Bob Jones|250000
3|Sales|David Brown|350000
[EOG]

[EOF]
```

### 7.4 Multi-Language Application

```
i18n_app.set

[THIS-FILE]
Version|4.0
Localize|NFC|multi|AUTO
[EOG]

[APP_INFO_EN]
AppName|Global Connect
Tagline|Connect with the world
WelcomeMessage|[{WELCOME_EN}]
HelpText|[{HELP_EN}]
[EOG]

[APP_INFO_ES]
AppName|Conexión Global
Tagline|Conecta con el mundo
WelcomeMessage|[{WELCOME_ES}]
HelpText|[{HELP_ES}]
[EOG]

[APP_INFO_FR]
AppName|Connexion Mondiale
Tagline|Connectez-vous au monde
WelcomeMessage|[{WELCOME_FR}]
HelpText|[{HELP_FR}]
[EOG]

[{WELCOME_EN}]
Welcome to Global Connect!

Start connecting with people around the world in your language.
Share ideas, collaborate, and build meaningful connections.
[EOG]

[{WELCOME_ES}]
¡Bienvenido a Conexión Global!

Comienza a conectarte con personas de todo el mundo en tu idioma.
Comparte ideas, colabora y construye conexiones significativas.
[EOG]

[{WELCOME_FR}]
Bienvenue sur Connexion Mondiale !

Commencez à vous connecter avec des gens du monde entier dans votre langue.
Partagez des idées, collaborez et créez des liens significatifs.
[EOG]

[{HELP_EN}]
Getting Started:
1. Create your profile
2. Add your interests
3. Start connecting

Need help? Contact support@globalconnect.com
[EOG]

[{HELP_ES}]
Primeros pasos:
1. Crea tu perfil
2. Añade tus intereses
3. Comienza a conectar

¿Necesitas ayuda? Contacta support@globalconnect.com
[EOG]

[{HELP_FR}]
Pour commencer :
1. Créez votre profil
2. Ajoutez vos intérêts
3. Commencez à vous connecter

Besoin d'aide ? Contactez support@globalconnect.com
[EOG]

[EOF]
```

### 7.5 Advanced Features Example

```
advanced.set

[THIS-FILE]
Version|4.0
Delimiters|:[]:{}:|:\:…:
Encode|UTF-8
Localize|NFC|en-US|LTR
Created|2025-11-27
[EOG]

Example of advanced Set file features including:
- Single-use fields (:::)
- Ellipsis shorthand
- Single-line delimiter override
- Text block references
- Runtime calculation pattern (::) for demonstration

[SALES_DATA]
{date|product|amount|tax_rate|::calculated_tax|::total}
2025-01-01|Widget A|100.00|0.08
2025-01-02|Widget B|150.00|0.08
2025-01-03|Widget C|200.00|0.08

Note: The :: fields above are an implementation pattern (see Section 2.2).
A parser supporting this pattern would calculate tax and total at runtime.
[EOG]

[CONTACTS]
{id|name|email|phone|address|city|state|zip|notes}
1|Alice Johnson|alice@example.com|555-1234|…
2|Bob Smith|bob@example.com|555-5678|123 Main St|Seattle|WA|98101|…
3|Carol White|carol@example.com|555-9999|…|:::note:Call before 3pm
[EOG]

[API_ENDPOINTS]
users|/api/users
products|/api/products
:!complex_url!https://api.example.com/v2/search?query=test|value&sort=name|desc!GET
orders|/api/orders
[EOG]

[PROJECT_INFO]
Name|Advanced Demo
Description|[{PROJECT_DESC}]
Readme|[{PROJECT_README}]
[EOG]

[{PROJECT_DESC}]
This project demonstrates all advanced features of Set file format v4.0.

Includes:
- Single-use fields (:::) for per-record metadata
- Ellipsis shorthand for sparse data
- Single-line delimiter override for complex URLs
- Text block references for multi-line content
- Runtime calculation pattern (::) demonstration (implementation-specific)
[EOG]

[{PROJECT_README}]
# Advanced Demo Project

## Features Demonstrated

1. **Single-Use Fields (:::)**
   - Per-record notes
   - Ad-hoc metadata without modifying field definition

2. **Runtime Calculation Pattern (::)**
   - Implementation-specific feature (see Section 2.2)
   - Calculated tax amounts and totals
   - Not part of core spec, but a common convention

3. **Ellipsis Shorthand**
   - Sparse data representation
   - Reduced file size

4. **Single-Line Delimiter Override**
   - Complex URLs with multiple pipes
   - Data containing standard delimiter

## Usage

Parse this file with a Set file parser that supports v4.0 features.
For runtime calculations, your parser must implement the :: pattern.
[EOG]

[EOF]
```

### 7.6 Environment-Specific Configuration

```
env_config.set

[THIS-FILE]
Version|4.0
Environment|production
[EOG]

[DATABASE_PRODUCTION]
Host|prod-db-01.example.com
Port|5432
Database|myapp_prod
User|prod_user
Password|[{DB_PROD_PASSWORD}]
PoolSize|50
Timeout|30
SSL|true
[EOG]

[DATABASE_STAGING]
Host|staging-db.example.com
Port|5432
Database|myapp_staging
User|staging_user
Password|[{DB_STAGING_PASSWORD}]
PoolSize|20
Timeout|30
SSL|true
[EOG]

[DATABASE_DEVELOPMENT]
Host|localhost
Port|5432
Database|myapp_dev
User|dev_user
Password|dev_password
PoolSize|5
Timeout|60
SSL|false
[EOG]

[CACHE_PRODUCTION]
Provider|redis
Host|prod-cache-01.example.com
Port|6379
TTL|3600
MaxMemory|2GB
[EOG]

[CACHE_STAGING]
Provider|redis
Host|staging-cache.example.com
Port|6379
TTL|1800
MaxMemory|1GB
[EOG]

[CACHE_DEVELOPMENT]
Provider|memory
TTL|300
MaxMemory|100MB
[EOG]

[{DB_PROD_PASSWORD}]
<encrypted_password_here>
[EOG]

[{DB_STAGING_PASSWORD}]
<encrypted_password_here>
[EOG]

[EOF]
```

---

## 8. Version History & Migration

### Version 4.0 (November 2025) - Major Simplification

**Philosophy Change:**
Version 4.0 represents a fundamental shift toward simplicity and implementation flexibility. The format is simplified while maintaining backward compatibility with most v3.x files.

**Major Changes:**

1. **Removed Mandatory Preamble**
   - v3.x: Required 4-7 line preamble with specific format
   - v4.0: Optional `[THIS-FILE]` group for configuration
   - Benefit: Simpler files, easier to get started

2. **Eliminated Group Type Distinction**
   - v3.x: `[=KEYVALUE=]` syntax for key-value groups
   - v4.0: Just `[GROUPNAME]` - can contain positional or key-value data
   - Benefit: Less syntax to remember, cleaner files

3. **Removed Comment Block Syntax**
   - v3.x: `{|[COMMENT]|}` ... `{|[/COMMENT]|}`
   - v4.0: Text outside groups is inherently a comment
   - Benefit: Simpler, more natural documentation

4. **Added Features:**
   - Single-line delimiter override: `:!field!field!field`
   - Implicit EOG via empty lines (explicit `[EOG]` still allowed)
   - Clearer escape sequence rules (minimal: just `\|` and `\\`)

5. **Simplified Escape Sequences**
   - v3.x: Required escaping `[`, `]`, `{`, `}`, space markers
   - v4.0: Only escape delimiter `\|` and backslash `\\`
   - Benefit: Less escaping needed, more readable

**Migration from v3.x to v4.0:**

**Step 1: Update Preamble**

v3.x format:
```
filename.set
UTF-8
:[]:{}:|:\:…:
NFC|en-US|LTR


VERSION: 3.3
```

v4.0 format:
```
filename.set

[THIS-FILE]
Version|4.0
Delimiters|:[]:{}:|:\:…:
Encode|UTF-8
Localize|NFC|en-US|LTR
[EOG]
```

**Step 2: Update Group Names**

v3.x format:
```
[=SETTINGS=]
Key|Value
[EOG]
```

v4.0 format:
```
[SETTINGS]
Key|Value
[EOG]
```

Simply remove the `=` signs from group names.

**Step 3: Replace Comment Blocks**

v3.x format:
```
{|[NOTE]|}
This is a comment
{|[/NOTE]|}

[DATA]
```

v4.0 format:
```
This is a comment

[DATA]
```

Or use unreferenced text blocks:
```
[{NOTE}]
This is a comment
[EOG]

[DATA]
```

**Step 4: Simplify Escape Sequences**

v3.x: Required escaping brackets and braces
```
Expression|\[value\] in \{range\}
```

v4.0: Only escape delimiter and backslash
```
Expression|[value] in {range}
```

Unless the line starts with `[`, brackets don't need escaping.

**Automated Migration Script:**

```python
def migrate_v3_to_v4(v3_filename, v4_filename):
    lines = read_file(v3_filename)
    output = []
    
    # Convert preamble to [THIS-FILE] group
    if is_v3_preamble(lines[0:7]):
        output.append(lines[0])  # filename
        output.append("")         # blank line
        output.append("[THIS-FILE]")
        output.append(f"Version|4.0")
        if lines[1].strip():
            output.append(f"Encode|{lines[1]}")
        if lines[2].strip():
            output.append(f"Delimiters|{lines[2]}")
        if lines[3].strip():
            output.append(f"Localize|{lines[3]}")
        output.append("[EOG]")
        output.append("")
        lines = lines[7:]  # Skip preamble
    
    # Convert group names
    for line in lines:
        # Remove [=NAME=] syntax
        line = re.sub(r'\[=(.+)=\]', r'[\1]', line)
        
        # Remove comment blocks
        if '{|[' in line and '|}'  in line:
            continue  # Skip comment block markers
            
        output.append(line)
    
    write_file(v4_filename, output)
```

**Backward Compatibility:**

v4.0 parsers **can** read most v3.x files with these caveats:
- Preamble must be converted to `[THIS-FILE]` group
- Comment blocks are not supported (but can be converted to text outside groups)
- `[=NAME=]` syntax works but is deprecated

v3.x parsers **cannot** reliably read v4.0 files that use:
- `[THIS-FILE]` group instead of preamble
- Text outside groups as comments
- Single-line delimiter override

**Recommendation:** When creating new files, use v4.0 format. When maintaining legacy files, consider migrating to v4.0 for simplicity.

---

### Version 3.3 (November 2025)

- Clarified progressive preamble definition
- Standardized group naming rules
- Updated `[EOG]` and `[EOF]` markers to optional
- Enhanced documentation

### Version 3.2 (November 2025)

- Added key-value groups `[=NAME=]`
- Added text block groups `[{NAME}]`
- Added text block reference system
- Enhanced validation rules

### Version 3.0 (September 2025)

- Added special functions (ellipses, single-use fields `:::`)
- Enhanced internationalization support
- Improved escape character handling
- Added SetQL query language

### Version 2.0

- Core format specification
- Escape sequences
- Comment blocks
- SetTag extensions

---

### Migration Best Practices

**When to migrate:**
- Creating new Set files → Use v4.0
- Simple v3.x files → Easy to migrate
- Complex v3.x files with many comment blocks → Evaluate benefits
- Production systems → Test thoroughly before migration

**Testing migration:**
1. Back up original files
2. Run migration script
3. Parse both versions with v4.0 parser
4. Compare data structures
5. Validate all references resolve
6. Test with your application

**Gradual migration:**
- Migrate configuration files first (simplest)
- Then data files
- Finally, complex files with many text blocks
- Keep v3.x files until v4.0 versions are validated

---

*End of Set File Format Specification v4.0*

**Questions or feedback?**  
Visit: https://github.com/kirksiqveland/setfile

**License:**  
Creative Commons Attribution 4.0 International (CC BY 4.0)  
Copyright (c) 2025 Kirk B Siqveland

