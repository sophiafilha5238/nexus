# Contributing to NEXUS

NEXUS is a single-file Claude Code subagent (`agents/nexus.md`) — a Markdown file with YAML frontmatter, not a codebase. Contributions are welcome and easy to review since there's one file that matters.

## Ways to contribute

- **Translations** — an English (or any other language) version of the agent instructions, as a separate file (e.g. `agents/nexus.en.md`), keeping the original Portuguese version intact.
- **Report format tweaks** — improvements to the impact-report structure, as long as they stay concise and scannable.
- **New stop conditions** — if you find a real-world scenario where NEXUS should pause and ask but currently doesn't, open an issue or PR describing the scenario.
- **Bug reports** — if NEXUS reports something as broken that isn't, or misses a genuine inconsistency, open an issue with a minimal repro (what changed, what NEXUS said, what it should have said).

## Guidelines

- Keep `agents/nexus.md` as plain, literal instructions — this is a prompt, not code. Avoid vague adjectives ("be smart about X") in favor of concrete rules.
- Any change to the **Stop Conditions** or **Scope** sections needs a clear justification in the PR description — these exist to keep the agent safe to run against real, live codebases.
- Don't add tool permissions (the `tools:` frontmatter field) beyond what's needed for the described procedure.
- Test your change against a real project before submitting — describe what you tested in the PR.

## Reporting issues

Open a GitHub issue with:
1. What changed in your codebase (the input NEXUS was given).
2. What NEXUS reported.
3. What you expected instead.

## License

By contributing, you agree your contributions are licensed under the project's [MIT License](LICENSE).
