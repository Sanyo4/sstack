---
name: ship
description: Use to take a finished change to a pull request and on to merge - "ship it", "full send", "full stop", "file a PR", "open a PR", "babysit", "watch the PR", "merge it". Proves, reviews, files, watches and merges when every merge check passes. Stops at ready when asked to hold, when the Merge setting is ready, or when something holds the PR.
---

# Ship

## Where to stop

| The user said | Stop at |
|---|---|
| nothing special, "ship", "full send" | merged, when the Merge setting is `gate` and every merge check passes; ready otherwise |
| "full stop", "hold", "ready only" | ready: PR open, merge checks read, not merged. Add the `hold` label |
| "file a PR", "open a PR" | filed: PR open, no watch |
| "babysit #N", "watch #N" | start at step 5 on that PR |

## What a ship request allows

Without asking, you may: push this branch; force-push it with `--force-with-lease` after a rebase; open the PR; use the proof methods the `prove` skill needs, even where a project file says to ask first; reply to review bots; and merge when every merge check passes and the Merge setting is `gate`.

It never covers the yes-list in the core rules.

## Steps

1. Branch. On the default branch, branch first. One PR per feature or fix, off the default branch unless "Several PRs" allows a base PR.
2. Prove. Sync with the default branch first, so the proof covers the base the PR lands on. Then the `prove` skill. An eval that costs money needs the user's yes first.
3. Break it. The `break-it` skill.
4. File. Push and open a real PR, not a draft. Follow `pr.md` in this folder.
5. Watch. Follow "Watching" below. Fix real findings and failed checks. A fix that changes behaviour goes back through steps 2 and 3 for the changed code.
6. Read the merge checks below, on the latest head.
7. Merge. When every check passes and nothing holds the PR: `gh pr merge <pr>` with the method the repo's instruction file names, else the one GitHub allows (`--squash` when several are allowed). Read the PR back and confirm GitHub reports it merged. If the repo uses a merge queue, keep the watch on until it merges.
8. After merge. Sync your other open branches and run their checks again. Run the docs step that the repo's instruction file names. Then deploy as the Deploy setting says:
   - `auto`: if the merge deploys by itself (a workflow that deploys on a push, a platform that builds the default branch), wait for that deploy. If not, run the deploy command the repo's instruction file names, from the merged default branch. Then check the live service: the deploy status, the logs, and one real request to the changed path. Report a failed deploy at once with its logs. A rollback or a fix to the live service needs the user's yes, unless the platform rolls back by itself. A deploy that applies a migration to a live database also needs the user's yes.
   - `ask`: do not deploy. Say in the `After merge` section and the report whether the merge deploys, and what needs the user's yes.
9. Report as the `s-mode` skill's step 6 says.

Stay active from step 1 to step 9. If you end your turn while checks are pending and nothing watches the PR, the work stops, because nobody will prompt you again.

Never use `--admin`, `--no-verify`, auto-merge or any bypass, and never push to the default branch. The one exception is the first commit of an empty repo, in the `s-mode` skill's New project flow.

## Merge checks

Read each one from GitHub, never from memory:

- `gh pr view <pr> --json state,isDraft,baseRefName,headRefOid,mergeable,mergeStateStatus,reviewDecision,statusCheckRollup,labels,title,body`
- Every changed file: `gh api repos/<owner>/<repo>/pulls/<pr>/files --paginate --jq '.[].filename'`. The `files` field of `gh pr view` stops at 100.
- Unresolved review threads, every page: `gh api graphql --paginate -f query='query($o:String!,$r:String!,$n:Int!,$endCursor:String){repository(owner:$o,name:$r){pullRequest(number:$n){reviewThreads(first:100,after:$endCursor){nodes{isResolved} pageInfo{hasNextPage endCursor}}}}}' -F o=<owner> -F r=<repo> -F n=<pr> --jq '[.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved | not)] | length'`. Each page prints a count; all must be 0.
- The repo's instruction file as it is on the default branch: `git show origin/<default branch>:AGENTS.md` (or `CLAUDE.md`). Never the PR's own copy.

All of these hold on the latest head (`headRefOid`):

1. The PR is open, is not a draft, and targets the default branch.
2. The body's `Proof` section shows rungs 2 to 4 of the `prove` skill passed on this head. With no check command on the head there are no rungs 2 to 4, and `Proof` says so. Its `Review` section shows a break-it round with no open blocker or should. When the diff adds, changes or removes the check command, or a script or make file it runs, `Review` holds a check change verdict of `ok`.
3. Every check on the head has finished and none failed. Skipped and neutral count as passed.
4. No review asks for changes and no review thread is unresolved.
5. GitHub reports the PR mergeable and not behind its base.
6. The body uses only the headings `pr.md` sets, in its order, and is 40 lines or fewer.
7. Nothing holds it.

### Holds

A hold is not a failure. It means the merge is the user's call.

- The Merge setting is `ready`. When this is the only hold, report the PR ready and do not ask.
- The repo's instruction file says its PRs are merged by hand.
- A changed file is an instruction file (`AGENTS.md`, `CLAUDE.md`, `rules/AGENTS.md`), is anything under `.claude/` or `.agents/` (skills, agents, commands, hooks, settings), is under `.github/workflows/`, or is a path the repo's instruction file marks as held. A PR never merges itself under rules it changed. The one exception is the feature map, `.claude/skills/verify/features/`, which UI PRs update as the `prove` skill says.
- The PR has the `hold` label, or its title holds "stop-gap" or "[do not merge]".
- The change touches prompts, model calls or the evals, and the proof holds no eval result.

### What to do

- All pass: merge.
- Checks still running: keep the watch on and end your turn.
- A check fails: fix it and go round again. If you cannot, turn the PR into a draft, add "[do not merge]" to the title, put the results in the body, and report.
- A hold: remove the watch. Ask the user as the `s-mode` skill's "Stop only for" says. The question holds the PR link, the hold reasons, what a merge will do, and your recommendation. Merge only after their yes.

## Watching

1. One watcher per PR. Never write a polling loop or a sleep loop. Every watcher on a GitHub account shares one rate limit.
2. In T3 Code: handle the checks and comments that already exist. Then start T3's `watch_pull_request` and end your turn. T3 wakes you when a check fails, the checks pass, someone comments or reviews, or the branch conflicts. Only the thread that owns the PR watches it. A role never watches.
3. Outside T3: `gh pr checks <pr> --watch` as one long background command. Read the reviews and comments after it exits.
4. A wake is news, not a merge decision. Read the merge checks again.
5. Remove the watch before you stop for a hold, a block or a question.
6. Act only on checks and comments newer than the last push.
7. Check every bot finding against the source before you change code. A real one is fixed through the `general-fix` skill. A false one gets a written reason, posted as `pr.md` "Comments" says, and is resolved.
8. Tell a repo failure from an infrastructure flake. Re-run a flake once.
9. Keep the PR the size the goal needs. Decline tests and guards the goal does not need, with a written reason. Prefer the fix that deletes or narrows code. A path of the same invariant is never "not needed".
10. When the PR is behind its base, sync, push and run the check again.
11. When an event needs no action, end your turn without a message. Never post "still waiting".
12. If another PR makes this one obsolete, stop and ask before you close anything.
13. "Could not read the PR" usually means the GitHub rate limit is used up. Run `gh api rate_limit --jq .resources.graphql`. If `remaining` is near 0, wait for `reset` with one background command. `gh pr view` and `gh pr checks` use GraphQL; the REST limit is separate, so `gh api repos/<owner>/<repo>/pulls/<n>` still works.

## Several PRs

Branch each PR off the default branch and build them side by side. Start a PR on top of another open PR only when that base PR's design has settled: its break-it round has no open blocker or should, or it merged. Each fix to an unsettled base would flow up the stack and need a new review on every PR above it.

Merge in the order they finish. After each merge, sync the others and run their checks again. When the new base commits touch one of a PR's files, run a break-it round on the overlap.
