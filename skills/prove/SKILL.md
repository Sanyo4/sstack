---
name: prove
description: Use before any change is called done, before a PR is filed, and whenever asked to verify, test, reproduce, screenshot or check that something works. Covers the proof ladder, the repo's verify skill and feature map, driving the real UI, judging screenshots with vision, and recording the proof in the PR.
---

# Prove

Proof is something you watched happen in this session: command output, a reproduction, a screenshot you looked at. A green mocked test does not prove that the screen works. "It compiles" proves nothing.

CI is slow. Local proof comes first, and CI confirms it.

## The ladder

Climb as far as the change reaches. Every rung can fail.

| Rung | What it proves | How |
|---|---|---|
| 1. Reproduce | the problem is real (fixes) | run the failing case before you change anything, and keep the output |
| 2. Static | types and encoded rules hold | the repo's typecheck and lint |
| 3. Tests | the behaviour holds | the related tests, run freely while you work, and one new test that fails without the change |
| 4. Check | nothing else broke | the repo's check command, the one CI runs, as a background command |
| 5. Run it | the real thing works | drive the real surface with the repo's verify skill: UI, CLI or API |
| 6. See it | the screen is right | screenshots from rung 5, judged with vision against the goal |
| 7. Evals | model behaviour holds | the repo's eval command |

Which rungs a change needs:

- Rungs 2 to 4, always, when the repo has a check command. A repo with code and no check command gets one first, through the New project flow in the `s-mode` skill. A repo with no code to check (only docs or prompts) has no rungs 2 to 4: say so in `Proof`, and climb rung 5 by running what the change describes.
- Rungs 5 and 6, when the diff touches anything a user sees.
- Rung 5, for any change to a CLI or an API.
- Rung 7, when the diff touches prompts, model calls or the evals themselves. An eval that costs money needs the user's yes first.
- Rung 1, for every fix.

How to drive, what to look at and what counts as right are your call.

### Where the check command comes from

The first of these that gives a command wins:

1. The check command the repo's instruction file names.
2. Something named check: a `check` script in `package.json`, run with the repo's package manager; a `check` target in the `Makefile`; a `check` recipe in the `justfile`.
3. The ecosystem's tests: the `typecheck`, `lint` and `test` scripts in `package.json` that exist, in that order (npm's placeholder `test` does not count); a `test` target in the `Makefile`; `Cargo.toml`; `go.mod`; a pytest config; `deno.json`; `mix.exs`; `gradlew`; `pom.xml`; `Package.swift`; a `.sln` file.

A monorepo or a mixed repo adds a root `check` script or a `check` make target in a normal PR.

## Rung 5: run it

1. Find the repo's verify skill at `.claude/skills/verify/SKILL.md`. If there is none, write one first, as `verify-skill.md` in this folder says. A repo with a UI and no verify skill cannot pass rung 5.
2. Find the feature files whose `paths` match the diff. Read them. They name the entry points and the end states.
3. Run the skill's Doctor step. A proof against a stale build is not proof.
4. Drive every entry point the feature file lists that the change can affect, and the success, cancel, error and empty paths. Use the real user path: clicks, keys, routes, commands. An internal function or a test-only endpoint is not the proof.
5. Capture the action and the end state. Check the side effects too: rows written, files made, messages queued.
6. Dispatch `runner` for a long click-through, with the exact steps. You read its report.
7. Run the skill's Cleanup step. Evidence survives cleanup.

Drive only the data that the verify skill's Data section allows. If that section says the instance holds real customer data and names no synthetic scope, stop and ask.

## Rung 6: see it

1. Save each screenshot where the verify skill's Evidence section says.
2. Dispatch `reviewer` (on the model the routing table names for judging screenshots) with the screenshot paths, the goal in one sentence and the feature file. Ask what each screenshot shows, whether it meets the goal, and what is broken or odd anywhere on the page, not only in the changed part. Give the goal, not the answer you hope for.
3. For a dense screen, send crops as well as the full page.
4. Write down each verdict: pass or fail, and what the reviewer saw.
5. A fail goes back to the build. Never judge the same screenshot again to get a pass.

Run the "Before shipping" check from the `design` skill on the real page here.

## Record it

The proof goes in the PR body's `Proof` section, as `ship/pr.md` says: one line per rung, the head commit it ran on, and for a fix the failing and the passing output. Proof from an older head does not count for a newer one. After a sync with the default branch, run rung 4 again.

## Keep the map true

- A PR that changes a mapped screen updates its feature file in the same PR.
- A changed UI path that no feature file claims gets added to one, or a new file.
- When the map and the app disagree, the app is right. Fix the map and say so in the PR.
- A full pass over the map is in `verify-skill.md`, "Maintenance pass". Run it when asked, or when three PRs in a row had to correct the map.

## When you cannot prove it

Say which rung you could not reach and why. Do not file a ready PR. Ask the user, or file a draft with "[do not merge]" and the results.

## Report

One line per rung climbed: what you ran, what you saw, the link. For a fix, give the failing output and the passing output. Never hand the user a check you could run.
