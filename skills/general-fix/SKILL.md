---
name: general-fix
description: Use whenever fixing a bug or a reported problem, or adding a rule, check, guard or special case to code. The rule is to fix the class of problem at the layer where the cases share a cause, not the one case that was reported. A stop-gap shaped by one incident adds if-statements, per-step copies and edge-case rules, and the code bloats with each new case. Covers how to find the general fix, closing the loop, briefing a fix, the signs of a stop-gap, when a narrow fix is right, and how to report it. Pairs with the design skill for UI work.
---

# Fix the class, not the case

A bug report names one case: one user, one record, one screen, one field. The fix that comes to mind first is shaped by that case. It adds a branch for it. The next report adds another branch, and after ten reports the code is ten if-statements that each know one story. The rule is a general fix instead: find the rule that broke and fix it where every case passes through, once.

Example. An order sat at "Pending" with nothing left to run. The first fix taught two checkout steps to retry under new keys (`:retry2`, `:retry3`). Three other steps could still stall the same way, and the refund path was not covered. The general fix was one rule in the workflow engine: a step that waits on an outside event holds a deadline, and the engine releases the hold when the deadline passes. Every step got it, and the per-step code was deleted.

## Before you write the fix

1. State the invariant that broke in one sentence, with no user, record, file or field name in it. "A waiting order always has an event to wait for or a deadline." If you can only state it with the case's specifics, you have not found it yet.
2. Find every path that can break it. Grep for the same pattern in other steps, other agents and other callers. Count them, and name them in your plan. For a rule about stored data, the paths are every writer of that data: create, edit, confirm, restore or reinstate, import and seed, upload, batch or model inference. List them all before you design, not only the one that crashed.
3. Find the layer they all pass through: the engine or runner, a shared helper, the schema, a shared component, or a shared instruction file. Fix it there, once. See "Where fixes go".
4. Delete what the general fix makes dead: the per-case branches, earlier stop-gaps, helpers and flags. A general fix that leaves the old special cases in place is not finished.
5. Test the invariant, not the incident. Write one test for each kind of path, for example one code step and one agent step. The incident is one of these tests, not all of them.
6. Close the loop. See the next section. A fix is not done while it opens a problem you can see.

## Close the loop

A fix often opens a new problem: a loop, a new dead state, a case it now handles wrongly. Seeing that problem and leaving it as a "risk" or a "follow-up" just moves the bug somewhere else. The aim is to end the class of bug, not to move it.

1. After the design or the change, list every new problem it could open. Check each invariant again against the new design, not the old code.
2. For each problem:
   - you see how to fix it: fold the fix into this design or change now, then go back to step 1 with the new version;
   - you can show it cannot happen: write the argument down in the PR, where it will be read;
   - only the user can decide it (money, law, a product choice): ask it as one plain question, with your recommendation.
3. Repeat until the list is empty. A risk paragraph that names a problem and its fix, but leaves the fix out, is not finished work.
4. If a second round finds another path of the same invariant, in this loop or in a `break-it` round, the layer is wrong. Stop patching. Take the design to `architect` with what each round found, and start the new design again at "Before you write the fix", step 3. Look at what the rule reads: if each round finds a new edge in how it compares things (by time, by name, by order of writes), change the rule so it no longer needs that input.

## Where fixes go

Read the "Where fixes go" section of the repo's instruction file when it has one. It says where each kind of rule lives in that repo, and it wins over this list.

If it does not say, put the rule in the first of these that every case passes through: the engine or shared runner, a shared helper, the schema, a shared component, a shared instruction file. Never one copy per step or per agent. A rule that holds for every agent goes in the shared instructions once. Judgement goes in instructions, not in a regex or a refusal in code.

For UI, the rule goes in one shared component or library, not in each screen. With the `design` skill: the screen shows the true state. It never explains around a state that should not exist. Fix the state.

## When you delegate the fix

A role or another session sees only its brief. It does not know this skill unless the brief carries it.

- For a fix or a new rule, the brief also carries: the invariant in one sentence; the close-the-loop rule in your own words; and "list every writer or path and mark each one: follows the rule, or fixed here". Never ask for items of the same invariant to be left "out of scope" or "for later". A reply that marks a path of the same invariant "out of scope", "still open" or "for later" is not done: that path is an open item, and the brief goes back.
- Ask for every new problem the change could open, with its fix folded in.
- A report that lists a same-class path as "still open", "follow-up" or "out of scope" is not finished. Send it back. Do not move it to the backlog, a ticket or a later PR.
- This covers only paths of the same invariant. Anything else found on the way (a new feature, a better screen, a different bug) goes to the repo's backlog.

## Signs a fix is a stop-gap

- It names the incident's specifics: a question kind, a key, an account code, an option's words, a customer.
- It is complete only if it is copied into other steps or agents, or it comes with a list of "also do the same in Y".
- It adds a new key, flag or status to route around behaviour that already exists.
- It treats the symptom after the fact (a retry, a re-ask, a screen message) while the state that caused it can still happen.
- It matches text with a regex to guess intent that a field or a type could carry.
- The commit message tells the story of one incident more than it states a rule.
- It names a risk it opens, and even how to fix that risk, and leaves the fix out.
- It reports a path of the same invariant as "still open", "follow-up" or a backlog item because it cannot crash anything today. Wrong data that does not crash is still the bug.

When you notice one of these in your own plan, stop and go back to step 1.

## When a narrow fix is right

- A live customer is blocked. Unblock them now, with a data fix or a small guard. In the same reply, say it is a stop-gap and propose the general fix. Never present the stop-gap as the fix.
- The general fix needs a migration, or a decision only the user can make. Ship the narrow fix with "stop-gap" in the PR title, and name the general fix as the follow-up.
- There is truly one path. If step 2 finds one caller and no sibling, the narrow fix is the general fix. Say so.

A stop-gap PR has "stop-gap" in its title. That holds it, so the user decides whether it merges.

## Report

In the PR body, under the headings the `ship` skill sets for it:

- Under `Invariant`: the invariant; how many paths it now covers; what was deleted.
- The path table: every path, and for a data rule every writer you checked, each marked "follows" or "fixed here". Put it under `Invariant` as a short table, inside the 40-line limit.
- Under `Risk`: the problems the fix could open, and for each one how it is closed: folded in, shown impossible, or a question for the user.

If the change is a stop-gap, say so in the title and name the general fix.
