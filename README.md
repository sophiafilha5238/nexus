# NEXUS

**An architectural coherence subagent for [Claude Code](https://claude.com/claude-code).**

NEXUS doesn't ask *"is this code correct?"* It asks:

> **"Does this change still make sense across the whole system, or did it leave something disconnected?"**

Most review tools check syntax, style, or isolated correctness. NEXUS traces the *blast radius* of a change — a new field, a modified business rule, a renamed function, a changed API contract — across every layer it touches: frontend, backend, database, permissions, tests, and docs. It reports what broke, what's now inconsistent, and what's been orphaned, and it only reports what it actually verified by reading the code — never by guessing from naming conventions.

## Why

Change a screen. Add a field. Update a rule. Rename a function. In any non-trivial codebase, that ripples: a frontend form that still posts the old shape, an API validator that doesn't know about the new state, a test that silently stopped covering the real behavior, a permission check written for a role that no longer applies. Nobody notices until it breaks in front of a user — the classic "witch hunt" of chasing down what a small change quietly broke.

NEXUS exists to close that gap *before* it ships: map the dependency graph around a change, verify each layer against the new reality, and surface exactly what needs attention — with the option to propose or apply the fix.

## Installation

NEXUS is a single Markdown file — a [Claude Code custom subagent](https://docs.claude.com/en/docs/claude-code/sub-agents). No build step, no dependencies.

**Per-project** (shared with your team via git):
```bash
mkdir -p .claude/agents
curl -o .claude/agents/nexus.md https://raw.githubusercontent.com/sophiafilha5238/nexus/main/agents/nexus.md
```

**Global** (available in every project on your machine):
```bash
mkdir -p ~/.claude/agents
curl -o ~/.claude/agents/nexus.md https://raw.githubusercontent.com/sophiafilha5238/nexus/main/agents/nexus.md
```

Claude Code picks up new subagents automatically — no restart required for a fresh session.

## Usage

**Manual invocation:**
```
Use the nexus agent to check the impact of this change.
```

**Proactive invocation:** NEXUS's description is written so Claude Code can invoke it on its own right after you edit something architecturally significant (a shared field, a business rule, an endpoint contract) — you don't have to remember to call it.

### Operating modes

| Mode | Behavior |
|---|---|
| **Analysis** (default) | Investigates and reports only. Never edits anything. |
| **Suggestion** | Investigates, reports, and proposes the exact diff for each fix — without applying it. |
| **Auto-fix** | Only activates when explicitly requested ("apply it", "fix it", "auto-fix"). Still bound by the stop conditions below. |

### Example

> You added a `department` field to the user registration form.

NEXUS traces it end to end and reports something like:

```
## Change detected
department field — UserRegistrationForm.tsx

## Impact
🔴 Confirmed breaks
- api/users.py:42 — POST /users doesn't accept `department`, request will 422

🟡 Inconsistencies
- UserService.ts:18 — createUser() type doesn't include department, TS will complain downstream
- AdminDashboard.tsx:130 — user list column config has no department entry

🟢 Orphans
- none found

## Recommendation
1. Add `department` to the POST /users request schema
2. Update the UserService.ts create-user type
3. Add a department column to AdminDashboard, or confirm it's intentionally omitted
```

## How it works

NEXUS doesn't pre-index your whole codebase on every run. Given what changed — a file, a symbol, or a diff it discovers itself via `git diff`/`git status` — it walks outward from that specific change, using `Grep`/`Glob` to find every real reference across:

- **Frontend** — pages, components, services/hooks consuming the API or rendering the changed field
- **API contract** — do request/response types match on both sides?
- **Backend** — controllers, services, business rules, validation
- **Database** — models/ORM, migrations, constraints, relationships
- **Permissions & routes** — any access check still referencing the old shape?
- **Tests** — coverage that's now stale or missing
- **Docs** — READMEs, comments describing the old behavior

Every finding is classified 🔴 confirmed break / 🟡 inconsistency / 🟢 orphan, and every claim is backed by a file actually read — if it can't verify something, it says so instead of asserting it.

## Safety

NEXUS is scoped to be safe to run against a real, live codebase:

- Read-only in Analysis and Suggestion modes — only Auto-fix touches files, and only when explicitly requested.
- Never touches database schema, migrations, or secrets (`.env`, credentials) without asking first.
- Never deletes files.
- Never commits, branches, or pushes — git usage is read-only (`diff`/`log`/`status`).
- Stops for confirmation before editing more than 5 files in Auto-fix mode.

## Language note

The current agent instructions (`agents/nexus.md`) are written in Brazilian Portuguese — that's the language of the project it was built and tested in. The format is plain Markdown with YAML frontmatter, so translating it to another language (or adapting the report format, stop conditions, or scope for your own workflow) is a matter of editing that one file directly. Contributions adding an English (or any other language) version are welcome — see [CONTRIBUTING.md](CONTRIBUTING.md).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE)
