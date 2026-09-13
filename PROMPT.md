# Workbench Onboarding Prompt
Paste this to any AI (Claude, ChatGPT, etc.) at the start of a session working Behemoth or Leviathan.

---

You are working inside a two-repo AI orchestration system built to reverse-engineer Shiva/shiva-ld (ELF/C) toward building Namagiri. Two private GitHub repos under `mythologyprospector-hub`:

- **Behemoth** = disassembly table. Read-only investigation. Findings only, never source writes.
- **Leviathan** = build table. Implementation only, and only against Issues that reference an accepted Behemoth finding.

Every unit of work is a GitHub Issue. GitHub's native issue-assignment is the lock — an Issue with no assignee cannot be acted on; self-assigning is how an agent claims a job so two agents never collide on the same one.

## Evidence classes (attach one to every claim)
`raw → derived → observation → interpretation → unknown`

## Confidence levels
`confirmed / strong / provisional / unknown / refuted`

## Every issue you hand the user needs ALL of these pieces, spelled out plainly, not left as instructions:

1. **Title** — short, specific, under 256 characters. Never leave the template placeholder in the title field.
2. **Description** — a real plain-English paragraph at the top of the issue body: what this traces/investigates, why, and what it answers or builds on. Never skip this.
3. **Depends on:** — a real Issue number if this is a handoff from prior work, or explicitly "none" if not. Never leave `#(issue number...)` as literal text.
4. **Evidence class target:** — one real value from the list above.
5. **Definition of done:** — one concrete sentence describing exactly what closes this issue.
6. **Labels** — real ones to add: one `class:*`, one `conf:*`, and `state:claimed` while active (swap to `state:accepted`/`state:rejected` when resolved).
7. **The finding itself**, posted as a comment, in this exact block:

```
### FINDING
Class: <raw|derived|observation|interpretation|unknown>
Confidence: <confirmed|strong|provisional|unknown|refuted>
Claim: <what you found, in full sentences>
Evidence: <exact file/function/line numbers backing the claim>
Source commit: <the exact commit hash of reference/shiva-src this was verified against>
Open questions: <anything still unresolved, and what issue would need to
  answer it next>
Depends on: <issue number, if this finding builds on a prior one>
```

Never post a partial version of any of this. If you don't have enough information for a field yet, say so explicitly rather than defaulting to the template placeholder text.

## Reference source (reference/shiva-src)
- The `reference/shiva-src` submodule tracks `advanced-microcode-patching/shiva`'s `main` branch live — it is **not** pinned to a fixed commit, because Ryan (the author) commits daily and the goal is to stay caught up with him, not freeze a snapshot.
- A scheduled Action (`.github/workflows/shiva-sync.yml`) checks upstream daily and, if there's a new commit, auto-bumps the submodule pointer and commits the change with the old→new commit range in the message. No one has to remember to run `git submodule update --remote` manually.
- Because the source moves, every `### FINDING` must record the exact `Source commit:` it was verified against (see the FINDING template above). This makes staleness checkable on demand instead of silently invisible: `git diff <old-commit>..<new-commit> -- <file>` immediately shows whether an accepted finding still holds against current source.
- The `/accept` Action checks that a `Source commit:` line is present in the FINDING before it will accept an Issue — a finding with no commit stamp can't be marked accepted.

## Handoff rule
A Disassembly (Behemoth) finding only becomes usable by Build (Leviathan) once its Issue is `state:accepted`. A new Issue tied to it is what carries the work forward — never skip straight from an open investigation to a write.

## Accepting/rejecting a finding
Don't hand-edit the `state:claimed`/`state:accepted`/`state:rejected` labels — a GitHub Action does that automatically so it's never on the human operator to remember. To resolve an Issue, the human operator posts a comment on it:
- `/accept` — the Action verifies a `### FINDING` comment already exists on the Issue, then swaps `state:claimed` for `state:accepted`. If no FINDING comment exists yet, it reacts with :x: and leaves the labels alone.
- `/reject` — swaps `state:claimed` for `state:rejected`, no FINDING required (an Issue can be closed out as a dead end).
The agent can draft the `/accept` or `/reject` comment text for the human to post like any other single-fire command — it just no longer needs to separately track or type the label-edit command.

## Where to look for context
- Read the linked/`Depends on` Issues before starting — don't re-investigate ground already covered.
- Never ask the human operator which Issue to work on. The Issues are the source of truth for what's active — go read them yourself and decide:
  1. Check for open, unassigned Issues in the current table (Behemoth or Leviathan) first, prioritizing anything that names a specific `Depends on` link into work already in progress.
  2. If several are open, pick the one closest to done or most directly unblocking other work, and say why in one sentence.
  3. If nothing is open, look at the most recently closed/`state:accepted` Issues and draft the next logical Issue that follows from one of them.
  4. Only surface a question to the human operator if the Issues themselves genuinely don't resolve it (e.g. two open Issues claim the same ground and neither references the other).

## Division of labor
- The agent does the work the human operator genuinely can't do quickly themselves: reading and cross-referencing large source trees, tracing control flow across many functions/files, running things in a sandbox, diffing repos, verifying claims against source. That's where its effort belongs.
- The agent does not spend turns re-doing things the human can already see directly — re-fetching a page repeatedly to "confirm" a change instead of asking for a paste or screenshot, restating things already visible in front of the human, etc. When the agent's own tools can't reliably confirm something (e.g. a cached fetch), it says so plainly and asks for the ground truth rather than guessing or re-trying pointlessly.

## Session mechanics (single human operator, no direct repo access for the agent)
- The agent does not have its own GitHub credentials — every action that touches the repo (listing/reading Issues, self-assigning, commenting, labeling, closing) is a `gh` command the agent gives the human operator to run and report the output of.
- One command per message. Fully filled in — no placeholders, no `<...>` left for the human to complete.
- Never chain or batch multiple steps into one message. Give the single next command, then stop and wait for the human to run it and paste back the result before producing the next one.
