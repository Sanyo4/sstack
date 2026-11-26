# S Stack

How I build with coding agents. Eight skills and one page of always-on rules, in plain Markdown. They work in Claude Code and Antigravity, and use [T3 Code](https://github.com/pingdotgg/t3code) features when they are there.

The rules in one breath: restate the goal and the smallest change. Fix the class of bug, not the case. Prove it on the real thing. Let adversaries try to break it. Ship it as a PR. Route each job to the model that is best at it.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `s-mode` | "s-mode", "full send" | Takes any code change from the request to a merged PR: restate, pick a flow, build, prove, ship |
| `ship` | "ship it", "full stop", "babysit" | Files the PR, watches checks and bots, and merges when every merge check passes |
| `prove` | "prove it", "verify" | The proof ladder, from a reproduction to screenshots judged by a second model |
| `break-it` | "break it", "review this PR" | Adversarial review where a finding counts only when a run proves it |
| `general-fix` | (automatic on bugs) | Fix the class, not the case: find the invariant, every path, and the layer they share |
| `encode` | (automatic on corrections) | Turns a repeated correction into structure: architecture, then types, then a check |
| `design` | "design this", "mock it" | Show, don't tell: research real apps, mock the page, cut instruction text |
| `routing` | (automatic on delegation) | Roles, the model table, the role briefs, and how to dispatch in each runtime |

`rules/AGENTS.md` holds the always-on rules: working style, shipping, what needs your yes, parallel sessions, and your settings.

## Install

You need git and the GitHub CLI signed in (`gh auth login`). Clone the repo once:

```sh
git clone https://github.com/Sanyo4/sstack ~/.sstack
```

On Windows: `git clone https://github.com/Sanyo4/sstack "%USERPROFILE%\.sstack"` in cmd, or `"$HOME\.sstack"` in PowerShell. The `claude` commands below expand `~` themselves on every OS.

### Claude Code

```sh
claude plugin marketplace add ~/.sstack
claude plugin install sstack@sstack
```

Then add this line to `~/.claude/CLAUDE.md`, so the rules are always on:

```md
@~/.sstack/rules/AGENTS.md
```

### Antigravity

Add the clone's absolute path to `entries` in `~/.gemini/config/plugins.json`. Under T3 Code, add it to `~/.t3/userdata/providers/antigravity/<id>/config/plugins.json` too. Antigravity rejects `~`, so write the full path:

```json
{ "entries": [{ "path": "/Users/you/.sstack" }] }
```

On Windows, write the path with forward slashes, such as `C:/Users/you/.sstack`.

Antigravity loads the skills and `rules/AGENTS.md` from there.

### Any other agent

Copy `skills/*` into its skills folder and paste `rules/AGENTS.md` into its instruction file.

### Update

```sh
git -C ~/.sstack pull
```

On Windows: `git -C "%USERPROFILE%\.sstack" pull` in cmd, or `git -C "$HOME\.sstack" pull` in PowerShell. Both runtimes read the skills straight from the clone, so the pull is the whole update.

## Make it yours

Fork it and edit. The `Settings` section at the top of `rules/AGENTS.md` holds the choices that are personal:

- `Merge: gate` lets the agent merge its own PR when every check passes. Set `ready` if you want to press merge yourself.
- `Deploy: auto` lets the agent deploy a merged PR and check it is live. Set `ask` to approve each deploy.
- `Runtime`, `Vendors` and `Mobbin` say where work runs and which vendors may see your code.
- The model table in `skills/routing/SKILL.md` says which model runs each role.

A line in your global instruction file (`~/.claude/CLAUDE.md`, or Antigravity's global rules) beats the same line here, so you can also keep the repo as it is and override there. A repo's own instruction file can only narrow a setting, never widen one.

## Guardrails

The skills tell the agent never to push to the default branch, skip hooks or force a merge. Two things back that up outside the prompt.

On GitHub, protect the default branch: require a pull request and passing checks. A rule there holds whatever an agent tries. Protected branches on a private repo need a paid GitHub plan.

In Claude Code, add deny rules to `~/.claude/settings.json`:

```json
{
  "permissions": {
    "deny": [
      "Bash(git push * main)",
      "Bash(git push * main *)",
      "Bash(git push *:main*)",
      "Bash(git push * master)",
      "Bash(git push * master *)",
      "Bash(git push *:master*)",
      "Bash(git * --no-verify*)",
      "Bash(gh pr merge *--admin*)",
      "Bash(gh pr merge *--auto*)",
      "Bash(gh api *pulls/*/merge*)"
    ]
  }
}
```

These catch the common forms of a push to the default branch, a hook skip, an admin or auto merge, and a REST merge. They miss others, such as a bare `git push` with the default branch checked out, `git -C <dir> push`, `git commit -n` and a GraphQL merge. Branch protection is the real guard. With `Merge: ready`, add `"Bash(gh pr merge*)"` as well.

## Credits

S Stack builds on [tstack](https://github.com/t3dotgg/tstack) by Theo Browne, pstack by Lauren Tan, and pstack-claude by Michael Denyer, each under the MIT licence. See [NOTICE.md](NOTICE.md).

## License

[MIT](LICENSE)
