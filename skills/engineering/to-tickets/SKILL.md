---
name: to-tickets
description: Break a plan, spec, or the current conversation into a set of tracer-bullet tickets, each declaring its blocking edges, published to the configured tracker — edges as text in one file per ticket locally, or native blocking links on a real tracker.
disable-model-invocation: true
---

# To Tickets

Break a plan, spec, or conversation into a set of **tickets** — tracer-bullet vertical slices, each declaring the tickets that **block** it.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Process

### 1. Gather context

Work from whatever is already in the conversation context. If the user passes a reference (a spec path, an issue number or URL) as an argument, fetch it and read its full body and comments. When the source is a spec, read its **Testing Decisions** — the seams were already agreed there with the user, and the tickets carry them forward.

### 2. Explore the codebase (optional)

If you have not already explored the codebase, do so to understand the current state of the code, and note the seams the existing tests already sit at. Ticket titles and descriptions should use the project's domain glossary vocabulary, and respect ADRs in the area you're touching.

Look for opportunities to prefactor the code to make the implementation easier. "Make the change easy, then make the easy change."

### 3. Draft vertical slices

Break the work into **tracer bullet** tickets.

<vertical-slice-rules>

- Each slice cuts a narrow but COMPLETE path through every layer (schema, API, UI, tests) — vertical, NOT a horizontal slice of one layer
- A completed slice is demoable or verifiable on its own
- Each slice is sized to fit in a single fresh context window
- Any prefactoring should be done first

</vertical-slice-rules>

Give each ticket its **blocking edges** — the other tickets that must complete before it can start. A ticket with no blockers can start immediately.

Give each ticket its **seams** — where the test boundary for that slice sits. Implementation runs unattended, so a seam left undecided here is decided by an agent with nobody in the room; deciding it now is the point.

<seam-rules>

- If the source was a spec, narrow its Testing Decisions to the seams this slice actually touches and restate them. **The spec wins** — if a slice seems to want a different seam, say so in step 4 rather than quietly overriding it.
- Otherwise propose them: prefer existing seams to new ones, use the highest seam possible, and keep the total across the change as low as you can — the ideal is one.
- Name seams by **module and interface**, never by file path — "the Order intake module's interface", not a path. Use the `/codebase-design` vocabulary (module, interface, port, adapter) and the project's `CONTEXT.md` domain names. This is the same rule as "no file paths" below, for the same reason: a module's name survives the prefactoring above, a path doesn't.
- Every ticket gets a seam, including prefactors and expand–contract batches. When a ticket adds no new test surface, say so — "None new — the existing suite over the Order intake module must stay green." An absent section reads as an oversight; an explicit "none" reads as a decision.

</seam-rules>

**Wide refactors are the exception to vertical slicing.** A **wide refactor** is one mechanical change — rename a column, retype a shared symbol — whose **blast radius** fans across the whole codebase, so a single edit breaks thousands of call sites at once and no vertical slice can land green. Don't force it into a tracer bullet; sequence it as **expand–contract**. First expand: add the new form beside the old so nothing breaks. Then migrate the call sites over in batches sized by blast radius (per package, per directory), each batch its own ticket blocked by the expand, keeping CI green batch to batch because the old form still exists. Finally contract: delete the old form once no caller remains, in a ticket blocked by every migrate batch. When even the batches can't stay green alone, keep the sequence but let them share an integration branch that all block a final integrate-and-verify ticket — green is promised only there.

### 4. Quiz the user

Present the proposed breakdown as a numbered list. For each ticket, show:

- **Title**: short descriptive name
- **Blocked by**: which other tickets (if any) must complete first
- **What it delivers**: the end-to-end behaviour this ticket makes work
- **Seams**: where this slice is tested, or "none new"

Ask the user:

- Does the granularity feel right? (too coarse / too fine)
- Are the blocking edges correct — does each ticket only depend on tickets that genuinely gate it?
- Should any tickets be merged or split further?
- Are the seams right? Flag any that depart from the spec's Testing Decisions.

Iterate until the user approves the breakdown.

### 5. Publish the tickets to the configured tracker

Publish the approved tickets. **How** depends on the tracker `/setup-matt-pocock-skills` configured — the tickets are the same either way, only the shape of the blocking edges changes:

- **Local files** → write one file per ticket under `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` in dependency order (blockers first). Each file's "Blocked by" lists the numbers/titles it depends on. Use the per-ticket file template below — one ticket per file, never a single combined file.
- **A real issue tracker (GitHub, Linear, …)** → publish one issue per ticket in dependency order (blockers first) so each ticket's blocking edges can reference real identifiers. Use the platform's native blocking / sub-issue relationship where it has one; otherwise set each ticket's "Blocked by" to the blocking issues. Apply the `ready-for-agent` triage label unless instructed otherwise — the tickets are agent-grabbable by construction.
When the tracker is GitHub Issues, use the `--blocked-by` flag on `gh issue create` to wire up GitHub's native issue dependency graph for each issue that has blockers. Since blockers are created first, their issue numbers are available at creation time.

```shell
# Example: issue being created is blocked by #12 and #15
gh issue create --title "..." --body "..." --label "..." --blocked-by 12,15
```

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

Do NOT close or modify any parent issue.

<local-ticket-template>

# <NN> — <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective — not a layer-by-layer implementation list.

**Seams:** where the test boundary sits — named by module and interface, never by file path. What to fake across it, and what must not be. Or "None new — the existing suite over <module> must stay green."

**Blocked by:** the numbers/titles of the tickets that gate this one, or "None — can start immediately".

**Status:** ready-for-agent

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

</local-ticket-template>

<issue-template>

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective — not layer-by-layer implementation.

## Seams

Where the test boundary for this slice sits — named by module and interface, never by file path ("the Order intake module's interface"). Name what to fake or substitute across it, and anything that should deliberately not be faked. If this ticket adds no new test surface, say so: "None new — the existing suite over <module> must stay green."

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Blocked by

- A reference to each blocking ticket, or "None — can start immediately".

</issue-template>

In either form, avoid specific file paths or code snippets — they go stale fast. This holds for the Seams section too: name the module and its interface, which survive a file moving, rather than the path, which doesn't. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.
