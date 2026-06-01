# Development Guidelines

## ⛔ STOP — Git Check (BEFORE ANY FILE CHANGE)

This is the FIRST thing to do on EVERY project interaction. NO EXCEPTIONS. Do NOT create, edit, or delete any file until this passes.

1. Run `git rev-parse --is-inside-work-tree` in the project root
2. If NOT a git repo:
   - `git init`
   - Create `.gitignore` (respect existing ignores)
   - `git add . && git commit -m "chore: initial commit - base project"`
3. If git exists but has no commits:
   - `git add . && git commit -m "chore: initial commit - base project"`
4. NEVER work on `main`/`master` — create a branch first:
   - `feature/`, `fix/`, `refactor/`, `chore/` prefixes
5. Only THEN proceed with the actual task.

Violation = broken diff history, no rollback possible, user loses trust.

---

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

## Git & File Exclusion Rules

**MANDATORY:** Always exclude AI agent temporary files from version control and archives.

### .gitignore
`.sisyphus/` MUST be in `.gitignore` for every project. This folder contains AI agent run data and temporary files that should never be committed.

### Zip/Archive Exclusion
When creating zip archives or distributing projects, ALWAYS exclude:
- `.sisyphus/` — AI agent temporary files
- `node_modules/` — Dependencies (reinstall via `npm install`)
- `vendor/` — PHP dependencies (reinstall via `composer install`)
- `.git/` — Version control metadata

### Zip Naming Convention (MANDATORY)
Format: `[ProjectName]-update[DD-MM-YYYY].zip`

Example:
- Project: `RRI`, Date: 1 Juni 2026 → `RRI-update01-06-2026.zip`
- Project: `MyApp`, Date: 25 Desember 2025 → `MyApp-update25-12-2025.zip`

Zip command with proper naming:
```bash
zip -r [ProjectName]-update[DD-MM-YYYY].zip . -x ".sisyphus/*" "node_modules/*" "vendor/*" ".git/*"
```

**Never commit or distribute `.sisyphus/` folder.**

## SQL Dump Rules (MANDATORY)

**NEVER use collation with `_ai_ci` suffix in SQL dumps.**

When generating, exporting, or writing SQL dumps (MySQL/MariaDB):

1. **FORBIDDEN collations** — any collation ending with `_ai_ci`:
   - `utf8mb4_0900_ai_ci`
   - `utf8mb4_de_pb_0900_ai_ci`
   - `utf8mb4_es_0900_ai_ci`
   - Any other `*_ai_ci` variant

2. **USE these collations instead:**
   - `utf8mb4_general_ci` (default, widely compatible)
   - `utf8mb4_unicode_ci` (better Unicode sorting)

3. **When dumping from MySQL 8.0+** (which defaults to `_ai_ci`):
   - Always add `--collation-server=utf8mb4_general_ci` flag, OR
   - Post-process the dump: replace all `_ai_ci` collations with `_general_ci`
   - Example: `sed -i 's/utf8mb4_0900_ai_ci/utf8mb4_general_ci/g' dump.sql`

4. **When writing SQL manually:**
   - Always specify `COLLATE utf8mb4_general_ci` explicitly in `CREATE TABLE` statements
   - Never rely on server default collation

**Why:** `_ai_ci` collations are MySQL 8.0+ specific and cause compatibility issues when importing to older MySQL versions or MariaDB.

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

**Linux/macOS:**
```bash
cp ~/.config/opencode/opencode.json ~/.config/opencode/opencode.json.bak.$(date +%Y%m%d)
```

**Windows (PowerShell):**
```powershell
Copy-Item -LiteralPath "$env:USERPROFILE\.config\opencode\opencode.json" -Destination "$env:USERPROFILE\.config\opencode\opencode.json.bak.$(Get-Date -Format 'yyyyMMdd')"
```

This is non-negotiable. Config changes can break the entire environment.



## Tools and MCP Servers

### Active Plugins (via opencode.json)

These plugins are loaded automatically. Understand what each provides:

| Plugin | What It Does |
|---|---|
| `oh-my-openagent` | Agent management, LSP integration, browser automation, category system |
| `@tarquinen/opencode-dcp` | Dynamic Context Pruning — automatic conversation compression to maintain context quality |
| `opencode-supermemory` | Persistent memory across sessions — store/recall project knowledge, patterns, preferences |
| `@ramtinj95/opencode-tokenscope` | Token usage analysis — track and optimize context consumption |
| `opencode-wakatime` | Time tracking — logs coding activity to WakaTime |
| `opencode-sync-plugin` | Session sync across devices |
| `cc-safety-net` | Safety guardrails — prevents dangerous operations |

### Context7 (via Librarian Agent)

Context7 is NOT an MCP server — it's accessed through the `librarian` agent's built-in tools (`context7_resolve-library-id` and `context7_query-docs`).

Use proactively when working with:
- Library/API documentation
- Code generation examples
- Setup or configuration steps
- Framework-specific patterns

Fire the `librarian` agent instead of calling Context7 directly.

### Playwright (Built-in Skill)

Playwright is NOT an MCP server — it's a built-in skill (`/playwright`).

Use for browser automation and testing:
- E2E testing
- Web scraping
- UI interaction testing
- Screenshot generation

### LSP Support

LSP integration is enabled through OhMyOpenAgent. Currently installed language servers:

- **PHP**: `intelephense` ✅ (installed)

**Not yet installed** (install when needed):
- TypeScript/JavaScript: `npm i -g typescript-language-server`
- Python: `pip install basedpyright-langserver`
- JSON/CSS/HTML/ESLint: `npm i -g vscode-langservers-extracted`
- YAML: `npm i -g yaml-language-server`
- Bash: `npm i -g bash-language-server`

Use LSP diagnostics (`lsp_diagnostics` tool) after editing supported source files.

### PDF Reading

Use the built-in `look_at` tool or `read` tool for PDFs first. If text extraction is incomplete, use the `multimodal-looker` agent for visual/document analysis.

### Supermemory (Persistent Memory)

Use `supermemory` tool to:
- `search` — find relevant stored knowledge
- `add` — store new project patterns, decisions, preferences
- `profile` — view user profile
- `list` — see recent memories
- `forget` — remove outdated memories

Store important decisions, project conventions, and learned patterns proactively.

## Agent Usage Guidelines

### Active Agents Overview

All agents below are configured in `oh-my-openagent.json`. Model and variant determine capability — higher variant = deeper reasoning but slower.

---

### Tier 1: Core Agents (Use Daily)

**sisyphus** (default) — Orchestrator & implementer. `qwen3.7-max` / max
- USE: Multi-step implementation, task decomposition, delegation to other agents
- This is the default agent. Most work starts here.
- Sisyphus delegates to specialists, does NOT implement everything itself.

**oracle** — Read-only high-IQ consultant. `glm-5.1` / max
- USE: Architecture decisions, hard debugging (after 2+ failed attempts), security/performance concerns, multi-system tradeoffs
- DO NOT: Ask for implementation — oracle is READ-ONLY
- EXPENSIVE. Consult only when the problem genuinely requires deep reasoning.

**explore** — Codebase grep. `deepseek-v4-flash` / high
- USE: "Where is X?", "Find all files that...", "Which module handles Y?"
- CHEAP and fast. Fire liberally in parallel (2-5 at once).
- DO NOT: Ask it to implement or modify code.

**librarian** — External reference search. `deepseek-v4-flash` / high
- USE: Library docs, OSS examples, framework best practices, unfamiliar packages
- Searches GitHub, Context7, web — NOT your local codebase (that's explore).
- Fire immediately when unfamiliar libraries are involved.

---

### Tier 2: Specialist Agents (Use When Needed)

**prometheus** — Complex problem solver. `glm-5.1` / max
- USE: Difficult debugging, performance optimization, complex refactoring
- vs oracle: Prometheus IMPLEMENTS solutions, oracle only ADVISES. Use prometheus when you need code changes, oracle when you need a decision.

**metis** — Code analyst & pre-planner. `qwen3.7-max` / high
- USE: Pre-planning analysis, identifying hidden intentions in ambiguous requests, scope clarification
- vs oracle: Metis analyzes REQUIREMENTS, oracle analyzes ARCHITECTURE.

**momus** — Plan critic. `qwen3.7-max` / high
- USE: Review work plans for clarity, verifiability, completeness BEFORE implementation
- Invoke with plan file path as prompt. Catches gaps and ambiguities early.

**hephaestus** — Heavy implementation. `deepseek-v4-pro` / max
- USE: Large-scale implementation tasks requiring sustained effort and deep code generation
- vs executor: Hephaestus handles COMPLEX multi-file work, executor handles SIMPLE well-defined tasks.

**executor** — Fast implementer. `deepseek-v4-pro` / high
- USE: Well-defined features, bug fixes, routine tasks with clear scope
- DO NOT: Give it ambiguous or open-ended tasks — it will guess and ship.

**reviewer** — Code quality reviewer. `deepseek-v4-pro` / max
- USE: Quality checks, best practices validation, pattern enforcement
- vs code-reviewer: Reviewer checks BEST PRACTICES, code-reviewer checks SMELLS & RISKS (see below).

**code-reviewer** (custom) — Smell detector & risk finder. Read-only.
- USE: Focused code review on diffs — security, performance, architecture, concurrency smells
- Outputs: What, Where, Why, Fix — ranked by severity.
- vs reviewer: Code-reviewer is SKEPTICAL and EVIDENCE-BASED, reviewer is BEST-PRACTICES oriented.

**tester** — Testing strategist. `deepseek-v4-pro` / high
- USE: Test generation, coverage analysis, test architecture design

**security-auditor** — Security specialist. `glm-5.1` / max
- USE: Vulnerability assessment, penetration testing mindset, security best practices
- vs reviewer: Security-auditor goes DEEP on security only, reviewer covers broader quality.

**refactorer** — Code cleanup. `deepseek-v4-pro` / high
- USE: Technical debt reduction, pattern improvements, code restructuring
- DO NOT: Use for bug fixes — refactorer changes structure, not behavior.

**doc-writer** — Documentation. `qwen3.7-max` / high
- USE: API docs, technical writing, README files

**deep-thinker** (custom) — Structured thinking partner. Read-only.
- USE: Ambiguous challenges, difficult decisions, breaking down complexity
- Asks clarifying questions, applies frameworks (five-whys, jobs-to-be-done, etc.), outputs structured analysis with next actions.
- vs metis: Deep-thinker explores the PROBLEM SPACE, metis analyzes the TASK SCOPE.

**multimodal-looker** — Media analyzer. `mimo-v2.5-pro`
- USE: PDFs, images, diagrams that need interpretation beyond raw text
- Extracts specific information from visual/document content.

**atlas** — Sustained worker. `deepseek-v4-pro` / high
- USE: Long-running tasks that need consistent effort without deep reasoning

---

### Categories (Task Delegation System)

Categories are domain-optimized configurations used via `task(category="...")`. Each category uses a model tuned for that domain.

| Category | Model | When to Use |
|---|---|---|
| `visual-engineering` | mimo-v2.5-pro | UI, CSS, styling, layout, animation, frontend components |
| `artistry` | glm-5.1 | Complex creative problem-solving beyond standard patterns |
| `ultrabrain` | glm-5.1 | Hard logic, algorithms, architecture — give goal only, not steps |
| `deep` | (varies) | Autonomous research + implementation, ONE goal per call |
| `quick` | deepseek-v4-flash | Trivial: single-file changes, typo fixes, simple mods |
| `implementation` | deepseek-v4-pro | General implementation tasks |
| `review` | deepseek-v4-pro | General review tasks |
| `testing` | deepseek-v4-pro | General testing tasks |
| `security` | glm-5.1 | Security-focused tasks |
| `writing` | qwen3.7-max | Documentation, prose, technical writing |
| `unspecified-low` | deepseek-v4-flash | Low-effort tasks that don't fit other categories |
| `unspecified-high` | deepseek-v4-pro | High-effort tasks that don't fit other categories |

**MANDATORY**: Visual work ALWAYS uses `visual-engineering`. Never delegate UI/styling to other categories.

---

### Decision Tree: Which Agent/Category?

```
Need to FIND something in codebase?
  → explore (internal) / librarian (external docs)

Need to UNDERSTAND a problem before acting?
  → deep-thinker (ambiguous problem)
  → metis (task scope analysis)
  → momus (plan review)

Need a DECISION or ADVICE (read-only)?
  → oracle (architecture/debugging)

Need to IMPLEMENT something?
  → Is it visual/UI? → category: visual-engineering
  → Is it trivial? → category: quick
  → Is it complex multi-file? → hephaestus or category: deep
  → Is it well-defined? → executor or category: implementation
  → Is it creative/unconventional? → category: artistry

Need to REVIEW code?
  → Diff-based smell detection? → code-reviewer
  → Best practices check? → reviewer
  → Security focus? → security-auditor or category: security

Need to TEST?
  → tester or category: testing

Need to REFACTOR?
  → refactorer (structure changes only, not bug fixes)

Need DOCUMENTATION?
  → doc-writer or category: writing
```

---

### Anti-Patterns (DO NOT)

- **Don't use oracle for implementation** — it's read-only, will only advise
- **Don't use explore for external docs** — it only searches YOUR codebase
- **Don't use librarian for local code** — it searches GitHub/web/docs
- **Don't use executor for ambiguous tasks** — it will guess and ship without asking
- **Don't use refactorer for bug fixes** — it changes structure, not behavior
- **Don't fire oracle for trivial questions** — it's expensive, use explore/librarian first
- **Don't skip categories for visual work** — `visual-engineering` is mandatory for UI tasks

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
