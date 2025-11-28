# Set Files - Minimal Quick Start
**The 5-Minute Guide to Set Files**

---

## What is a Set File?

A Set file (`.set` or `.qset`) is a human-readable text file for storing configuration data and settings.

Think: **CSV meets INI files, but all grown up and actually readable.**

---

## The Absolute Minimum

### 1. Create a File

```
myconfig.set
```

That's the filename on the first line. That's it.

### 2. Add a Group

Groups are sections that hold related data. Start with `[GROUPNAME]`:

```
myconfig.set

[DATABASE]
Host|localhost
Port|5432
User|admin
```

**Rules:**
- Group names in square brackets: `[NAME]`
- Data uses pipe `|` to separate fields
- One piece of data per line

### 3. Done!

That's a valid Set file. You can read it, edit it, parse it.

---

## Two Ways to Store Data

### Key-Value Pairs (Settings)

```
[SETTINGS]
AppName|My App
Theme|dark
Language|en-US
```

**Use for:** Configuration, preferences, simple settings

### Tables (Multiple Records)

```
[USERS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com
3|Carol|carol@example.com
```

The `{field|names}` line is optional but helpful.

**Use for:** Lists, tables, structured data

---

## Multi-Line Text

For long text (descriptions, licenses, etc.), use text blocks:

```
[APP_INFO]
Name|My Application
Description|[{APP_DESCRIPTION}]

[{APP_DESCRIPTION}]
This is my application.

It does many things across
multiple lines of text.

No escaping needed!
[EOG]
```

**Text blocks:**
- Start with `[{NAME}]`
- End with `[EOG]` or blank line
- Everything inside is literal - no special characters
- Reference them with `[{NAME}]` in regular fields

---

## Escaping the Delimiter

If your data contains a pipe `|`, escape it with backslash:

```
[DATA]
Expression|value > 10 \| value < 5
```

**That's the only escape you need.**

---

## Comments

Any text outside groups is a comment:

```
myconfig.set

This is a comment.
It explains what this file does.

[SETTINGS]
Port|8080
```

---

## Complete Minimal Example

```
myconfig.set

Application configuration
Created: 2025-11-27

[DATABASE]
Host|localhost
Port|5432
Database|myapp

[SETTINGS]
Theme|dark
Language|en-US
MaxUsers|50

[USERS]
{id|name|email}
1|alice|alice@example.com
2|bob|bob@example.com

[{WELCOME_MESSAGE}]
Welcome to My Application!

Get started by configuring your settings above.
[EOG]
```

---

## Parsing (Pseudocode)

```python
# Read the file
lines = read_file("myconfig.set")

# Find groups
for line in lines:
    if line.startswith('[') and not line.startswith('[{'):
        group_name = extract_name(line)
        # Collect data until blank line or next group
        
    elif line.startswith('[{'):
        text_block_name = extract_name(line)
        # Collect all text until [EOG] or blank line
```

That's it. Split lines on `|`, store in a dictionary or object.

---

## That's Everything You Need

With just these basics you can:
- ✅ Store configuration data
- ✅ Create simple databases
- ✅ Save settings and preferences
- ✅ Handle multi-line text
- ✅ Add comments and documentation

---

## Want More?

**Additional features (all optional):**
- **Complex arrays** - More than two fields: `Protocol|rs232|9600|8|1|n`
- **Custom delimiters** - Use `;` or `,` instead of `|`
- **Single-use fields** - Add extra fields to just one line
- **Alternative encodings** - UTF-16, ASCII, etc.
- **Internationalization** - Locale and text direction settings

See the full specification: [Set File Format Specification v4.0](SetFile_Spec_v4_0.md)

---

## Rules Summary

1. **Filename on first line** (optional but recommended)
2. **Groups** start with `[NAME]`
3. **Data** separated by `|` pipe
4. **Text blocks** use `[{NAME}]` for multi-line content
5. **Escape pipes** in data with `\|`
6. **Comments** are any text outside groups
7. **Blank lines** end groups (or use `[EOG]`)

---

**Start using Set files in 30 seconds:**

1. Create `myfile.set`
2. Add `[SETTINGS]`
3. Add `Key|Value` pairs
4. Done!

---

*This is the minimal Q-Set approach. For the complete specification, see [SetFile_Spec_v4_0.md](SetFile_Spec_v4_0.md)*
