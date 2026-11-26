# The repo's verify skill

Every repo with a surface a user touches needs a scripted way to drive the real thing and prove behaviour: launch it, use a feature the way a user would, and capture evidence. That is the repo's verify skill. Rung 5 of the `prove` skill runs it.

You write it for the next agent, not for a person. It will be read cold, mid-task, by an agent that has never seen the app.

## Where it lives

The skill lives at `.claude/skills/verify/` in the repo. That is the one place both runtimes can read. Claude Code reads only `.claude/skills/`. Add `.agents/skills.json` with `{"entries":[{"path":".claude/skills"}]}`, so Antigravity reads the same folder.

```text
.claude/skills/verify/
  SKILL.md                 Launch, Doctor, Drive, Data, Evidence, Cleanup, Helpers
  features/README.md       The feature map: index, baseline, conventions, sweep order
  features/<feature>.md    One file per feature, shaped as in section 4
  scripts/                 Helpers the skill ships, each executable
```

## 1. Interview the repo, not the user

Answer these from the code. Ask the user only what you cannot observe.

- Surface: what does a user touch? A web UI, a CLI or TUI, a desktop app, an API, a mobile app, a library. A repo can have several. Pick the main one and note the rest.
- Run: how does the app start locally? Prefer the repo's own documented command: its package scripts, task runner or README. Note ports, environment variables, seed data and sign-in.
- Drive: how can an agent use it from a script? An existing harness first: end-to-end specs, a debug port, plain HTTP, a PTY helper. Only then a generic recipe: a browser driver for web and desktop shells, a PTY or terminal multiplexer for a CLI or TUI, plain HTTP for a service.
- Observe: what evidence can be captured? Screenshots, terminal transcripts, response bodies, logs, exit codes, stored state.
- Data: which database, files and outside services does a local instance talk to? Are any of them shared with production or hold real customer data?
- Isolate: can two instances run side by side (ports, data dirs, profiles)? If not, the skill says so. Refusing to drive a shared instance twice beats corrupting the user's session.

If the checkout does not build or start as it is, fix that first or report it exactly. A skill written against a broken base teaches wrong steps. When a missing asset that does not matter blocks startup, the skill may create it, marked as verification scaffolding, and remove it in Cleanup.

## 2. Write SKILL.md

Frontmatter is `name: verify` and a `description` that names the app, the surface and when to use it. Without frontmatter the skill never loads.

The body has these sections, in this order. Each comes from what the interview found. No placeholder is left.

- `## Launch`: the exact command that starts the app for verification, and how to tell it is ready, such as a log line or a prompt. Launch reaches a usable state from an empty data dir, migrations and seed data included. Ready means usable, not only that a process answers. For a short-lived CLI there is no server: Launch builds the binary or installs dependencies once, and each drive starts in its own isolated session.
- `## Doctor`: one read-only check that answers "is this instance worth driving?": the process is up, the build is current, the port is ours, sign-in is valid. Run it first, and again whenever anything looks off.
- `## Drive`: the harness recipe with real selectors and commands from this repo, not examples. Prefer stable handles (accessible names, data attributes, prompt strings, routes) over coordinates and tab order.
- `## Data`: see the next section. Required.
- `## Evidence`: what to capture for a proof and where it goes. The proof standards: use the real user path, not internal setters or test-only endpoints; capture the action and the resulting state, not only the final screen; check side effects (files written, rows inserted, messages queued) as well as what is visible; mock only where a production boundary already isolates the outside system. When the safe path is a dry run or a test mode, check by observation what it skips (files, network, git refs). Some dry runs still touch the network.
- `## Cleanup`: how to stop what the run started. Never kill by process name. Kill what you started. Cleanup removes instances and scratch state, never the evidence. Evidence survives in the place the skill names.
- `## Helpers`: every script the skill ships is executable, and its call is shown in the skill body. A helper the reader has to reverse-engineer is not a helper.

## 3. The Data section

`## Data` is required. A UI proof from a repo whose verify skill has no `## Data` section does not count. It says:

1. Which database, storage and outside services a verification run talks to, and whether any of them is shared with production.
2. What data a run creates, and where: the tables, files, queues or accounts, and the name or prefix that marks a row or file as made by a run.
3. What data a run may touch, and what it must never touch.
4. How synthetic data is made (seed command, fixture, factory) and how it is removed. Cleanup runs the removal.
5. Whether screenshots and logs can hold real customer data. Only evidence that cannot may go into a PR.

If the instance holds real customer data and the section names no synthetic scope, a run stops and asks the user. If the run writes to a live service, that is on the yes-list: record the user's standing decision here, with its date.

## 4. Seed the feature map

Write `features/README.md` and one file per user-facing feature. Start with the top 3 to 5, found from routes, commands, menus or docs.

The README holds:

- Baseline: the preconditions every recipe starts from (fresh build, Doctor passed, seed data, isolated data dir).
- Conventions: stable handles first, wait on observable end states and never on fixed sleeps, restore seeded data after a mutation, never remove evidence.
- Proof and skip reporting: drive every reachable entry point and the success, cancel, error, empty and persistence paths the change can affect. Name an unreachable path with the attempted command and the missing precondition. Never report a skipped entry point as verified through another path.
- Index: one line per feature file, with what it covers.
- Sweep order: the order to walk the map for a broad regression.

Each feature file:

- starts with frontmatter `paths: [<globs>]`: the source paths that belong to the feature, so a diff maps to its feature files;
- then an H1 title and one paragraph on what the user sees;
- then exactly four `##` sections, in this order: `Sub-features`, `How to get to it`, `Driving it`, `Gotchas`;
- is written from the user's point of view, and names the end state that proves the feature works.

A missing precondition is a `Gotchas` entry and a reported failure. Never weaken a journey to make it pass. The map is the repo's verification source: a proof that drives one easy entry point is not complete when the map lists others.

## 5. Prove the skill before you hand it over

Run its own steps end to end once: Launch, Doctor, drive one mapped feature, capture evidence, Cleanup. After Cleanup, confirm the evidence is still at the place the skill names. A Cleanup that eats the proof fails this step.

Fix what fails, and run Cleanup after every failed attempt too, so broken attempts do not leave processes and ports behind. A skill that never ran is a draft.

If you proved it in a stand-in for the real target (a container in place of a fresh machine, a local stack in place of a remote one), list every setup step the stand-in did that the target will not. Those steps stay unverified until a run on the real target covers them.

## Maintenance pass

The map rots when the app changes. Run this pass when the user asks, or when three PRs in a row had to correct the map. The unit is the feature: cover every feature file from source and drive every feature live.

Edit only the verify skill's own folder: `SKILL.md`, `features/` and its scripts. Never edit product code in this pass. A behaviour the map describes that the app no longer has is either drift (fix the map) or a product regression (report it, never hide it in the docs).

1. Index. Read `features/README.md` and list the files beside it. Fix missing, extra, duplicate and dead entries.
2. Source wave. Dispatch one `scout` per feature file, all at once, read-only. Each explains from source how the feature works, flags likely drift with file and line, and returns one live recipe. They never drive the app and never edit files. Reply shape: summary, source entry points, drift or none, one recipe.
3. Reconcile. Every feature file has a summary back. Merge overlapping recipes into as few app states as you can. Spot-check the cited drift. Scan recent changes for user-facing surfaces the map lacks. Call one missing only with a source path.
4. Live pass. Required even when the source looks clean. You drive, one instance at a time, as the skill's Launch section says. Drive every feature once. Hold three rules the whole pass:
   - Never drive an instance that has not passed Doctor since it last did something surprising. Doctor before the first drive, on each fresh session, and after any failed drive. Where Doctor cannot see the fault, reset to a known state or relaunch.
   - Evidence captured so far survives every cleanup. Check it at its named place. Do not assume.
   - Nothing a drive started outlives its use. Clean up after a failed attempt too. On a shared instance, clean up what you made, not the instance.
   A Doctor failure caused by drift in the skill is drift: fix it and retry once. A feature you cannot reach is `unreachable` only with the exact precondition (sign-in, plan, platform, outside state) and the route you tried. If the map omits that precondition, that is drift. Drive any harness fix live before it ships. Tear down after the last drive.
5. Triage. A wrong or missing description is drift: fix it. Working behaviour the harness cannot drive is a harness gap: fix it, under the Helpers rule. Behaviour that is broken is a product gap: report it to the user and keep it out of this PR.
6. Ship or stop. Say which outcome it was:
   - `clean`: every feature had source and live coverage, and nothing needs to change. No branch, no PR.
   - `changed`: one PR of proven corrections, filed through the `ship` skill. Read every changed file again first.
   - `blocked`: coverage could not finish, or a proven fix could not ship safely. Say exactly what blocked it.

Keep short run notes (features covered, unreachable preconditions, drift found, outcome) outside git.
