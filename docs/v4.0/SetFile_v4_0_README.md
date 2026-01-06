# Set File Format v4.0 - Documentation

This directory contains the complete Set File Format v4.0 documentation, split into two focused documents:

## 📘 [Set File Format Specification v4.0](SetFile_Spec_v4_0.md)

**The Core Format Definition** (Sections 1-5)

Defines the Set file format itself - syntax, rules, and structure. This is what you need to understand the format and ensure compatibility.

**Contents:**
1. Introduction & Philosophy
2. Minimum Core Specification
3. File Configuration
4. Group Types in Detail
5. Optional Advanced Features

**Read this if you:**
- Want to understand the Set file format
- Need to implement a parser
- Are creating Set files manually
- Need the authoritative format reference

---

## 📗 [Set File Implementation Guide v4.0](SetFile_ImplementationGuide_v4_0.md)

**Patterns, Examples, and Best Practices** (Sections 1-8)

Comprehensive guidance for building Set-based systems. Shows how to use the format effectively and implement advanced features.

**Contents:**
1. Query Language (SetQL)
2. Implementation Patterns & Conventions
3. SetTag Extensions
4. CRUD Operations
5. Validation & Error Handling
6. Programming Interface Guidelines
7. Complete Examples
8. Version History & Migration

**Read this if you:**
- Are building a Set file parser or library
- Need examples and usage patterns
- Want to implement SetQL queries
- Need migration guidance from v3.x

---

## Quick Links

- **[Full Specification](SetFile_Spec_v4_0.md)** - Core format definition
- **[Implementation Guide](SetFile_ImplementationGuide_v4_0.md)** - How to build with it
- **[Quick Reference](SetFile_QuickRef_v4_0.md)** - One-page cheat sheet *(coming soon)*

---

## What Changed in v4.0?

**Major Simplifications:**
- Removed mandatory preamble → Use optional `[THIS-FILE]` group
- Removed `[=KEYVALUE=]` notation → Just use `[GROUPNAME]`
- Removed comment block syntax → Text outside groups is inherently comments
- Simplified escape sequences → Only `\|` needed (delimiter escape)
- Added single-line delimiter override → `:!field!field!field`

**Better Organization:**
- Core spec (5 sections) clearly separated from implementation guidance
- UTF-8 direct input emphasized over escape sequences
- Runtime calculations (`::`) moved to implementation patterns
- Single-use fields (`:::`) properly documented with correct syntax

---

**License:** Creative Commons Attribution 4.0 International (CC BY 4.0)  
**Copyright:** (c) 2025 Kirk Siqveland

**Questions or feedback?**  
Visit: https://github.com/kirksiqveland/setfile
