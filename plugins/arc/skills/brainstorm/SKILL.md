---
name: brainstorm
description: You MUST use this skill for any design exploration, architecture decision, or trade-off analysis before implementation begins — especially when the user says "brainstorm", "explore the design", "think through", "what approach should we take", or describes a feature with multiple valid strategies. This is the arc-native brainstorming skill that writes designs to docs/plans/ and registers them as encrypted shares (local or remote) for review via arc share. Always prefer this over generic brainstorming when the project uses arc issue tracking.
---

# Brainstorm — Design Discovery

Explore requirements through Socratic dialogue before any implementation begins.

## Hard Gate

**Do NOT write any implementation code, scaffold any project, or take any implementation action until the design is approved.** Brainstorming produces a design document — not code.

## Workflow

Create a task for each step below using `TaskCreate`. Mark each as `in_progress` when starting and `completed` when done. This creates a visible progress list in the CLI that carries forward into the plan skill.

### 1. Explore Project Context

- Check existing files, docs, recent commits
- Review existing arc issues (`arc list`)
- Understand what already exists and what constraints are in play

### 2. Ask Clarifying Questions

- Ask questions **one at a time** — don't dump a list
- **Use the AskUserQuestion tool** for multiple-choice decisions (2-4 options)
- Use open-ended text questions only when you need freeform feedback
- Understand: purpose, constraints, success criteria, target users
- Continue until you have enough to propose approaches

**Example AskUserQuestion usage:**
```
Question: "How should we handle session persistence?"
Options:
  - "In-memory only" (simplest, lost on restart)
  - "SQLite" (persistent, single-node, matches existing storage)
  - "Redis" (distributed, adds infrastructure dependency)
```

### 3. Propose 2-3 Approaches

- Each approach: summary, trade-offs, estimated complexity
- Include a recommendation with reasoning
- **Use the AskUserQuestion tool** to present approaches as structured choices
- Apply YAGNI — remove features from all designs that aren't explicitly required

**Example AskUserQuestion usage:**
```
Question: "Which approach should we go with?"
Options:
  - "Approach A: ..." (recommended — trade-offs...)
  - "Approach B: ..." (trade-offs...)
  - "Approach C: ..." (trade-offs...)
```

### 4. Present Design Section by Section

- Break the design into logical sections (data model, API, UI, etc.)
- Present each section and get user approval before moving to the next
- Iterate on sections as needed based on feedback

### 5. Identify Shared Contracts (Parallel Readiness)

If the design will produce multiple implementation tasks that could run in parallel, explicitly identify the **shared contracts** — types, interfaces, config keys, constants, and function signatures that multiple tasks will reference.

Contracts fall into two tiers:

- **Shared contracts** (referenced by 2+ tasks): produce **exact, copy-pasteable code blocks** including the type definition AND a contract test assertion. The T0 foundation task will write these verbatim.
- **Task-internal types** (used within a single task): use typed pseudocode (e.g., `FeedbackRequest { memory_id: i64, rating: i8 }`) — the subagent adapts to language idioms during implementation.

Present shared contracts to the user as a "foundation layer" with exact code:

```go
// internal/types/config.go

// SessionConfig holds session-related settings.
type SessionConfig struct {
	Timeout  time.Duration `json:"timeout"`
	MaxIdle  int           `json:"max_idle"`
	Secure   bool          `json:"secure"`
}
```

```go
// internal/storage/storage.go

// GetSession retrieves a session by ID.
// Returns nil and no error if the session does not exist.
GetSession(ctx context.Context, id string) (*Session, error)
```

Contract test assertions verify that the shared types satisfy compile-time expectations. Place these **inline in each relevant test file** with a clear separator:

```go
// internal/types/config_test.go

// --- Contract assertions ---

// Verify SessionConfig fields exist with expected types.
var _ time.Duration = SessionConfig{}.Timeout
var _ int = SessionConfig{}.MaxIdle
var _ bool = SessionConfig{}.Secure
```

```go
// internal/storage/sqlite/sqlite_test.go

// --- Contract assertions ---

// Verify SQLiteStore satisfies the Storage interface.
var _ storage.Storage = (*SQLiteStore)(nil)
```

These exact definitions and contract tests become the **T0 foundation task** during planning — implemented sequentially before any parallel work begins. The T0 task writes the shared type files and embeds contract test assertions inline in each relevant test file, so that parallel agents can import these types immediately and any drift is caught at compile time.

**Skip this step** if the design maps to a single task or purely sequential work.

### 6. Save Design and Register for Review

Write the design document to `docs/plans/`, then register it as an encrypted **share** for review.

#### 6a. Write the design file

```bash
# Use YYYY-MM-DD-<topic>.md naming convention
cat > docs/plans/YYYY-MM-DD-<topic>.md <<'EOF'
<design content>
EOF
```

#### 6b. Choose share mode (local vs remote)

Use the **AskUserQuestion tool** to pick where reviewers will see the plan. This decision controls the `arc share create` flag:

```
Question: "Where should reviewers see this plan?"
Options:
  - "Local share — review on this machine only (--local)"
      Solo work, fastest, plan stays on the local arc-server.
  - "Remote share — share via arcplanner.sentiolabs.io (--share)"
      Human teammate or another device needs to review.
```

Default to **local** unless the user has indicated they want outside review.

#### 6c. Create the share

Run the corresponding `arc share create` command:

```bash
# For "local":
arc share create docs/plans/YYYY-MM-DD-<topic>.md --local

# For "remote":
arc share create docs/plans/YYYY-MM-DD-<topic>.md --share
```

`arc share create` prints **two URLs**:

- **Share URL** — what reviewers open. No edit token. Looks like `http://.../share/<id>#k=<key>` (local) or `https://arcplanner.sentiolabs.io/share/<id>#k=<key>` (remote).
- **Author URL** — what *you* open. Same URL plus `&t=<edit_token>` in the fragment, which unlocks Accept/Resolve/Reject controls.

It also returns a **share ID**. Save it — you'll pass it to the plan skill in step 8.

> Note: edit tokens are stored in the arc-server's local keyring (a `shares` table in `~/.arc/data.db`). They are never written to disk as JSON. If you lose the Author URL, regenerate it with `arc share show <id> --author-url`.

### 7. Review Loop

After `arc share create` returns, **ALWAYS output BOTH URLs** so the user can click them directly in their terminal:

```
Plan ready for review:

  Share URL  (give to reviewers): <share-url>
  Author URL (open this yourself): <author-url>

```

Then use the **AskUserQuestion tool** — include the Author URL in the options so the user sees it without scrolling:
```
Question: "Plan ready for review (open the Author URL above) — how would you like to proceed?"
Options:
  - "Approve" (proceed to /arc:plan for implementation breakdown)
  - "I've submitted feedback in the share" (read comments, revise, re-present)
  - "Save for later" (leave as draft — resume in a new session)
```

**If user approves:**
```bash
arc share approve <share-id>
```
Then proceed to step 8.

**If user says "feedback submitted":**
```bash
# Read accepted review comments only (the brainstorm-flow form):
arc share pull <share-id>

# Or read all comments + statuses:
arc share comments <share-id>

# Re-read content in case the user or a reviewer edited it:
arc share show <share-id>
```
Revise the design file based on the feedback, push the new content with:
```bash
arc share update <share-id> docs/plans/YYYY-MM-DD-<topic>.md
```
Then re-present the URLs and options. Repeat until approved.

**If user says "save for later":**
Tell the user they can resume by running `/arc:brainstorm` in a new session and referencing the plan file and share ID.

### 8. Routing Analysis & Transition

After the plan is approved, **you MUST produce a routing analysis before presenting options**. This analysis helps the user make an informed decision about what to do next.

#### Routing Analysis

Evaluate the approved design against these criteria and present a summary:

| Factor | Assessment |
|--------|------------|
| **Work items** | Count of distinct implementation tasks identified in the design |
| **Parallel readiness** | Were shared contracts identified in step 5? (yes = plan needed for T0 sequencing) |
| **Files touched** | Approximate number of files created or modified |
| **Layers crossed** | Which architecture layers are involved (storage, API, CLI, frontend, tests) |
| **Risk areas** | Any migrations, API changes, or breaking changes? |
| **Scale** | Small / Medium / Large (from Scale Detection table) |

Then produce a **recommendation** with reasoning:

```
📊 Routing Analysis
───────────────────
Work items:       N tasks identified
Parallel ready:   Yes/No (shared contracts in step 5)
Files touched:    ~N files across N directories
Layers crossed:   [storage, API, CLI, ...]
Risk areas:       [migrations, breaking changes, none, ...]
Scale:            Small / Medium / Large

➤ Recommendation: /arc:plan | /arc:implement
  Reason: <1-2 sentence justification based on the factors above>
```

**Routing rules** (use these to drive the recommendation):
- **→ arc:plan** when ANY of: 2+ work items, shared contracts exist, multiple layers crossed, migrations or breaking changes present, medium/large scale
- **→ arc:implement** when ALL of: single work item, no shared contracts, single layer, no risk areas, small scale
- When borderline, recommend `arc:plan` — the overhead of planning is low, but the cost of a disorganized multi-task implementation is high

After the analysis, use the **AskUserQuestion tool** — mark the recommended option:
```
Question: "Design approved! What's next?"
Options:
  - "Break into tasks with /arc:plan" (recommended — <brief reason from analysis>)
  - "Implement directly with /arc:implement" (for small, single-task work)
  - "Done for now" (design is saved — continue in a new session)
```

If `/arc:implement` is recommended instead, swap which option gets the "(recommended)" tag.

- **Break into tasks**: invoke the `plan` skill, passing the plan ID
- **Implement directly**: invoke the `implement` skill
- **Done for now**: tell the user the plan is approved and they can run `/arc:plan` in a new session

## Scale Detection

| Indicator | Scale | Structure |
|-----------|-------|-----------|
| Multiple phases, weeks of work, cross-cutting concerns | Large | Meta epic → phase epics → tasks |
| Single feature, days of work, contained scope | Medium | Epic → tasks |
| One task, hours of work, obvious approach | Small | Single issue |

## Rules

- The ONLY next skill after brainstorm is `plan` (or `implement` for small work)
- Never invoke implementation skills from brainstorm
- Design documents go in `docs/plans/` and are registered via `arc share create <file-path> --local|--share`
- Arc issues track persistent work; TaskCreate/TaskUpdate tracks workflow progress in the CLI
- YAGNI: if the user didn't ask for it, don't design it
- Format all arc content (descriptions, plans, comments) per `skills/arc/_formatting.md`
