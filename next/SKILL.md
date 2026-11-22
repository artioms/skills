---
name: next
description: Find the next meaningful action in a GitHub issue backlog and recommend the appropriate AI Hero skill. Use for /next, /next bugs, /next triage, /next ready, or choosing what to work on next.
metadata:
  version: "1.5.1"
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

Use the installed native GitHub CLI (`gh issue`, `gh pr`, `gh api`, including pagination) for queries. Use `--json`, `--jq`, or `jq` to process results. Do not use Python for routine GitHub queries or hierarchy traversal. If the tracker is not GitHub, follow its documented interface instead of pretending `gh` applies.

**Done when:** the repository, goal map, and applicable tracker rules are clear, or limitations are stated.

### 2. Navigate

For `/next` or a broad scope, show short, individually **numbered** choices from the actual hierarchy. Let the user drill down through initiatives, workstreams, and feature groups, or choose the entire current scope. Keep a breadcrumb. Offer a numbered Back option only when there is a parent navigation level; never show Back at repository roots. At the root, call the repository-wide choice **Entire repository — find work across all initiatives**.

Traverse native parent/child relationships to find open descendants, including those under closed intermediate issues. Closed issues provide structure, progress, and relevant decisions; they are never pickup candidates. Parentage is not a blocking dependency. Where hierarchy is unavailable, label any documented grouping as a view, not as a native parent relationship.

Use lightweight discovery during navigation; do not fetch every closed issue's comments or perform readiness audits before a scope is selected. Distinguish immediate-child counts from descendant counts, and disclose incomplete traversal.

**Done when:** the user has chosen a scope, issue, or shortcut search.

### 3. Find what can move forward

Evaluate the **open actionable issues** in the chosen scope. Check their current state, relevant specifications and acceptance criteria, assignments or claims, and **native open blockers**. Inspect linked PRs and comments when they affect the next action. Verify uncertain facts rather than treating missing data as proof of readiness. Keep related issues outside the selected hierarchy visible as dependencies, not invented descendants.

Look for the next meaningful gate toward the selected goal:

- **Clarify:** a decision, requirement, or domain question is unresolved.
- **Specify:** the intended outcome is known but lacks a usable specification.
- **Break down:** an approved specification needs agent-sized, dependency-aware tickets.
- **Triage:** an incoming bug or request has not been assessed or routed.
- **Implement:** a sufficiently specified ticket is open, unblocked, and unclaimed.
- **Review:** a real PR or change awaits review.
- **Validate:** human acceptance, evidence, or sign-off is the next gate.

Do not automatically send tickets produced by `/to-tickets` back through `/triage`. Assigned work is not available for new pickup. A blocked issue may still have a useful preparation action, but do not recommend implementing it before its blockers clear.

Prefer actions that unblock other work or advance an explicit business outcome. Do not fabricate priority. If nothing is ready for an agent, say what prevents progress and identify the next human or clarification action. If coverage is incomplete, label conclusions provisional.

**Done when:** a short, evidence-based set of next actions is identified, or the reason none can be verified is clear.

### 4. Rank, recommend, and stop

Present **up to ten distinct, numbered next actions**, ordered from most useful to least useful for advancing the selected goal. Show fewer than ten if fewer credible actions exist; never pad with unrelated, closed, or invented tickets. Do not stop after recommending only the highest-ranked issue when other credible actions exist.

Rank by the evidence available: (1) work that unblocks other progress, (2) direct contribution to the selected business outcome, (3) whether the next step can be taken now, (4) unclaimed ownership for agent pickup, and (5) explicit repository priority. This is judgment, not a fabricated numeric score. Do not rank blocked implementation as ready; recommend its actionable prerequisite instead. Assigned work can be mentioned only when its next step is a legitimate coordination, review, or human action, clearly marked as claimed.

Use a compact, scannable format for each item:

`1. [Issue title (#123)](issue-url) — /skill-name`

`   Why: One sentence explaining what this advances or unblocks.`

For human acceptance or other work with no suitable installed AI Hero skill, write **Human action** instead of inventing a skill. Verify that named skills exist in the environment. Do not include `Run:` sections, generated invocation prompts, code fences, or expected-result paragraphs. Avoid repeating the same issue in multiple categories.

Briefly state the selected scope and coverage, including any uncertainty. The list itself is the final deliverable: **do not require the user to choose an item for a second recommendation**, and do not invoke any skill. The user chooses which skill to run next.

**Stop here.** Do not conduct interviews, specify, create tickets, implement, review, validate, or modify the tracker.

## Boundaries

- `/next` is **read-only**. Do not create, assign, claim, comment on, close, relabel, reparent, or change dependencies of GitHub issues or PRs.
- Follow the repository's hierarchy, label vocabulary, acceptance rules, and relevant ADRs. Keep repository-specific IDs and policies out of this generic skill.
- Prefer concise, business-oriented wording and linked issue **names**, not walls of issue numbers.
- Never claim a queue is empty or work is unblocked unless the relevant open candidates and dependencies have been verified. Explain partial coverage plainly.
- The outcome of `/next` is **a recommendation, not execution**.
