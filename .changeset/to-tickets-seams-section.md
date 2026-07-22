---
"mattpocock-skills": patch
---

Teach `/to-tickets` to emit a **Seams** section on every ticket, so the testing-boundary decision is made while a human is in the room rather than by an unattended agent mid-implementation.

Tickets are published `ready-for-agent` and built from a fresh context, so anything the ticket doesn't say is decided with no oversight. Both templates (`<issue-template>` and `<local-ticket-template>`) now carry a `Seams` section directly after "What to build", and the step 4 quiz asks about seams alongside granularity and blocking edges.

- **Carried forward, not re-decided.** `/to-spec` already agrees seams with the user under **Testing Decisions**, so `/to-tickets` narrows those to the seams each slice touches and restates them — **the spec wins on conflict**, and a slice that wants to depart is raised in the quiz rather than quietly overriding it. Only when there's no spec does it originate seams, inheriting `/to-spec`'s rules (prefer existing, aim high, keep the total low). Step 1 now reads the spec's Testing Decisions; step 2 notes the seams existing tests already sit at.
- **Named by module, never by path.** This extends the skill's existing no-file-paths rule instead of carving an exception out of it — both rest on the same rationale, that a module's name survives the prefactoring step 2 encourages and a path doesn't.
- **Present on every ticket**, including prefactors and expand–contract batches, which say "None new — the existing suite over \<module\> must stay green." Omission would be ambiguous exactly where this change exists to remove ambiguity.

New ADR `0003-tickets-carry-seams-forward-from-the-spec.md` records the single-decision-owner choice and why coverage-gap assessment is deliberately out of scope. Docs page for `to-tickets` re-synced.
