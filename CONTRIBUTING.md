# Contributing to Set Files

Thank you for your interest in contributing to Set Files! This document provides guidelines and information for contributors.

---

## Ways to Contribute

### 1. Report Issues

Found a bug or unclear documentation?

- Check [existing issues](https://github.com/kirksiqveland/setfile/issues) first
- Use the issue template
- Provide clear examples and expected vs. actual behavior
- Include Set file examples that demonstrate the issue

### 2. Suggest Features

Have an idea for improvement?

- Open a [discussion](https://github.com/kirksiqveland/setfile/discussions) first
- Explain the use case and benefit
- Consider backward compatibility
- Be open to feedback

### 3. Improve Documentation

Documentation can always be better:

- Fix typos and grammar
- Clarify confusing sections
- Add more examples
- Translate to other languages (future)

### 4. Create Examples

Share your Set file use cases:

- Real-world applications
- Creative uses
- Best practices demonstrations
- Edge cases for testing

### 5. Write Parser Implementations

We need parsers in many languages! See [Parser Guidelines](#parser-implementations) below.

---

## Getting Started

1. **Read the docs**
   - [Quick Start](docs/SetFile_QuickStart.md)
   - [Specification v4.0](docs/SetFile_Spec_v4_0.md)
   - [Implementation Guide](docs/SetFile_ImplementationGuide_v4_0.md)

2. **Try Set files**
   - Create some test files
   - Parse them with existing implementations
   - Understand the format hands-on

3. **Join the discussion**
   - Introduce yourself in [Discussions](https://github.com/kirksiqveland/setfile/discussions)
   - Ask questions
   - Share your plans

4. **Start small**
   - Look for issues tagged "good first issue"
   - Fix typos or improve examples
   - Get familiar with the contribution process

---

## Contribution Process

### For Small Changes (Typos, Documentation)

1. Fork the repository
2. Make your changes
3. Submit a pull request
4. Respond to feedback

### For Larger Changes (Features, Parsers)

1. Open a [discussion](https://github.com/kirksiqveland/setfile/discussions) first
2. Get feedback on your approach
3. Fork the repository
4. Make your changes
5. Submit a pull request
6. Respond to feedback and reviews

---

## Parser Implementations

### Guidelines

**Requirements:**
- Must support Set File Format Specification v4.0
- Must handle all core features (sections 1-4 of spec)
- Should include tests
- Should include documentation
- Should follow language conventions

**Optional Features:**
- Advanced features (section 5 of spec)
- SetQL query language
- SetTag extensions
- Additional utilities

### Structure

Place your parser in `implementations/{language}/`:

```
implementations/javascript/
├── README.md              # Language-specific docs
├── package.json          # Dependencies
├── src/                  # Source code
├── tests/                # Test suite
└── examples/             # Usage examples
```

### Testing

- Include unit tests for core functionality
- Test with files from `tests/test-cases/`
- Test edge cases (empty files, malformed input, etc.)
- Aim for high code coverage

### Documentation

Your parser should include:
- Installation instructions
- Basic usage examples
- API documentation
- Supported features list
- Known limitations

---

## Code Style

### Set File Examples
- Use consistent formatting
- Include comments explaining non-obvious features
- Follow the examples in `examples/` directory
- Use realistic data

### Parser Code
- Follow the conventions of the target language
- Include inline comments for complex logic
- Use clear variable and function names
- Keep functions focused and testable

### Documentation
- Write in clear, simple English
- Use code blocks for examples
- Include expected output where helpful
- Link to relevant sections of the spec

---

## Commit Messages

Use clear, descriptive commit messages:

**Good:**
```
Add Python parser with core v4.0 support
Fix delimiter escaping in JavaScript parser
Update Quick Start with RS232 example
```

**Not as good:**
```
Update
Fix bug
Changes
```

Format:
- Start with a verb (Add, Fix, Update, Remove)
- Be specific about what changed
- Keep first line under 72 characters
- Add details in body if needed

---

## Pull Request Process

1. **Create a branch** with a descriptive name
   - `feature/python-parser`
   - `fix/escape-sequence-bug`
   - `docs/clarify-text-blocks`

2. **Make your changes**
   - Keep changes focused
   - Include tests if applicable
   - Update documentation

3. **Test your changes**
   - Run existing tests
   - Add new tests for new features
   - Verify documentation renders correctly

4. **Submit pull request**
   - Describe what you changed and why
   - Reference related issues
   - Explain any trade-offs or decisions

5. **Respond to review**
   - Address feedback promptly
   - Ask questions if unclear
   - Make requested changes

6. **Merge**
   - Maintainer will merge when approved
   - Your contribution will be credited

---

## Testing

### Test Files

Use test files in `tests/test-cases/`:
- `valid/` - Files that should parse successfully
- `invalid/` - Files that should fail validation
- `edge-cases/` - Unusual but valid files

### Adding Tests

When adding features or fixing bugs:
1. Add test case files
2. Add parser tests
3. Document expected behavior
4. Verify all tests pass

---

## Documentation Style

- Use Markdown for all documentation
- Include code examples in fenced blocks with language tags
- Use Set file syntax highlighting when available
- Link to relevant specification sections
- Keep examples simple and focused

---

## Community Guidelines

### Be Respectful
- Different viewpoints and experiences are valuable
- Critique ideas, not people
- Be welcoming to newcomers

### Be Constructive
- Provide helpful feedback
- Suggest alternatives
- Explain your reasoning

### Be Collaborative
- We're all here to learn and improve
- Share knowledge freely
- Help others succeed

### Be Patient
- Everyone was new once
- Not everyone has the same background
- Technical discussions take time

---

## Questions?

- **General questions:** [GitHub Discussions](https://github.com/kirksiqveland/setfile/discussions)
- **Bug reports:** [GitHub Issues](https://github.com/kirksiqveland/setfile/issues)
- **Direct contact:** kirk@setfiles.org

---

## Recognition

Contributors are recognized in:
- README.md contributors section
- Release notes for their contributions
- Documentation credits

Thank you for contributing to Set Files!

---

**[View Specification →](docs/SetFile_Spec_v4_0.md)** | **[See Examples →](examples/)** | **[Join Discussions →](https://github.com/kirksiqveland/setfile/discussions)**
