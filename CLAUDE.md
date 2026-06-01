# CLAUDE.md

This file documents the repository conventions, environment, and workflows for AI assistants (Claude Code and others) working in this codebase.

## Repository Overview

**Repository:** `payasbeth-design/Claude`
**Status:** New repository — currently being bootstrapped.

This repository is managed via [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web) and runs in a managed remote execution environment. Sessions are ephemeral: the repo is cloned fresh on container start, and anything worth keeping must be committed and pushed before the session ends.

---

## Execution Environment

- **Platform:** Linux (ephemeral container, remote)
- **Shell:** bash
- **Working directory:** `/home/user/Claude`
- **Git remote:** `payasbeth-design/Claude` on GitHub
- **Sessions start fresh** — no local state persists between sessions

Because the container is reclaimed after inactivity, always commit and push completed work. Do not rely on local files surviving across sessions.

---

## Git Workflow

### Branching

Feature branches are created automatically by the Claude Code harness and follow this pattern:

```
claude/<short-description>-<random-suffix>
```

Examples:
- `claude/claude-md-docs-7hL2F`
- `claude/add-feature-xK9pQ`

**Rules:**
- Always develop on the branch assigned at session start — never push to `main` without explicit user permission.
- Create the branch locally if it doesn't already exist: `git checkout -b <branch>`
- Never force-push, `reset --hard`, or bypass hooks (`--no-verify`) without explicit user instruction.

### Committing

- Stage specific files by name — avoid `git add -A` or `git add .` (risks including `.env` or secrets).
- Commit messages should be concise, describe the *why*, and include the session URL as a footer.
- Always pass commit messages via a heredoc to preserve formatting:

```bash
git commit -m "$(cat <<'EOF'
Short summary of what and why

https://claude.ai/code/session_<id>
EOF
)"
```

### Pushing

Always push with the `-u` flag:

```bash
git push -u origin <branch-name>
```

Retry on network failure with exponential backoff: 2s, 4s, 8s, 16s (max 4 retries).

### Pull Requests

Do **not** create a PR unless the user explicitly requests one.

---

## Available MCP Integrations

The following MCP servers are configured for this environment:

### GitHub MCP (`mcp__github__*`)

Full GitHub API access scoped to `payasbeth-design/Claude`. Key tools:

| Tool | Purpose |
|---|---|
| `mcp__github__get_file_contents` | Read files from the remote repo |
| `mcp__github__create_or_update_file` | Write/update files via API |
| `mcp__github__push_files` | Push multiple files in one commit |
| `mcp__github__list_branches` | List branches |
| `mcp__github__create_branch` | Create a new branch |
| `mcp__github__list_pull_requests` | List PRs |
| `mcp__github__create_pull_request` | Open a PR (only when asked) |
| `mcp__github__list_issues` | List issues |
| `mcp__github__add_issue_comment` | Comment on an issue |
| `mcp__github__subscribe_pr_activity` | Watch a PR for CI/review events |

> Tool schemas are deferred — call `ToolSearch` with `select:<name>` before first use.

### Notion MCP (`mcp__d9668f0d-e8fe-4d10-9bc0-58405a28fe69__notion-*`)

Notion integration is available. Key tools:

| Tool | Purpose |
|---|---|
| `notion-fetch` | Fetch a page or database |
| `notion-search` | Search across Notion workspace |
| `notion-create-pages` | Create new pages |
| `notion-update-page` | Update page content |
| `notion-create-database` | Create a database |
| `notion-get-users` | List workspace users |

> Always load tool schemas via `ToolSearch` before calling Notion tools.

---

## Code Conventions

Since this repository is new, conventions will be established as the project grows. The following defaults apply:

- **No comments** unless the *why* is non-obvious (hidden constraint, workaround, subtle invariant).
- **No docstrings or multi-line comment blocks** — one short line max.
- **No emojis** in code or commit messages unless the user explicitly requests them.
- **No backwards-compatibility shims** for removed code — delete cleanly.
- **No features beyond what the task requires** — no speculative abstractions.
- **Security:** Never commit `.env` files, credentials, or secrets. Never introduce SQL injection, XSS, command injection, or other OWASP Top 10 vulnerabilities.

---

## Working with AI Assistants

### Tools preference

1. Use dedicated tools (`Read`, `Edit`, `Write`) over `Bash` whenever possible.
2. Run independent tool calls in parallel.
3. Use `Agent` with `subagent_type=Explore` for broad codebase searches spanning more than 3 queries.

### Confirming before acting

Always check with the user before:
- Destructive operations (`rm`, `reset --hard`, `branch -D`, `push --force`)
- Actions visible to others (pushing, creating PRs, posting comments, sending messages)
- Modifying shared infrastructure or permissions

### Session hygiene

- Commit and push all meaningful work before the session ends.
- Do not leave the working tree in a dirty state.
- Do not create intermediate planning documents unless the user requests them.

---

## Skills Available

The following slash-command skills are configured:

| Skill | Trigger |
|---|---|
| `/deep-research` | Multi-source researched reports |
| `/code-review` | Review diff for bugs and cleanups |
| `/security-review` | Security review of pending changes |
| `/verify` | Run app and verify a change works |
| `/run` | Launch the project app |
| `/simplify` | Simplify/clean changed code |
| `/init` | (Re)initialize CLAUDE.md |
| `/review` | Review a pull request |
| `/update-config` | Update Claude Code settings |
| `/session-start-hook` | Set up a SessionStart hook |
| `/claude-api` | Build/debug Anthropic SDK apps |

---

## Updating This File

Run `/init` or ask Claude Code to update CLAUDE.md whenever:
- New dependencies, tools, or MCP integrations are added.
- The project structure changes significantly.
- New conventions are adopted.
- Build, test, or lint commands are established.
