# Set File Format Specification
**Version 4.0**  
**Updated: November 2025**

---

## License

[![CC BY 4.0][cc-by-shield]][cc-by]

This specification is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

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

## Table of Contents

1. [Introduction & Philosophy](#1-introduction--philosophy)
2. [Minimum Core Specification](#2-minimum-core-specification)
3. [File Configuration](#3-file-configuration)
4. [Group Types in Detail](#4-group-types-in-detail)
5. [Optional Advanced Features](#5-optional-advanced-features)

**For implementation guidance, examples, and patterns, see:**  
[Set File Implementation Guide v4.0](SetFile_ImplementationGuide_v4_0.md)

---

# THE SPECIFICATION

## 1. Introduction & Philosophy

### What are Set files?

Set files (.set file extension) are machine-readable and human-readable data files designed for configuration files, structured data, and any scenario requiring both machine-parseability and human-editability.

Set files are used for storing variable format settings, configuration data, and structured information in a format that is easy for both humans to read and edit, and for programs to parse and process.

The primary convention with Set files is pipe-delimited fields, though the specification allows the pipe delimiter to be replaced with a different delimiter when needed or desired.

### Design Principles

**Human First**  
Set files prioritize human readability over parsing efficiency.

**Simple Specification**  
The core rules fit on a few pages. 
Complex features are optional extensions, not core requirements.

**Flexible Implementation**  
Parse as much or as little as you need. 
A minimal parser can be written in ~50 lines of code. Advanced features are available when needed.

**Convention Over Enforcement**  
The format enables patterns but doesn't mandate them. 
Implementations choose their own conventions to match their environment.

**No Magic**  
Everything is explicit and visible. No hidden behaviors, no surprising type coercion, no implicit conversions.


### When to Use Set files

**Good fits:**
- Configuration files
- Settings and preferences
- Structured data with mixed types (key-value + tables + text)
- Multi-line text content (licenses, descriptions, documentation)
- Human-editable data that needs version control
- CSV replacement with multiple distinct data sets, comments and where readability matters 

**Poor fits:**
- Real-time data streaming (use binary protocols)
- Deep nested hierarchies (consider JSON/XML)
- Large binary data (reference external files instead)
- Performance-critical parsing (use binary formats)

### File Extensions

- `.set`  - Standard Set file (may use any features)
- `.qset` - Suggests minimal/simple implementation (conventionally uses only sections 2-4)

**Note:** The "q" in `.qset` denotes a simplified or "quick" implementation, but functionally both extensions use the same format and specification.

### Implementation Flexibility

This specification defines the complete Set file format. Implementations may choose to support:

- **Minimum (Q-Set approach):** Sections 2-4 only
- **Standard** Sections 2-5
- **Full** Sections 2-5 plus optional implementation guidance features
- **Mix-and-Match** Implement only relevant sections as needed

---

## 2. Minimum Core Specification

This section defines the absolute minimum requirements for a SET file and parser. Everything in this section is **required** for basic compliance.

### 2.1 File Structure

A Set file is organized into:

- **Preamble** (optional): containing information about the file and its contents
- **Group(s)**: delimited sections of the file containing related-format information
  - Each Group MUST have a unique name within the file
  - a Group might contain Name|Value pairs (an array of two items)
  - a Group might contain arrays of setting information as delimited lines of text
  - a Text Group may be used to store extensive text information (even multi-line)
  - a field in a line, may reference a Text Group
- **Comments**: Any text outside of a [GROUP] is considered a comment
  - Any comment immediately preceding a [GROUP] is assumed to be related to that group

Typically a Set file consists of:
- Optional but recommended, filename identifier on the first line
- Optional documentation/comments (text outside groups)
- Optional conventions used in the file, stored in a [THIS-FILE] group
- One or more [GROUPS] containing data
- Optional but encouraged End-of-Group markers: [EOG]
- Optional end-of-file marker [EOF]

**Example:**
```
myconfig.set

This file contains application configuration.
Created: 2025-11-27

[DATABASE]
Host|localhost
Port|5432

[APP_SETTINGS]
Theme|dark
Language|en-US

[PROTOCOL]
RS232|9600|8|N|1|Off
```

### 2.2 Groups

Groups are the fundamental data containers in Set files.

**Syntax:** `[GROUPNAME]`

**Naming Rules:**
- The line with the Group name must begin with the Group Name character "["
- No characters or spaces must exist before the Group Name character "["
- Only letters (a-z, A-Z), numbers (0-9), hyphens (-), and underscores (_)
- No spaces allowed in the name (use underscore or hyphen for word separation)
- ALL_CAPS is conventional but not required
- Must be unique within the file

**Examples:**
- `[DATABASE]` ✓
- `[App_Settings]` ✓
- `[USER-LIST]` ✓
- `[Config 2]` ✗ not good (contains space)
- `[My.Config]` ✗ not good (contains period)

### 2.3 Group Content

Groups contain data stored by line
- This can be seen as Name|Value pairs or
- More complex arrays of information
- The first line of a [GROUP] may contain field names enclosed in curly braces {}
- Field-names applay to all the content line of their [GROUP]
- Single-use fields may be applied at the end of a normal line (defined later in spec)
- A field may contain the name of a Text Group as a linked reference.

**Key-Value Pairs:**
```
[SETTINGS]
Key|Value
AnotherKey|Another Value
```

**Positional Fields:**
```
[USERS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com
```

Lines in a set file may end with a single LF character or a LF and a CR (line-feed and carriage return)
Group contents are always separated by the defined delimiter, typically a pipe - "|"

### 2.4 End of Group

Groups end when:
1. An empty line is encountered (implicit `[EOG]`)
2. An explicit `[EOG]` marker is present
3. Another group begins
4. End of file is reached

**Examples:**
```
[SETTINGS]
Key|Value

[ANOTHER_GROUP]
```

```
[SETTINGS]
Key|Value
[EOG]

[ANOTHER_GROUP]
```

Both examples are equivalent. The `[EOG]` marker is optional but recommended for clarity.

### 2.5 Text Blocks

Text blocks store multi-line content without any delimiter processing or escape sequences.

**Syntax:** `[{GROUPNAME}]`

**Content Rules:**
- All content between `[{GROUPNAME}]` and the end marker is preserved exactly
- No escape sequences are processed
- No delimiters are processed
- Every space, tab, blank line, and character is preserved
- Content ends at empty line, `[EOG]`, another group marker, or end of file

**Example:**
```
[{LICENSE_TEXT}]
MIT License

Copyright (c) 2025 Kirk Siqveland

Permission is hereby granted, free of charge...
[EOG]
```

### 2.6 Text Block References

Regular groups can reference text blocks using the syntax `[{GROUPNAME}]` as a value.

**Example:**
```
[APP_INFO]
Name|My Application
License|[{LICENSE_TEXT}]

[{LICENSE_TEXT}]
MIT License
Copyright (c) 2025...
```

When parsing, the value `[{LICENSE_TEXT}]` should be replaced with the content of the `[{LICENSE_TEXT}]` text block.

**Rules:**
- Referenced text block must exist in the same file
- No circular references allowed
- No nested references (text blocks cannot reference other text blocks)

### 2.7 Delimiters

**Default Delimiter:** `|` (pipe character, ASCII 124)

- Alternative delimiters can be defined using the 
  `Delimiter|:[]:{}:|:\:…:`  line in the [THIS-FILE] Group
- Example `Delimiter|:[]:{}:#:\:…:`  Notice # in place of |

By default the pipe character "|" separates:
- Fields in positional data (arrays)
- Keys from values in key-value pairs (two item arrays)
- Field names in field definitions  {First|Last|Middle|E-Mail}

**Do not begin or end lines with delimiters** - this would shift all field positions.

### 2.8 Escape Sequences

Escape sequences are **only needed in regular groups**, not in text blocks.

**The default Escape Character:** `\` (backslash)

**Primary Use:** Escape the field delimiter within data

**Syntax:** `\|`

**Example:**
```
[SETTINGS]
Expression|value > 10 \| value < 5
Path|C:\Program Files\App\data.txt
```

The `\|` escapes the pipe character so it's treated as literal text, not a field separator.

**Edge Case - Field Ending with Backslash:**

If a field value ends with a backslash, add a space before the delimiter to prevent ambiguity:

```
[PATHS]
WindowsPath|C:\Program Files\App\ |NextField
```

The space after the trailing backslash prevents `\|` from being interpreted as an escaped delimiter.

**In Text Blocks:** No escaping is needed. Everything is literal.

```
[{CODE_SAMPLE}]
if (value | flag) {
    path = C:\Program Files\App\
}
[EOG]
```

All pipes and backslashes in the text block above are literal - no escaping required.

**Note on Character Encoding:**

Since Set files use UTF-8 encoding by default, Unicode characters can be included directly without escape sequences:

```
[MESSAGES]
Welcome|Café ☕
Greeting|你好世界
Symbol|★ ♥ ✓
[EOG]
```

For alternative methods of representing special characters, see Section 7 (Implementation Patterns).

### 2.9 Comments and Documentation

**Text Outside Groups:**  
Any text outside of group markers is ignored by parsers and serves as comments or documentation.

```
myconfig.set

This is a comment.
It will be ignored by parsers.

[DATABASE]
Host|localhost
```

**Documentation Before Groups:**  
Text immediately before a group marker (with no blank line) is considered documentation for that group.

```
Database connection settings for production
[DATABASE]
Host|prod.example.com
Port|5432
```

**Unreferenced Text Blocks:**  
Text blocks that are not referenced anywhere can serve as coherent comment blocks.

```
[{NOTES}]
These are internal notes.
Not referenced by any group, 
so effectively a comment.
[EOG]
```

### 2.10 Empty Lines

- Empty lines between groups are ignored
- Empty lines within groups are ignored
- An empty line after a group implies `[EOG]`
- Multiple consecutive empty lines are treated as one empty line

### 2.11 Field Definitions

Groups using positional fields should define field names on the first line after the group marker.

**Syntax:** `{field1|field2|field3}`

**Example:**
```
[USERS]
{id|username|email|role}
1|alice|alice@example.com|admin
2|bob|bob@example.com|user
```

Field definitions are optional but strongly recommended for clarity and validation.

### 2.12 End of File

**Syntax:** `[EOF]`

The `[EOF]` marker is optional. End of file is implicit when the file ends.

---

## 3. File Configuration

This section describes the recommended conventions for configuring parser behavior and storing file metadata.

### 3.1 Filename as First Line

**Convention:** The first line of the file should be the filename.

```
myconfig.set

[SETTINGS]
```

This helps identify the file when contents are copied, embedded, or transmitted separately from filesystem metadata.

### 3.2 The `[THIS-FILE]` Group

Parser configuration and file metadata should be stored in a group, typically named `[THIS-FILE]`.

**Example:**
```
myconfig.set

[THIS-FILE]
Version|4.0
Created|2025-11-27
Author|Kirk Siqveland
Delimiters|:[]:{}:|:\:…:
Encode|UTF-8
Localize|NFC|en-US|LTR
[EOG]

[SETTINGS]
AppName|My App
```

### 3.3 Configuration Keys

**Recommended keys for `[THIS-FILE]` group:**

#### Delimiters

Specifies custom delimiter set for the entire file.

**Format:** `Delimiters|:[]:{}:|:\:…:`

**How to Read the Delimiter Definition:**

The delimiter definition line uses a self-describing format. The first character(s) define how to parse the rest of the line.

**Example:** `:[]:{}:|:\:…:`

Breaking this down:
```
:  []  :  {}  :  |  :  \  :  …  :
^  ^^  ^  ^^  ^  ^  ^  ^  ^  ^
|  |   |  |   |  |  |  |  |  └─ Optional trailing delimiter
|  |   |  |   |  |  |  |  └──── Ellipsis marker
|  |   |  |   |  |  |  └─────── Delimiter
|  |   |  |   |  |  └────────── Escape character
|  |   |  |   |  └───────────── Delimiter
|  |   |  |   └──────────────── Field delimiter
|  |   |  └──────────────────── Delimiter
|  |   └─────────────────────── Text block brackets
|  └─────────────────────────── Delimiter
└────────────────────────────── Group header brackets
```

**Reading process:**
1. First character (`:`) is the **preamble delimiter** - used only to parse this line
2. Split the rest of the line by this delimiter
3. Extract each component in order:
   - `[]` = Group header brackets
   - `{}` = Text block brackets
   - `|` = Field delimiter (used throughout the file)
   - `\` = Escape character
   - `…` = Ellipsis marker

**Custom Example:**
```
[THIS-FILE]
Delimiters|;[];{};,;\;...;
[EOG]
```

This sets:
- Preamble delimiter: `;`
- Group headers: `[]`
- Text blocks: `{}`
- Field delimiter: `,` (comma instead of pipe)
- Escape character: `\`
- Ellipsis: `...` (three periods instead of single character)

**Default:** If not specified, assumes `:[]:{}:|:\:…:`

#### Encode

Character encoding for the file.

**Format:** `Encode|UTF-8`

**Common values:**
- `UTF-8` (default and recommended)
- `UTF-16`
- `ASCII`
- `ISO-8859-1`

**Default:** `UTF-8`

#### Localize

Internationalization settings affecting text processing, sorting, and comparison.

**Format:** `Localize|NORMALIZATION|LOCALE|DIRECTION`

Components:
- **NORMALIZATION:** Unicode normalization (NFC, NFD, NFKC, NFKD)
- **LOCALE:** Language-region code (en-US, es-ES, zh-CN, ar-SA, multi)
- **DIRECTION:** Text direction (LTR, RTL, AUTO)

Default: `NFC|en-US|LTR`

Examples:
```
Localize|NFC|en-US|LTR         # English (US), left-to-right
Localize|NFC|ar-SA|RTL         # Arabic, right-to-left  
Localize|NFC|multi|AUTO        # Multiple languages, auto-detect
```

### 3.4 Metadata Keys

**Common metadata keys:**
- `Version` - Specification version or file format version
- `Created` - Creation date
- `Modified` - Last modification date
- `Author` - File creator
- `Copyright` - Copyright notice
- `Description` - File description or purpose

These are conventions only. Implementations may define their own metadata keys.

### 3.5 Placement

The `[THIS-FILE]` group should be placed:
- After the filename (first line)
- Before other data groups
- Immediately after any file-level comments

This is conventional, not required. 
The group can be placed anywhere in the file. However, if alternative delimiters are defined, 
they may not be available until the parser has read the [THIS-FILE] group, causing errors.

---

## 4. Group Types in Detail

### 4.1 Regular Groups (Positional Fields)

Regular groups contain delimited data with positional fields - similar to CSV but more readable.

**Syntax:** `[GROUPNAME]`

**Structure:**
```
[GROUPNAME]
{field1|field2|field3}     ← Optional field definition
value1|value2|value3        ← Data records
value1|value2|value3
[EOG]                       ← Optional end marker
```

**Example:**
```
[EMPLOYEES]
{id|first_name|last_name|department|hire_date}
101|Alice|Smith|Engineering|2023-01-15
102|Bob|Jones|Marketing|2023-02-20
103|Carol|White|Engineering|2023-03-10
[EOG]
```

**Rules:**
- Each data line must have the same number of fields as defined
  - An exception is the addition of single-use fields as a final field
- Empty fields are represented by nothing between delimiters: `value1||value3`
- Leading/trailing spaces in fields must be handled by the parser, default is to Trim() them. 
- Empty lines within a group are not accepted, they will cause the Group to end [EOG]

**Field Order:**
Field order is significant. Each line must match the field definition order.

### 4.2 Key-Value Groups

The same `[GROUPNAME]` syntax works when used for key-value pairs instead of positional fields.

**Structure:**
```
[GROUPNAME]
Key1|Value1
Key2|Value2
Key3|Value3
[EOG]
```

**Example:**
```
[DATABASE_CONFIG]
Host|localhost
Port|5432
Database|myapp
Username|admin
Password|secret123
MaxConnections|100
Timeout|30
[EOG]
```

**Key Naming Rules:**
Keys follow the same rules as group names:
- Letters, numbers, hyphens, underscores only
- No spaces
- Case-sensitive

**Value Rules:**
- The first field is typically the name or id
- Everything after the first `|` is the value
- Values can be empty: `Key|`
- Values can contain any characters (use escape for delimter)
- Values can reference text blocks: `Key|[{TEXTBLOCK}]`

**Distinguishing Key-Value from Positional data:**
There is no syntactic difference. The distinction is semantic:
- If using `{field|names}` definition → positional fields
- If not using field definition → typically key-value pairs
- Implementations may choose conventions (e.g., ALL_CAPS keys for config)

### 4.3 Text Block Groups

Text blocks store raw, unprocessed multi-line content.
All content beginning the line after the [{GROUPNAME}] all the way to the [EOG] 
is stored as exact text (including non-printing characters). 
- Parsing may need to handle unwanted final line-feed and or carriage-return

**Syntax:** `[{GROUPNAME}]`

**Structure:**
```
[{GROUPNAME}]
Raw text content here.
Everything preserved exactly.
No escaping needed!
[EOG]
```

**Characteristics:**
- **No escape sequences** - Backslashes are literal
- **No delimiter processing** - Pipes are literal
- **Exact preservation** - Every space, tab, newline preserved
- **Binary-safe** - Can contain any byte sequence (subject to file encoding)
  - If binary bit-structure may be different, define parsing in [THIS-FILE]

**Use Cases:**
- Multi-paragraph text (descriptions, documentation)
- Source code snippets
- JSON/XML/YAML content
- License texts
- Formatted content (markdown, HTML)
- Base64-encoded binary data

**Example:**
```
[{README}]
# My Application

## Installation

bash
npm install my-app

## Usage

Run the application with: `./myapp --config=settings.set`

For more information, see the documentation.
[EOG]
```

**Content Boundaries:**
Text block content ends at:
1. Explicit `[EOG]` marker
2. Another group marker
3. End of file

### 4.4 Text Block References

Text blocks can be referenced in regular group values.

**Syntax:** Use `[{GROUPNAME}]` as a value

**Example:**
```
[APP_INFO]
Name|My Application
Version|2.0.0
Description|[{APP_DESCRIPTION}]
License|[{LICENSE_TEXT}]
Readme|[{README}]
[EOG]

[{APP_DESCRIPTION}]
A powerful tool for managing workflows.

Features include:
- Task tracking
- Team collaboration
- Real-time sync
[EOG]

[{LICENSE_TEXT}]
MIT License
Copyright (c) 2025 Kirk Siqveland
[EOG]

[{README}]
See documentation at: https://example.com/docs
[EOG]
```

**Parser Behavior:**
When parsing, the value `[{APP_DESCRIPTION}]` should be replaced with the full content of the text block named `APP_DESCRIPTION`.

**Reference Rules:**
- Referenced group must exist
- Referenced group must be a text block (use `[{NAME}]` syntax)
- No circular references
- No nested references (text blocks can't reference other text blocks)

**Multiple References:**
The same text block can be referenced by multiple groups:
```
[CONFIG_EN]
Welcome|[{WELCOME_TEXT}]

[CONFIG_ES]
Welcome|[{BIENVENIDA_TEXT}]

[{WELCOME_TEXT}]
Welcome to our application!
[EOG]

[{BIENVENIDA_TEXT}]
¡Bienvenido a nuestra aplicación!
[EOG]
```

### 4.5 Mixed Usage

A single Set file can contain any combination of group types:

```
myapp.set

[THIS-FILE]
Version|4.0
Created|2025-11-27

[DATABASE]
Host|localhost
Port|5432

[USERS]
{id|username|email}
1|alice|alice@example.com
2|bob|bob@example.com

[{LICENSE}]
MIT License
Copyright (c) 2025...
[EOG]
```

---

## 5. Optional Advanced Features

This section describes advanced features that extend the minimum specification. 
Implementations may choose to support some, all, or none of these features.

### 5.1 Single-Use Fields (`:::`)

**Note:** The preamble delimiter (`:` by default) can be replaced using the Delimiter definition in `[THIS-FILE]`.

Fields prefixed with `:::` allow a single field to be added to a single line, eliminating the need for a large number of empty fields in a typical table definition.

**Syntax:** `:::fieldname:value`

The `:::` marker is followed by the field name, then the preamble delimiter (`:` by default), then the value.

**Example:**
```
[USERS]
{id|username|email|role}
1|alice|alice@example.com|admin
2|bob|bob@example.com|user|:::temp_note:Pending verification
3|charlie|charlie@example.com|user
```

In line 2, the `temp_note` field is added just for that record without modifying the field definition. Other records don't need to have empty values for this field.

**Use Cases:**
- Temporary data during import/migration
- One-off notes or metadata for specific records
- Processing flags that only apply to some records
- Avoiding sparse tables with many empty fields

**Multiple single-use fields on one line:**
```
[CONTACTS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com|:::phone:555-1234|:::department:Engineering
3|Carol|carol@example.com
```

**With custom preamble delimiter:**

If your `[THIS-FILE]` defines delimiters as `;[];{};,;\;...;`, then single-use fields would use `;`:

```
[DATA]
{id|value}
1|100
2|200;;;note;Special case
```

### 5.2 Ellipsis Shorthand

The ellipsis character `…` (or three periods `...`) indicates "remaining fields are empty."

**Example:**
```
[CONTACTS]
{id|name|phone|email|address|city|state|zip}
1|Alice|555-1234|alice@example.com|…
2|Bob|555-5678|…
```

Is equivalent to:
```
1|Alice|555-1234|alice@example.com||||
2|Bob|555-5678|||||
```

**Rules:**
- Ellipsis must be the last value on the line
- All fields after ellipsis are treated as empty
- Useful for reducing file size when many trailing fields are empty

### 5.3 Single-Line Delimiter Override

For individual lines that contain instances of the standard delimiter, you can override the delimiter for just that line.

**Syntax:** Line starts with preamble delimiter followed by the single-use delimiter

**Example:**
```
[SETTINGS]
AppName|My Application
Port|8080
:!URL!https://example.com/api?param1=value|param2=value|param3=value
:!Expression!(a | b) & (c | d) | (e | f)!notes!Additional field
Database|localhost
[EOG]
```

**How it works:**
1. Line starts with `:` (the preamble delimiter from `[THIS-FILE]`)
2. Immediately followed by `!` (the single-use delimiter for this line only)
3. Rest of line uses `!` instead of `|` as field delimiter
4. Next line returns to standard delimiter

**Use cases:**
- URLs with query parameters containing pipes
- Mathematical expressions with logical OR operators
- File paths or data with many delimiter characters
- Any field where escaping every delimiter would be tedious

**Without single-line override:**
```
URL|https://example.com/api?param1=value\|param2=value\|param3=value
```

**With single-line override:**
```
:!URL!https://example.com/api?param1=value|param2=value|param3=value
```

Much more readable!

### 5.4 Alternative Delimiters

The default delimiter is `|` (pipe), but the entire file can use custom delimiters via the `[THIS-FILE]` configuration.

**Example:**
```
myconfig.set

[THIS-FILE]
Delimiters|;[];{};,;\;...;
[EOG]

[SETTINGS]
Key,Value
Another,Value
[EOG]
```

This changes the field delimiter from `|` to `,` (comma) for the entire file.

**When to use alternative delimiters:**
- Your data frequently contains pipes
- Importing from CSV (use comma delimiter)
- Interfacing with systems that expect specific delimiters
- Personal preference for readability

**Recommendations:**
- Stick with defaults unless you have a specific reason to change
- Document custom delimiters clearly in file comments
- Consider whether single-line override (section 5.3) is sufficient for edge cases
- Test thoroughly - parser must correctly interpret delimiter definition line

### 5.5 Extended Localization

Beyond the basic `Localize` setting, implementations may support:

**Multiple Locale Support:**
```
[THIS-FILE]
Localize|NFC|multi|AUTO
LocaleDetails|en-US,es-ES,fr-FR
[EOG]
```

**Collation Rules:**
```
[THIS-FILE]
Localize|NFC|en-US|LTR
Collation|unicode
CaseSensitive|true
[EOG]
```

**Bidirectional Text:**
For mixed LTR/RTL content, implementations may embed Unicode bidirectional control characters or provide directionality hints per-field.

These are implementation-specific extensions and not part of the core specification.

---


---

## Related Documents

**[Set File Implementation Guide v4.0](SetFile_ImplementationGuide_v4_0.md)**  
Comprehensive guide covering:
- Query Language (SetQL)
- Implementation Patterns & Conventions
- SetTag Extensions
- CRUD Operations
- Validation & Error Handling
- Programming Interface Guidelines
- Complete Examples
- Version History & Migration

---

*End of Set File Format Specification v4.0*

**Questions or feedback?**  
Visit: https://github.com/kirksiqveland/setfile

**License:**  
Creative Commons Attribution 4.0 International (CC BY 4.0)  
Copyright (c) 2025 Kirk Siqveland
