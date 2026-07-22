# Tickets carry seams forward from the spec; they don't decide them afresh

`/to-tickets` publishes tickets labelled `ready-for-agent`. Those tickets are executed by an agent working alone, from a fresh context, with the ticket as its brief. Anything the ticket doesn't say is decided unattended — including where the tests go.

That is backwards relative to when a human is actually available. `/to-tickets` runs interactively; implementation does not. So the ticket gains a **Seams** section, and the seam decision moves to the moment someone can weigh in on it.

## The constraint: `/to-spec` already decides seams

The obvious move — have `/to-tickets` decide seams per ticket — creates a second decision-owner. `/to-spec` already sketches the seams with the user ("Existing seams should be preferred to new ones. Use the highest seam possible… Check with the user that these seams match their expectations") and records them under **Testing Decisions**. `/implement` then builds at those "pre-agreed seams."

If `/to-tickets` decided independently, a ticket's seams could silently contradict the spec that produced it — and the implementing agent, reading only the ticket, would never see the conflict. Two documents, both authoritative, one of them invisible at the point of use.

But `/to-tickets` explicitly accepts "a plan, spec, or the current conversation." The no-spec path is a first-class input, not an edge case, so pure carry-forward leaves the original gap open exactly where no seam was ever agreed.

## Decision

- When the source is a **spec**, `/to-tickets` narrows its Testing Decisions to the seams each slice touches and **restates** them. It does not re-decide. **The spec wins on conflict**; a slice that seems to want a different seam is raised with the user during the step 4 quiz rather than quietly overridden.
- When there is **no spec**, `/to-tickets` originates the seams, inheriting `/to-spec`'s rules — prefer existing seams, use the highest one, keep the total low.
- Seams are named by **module and interface**, never by file path. This extends the skill's existing no-file-paths rule rather than carving an exception out of it: the two share a rationale (a module's name survives a file move; a path doesn't), which matters because step 2 actively encourages prefactoring that moves files.
- **Every** ticket carries the section, including prefactors and expand–contract batches, which say "None new — the existing suite over \<module\> must stay green." Omission would be ambiguous in the way this change exists to remove: an agent cannot distinguish "we decided no seam was needed" from "nobody decided."

## Invariants this creates

- Seams in a ticket are always traceable to a human decision — carried from the spec, or confirmed in the step 4 quiz.
- There is exactly one authoritative seam decision per feature. The ticket is a projection of it, never a competing source.
- Assessing *deficiencies* in existing coverage is deliberately out of scope. An open-ended "improve coverage here" instruction in a `ready-for-agent` ticket has no stopping condition, which reintroduces an unbounded scope decision made with nobody watching — the same failure this ADR closes. If that need is served, it belongs in its own workflow.
