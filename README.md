<div align="center">

# Test Audit

**AI writes code fast. But it doesn't write tests.**<br/>
Test Audit fills the gap — automatically.

[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue?logo=anthropic)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)]()
[![GitHub Stars](https://img.shields.io/github/stars/gtskevin/test-audit?style=social&logo=github)](https://github.com/gtskevin/test-audit)

</div>

---

<div align="center">

```
  You run /test-audit
          │
          ▼
  ┌─────────────────────────────┐
  │  Step 1: Locate Changes     │  git diff (uncommitted/staged/last commit)
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 2: Warmup             │  Read conftest, existing tests, DB patterns
  │                             │  ⚡ Prevents 80% of repeat failures
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 3: Coverage Analysis  │  What's already tested? What's missing?
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 4: Write Tests        │  Reuse project patterns & helpers
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  Step 5: Run & Converge     │  Max 2 retries. Bug? Record & continue.
  └─────────────┬───────────────┘
                ▼
  ┌─────────────────────────────┐
  │  ✅ Ready to commit?        │
  │  Files reviewed: N          │
  │  Tests added: N             │
  │  Bugs found: 0 / list       │
  └─────────────────────────────┘
```

</div>

## ✨ Highlights

- **Fully autonomous** — Analyzes changes, writes tests, runs them, reports. Zero interaction.
- **Finds real bugs** — Not just test gaps. Found 3 production bugs in its first project (route ordering, missing table, missing column).
- **Respects your project** — Reads your conftest, reuses your helpers, follows your patterns. Never introduces new dependencies.
- **Converges, doesn't retry** — 2-retry limit forces understanding over blind repetition. The key innovation.

## 🎯 Example Prompts

> Copy any of these into your AI coding agent and see test-audit in action:

| Trigger | What Happens |
|---------|-------------|
| `/test-audit` | Full autonomous audit: locate changes → analyze → write → run → report |
| "check tests before I commit" | Scans uncommitted changes, fills coverage gaps |
| "add tests for the new API endpoints" | Writes targeted tests for changed files |

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
🧪 Tests added: 12 (3 new test files)
✅ Result: ALL PASS
🐛 Bugs found: 1 (route ordering in analyses.py)
→ Can commit? Fix the bug first, then YES
```

## 🆚 Why This Is Different

| Typical "write tests" prompt | test-audit |
|------------------------------|------------|
| You decide what to test | AI analyzes changes and decides |
| Tests may not match project style | Reads conftest + existing tests first |
| Repeated failures → keep retrying | 2-retry limit, then stop and re-understand |
| One file at a time | Full diff scan, prioritize by risk |
| Only finds test gaps | Also finds **bugs in production code** |
| Needs back-and-forth with you | Zero interaction until the final report |

## 📖 How It Works

### The Warmup Step (Secret Weapon)

Before writing any tests, test-audit spends 2 minutes reading your project's test infrastructure:

1. **`conftest.py`** — Finds and reuses your helpers (`make_test_client`, `register_and_login`, `seed_data`)
2. **An existing test file** — Learns your patterns: how DBs are created, how mocking is done
3. **A simple existing test is run** — Confirms the test environment actually works

This prevents 80% of "wrong environment" failures that plague AI-generated tests.

### The 2-Retry Rule (Key Innovation)

Most AI agents keep retrying failing tests with minor variations. Test-audit enforces a strict limit:

```
1st failure → Read error, locate cause, fix
2nd failure → STOP. Re-understand the problem.
              Is it the test infrastructure? → Re-read conftest
              Is it a production code bug?   → Record it, move on
```

**Same error twice means your mental model is wrong** — not that you need another try.

### Bug Discovery (Bonus Feature)

Test-audit doesn't just add tests. In practice, it finds real bugs:

| Bug Found | Root Cause | Detection Method |
|-----------|-----------|-----------------|
| Route 422 on valid endpoint | Fixed path matched by `{param}` route | Test got unexpected 422 → checked route ordering |
| `no such table` | Table only in migration, not in schema init | Test DB creation failed → checked `_ensure_schema` |
| `no such column` | Query referenced column not in schema init | Test assertion failed → checked `_ensure_table_columns` |

When one bug is found, test-audit scans for similar issues (one discovery, batch investigation).

### Judgment Over Rules

The skill doesn't apply a rigid checklist. It reads your code and makes three judgments:

1. **Is this file worth testing?** — Type definitions → skip. API endpoints → test thoroughly.
2. **What type of test?** — Unit, integration, API-level — whichever is most effective.
3. **How deep?** — Auth code → per-path. Helper function → happy path + one edge case.

## 🌐 Cross-Agent Compatibility

While designed for Claude Code, the instructions are agent-agnostic:

| Agent | How to Use |
|-------|-----------|
| **Claude Code** | `mkdir -p ~/.claude/skills/test-audit && cp skill.md ~/.claude/skills/test-audit/skill.md` |
| **Codex** | Copy skill content into `AGENTS.md` as a workflow instruction |
| **Gemini CLI** | Copy into `GEMINI.md` as an available skill definition |
| **Cursor / Windsurf** | Copy into `.cursorrules` or `.windsurfrules` |

## 📊 Real Results

From 7 rounds of test-audit on a production FastAPI + React project:

- **35 new tests** added across 3 test files
- **3 real bugs found** in production code (not test bugs)
- **0 false positives** — every finding was a real issue
- Refined the skill's "Common Pitfall Patterns" table based on real failures

## ❓ FAQ

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

Copilot generates tests for the file you have open. test-audit scans your entire diff, prioritizes by risk, reuses your project's patterns, and enforces a convergence discipline (stop retrying, start understanding). It also finds bugs in production code, not just coverage gaps.
</details>

<details>
<summary>Q: Can I use it with non-Python projects?</summary>

Yes. The workflow (locate changes → understand test patterns → write tests → run → report) is language-agnostic. The "Common Pitfall Patterns" table covers general patterns (route ordering, schema mismatches, bracket errors) that apply across frameworks.
</details>

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## ⭐ Star History

[![Star History](https://img.shields.io/github/stars/gtskevin/test-audit?style=for-the-badge&logo=github&color=f59e0b&label=%E2%AD%90%20Star%20History)](https://star-history.com/#gtskevin/test-audit&Date)

---

<div align="center">
<sub>Built with 🧪 by <a href="https://github.com/gtskevin">@gtskevin</a> — Test more, debug less.</sub>
</div>
