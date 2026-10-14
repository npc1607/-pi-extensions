# pi-extensions

Personal extensions for [pi](https://github.com/earendil-works/pi) coding agent.

## Extensions

### confirm-edit-only.ts

An approval guard for pi: every `edit` / `write` file change and every non-allow-listed `bash` command goes through a **styled confirmation dialog** before running. Read-only and local dev commands pass silently.

```
╭──────────────────────────────────────────────────────────────────────────────╮
│ Run command?                                                          [BASH] │
│──────────────────────────────────────────────────────────────────────────────│
│ Command (redirection/substitution):                                          │
│  1│ cd /path/to/your/project                                                 │
│  2│ go test ./internal/biz/... -run                                          │
│    │ "TestExample" -count=1 -timeout 300s 2>&1 | tee test.log                │
│  3│ rm -rf /tmp/test-data                                                    │
│──────────────────────────────────────────────────────────────────────────────│
│ y / Enter = approve   n / Esc = reject                                       │
╰──────────────────────────────────────────────────────────────────────────────╯
```

#### Features

- **Styled overlay dialog** — rounded border, centered overlay, semantic theme colors (danger in red, warnings in yellow), rendered with pi's TUI component system. Falls back to the built-in plain `confirm()` outside interactive TUI mode (e.g. RPC clients).
- **Formatted commands** — command chains (`&&`, `||`, `;`) are split into numbered segments; long commands wrap with aligned continuation indents and overflow folding.
- **Keyboard shortcuts** — `y` / `Enter` to approve (green), `n` / `Esc` / `q` / `Ctrl+C` to reject (red).
- **Quote-aware parsing** — shell syntax inside quotes is treated as literal: `rg "A|B"` (regex alternation), `grep ">&"` (search pattern), and `awk '{print $1}'` never trigger false positives, while real redirections (`>`), command substitution (`$(...)`, even inside double quotes), and pipelines between unknown commands are still caught.
- **Three-tier command classification** (default-deny):

  | Tier | Behavior | Examples |
  |---|---|---|
  | 🟢 Allow-listed | Runs silently | `ls`, `rg`, `cat`, `go build/test/run`, `make`, `kill`, `mkdir/cp/mv`, `git status/diff/add/commit/pull/rebase`, `glab mr list/view`, `curl` to `localhost` or GET-only |
  | 🔴 Danger-listed | Confirmed with a specific dialog | `rm` (any form), `git push` (incl. force), `git reset/clean/restore/checkout -- <path>/branch -D/stash drop`, `sudo`, `ssh/scp/rsync`, `curl -X POST/PUT/PATCH/DELETE` to non-local hosts, `glab` write actions |
  | ⚡ Unknown | Confirmed generically | anything not in either list, shell redirections (`>`), command substitution `$(...)` |

  Also allow-listed as safe segments: `cd` / `pushd` / `popd` (navigation), `tee`, `obsidian`, and harmless stderr redirects (`2>&1`, `2>/dev/null`).

#### Install

Copy into your pi extensions directory:

```bash
mkdir -p ~/.pi/agent/extensions
cp confirm-edit-only.ts ~/.pi/agent/extensions/
```

Then start pi (or run `/reload` in a running session).

#### Tune the rules

All behavior lives in the classification tables at the top of the file:

- `PASS_EXECUTABLES` — executables that always run silently
- `GIT_PASS` — safe `git` sub-commands
- `GLAB_RISKY` — `glab` actions that need approval
- `classifySegment()` / `classifyGit()` / `classifyCurl()` — per-command logic

The dialog width adapts to the terminal (up to 88%, min 72 columns, max 100).

## License

[MIT](LICENSE)
