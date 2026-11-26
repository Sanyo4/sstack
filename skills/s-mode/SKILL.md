---
name: s-mode
description: Use for any task that changes code - a feature, a bug fix, a refactor, a migration, a UI change. Also for "s-mode", "sstack", "full send" and "full stop". Restates the goal, sizes the smallest change, picks a flow, proves the result and ships it as a pull request.
---

# S mode

You own the task from the request to a merged PR, or to a PR held for the user. Assume the user is not watching. Your updates go unread until you stop.

This works in any repo, including a new empty one. Read the language, framework, folder layout, package manager and commands from the repo. Never assume them.

## 0. Pick the runtime

Pick the lead's runtime once, before any other work. The allowed runtimes are the ones whose vendor the Vendors setting in the core rules allows, less any the repo's instruction file removes. The request wins when it names one. Otherwise use the Runtime setting. A repo's instruction file only takes runtimes out of the allowed set; it never picks one. A named runtime that is not allowed does not count: tell the user, then ask. If the answer is `ask`, ask one question with the question tool, offering the allowed runtimes, Claude Code first and marked recommended. With one allowed runtime, use it.

Use the answer for the whole task. Do not ask again inside the same task. Name the runtime in the report.

If the chosen runtime is not the one this session runs on, hand the task to a new session on that runtime, as the `routing` skill says. Its first message carries the request and the chosen runtime. Then stop. If this session cannot start one, say so and stop.

## 1. Restate

Write two lines before any tool call that changes a file:

- Goal: what the user wants, in their terms.
- Smallest change: the least code that meets the goal. Name what you will delete.

If the goal is unclear in a way that changes the result, ask one question. Otherwise state your assumption and go.

## 2. Pick a flow

| The task is | Flow |
|---|---|
| A bug or a reported problem | Fix |
| New behaviour | Feature |
| Same behaviour, better code | Refactor |
| A screen, a component or UI copy | Feature or Fix, with Design first |
| A question with no change asked for | Investigate, then stop and report |
| A correction the user has made before | the flow that fits, then Encode |
| A repo with code but no check command, or no code yet | New project first, then the flow that fits. A repo of only docs or prompts needs no check command |

### Fix

1. Reproduce it on the real surface with the `prove` skill. Keep the failing output.
2. Load the `general-fix` skill. State the invariant, list every path, find the layer.
3. If the fix changes a rule that the whole system or every agent follows, dispatch `architect` for the design first.
4. Build it. The failing reproduction lands as a test before the fix.
5. Prove that the same reproduction now passes.

### Feature

1. Read the code the feature touches. Dispatch `scout` to map an unfamiliar area.
2. Name the data shape first. Prefer a type, a table or a state machine to branches.
3. Put it behind a switch that is off by default when it is risky or incomplete.
4. Build in units that each end in a check. One owner for a coupled change. Fan out only over files that do not overlap.
5. Prove it on the real surface.

### Refactor

1. Prove the current behaviour first, so you can show it did not change.
2. Delete dead code before you move anything.
3. Migrate every caller and delete the old path in the same PR.
4. Prove the same behaviour after.

### Design

Load the `design` skill before any UI code. The mock comes before the build.

### New project

1. No remote: create the repo or ask the user for its URL. Keep it private unless the user said otherwise.
2. No commits yet: a PR needs a base. Make one first commit on the default branch yourself that holds only a README and the ignore file, and push it. This is the one exception to "never push to the default branch", because there is nothing yet to branch from.
3. If the user's setup has a skill for new apps or a default stack, load it for the stack choices. Otherwise pick the stack the request implies, and state it as an assumption.
4. A repo with code and no check command: the first PR adds one, the way the stack's language names it (a `check` script, a `check` make target, or the ecosystem's tests). It runs the typecheck, the lint and the tests, and it is the command CI runs. No feature work goes in this PR.
5. Then go on with the flow that fits the request.

### Investigate

Find the answer with evidence: file and line, a run, a log. Report it. Change nothing unless asked.

### Encode

When the user corrects you, fix the mistake. Then load the `encode` skill and make the mistake impossible, or caught by a check.

## 3. Build

- Route by the `routing` skill. You plan, brief, review and verify. Roles build and check.
- Read every diff a role returns. Write your own summary.
- A fork you can settle by running something: run it, do not ask.
- Do not stop for a choice you can undo. Make it, record it, go on.

## 4. Prove

Load the `prove` skill. No proof, no ready PR.

## 5. Ship

Load the `ship` skill. With the Merge setting at `gate`, merge when every merge check passes. "Full stop" or "hold" means stop at ready.

## 6. Report

One message, goal first:

- The PR link and its state: merged, held (with the reason) or blocked.
- The runtime, and what chose it.
- What changed, in two or three lines.
- Proof: what you ran and saw, with links.
- Decisions you made for the user.
- Whether the merge deploys, as far as you can tell from the repo (its workflows, its platform config, its instruction file), and what needs their yes next, such as a manual deploy or a migration.
- What went to the backlog.

## Stop only for

- An action on the yes-list in the core rules.
- A proof you cannot get.
- A hold, or a merge check you cannot make pass.
- A goal that is ambiguous in a way that changes the result.

Ask with the question tool, so the thread shows that it waits. Put the PR link, the blocker and your recommendation in the question. If no question tool exists or no answer comes, take the safe path: leave the PR open and start your final message with the question.
