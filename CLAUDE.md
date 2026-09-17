# Working on code-craft

This repository is the `code-craft` plugin itself — five instruction files that tell agents how to build software in someone else's repository. `README.md` documents the roles, the loop and what a project must declare; don't restate it here.

The working copy is also the installation: `~/.claude/skills/code-craft/` loads as `code-craft@skills-dir`, with no marketplace and no install step.

## After changing anything, to actually use it

Nothing syncs on its own. A merge on GitHub does not reach a session until the working copy is pulled:

```bash
git -C ~/.claude/skills/code-craft pull --ff-only
```

Then, in every session already running:

```
/reload-plugins
```

New sessions pick the change up on their own.

**Skills and agents differ, and the difference is easy to miss.** A change to `skills/*/SKILL.md` takes effect immediately in a running session. A change to anything in `agents/` does not: the session keeps serving the copy it loaded at startup, silently, with no sign that it is stale. So after touching an agent, reload before dispatching one — otherwise the old text runs and the report looks perfectly normal.

To see what is actually loaded rather than trusting it:

```bash
claude plugin details code-craft
```

And when it really matters, ask the agent to quote the opening sentence of its own instructions back. It is the only way to be certain which copy ran.

## Working here

`main` is protected: every change is a branch and a pull request, and only the owner merges. The process is deliberately light — no briefs, no gates, no agents. The owner reads the diff.

Before opening the pull request:

```bash
claude plugin validate .
```

Size is not what decides how carefully a change should be read. A reworded bullet in thrall's routing rule changes what an agent does as surely as a new agent would; a typo fixed in the README does not.

## The one rule that is easy to break

Nothing here may name a project's stack, commands, tracker or owner. The framework holds its judgments — the gate order, the barrier, the same-commit rule, the cap on fix pushes — and each project declares its facts in `.claude/code-craft.json`. If a new instruction needs to know something about a project, it reads the manifest; if the manifest cannot express it, that is a conversation with the owner, not a hard-coded default.

`version` in `.claude-plugin/plugin.json` gates updates only for marketplace installs. Loaded from a skills directory, the plugin is whatever is on disk, so a version bump is a release note rather than a mechanism.
