# Test Audit

> **AI writes code fast. But it doesn't write tests.** Test Audit automatically fills the gap — analyzing your changes, deciding what needs testing, writing the tests, running them, and reporting results. Zero interaction required.

A [Claude Code](https://claude.ai/code) Skill that acts as your autonomous test engineer. After you (or AI) write code, invoke test-audit and it handles the rest.

[English](#how-it-works) | [中文说明](#中文说明)

---

## The Problem

If you use AI coding agents, you've seen this pattern:

```
AI generates feature code → Works! Ship it.
                          → Later: bug found in production
                          → Root cause: no tests, untested edge case
```

AI agents are great at writing code but terrible at **deciding what to test and writing those tests**. You could ask "write tests for X" — but now you're doing the analysis work that AI should do for you.

## How It Works

```
  You run /test-audit
          |
          v
  +-----------------------------+
  |  Step 1: Locate Changes     | <-- git diff (uncommitted/staged/last commit)
  +-------------+---------------+
                |
                v
  +-----------------------------+
  |  Step 2: Warmup             | <-- Read conftest, existing tests, DB patterns
  |                             |     (2 min that prevents 80% of repeat failures)
  +-------------+---------------+
                |
                v
  +-----------------------------+
  |  Step 3: Coverage Analysis  | <-- What's already tested? What's missing?
  +-------------+---------------+
                |
                v
  +-----------------------------+
  |  Step 4: Write Tests        | <-- Reuse project patterns & helpers
  |                             |     Never separate "analysis" from "execution"
  +-------------+---------------+
                |
                v
  +-----------------------------+
  |  Step 5: Run & Converge     | <-- Max 2 retries per file
  |                             |     Bug found? Record & continue.
  |                             |     Same error twice? Stop & re-understand.
  +-------------+---------------+
                |
                v
         Concise Report
  +-----------------------------+
  | Files reviewed: N           |
  | Tests added: N              |
  | Result: PASS / FAIL         |
  | Bugs found: 0 / list        |
  |                             |
  | -> Can I commit? YES / NO   |
  +-----------------------------+
```

## Why This Is Different From "AI Write Tests"

| Typical "write tests" prompt | test-audit |
|------------------------------|------------|
| You decide what to test | AI analyzes changes and decides |
| Tests may not match project style | Reads conftest + existing tests first |
| Repeated failures -> keep retrying | 2-retry limit, then stop and re-understand |
| One file at a time | Full diff scan, prioritize by risk |
| Only finds test gaps | Also finds **bugs in production code** |
| Needs back-and-forth with you | Zero interaction until the final report |

## Real Results

In production use on a FastAPI + React project (383 tests):

- **7 rounds of test-audit** over multiple commits
- **3 real bugs found** in production code (not test bugs):
  - Route ordering: fixed path matched by parameterized route -> 422
  - Missing table: table only in migration, not in schema init -> `no such table`
  - Missing column: query referenced column not in schema -> `no such column`
- **35 new tests added** across feedback API, finding classification, and LLM routing
- Skill itself was refined based on these real failures (the "Common Pitfall Patterns" table)

## Install

```bash
# Clone and install
git clone https://github.com/gtskevin/test-audit.git
mkdir -p ~/.claude/skills/test-audit
cp test-audit/skill.md ~/.claude/skills/test-audit/skill.md
```

## Use

```
/test-audit
```

That's it. The skill will:
1. Find your latest code changes (uncommitted, staged, or last commit)
2. Analyze what needs testing
3. Write and run the tests
4. Report: can you commit?

## When to Use

| Trigger | Example |
|---------|---------|
| After AI generates code | "AI just wrote a feature, let me check if it's tested" |
| Before committing | Final safety check before `git commit` |
| After a refactoring session | "Did I break anything?" |
| Periodic coverage audit | "What's missing tests in this module?" |

## Design Principles

**Judgment over rules.** The skill doesn't apply a rigid checklist. It reads your code and decides:
- Is this file worth testing? (Type definitions -> skip. API endpoints -> test thoroughly.)
- What type of test? (Unit, integration, API-level -- whichever is most effective.)
- How deep? (Auth code -> per-path. Helper function -> happy path + one edge case.)

**Respect the project.** Never introduces new test dependencies. Always reads `conftest.py` and reuses existing helpers. Adapts to the project's testing patterns.

**Converge, don't retry.** The 2-retry limit is the key innovation. Most AI agents keep retrying with minor variations. Test-audit forces a stop after 2 failures -- either your understanding is wrong (go back to Step 2) or the code has a bug (record it and move on).

**Find bugs, not just gaps.** Test-audit isn't just about coverage. In practice, it finds real bugs in the code being tested. When a bug is found, it scans for similar issues in the same file (one discovery, batch investigation).

## Cross-Agent Compatibility

While designed for Claude Code, the principles apply to any coding agent:

| Agent | How to Use |
|-------|-----------|
| **Claude Code** | Install as a skill (shown above) |
| **Codex** | Copy the skill content into `AGENTS.md` as a workflow instruction |
| **Gemini CLI** | Copy into `GEMINI.md` as an available skill definition |
| **Cursor / Windsurf** | Copy into your project's `.cursorrules` or `.windsurfrules` |

The skill's instructions are agent-agnostic -- they describe *what to do*, not *which tool to call*.

## 中文说明

### 这个工具解决什么问题？

用 AI 写代码很快，但 AI 不会自动写测试。结果就是：代码写完了，测试没跟上，bug 在生产环境才发现。

Test Audit 是一个自动测试专家——你运行 `/test-audit`，它会：
1. 自动找到你的代码变更
2. 分析哪些文件需要测试
3. 写测试、跑测试
4. 报告结果：**能不能提交？**

全程零交互。它还会发现你代码里的 bug。

### 怎么安装

```bash
git clone https://github.com/gtskevin/test-audit.git
mkdir -p ~/.claude/skills/test-audit
cp test-audit/skill.md ~/.claude/skills/test-audit/skill.md
```

### 怎么使用

在 Claude Code 中输入 `/test-audit`，然后什么都不用做。等它报告结果。

### 和"让 AI 写测试"有什么不同？

- **不需要你告诉它测什么** — 它自己分析代码变更，自己决定
- **会先读懂项目的测试模式** — 读 conftest、已有测试、数据库访问模式，避免 80% 的环境问题
- **2 次失败后必须停下来分析** — 不盲目重试，而是重新理解问题
- **不只补测试，还会发现 bug** — 实际使用中发现了 3 个生产代码的 bug

### 核心原则

- **判断力优先** — 不是套规则表，而是根据代码实际情况决定测什么、测多深
- **尊重项目** — 复用已有的测试框架和 helper，不引入新依赖
- **收敛而非重试** — 同一个错误出现 2 次就停下来重新理解，不盲目修改
- **发现 bug 后批量排查** — 找到一个路由顺序错误，就检查同文件其他路由

## License

MIT
