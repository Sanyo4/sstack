# Filing the PR

Rules for the title, the body and the comments of a PR that the `ship` skill files.

## Before you open it

1. Look for an existing PR for the branch before you open one. If one exists, update it.
2. Read the diff against the default branch. Its contents must match the goal and nothing else.

## Title

Follow the repo's convention. Read it from recent merged PRs and the git log. The title usually becomes the merge commit message.

Say why the change matters, not which function moved.

- Weak: `perf(server): negotiate permessage-deflate on the websocket`
- Better: `perf(server): cut websocket frame size by 70% with compression`

A stop-gap has "stop-gap" in the title. That holds it for the user.

## Body

40 lines at most. Use these `##` headings in this order. Drop a heading that has nothing under it.

| Heading | What goes under it |
|---|---|
| `Why` | The problem in the user's words, then the fix in a sentence or two. No list of files. |
| `What changed` | The change in a few lines. |
| `Invariant` | Fixes only. The invariant, how many paths it covers, the path table in short. What was deleted. |
| `Proof` | One line per rung climbed, with the head commit it ran on. For a fix, the failing output and the passing output, a few lines each. |
| `Review` | The break-it seats and rounds, and each blocker and should with its outcome: fixed, declined with the user's answer, or dropped. |
| `Risk` | What could break and for whom. The switch name and its default, if there is one. Each problem the change could open, and how it is closed. |
| `After merge` | Whether the merge deploys, as far as the repo shows. Each step that needs the user's yes, such as a manual deploy, a migration or turning on a switch. |

Start `Why` from the problem, not from an inventory of the change.

- Weak: "Removed implicit carry-over from every new-thread entry point. Deleted three helpers."
- Better: "My default was ignored when I started a new thread. Now the default always applies."

Keep logs and long tables out of the body. Give a short summary inside the 40-line limit. Attach a screenshot only when it holds no real customer data.

The last line is `Made with <model> in <harness>.` Use the names the harness reports. Include the models of the roles that wrote part of the change. Name no model you cannot verify. If the harness reports no model name, name the harness only.

## Opening it

1. Open a real PR, so checks and review bots run. A draft with "[do not merge]" in the title is only for a failed proof.
2. Read the title, the body and every attachment for secrets and private details before you submit.
3. If `gh pr create` fails because the GraphQL rate limit is used up, open the PR through the REST API: `gh api repos/<owner>/<repo>/pulls -f title=<title> -f head=<branch> -f base=<default branch> -F body=@<file>`.

## Comments

Post a comment only when it serves the request. Every comment posted for the user follows these rules. That covers top-level comments, inline review comments, replies to bots and reviewers, and the reason you write before you resolve or dismiss a finding. It does not cover the PR body, commit messages, or text the user asked you to post word for word.

1. The comment starts with a note line that names the model and says it answers on the user's behalf:

   ```md
   > [!NOTE]
   > <model> responding on behalf of <user>

   The reply starts here.
   ```

   Use the model name the harness reports, or the harness name if it reports none. Use the user's name or GitHub login. Add no second attribution and no signature.
2. Lead with the conclusion. Cite the code, the check, the run or the reproduction. Keep it short. Say which parts are verified facts and which are judgement.
3. Read the whole comment, quotes and attachments included, for secrets and private details before you post.
4. Confirm that GitHub returned the comment URL. Resolve a review thread only after the reply posted.
