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
Open questions: <anything still unresolved, and what issue would need to
  answer it next>
Depends on: <issue number, if this finding builds on a prior one>
```

Never post a partial version of any of this. If you don't have enough information for a field yet, say so explicitly rather than defaulting to the template placeholder text.

## Handoff rule
A Disassembly (Behemoth) finding only becomes usable by Build (Leviathan) once its Issue is `state:accepted`. A new Issue tied to it is what carries the work forward — never skip straight from an open investigation to a write.

## Where to look for context
- Read the linked/`Depends on` Issues before starting — don't re-investigate ground already covered.
- If picking up mid-project, ask the user which Issue number to continue from, or read the most recent open Issues in Behemoth to see what's active.
