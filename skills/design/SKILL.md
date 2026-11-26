---
name: design
description: Use whenever building or changing any user interface - web pages, apps, components, dashboards, landing pages, interactive widgets - and whenever writing UI copy (hints, captions, subtitles, helper text, empty states, labels). Also for "design this screen", "mock", "mock it up" and "Mobbin". The rule is that an interface shows how it works instead of explaining itself; instruction text and explanatory subtext read as AI slop. Covers how to design a screen (research how real apps solve it on Mobbin and the web, the product's design language, judging the whole page, a mock for comments), what to cut, what to show instead, motion and haptics, and a check before shipping.
---

# Design

An interface should show how it works. Text that explains the interface ("Scroll the stubs, or use the arrow keys") is the clearest sign a page was made by an AI that did not trust its own design. The rule is that it goes, every time.

This works in any repo and on any surface. Read the product's stack, components and design language from the repo. Never assume a framework.

## Show, don't tell

The screen shows the true state. When a state should not exist, fix the state. Never explain around it.

## Cut on sight

- Usage instructions: "Scroll to...", "Click to...", "Tap to...", "Hover to...", "Swipe...", "Drag...", "Use the arrow keys...".
- Captions that narrate the interaction: "The map updates as you scroll", "fills in as you go", "Select a flight to see its route".
- Subtitles that restate the heading, and stacks of eyebrow label plus title plus standfirst plus hint over one component.
- Filler invitations: "Explore", "Discover", "Here you can...", "Take a look", "Learn more" with nothing specific behind it.

Text that explains around a confusing state ("Saved. We will carry on from here." on an order that will never ship) is a bug report, not copy. Fix the state, then show it plainly. the `general-fix` skill covers how.

Keep text a person needs in order to act or to stay safe: form errors, a confirm on a destructive action, a legal line. Keep accessible names (`aria-label`, `.sr-only` text): they are for screen readers, not visual chatter, and removing the visible hint must never remove the accessible one.

## Show it instead

| Instead of saying | Show |
|---|---|
| "Scroll sideways" | Let the next item peek past the edge. Give the row one small nudge the first time it comes into view. Put a track under it (ticks, a belt, a conveyor) that bobs while idle and runs with the scroll. Snap points. |
| "Click or tap to open" | Hover and press states, a pointer cursor, a + or chevron that turns. |
| "Drag" | A grab cursor, a visible handle, a slight wobble on first view. |
| "This updates live" | Let it update, and animate the change. |
| "Use the keyboard" | A visible focus ring and keys that just work (arrows, Home, End). |
| "You can feel it" | Haptics on discrete steps. |

## Motion and haptics rules

- One idle hint at most, and it stops for good once the person interacts. No looping attention-grabbers.
- `prefers-reduced-motion`: no nudge, no bob. The still state must read on its own.
- Haptics only after a user gesture, only on discrete events (an item snapping to centre, a toggle), short (5 to 10 ms), throttled to one per ~45 ms, never continuous. Android: `navigator.vibrate(8)`, wrapped in try. iOS Safari has no Vibration API; from iOS 18, clicking a hidden `<label><input type="checkbox" switch></label>` gives a system haptic. Keep it `aria-hidden`, off screen, and restore keyboard focus after the click. Say plainly that phone haptics are untested until someone has felt them on a device.
- Without JS the page must still make sense with no instruction text: plain lists and headings.

The web names above are for a web UI. On another surface, use that platform's own reduced-motion setting, haptics and accessible names, under the same rules.

## How to design a screen

Use this method for every build or change that touches a UI. The mock comes before the build.

1. Check the plan three ways: a design critique (hierarchy, consistency, usability), a UX copy pass (every string earns its place, under the rules above), and an accessibility review (contrast, keyboard, focus, touch target size, screen reader names). Use the runtime's design critique, UX copy and accessibility skills when it has them. If not, do the three checks yourself.
2. Look at how real apps solve the same screen before you design it. Dispatch `researcher` with the screen's job in one sentence and the platform. Ask for patterns with their sources, never for a design.
   - If the Mobbin setting in the core rules is `use` and the session lists the Mobbin tools, the brief says to use them: search screens, flows and sections for the same job; save each pick worth citing to one collection per screen, made or found by name; add a note to each pick on what to take from it; and return the collection's contents with links.
   - Otherwise (the setting is `off`, or the session lists no Mobbin tools), the brief asks for web search instead, and your report says the research used web search and why.
   - Never describe a screen you did not open. A pattern with no source is left out.
3. Fit the product's own design language. Read the brand and design docs that the repo's instruction file names before you propose anything. If it names none, read the existing screens and shared components, and match them.
4. Judge the whole page, not one component. Show the component inside the real page, with whatever sits above and below it. A part that looks right alone can clash with what sits under it, and the fix is often to merge the two.
5. Give each page one purpose. Check every proposal against the product's page map, and do not repeat one page's job on another.
6. Show the full page as a mock in the real app frame, that the user can open in a browser. Cite the saved picks it draws on. If the user asked to review the mock, post it and stop as the `s-mode` skill "Stop only for" says. Otherwise post it, and build from it.
7. A role or another session cannot see this skill. Write steps 1 to 6 into the brief of any role that builds or changes UI, with the rules of this skill that the work touches.

After the build, prove it with the `prove` skill rung 6: screenshots of the real page, judged with vision against the goal and the approved mock.

## Copy a customer reads

Copy a customer reads never casts them as late or holding things up, however soft the words. State the facts (a state, a chip, a count) and cut sentences that nudge or remind.

## Patterns to avoid

These read as generated. Do not use them unless the product's design language already does:

- a cream or off-white page background;
- italic accent words in headlines;
- numbered labels such as 01, 02, 03;
- monospace labels;
- pill-shaped buttons.

## Before shipping

Run these on the real page during the `prove` skill rung 6.

1. Read every visible string on the page. For each, ask whether someone who never sees it would be worse off. If not, delete it.
2. Grep the built page's visible copy for: scroll, click, tap, hover, swipe, drag, "use the", "as you", "here you can". Each hit needs a reason to stay.
3. Check the replacement affordance actually shows: open the page cold, do not touch anything for two seconds, and see whether it is obvious what moves.
