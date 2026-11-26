---
name: encode
description: Use when the user corrects you, when a review or break-it finds the same mistake a second time, or when you catch yourself writing the same instruction twice. Turns a repeated correction into structure, so the next agent cannot make the mistake. Tries architecture first, then types, then a lint or check whose error names the fix, then a test, and writes prose last. Proves each check fails on a real past mistake and keeps the repo's rule table.
---

# Encode

A correction that lives only in prose is made again. Change the repo so the next agent cannot make the mistake, or so a check catches it.

This works in any repo. Use the repo's own language, type system, linter, test runner and CI. If it has none of these, the first lint or test you add sets one up, and its command joins the repo's `check` command.

## When to run it

- The user corrects you. Fix the mistake first, then encode it.
- A review or a `break-it` round finds a mistake a second time.
- You write the same instruction a second time, in a brief, a comment or a doc.

## Who you are designing for

Assume every contributor is an agent that sees only the files it opened, copies the nearest example, and takes the shortest path that compiles. Design the repo so a change that looks right from one file is right for the whole repo.

## 1. Find the class

Read recent commits, reverts, review comments, the repo's instruction file, and comments that explain a workaround. Group the mistakes into classes. A class counts once it has happened twice.

A new action that needs the user's yes is a class too. Add it to the yes-list in the repo's instruction file, and to a permission rule that asks or denies where the runtime has one (Claude Code's `permissions.ask` or `permissions.deny` in `.claude/settings.json`). When it holds for every repo, add it to the yes-list in your S Stack fork's `rules/AGENTS.md`.

## 2. Fix it at the highest level that works

Take the first level that makes the mistake impossible. Go down a level only when the one above cannot hold the rule.

1. Architecture. Give each piece of state one owner and each task one supported way. Hide internals, so the wrong import fails. Replace a hand-synced list with one source. Delete the old way and any dead code an agent would copy.
2. Types, where the language has them. Make the bad state impossible to write: a closed set of values in place of a free string, a distinct type in place of a raw value, a required field in place of an optional one. A schema does the same job for data.
3. A lint or a check. When bad code still compiles, add a lint rule or a check script that CI runs. Its error message names the file, type or function to use instead. If the pattern is already common, fail only when a change adds a new case.
4. A test of the behaviour. Fix or delete any test that still passes when the behaviour it checks breaks, or that fails when only the internals change.
5. A prose rule. Only for a judgement call that none of the above can hold. Nothing fails when an agent skips it, so make it prominent and add one example of the mistake.

If the fix is structural, use only the structural fix. The instruction was the symptom. Do not keep the prose rule beside the check.

## 3. Prove each check

1. Prove each new check fails on a real past mistake. Check out or reproduce the commit that made the mistake, run the check, and keep the failing output.
2. Test the lint itself: one case per rule, and all rules together in one file.
3. Run the same command locally and in CI. A check that only runs on one of them is not a check.
4. Fix the most frequent classes now, one commit each.

## 4. Exceptions

An exception sits on the offending line. It carries a reason, an expiry date and the user's approval. An exception without all three is a failing check.

## 5. Keep the rule table

The repo's instruction file holds one table. When it has none yet, add the table with the first row you encode, under a `## Rules` heading:

| Rule | Enforced by |
|---|---|
| One sentence | The file, type, lint rule, check or test. A prose-only rule says "judgement" |

- Add a row for each class you encode.
- When a rule is already in the table with nothing enforcing it and the mistake happens again, that is a repeat. Raise its level in the same change.
- Drop a row once its mistake cannot happen.

## Reply

For each class: the evidence (commits, comments or corrections, with links), the level you chose, the check and its failing output on the past mistake, and why a higher level did not work.
