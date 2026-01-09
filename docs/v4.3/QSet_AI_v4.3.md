# Q-Set File Format - Claude Skill
(AI-Learning Doc)

**Version:** 4.3  
**Purpose:** Parse and generate Q-Set (.qset) files  
**Spec Reference:** QSet_Spec_v4_3.md

---

## Core Concept

Q-Set is a simplified, human-readable data format for configuration files, tables, and structured text. Files use `.qset` extension (or `.set` with Q-Set conventions).

**Key Principle:** Default delimiters, line-based parsing, minimal escaping.

---

## Default Delimiters

```
Group markers:     [ ]
Text group:        [{ }]
Field delimiter:   |
Escape character:  \
Secondary delim:   ! (for nested arrays)
End of group:      [EOG] or empty line
```

---

## File Structure

```
filename.qset                    ← Filename (optional but strongly recommended)

Comments outside groups          ← Ignored by parser
Can span multiple lines

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

## Generation Templates

### Simple Config File

```
config.qset

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
data.qset

[USERS]
{id|username|email|active}
1|alice|alice@example.com|true
2|bob|bob@example.com|false
3|charlie|charlie@example.com|true
```

### Group Without Field Definitions

```
config.qset

[SERIAL_PORT]
protocol|rs232
9600|8|n|1|none
flow_control|rts/cts
```
No field validation - each line parsed independently.

### Text Group (No Escaping)

```
content.qset

[{README}]
# Project Title

This content is **preserved exactly**.
Pipes | and backslashes \ are literal.
No escape processing happens here.
[EOG]

[CONFIG]
AppName|MyApp
```

### Mixed Content

```
mixed.qset

Application configuration and documentation

[METADATA]
Name|MyApplication
Version|2.1.0

[{LICENSE}]
MIT License
Copyright (c) 2025 Your Name
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

## Generation Guidelines

### When Creating Q-Set Files

1. **Use regular groups for:**
   - Key-value configuration
   - Tabular data with consistent structure
   - Lists with field definitions

2. **Use text groups for:**
   - Multi-line text (README, licenses, templates)
   - Code samples
   - Any content with literal `|` or `\`
   - Base64-encoded data

3. **Naming conventions:**
   - Groups: `ALL_CAPS` or `kebab-case`
   - Fields: `lowercase` or `camelCase`

4. **File organization:**
   - Filename on line 1 (optional but **strongly recommended**)
   - Comments before groups (optional)
   - Related groups together
   - Optional `[EOF]` marker at end

5. **Readability:**
   - Blank lines between groups
   - Use `[EOG]` for explicit boundaries
   - Comment sections for context

---

## Error Handling

### Common Parse Errors

```
Unclosed text group:
[{TEXT}]
Content...
[NEXT_GROUP]    ← Missing [EOG]

Invalid group name:
[My Config]     ← Spaces not allowed

Mismatched field count (only when field definition exists):
{id|name|email}
1|Alice         ← Missing email field
```

### Recovery Strategies
- Unclosed text group: treat EOF as implicit `[EOG]`
- Missing fields: pad with empty strings or error
- Extra fields: truncate or error based on strictness

---

## Quick Reference

| Element | Syntax | Example |
|---------|--------|---------|
| Regular group | `[NAME]` | `[CONFIG]` |
| Text group | `[{NAME}]` | `[{LICENSE}]` |
| Field definition | `{f1\|f2\|f3}` | `{id\|name\|email}` |
| Key-value | `key\|value` | `Host\|localhost` |
| End group | `[EOG]` or empty line | `[EOG]` |
| Escape pipe | `\|` | `expr\|a > 5 \| b < 3` |
| Backslash+pipe | `\ \|` | `path\|\ \|data` |
| Nested array | `!` | `items\|a!b!c` |
| Comment | Outside groups | `# Any text` |
| Trim fields | Auto after split | `\| value \|` → `"value"` |

---

## Implementation Checklist

When implementing Q-Set parser:

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

When generating Q-Set files:

- [ ] Choose appropriate group type (regular vs text)
- [ ] Escape `|` in regular group data (use `\|`)
- [ ] For backslash before pipe: use space (`\ |`)
- [ ] Add field definitions for tables
- [ ] Use `[EOG]` for text groups
- [ ] Add blank lines between groups for readability
- [ ] Optional: add comments for documentation

---

## Example: Parsing Algorithm

```
1. Initialize: groups = {}, current_group = null, in_text_group = false

2. For each line:
   a. If line starts with `[{NAME}]` (at column 0):
      - Set current_group = NAME, in_text_group = true
      - Create groups[NAME] = {type: 'text', content: ''}
   
   b. Else if line starts with `[NAME]` (at column 0):
      - Set current_group = NAME, in_text_group = false
      - Create groups[NAME] = {type: 'regular', rows: []}
   
   c. Else if line starts with `[EOG]` (at column 0):
      - Set current_group = null, in_text_group = false
   
   d. Else if line is empty:
      - If not in_text_group: set current_group = null
      - If in_text_group: append '\n' to content
   
   e. Else if current_group exists:
      - If in_text_group: append line to content
      - Else: split on '|', process escapes, trim() each field, add to rows
   
   f. Else: ignore (comment)

3. Return groups
```

---

## Advanced Features

Q-Set uses default delimiters only. For additional features including:
- Single-use fields (`:::`)
- Ellipsis shorthand (`…`)
- Single-line delimiter override (`:!`)
- SET Tags (embedded configuration)
- Custom settings, delimiters and localization via `[THIS-FILE]`

See **SET File Core Specification v4.3**

---

## License

Spec Copyright (c) 2025 Kirk Siqveland  
Licensed under CC BY 4.0

---

_End of Q-Set Claude Skill v4.3_
