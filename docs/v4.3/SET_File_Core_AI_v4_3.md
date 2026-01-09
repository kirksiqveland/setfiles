# SET File Format Core - Claude Skill 
(AI-Learning Doc)

**Version:** 4.3  
**Purpose:** Parse and generate SET (.set) and Q-Set (.qset) files with full Core specification support  
**Spec Reference:** SET_File_Core_Spec_v4_3.md

---

## Core Concept

SET files are a human-readable data format for configuration files, tables, and structured text. Files use `.set` extension (full features) or `.qset` extension (defaults-only).

**Key Principle:** Line-based parsing, flexible delimiters, minimal escaping, mixed content types (tables + key-value + text blocks).

**This skill covers:** Core specification (Sections 1-3) including Q-Set defaults plus optional advanced features.

---

## Default Delimiters

```
Group markers:     [ ]
Text group:        [{ }]
Field delimiter:   |
Escape character:  \
Secondary delim:   ! (for nested arrays)
Empty fields:      … (or ...)
Preamble delim:    : (for config and special features)
End of group:      [EOG] or empty line
```

---

## File Structure

```
filename.set                     ← Filename (optional but strongly recommended)

Comments outside groups          ← Ignored by parser
Can span multiple lines

[THIS-FILE]                      ← Optional configuration group (Core feature)
Version|4.3
Delimiters|:[]:{}:|:\:…:!
Encode|UTF-8
[EOG]

[GROUPNAME]                      ← Group start
Key|Value                        ← Key-value pairs
AnotherKey|AnotherValue
                                 ← Empty line = end of group

[TABLE_GROUP]                    ← Another group
{field1|field2|field3}           ← Field definition
row1val1|row1val2|row1val3       ← Data rows
row2val1|row2val2|row2val3

[{TEXT_GROUP}]                   ← Text group (no escaping)
Raw text content here
Pipes | and backslashes \ are literal
No delimiter processing
[EOG]                            ← Explicit end (text groups don't end on empty lines)
```

---

## Parsing Rules

### Line Processing
1. Read file line-by-line (LF or CRLF)
2. Text outside groups = comments (ignore)
3. Empty lines between groups = ignored
4. Empty line after group data = end of group

### Group Detection
- `[GROUPNAME]` = Regular group start
- `[{GROUPNAME}]` = Text group start
- `[EOG]` = Explicit group end
- Group names: `[A-Za-z0-9_-]+` (no spaces/special chars)
- **CRITICAL:** Group markers MUST start at beginning of line (no leading whitespace)
- Group markers mid-line are treated as literal content

### Regular Groups
- Process escape sequences (`\|` → `|`)
- Split fields on `|` delimiter
- **Optional:** First line with `{field|names}` = field definition
  - When present: group becomes a table, subsequent lines are data rows
  - Data rows should match field count (trailing empty fields may be omitted)
- If no field definition: each line parsed independently (no field count validation)
- Ends at: empty line, `[EOG]`, new group, or EOF

### Text Groups
- **No escape processing**
- **No delimiter processing**
- Content preserved exactly as-is
- Does NOT end at empty lines
- Ends at: `[EOG]`, new group marker, or EOF

---

## SET Tags (Embedded Configuration)

Embed SET content in other file formats using comment syntax.

**Pattern:** Comment syntax + space + `{SETTAG:name}` ... `{/SETTAG/}`

**Examples:**

```html
<!-- {SETTAG:ConfigInfo}
[SETTINGS]
Theme|dark
{/SETTAG/} -->
```

```javascript
// {SETTAG:AppConfig}
// [DATABASE]
// Host|localhost
// {/SETTAG/}
```

**Languages:**
- JavaScript/PHP: `// {SETTAG:name} ... {/SETTAG/}`
- Python: `# {SETTAG:name} ... {/SETTAG/}`
- HTML/Markdown: `<!-- {SETTAG:name} ... {/SETTAG/} -->`
- CSS: `/* {SETTAG:name} ... {/SETTAG/} */`

---

## [THIS-FILE] Configuration Group

Optional group for parser configuration and metadata. If used, should be first group in file.

**Common keys:**

```
[THIS-FILE]
Version|4.3
Created|2026-01-07
Author|Name
Delimiters|:[]:{}:|:\:…:!
Encode|UTF-8
Localize|NFC|en-US|LTR
[EOG]
```

### Delimiter Configuration

**Format:** `Delimiters|:[]:{}:|:\:…:!`

**Reading the delimiter string:**
1. First char (`:`) = preamble delimiter (parses this line only)
2. Split rest by preamble delimiter
3. Components in order:
   - `[]` = Group brackets
   - `{}` = Text block brackets
   - `|` = Field delimiter
   - `\` = Escape character
   - `…` = Empty fields marker (can be `...`)
   - `!` = Nested field delimiter

**Custom example (comma-separated):**
```
Delimiters|;[];{};,;\;...;!
```
Changes field delimiter from `|` to `,` for entire file.

### Encoding

**Format:** `Encode|UTF-8`

Common values: UTF-8 (default), UTF-16, ASCII, ISO-8859-1

### Localization

**Format:** `Localize|NORMALIZATION|LOCALE|DIRECTION`

Examples:
```
Localize|NFC|en-US|LTR         # English, left-to-right
Localize|NFC|ar-SA|RTL         # Arabic, right-to-left  
Localize|NFC|multi|AUTO        # Multiple languages
```

---

## Generation Templates

### Simple Config File

```
config.set

[DATABASE]
Host|localhost
Port|5432
User|admin

[APP_SETTINGS]
Theme|dark
Debug|true
```

### Table with Field Definitions

```
data.set

[USERS]
{id|username|email|active}
1|alice|alice@example.com|true
2|bob|bob@example.com|false
3|charlie|charlie@example.com|true
```

### Group Without Field Definitions

```
config.set

[SERIAL_PORT]
protocol|rs232
9600|8|n|1|none
flow_control|rts/cts
```
No field validation - each line parsed independently.

### Text Group (No Escaping)

```
content.set

[{README}]
# Project Title

This content is **preserved exactly**.
Pipes | and backslashes \ are literal.
No escape processing happens here.
[EOG]

[CONFIG]
AppName|MyApp
```

### Mixed Content with Configuration

```
myapp.set

[THIS-FILE]
Version|4.3
Author|Kirk Siqveland
Delimiters|:[]:{}:|:\:…:!
[EOG]

[METADATA]
Name|MyApplication
Version|2.1.0

[{LICENSE}]
MIT License
Copyright (c) 2026 Your Name
[EOG]

[FEATURES]
{id|name|enabled}
1|Authentication|true
2|Logging|true
3|Analytics|false
```

---

## Escape Sequences (Regular Groups Only)

**When to escape:**
- Literal pipe in data: `\|`
- Literal backslash before pipe: add space `\ |`

**Data is trimmed:**
- Leading/trailing whitespace removed from each field
- Whitespace = spaces and tabs
- Applies after delimiter split

```
[PATHS]
WindowsPath|C:\Program Files\App
Expression|value > 10 \| value < 5
BackslashPipe|\mypath\ |data
Spaced| value with spaces |trimmed
```

Result after parsing:
```
WindowsPath → "C:\Program Files\App"
Expression → "value > 10 | value < 5"
BackslashPipe → "\mypath\ |data"
Spaced → "value with spaces" (outer spaces trimmed)
```

**Text groups need NO escaping:**
```
[{CODE}]
if (x | y) {
    path = C:\Windows\System32
}
[EOG]
```

---

## Quote Wrapping Convention

Some implementations use quotes to preserve exact string content.

**Pattern:**
```
[DATA]
Field|"  value with spaces  "
Field|normal value
Field|"value with | pipe inside"
```

**Implementation:**
- If field starts and ends with `"`: strip quotes, preserve content exactly
- Otherwise: process escapes and trim as normal
- Useful for: whitespace preservation, CSV compatibility, avoiding escapes

**Note:** Implementation convention, not spec requirement. Document if used.

---

## Common Patterns

### Nested Arrays (Secondary Delimiter)

```
[MENU]
{id|label|subitems}
1|File|New!Open!Save!Exit
2|Edit|Cut!Copy!Paste
```

Parse: Split on `|`, then split `subitems` field on `!`

### Empty Fields

```
[CONTACTS]
{id|name|email|phone}
1|Alice|alice@example.com|
2|Bob||555-1234
```

Empty string between delimiters = empty field.

### Single Values per Line

```
[TAGS]
production
staging
development
```

Each line = one value (no field delimiter).

### Text Block References

```
[APP_INFO]
Name|My Application
License|[{LICENSE_TEXT}]

[{LICENSE_TEXT}]
MIT License
Copyright (c) 2026...
[EOG]
```

Value `[{LICENSE_TEXT}]` references the text block content.

---

## Validation Rules

### Group Names
- ✅ Valid: `CONFIG`, `app-settings`, `Users_2024`
- ❌ Invalid: `My Config`, `[NESTED]`, `settings!`

### Field Definitions (Optional)
- **Only for structured tables** - not required for all groups
- Must be first line after group marker (if used)
- Format: `{field1|field2|field3}`
- Field names: same rules as group names

### Data Integrity (Only applies when field definitions present)
- Row field count should match definition
- Trailing empty fields can be omitted
- Extra fields = parser-dependent (warn or error)

**Groups without field definitions:**
```
[SERIAL_CONFIG]
protocol|rs232
9600|8|n|1|none
```
Each line is independently valid. No field count validation needed.

---

## Edge Cases

### Empty Groups
```
[EMPTY_GROUP]

[NEXT_GROUP]
Key|Value
```
Valid. Empty group contains no data.

### Comments Between Data
```
[CONFIG]
Key1|Value1
This is NOT a comment - it's data without delimiter
Key2|Value2
```
Lines without `|` in regular groups = single values.

### Text Group with Group-Like Content
```
[{DOCUMENTATION}]
Configuration groups use [GROUPNAME] syntax.
The [EOG] marker ends groups.
[EOG]
```
Content preserved literally until `[EOG]` **at start of line**.

**Important:** Only `[EOG]` at the start of a line ends the group. Mid-line `[EOG]` is literal text.

### Start-of-Line Rule Example
```
[{CODE_SAMPLE}]
To end a group, use [EOG] at the start of a line.
    [EOG] with leading spaces is treated as content.
Not this [EOG] either - it's mid-line.
[EOG]
```

Only the final `[EOG]` (at column 0) terminates the group. The previous two are literal text content.

### Duplicate Group Names
```
[CONFIG]
Key1|Value1

[CONFIG]
Key2|Value2
```
**INVALID:** Each group MUST have a unique name within the file. Duplicate group names violate the spec.

---

## Optional Features (Section 3)

These features extend the minimum specification. Implementations may choose to support some, all, or none.

---

### Single-Use Fields (3.1)

Add extra fields to specific rows without redefining table structure.

**Syntax:** `:::value` or `:::fieldname:value`

```
[USERS]
{id|username|email}
1|alice|alice@example.com
2|bob|bob@example.com:::temp_note:Pending verification
3|charlie|charlie@example.com
```

Row 2 has an extra `temp_note` field. Other rows don't need empty placeholders.

**Common uses:**
- Import flags
- One-off metadata
- Temporary notes
- Migration data

**Multiple single-use fields:**
```
[CONTACTS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com:::phone:555-1234:::department:Engineering
3|Carol|carol@example.com
```

**With custom preamble delimiter:**

If `Delimiters|;[];{};,;\;...;!`, then single-use fields use `;`:
```
[DATA]
{id|value}
1|100
2|200;;;note;Special case
```

---

### Ellipsis Shorthand (3.2)

Indicates "remaining fields are empty" - reduces file size for sparse data.

**Syntax:** `…` (ellipsis) or `...` (three periods) as last value on line

```
[CONTACTS]
{id|name|phone|email|address|city|state|zip}
1|Alice|555-1234|alice@example.com|…
2|Bob|555-5678|…
```

Equivalent to trailing empty fields: `||||`

**Parser rule:** Must be last value on line. All fields after are treated as empty.

---

### Single-Line Delimiter Override (3.3)

Override delimiter for a single line when data contains many instances of standard delimiter.

**Syntax:** Line starts with preamble delimiter (`:` default) + replacement delimiter + data using new delimiter

```
[SETTINGS]
AppName|My Application
:!URL!https://example.com/api?param1=value|param2=value|param3=value
:!Expression!(a | b) & (c | d) | (e | f)
Database|localhost
```

**How it works:**
1. Line starts with `:` (preamble delimiter)
2. Followed by replacement delimiter `!`
3. Rest of line uses `!` instead of `|`
4. Next line returns to standard delimiter

**Use cases:**
- URLs with query parameters containing pipes
- Mathematical expressions with logical OR
- Data with many delimiter characters
- Avoiding tedious escaping

---

## Quick Reference

| Element | Syntax | Example |
|---------|--------|---------|
| Regular group | `[NAME]` | `[CONFIG]` |
| Text group | `[{NAME}]` | `[{LICENSE}]` |
| Config group | `[THIS-FILE]` | See above |
| Field definition | `{f1\|f2\|f3}` | `{id\|name\|email}` |
| Key-value | `key\|value` | `Host\|localhost` |
| End group | `[EOG]` or empty line | `[EOG]` |
| Escape pipe | `\|` | `expr\|a > 5 \| b < 3` |
| Backslash+pipe | `\ \|` | `path\|\ \|data` |
| Nested array | `!` | `items\|a!b!c` |
| Comment | Outside groups | `# Any text` |
| Trim fields | Auto after split | `\| value \|` → `"value"` |
| SET Tag | `{SETTAG:name}` | See above |
| Single-use field | `:::field:value` | Row-specific data |
| Empty fields | `…` or `...` | Trailing empties |
| Line delimiter override | `:!data!data` | Override for one line |

---

## Implementation Checklist

### Q-Set (Defaults-Only) Implementation

**Parser features:**
- [ ] Line-by-line reader (handle LF/CRLF)
- [ ] Group detection (regular vs text)
- [ ] Field delimiter splitting (regular groups)
- [ ] Escape sequence processing (regular groups)
- [ ] Trim whitespace from each field (regular groups)
- [ ] Text group raw content capture
- [ ] Empty line group termination
- [ ] `[EOG]` explicit termination
- [ ] Field definition parsing
- [ ] Comment filtering (outside groups)

**Generator features:**
- [ ] Choose appropriate group type (regular vs text)
- [ ] Escape `|` in regular group data (use `\|`)
- [ ] For backslash before pipe: use space (`\ |`)
- [ ] Add field definitions for tables
- [ ] Use `[EOG]` for text groups
- [ ] Add blank lines between groups for readability
- [ ] Optional: add comments for documentation

### Core (Full) Implementation

**All Q-Set features PLUS:**

**Parser features:**
- [ ] SET Tag recognition and extraction
- [ ] `[THIS-FILE]` configuration parsing
- [ ] Custom delimiter interpretation
- [ ] Delimiter definition parsing
- [ ] Encoding/localization support
- [ ] Single-use field parsing (`:::`)
- [ ] Ellipsis shorthand handling (`…`)
- [ ] Single-line delimiter override (`:!`)

**Generator features:**
- [ ] Generate `[THIS-FILE]` group
- [ ] Custom delimiter configuration
- [ ] Embed SET Tags in other formats
- [ ] Single-use field generation
- [ ] Ellipsis shorthand for sparse data
- [ ] Single-line delimiter override when needed

---

## Example: Parsing Algorithm

```
1. Initialize: groups = {}, current_group = null, in_text_group = false, config = default_delimiters

2. For each line:
   a. If line starts with `[THIS-FILE]` (at column 0):
      - Parse configuration, update delimiters/encoding
   
   b. If line starts with `[{NAME}]` (at column 0):
      - Set current_group = NAME, in_text_group = true
      - Create groups[NAME] = {type: 'text', content: ''}
   
   c. Else if line starts with `[NAME]` (at column 0):
      - Set current_group = NAME, in_text_group = false
      - Create groups[NAME] = {type: 'regular', rows: []}
   
   d. Else if line starts with `[EOG]` (at column 0):
      - Set current_group = null, in_text_group = false
   
   e. Else if line is empty:
      - If not in_text_group: set current_group = null
      - If in_text_group: append '\n' to content
   
   f. Else if current_group exists:
      - If in_text_group: append line to content
      - Else if line starts with preamble+delimiter (e.g., ':!'): 
        - Parse with override delimiter
      - Else: split on delimiter, process escapes, check for ':::' or '…', trim() each field, add to rows
   
   g. Else: ignore (comment)

3. Return groups and config
```

---

## Advanced Features

For advanced usage patterns including:
- **SetQL** - Query language
- **Data typing strategies** - Type hints and validation
- **Schema validation** - Formal structure definitions
- **Extended localization** - Multi-language support
- **Performance optimization** - Caching and indexing
- **Format conversion** - JSON/CSV/YAML interop
- **Security patterns** - Encryption and signing

Visit: **https://setfiles.org/advanced**

---

## License

Spec Copyright (c) 2026 Kirk Siqveland  
Licensed under CC BY 4.0

---

_End of SET File Core Claude Skill v4.3_
