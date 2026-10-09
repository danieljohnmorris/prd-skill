---
name: prd
description: "Write a PRD and split it into tracker-ready tickets that an autonomous coding agent can take from spec to PR. Use when planning a feature, writing a ticket, or asked to create a PRD/spec. Triggers on: create a prd, write prd for, write a ticket, spec out, plan this feature, requirements for. Tracker-agnostic: Plane, Linear, GitHub Issues, or plain markdown."
user-invocable: true
---

# PRD Generator (factory edition)

Forked from [snarktank/ralph `skills/prd`](https://github.com/snarktank/ralph/blob/main/skills/prd/SKILL.md) (MIT). Keeps Ralph's structure (clarifying questions → user stories → functional requirements → non-goals → acceptance). Adds:

- **One P0 slice per job.** Every ticket is one agent session, one branch, one PR.
- **Verifiable checks**, including concrete UI checks a headless agent or reviewer can run.
- **Spec in, PR out.** Each ticket is self-contained enough to hand straight to a coding agent.
- **Tracker as a plug-in.** The skill never assumes a specific tracker, host, or personal setup.

**Important:** Do NOT implement. Produce the PRD and tickets only.

---

## The Job

1. Receive a feature description (from the user or an existing issue).
2. Ask 3–5 essential clarifying questions with lettered options. Skip any already answered.
3. Write the PRD (structure below).
4. Split it into ordered **slices**. Mark exactly one as **P0** (the next job).
5. Output: save the PRD as markdown, and if a tracker is configured, emit tickets in that tracker's shape (see *Tracker adapters*). Never create or post tickets without explicit user approval; print them for review first.

---

## Step 1: Clarifying Questions

Ask only where the prompt is ambiguous. Focus on:

- **Problem/Goal:** What problem does this solve?
- **Core Functionality:** What are the key actions?
- **Scope/Boundaries:** What should it NOT do?
- **Success Criteria:** How do we know it's done?
- **First slice:** What is the smallest shippable thing? (becomes P0)

Format:

```
1. What is the primary goal?
   A. Fix a visible bug
   B. Add a new capability
   C. Improve existing UX
   D. Other: [specify]

2. What should the first PR deliver?
   A. Minimal visible fix only
   B. Backend/data change only
   C. UI + data for one flow
   D. Other: [specify]
```

Users reply "1C, 2A". Indent options.

---

## Step 2: PRD Structure

### 1. Introduction / Overview
Feature and problem in 2–4 sentences.

### 2. Goals
Specific, measurable bullets.

### 3. User Stories
Each story:
- **ID + Title**
- **Description:** "As a [user], I want [feature] so that [benefit]."
- **Acceptance Criteria:** verifiable checklist.
- **Priority:** P0 / P1 / P2.

Each story must fit **one focused agent session** (roughly: one PR, reviewable in under ~15 minutes, touching a small, named set of areas). If not, split it.

```markdown
### US-001: [Title]  — P0
**Description:** As a [user], I want [feature] so that [benefit].

**Acceptance Criteria:**
- [ ] Specific verifiable criterion
- [ ] Typecheck/lint/tests pass (project's own commands)
- [ ] **[UI only]** UI checks below pass
```

### 4. Functional Requirements
Numbered, unambiguous: "FR-1: When X, the system must Y."

### 5. Non-Goals (Out of Scope)
What this will NOT do. Always include:
- No changes outside the slice's named areas.
- No secrets, credentials, personal data, or environment-specific hosts/URLs committed.

### 6. Design Considerations (optional)
UI/UX notes, existing components to reuse, mockups/screenshots.

### 7. Technical Considerations (optional)
Constraints, dependencies, integration points, performance.

### 8. Success Metrics
How success is measured.

### 9. Open Questions
Anything still unresolved. Must not block P0; if it does, ask now.

### 10. Slices
Ordered list. Exactly one P0. Each slice references its user stories and becomes one ticket / one job.

```markdown
| # | Slice | Stories | Priority | Depends on |
|---|-------|---------|----------|------------|
| 1 | Title wraps on job page | US-001 | P0 | — |
| 2 | Live elapsed time | US-002 | P1 | 1 |
```

---

## Acceptance Criteria Rules

Bad: "Works correctly", "Looks good", "Is fast".
Good: "Title longer than 120 chars wraps onto multiple lines; no ellipsis; no horizontal scroll at 375px width."

### UI checks (required for any story with UI changes)

Write checks a headless browser agent or reviewer can run without guessing:

- [ ] **Route + state:** which page/URL and what data state to test (e.g. "job detail page for a job with a 200-char title").
- [ ] **Observable result:** exact element/text/behaviour expected (visible, wraps, updates every second, link opens X).
- [ ] **Viewports:** at least desktop (1280px) and mobile (375px).
- [ ] **No regressions:** no new console errors; no horizontal overflow; existing primary actions still visible.
- [ ] **Evidence:** before/after screenshots attached to the PR.

Use whatever browser-verification tool the environment provides (Playwright, a dev-browser skill, etc.). Name the tool only if the project already uses one.

### Code checks (every story)
- [ ] Project's typecheck/lint/test commands pass (discover from repo; don't invent).
- [ ] New behaviour has a test where the project has tests for that layer.

---

## Spec-in, PR-out Ticket Shape

Each slice becomes a ticket an agent can execute with no extra context:

```markdown
# [Slice title]

**Priority:** P0
**Repo / area:** [repo, paths or components likely touched]
**Context:** 2–3 sentences: problem + link to the parent PRD.

## Goal
One sentence: what the PR delivers.

## Acceptance Criteria
- [ ] ...
- [ ] UI checks (route, observable result, viewports, no regressions, screenshots)
- [ ] Typecheck/lint/tests pass

## Non-Goals
- ...

## Definition of Done
- Branch + PR opened (draft is fine) referencing this ticket.
- PR description lists each acceptance criterion with ✅ and evidence.
- Ticket updated with the PR link (and preview URL if the project has one).
```

---

## Tracker Adapters (plug-in)

The tracker is configuration, not part of this skill. Resolve it in this order:

1. Explicit instruction from the user ("put it in Linear").
2. Project config: `.prd.json` / `.prd.yml` at repo root, or a `prd` section in an existing agent config.
3. Environment: `PRD_TRACKER` = `plane` | `linear` | `github` | `markdown`.
4. Default: `markdown`.

Credentials and hosts come only from env/config (e.g. `PLANE_BASE_URL`, `PLANE_API_TOKEN`, `PLANE_WORKSPACE`, `PLANE_PROJECT`; `LINEAR_API_KEY`, `LINEAR_TEAM`; `GITHUB_TOKEN`, `GITHUB_REPO`). Never write them into the PRD, ticket, or repo.

Field mapping:

| PRD field | Plane | Linear | GitHub Issues | Markdown |
|-----------|-------|--------|---------------|----------|
| Slice title | name | title | title | `# heading` |
| Ticket body | description (md/html) | description (md) | body (md) | file body |
| P0/P1/P2 | priority urgent/high/medium | priority 1/2/3 | label `P0`/`P1`/`P2` | `**Priority:**` |
| Parent PRD | parent issue or module | project / parent issue | tracking issue + task list | link to PRD file |
| Depends on | relation "blocked by" | relation "blocked by" | "Blocked by #n" in body | table column |

If the tracker tool/API isn't available, fall back to markdown and say so. Creating tickets is an external action: show the tickets first and create them only when the user approves.

---

## Writing for Agents and Junior Developers

- Be explicit; define jargon.
- Name files, routes, components when known; say "likely" when guessing.
- Number everything for reference.
- Use concrete examples and edge cases (long text, empty state, error state).
- Keep each ticket self-contained; no "see chat".
- Keep it generic: no personal names, hostnames, internal infra, or secrets in the PRD or tickets unless the user explicitly provides them for this project.

---

## Output

- PRD: `tasks/prd-[feature-name].md` (kebab-case), or the path the project config specifies.
- Tickets: one per slice, in the configured tracker's shape, or `tasks/[feature-name]/NN-[slice].md` for markdown.
- Reply with: PRD path, slice table, and the P0 ticket in full.

---

## Example (abridged)

```markdown
# PRD: Readable Job Titles

## Introduction
Job titles on the job list and detail pages are truncated with an ellipsis, so users can't tell jobs apart.

## Goals
- Full job title is readable on list and detail pages at all widths.

## User Stories
### US-001: Wrap job title on detail page — P0
**Description:** As a user, I want to see the whole job title so I know which job I'm looking at.

**Acceptance Criteria:**
- [ ] On the job detail page for a job with a 200-character title, the full title is visible and wraps; no ellipsis.
- [ ] At 375px and 1280px, no horizontal scroll; the Delete action stays visible and secondary.
- [ ] No new console errors.
- [ ] Before/after screenshots in the PR.
- [ ] Typecheck/lint/tests pass.

### US-002: Wrap job title in job list — P1
...

## Functional Requirements
- FR-1: Job title must wrap instead of truncating on the detail page.
- FR-2: Job list rows show at least two lines of title before clamping, with full title on hover/focus.

## Non-Goals
- No change to how titles are generated.
- No redesign of other page sections.
- No secrets or environment-specific config committed.

## Slices
| # | Slice | Stories | Priority | Depends on |
|---|-------|---------|----------|------------|
| 1 | Wrap title on detail page | US-001 | P0 | — |
| 2 | Wrap/clamp title in list | US-002 | P1 | 1 |

## Open Questions
- Should very long titles clamp at N lines in the list?
```

---

## Checklist

Before saving:

- [ ] Asked clarifying questions with lettered options (or confirmed none needed)
- [ ] Stories are small; each fits one agent session / one PR
- [ ] Exactly one P0 slice
- [ ] Every UI story has route, observable result, viewports, no-regression, and screenshot checks
- [ ] FRs numbered and unambiguous
- [ ] Non-goals set clear boundaries (incl. no secrets/env-specific config)
- [ ] No tracker, host, or personal specifics hard-coded; tracker resolved from config/env
- [ ] Tickets shown for approval before any are created in a tracker
- [ ] Saved PRD to `tasks/prd-[feature-name].md` (or configured path)
