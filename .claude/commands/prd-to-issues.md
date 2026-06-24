---
name: prd-to-issues
description: Break a PRD down into individual, actionable issues for the project issue tracker. Use after a PRD has been written (e.g. with write-a-prd) when the user wants to slice the work into trackable issues.
---

The PRD is already in context. Do NOT re-read or re-summarize it — go straight to slicing.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Process

1. Read the PRD from context (do not ask the user to paste it).

2. Propose a **vertical slice breakdown** — a numbered list of issues where each issue delivers an end-to-end slice of value (schema + API + UI if applicable), or is a well-scoped unit of backend/infra work. Aim for issues that can be implemented independently or in a clear sequence.

Present the breakdown to the user in the following format before creating any issues:

Here's my proposed breakdown:

N. <Issue Title> (<layers involved, e.g. schema + API + UI>)

- Type: AFK
- Blocked by: <None | #N>
- User stories: <comma-separated numbers from the PRD's user stories list>
- Scope: <1-2 sentence plain-language description of exactly what this issue covers>

(repeat for each issue)


Check with the user that the breakdown looks right before proceeding.

3. Once confirmed, create each issue in the project issue tracker:
   - Title: as proposed
   - Body: include Scope, relevant User Stories (quoted), and any Implementation Decisions from the PRD that apply specifically to this issue.
   - Type: AFK (unless the user specifies otherwise)
   - Label: `ready-for-agent`
   - Blocked-by links: set where applicable

4. Reply with a summary list of the created issue links.

## Rules

- One issue per slice. Do NOT bundle unrelated concerns into one issue.
- Prefer vertical slices (schema + API + UI together) over horizontal layers ("all DB changes", "all API routes").
- Each issue must be self-contained enough for an agent to implement without needing to read every other issue.
- Do NOT invent scope that is not in the PRD. If something is marked Out of Scope in the PRD, it stays out of scope.
- Do NOT include file paths or code snippets in issue bodies.
- Keep Scope descriptions short and precise — what changes, not how.
