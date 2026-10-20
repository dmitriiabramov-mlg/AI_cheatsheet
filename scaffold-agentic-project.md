---
name: scaffold-agentic-project
description: Scaffold a new project with the agent-ready baseline - AGENTS.md/CLAUDE.md with brutally-honest system prompt, ADR process with machine-readable Relations, spec-driven development rules, GitHub PR templates (implementation + spec change), .gitignore, and Claude Code permission deny for gh pr merge. Use when the user asks to bootstrap, scaffold, initialize, or set up a new project/repo with "the usual template", "my project template", ADR + specs + PR templates, or runs /scaffold-agentic-project.
---

# Scaffold agentic project

Creates the baseline structure in the **current working directory** (or a path the
user gives).

## Procedure

1. **Check the target.** List the directory. If any file from the manifest below
   already exists, STOP and show the conflicts to the user. Do not overwrite
   anything without explicit confirmation per file.
2. **Parameters.**
   - `{{DATE}}`: today's date, `YYYY-MM-DD`.
   - Maintainer mode: default **solo**. If the user says the project has several
     maintainers, apply the "Team variant" edits at the end.
3. **Git.** If the directory is not a git repo, run `git init -b main`.
   Do not add a remote.
4. **Write every file** from the manifest exactly as given, replacing `{{DATE}}`.
   Inner fences in this skill use `~~~`. Do not copy the fence lines themselves.
5. **Do NOT commit.** Show `git status --short -uall` and propose this commit
   message. Commit only if the user explicitly asks:
   ~~~
   Initial project setup: agent instructions, ADR and spec process, PR templates

   - AGENTS.md / CLAUDE.md: agent rules and project system prompt
   - docs/adr: ADR template, ADR-0001 (use ADRs), ADR-0002 (spec-driven dev)
   - specs/_template.md: feature spec template with stable requirement IDs
   - .github: implementation and spec-change PR templates
   - .claude/settings.json: deny gh pr merge for agents
   - .gitignore
   ~~~
6. **Report next steps** to the user, briefly:
   - Create the GitHub remote, then enable branch protection on `main`
     ("Require a pull request before merging", no required approvals for solo).
     Without it, agents can push to `main` directly and the PR rules mean nothing.
   - Write ADR-0003: project purpose and tech stack. Every later ADR needs it
     as a base.

## Manifest

### `CLAUDE.md`
~~~markdown
# System prompt

I want you to be brutally honest, straightforward, and logical. Challenge my assumptions, question my reasoning, and call out any flaws, contradictions, or unrealistic ideas you notice. Don't soften the truth or sugarcoat anything to protect my feelings, I care more about growth and accuracy than comfort. Avoid empty praise, generic motivation, or vague advice. I want hard facts, clear reasoning, and actionable feedback. Think and respond like a no-nonsense coach or a brutally honest friend who's focused on making me better, not making me feel better. Push back whenever necessary, and never feed me bullshit. Stick to this approach for our entire conversation, regardless of the topic

# Project instructions

@AGENTS.md
~~~

### `AGENTS.md`
~~~markdown
# AGENTS.md

Instructions for AI agents (Claude Code, Codex, Cursor, etc.) working in this repository.

## Communication style

Be brutally honest, straightforward, and logical. Challenge assumptions, question
reasoning, call out flaws, contradictions, and unrealistic ideas. No empty praise,
no sugarcoating, no vague advice — hard facts, clear reasoning, actionable feedback.
Push back whenever necessary.

## Spec-driven development

Specs in `specs/` are the source of truth for behavior. Code follows specs, never
the other way around. See ADR-0002.

### Layout

- One spec per feature: `specs/<feature-slug>.md`, copied from `specs/_template.md`.
- Each requirement has a stable ID `REQ-<FEATURE>-NNN`. IDs are never reused or
  renumbered. Removed requirements are marked `Status: Removed`, not deleted.
- Spec = *what* the system does (observable behavior, acceptance criteria).
  ADR = *how/why* it is built. Don't put implementation details in specs.

### Spec lifecycle

`Draft` → `Approved` → `Implemented` → `Deprecated`

- Only `Approved` specs may be implemented.
- A spec with blocking open questions cannot be `Approved`.
- Status transitions are spec changes and go through a spec PR.

### Hard rules for agents

1. **Never modify `specs/` in an implementation PR.** Not even "small fixes".
2. **If the spec must change** (wrong, incomplete, contradicts reality, or the user
   asks for new behavior): stop, explicitly tell the user what you want to change,
   why, and what it affects. Then open a **separate spec PR** using
   `.github/PULL_REQUEST_TEMPLATE/spec-change.md`. Implementation of changed
   behavior waits until the spec PR is merged.
3. **Ambiguity is a blocker, not a judgment call.** If a requirement can be read
   more than one way, ask. Do not pick an interpretation silently.
4. **No scope creep.** Do not implement behavior not covered by a requirement.
   Items under `Non-goals / Out of scope` are forbidden.
5. **Traceability.** Every implementation PR lists the requirement IDs it covers.
   Tests reference the requirement ID they verify (in the test name or a comment).
6. **Code vs spec mismatch.** If code disagrees with the spec, the code is the bug
   — fix it in an implementation PR. If you believe the spec is the bug, rule 2.
7. **Spec vs ADR conflict.** Flag it to the user. Do not resolve it in either
   direction on your own.
8. **Agents never merge PRs.** The project has a single maintainer (the owner);
   the owner's merge of a spec PR *is* the approval. `gh pr merge` is denied in
   `.claude/settings.json`.

Changes without behavior impact (refactoring, tooling, CI, docs) don't need a spec,
but must not change observable behavior.

## Pull requests

Every PR uses a template from `.github/`:

| PR type | Template | How |
|---|---|---|
| Implementation, fix, refactoring, infra, docs | `.github/pull_request_template.md` | applied by default |
| Spec change | `.github/PULL_REQUEST_TEMPLATE/spec-change.md` | `gh pr create --template spec-change.md` |

- Fill every section. Write `none` explicitly instead of deleting required sections.
- A PR is either a spec PR or an implementation PR — never both.
- One logical change per PR. Split large PRs.
- Do not commit or open PRs unless the user explicitly asks.

## Architecture Decision Records (ADR)

Every significant architectural decision MUST be recorded as an ADR in `docs/adr/`.

### What counts as "significant"

Write an ADR when a decision:
- picks a technology, framework, library, or external service;
- defines a data model, API contract, or integration boundary;
- affects security, secrets handling, deployment, or infrastructure;
- is expensive to reverse, or constrains future decisions.

Do not write ADRs for trivial, easily reversible choices (variable names, formatting).

### Format

- File name: `docs/adr/NNNN-short-kebab-title.md` (sequential, zero-padded, never reused).
- Copy `docs/adr/0000-template.md`.
- ADRs are immutable once `Accepted`. To change a decision, write a new ADR that
  `Supersedes` the old one and set the old one's status to `Superseded by NNNN`.
  Only the status line of an old ADR may be edited.

### Relations (mandatory)

Every ADR has a `Relations` section with machine-readable links to other ADRs:

- `Depends-on: NNNN` — this decision assumes NNNN holds.
- `Supersedes: NNNN` — this decision replaces NNNN.
- `Constrains: NNNN` — this decision limits options in NNNN.
- `Related: NNNN` — informational link only.

Use `none` if there are no relations. These links form the dependency graph of
the architecture — keep them accurate.

### Agent duties

- **Before proposing a change** that touches an area covered by an ADR, read
  `docs/adr/`, build the dependency graph from `Relations`, and explicitly list
  every ADR (and the components it describes) that would break or need revision.
  Do not silently contradict an accepted ADR.
- **When making a significant decision**, draft the ADR in the same change.
- **When superseding**, walk all ADRs that `Depends-on` the superseded one
  transitively and flag each as needing review.

### Decision trees

No separate decision-tree artifacts. Alternatives and the reasoning for rejecting
them go into the ADR's `Considered options` section. If a decision genuinely has
multi-level branching (choice A forces sub-choices B/C), add a Mermaid `flowchart`
inline in that ADR — not as a standalone file.
~~~

### `.gitignore`
~~~gitignore
# Local environment files
.env
.env.*

# IDE
.idea/workspace.xml
.idea/shelf/

# OS
.DS_Store
~~~

### `.claude/settings.json`
~~~json
{
  "permissions": {
    "deny": [
      "Bash(gh pr merge:*)"
    ]
  }
}
~~~

### `docs/adr/0000-template.md`
~~~markdown
# NNNN. Title

- Status: Proposed | Accepted | Deprecated | Superseded by NNNN
- Date: YYYY-MM-DD

## Relations

- Depends-on: none
- Supersedes: none
- Constrains: none
- Related: none

## Context

What problem are we solving? What forces and constraints are in play?

## Considered options

1. Option A — pros / cons
2. Option B — pros / cons

## Decision

What we chose and why the other options lost.

## Consequences

What becomes easier, what becomes harder, what we must now live with.
Which components are affected.
~~~

### `docs/adr/0001-record-architecture-decisions.md`
~~~markdown
# 0001. Record architecture decisions

- Status: Accepted
- Date: {{DATE}}

## Relations

- Depends-on: none
- Supersedes: none
- Constrains: none
- Related: none

## Context

Architectural decisions need to be traceable, and AI agents need a structured
source to detect which decisions break when an earlier one changes.

## Considered options

1. ADRs with explicit `Relations` links — lightweight, graph-friendly.
2. ADRs plus standalone decision trees — more artifacts, duplicates the
   "Considered options" section, drifts out of sync.
3. No formal records — knowledge lives in chat logs and heads.

## Decision

Option 1. Decision trees are allowed only as inline Mermaid diagrams inside an
ADR when the branching is genuinely multi-level.

## Consequences

Every significant decision costs a short markdown file. Agents can build a
dependency graph from `Relations` and flag impacted ADRs on change.
~~~

### `docs/adr/0002-spec-driven-development.md`
~~~markdown
# 0002. Spec-driven development

- Status: Accepted
- Date: {{DATE}}

## Relations

- Depends-on: none
- Supersedes: none
- Constrains: none
- Related: 0001

## Context

Most code is written by AI agents. Without a fixed source of truth, agents
interpret requirements on the fly, drift in scope, and silently "fix" requirements
to match whatever they implemented.

## Considered options

1. Specs in `specs/` as the source of truth, changed only via dedicated spec PRs
   with human approval; code PRs reference requirement IDs.
2. Specs allowed to change together with code in the same PR — less ceremony,
   but the reviewer cannot tell whether behavior changed intentionally, and agents
   can rewrite the spec to match their code.
3. No specs, requirements live in issues/chat — no traceability.

## Decision

Option 1. Spec PRs and implementation PRs are separate. Each requirement has a
stable ID; implementation PRs and tests reference those IDs.

## Consequences

- Behavior changes cost an extra PR and a human approval round.
- Agents must stop and report when the spec is wrong or ambiguous instead of
  guessing.
- Single-maintainer project: GitHub does not allow approving your own PR, so
  approval = the owner's merge. Agents are denied `gh pr merge`.
- Rules in AGENTS.md are not enforcement. Until CI checks and CODEOWNERS exist,
  the separation relies on agent discipline and review.
~~~

### `specs/_template.md`
~~~markdown
# Spec: Feature title

- Status: Draft | Approved | Implemented | Deprecated
- Feature ID: <FEATURE> <!-- short uppercase slug used in requirement IDs, e.g. AUTH -->
- Owner: <!-- person responsible for approving changes -->
- Related ADRs: <!-- NNNN, or "none" -->
- Related specs: <!-- specs/<other>.md, or "none" -->

## Problem

<!-- What user/business problem this solves. Why it matters. -->

## Goals

-

## Non-goals / Out of scope

<!-- Explicitly what this spec does NOT cover. Agents must not implement these. -->

-

## Requirements

<!--
Each requirement:
- has a stable ID REQ-<FEATURE>-NNN (never reused or renumbered);
- describes observable behavior (what), not implementation (how);
- has acceptance criteria that can be turned into a test.
Status per requirement: Active | Removed
-->

### REQ-<FEATURE>-001: Short title

- Status: Active
- Description: <!-- The system MUST/SHOULD/MAY ... -->
- Acceptance criteria:
  - Given <!-- precondition -->, when <!-- action -->, then <!-- observable result -->

## Constraints

<!-- Non-functional requirements: performance, security, compliance, limits. Give numbers. -->

-

## Open questions

<!-- Unresolved items. A spec with blocking open questions cannot be Approved. -->

-

## Changelog

| Date | PR | Change |
|------|----|--------|
| YYYY-MM-DD | # | Initial draft |
~~~

### `.github/pull_request_template.md`
~~~markdown
<!--
Implementation PR template (default).
Spec changes MUST NOT go here — use .github/PULL_REQUEST_TEMPLATE/spec-change.md
(`gh pr create --template spec-change.md`).
Delete comment blocks before submitting. Sections marked (optional) may be removed if empty.
-->

## Summary

<!-- 1–3 sentences: what this PR does and why. No marketing. -->

## Spec traceability

- Spec: <!-- specs/<feature>.md, or "none — not a behavior change" -->
- Requirements implemented: <!-- REQ-<FEATURE>-001, REQ-<FEATURE>-002 -->
- Requirements partially implemented / deferred: <!-- IDs + reason, or "none" -->

## Type of change

- [ ] Feature (implements spec requirements)
- [ ] Bug fix (code did not match spec)
- [ ] Refactoring (no behavior change)
- [ ] Infrastructure / tooling / CI
- [ ] Documentation
- [ ] Other: <!-- describe -->

## Changes

<!-- Bullet list of meaningful changes. Group by component if large. -->

-

## Architecture

- ADRs affected: <!-- NNNN, or "none" -->
- New ADR in this PR: <!-- NNNN, or "none" -->
- Conflicts with accepted ADRs: <!-- list, or "none" -->

## Testing

<!-- How this was verified. Name the tests and which REQ IDs they cover. -->

- [ ] Automated tests added/updated
- [ ] Tests pass locally
- [ ] Manually verified: <!-- how, or remove -->

## Risks and rollback

<!-- What can break, blast radius, how to roll back. "Low risk" without reasoning is not an answer. -->

## Security and secrets

- [ ] No secrets, tokens, or credentials in code, config, logs, or tests
- [ ] No new external data exposure (or described above)

## Checklist

- [ ] PR does **not** modify files in `specs/`
- [ ] No behavior beyond what the spec requires (no scope creep)
- [ ] Spec ambiguities found during implementation were raised, not silently resolved
- [ ] PR is small enough to review in one sitting; otherwise split
- [ ] Self-reviewed the diff

## Open questions (optional)

<!-- Anything the reviewer must decide. -->
~~~

### `.github/PULL_REQUEST_TEMPLATE/spec-change.md`
~~~markdown
<!--
Spec change PR template.
Use: `gh pr create --template spec-change.md`
     or append `?template=spec-change.md` to the GitHub compare URL.
This PR contains ONLY changes under specs/. No code, no tests.
Delete comment blocks before submitting.
-->

## Summary

<!-- What changes in the spec, in 1–3 sentences. -->

## Motivation

<!-- Why the current spec is wrong, incomplete, or outdated.
     Be concrete: what was discovered, by whom, in which PR/issue/conversation. -->

- Triggered by: <!-- PR #, issue #, user request, implementation finding -->

## Change classification

- [ ] New spec (Draft → Approved)
- [ ] New requirement(s) added
- [ ] Existing requirement changed — **breaking** (implemented behavior must change)
- [ ] Existing requirement changed — non-breaking (clarification, no behavior change)
- [ ] Requirement removed (marked `Removed`, ID not reused)
- [ ] Editorial only (typos, formatting)

## Requirements diff

<!-- One row per touched requirement. -->

| ID | Change | Before | After |
|----|--------|--------|-------|
| REQ-<FEATURE>-001 | added / changed / removed | | |

## Impact analysis

- Implemented code affected: <!-- modules/files, or "none — not yet implemented" -->
- Tests that will need to change: <!-- list, or "none" -->
- Other specs affected: <!-- list, or "none" -->
- ADRs affected / contradicted: <!-- NNNN, or "none". If an ADR must change, that is a separate ADR PR. -->
- Blocked implementation PRs: <!-- PRs waiting on this, or "none" -->

## Alternatives considered

<!-- What else could have been changed instead, and why not. "None" requires justification. -->

## Checklist

- [ ] PR touches **only** files under `specs/`
- [ ] Every new/changed requirement has a stable ID and testable acceptance criteria
- [ ] No requirement IDs reused or renumbered
- [ ] Removed requirements are marked `Removed`, not deleted
- [ ] Spec `Changelog` section updated
- [ ] Spec `Status` is correct
- [ ] Out-of-scope section still accurate
- [ ] No implementation details leaked into the spec (the spec says *what*, not *how*)

## Approval

<!-- Single-maintainer project: the owner's merge is the approval. Agents never merge. -->

- [ ] Owner has read the full requirements diff and impact analysis before merging
~~~

## Team variant (only if the user says there are multiple maintainers)

- `AGENTS.md` rule 8 → "**Agents never approve or merge PRs.** Spec PRs require
  approval from a maintainer other than the author. `gh pr merge` is denied in
  `.claude/settings.json`."
- ADR-0002: replace the "Single-maintainer project…" bullet with "Spec PRs
  require at least one approving review from a maintainer; enforced via branch
  protection and CODEOWNERS on `specs/`."
- `spec-change.md` Approval section → `- [ ] Approved by a maintainer other than the author`.
- Suggest adding `.github/CODEOWNERS` with `/specs/ @<maintainers>` (ask for handles).
