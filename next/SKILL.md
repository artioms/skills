---
name: next
description: Navigate GitHub issues to find the next actionable step toward a project goal and recommend an AI Hero main-flow skill. Use for /next, /next bugs, /next triage, /next ready, or deciding what work to do next.
metadata:
  version: "1.5.4"
---

# Next

Find the work that can move the selected goal forward. Present **up to ten ranked, actionable open issues**, with the appropriate **AI Hero main-flow skill** beside each. **Stop after recommending.** This is a read-only navigator, not a delivery workflow.

## Invocation

- `/next` — navigate the repository's documented initiatives and workstreams.
- `/next <issue-number>` — inspect that issue or navigate its open descendants.
- `/next bugs` — rank open bug issues by their next actionable step.
- `/next triage` — rank open issues carrying the repository's triage label.
- `/next ready` — rank open, unclaimed, unblocked agent-delivery issues.
- `/next --version` — print `metadata.version` and stop; do not query GitHub.

These are simple entry points, not a query language. Narrow the scope conversationally. Never guess the active workstream from the branch or a previous session.

## Process

### 1. Orient

Identify the current repository and read its relevant agent, issue-tracker, label, and hierarchy conventions. Use the documented initiative roots and goals, not invented priorities.

Use the **native GitHub CLI** (`gh issue`, `gh pr`, `gh api`, `--jq`/`jq`) and targeted, paginated queries. Prefer `gh issue list --state open` with the relevant label/search/scope and only required `--json` fields. Use `gh issue view --json` for shortlisted issues and `gh api` for native sub-issues and dependency relationships. Check supported flags/endpoints against the installed `gh` version when necessary. Reuse retrieved results; do not use Python for routine GitHub querying or traversal. Never confuse a page limit with complete coverage.

**Done when:** repository, goal, root map, and tracker conventions are established or gaps are disclosed.

### 2. Navigate

For broad `/next`, present concise **numbered** choices: initiatives, workstreams, and feature/specification groupings, plus **Entire scope**. Keep a breadcrumb. Offer **Back** only below the repository root. Name the repository-wide option **Entire repository — find work across all initiatives**. Avoid decorative headings such as “Offering nine choices.”

Use native parent/child relationships. **Parentage is not a blocker.** Use closed issues only when needed for initial hierarchy orientation, ancestry, or historical decisions. Discover all pickup candidates from **open issues only**; never recommend closed issues.

**Hierarchy invariant:** a parent stays open while any descendant remains open. A closed parent with an open descendant is a governance violation: flag it, preserve visibility of the open descendant, and never silently discard that work. Closing all children does not itself prove parent acceptance. State whether counts mean direct children or all descendants.

**Done when:** the user selects a scope, issue, or shortcut.

### 3. Discover the complete lightweight candidate set

For the chosen scope/filter, enumerate **all matching open issues**, using pagination and narrow native queries. Include open descendants at every level; verify ancestry when the query does not directly express native hierarchy. Do not scan all closed issues. Distinguish parent maps, specifications, delivery tasks, bugs, review requests, acceptance gates, and release decisions.

For `/next triage`, evaluate **every open issue with the repository's documented triage label** at least at lightweight level. This includes parent and gate issues, but their roles affect ranking. For `/next ready`, enforce the repository's actual readiness conventions. For `/next` across a scope, compare candidates across the **whole main-flow lifecycle**, not only implementation-ready tickets.

Lightweight comparison means examining the entire candidate set's titles, states, labels, assignees, roles, hierarchy location, and relevant readily available descriptions. It does **not** require fetching all comments, blocker lists, or full histories. Use these signals to identify a broad shortlist covering plausible high-impact actions across the goal and workflow stages. Do not select only the first page, first ten search results, or most prominent milestone titles.

**Done when:** every matching open candidate has been considered at lightweight level, with total coverage and any inaccessible records stated. If this cannot be achieved, label the subsequent ranking **provisional** and never claim it is the repository's best ten.

### 4. Verify the actionable dependency frontier

Before ranking, inspect **native GitHub relationships** for the shortlist, using the repository's actual dependency endpoints:

- `blocked_by`: fetch the linked issues and **verify each issue's current open/closed state**. A count alone is insufficient.
- `blocking`: verify downstream issues when claiming that completing a candidate unlocks other work.
- Native parent/child links: verify ownership and ancestry independently of dependency links.
- Assignment/claim, acceptance criteria, linked PRs, and relevant comments: inspect when they affect the proposed next action.

If a candidate's proposed next step depends on an **open** blocker, follow `blocked_by` recursively toward the **actionable frontier**: an open prerequisite whose proposed next step can begin now. Continue through multi-level chains as needed; deduplicate visited nodes, detect cycles, and report missing permissions or failed lookups. A blocker outside the selected scope may explain the bottleneck, but label it as **external dependency**, not a descendant. Prefer recommending an actionable in-scope prerequisite; disclose when progress depends on out-of-scope work.

**Separate preparation from execution.** An issue blocked for implementation or final approval may still be independently actionable for `/triage`, `/grill-with-docs`, or `/to-spec`. Recommend that preparation only if there is a **specific unresolved question or readiness gap that can be resolved now**. State explicitly that execution remains blocked. A label alone does not establish such a gap.

**Never recommend blocked implementation or approval as ready work.** Do not rank a high-level blocked gate first solely because it is strategically important. Instead identify the nearest verified actionable prerequisite or, if none exists, report the blocker and the necessary human/coordination step.

For every final recommendation, verify the **specific next action** can start now, and check native blockers to the extent relevant to that action. If dependency status cannot be verified, say **unverified**, not **unblocked**. Refill the shortlist with additional candidates when verification eliminates too many; aim for ten credible items when available.

**Done when:** every displayed recommendation has a verified action, a checked dependency position, and a clear reason it advances the goal; unresolved verification gaps are explicit.

### 5. Choose the main-flow handoff

Recommend the appropriate **main-flow AI Hero skill**, verifying installed names where necessary:

- `/grill-with-docs` — unresolved product, business, architecture, or specification decisions.
- `/to-spec` — outcome understood but specification missing or inadequate.
- `/to-tickets` — approved specification needs dependency-aware, agent-sized tickets.
- `/triage` — incoming issue needs ownership, scope, evidence, or readiness assessment.
- `/implement` — specified, open, **unblocked**, unclaimed delivery ticket.
- `/code-review` — an actual linked PR or reviewable change.
- **Human action** — acceptance, approval, or coordination without a suitable main-flow skill.

Do not recommend specialized techniques (`/tdd`, `/research`, `/diagnosing-bugs`) as the `/next` handoff. Do not route completed `/to-tickets` work automatically through `/triage`. Assigned work is not an unclaimed pickup; mention it only for a genuine human, review, or coordination action.

**Done when:** each selected action has one appropriate handoff and a verified readiness explanation.

### 6. Rank, present, stop

Rank by: **(1) verified downstream unblocking impact; (2) contribution to the selected business outcome; (3) whether the next step is actionable now; (4) ability to turn unclear work into agent-ready work; (5) ownership and explicit repository priority.** Compare across workflow stages for broad `/next`; respect the user's filter for `/next triage`, `/next bugs`, and `/next ready`. Use judgment, not fabricated scores or quotas. Avoid filling the list with blocked release/acceptance gates when actionable prerequisite work exists.

Present **up to ten distinct numbered actions**, highest value first. Show fewer only when fewer credible candidates exist; do not pad with irrelevant tickets.

Format:

`1. [Issue title (#123)](issue-url) — /skill-name`

`   Why: One sentence stating the actionable next step and what it advances or unblocks.`

If preparation is possible but execution is blocked, say so in the Why sentence. If the blocker is outside scope, identify that explicitly. Use **Human action** when appropriate.

Briefly state scope, `evaluated X of Y matching open issues`, and verification limitations. Use **provisional** when lightweight coverage is incomplete or important relationships remain unverified. Never claim a definitive top ten from a small sample. Omit long invocation commands, generated prompts, expected-result sections, and second-stage queue menus.

**Stop.** The user chooses the next AI Hero skill. Do not invoke it, conduct the interview, create tickets, implement, review, or modify GitHub.

## Boundaries

- **Read-only:** never create, assign, claim, comment, close, label, reparent, or alter dependencies.
- Respect repository-specific issue rules and domain documentation; keep project-specific issue IDs out of this generic skill.
- Closed issues provide context, not pickup recommendations.
- Native dependency states and relationships are authoritative; issue-body claims and counts alone are not.
- Report incomplete coverage and relationship failures plainly. The result is a **recommendation, not execution**.
