---
name: next
description: Find the next meaningful action in a GitHub issue backlog and recommend the appropriate AI Hero skill. Use for /next, /next bugs, /next triage, /next ready, or choosing what to work on next.
metadata:
  version: "1.5.2"
---

# Next

Find what can move the project toward its goal, then present **up to ten ranked next actions and the appropriate AI Hero skill for each**. Stop after presenting the list. This skill navigates and advises; it does not do the work.

## Invocation

- `/next` — choose a goal or scope through the repository's issue hierarchy.
- `/next <issue-number>` — start at that issue, or its descendants if it groups work.
- `/next bugs` — find open bugs needing a next action.
- `/next triage` — find incoming issues that still need triage.
- `/next ready` — find unclaimed, unblocked work ready for agent delivery.
- `/next --version` — report `metadata.version` and stop without querying GitHub.

These are starting points, **not a filter language**. Do not introduce combined expressions or extra parameters. A user can narrow the scope conversationally. With no scope supplied, start from the repository's documented roots; do not infer the active workstream from the branch or previous sessions.

## Process

### 1. Orient

Resolve the current repository. Read its issue-tracker conventions and agent instructions (`AGENTS.md`, tracker and triage docs where present). Identify the project's documented goals and root issue maps. Do not invent a goal, priority, or ownership rule.

Use the installed native GitHub CLI (`gh issue`, `gh pr`, `gh api`, including pagination). Query narrowly: scope, state, labels, and required fields first; expand only to resolve missing evidence. Prefer `gh issue list --state open` with appropriate `--label`, `--search`, `--json` and `--limit`, `gh issue view --json` for selected issues, and native `gh api` sub-issue/dependency endpoints for relationships. Use `--jq` or `jq` for processing; avoid Python for routine GitHub queries or hierarchy traversal. Paginate when needed, never mistake a limited result page for full coverage, and reuse results within the session. If the tracker is not GitHub, follow its documented interface instead of pretending `gh` applies.

**Done when:** the repository, goal map, and applicable tracker rules are clear, or limitations are stated.

### 2. Navigate

For `/next` or a broad scope, show short, individually **numbered** choices from the actual hierarchy. Let the user drill down through initiatives, workstreams, and feature groups, or choose the entire current scope. Keep a breadcrumb. Offer a numbered Back option only when there is a parent navigation level; never show Back at repository roots. At the root, call the repository-wide choice **Entire repository — find work across all initiatives**.

Navigate native parent/child relationships, including closed records only as needed to establish the initial map, verify ancestry, or recover historical decisions. After orientation, discover candidate work from **open issues only**. Inspect closed records individually when necessary for context or dependency verification; avoid repeated full-history scans. Closed issues are never pickup candidates. Parentage is not a blocking dependency. Where hierarchy is unavailable, label any documented grouping as a view, not as a native parent relationship.

Use lightweight, targeted discovery during navigation. Closed issues are historical context, not the candidate inventory. A parent must remain open while any native descendant is open; all descendants being closed is necessary but not sufficient for parent closure (the parent's own acceptance still applies). Treat a closed parent with an open descendant as a hierarchy-governance violation: report it, retain visibility of the open work, and do not silently normalize it. Do not modify the tracker. Distinguish immediate-child counts from descendant counts, and disclose incomplete traversal.

**Done when:** the user has chosen a scope, issue, or shortcut search.

### 3. Find what can move forward

Evaluate **open actionable issues only** in the chosen scope. Query open issues first, then verify native ancestry as needed; do not re-fetch closed history for every recommendation. Check their current state, relevant specifications and acceptance criteria, assignments or claims, and **native open blockers**. Inspect linked PRs and comments for shortlisted candidates when they affect the next action; avoid bulk comment retrieval. Verify uncertain facts rather than treating missing data as proof of readiness. Keep related issues outside the selected hierarchy visible as dependencies, not invented descendants.

Look for the next meaningful gate toward the selected goal:

- **Clarify:** a decision, requirement, or domain question is unresolved.
- **Specify:** the intended outcome is known but lacks a usable specification.
- **Break down:** an approved specification needs agent-sized, dependency-aware tickets.
- **Triage:** an incoming bug or request has not been assessed or routed.
- **Implement:** a sufficiently specified ticket is open, unblocked, and unclaimed.
- **Review:** a real PR or change awaits review.
- **Validate:** human acceptance, evidence, or sign-off is the next gate.

Recommend **main-flow AI Hero skills** as the handoff: `/grill-with-docs` for unresolved decisions, `/to-spec` for a missing specification, `/to-tickets` for breakdown, `/triage` for unprocessed incoming issues, `/implement` for ready delivery, and `/code-review` for reviewable changes. Specialized techniques such as `/research`, `/tdd`, and `/diagnosing-bugs` belong inside the subsequent workflow, not in `/next`'s recommendation. Verify available skill names before presenting them.

Do not automatically send tickets produced by `/to-tickets` back through `/triage`. Assigned work is not available for new pickup. A blocked issue may still have a useful preparation action, but do not recommend implementing it before its blockers clear.

Prefer actions that unblock other work or advance an explicit business outcome. Do not fabricate priority. If nothing is ready for an agent, say what prevents progress and identify the next human or clarification action. If coverage is incomplete, label conclusions provisional.

**Done when:** a short, evidence-based set of next actions is identified, or the reason none can be verified is clear.

### 4. Rank, recommend, and stop

Present **up to ten distinct, numbered next actions**, ordered from most useful to least useful for advancing the selected goal. Show fewer than ten if fewer credible actions exist; never pad with unrelated, closed, or invented tickets. Do not stop after recommending only the highest-ranked issue when other credible actions exist.

Rank by the evidence available: (1) work that unblocks other progress, (2) direct contribution to the selected business outcome, (3) whether the next step can be taken now, (4) unclaimed ownership for agent pickup, and (5) explicit repository priority. This is judgment, not a fabricated numeric score. Do not rank blocked implementation as ready; recommend its actionable prerequisite instead. Assigned work can be mentioned only when its next step is a legitimate coordination, review, or human action, clearly marked as claimed.

Use a compact, scannable format for each item:

`1. [Issue title (#123)](issue-url) — /skill-name`

`   Why: One sentence explaining what this advances or unblocks.`

For human acceptance or other work without a suitable main-flow skill, write **Human action** instead of inventing a skill. Do not include `Run:` sections, generated invocation prompts, code fences, or expected-result paragraphs. Avoid repeating the same issue in multiple categories.

Briefly state the selected scope and coverage, including any uncertainty. The list itself is the final deliverable: **do not require the user to choose an item for a second recommendation**, and do not invoke any skill. The user chooses which skill to run next.

**Stop here.** Do not conduct interviews, specify, create tickets, implement, review, validate, or modify the tracker.

## Boundaries

- `/next` is **read-only**. Do not create, assign, claim, comment on, close, relabel, reparent, or change dependencies of GitHub issues or PRs.
- Follow the repository's hierarchy, label vocabulary, acceptance rules, and relevant ADRs. Keep repository-specific IDs and policies out of this generic skill.
- Prefer concise, business-oriented wording and linked issue **names**, not walls of issue numbers.
- Never claim a queue is empty or work is unblocked unless the relevant open candidates and dependencies have been verified. A closed parent with an open descendant is a reported governance exception, not permission to omit that descendant. Explain partial coverage plainly.
- The outcome of `/next` is **a recommendation, not execution**.
