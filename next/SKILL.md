---
name: next
description: Navigate a repository's GitHub issue backlog to choose the next piece of work. Use when the user invokes /next, asks what to work on next, or wants to explore initiatives, features, and actionable tickets before entering an existing AI Hero workflow.
---

# Next — backlog navigator

Find a useful next action without creating a second project-management workflow. This is a **read-only discovery and routing skill**: GitHub issues and pull requests remain authoritative; existing skills own decisions, triage, specification, delivery, and review.

## Invocation

- `/next` — discover the repository and offer its top-level issue scopes.
- `/next <issue-number>` — open that issue as a scope; show its descendants and work queues, or the issue's own next action if it is a leaf.
- `/next <scope>` — use a documented scope alias such as `mvp` or `all` if the repository defines it. `all` means the complete issue inventory.
- `/next` after a selection in the same session — continue from the selected scope when unambiguous; otherwise show the top-level choices. Never infer scope from the current Git branch, latest issue activity, or unrelated chat history.

## 1. Discover the tracker

1. Determine the current repository from its remote or explicit user context. If ambiguous, ask which repository; do not guess.
2. Read the repository's agent instructions and issue-tracker configuration, if present. Prefer `AGENTS.md`, `docs/agents/issue-tracker.md`, and `docs/agents/triage-labels.md`, or their documented equivalents. These define issue ownership, hierarchy, triage labels, and native dependency conventions. Consult domain documentation only when a candidate needs domain-level interpretation.
3. Fetch **open and closed** issues, paginating until the inventory is complete. Exclude pull requests from the issue inventory; retrieve linked PRs separately when evaluating review work. Obtain native parent/child relationships and relevant dependency information through the tracker API, using documented fallbacks only where native support is unavailable.
4. If the scan is incomplete or an API relationship cannot be retrieved, explicitly identify the missing coverage and limit conclusions accordingly. Do not treat a partial page of results as the complete backlog.

**Done when:** the repository, tracker conventions, inventory coverage, and known hierarchy roots are established or their limitations are explicit.

## 2. Explore scope, progressively

Present a **short, selectable set of high-level scopes** derived from the repository's actual hierarchy (for example initiatives or root maps). For each, show a name, issue link, concise purpose, and compact open/closed progress summary. Use names with linked issue numbers rather than bare numbers.

After selection, show its immediate child scopes (for example workstreams), then feature/specification groupings **where they actually exist**. At each step:

- Offer **This entire scope** to skip further drilling.
- Allow drill-down as deep as the native hierarchy warrants; three levels are a useful default, not a mandatory schema.
- Include closed ancestors and children for orientation, but reserve pickup recommendations for actionable open work.
- Respect native parentage; a parent-child edge means ownership/grouping, **not** a blocking dependency.
- If no meaningful hierarchy exists, offer the repository's documented grouping (milestones, labels, or explicit maps), then an ungrouped issue list. Describe such groups as views, not invented parents.
- Preserve the selected scope within the current conversation; explicit navigation overrides it. Offer Back and Change scope.

**Done when:** the user selects an issue subtree or all-repository scope, or explicitly asks for a direct recommendation.

## 3. Choose the work queue

Within the selected scope, present these **four work-type choices before listing tickets**. Show verified counts when available; otherwise omit counts rather than estimating them.

| Queue | Meaning | Default handoff |
| --- | --- | --- |
| **Needs decisions** | Unresolved business, product, architectural, or specification questions that need human input | `/grill-with-docs` (or the repo's equivalent grilling skill) |
| **Needs triage** | Issues whose claim, ownership, scope, evidence, or readiness must be evaluated | `/triage` |
| **Ready for delivery** | Fully specified open work that is unblocked, unclaimed, and permitted for agent pickup | `/implement` or the repository's delivery skill |
| **Needs review** | Implemented changes with an actual linked PR/diff or documented review request | `/code-review` |

These queues are **derived views**, not new GitHub hierarchy levels or labels. Use the repo's mapped triage labels as evidence, never as the sole criterion. Distinguish confidently verified candidates from tickets that need verification. If a candidate fits multiple queues, prefer its **earliest unresolved gate** (decision before triage, triage before delivery, delivery before review), unless the actual issue/PR evidence establishes a later stage. A `needs-triage` label alone does not prove that a decision interview is needed; a `ready-for-agent` label alone does not prove that pickup is possible.

When work requires manual UAT, business sign-off, or another human action outside the four AI Hero workflows, surface it as a clearly identified **human action** in the relevant queue or scope summary; do not misrepresent it as code review or agent delivery.

**Done when:** the user chooses one work type.

## 4. Show pickup candidates

Only now list matching tickets under the selected subtree. Default to a **small, useful batch (about 5)** with an option to show more. Each candidate must show:

- Linked issue **title and number**; one-line business outcome or purpose.
- Current open state, relevant readiness label, assignee/claim status, and any **open native blockers** (or documented fallback).
- Why it belongs in this queue; what is still uncertain.
- The suggested existing skill or human action.

For delivery, require an open actionable issue, adequate specification/acceptance criteria under repo policy, valid ownership if required, **zero unresolved blockers**, and no current assignee/claim. Exclude blocked or assigned issues from the pickup list; optionally summarize them separately as unavailable. For review, inspect linked PR state and actual review need rather than treating an issue label as a review request. For decision/triage queues, clearly mark tickets already claimed by someone else and do not offer them as unclaimed pickup work.

Prefer business importance, dependency-unblocking value, and repository-defined priorities over arbitrary issue number or last-updated date. When priority is unknown, say so; never invent urgency. Closed issues are context and progress evidence, not pickup candidates.

**Done when:** the user can choose a specific eligible ticket with an explained next action.

## 5. Hand off, do not take over

On selection, re-read the issue body, comments, labels, assignees, native parent, dependency status, and linked PRs where relevant. Recheck eligibility because GitHub may have changed since the overview. Summarize the current evidence and then invoke or guide the user into the **existing** skill for that queue, passing the issue link and necessary context. Follow that skill's own requirements, including domain docs, approvals, claim semantics, and acceptance checks.

If the environment cannot invoke another skill directly, provide the exact suggested command and issue link instead of pretending to have executed it.

**Read-only boundary:** navigation never creates, closes, assigns, comments on, relabels, reparents, or changes dependencies of issues or PRs. Request explicit approval before any such mutation, and let the appropriate existing skill perform it. Never silently launch an implementation or claim an issue.

## Presentation contract

- Prefer compact, conversational choices: **scope → work type → tickets → handoff**. Avoid dumping the full backlog or forcing the user to know issue numbers.
- Provide a short breadcrumb and a Back/Change scope choice at each drill-down stage.
- Show progress as open/closed counts with a clearly stated denominator; don't equate issue closure with accepted business functionality.
- Use the available interface's native interactive selection when supported; otherwise numbered options in ordinary text are sufficient. Do not require a custom dashboard, persistent database, or new labels.
- Keep repository-specific names, root IDs, and label mappings **out of this skill**; discover them at runtime from the repository.
