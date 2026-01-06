# SET File Examples - Detailed Guide

This guide provides comprehensive information about each example file, including what features they demonstrate, common use cases, and implementation notes.

**Version Note:** These examples were created for SET File Format v4.0 and remain fully compatible with v4.2.

---

## Table of Contents

* [simple-config.set](#simple-configset) - Basic Configuration
* [database-records.set](#database-recordsset) - Relational Data
* [test_data_complete.qset](#test_data_completeqset) - Parser Testing & Edge Cases

---

## simple-config.set

**What it demonstrates:** Basic key-value configuration with logical grouping

### Overview

This example shows the most common Set file pattern: organizing configuration settings into related groups. Each group contains key-value pairs using the pipe delimiter.

### Features Demonstrated

* **Multiple groups** - Related settings organized together
* **Key-value pairs** - Simple `Key|Value` format
* **Clear structure** - Easy to read and maintain
* **Comments** - Preamble provides context

### Use Cases

Perfect for:
- Application configuration files
- Environment settings
- Service connection details
- Feature flags and preferences
- Build configurations

### Key Sections

```
[DATABASE]
Host|localhost
Port|5432
Database|myapp
User|admin
```

```
[APP_SETTINGS]
AppName|My Application
Version|1.0.0
Theme|dark
Language|en-US
```

### Why This Pattern Works

* **Human readable** - Non-technical users can edit safely
* **No nesting confusion** - Flat structure is predictable
* **Easy parsing** - Split on `|`, done
* **Self-documenting** - Group names and keys explain purpose

### Implementation Notes

When parsing this file:
- Lines without `|` are treated as comments or single values
- Empty lines are ignored
- Group names in `[BRACKETS]` start new sections
- Each key appears once per group

---

## database-records.set

**What it demonstrates:** Tabular data with field definitions and multiple related tables

### Overview

This example shows how Set files handle structured, relational data - similar to database tables or CSV files, but with multiple tables in one file and better organization.

### Features Demonstrated

* **Field definitions** - `{field1|field2|field3}` syntax
* **Tabular data** - Multiple rows with consistent columns
* **Multiple tables** - Related data sets in one file
* **Group termination** - `[EOG]` marks end of each table
* **Relational structure** - Tables reference each other (department_id)

### Use Cases

Perfect for:
- Test data sets
- Sample database records
- Lookup tables
- Import/export formats
- Data migration files
- Report templates

### Key Sections

**Employee Table:**
```
[EMPLOYEES]
{id|first_name|last_name|department|hire_date|salary|email}
101|Alice|Smith|Engineering|2023-01-15|95000|alice.smith@example.com
102|Bob|Jones|Marketing|2023-02-20|75000|bob.jones@example.com
[EOG]
```

**Departments Table:**
```
[DEPARTMENTS]
{id|name|manager|budget|location}
1|Engineering|Alice Smith|500000|Building A
2|Marketing|Bob Jones|250000|Building B
[EOG]
```

**Projects Table:**
```
[PROJECTS]
{id|name|department_id|start_date|end_date|status}
1|Website Redesign|2|2025-01-01|2025-06-30|active
2|Mobile App|1|2025-02-01|2025-12-31|active
[EOG]
```

### Why This Pattern Works

* **Multiple tables** - All related data in one file
* **Explicit fields** - Field definitions make structure clear
* **CSV-like** - Familiar to anyone who's worked with spreadsheets
* **No quote escaping** - Unlike CSV, pipes rarely appear in data
* **Termination markers** - Clear boundaries between tables

### Implementation Notes

When parsing tables:
- First line with `{...}` defines field names
- Subsequent lines are data rows
- `[EOG]` ends the table (optional but recommended)
- Field count should match across all rows
- Missing values can be represented as empty between pipes: `value1||value3`

### Advantages Over CSV

1. **Multiple tables in one file** - CSV requires separate files
2. **Built-in field names** - No ambiguity about what columns mean
3. **No quote escaping** - Commas in data don't break parsing
4. **Comments supported** - Document your data
5. **Mixed data types** - Can have both tables and key-value pairs

---

## test_data_complete.qset

**What it demonstrates:** Comprehensive feature coverage and edge case handling

### Overview

This is a complete validation suite for parser implementations. It tests every significant feature of the QSet format (simplified SET-File subset) including edge cases that often trip up parsers.

**Important:** This is a `.qset` file, which uses a simplified SET-File subset. The core syntax is identical, but QSet has some implementation differences.

### Features Demonstrated

* **All data types** - Key-value, tables, text groups, single values
* **Escape sequences** - Escaped pipes `\|` and backslashes
* **Empty fields** - Testing how parsers handle missing data
* **Unicode content** - Multi-language support
* **Text groups** - Code blocks, markdown, special characters
* **Whitespace handling** - Leading/trailing spaces, tabs
* **Numeric data** - Integers, decimals, scientific notation
* **Edge cases** - Empty groups, single rows, duplicate values
* **Special characters** - Quotes, symbols, percent signs
* **Long values** - Testing line length handling
* **Boundary conditions** - First/last rows, empty text groups

### Use Cases

Perfect for:
- **Parser validation** - Ensure your implementation is complete
- **Regression testing** - Verify updates don't break existing features
- **Documentation examples** - Real-world edge cases
- **Learning the format** - See all features in one place
- **Debugging** - Isolate specific parsing issues

### Critical Test Sections

**Escaped Characters:**
```
[ESCAPED_PIPES]
{key|value}
Expression|value > 10 \| value < 5
Path|C:\Windows\System32
Mixed|normal\|pipe\|multiple
```
**Tests:** Backslash-pipe escape sequence, mixed content

**Empty Fields:**
```
[EMPTY_FIELDS]
{col1|col2|col3|col4}
a||c|d
|b|c|
a|b||d
|||
```
**Tests:** Missing data in various positions, completely empty rows

**Trailing Empty Fields:**
```
[TRAILING_EMPTY]
{name|value1|value2|value3}
first|a|b|c
second|x||
third|||
```
**Tests:** Trailing pipes and empty fields at end of rows

**Whitespace Handling:**
```
[WHITESPACE_HANDLING]
{field1|field2|field3}
  leading|  spaces  | trailing
	tabs|	between	|	fields
mixed  |  space 	| and	tabs
```
**Tests:** How parser handles spaces/tabs in various positions

**Text Groups:**
```
[{TEXT_GROUP_CODE}]
function parseQSet(content) {
    const lines = content.split('\n');
    // Process line with | delimiters
    if (line.includes('|')) {
        const fields = line.split('|');
    }
}
[EOG]
```
**Tests:** Code blocks with pipes preserved literally

**Unicode Content:**
```
[UNICODE_CONTENT]
{name|city|greeting}
François|Paris|Bonjour
李明|北京|你好
José|México|Hola
```
**Tests:** UTF-8 multi-language support

**Special Characters in Text Groups:**
```
[{SPECIAL_CONTENT}]
Pipes: | | | | |
Backslashes: \ \ \ \ \
Brackets: [[ ]] {{ }}

Even things like [EOG] mid-line are literal.
Only [EOG] at start of line ends the group.
[EOG]
```
**Tests:** Text groups preserve everything except `[EOG]` at line start

### Parser Implementation Checklist

Use this file to verify your parser handles:

- ✅ **Basic key-value pairs** - [SIMPLE_KEYVALUE]
- ✅ **Tables with field definitions** - [TABLE_WITH_FIELDS]
- ✅ **Variable field counts** - [VARIABLE_FIELD_COUNT]
- ✅ **Empty field handling** - [EMPTY_FIELDS], [TRAILING_EMPTY]
- ✅ **Escape sequences** - [ESCAPED_PIPES], [ESCAPED_BACKSLASH_PIPE]
- ✅ **Text groups** - [TEXT_GROUP_SIMPLE], [TEXT_GROUP_MULTILINE], [TEXT_GROUP_CODE]
- ✅ **Unicode support** - [UNICODE_CONTENT]
- ✅ **Whitespace normalization** - [WHITESPACE_HANDLING]
- ✅ **Long values** - [LONG_VALUES]
- ✅ **Special characters** - [SPECIAL_CHARACTERS], [SPECIAL_CONTENT]
- ✅ **Numeric data** - [NUMERIC_DATA]
- ✅ **Boolean-like values** - [BOOLEAN_LIKE]
- ✅ **Empty groups** - [EMPTY_GROUP], [FIELD_DEF_ONLY], [EMPTY_TEXT_GROUP]
- ✅ **Boundary cases** - [BOUNDARY_CASES], [SINGLE_ROW]
- ✅ **Duplicate values** - [DUPLICATE_VALUES]
- ✅ **Complex tables** - [COMPLEX_TABLE]

### Why This File Matters

**For developers:**
- Comprehensive test coverage
- Documents expected behavior
- Catches edge cases early
- Validates spec compliance

**For users:**
- Shows advanced features
- Demonstrates best practices
- Provides ready-to-use patterns
- Explains escape sequences

### Implementation Notes

**Critical points for parsers:**

1. **Escape sequences:**
   - `\|` becomes literal pipe `|`
   - `\\` becomes literal backslash `\`
   - Only process escapes in data groups (not text groups)

2. **Text groups:**
   - Everything between `[{NAME}]` and `[EOG]` is literal
   - `[EOG]` only ends group when at start of line
   - No escape processing in text groups

3. **Empty fields:**
   - `||` represents empty field
   - Trailing pipes create empty fields
   - All-empty rows are valid: `|||`

4. **Whitespace:**
   - Leading/trailing spaces in fields are typically preserved
   - Empty lines between groups are ignored
   - Whitespace in group/field names is typically not allowed

5. **Field definitions:**
   - `{field1|field2|field3}` defines column names
   - All data rows should match field count
   - Field names should be consistent (no spaces)

---

## Testing Your Parser

To validate a parser implementation:

1. **Parse test_data_complete.qset**
2. **Verify all groups are detected**
3. **Check field definitions are recognized**
4. **Validate data integrity** (no lost fields, proper escape handling)
5. **Test text groups preserve content exactly**
6. **Verify empty field handling**
7. **Check Unicode support**

---

## Creating Your Own Examples

When creating Set files:

### For Configuration Files
```
[SECTION]
Key|Value
AnotherKey|Another Value
```

### For Data Tables
```
[TABLENAME]
{field1|field2|field3}
value1|value2|value3
value1|value2|value3
[EOG]
```

### For Text Content
```
[{TEXTBLOCK}]
Any content here is preserved exactly.
No escaping needed.
[EOG]
```

### For Mixed Content
```
preamble.set

Optional preamble text

[CONFIG]
Setting|Value

[DATA]
{col1|col2}
a|b

[{NOTES}]
Additional documentation
[EOG]
```

---

## Additional Resources

* **[SET File Specification](../docs/v4.2/)** - Complete format definition
* **[setfiles.org](https://setfiles.org)** - Documentation and community
* **[Quick Start](https://setfiles.org/Main/QuickStart)** - 5-minute introduction
* **[Parser Implementations](https://setfiles.org/Parsers/Main)** - Available libraries

---

## Questions or Issues?

- **GitHub Issues:** [Report problems or request examples](https://github.com/kirksiqveland/setfile/issues)
- **GitHub Discussions:** [Ask questions or share use cases](https://github.com/kirksiqveland/setfile/discussions)
- **Email:** kirk@setfiles.org

---

**Version:** Examples created for v4.0, compatible with v4.2  
**Last Updated:** January 2026
