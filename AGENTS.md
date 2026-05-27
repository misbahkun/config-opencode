# Development Guidelines

## Coding Principles

### General Guidelines
- Avoid nested if statements
- Follow the single responsibility principle
- Follow the guard clause pattern
- Keep things smart and simple
- Write clean, maintainable code
- Prioritize readability over cleverness

### Code Quality
- Use meaningful variable and function names
- Keep functions small and focused
- Avoid code duplication
- Write self-documenting code
- Add comments only when necessary to explain "why", not "what"

### Error Handling
- Handle errors explicitly
- Provide meaningful error messages
- Use try-catch blocks appropriately
- Don't swallow exceptions silently

### Testing
- Write tests for critical functionality
- Test edge cases and error conditions
- Keep tests simple and focused
- Use descriptive test names

## Architecture Patterns

### Design Principles
- SOLID principles
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- YAGNI (You Aren't Gonna Need It)
- Separation of concerns

### Code Organization
- Organize code by feature, not by type
- Keep related code close together
- Use clear folder structure
- Maintain consistent naming conventions

## Best Practices

### Version Control
- Write clear, descriptive commit messages
- Make small, focused commits
- Review code before committing
- Keep commits atomic

### Documentation
- Document public APIs
- Keep documentation up to date
- Use inline documentation for complex logic
- Maintain a clear README

### Performance
- Optimize only when necessary
- Profile before optimizing
- Consider scalability early
- Avoid premature optimization

### Security
- Validate all inputs
- Sanitize user data
- Use parameterized queries
- Keep dependencies updated
- Never commit secrets or credentials

## Language-Specific Guidelines

### TypeScript/JavaScript
- Use TypeScript for type safety
- Prefer const over let
- Use async/await over callbacks
- Avoid any type when possible
- Use strict mode

### React
- Use functional components with hooks
- Keep components small and focused
- Lift state up when needed
- Use proper key props in lists
- Memoize expensive computations

### Node.js
- Handle async errors properly
- Use environment variables for configuration
- Implement proper logging
- Use middleware for cross-cutting concerns

## OpenCode Configuration Management

### Backup Rule (MANDATORY)
Before modifying ANY opencode configuration file, ALWAYS create a backup first:
- `opencode.json` → `opencode.json.bak.YYYYMMDD`
- `oh-my-openagent.jsonc` → `oh-my-openagent.jsonc.bak.YYYYMMDD`
- Any other config under `~/.config/opencode/` → same pattern

```bash
Copy-Item -LiteralPath "$env:USERPROFILE\.config\opencode\opencode.json" -Destination "$env:USERPROFILE\.config\opencode\opencode.json.bak.$(Get-Date -Format 'yyyyMMdd')"
```

This is non-negotiable. Config changes can break the entire environment.

## Tools and MCP Servers

### Context7 MCP
Use Context7 automatically when you need:
- Library/API documentation
- Code generation examples
- Setup or configuration steps
- Framework-specific patterns

Don't wait for explicit requests - proactively use Context7 when it would help.

### Playwright MCP
Use for browser automation and testing:
- E2E testing
- Web scraping
- UI interaction testing
- Screenshot generation

### LSP Support
OpenCode LSP integration is enabled through OhMyOpenAgent. Common language servers installed on this machine include:
- TypeScript/JavaScript: `typescript-language-server`
- Python: `basedpyright-langserver`
- PHP: `intelephense`
- JSON/CSS/HTML/ESLint: `vscode-langservers-extracted`
- YAML: `yaml-language-server`
- Bash: `bash-language-server`

Use LSP diagnostics after editing supported source files when available.

### PDF Reading
When asked to read or analyze a PDF, first try the built-in file reading tools. If text extraction is incomplete or Markdown output is needed, run:

```bash
python "C:\Users\Misuba\.config\opencode\scripts\pdf_to_md.py" "C:\path\to\file.pdf"
```

To save the Markdown output to a file, provide an output path:

```bash
python "C:\Users\Misuba\.config\opencode\scripts\pdf_to_md.py" "C:\path\to\file.pdf" "C:\path\to\file.md"
```

## Agent Usage Guidelines

### When to Use Specific Agents

**sisyphus** - Ultra-complex, multi-step tasks requiring deep reasoning
- System architecture design
- Complex algorithm implementation
- Multi-service integration

**oracle** - Strategic planning and high-level decisions
- Technology stack selection
- Architecture patterns
- Long-term technical strategy

**prometheus** - Complex problem solving
- Debugging difficult issues
- Performance optimization
- Complex refactoring

**metis** - Intelligent code analysis
- Code review
- Pattern detection
- Optimization suggestions

**executor** - Fast implementation
- Well-defined features
- Bug fixes
- Routine tasks

**reviewer** - Code review
- Security analysis
- Quality checks
- Best practices validation

**tester** - Testing strategy
- Test generation
- Coverage analysis
- Test architecture

**security-auditor** - Security analysis
- Vulnerability assessment
- Security best practices
- Penetration testing mindset

**refactorer** - Code refactoring
- Technical debt reduction
- Code cleanup
- Pattern improvements

**doc-writer** - Documentation
- API documentation
- Technical writing
- README files

**explorer** - Fast codebase exploration
- Finding files and patterns
- Understanding code structure
- Quick searches

## Response Style

### Communication
- Be clear and concise
- Provide context when needed
- Explain complex decisions
- Ask clarifying questions when uncertain

### Code Changes
- Explain what you're changing and why
- Show before/after when helpful
- Highlight potential impacts
- Suggest alternatives when appropriate

## Continuous Improvement

- Learn from mistakes
- Stay updated with best practices
- Refactor when you see opportunities
- Share knowledge through documentation
