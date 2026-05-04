---
name: plan-epic
description: "Plan an entire epic: read Jira epic, investigate codebase broadly, design architecture/data models/deliverable breakdown, and present for approval. Usage: /plan-epic MLID-XXXX"
user_invocable: true
arguments:
  - name: epic_id
    description: "Jira epic ID (e.g., MLID-2011)"
    required: true
---

# /plan-epic $ARGUMENTS

You are starting the **epic-level planning phase**. Your goal is to design the big picture: architecture, data models, integration strategy, and deliverable breakdown for an entire epic. You use **opus-level reasoning** for deep architectural decisions.

**This skill does NOT plan individual tasks.** It produces the master plan that individual tasks will reference. For task-level planning, use `/plan-task`.

## Step 1 — Read the Jira epic

Use the Atlassian MCP to read the Jira issue `$ARGUMENTS`:

- Summary, description, and acceptance criteria
- Story points and issue type (should be Epic or Story with sub-tasks)
- Sub-tasks / child issues — list them all
- Links, attachments, and design references

Also read any linked issues or sub-tasks to understand the full scope.

## Step 2 — Read the branch strategy

Read `docs/agomez/dx/epic-git-branch-strategy.md` to understand the epic branch model.

Also check the current git branch with `git branch --show-current`.

If an epic plan-progress file already exists at `docs/agomez/plans/`, read it to understand what's already been decided. This skill may be run to **create** or **update** the epic plan.

## Step 3 — Investigate the codebase (broad)

Launch the `codebase-investigator` agent with a prompt that includes:

- The full epic description and scope
- All sub-tasks and their descriptions
- Key domain areas mentioned in the epic

The investigation should be **broad** — this is about understanding the landscape, not a single feature. Ask the investigator to find:

- All existing code related to the epic's domain (models, services, API routes, pages)
- Data shapes and relationships between entities
- Integration points with external systems
- Existing patterns for similar features
- Architecture constraints (auth, RBAC, feature flags, real-time)

Wait for the investigation results.

## Step 4 — Design the epic architecture

Launch the `solution-architect` agent (opus model) with a prompt that includes:

- The full Jira epic details (from Step 1)
- The codebase investigation summary (from Step 3)
- The branch strategy (from Step 2)
- The epic plan template: `docs/agomez/dx/epic-workflow.md`

**Tell the architect to focus on:**

1. **Architecture decisions** — data models, service layer design, API contract design, real-time strategy, integration approach
2. **Data model design** — new collections, schema shapes, indexes, relationships to existing data
3. **Deliverable breakdown** — group sub-tasks into self-contained, testable deliverables that can be PRed to develop independently
4. **Dependency graph** — which deliverables depend on others, what's the critical path
5. **Risk identification** — technical risks, open questions, decisions that need stakeholder input

**Tell the architect NOT to:**
- Write detailed implementation steps for individual tasks (that's `/plan-task`'s job)
- Copy patterns from existing code — apply best practices constrained by the tech stack

The architect writes the plan to `docs/agomez/plans/$ARGUMENTS-plan-progress.md` (or updates it if it already exists).

## Step 5 — Present the plan for approval

After the architect completes:

1. Read the plan file that was written
2. Present the full plan to the user inline
3. **STOP and wait for approval**

Tell the user:
- "Here's the epic plan. Review architecture decisions, deliverable breakdown, and data models. Let me know if you'd like changes, or approve to start task-level planning."
- Do NOT proceed to write any implementation code
- Do NOT plan individual tasks — the user will run `/plan-task` for each one

## Important Rules

- **No implementation code** — this skill only produces the epic plan document
- **Always use both agents** — investigator (sonnet) first, then architect (opus)
- **Always stop for approval** — never auto-proceed
- **Big picture only** — architecture, data models, deliverables, dependency graph. NOT implementation steps for individual tasks.
- **Plan file**: `docs/agomez/plans/$ARGUMENTS-plan-progress.md`
