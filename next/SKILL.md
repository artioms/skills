---
name: next
description: Use when the user wants to find the next GitHub issue to move forward, explore a backlog, or choose an issue for an existing workflow.
metadata:
  version: "1.4.0"
---

# Next

Find the next issue worth moving forward. The issue tracker is the **map**; the **frontier** is open work whose next step can be taken. Navigate the map, identify the frontier, and hand the chosen issue to the right existing skill. `/next` finds work; it does not do the work.

## Start

- `/next` — explore the repository, starting at its documented root scopes.
- `/next <number>` — explore that issue and its descendants, or assess the issue itself if it has no children.
- `/next bugs` — find open bugs and group them by their next action.
- `/next triage` — find incoming issues that need triage.
- `/next ready` — find unclaimed, unblocked issues ready for implementation.
- `/next --version` — print `metadata.version` from this loaded skill and stop.

Shortcuts search the whole repository unless a scope is explicitly supplied. Continue within the scope already selected in this conversation when the user's reply is an unambiguous menu choice. For unknown arguments, explain the supported starts and ask what they want; do not silently interpret them as filters.

## 1. Orient

Identify the repository from the current Git remote or the user's explicit reference. Read its `AGENTS.md` and issue-tracker/triage conventions, if present. The repository defines roots, labels, ownership, acceptance, and dependencies; this skill defines none of them.

Use the installed **GitHub CLI** (`gh issue`, `gh pr`, `gh api`) for discovery, native sub-issues, and blockers. Use pagination and `--json`/`--jq` or `jq` to process results. Check command help when needed. Prefer these native interfaces to Python scripts for routine GitHub queries.

Find open and closed issues as needed to reconstruct the map. A complete issue inventory is not proof that every parent/child relationship has been traversed.

**Done when:** the repository, its issue conventions, and the starting scope are established, or any uncertainty is stated.

## 2. Navigate the map

If the user did not choose a specific issue or shortcut, show a short **numbered** menu of real root scopes. Let them drill into workstreams and meaningful feature groupings, or choose **This entire scope**. Preserve a breadcrumb and numbered **Back** option. Every selectable choice gets its own number.

Follow native parent/child relationships, including through closed parents. Parents group work; they do not automatically block it. Closed issues supply structure and relevant decisions, never pickup candidates. Where hierarchy is unavailable, use only repository-documented grouping conventions and label the view as a fallback.

Show direct-child counts only as direct-child counts. Show descendant totals only after verifying the descendant tree. If no meaningful grouping exists, say so and move on.

**Done when:** the user has chosen the area of work, or a shortcut has established it.

## 3. Find the frontier

Traverse all descendants of the chosen scope, including descendants of closed issues. Follow pagination; track missing relationships and duplicate references. Consider only **open actionable issues** for pickup. If the selected issue is itself actionable, include it and say so.

Check the next gate for each relevant open issue using its body, labels, assignment, native blockers, and, where necessary, comments and linked PRs. Verify blocker states rather than assuming a dependency is resolved. Read closed issue details only when they clarify an open issue. Keep issues outside the selected subtree out of its pickup list, even when they are related dependencies.

For a normal navigation session, offer these **numbered next actions** before listing tickets:

1. **Needs decisions** — unresolved questions; hand off to `/grill-with-docs`.
2. **Needs triage** — incoming reports or requests not yet evaluated; hand off to `/triage`.
3. **Ready to implement** — sufficiently specified, unblocked, unclaimed work; hand off to `/implement`.
4. **Needs review** — an actual reviewable diff/PR or explicit review request; hand off to `/code-review`.

For `/next bugs`, find open issues classified as bugs under the repository's conventions, then offer the relevant next actions. For `/next triage` and `/next ready`, go straight to the corresponding ticket list. Do not send tickets already produced by `/to-tickets` back through `/triage` merely because they are open.

A label is evidence, not a complete readiness check. Assigned work is claimed; show it as context, not unclaimed pickup. Human acceptance or business validation is a human gate, not an implementation or code-review ticket. Do not present an issue with unknown blocker state as verified ready.

**Done when:** every open actionable issue in the selected scope has been considered for the requested next action, or the result is explicitly provisional.

## 4. Let the user choose

Show a short numbered list of matching tickets (about five at a time). Use **linked titles**, not bare issue numbers. For each, give a one-line reason it matters, its next gate, and any claim or blocker that affects pickup. Include numbered **Show more** and **Back** choices.

State how many open candidates were checked and whether the result is complete. Say **no available tickets** only when coverage and eligibility were verified; otherwise explain what remains unknown. Prefer repository-defined priority and work that unblocks other work. Never invent urgency.

**Done when:** the user chooses a ticket or understands why none is available.

## 5. Hand off

Re-read the chosen issue and its relevant comments, native parent, blockers, assignee, and linked PRs. Give the user the issue link, why it is next, and the appropriate existing skill. Invoke that skill only if the environment supports it and the user chooses to proceed; otherwise provide the exact command and stop.

The destination is **completed, validated work**, but `/next` stops at the handoff. The receiving skill owns its workflow and approval rules.

**Read-only boundary:** navigation never creates, assigns, labels, comments on, closes, reparents, or changes dependencies on issues or PRs. Obtain explicit approval before any GitHub mutation, and let the appropriate workflow carry it out.
