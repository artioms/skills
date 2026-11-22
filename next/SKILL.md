---
name: next
description: Navigate a repository's issue backlog to choose the next piece of work. Use when the user invokes /next, asks what to work on next, or wants to explore initiatives, features, and actionable tickets before entering an existing AI Hero workflow.
---

# Next — backlog navigator

Find the next useful action without creating another project-management workflow. This is a **read-only discovery and routing skill**. The issue tracker remains authoritative; existing skills own decisions, triage, specification, delivery, and review.

## Invocation

- `/next` — discover the repository and offer top-level issue scopes.
- `/next <issue-number>` — open an issue as a scope, or assess its next action if it is a leaf.
- `/next <scope>` — use a documented scope alias such as `mvp`, or `all` for the full inventory.
- `/next` in the same session — continue the unambiguous selected scope; otherwise offer top-level choices. Never infer scope from branch names, recent activity, or unrelated chat history.

## 1. Discover the tracker

1. Determine the repository from its remote or explicit context; ask if ambiguous.
2. Read the repository's agent and issue-tracker conventions, when present (for example `AGENTS.md`, `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`). These define hierarchy, ownership, labels, and dependency semantics. Read domain docs only when interpreting a candidate requires them.
3. Fetch **all open and closed issues**, following pagination to completion. Exclude pull requests from the issue inventory; inspect linked PRs separately for review work. Discover native parent/child relationships and blockers through the tracker API, with documented fallbacks only where native support is unavailable.
4. Establish coverage: issues fetched versus total reported, relationship retrieval failures, and any inaccessible scopes. Never describe an incomplete scan as complete.

**Done when:** repository, tracker rules, root scopes, and inventory coverage are established or limitations are explicit.

## 2. Explore scopes

Offer a short list of actual root initiatives/maps, each with linked name and issue number, purpose, and progress. At each selection show immediate child scopes and offer **This entire scope** and **Back / Change scope**.

- Navigate native hierarchy to whatever depth exists; do not require three levels. Include closed grouping issues for orientation.
- For every displayed count, state whether it covers **immediate children** or **all descendants**, and whether the scope issue itself is excluded. Use `open / closed / total` with a clear denominator. Closure is not proof of business acceptance.
- Prefer concise descendant totals on high-level scope choices where they can be verified; otherwise show direct-child counts with that label or omit the count.
- Before skipping feature/specification navigation, inspect actual children. If none are meaningful groupings, say so briefly (for example, “No further feature groupings found; showing queues for the entire workstream”). Do not claim this without checking.
- If the tracker lacks native hierarchy, use documented groupings such as milestones or labels as **views**, not invented parents.

**Done when:** the user selects a subtree or the whole repository, or explicitly requests a direct recommendation.

## 3. Traverse the selected subtree before classifying

**A complete issue inventory is not a complete subtree audit.** For the selected scope, recursively traverse **every native child**, including descendants beneath **closed intermediate parents**, until all reachable descendants are accounted for. Use pagination for child lists and detect cycles, duplicate references, and failed relationship lookups. If using a fallback grouping, disclose its limitations. Include the selected issue itself only if it is actionable, and state that choice.

Construct the candidate set from **all open actionable descendants**, at any depth. Closed descendants provide history, not pickup candidates. Keep hierarchy edges separate from blocker edges. Resolve readiness, assignments, and open blockers for potential matches; distinguish unverified metadata from a verified absence of blockers.

Before reporting any queue as empty, confirm that **every open actionable descendant** has been evaluated for that queue, or say the result is provisional. Never infer a subtree-wide absence from direct children alone.

**Done when:** descendant counts, open-candidate coverage, and any traversal/verification gaps are known.

## 4. Choose the work queue

Present four choices **before listing tickets**, optionally with verified counts:

| Queue | Meaning | Default handoff |
| --- | --- | --- |
| **Needs decisions** | Unresolved business, product, architectural, or specification questions requiring human input | `/grill-with-docs` or equivalent |
| **Needs triage** | Claims, ownership, scope, evidence, or readiness requiring evaluation | `/triage` |
| **Ready for delivery** | Specified, unblocked, unclaimed open work permitted for pickup | `/implement` or equivalent |
| **Needs review** | Implemented changes with linked PR/diff or documented review request | `/code-review` |

Queues are derived views, **not** hierarchy levels or new labels. Labels are evidence, not conclusive classification. Prefer the earliest unresolved gate when stages overlap, unless issue/PR evidence establishes a later stage. Flag human UAT or business sign-off separately; do not misclassify it as code review.

**Done when:** the user chooses a queue.

## 5. Show pickup candidates

Show about five matching tickets initially, with **Show more**. For each: linked title/number, business outcome, current state/label, assignee, open blockers, why it fits, uncertainties, and suggested handoff.

- Delivery eligibility requires adequate specification/acceptance criteria, valid ownership under repository rules, **zero verified open blockers**, and no current claim/assignee. Keep blocked or assigned tickets out of unclaimed pickup results; optionally summarize them separately.
- Review requires an actual reviewable change or explicit review request, not merely an issue label.
- Decision/triage candidates already claimed by someone else must be clearly marked and not offered as unclaimed work.
- Prefer repository priority, business impact, and dependency-unblocking value; do not invent urgency.
- Report queue coverage, e.g. `Evaluated 14 of 14 open actionable descendants across 3 levels; 4 matches`, or `Evaluated 11 of 14; 3 unverified — results provisional`. Distinguish **no verified matches** from **verified empty queue**.
- If no candidates exist, offer another queue or scope rather than pulling unrelated tickets into the selection.

**Done when:** the user can choose a specific eligible ticket and understands the next action and coverage.

## 6. Hand off without taking over

When a ticket is selected, re-read its body, comments, labels, assignees, native parent, blockers, and linked PRs where relevant. Recheck eligibility because the tracker may have changed. Explain the evidence, then invoke or guide the user to the existing skill with the issue link and necessary context. Follow that skill's own approval, claim, domain, and acceptance rules. If skill invocation is unavailable, provide the exact command instead of claiming it ran.

**Read-only boundary:** navigation does not create, assign, comment on, close, relabel, reparent, or change issue/PR dependencies. Seek explicit approval for mutations and let the appropriate existing skill perform them. Never silently claim or implement work.

## Presentation contract

- Keep navigation compact: **scope → work type → tickets → handoff**, with breadcrumb and Back / Change scope.
- Show both direct-child and all-descendant counts only when their meanings are clear; never mix denominators.
- Prefer native interactive choices when supported; otherwise numbered text options suffice.
- Keep repository-specific issue IDs, root names, and label mappings out of this skill; discover them at runtime.
- No dashboard, persistent database, or new GitHub labels required.
