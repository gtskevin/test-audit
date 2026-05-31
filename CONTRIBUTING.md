# Contributing to Test Audit

Thanks for your interest in improving test-audit!

## How to Contribute

1. **Fork** the repository
2. **Create a branch** for your change: `git checkout -b my-improvement`
3. **Make your changes** to `skill.md` (the core skill file)
4. **Test it** — install the modified skill locally and run `/test-audit` on a real project
5. **Submit a PR** with a description of what changed and how you tested it

## Guidelines

- **Keep it agent-agnostic** — The skill should work with any coding agent (Claude Code, Codex, Gemini CLI, Cursor). Don't add Claude Code-specific tool calls.
- **No project-specific content** — Don't add references to specific frameworks, databases, or project structures. Use generic examples.
- **Judgment over rules** — The skill's strength is flexibility. Don't add rigid checklists that override the AI's ability to read code and decide.
- **Update README if needed** — If you change the skill's behavior, update the README to match.

## Reporting Issues

Found a bug or have a suggestion? Open an [issue](https://github.com/gtskevin/test-audit/issues/new) with:

- What agent and test framework you're using
- What happened vs. what you expected
- Relevant log output

## Good First Issues

Look for issues labeled [`good first issue`](https://github.com/gtskevin/test-audit/labels/good%20first%20issue) for beginner-friendly tasks.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
