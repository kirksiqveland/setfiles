# Set File Format

**Human-readable files for settings, configuration and data storage**

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Version](https://img.shields.io/badge/version-4.0-blue.svg)](https://setfiles.org)

Set files (.set or .qset) are a simple, flexible format for storing configuration data, settings, and structured information.

**Think:** CSV meets INI files, but all grown up and actually readable.

---

## Features

- ✅ **Human readable and editable** - No quote escaping nightmares
- ✅ **Flexible structure** - Mix key-value pairs, tables, and text blocks
- ✅ **Simple parsing** - Split on delimiter, done
- ✅ **Multi-line text** - Text blocks handle it naturally
- ✅ **No type coercion issues** - Everything is explicit
- ✅ **Comments supported** - Document your configuration

---

## Quick Example

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

[{WELCOME_MESSAGE}]
Welcome to My Application!

Get started by configuring your settings above.
[EOG]
```

---

## Getting Started

### 5-Minute Quick Start

1. **Create a file** with `.set` extension
2. **Add a group**: `[GROUPNAME]`
3. **Add data**: `Key|Value` or `Field1|Field2|Field3`
4. **Done!**

[Read the Quick Start Guide →](https://setfiles.org/Main/QuickStart)

### Documentation

- 📘 **[Specification](docs/SetFile_Spec_v4_0.md)** - Complete format definition
- 📗 **[Implementation Guide](docs/SetFile_ImplementationGuide_v4_0.md)** - Build parsers and use advanced features
- 🚀 **[Quick Start](docs/SetFile_QuickStart.md)** - Learn in 5 minutes
- 📋 **[Examples](examples/)** - See Set files in action

### Website

**[setfiles.org](https://setfiles.org)** - Documentation, examples, and community

---

## Installation

### For Users (Reading/Writing Set Files)

No installation needed! Set files are plain text. Use any text editor.

### For Developers (Parsing Set Files)

**JavaScript/Node.js:**
```bash
npm install setfile
```

**Python:**
```bash
pip install setfile
```

**Other languages:** See [available parsers](https://setfiles.org/Implementations/Parsers)

---

## Repository Structure

```
setfile/
├── docs/                          # Documentation
│   ├── SetFile_Spec_v4_0.md
│   ├── SetFile_ImplementationGuide_v4_0.md
│   └── SetFile_QuickStart.md
├── examples/                      # Example Set files
│   ├── simple-config.set
│   ├── database-records.set
│   ├── multilingual.set
│   └── README.md
├── implementations/               # Parser libraries
│   ├── javascript/
│   ├── python/
│   └── README.md
├── tests/                        # Test files and validation
│   └── test-cases/
├── LICENSE                       # CC BY 4.0
└── README.md                     # This file
```

---

## Basic Syntax

### Groups

```
[GROUPNAME]
Key|Value
AnotherKey|Another Value
```

### Tables

```
[USERS]
{id|name|email}
1|Alice|alice@example.com
2|Bob|bob@example.com
```

### Text Blocks

```
[{DESCRIPTION}]
Multi-line text content.
No escaping needed!
[EOG]
```

### Comments

```
This is a comment outside any group.

[DATA]
Field|Value
```

---

## Why Set Files?

### vs. JSON
- No quote escaping hell
- Natural multi-line text
- Comments supported
- Human-editable without breaking syntax

### vs. YAML
- Simpler spec with fewer edge cases
- More predictable parsing
- No indentation sensitivity

### vs. CSV
- Multiple data sets in one file
- Comments and documentation
- Text blocks for descriptions
- Mixed data types

### vs. INI
- Structured data support
- Tables with field definitions
- Proper text block handling
- More flexible

---

## Contributing

We welcome contributions! See [Contributing Guidelines](CONTRIBUTING.md).

Ways to contribute:
- **Parser implementations** in new languages
- **Example files** and use cases
- **Documentation improvements**
- **Bug reports** and feature requests

---

## Community

- **Website:** [setfiles.org](https://setfiles.org)
- **Discussions:** [GitHub Discussions](https://github.com/kirksiqveland/setfile/discussions)
- **Issues:** [GitHub Issues](https://github.com/kirksiqveland/setfile/issues)
- **Contact:** kirk@setfiles.org

---

## Version

**Current Version:** 4.0 (November 2025)

### What's New in v4.0

- Removed mandatory preamble → Use optional `[THIS-FILE]` group
- Simplified escape sequences → Only `\|` needed
- Better organization → Spec separated from implementation guide
- UTF-8 direct input emphasized

[See version history →](docs/SetFile_Spec_v4_0.md#version-history)

---

## License

**Specification:** [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)

**Copyright © 2025 Kirk Siqveland**

You are free to:
- Share and adapt the material for any purpose
- Create your own parsers and implementations
- Use Set files commercially or non-commercially

Just provide appropriate attribution.

**Implementations:** Each parser/library uses its own license.

---

## Credits

**Created and maintained by [Kirk Siqveland](https://github.com/kirksiqveland)**

Special thanks to all [contributors](https://github.com/kirksiqveland/setfile/graphs/contributors).

---

## Related Projects

- **[EMU Protocols](https://emucode.org)** - Embedded MarkUp tools
- **[Aikode](https://aikode.org)** - Development projects

---

**[Get Started →](https://setfiles.org/Main/QuickStart)** | **[Read the Spec →](docs/SetFile_Spec_v4_0.md)** | **[See Examples →](examples/)**
