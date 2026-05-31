<div align="center">

# Test Audit

**You don't know what to test. This skill does.**

The test judgment a senior developer has — available to everyone.

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue?logo=anthropic)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)]()
[![GitHub Stars](https://img.shields.io/github/stars/gtskevin/test-audit?style=social&logo=github)](https://github.com/gtskevin/test-audit)

</div>

---

> [!NOTE]
> **Who is this for?** You build things with AI coding agents. You're not sure when code needs tests, what kind of tests, or how many. Test-audit makes those decisions for you — then writes and runs the tests.

## The Gap This Fills

There are plenty of testing tools. But they all assume you already know **what to test**:

| Tool | What it does | What it assumes |
|------|-------------|-----------------|
| **TDD skills** | Enforces "write test first" process | You know WHAT to test and HOW DEEP |
| **Coverage tools** | Measures % of lines covered | You interpret the number yourself |
| **"Write tests for X"** | Generates tests for a specific file | You chose the file and scope |
| **Copilot test gen** | Suggests tests for your current file | You know which file matters |

**Test-audit is different.** It answers the question you can't: *given these code changes, what actually needs testing?*

```
A senior developer looks at your diff and thinks:
  "Type definitions? → Skip."
  "Auth routes? → High risk. Test every path."
  "Helper function? → Happy path + one edge case."
  "Database query? → Test with isolation."

Test-audit encodes this judgment.
```

## ✨ Highlights

- **Makes test decisions for you** — Reads your diff, decides which files need tests, what type, and how deep. Zero input required.
- **Finds real bugs, not just coverage gaps** — Found 3 production bugs in its first project (route ordering, missing table, missing column).
- **Respects your project** — Reads your conftest, reuses your helpers, follows your patterns. Never introduces new dependencies.
- **Converges, doesn't retry** — 2-retry limit forces understanding over blind repetition.

## 🎯 Who Uses This

**The vibe coder** — You build with AI agents but aren't a professional developer. You don't have the "test intuition" that comes from years of shipping. Test-audit gives you that intuition.

**The solo developer shipping fast** — You know testing matters but don't have time to think about coverage strategy. Run `/test-audit` before each commit as a safety net.

**The team lead reviewing PRs** — Use test-audit to quickly assess whether a PR has adequate test coverage without reading every line.

## 📦 Installation

⏱️ Get started in 30 seconds

```bash
# Clone and install (one file)
git clone https://github.com/gtskevin/test-audit.git
mkdir -p ~/.claude/skills/test-audit
cp test-audit/skill.md ~/.claude/skills/test-audit/skill.md
```

## ⚡ Quick Start

1. Install the skill (above)
2. Open Claude Code in any project with code changes
3. Type: `/test-audit`
4. Expected output:

```
📁 Files reviewed: 5
   → 2 files skipped (type definitions, constants)
   → 3 files need testing
🧪 Tests added: 12 (3 new test files)
✅ Result: ALL PASS
🐛 Bugs found: 1 (route ordering in analyses.py)
   → Scanned similar routes: 2 more issues found
→ Can commit? Fix the bugs first, then YES
```

## 🆚 How It's Different

### vs. TDD Skills

TDD says "write the test first." But **which test?** Testing an auth route needs per-path coverage. Testing a helper function needs one happy path. How do you know? Experience.

Test-audit **has that experience built in.** It reads the code and makes the judgment call — so you don't need years of practice to know what "good testing" looks like.

### vs. "Write tests for X"

When you say "write tests for auth.py," you've already made the decision that auth.py needs tests. But what if the real risk is in a helper that auth.py calls? Or what if auth.py already has great coverage, but a new migration file broke the schema?

Test-audit scans **your entire diff**, not just one file you pointed at.

### vs. Coverage Reports

Coverage says "87% of lines are executed." It doesn't tell you:
- Whether the 13% gap matters
- Whether the 87% tests are actually asserting anything meaningful
- Whether a file with 100% coverage is testing the right things

Test-audit makes **qualitative** judgments, not just quantitative.

## 📖 How It Works

<div align="center">

```
  You run /test-audit
          │
          ▼
  ┌─────────────────────────────┐
  │  Step 1: Locate Changes     │  git diff — what changed?
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 2: Judge Each File    │  ← The core value
  │                             │
  │  Worth testing? → Skip or   │
  │  What type?      → Decide   │
  │  How deep?       → Commit   │
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 3: Read the Room      │  conftest, existing tests, DB patterns
  │                             │  ⚡ Prevents 80% of environment failures
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 4: Write & Run Tests  │  Reuse project patterns & helpers
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 5: Converge           │  Max 2 retries per file.
  │                             │  Same error twice? Re-understand.
  │                             │  Bug found? Record + scan for similar.
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  ✅ Can I commit?           │
  │  Judgment + Results + Bugs  │
  └─────────────────────────────┘
```

</div>

### The Judgment Engine (Core Value)

For each changed file, test-audit makes three decisions that experienced developers make instinctively:

**1. Is this file worth testing?**
```
Type definitions, constants, pure CSS     → Not worth it
Helper functions, simple data transforms   → Probably worth it
API endpoints, database operations, auth   → Definitely worth it
```

**2. What type of test?**
```
Pure logic function    → Unit test
API route with auth    → Integration test via TestClient
React component        → Test the logic, skip the rendering if deps are heavy
Database query         → Test with real DB, not mocks
```

**3. How deep?**
```
Auth/permissions/data isolation   → Every path: happy + unauthorized + edge cases
Simple helper                     → Happy path + one boundary value
Complex state machine             → State transitions table
```

These aren't hard rules — they're guidelines the AI applies with **judgment based on your actual code.**

### Bug Discovery (Bonus)

While writing tests, test-audit often discovers bugs in the code being tested:

| Bug Found | Root Cause | How test-audit caught it |
|-----------|-----------|------------------------|
| Route 422 on valid endpoint | Fixed path matched by `{param}` route | Test got unexpected 422 → investigated route ordering |
| `no such table` | Table only in migration, not schema init | Test DB creation failed → checked `_ensure_schema` |
| `no such column` | Column not in schema init | Test assertion failed → checked `_ensure_table_columns` |

When one bug is found, it scans for similar issues (one discovery, batch investigation).

### The 2-Retry Rule

Most AI agents keep retrying failing tests with minor variations. Test-audit enforces a strict limit:

```
1st failure → Read error, locate cause, fix
2nd failure → STOP. Re-understand the problem.
              Test infrastructure misunderstood? → Re-read conftest
              Production code has a bug?         → Record it, move on
```

**Same error twice = your mental model is wrong** — not that you need another try.

## 🌐 Cross-Agent Compatibility

While designed for Claude Code, the skill's instructions are agent-agnostic:

| Agent | How to Use |
|-------|-----------|
| **Claude Code** | `mkdir -p ~/.claude/skills/test-audit && cp skill.md ~/.claude/skills/test-audit/skill.md` |
| **Codex** | Copy skill content into `AGENTS.md` as a workflow instruction |
| **Gemini CLI** | Copy into `GEMINI.md` as an available skill definition |
| **Cursor / Windsurf** | Copy into `.cursorrules` or `.windsurfrules` |

## 📊 Real Results

From 7 rounds of test-audit on a production FastAPI + React project:

- **3 files correctly skipped** (type definitions, CSS, constants) — not everything needs tests
- **35 new tests** added across 3 test files — right depth for each file's risk level
- **3 real bugs found** in production code — test-audit caught what the developer missed
- **0 false positives** — every finding was a real issue
- Skill's pitfall patterns refined from these real failures

## ❓ FAQ

<details>
<summary>Q: I'm a senior developer. Do I need this?</summary>

Maybe not for judgment — you already know what to test. But you might still find value in: (1) the automated diff scanning, (2) bug discovery during test writing, and (3) the convergence discipline that prevents AI agents from infinite retry loops.
</details>

<details>
<summary>Q: Does it work with my test framework?</summary>

Yes. test-audit reads your existing test files and conftest to understand your framework (pytest, vitest, jest, Go testing, etc.) before writing any tests. It adapts to whatever you're using.
</details>

<details>
<summary>Q: What if my project has no tests at all?</summary>

test-audit will automatically set up the test infrastructure — install dependencies, create configuration, and verify the environment — before writing the first test. No manual setup needed.
</details>

<details>
<summary>Q: How is this different from Copilot's "generate tests"?</summary>

Copilot generates tests for the file you have open. You chose the file. You decided it needs tests. test-audit scans your entire diff, decides which files matter, and determines the right depth — then writes the tests. It's the difference between having a typist and having a test architect.
</details>

<details>
<summary>Q: Can I use it with non-Python projects?</summary>

Yes. The judgment engine (is it worth testing, what type, how deep) is language-agnostic. The workflow adapts to any test framework.
</details>

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## ⭐ Star History

[![Star History](https://img.shields.io/github/stars/gtskevin/test-audit?style=for-the-badge&logo=github&color=f59e0b&label=%E2%AD%90%20Star%20History)](https://star-history.com/#gtskevin/test-audit&Date)

---

<div align="center">
<sub>Built with 🧪 by <a href="https://github.com/gtskevin">@gtskevin</a> — Test judgment for everyone.</sub>
</div>
