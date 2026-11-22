---
name: next
description: Navigate a repository's GitHub issue hierarchy and open work to choose the next ticket and hand it to an existing AI Hero skill. Use for /next, backlog exploration, or deciding what to work on next.
metadata:
  version: "1.3.1"
---

# Next — backlog navigator

Find the next useful action through **scope → work queue → open ticket → existing skill**. This is read-only navigation, not another issue-management workflow. GitHub and the repository's agent conventions remain authoritative.

## Invocation

- `/next --version` — report `metadata.version` from this loaded skill and stop; do not query GitHub. This identifies the version the agent has loaded, not necessarily the version on GitHub or in the shared skill store.

- `/next` — discover the current repository and offer its top-level scopes.
- `/next <issue-number>` — jump to that issue's subtree, or assess the issue itself if it is a leaf.
- `/next <scope>` — use repository-documented aliases; `all` means all issues in the repository.
- Continue the selected scope within an unambiguous session; otherwise show top-level choices. Do not guess scope from branch, recent activity, or unrelated conversations.

## 1. Discover the repository and conventions

1. Resolve the GitHub repository from the current remote or explicit user input; ask if ambiguous.
2. Read `AGENTS.md` and the repository's issue-tracker and triage-label configuration, where present (for example `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`). Consult domain documentation only when interpreting a selected ticket requires it.
3. **Use the installed GitHub CLI as the primary interface:** `gh issue`, `gh pr`, and `gh api`; inspect `gh --version` and the relevant command's `--help` if compatibility is uncertain. Use `--repo` when needed. Use `--json`, `--jq`, `jq`, and `gh api --paginate` where supported. Prefer native CLI/API queries over Python, custom scripts, or scraping for routine GitHub discovery and relationship traversal. If native CLI features cannot express a query, use `gh api`; disclose unavailable data rather than inventing it.
4. Fetch a paginated inventory of **open and closed issues** for navigation and progress, excluding PRs. Keep issue identity, state, title, and relationships; avoid eagerly fetching every issue's comments, blockers, and detailed evidence. Identify documented roots and any missing coverage. A complete inventory does **not** establish complete relationship coverage.

**Done when:** repository, applicable conventions, root scopes, and inventory limitations are known.

## 2. Navigate scopes

Present a short **numbered** menu of real root initiatives or documented grouping views. On selection, show immediate child scopes and allow further drill-down. Offer **This entire scope** at each level. Do not force a fixed depth: inspect actual children before skipping feature/specification groupings. If there are no meaningful child groupings, explain briefly and proceed to queues.

- Native parent/child edges represent grouping or ownership, **not** blockers. Where native hierarchy is absent, use documented milestones/labels as clearly marked views, never invented parent edges.
- Include closed grouping/ancestor issues when necessary to find open descendants or explain history. Closed issues are **not pickup candidates**.
- For each displayed progress count, specify **direct children** or **all descendants**, and whether the scope issue itself is excluded. Use `open / closed / total`; closure does not establish business acceptance.
- Preserve breadcrumb and numbered **Back / Change scope** options.

**Done when:** the user selects a subtree, an individual issue, or the whole repository.

## 3. Establish the selected subtree's open-work frontier

Traverse the selected scope's **entire native descendant tree**, including through closed intermediate parents, with pagination and cycle/duplicate detection. Record relationship lookup failures. This structural pass is needed to avoid overlooking open tickets; it does **not** require detailed review of closed issue bodies or comments.

Build the work frontier from **open actionable descendants only**, at any depth. Include the selected issue itself only if actionable and state that explicitly. Closed issues remain structural nodes and optional contextual evidence. If a repository supplies a trustworthy open-issue filter plus ancestry data, use it to reduce calls while still proving that no open descendants are omitted.

**Do not audit all closed tickets for readiness.** Fetch closed-ticket content only when relevant to an open candidate's specification, historical decision, parentage, or dependency. Keep external blockers visible but outside the selected subtree's pickup candidates.

Report scope coverage separately: number of descendants found, open candidates found, relationship failures, and any uncertainty. Do not equate the issue inventory count with subtree verification.

**Done when:** the open-work frontier is complete or explicitly provisional.

## 4. Choose a work queue

Present exactly four **separately numbered** choices before listing tickets:

1. **Needs decisions** — unresolved product, business, architecture, or specification questions → `/grill-with-docs` (or equivalent).
2. **Needs triage** — claim, ownership, scope, evidence, or readiness needs evaluation → `/triage`.
3. **Ready for delivery** — adequately specified, unblocked, unclaimed open work → `/implement` (or equivalent).
4. **Needs review** — an actual reviewable change/linked PR or explicit review request → `/code-review`.

Queues are derived views, not new hierarchy levels or labels. Use labels as evidence, not proof. Prefer the earliest unresolved gate unless actual issue/PR evidence establishes a later stage. Surface manual UAT or business sign-off as a distinct human action, not code review.

**Done when:** the user chooses a queue.

## 5. Evaluate open candidates and show tickets

For the selected queue, inspect **all open actionable issues in the selected frontier** sufficiently to classify them. Retrieve labels, assignments, native blockers, and issue content as needed. Retrieve detailed comments and linked PR information **selectively** when classification depends on them; do not perform full evidence audits for every closed issue. Verify whether each blocker is still open. An unknown blocker state is **not** verified unblocked.

- For delivery, require valid ownership under repository policy, adequate acceptance criteria, no verified open blockers, and no assignee/claim. Exclude blocked, assigned, or uncertain candidates from the unclaimed pickup list; summarize them separately if useful.
- For review, verify a linked PR/diff or explicit review request; don't infer review readiness from labels alone.
- For decision/triage, identify already-claimed tickets and do not present them as unclaimed pickup work.
- Rank using documented priority, business value, and dependency-unblocking impact; never invent urgency.
- Show about five matches initially, then a numbered **Show more** option. Each ticket shows linked title and number, business outcome, relevant state/labels, assignee, open blockers or uncertainty, reason for classification, and recommended skill.
- State **evaluated open candidates / total open actionable descendants**, number of matches, and unverified cases. Say **verified empty queue** only when every open actionable descendant has been evaluated for that queue; otherwise say **no verified matches yet** and describe the coverage gap.

**Done when:** the user can select an eligible open ticket, or understands why no eligible ticket was found.

## 6. Hand off without taking over

Re-read the selected issue's body, comments, labels, assignees, native parent, blockers, and relevant PRs; recheck eligibility immediately before handoff. Pass the issue link and essential evidence to the existing AI Hero skill, following that skill's own approval and claim rules. If direct skill invocation is unavailable, provide the exact command rather than claiming to have invoked it.

**Read-only boundary:** `/next` does not create, assign, comment on, close, relabel, reparent, claim, or change issue/PR dependencies. Ask for explicit approval before any mutation and let the appropriate skill perform it. Do not silently implement work.

## Presentation contract

- **Every selectable option is individually numbered**: initiatives, workstreams, features, queues, tickets, This entire scope, Show more, Back, and Change scope. Never combine multiple selectable workstreams under one number. Keep numbering unambiguous within the current menu.
- Keep output compact and navigable, with a breadcrumb and brief business descriptions. Use native interactive controls when available, but preserve numbered text alternatives.
- Distinguish inventory, hierarchy, and candidate-evaluation coverage; distinguish direct-child counts from descendant counts.
- Prefer `gh` CLI and its native API support; use `jq` for JSON processing. Do not use Python for routine GitHub querying or hierarchy traversal.
- Keep repository-specific root IDs, labels, domain terms, and ownership policies in the repository, not this skill. No new dashboard, database, or GitHub labels.
