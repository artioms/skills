---
name: next
description: Find the next meaningful action in a GitHub issue backlog and recommend the appropriate AI Hero skill. Use for /next, /next bugs, /next triage, /next ready, or choosing what to work on next.
metadata:
  version: "1.5.0"
---

# Next

Find what can move the project toward its goal, then recommend **one next action and the AI Hero skill to use**. Stop after the recommendation. This skill navigates and advises; it does not do the work.

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

For `/next` or a broad scope, show short, individually **numbered** choices from the actual hierarchy. Let the user drill down through initiatives, workstreams, and feature groups, or choose the entire current scope. Keep a breadcrumb and numbered Back option. At the root, call the repository-wide choice **Entire repository — find work across all initiatives**.

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

### 4. Recommend and stop

Show **up to five numbered next actions**, not a menu of empty workflow queues and not the same tickets twice. For each, give the linked issue title, what moves forward, why it matters, any blocker or claim, and the suggested AI Hero skill. Let the user select an action when there is more than one credible choice.

For the selected action, recheck the issue's relevant status and give a compact final handoff:

- **Next action:** issue title and link, or the named human gate.
- **Why now:** the business outcome or dependency this advances.
- **Use:** the appropriate existing AI Hero skill, such as `/grill-with-docs`, `/to-spec`, `/to-tickets`, `/triage`, `/implement`, or `/code-review`.
- **Run:** a copyable invocation with the issue link and concise context.
- **Expected result:** the artifact or decision that should exist after that skill finishes.

Recommend only skills that are actually available; if a needed skill is not installed, say so and name the required workflow without pretending it can run. For human validation, recommend the human acceptance process rather than mislabeling it as an AI Hero skill.

**Stop here.** Do not invoke the recommended skill, start an interview, write a spec, create tickets, implement, review code, or perform human acceptance. The user decides when to continue.

## Boundaries

- `/next` is **read-only**. Do not create, assign, claim, comment on, close, relabel, reparent, or change dependencies of GitHub issues or PRs.
- Follow the repository's hierarchy, label vocabulary, acceptance rules, and relevant ADRs. Keep repository-specific IDs and policies out of this generic skill.
- Prefer concise, business-oriented wording and linked issue **names**, not walls of issue numbers.
- Never claim a queue is empty or work is unblocked unless the relevant open candidates and dependencies have been verified. Explain partial coverage plainly.
- The outcome of `/next` is **a recommendation, not execution**.
