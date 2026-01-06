# SET File Q-Set Specification

**Version 4.2 - Simplified Implementation**  
**Updated: December 2025**

* * *

## What is Q-Set?

Q-Set (Quick SET) is a simplified subset of the full SET file specification. It uses default delimiters and excludes advanced features, making it ideal for:

- Simple configuration files
- Quick implementations without complex parsers
- Applications that don't need customization
- Learning SET files before advancing to full features

**File Extension:** `.qset` (or `.set` files that follow Q-Set conventions)

**Philosophy:** Maximum simplicity. Stick to defaults. When you need more, graduate to the Core or Full spec.

**Pick & Choose:** SET File format is fundamentally a flexible tool, intended to be adapted and applied to fit the needs of a particular application. Q-Set (.qset) files are presumed to fit inside the SET File specification, but not use the full specification. Therefore there is no reason a .qset file cannot use delimiter customization or internationalization if needed for your use case. The Q-Set spec simply documents the most common, straightforward implementation path - but you're free to mix features as your application requires.

* * *

## What's Typically in Q-Set

**✅ Core Features (Recommended):**
- Basic groups `[GROUPNAME]`
- Text groups `[{GROUPNAME}]`
- Key-value pairs
- Tables with field definitions
- Comments
- Default delimiters

**⚠️ Advanced Features (Available but not covered in this guide):**
- Custom delimiters (see Core Spec)
- Encoding configuration (see Core Spec)
- Localization settings (see Core Spec)
- Advanced features like single-use fields, ellipsis (see Full Spec)

**Note:** Q-Set is a recommended starting point, not a hard restriction. If your application needs custom delimiters or encoding, use them! This guide focuses on the simplest, most common path.

* * *

## Table of Contents

1. File Structure
2. Groups
3. Text Groups
4. Basic Examples

* * *

# THE Q-SET SPECIFICATION

## 1. File Structure

### Default Delimiters (Q-Set Recommendations)

Q-Set recommends using the standard SET file defaults. If you need custom delimiters, refer to Section 2 of the Core Specification.

- **Group start:** `[`
- **Group end:** `]`
- **Text group start:** `[{`
- **Text group end:** `}]`
- **Field delimiter:** `|` (pipe)
- **Escape character:** `\` (backslash)
- **Secondary delimiter:** `!` (exclamation, for nested arrays) - Optional, less used in qset
- **Preamble delimiter:** `:` (colon, for delimiter definition parsing) - Only needed when modifying delimiters
- **Empty fields marker:** `…` (ellipsis, indicates remaining fields are empty) - Optional, less used in qset
- **End of group:** `[EOG]` or empty line - Optional, less used in qset

### File Organization

A Q-Set file consists of:

1. **Filename** (optional but recommended) - First line of the file
2. **Comments** - Any text outside of groups
3. **Groups** - Sections containing data

**Example:**

```
myconfig.qset

Configuration file for MyApp
Created: 2025-12-14

[DATABASE]
Host|localhost
Port|5432

[APP_SETTINGS]
Theme|dark
Language|en-US
```

### Parsing Rules

- Files are parsed line-by-line (newline = LF or CRLF)
- Empty lines between groups are ignored
- Empty line after group data = end of group (same as `[EOG]`)
- Text outside groups = comments

## 2. Groups

### Basic Group Syntax

**Format:** `[GROUPNAME]`

**Naming Rules:**
- Letters (a-z, A-Z)
- Numbers (0-9)
- Hyphens (`-`)
- Underscores (`_`)
- No spaces or special characters
- Convention: ALL_CAPS (not required)

**Example:**

```
[CONFIG]
AppName|MyApplication
Version|1.0.0
Debug|false

[USERS]
{id|username|email}
1|alice|alice@example.com
2|bob|bob@example.com
```

### Group Content Types

Groups can contain:

**1. Key-Value Pairs:**
```
[SETTINGS]
Width|1920
Height|1080
```

**2. Tables with Field Definitions:**
```
[EMPLOYEES]
{id|name|department}
101|Alice|Engineering
102|Bob|Sales
```

**3. Mixed Single Values:**
```
[CONFIG]
This is a single value on line 1
Another value on line 2
```

### End of Group

A group ends when:
1. Explicit `[EOG]` marker is encountered
2. An empty line is encountered
3. Another group begins
4. End of file

**Example with explicit markers:**

```
[DATABASE]
Host|localhost
Port|5432
[EOG]

[APP]
Name|MyApp
[EOG]
```

**Example with empty lines:**

```
[DATABASE]
Host|localhost
Port|5432

[APP]
Name|MyApp
```

Both examples above are equivalent.

## 3. Text Groups

Text groups store multi-line content with no escaping or delimiter processing.

### Syntax

**Format:** `[{GROUPNAME}]`

**Rules:**
- Content is preserved exactly as-is
- No escape sequences processed
- No delimiter processing
- Ends at `[EOG]`, another group marker, or EOF
- Does NOT end at empty lines (unlike regular groups)

**Example:**

```
[{LICENSE}]
MIT License

Copyright (c) 2025 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files...
[EOG]
```

### Base64 Encoded Data in Text Groups

Text groups can store base64-encoded binary data:

```
[{ICON_PNG}]
iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg==
[EOG]
```

### Referencing Text Groups

Text groups can be referenced in regular groups:

```
[APP_INFO]
Name|MyApp
License|[{LICENSE}]

[{LICENSE}]
MIT License
Copyright (c) 2025...
[EOG]
```

When parsing, `[{LICENSE}]` as a value would be replaced with the content of the text group.

## 4. Basic Examples

### Example 1: Simple Configuration

```
config.qset

[DATABASE]
Host|localhost
Port|3306
Username|admin
Password|secret123

[APP]
Name|MyApplication
Version|2.1.0
Debug|true
```

### Example 2: User Table

```
users.qset

User database for MyApp

[USERS]
{id|username|email|role}
1|alice|alice@example.com|admin
2|bob|bob@example.com|user
3|charlie|charlie@example.com|user
[EOG]
```

### Example 3: With Text Content

```
app.qset

[CONFIG]
AppName|MyApp
Version|1.0

[{README}]
# MyApp

This is a simple application that demonstrates Q-Set configuration.

## Features
- Fast loading
- Simple config
- Easy to edit
[EOG]

[{LICENSE}]
MIT License
Copyright (c) 2025
[EOG]
```

### Example 4: Nested Arrays

Using the secondary delimiter `!` for nested data:

```
[MENU_ITEMS]
{id|label|subitems}
1|File|New!Open!Save!Exit
2|Edit|Cut!Copy!Paste
3|Help|About!Documentation
```

Parse each field, then split subitems on `!` to get the array.

### Example 5: Escape Sequences

When you need a literal pipe character in data:

```
[EXPRESSIONS]
Condition|value > 10 \| value < 5
Path|C:\Program Files\MyApp\data
```

The `\|` becomes a literal pipe in the data.

## Comments

Text outside of groups is ignored and serves as comments:

```
myfile.qset

This is a comment.
It will be ignored by the parser.

[CONFIG]
Key|Value

Another comment here.

[DATABASE]
Host|localhost
```

**Documentation before a group:**

Text immediately before a group (no blank line) is considered related to that group:

```
Database connection settings
[DATABASE]
Host|localhost
```

## Field Definitions

For tables, define field names on the first line after the group marker:

**Syntax:** `{field1|field2|field3}`

```
[CONTACTS]
{id|name|email|phone}
1|Alice|alice@example.com|555-1234
2|Bob|bob@example.com|555-5678
```

Field definitions help with:
- Self-documenting data
- Validation
- Understanding column order

## Escape Sequences

**Only needed in regular groups** (not in text groups).

**Default escape character:** `\` (backslash)

**Primary use:** Escape the field delimiter within data

```
[SETTINGS]
Expression|value > 10 \| value < 5
WindowsPath|C:\Program Files\App\
```

**In text groups:** No escaping needed

```
[{CODE_SAMPLE}]
if (value | flag) {
    path = C:\Program Files\
}
[EOG]
```

## End of File Marker

**Optional:** `[EOF]`

```
[CONFIG]
Key|Value

[EOF]
```

Most parsers don't require this, but it can be useful for:
- Explicit file termination
- Preventing accidental truncation detection
- Standardizing file endings

* * *

## Graduating to Core or Full Spec

When you need features beyond Q-Set:

**Move to Core Spec when you need:**
- Custom delimiters
- Encoding configuration (UTF-8, UTF-16, etc.)
- Localization settings
- File metadata in `[THIS-FILE]`

**Move to Full Spec when you need:**
- Single-use fields (`:::`)
- Ellipsis shorthand (`…`)
- Single-line delimiter overrides
- Extended localization

* * *

## License

This specification is licensed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

**Copyright (c) 2025 Kirk Siqveland**

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose, even commercially

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made

Full license text: https://creativecommons.org/licenses/by/4.0/

* * *

_End of SET File Q-Set Specification v4.2_

**Questions or feedback?**  
Visit: https://github.com/kirksiqveland/setfile

**License:**  
Creative Commons Attribution 4.0 International (CC BY 4.0)  
Copyright (c) 2025 Kirk Siqveland
