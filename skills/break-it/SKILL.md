---
name: break-it
description: Use to review a change before it ships, and for "break it", "interrogate", "adversarial review", "stress test this", "review this PR". Adversaries try to break the change. A finding counts only when a run proves it. Runs on a default or a risk panel of reviewer seats.
---

# Break it

Adversaries try to break a change. A finding survives only when a run proves it. Opinions about style, naming and structure are left out.

Every PR passes this before merge. The size changes. Whether it runs does not.

## The review bar

Block only for a failing run at this head or a bug a user would hit. A problem the base already has gets one line. Everything else goes to the backlog, not into another round.

## 1. Frame

- Scope: the branch diff from its merge base with the default branch, through the working tree. Or what the user names.
- Intent: restate it in two sentences, from the user's request, the commits and the PR body. For a fix, add the invariant in one sentence.
- Checks: the proof you already have. Do not send out code that fails its own checks.
- The check command: whether the diff adds, changes or removes it, or a script or make file it runs. Find the command on this branch and on the default branch, as the `prove` skill's "Where the check command comes from" says.

## 2. Seats

| Panel | Seats | Use it when |
|---|---|---|
| default | one `reviewer` | the diff is not risky |
| risk | one `reviewer`, plus one seat on Opus 5.5 at high | the diff touches sign-in, money, permissions, data loss or concurrency, changes a rule that every agent or step follows, or you judge it risky |

The reviewer's model comes from the `routing` table. A seat on a vendor the session cannot reach falls back as the `routing` skill's dispatch rule says. Each seat works in its own checkout. Dispatch all seats in one message.

If every seat is the same model family, say so in the output: the review had no model diversity.

## 3. The brief

Give every seat the scope, the intent, the checks, the review bar above, and this text:

> Break this change. Read anything and run anything in your own checkout. Do not edit the author's branch, push, or comment on the PR.
>
> Prove each finding with one targeted run against the real code: one test file, one reproduction script or one command that shows the wrong behaviour. Never run the full test suite or the repo's check command to prove a finding. Report each finding with a title, the exact command, the output it printed, and the output the intent requires.
>
> If the change is a fix, the invariant is: (the invariant). Look for another writer, caller, step or agent that can still break it, and prove it with a run. A proven failing run on another path of the same invariant is in scope.
>
> Leave out anything you did not run. That covers style, naming, structure, and risks that need a future change to matter. No findings is a valid result.
>
> Severity: blocker = a failing run at this head (the intended case is wrong, or the invariant still breaks on some path). should = a bug a user would hit, inside the intent. note = the rest, including a problem the base already has (one line).

When the diff adds, changes or removes the check command, or a script or make file it runs, add this to every seat's brief:

> This change touches the repo's check command: (the command on the default branch, or none) to (the command on the head, or none). Judge whether it weakens proof. Run both and compare what they run. It weakens proof when fewer tests run, a stage is dropped (the typecheck, the lint, a test suite), or a command cannot fail, such as `true`, `exit 0`, `|| true` or a test runner told to pass with no tests. Report "weakens" or "holds", with the commands you ran and what each one ran.

## 4. Verify

Rerun each finding's targeted command yourself, never the full suite. Keep a finding as a blocker or should only when it meets the review bar above. Drop unproven claims and count them. Merge findings that share a root cause. Set the severity yourself. A seat's label is a claim.

## 5. Answer

- blocker and should: fix through the `general-fix` skill. Prefer the fix that deletes or narrows code. A path of the same invariant is part of this PR.
- note (including a problem the base already has, or anything outside the intent and the invariant): send to the backlog, not into another round.
- You disagree: say why with evidence, such as a file and line, a test or output. Do not agree just to end the loop. A blocker or a should you decline needs the user's answer. Record it in the PR's `Review` section. Without it, the PR does not merge.
- A check change: set the verdict yourself from the seats' runs. `ok` when the new check proves at least what the old one did, or the repo had none. `blocker` when it weakens proof: fix the check in this PR, then run a round on the fix. A check change is never held for the user; this verdict is the review.

## 6. Rounds

- Round 1 is full. A later round gives a fresh seat the round 1 brief, the findings so far, and only the code that changed since.
- Budget: 2 rounds, or 3 when the `risk` panel ran.
- Stop when a round returns no blocker and no should.
- An item still disputed after two replies, or a budget that ends with one open, goes to the user with both sides in a few lines each. A budget that ends is not approval.
- If the second round of a fix finds another path of the same invariant, follow "Close the loop" step 4 in the `general-fix` skill.

## 7. Record

Write the result in the PR body's `Review` section, as `ship/pr.md` says: the seats and the rounds, each blocker and should with its outcome (`fixed`, `declined` with the user's answer, or `dropped`), and the head commit the last round ran on. When the head changes the check, add the check change verdict (`ok` or `blocker`) and its reason. A later push that changes behaviour needs a round on the changed code.

## Output

- The intent, and the invariant for a fix.
- Findings: for each verified one, the command, its output, how it breaks the intent, and its outcome.
- Dropped: a count per seat, and why.
- Seats and rounds, and whether the panel had more than one model family.
- The check change verdict and its reason, when the diff changes the check.
