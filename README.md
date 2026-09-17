# code-craft

Brief-first agent development, as a Claude Code plugin. The owner decides what gets built and what gets merged; the agents do the rest, inside written rules.

| Role                      | Kind  | Job                                                                       |
| ------------------------- | ----- | ------------------------------------------------------------------------- |
| `/code-craft:medivh`      | Skill | Interviews the owner and writes a brief. Stops there.                     |
| `/code-craft:thrall`      | Skill | Runs the loop from an approved brief to a pull request ready to merge.    |
| `peon`                    | Agent | Builds exactly what the brief scopes and opens the pull request.          |
| `uther`                   | Agent | Reviews the code against the brief and the project's rules.               |
| `khadgar`                 | Agent | Tests the brief's "Done when" steps in the running app, as a user would.  |

## The loop

1. The owner describes an idea. **Medivh** interviews them one question at a time and writes a brief. The owner approves it.
2. **Thrall** takes over: it dispatches **peon**, which builds the brief and opens a pull request.
3. Thrall verifies peon's proof rather than relaying it, then runs the gates on the pushed commit — **uther** first, then **khadgar**, never at once.
4. Findings reach peon in one batch, and only after every gate has reported. Approval attaches to a commit: any push invalidates every gate that ran before it.
5. The cap is peon's build plus two fix pushes. Past that, thrall stops and brings it to the owner.
6. The owner merges. Nobody else ever does.

The framework's opinions — the cap, the ordering, the barrier, the same-commit rule — are not configurable. A project declares its facts; the framework keeps its judgments.

## What a project must declare

Create `.claude/code-craft.json` in the project:

```json
{
  "owner": "Reza",
  "gate": "pnpm check",
  "briefs": "docs/briefs",
  "app": {
    "up": "pnpm qa:up",
    "down": "pnpm qa:down",
    "restart": "pnpm qa:restart-api"
  }
}
```

| Field    | Meaning                                                                                                    |
| -------- | ---------------------------------------------------------------------------------------------------------- |
| `owner`  | The person whose decisions the agents escalate to, and the only one who merges.                            |
| `gate`   | The one command that must pass before anything is considered done.                                         |
| `briefs` | Where briefs live, relative to the repository root.                                                        |
| `app`    | How to start, stop and restart the app for testing. **Optional** — omit it when the project has no app.    |

`app.up` must give khadgar a disposable instance with its own data — never the owner's real database. `app.restart` must restart the app while keeping that data, so a brief can ask whether something survived a restart.

**A project that declares no `app` has no app to test.** Khadgar does not run there, and uther is the only gate. That is how this plugin's own repository works.

Everything else stays in the project's own `CLAUDE.md`: its stack, its layers, its decision records, where open work is tracked, and the conventions peon follows when opening a pull request.

## Installing

This plugin lives in a skills directory, so it loads on its own:

- **Personal, every project:** keep it at `~/.claude/skills/code-craft/`. It loads as `code-craft@skills-dir` with no marketplace and no install step.
- **Testing a change:** `claude --plugin-dir ~/.claude/skills/code-craft`, which takes precedence for that session.
- **Sharing later:** add a `.claude-plugin/marketplace.json` and it becomes installable with `/plugin install`. Nothing else changes.

Skill text takes effect immediately. Changes to agents need `/reload-plugins` or a new session.

## Working on this repository

Light process, deliberately: branch, pull request, `claude plugin validate .`, the owner merges. No briefs, no gates, no agents — the whole repository is prose, and the owner reads the diff.

The distinction that matters when deciding how carefully to review a change is not its size: a reworded bullet in thrall's routing rule changes what an agent does as surely as a new agent would, while a typo in this README does not.
