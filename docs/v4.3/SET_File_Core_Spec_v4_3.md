# SET File Format Core Specification

**Version 4.3 - Core Implementation**  
**Updated: January 2026**

**This document contains the complete core specification (Sections 1-3).**  
For simplified defaults-only implementation, see the Q-Set Specification.  
For advanced usage patterns and community contributions, visit setfiles.org/advanced

* * *

## Table of Contents

Introduction & Philosophy

1.  Minimum Core Specification
    
2.  File Configuration
    
3.  Optional Features

* * *

# THE SPECIFICATION

## Introduction & Philosophy

### What are SET files?

SET files (.set or .qset file extension) are machine-readable and human-readable data files; designed for storing settings & configuration data, as well as structured data. They are intended more for use internal to a program or environment and less for broadly sharing data, with that in mind they allow both considerable optimization and flexibility relative to other similar data storage formats. The SET file format really shines in smaller files with multiple data structure that are mostly in string format. Which might sound limiting but in fact covers the majority of the uses for data storage in small to medium sized files, typically under around 10,000 lines. The format optimizes parsing by eliminating escaping and data-typing (allowed but not needed for strings.) More importantly, SET files allow distinctly structured Groups of data, comments, and larger blocks of text or even binary data to be included in a single file.

Most notable is how the format allows easy human readable recognition of groups by name, and content stored in a compact pipe-delimited format: **RS232|9600|8|N|1|Off**  
A single file might include, author and date information, groups of key-value pairs, several different data-tables and even the full text of a license.

The design philosophy is a format flexible enough to allow the use of as much, or as little of the protocol as is needed. Allowing only a few lines of code to parse a read-only file, and locate relevant data. However, by having a straight forward framework, with a little creativity, SET files make handling even complex, and diverse data sets easy and natural.

With careful use of well designed parsing functions, it is arguably noticeably more efficient than JSON files. Not just in execution, but most especially during development and debugging.

### Design Principles

**Human First**  
SET files prioritize simplicity, flexibility, and human readability over complex data handling within a known program or environment; rather than for sharing unknown data between programs.

**Simple Specification**  
The core rules can fit on a single page. To add more complex features there are deeper levels of the protocol but the really advanced features are optional extensions, not core requirements.

**Flexible Implementation**  
Parse as much or as little as you need. Most users will select the parsing functions they want, adapt to their needs, and ignore the rest. A minimal parser can be written in ~50 lines of code. Advanced features are available when needed.

**Convention Over Enforcement**  
The format enables patterns but doesn't mandate them. Implementations may choose their own conventions to match their environment and needs.

**No Magic**  
Everything is explicit and visible. No hidden behaviors, no surprising type coercion, no implicit conversions.

### When to Use SET files

**Good fits:**

*   Configuration files
    
*   Settings and preferences
    
*   Structured data with mixed types (key-value + tables + text)
    
*   Data that is largely in string format or easy to interpret from string format
    
*   Multi-line text content (licenses, descriptions, documentation)
    
*   Human-editable data that needs version control
    
*   CSV replacement with multiple distinct data sets, comments and where readability matters
    

**Poor fits:**

*   Real-time data streaming (use binary protocols)
    
*   Deep nested hierarchies (consider JSON/XML)
    
*   Large binary data (reference external files instead)
    
*   Performance-critical parsing (use binary formats)
    

### File Extensions

*   `.set` - Standard SET file (may use any features)
    
*   `.qset` - Using only minimal/simple implementation (conventionally uses defaults only)
    

**Note:** The "q" in `.qset` denotes a simplified or "quick" implementation, but it still fully complies with the specification.

### Implementation Flexibility

This specification defines the complete SET file format. Implementations may choose to support:

*   **Minimum (Q-Set approach):** Defaults only, typically with parsing built-in rather than include libraries
    
*   **Standard (Core):** Sections 1-3 of this specification
    
*   **Extended:** Core plus advanced usage patterns from community resources
    
*   **Mix-and-Match:** Implement only relevant sections as needed, for internal use
    

* * *

## 1\. Minimum Core Specification

This section defines the minimum requirements for a SET file. Everything in this section is **required** for basic compliance.

### 1.1 File Structure

SET file organization:

*   **Parsing** SET files are initially parsed on NewLine (LF or CRLF)
    
*   **Preamble** (optional): comments containing information about the file and its content
    
*   **Group(s)**: Groups of delimited sections of the file containing related information
    
    *   Each Group MUST have a unique name within the file
        
    *   Names are written with any combination of Letters, Numbers, hyphen `-` and underscore `_` ONLY
        
    *   Typically but not required Group Names are in all caps
        
    *   a Group may contain diverse types of information: Key|Value, Table, Delimited String
        
    *   a Text Group may be used to store extensive text information (even multi-line)
        
*   **Text-Group(s)**: a special type of Group
    
    *   all the contents of a Text-Group are a single entity allowing multiple lines and unusual characters
        
    *   no escaping is needed inside a Text-Group, all content is "as-is"
        
    *   unlike other Groups Text-Groups do not end on empty lines.
        
    *   Text Group names may be use in a Line, as a Linked Reference - `License | [{LICENSE}]`
        
*   **Line(s)**: within a group hold the specific information in delimited arrays.
    
    *   a Line could be a single entry array. The parser might look for "line 3" of "\[CONFIG\]" - no delimiter
        
    *   a Line may be a Key|Value pair - one delimiter
        
    *   a Line may be a delimited array of data like a csv - but normally "|" pipe delimited
        
    *   a field in a line, may reference a Text Group in the same SET File
        
    *   spaces before/after delimiters may provide readability but the parser must account for its use or not as needed
        
    *   a field within a line may itself be an array of data (nested arrays), using the secondary delimiter (default=`!`)
        
    *   an empty line is considered an End-of-Group marker and is equivalent to `[EOG]`
        
*   **Comments**: Any text outside of a Group is considered a comment
    
    *   Any comment immediately preceding a Group is assumed to be related to that group
        

Typically a SET file consists of:

*   Optional but recommended, filename identifier on the first line
    
*   Optional documentation/comments (text outside groups)
    
*   Optional convention-map used in the file, stored in a \[THIS-FILE\] group - see below
    
*   One or more \[GROUPS\] containing data
    
*   Optional but encouraged End-of-Group markers: \[EOG\]
    
*   Optional end-of-file marker \[EOF\]
    

**Example:**

```
myconfig.set

This file contains application configuration.
Created: 2026-01-07

[DATABASE]
Host|localhost
Port|5432

[APP_SETTINGS]
Theme|dark
Language|en-US

[PROTOCOL]
RS232|9600|8|N|1|none
```

### 1.2 SET Tags

It can often be convenient to include settings information inside of some other document or file, particularly in file formats that allow comments, such as Markdown or program files (PHP, Rust, JavaScript, Python, etc.).

When embedding a SET file,

*   Use the native comment format, immediately followed by a single space,
    
*   followed by an opening curly brace `{` and the word `SETTAG:` (in all caps, ending with a colon)
    
*   followed by a name for the tag, a closing curly brace `}` and a new line
    
*   Then all the normal SET File content is included until the end of the SET content
    
*   Ending with `{/SETTAG/}` a space and the closing comment markup
    
*   In formats where each line must use comment format, the comment marks are followed by a single space and then the normal SET file content.
    

**Example (HTML/Markdown):**

```
<!-- {SETTAG:This_Tag_Name} 
what follows is the SET file content all the way to 
an ending marker 
{/SETTAG/} -->

or

// {SETTAG:This_Tag_Name} 
// what follows is the SET file content all the way to 
// an ending marker 
// {/SETTAG/} 
```

**Note:** The same pattern applies in other languages using their respective comment syntax:

*   JavaScript/PHP: `// {SETTAG:ConfigInfo} ... {/SETTAG/}`
    
*   Python: `# {SETTAG:ConfigInfo} ... {/SETTAG/}`
    
*   Rust: `// {SETTAG:ConfigInfo} ... {/SETTAG/}`
    
*   CSS: `/* {SETTAG:ConfigInfo} ... {/SETTAG/} */`
    

### 1.3 Groups

Groups are the fundamental data containers in SET files.

**Syntax:** `[GROUPNAME]`

**Naming Rules:**

*   The line with the Group name must begin with the Group-Name character, by default "\["
    
*   No characters or spaces must exist before the Group Name character
    
*   Only letters (a-z, A-Z), numbers (0-9), hyphens (-), and underscores (\_)
    
*   No spaces allowed in the name (use underscore or hyphen for word separation)
    
*   ALL\_CAPS is conventional but not required
    
*   Must be unique within the file
    
*   \[GROUPS\] end with "\[EOG\]", the next \[GROUP\], or the first blank line
    

**Examples:**

*   `[DATABASE]` ✓
    
*   `[App_Settings]` ✓
    
*   `[USER-LIST]` ✓
    
*   `[Config 2]` ✗ (contains space)
    
*   `[My.Config]` ✗ (contains period)
    

### 1.4 Regular Groups (Positional Fields)

Regular groups contain delimited data with positional fields - similar to CSV but more readable and less problematic.

**Syntax:** `[GROUPNAME]`

**Structure:**

```
[GROUPNAME]
{field1|field2|field3}     ← Optional field definition
value1|value2|value3       ← Data records
value1|value2|value3
[EOG]                      ← Optional end marker
```

**Key-Value Pairs:**

```
[SETTINGS]
Key|Value
AnotherKey|Another Value
```

**Mixed Line Arrays**

```
[SETTINGS]
Owner|Kirk
Model|THX1138
Protocol|RS232|9600|8|None|1
```

**Positional Fields (Tables):**

```
[USERS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com
```

**Example with complete table:**

```
[EMPLOYEES]
{id|first_name|last_name|department|hire_date}
101|Alice|Smith|Engineering|2023-01-15
102|Bob|Jones|Marketing|2023-02-20
103|Carol|White|Engineering|2023-03-10
[EOG]
```

**Rules:**

*   If {Field|Names} are used each data line must have the same number of fields as defined
    
*   Empty fields are represented by nothing (no spaces) between delimiters: `value1||value3`
    
*   Leading/trailing spaces in fields are typically trimmed by the parser
    
*   If trim() is default, strings with starting/trailing spaces could be enclosed in quotes - and parsing rules set
    
*   Empty lines within a group are not accepted, they will cause the Group to end \[EOG\]
    

**Field Order:** Field order is significant. If field names are used, each line of that Group must match that order.

**Nested Arrays:**

```
[PRODUCTS]
{id|name|colors|sizes}
1|T-Shirt|Red!Blue!Green|S!M!L!XL
2|Jeans|Blue!Black|28!30!32!34
```

In this example, the pipe `|` delimits the main fields, while the exclamation point `!` delimits items within the nested arrays (colors and sizes). Lines in a SET file may end with a single LF character or a LF and a CR (line-feed and carriage return) Group contents are always separated by the defined delimiter, typically a pipe - "|" Lines should not start with or end with a delimiter unless the last field is empty

### 1.5 Key-Value Groups

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

**Key Naming Conventions:**

*   Keys follow the same rules as group names:
    
    *   Letters, numbers, hyphens, underscores only
        
    *   No spaces
        
    *   Case-sensitive
        

**Value Conventions:**

*   The first field is typically the name or id
    
*   Everything after the first `|` is a value
    
*   Values can be empty: `Key|`
    
*   Values can contain any characters (use escape for delimiter)
    
*   Values are stored as string data, but may represent other formats
    
*   Values can reference text blocks: `Key|[{TEXTBLOCK}]`
    

**Distinguishing Key-Value from Positional data:** There is no syntactic difference. The distinction is semantic:

*   If using `{field|names}` definition → positional fields
    
*   If not using field definition → typically key-value pairs or mixed content lines
    
*   Implementations may choose conventions (e.g., ALL\_CAPS keys for config)
    

### 1.6 End of Group

Groups end when:

1.  An explicit `[EOG]` marker is present
    
2.  An empty line is encountered (implicit `[EOG]`)
    
3.  Another group begins
    
4.  End of file is reached
    

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

### 1.7 Text Block Groups

Text blocks store raw, unprocessed multi-line content. All content beginning the line after the \[{GROUPNAME}\] all the way to the \[EOG\] is stored as exact text (including non-printing characters).

*   Parsing may need to handle unwanted final line-feed and or carriage-return
    

**Syntax:** `[{GROUPNAME}]`

**Structure:**

```
[{GROUPNAME}]
Raw text content here.
Everything preserved exactly.
No escaping needed!
[EOG]
```

**Content Rules:**

*   All content between `[{GROUPNAME}]` and the end marker is preserved exactly
    
*   No escape sequences are processed
    
*   No delimiters are processed
    
*   Every space, tab, blank line, and character (printing on non-printing) is preserved
    
*   Content ends at `[EOG]`, another group marker, or end of file - but NOT at an empty line
    

**Characteristics:**

*   **No escape sequences** - Backslashes are literal
    
*   **No delimiter processing** - Pipes are literal
    
*   **Exact preservation** - Every space, tab, newline preserved
    
*   **Binary-safe** - Can contain any byte sequence (subject to file encoding)
    

**Use Cases:**

*   Multi-paragraph text (descriptions, documentation)
    
*   Source code snippets
    
*   JSON/XML/YAML content
    
*   License texts
    
*   Formatted content (markdown, HTML)
    
*   Base64-encoded binary data
    

**Example:**

```
[{LICENSE_TEXT}]
MIT License

Copyright (c) 2026 Kirk Siqveland

Permission is hereby granted, free of charge...
[EOG]
```

**Example with formatted content:**

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

**Content Boundaries:** Text block content ends at:

1.  Explicit `[EOG]` marker
    
2.  Another group marker
    
3.  End of file
    
4.  Notice: Empty Lines do NOT trigger \[EOG\]
    

**Encoded Binary Data in Text Groups:**

Text groups can store base64-encoded binary data, making them useful for embedding small binary assets like icons, cryptographic keys, or checksums.

**Example:**

```
[{ICON_PNG}]
iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==
[EOG]

[{PUBLIC_KEY_ED25519}]
MCowBQYDK2VwAyEA8nE7YvqWKxRHEaoKbqXXEj1cTKN0XjJqKqvKqvKqvKqv
[EOG]

[ASSETS]
{name|type|data}
icon|png|[{ICON_PNG}]
pubkey|ed25519|[{PUBLIC_KEY_ED25519}]
[EOG]
```

Applications decode the base64 content as needed. This pattern is useful for:

*   Small embedded images or icons
    
*   Cryptographic keys and certificates
    
*   Hash values and checksums
    
*   Test fixtures that need to be inline
    

### 1.8 Text Block References

Regular groups can reference text blocks using the syntax `[{GROUPNAME}]` as a value.

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
Copyright (c) 2026 Kirk Siqveland
[EOG]

[{README}]
See documentation at: https://example.com/docs
[EOG]
```

**Text Block Location:** Text Blocks may be defined anywhere in the file, before or after the groups that reference them. Parsers should be designed to locate text blocks when their references are encountered.

**Parser Behavior:** Typically when parsing, the value `[{APP_DESCRIPTION}]` would be replaced with the full content of the text block named `APP_DESCRIPTION`.

**Rules:**

*   Referenced text block must exist in the same file
    
*   No circular references allowed
    
*   No nested references (text blocks cannot reference other text blocks)
    
*   Multiple references to the same block are not considered a problem (unless you design your implementation that way)
    

**Multiple References:** The same text block can be referenced by multiple groups:

```
[CONFIG_EN]
Welcome|[{WELCOME_TEXT}]
HomePageTitle|[{WELCOME_TEXT}]

[{WELCOME_TEXT}]
Welcome to SET Files!
[EOG]
```

### 1.9 Delimiters

**Default Delimiters:**

*   The most visible is the use of `|` as the line-delimiter
    
*   Alternative delimiters can be defined in the \[THIS-FILE\] Group using the line `Delimiter|:[]:{}:|:\:…:!`
    
*   the assumed default value (Map) is `:[]:{}:|:\:…:!`
    
*   the 1st character `:` is the settings delimiter, primarily just for this line
    
*   the 2nd character `[` begins a GROUP
    
*   the 3rd character `]` ends a GROUP
    
*   the 5th character `{` begins a TEXT-GROUP
    
*   the 6th character `}` ends a TEXT-GROUP
    
*   the 8th character `|` is the line delimiter
    
*   the 9th character `\` is the escape character
    
*   the 10th character `…` indicates empty remaining fields
    
    *   the default 10th character can be either `…` or `...`
        
*   the 11th character `!` is the second level delimiter (for a nested array)
    

Example `Delimiter|:[]:{}:#:\:…:!` Changes the line-delimiter from `|` to `#`

By default the pipe character "|" separates:

*   Fields in positional data (arrays)
    
*   Keys from values in key-value pairs (two item arrays)
    
*   Field names in field definitions {First|Last|Middle|E-Mail}
    

**Do not begin or end lines with delimiters** - this would shift all field positions.

### 1.10 Escape Sequences

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

Here the `\|` escapes the pipe character so it's treated as literal text, not a field separator.

**Edge Case - Field Ending with Backslash:**

If a field value ends with a backslash, add a space before the delimiter to prevent ambiguity:

```
[PATHS]
WindowsPath|C:\Program Files\App\ |NextField
```

The space after the trailing backslash prevents `\|` from being interpreted as an escaped delimiter.  
This specific case has important implications when parsing and `trim()`ing your data

**In Text Blocks:** No escaping is needed. Everything is literal.

```
[{CODE_SAMPLE}]
if (value | flag) {
    path = C:\Program Files\App\
}
[EOG]
```

All pipes, backslashes and line-feeds in the text block above are literal - no escaping required.

**Note on Character Encoding:** Since SET files use UTF-8 encoding by default, Unicode characters can be included directly without escape sequences:

```
[MESSAGES]
Welcome|Café ☕
Greeting|你好世界
Symbol|★ ♥ ✓
[EOG]
```

### 1.11 Comments and Documentation

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
Text immediately before a group marker (with no blank line) is typically considered documentation for that group.

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

### 1.12 Empty Lines

*   Empty lines between groups are ignored
    
*   An empty line after a group implies `[EOG]`
    

### 1.13 Field Definitions

Groups using positional fields (columns) should define field names on the first line after the group marker. When present, they apply to all data lines in that group. If your group has mixed structures, do not use field definitions - handle field identification in your application code. The use of Nested Arrays allows for Key:Value pairs in a single line `ID!007|Name!James|Website!https://www.setfiles.org|Email!james@setfiles.org`

**Syntax:** `{field1|field2|field3}`

**Example:**

```
[USERS]
{id|username|email|role}
1|alice|alice@example.com|admin
2|bob|bob@example.com|user
```

Field definitions are optional but strongly recommended for clarity and validation.

### 1.14 Mixed Group Usage

A single SET file can contain any combination of group types:

```
myapp.set

[THIS-FILE]
Version|4.3
Created|2026-01-07

[ENVIRONMENT]
Name|Phred
Connection|Serial
Protocol|RS232|9600|8|N|1|Off

[USERS]
{id|username|email}
1|alice|alice@example.com
2|bob|bob@example.com

[{LICENSE}]
MIT License
Copyright (c) 2026...
[EOG]
```

### 1.15 End of File

**Syntax:** `[EOF]`

The `[EOF]` marker is optional.  
End of file is implicit when the file ends.

* * *

## 2\. File Configuration

This section describes the recommended conventions for configuring parser behavior and storing file metadata.

### 2.1 Filename as First Line

**Convention:** The first line of the file should be the filename.

```
myconfig.set

[SETTINGS]
```

This helps identify the file when content is copied, embedded, or transmitted separately from filesystem metadata.

### 2.2 The `[THIS-FILE]` Group

Parser configuration and file metadata should be stored in a group named `[THIS-FILE]`.

If using a \[THIS-FILE\] it should be the very first Group in the file.

**Example:**

```
myconfig.set

[THIS-FILE]
Version|4.3
Created|2026-01-07
Author|Kirk Siqveland
Delimiters|:[]:{}:|:\:…:!
Encode|UTF-8
Localize|NFC|en-US|LTR
[EOG]

[SETTINGS]
AppName|My App
```

### 2.3 Configuration Keys

**Recommended keys for** `[THIS-FILE]` **group:**

### Delimiters

Specifies custom delimiter set for the entire file.

**Format:** `Delimiters|:[]:{}:|:\:…:!`

**How to Read the Delimiter Definition:**

The delimiter definition line uses a self-describing format. The first character define how to parse the rest of the line.

**Example:** `:[]:{}:|:\:…:!`

Breaking this down:

```
:  []  :  {}  :  |  :  \  :  …  : ! 
^  ^^  ^  ^^  ^  ^  ^  ^  ^  ^
|  |      |      |     |     |    └── Secondary Line Delimiter  Socks|White!Black!Gray|S!M!L|$8.99
|  |      |      |     |     |        
|  |      |      |     |     └─────── Empty Fields Marker
|  |      |      |     |              (indicated empty fields in remainder of line)
|  |      |      |     └───────────── Escape Character
|  |      |      |     
|  |      |      └─────────────────── Line Delimiter
|  |      |     
|  |      └────────────────────────── Text Block Brackets
|  └───────────────────────────────── Group Header Brackets
└──────────────────────────────────── Preamble Delimiter
```

**Reading process:**

1.  First character (`:`) is the **preamble delimiter** - used only to parse this line
    
2.  Split the rest of the data on this line by this delimiter
    
3.  Extract each component in order:
    
    *   `[]` = Group Header brackets
        
    *   `{}` = Text Block brackets
        
    *   `|` = Field Delimiter (used to parse line data)
        
    *   `\` = Escape character
        
    *   `…` = Empty Fields Marker (rather than multiple delimiters which may not number correctly)
        
        *   the Empty Fields Marker may only be used after the last non-empty field
            
    *   `!` = Nested Field Delimiter e.g. Socks|White!Black!Gray|S!M!L|8.99
        

**Custom Example:**

```
[THIS-FILE]
Delimiters|;[];{};,;\;...;!
[EOG]
```

This sets:

*   Preamble Delimiter: `;`
    
*   Group Headers: `[]`
    
*   Text Blocks: `{}`
    
*   Field Delimiter: `,` (comma instead of pipe)
    
*   Escape Character: `\`
    
*   Empty Fields Marker: `...` (three periods instead of single character)
    
*   Nested Delimiter: `!`
    

**Default:** If not specified, assumes `:[]:{}:|:\:…:!`

**When to Use Alternative Delimiters:**

Consider changing delimiters when:

*   Your data frequently contains pipes (`|`) making heavy escaping tedious
    
*   Exporting to CSV format (use comma `,` as primary delimiter)
    
*   Interfacing with legacy systems expecting specific delimiters
    
*   Personal/team preference for readability
    

**Recommendations:**

*   Stick with defaults unless you have a specific reason to change
    
*   Document custom delimiters clearly in file comments
    
*   Consider single-line override (Section 3.3) for edge cases
    
*   Test thoroughly - parser must correctly interpret delimiter definition line
    

### Encoding

Character encoding for the file.

**Format:** `Encode|UTF-8`

**Common values:**

*   `UTF-8` (default and recommended)
    
*   `UTF-16`
    
*   `ASCII`
    
*   `ISO-8859-1`
    

#### Localize

Internationalization settings affecting text processing, sorting, and comparison.

**Format:** `Localize|NORMALIZATION|LOCALE|DIRECTION`

Components:

*   **NORMALIZATION:** Unicode normalization (NFC, NFD, NFKC, NFKD)
    
*   **LOCALE:** Language-region code (en-US, es-ES, zh-CN, ar-SA, multi)
    
*   **DIRECTION:** Text direction (LTR, RTL, AUTO)
    

Default: `NFC|en-US|LTR`

Examples:

```
Localize|NFC|en-US|LTR         # English (US), left-to-right
Localize|NFC|ar-SA|RTL         # Arabic, right-to-left  
Localize|NFC|multi|AUTO        # Multiple languages, auto-detect
```

### 2.4 Metadata Keys

**Common metadata keys:**

*   `Version` - Specification version or file format version
    
*   `Created` - Creation date
    
*   `Modified` - Last modification date
    
*   `Author` - File creator
    
*   `Copyright` - Copyright notice
    
*   `Description` - File description or purpose
    

These are conventions only. Implementations may define their own metadata keys.

### 2.5 Placement

The `[THIS-FILE]` group, if used, should be placed:

*   After the filename (first line)
    
*   Immediately after any file-level comments
    
*   Before any other data groups
    

This is conventional, not required. The group can be placed anywhere in the file.  
However, if alternative delimiters are defined, they may not be available until the parser has read the \[THIS-FILE\] group, causing errors.

* * *

## 3\. Optional Features

This section describes optional features that extend the minimum specification. Implementations may choose to support some, all, or none of these features.

### 3.1 Single-Use Fields

Append data to a single Table-Row without adding a whole column

at the end of the normal line, add `:::` (three preamble delimiters) to flag additional information to follow

**Note:** The preamble delimiter (`:` by default) can be replaced using the Delimiter definition in `[THIS-FILE]`.

Fields prefixed with `:::` allow a single field to be added to a single line, eliminating the need for a large number of empty fields in a typical table definition.

**Syntax:** `:::value` or `:::fieldname:value`

The `:::` marker is followed by the value, or a key/name, the preamble delimiter (`:` by default) and then the value.

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

*   Temporary data during import/migration
    
*   One-off notes or metadata for specific records
    
*   Processing flags that only apply to some records
    
*   Avoiding sparse tables with many empty fields
    

**Multiple single-use fields on one line:**

```
[CONTACTS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com|:::phone:555-1234|:::department:Engineering
3|Carol|carol@example.com
```

**With custom preamble delimiter:**

If your `[THIS-FILE]` defines delimiters as `;[];{};,;\;...;!`, then single-use fields would use `;`:

```
[DATA]
{id|value}
1|100
2|200;;;note;Special case
```

### 3.2 Ellipsis Shorthand

The Empty Fields character `…` (or three periods `...`) indicates "remaining fields are empty." Parsers may allow both versions or be strict to a single option.

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

*   Empty Fields character must be the last value on the line
    
*   All fields after Empty Fields character are treated as empty
    
*   Useful for reducing file size when many trailing fields are empty
    
*   Eliminates need for counting empty fields
    

### 3.3 Single-Line Delimiter Override

For individual lines that contain instances of the standard delimiter, you can override the delimiter for just that line.

**Syntax:** Line starts with preamble delimiter followed by the single-use delimiter to be used

**Example:**

```
[SETTINGS]
AppName|My Application
Port|8080
:!URL!https://example.com/api?param1=value|param2=value|param3=value
:!Expression!(a | b) & (c | d) | (e | f)
Database|localhost
[EOG]
```

**How it works:**

1.  Line starts with `:` (the preamble delimiter from `Delimiter|:[]:{}:|:\:…:!` in `[THIS-FILE]`)
    
2.  Immediately followed by the replacement delimiter, `!` (single-use for this line only)
    
3.  The rest of the line now uses `!` instead of `|` as field delimiter
    
4.  Next line returns to standard delimiter
    

**Use cases:**

*   URLs with query parameters containing pipes
    
*   Mathematical expressions with logical OR operators
    
*   File paths or data with many delimiter characters
    
*   Any field where escaping every delimiter would be tedious
    

**Without single-line override:**

```
Expression|value > 10 \| value < 5 \| value > 100
```

**With single-line override:**

```
:!Expression!(value > 10 | value < 5 | value > 100)
```

The second approach is more readable when there are many instances of the delimiter character in the data.

* * *

## License

This specification is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

**Copyright (c) 2026 Kirk Siqveland**

You are free to:

*   **Share** — copy and redistribute the material in any medium or format
    
*   **Adapt** — remix, transform, and build upon the material for any purpose, even commercially
    

Under the following terms:

*   **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made
    

Full license text: https://creativecommons.org/licenses/by/4.0/

Implementations of this specification may use any license of the implementer's choosing.

* * *

## Related Documents

**Q-Set Specification v4.3**  
Simplified implementation guide using defaults only

**SET File Implementation Guide**  
Comprehensive guide covering:

*   Query Language (SetQL)
    
*   Implementation Patterns & Conventions
    
*   SetTag Extensions
    
*   CRUD Operations
    
*   Validation & Error Handling
    
*   Programming Interface Guidelines
    
*   Complete Examples
    
*   Version History & Migration
    

**Advanced Usage**  
For advanced usage patterns, specialized implementations, and community-contributed methods, visit:  
**https://setfiles.org/advanced**

* * *

_End of SET File Core Specification v4.3_

**Questions or feedback?**  
Visit: https://github.com/kirksiqveland/setfile

**License:**  
Creative Commons Attribution 4.0 International (CC BY 4.0)  
Copyright (c) 2026 Kirk Siqveland
