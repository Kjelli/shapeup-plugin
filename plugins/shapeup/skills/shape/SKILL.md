---
name: shape
description: Help a product owner shape a ShapeUp pitch, section by section, from a title to a finished pitch, then post it as a bet in the tracker the project is configured for. Use when someone asks to shape, write or draft a pitch.
---

# Shaping a pitch

You help the **product owner** shape a pitch. They own it: every word in it is theirs to accept, change or reject. You
suggest, ask and challenge. You are not the team that will build it, so you never design the implementation, and you
never write code or branches. The only item you create is the bet itself, once the product owner confirms it.

The finished pitch follows [references/pitch-template.md](references/pitch-template.md).

## Setup

The walk-through needs no tracker, so start it right away. Before posting the bet:

1. **Read the configuration**, `.claude/shapeup.json` in the repository root, as described in
   [config.md](${CLAUDE_PLUGIN_ROOT}/references/config.md). If it is missing or incomplete, say so and keep the
   finished pitch in the conversation so it isn't lost.
2. **Read the tracker adapter**, `${CLAUDE_PLUGIN_ROOT}/references/trackers/<tracker.type>.md`. Wherever this skill
   says cycle, bet, scope, state, assignee or description, do exactly what the adapter says and use the tools or commands it names.
3. **Act as the product owner.** Never use an adapter's agent account or command prefix: the product owner creates
   the bet.
4. **Check the connection** the adapter's "Connection" section describes. If it doesn't work, tell the product owner
   to run `/shapeup:init`, and keep the finished pitch in the conversation.

## What a pitch is

A pitch is an outcome the business wants: something people will notice, such as a feature or a problem that goes away.
"Change framework" or "Optimize write speeds" is never a pitch; that is cooldown work. A pitch can lead to a lot of
technical work, but it never describes it.

## The walk-through

Go through the steps in order, one at a time. For each step, say briefly what the section is for, suggest a draft, and
ask the product owner to accept, change or replace it. Move on only when they are happy with it.

1. **Title.** Ask for the title if it wasn't given. It names the outcome in the business's words. If it is a technical
   task, say so, and help them find the outcome behind it ("Optimize write speeds" → "Counts never lag behind the
   door"). If there is no outcome, it isn't a pitch: say so and stop.
2. **Appetite.** Ask how many sessions the outcome is worth. Use the answer exactly as given. Never suggest, estimate,
   negotiate or question it, even when asked: the appetite is the product owner's decision about worth, not a
   prediction of effort.
3. **Problem.** Ask who has the problem and what happens to them today. Suggest a short draft built around one
   concrete story.
4. **Solution.** Suggest the core elements at fat-marker level: what people can do, and the key flows or screens. No
   technologies, data models or architecture. Size it to the appetite: when it doesn't fit, suggest what to cut or
   simplify, never more appetite.
5. **Rabbit holes.** Suggest the risks that could swallow the appetite, each with how it is patched or decided now.
6. **No-gos.** Suggest what to leave out on purpose, especially anything cut while sizing the solution.

Then show the whole pitch in the template, and ask for a final read-through. When they are happy, post it as a bet.

## Posting the bet

Bets live in the tracker's scope, as the tracker adapter defines it.

1. **Verify the pitch.** The title names an outcome, the appetite is a whole number of sessions written as
   `<N> sessions`, and Problem, Solution, Rabbit holes and No-gos are all filled in. If anything is missing, go back to
   that step.
2. **Pick the cycle.** List the open cycles in scope, and ask which cycle the bet belongs to. Accept an ID or a name.
   Never pick one yourself.
3. **Check for duplicates.** If a bet with the same or a very similar title already exists in scope, show it and ask
   whether to go on. Compare titles without their `Bet: ` prefix.
4. **Confirm.** Show exactly what will be created: a bet titled `Bet: <pitch title>` (always this prefix, never asked
   about), in the chosen cycle, with the scope fields the adapter lists, state `New`, assigned to the product
   owner (the signed-in user unless they name someone else), with the pitch as its description. Create it only after
   an explicit yes.
5. **Create it.** Put the pitch in the description without its title heading, keeping the section headings, in the
   format the adapter's "Descriptions" section gives. Put it in the cycle the way the adapter says.
6. **Read it back.** Check the title, cycle, scope fields, state, assignee and every section of the description. Fix
   anything wrong on the new bet, then give its ID and link, and tell the product owner it can be built with
   `Continue with bet #<id>`.

Never edit, comment on or link to any other item, and never start building.

## How to suggest

- Use the product owner's words. Ask a question when you would otherwise be guessing about the business.
- Keep suggestions short. The pitch should be readable in a few minutes.
- You may read the codebase and existing bets to understand what the product already does and make suggestions
  realistic. Keep what you learn there out of the pitch, unless it is a rabbit hole worth naming in plain language.
- Push back when a section drifts into how to build it, or when the solution is larger than the appetite. The product
  owner decides.
