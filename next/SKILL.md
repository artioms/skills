---
name: next
description: Navigate GitHub issues to find the next actionable step toward a project goal and recommend an AI Hero main-flow skill. Use for /next, /next bugs, /next triage, /next ready, or deciding what work to do next.
metadata:
  version: "2.0.1"
---

# Next

Navigate from a project goal to the most valuable actions that can advance it now. Recommend **up to ten ranked GitHub issues**, each with its appropriate AI Hero main-flow skill, then **stop**. This skill is read-only: it recommends work; the other skills perform it.

## Invocations

- `/next` — navigate to a goal/scope, then rank actions across the main flow.
- `/next <issue-number>` — use that issue as the goal; explore open descendants and prerequisites, or assess the issue itself when appropriate.
- `/next bugs` — focus on open bugs; choose the next appropriate main-flow step for each.
- `/next triage` — focus on open issues marked for triage; determine the useful next action, which may be `/triage` or another main-flow skill if the evidence warrants it.
- `/next ready` — focus on open, unclaimed delivery work that can start now; normally `/implement`.
- `/next --version` — print `metadata.version` from the installed `SKILL.md` YAML frontmatter and stop. Do not look for `metadata.json` or query GitHub.

Filters narrow the candidate set. They do **not** replace judgment about what action will move the goal forward.

## Workflow

### 1. Establish the goal

Identify the repository and read its relevant issue-tracker, label, and hierarchy conventions. Use the repository's own documentation rather than hardcoding project-specific labels or issue numbers. If the goal is implicit, derive it from the selected root/initiative issue and its description. When user intent, success criteria, or priorities would materially change the ranking, ask one focused question. For a substantial unresolved goal or trade-off, use the existing `/grill-me` skill as an optional interactive clarification step, then resume `/next` with the agreed goal. Keep the interview proportional; do not invoke it for a clear request. Investigate GitHub facts yourself rather than asking the user to retrieve them. `/grill-me` clarifies navigation decisions; it never replaces `/grill-with-docs` or another main-flow handoff.

For `/next` without a scope, present numbered root initiatives plus **Entire repository**. Navigate with numbered workstreams/feature groups, **Entire scope**, and **Back**; show a short breadcrumb. A user may select a whole scope without drilling down. A filter may also be narrowed to a goal conversationally.

**Complete when:** the goal, repository, scope, and optional filter are clear.

### 2. Discover relevant open issues

Use targeted, paginated native GitHub CLI queries (`gh issue`, `gh api`, `jq`), with explicit `--repo OWNER/REPO` where possible. Fetch lightweight fields first. Inspect the relevant open issues across the chosen scope/filter, not merely the first page or first promising match. A failed query is not an empty backlog.

Use native GitHub parent/sub-issue relationships to navigate the scope. Closed issues are historical context, never recommendations. If a parent is closed while descendants remain open, flag the hierarchy inconsistency and retain those open descendants. Parentage alone does not mean blocked.

Identify the **work frontier**: issues that can advance the goal directly, prepare later work, or remove a prerequisite. Prefer concrete leaf work over organizational parent maps. A parent qualifies only when it has its own unresolved, actionable decision or preparation task.

**Complete when:** there is a diverse candidate pool spanning plausible next actions, or an explicit retrieval limitation.

### 3. Verify actionability and choose the handoff

For each proposed recommendation, check its current open state, native parentage, assignee, relevant issue details, and native `blocked_by` edges to the extent necessary for the proposed action. Check whether referenced blockers are still open; use `blocking` edges only when claiming downstream unblocking impact. Expand the dependency chain **only as needed** to find a genuinely actionable prerequisite. Do not infer no blockers from missing or failed data. Verification is action-specific: uncertainty about implementation readiness may itself make `/triage` actionable; it does not disqualify a candidate from triage.

**Actionable means the recommended step can make meaningful progress now**, not necessarily that implementation can start:

- **`/triage`** — there is a concrete uncertainty or classification/ownership/dependency/readiness question to resolve. An open delivery blocker does not automatically prevent triage. A `needs-triage` label is a discovery signal, not proof that triage is the best handoff.
- **`/grill-with-docs`** — requirements or decisions need structured clarification using project documentation.
- **`/to-spec`** — a sufficiently understood goal needs a written specification.
- **`/to-tickets`** — a sufficiently specified scope needs actionable delivery tickets.
- **`/implement`** — an adequately specified, open, unclaimed delivery issue has no open native blockers.
- **`/code-review`** — there is an actual reviewable PR or explicit review request.
- **Human action** — an approval, acceptance, or decision cannot be delegated to the available main-flow skills.

For `/next ready`, show only verified implementation-ready issues. For `/next triage`, search the triage-labeled set, but choose the *actual* next action: a ready ticket may warrant `/implement`, while an unresolved issue warrants `/triage` or preparation. If the label conflicts with reality, briefly identify the mismatch. For `/next bugs`, retain the bug filter but select the appropriate stage. Do not recommend executing blocked delivery; recommend an independent preparation step or actionable prerequisite instead.

**Complete when:** every displayed issue has a defensible next action and its material blockers/limitations are known.

### 4. Rank and stop

Rank candidates by **contribution to the selected goal**, ability to progress now, and downstream unblocking value; use explicit project priorities as evidence, not a substitute for actionability. Prefer the closest actionable step on a critical dependency path over a high-profile but blocked milestone. Aim for ten useful recommendations when the pool supports them; verify replacement candidates before returning a very short list. Never pad with organizational maps, blocked actions, or unverifiable issues.

Output a compact numbered list, with scope and coverage stated briefly:

`1. [Issue title (#123)](https://github.com/OWNER/REPO/issues/123) — /to-spec`
`   Why: Clarifying this workflow enables two dependent delivery tickets.`

Each item has a **real issue link**, exactly one recommended main-flow skill (or **Human action**), and a concrete reason tied to the goal. Mention an open delivery blocker when relevant, distinguishing it from a blocker to the *recommended* preparation step. If fewer than ten qualify, continue examining plausible candidates before concluding that the pool is exhausted; give a brief, evidence-based explanation. If discovery was incomplete, mark the result provisional without presenting unverified execution readiness as fact. Missing issue information can justify a concrete `/triage` recommendation when the issue itself is verified open and relevant.

**Stop.** Do not run the recommended skill, edit issues, change labels, assign work, or modify GitHub.
