# Software Factory with OpenSpec

This README explains how to set up and use the software factory described in [software-factory-openspec-prompt.md](software-factory-openspec-prompt.md). The prompt is for creating the `/factory` skill, subagents, and dashboard used in a software factory workflow driven by OpenSpec.

## How the pieces fit together

- **OpenSpec** owns the change proposal, requirements, design artifacts when needed, and implementation task checklist.
- **The factory** assigns OpenSpec task IDs to implementation subagents, coordinates parallel work, integrates the changes, and runs reviews.
- **The dashboard** is a live Kanban view of active changes and an archive view of past changes. It shows factory stage, assigned agents, and OpenSpec task progress. It is read-only; OpenSpec remains the source of truth.

Do not create a separate factory backlog, feature spec, or implementation checklist.

## Set up in a repository

1. Set up OpenSpec in the target repository using the [official setup guide](https://openspec.dev/docs/setup). Check the repository's configured schema, active changes, CLI, and installed OpenSpec skills.
2. Give the target-repository coding agent the full contents of [software-factory-openspec-prompt.md](software-factory-openspec-prompt.md) and ask it to build the factory there. The prompt tells it to create the orchestrator, specialist subagents, and dashboard aligned to the repository's actual OpenSpec setup.
3. Review the generated `.claude/skills/factory/SKILL.md`, agent definitions, and dashboard. Confirm the implementation agents use OpenSpec tasks as their assignments and do not create duplicate task tracks.
4. Use the OpenSpec proposal review as the planning gate: check the problem, requirements, and tasks before asking the factory to implement the change.

OpenSpec's default spec-driven workflow commonly produces a proposal, spec deltas, an optional design, and tasks. Follow the target project's actual schema; artifact names and available commands can vary.

## Feature workflow

### 1. Create or review the OpenSpec change

Start a proposal with the OpenSpec workflow installed in the target repository. For example, describe the change and ask OpenSpec to propose it. Review the resulting proposal, spec requirements, design when present, and tasks before implementation begins.

You can also start with `/factory <feature>` if the generated factory skill supports creating or continuing an OpenSpec change. It should delegate planning to OpenSpec and stop for plan review before starting implementation.

### 2. Dispatch implementation tasks

Run `/factory <change-name>` to continue a specific active change, or `/factory next` to select an eligible active change with unchecked tasks. The orchestrator should:

- Read the change artifacts and OpenSpec apply instructions.
- Assign each unfinished OpenSpec task to an accountable specialist, such as frontend, backend/API, data/platform, AI/agent systems, or QA/reliability.
- Identify task dependencies, shared interfaces, and file ownership before starting parallel work.
- Parallelize independent tasks; sequence tasks that share files or depend on earlier work.
- Integrate the subagents' changes and validate them against OpenSpec requirements, scenarios, and the repository's documented checks.
- Mark tasks complete in the OpenSpec task file only after their work is integrated and verified.

The factory may show task IDs, agent assignments, and mirrored progress on its dashboard. Do not use the dashboard as a second task checklist.

#### Parallel implementation with git worktrees

When the environment supports isolated workspaces, the factory should prefer git worktrees for parallel task execution instead of sharing one working tree across multiple agents. A typical setup is:

- one worktree for the integration branch and orchestrator
- one worktree per independent specialist task or role
- each worktree scoped to only the files and task IDs assigned to that agent
- the integration lead merges worktree changes back into the feature branch after validation

This keeps task ownership clear and avoids overlapping edits during parallel implementation.

Example:

```bash
git worktree add ../repo-frontend -b factory/my-change-frontend
git worktree add ../repo-backend -b factory/my-change-backend
git worktree list
```

Use Git worktrees only when task dependencies and write sets do not conflict. The OpenSpec change remains the source of truth; the worktree layout is a coordination mechanism.

### 3. Review and rework

After integration, the factory runs the applicable reviews, such as security, UX/accessibility, UI, code/architecture, and AI quality/safety when AI behavior changes. It waits for every applicable reviewer to finish before starting a rework cycle.

Useful factory commands:

```text
/factory <feature>                   create or continue an OpenSpec change and run it through the factory
/factory next                         dispatch unfinished tasks from an eligible active change
/factory dashboard                   start and open the live factory Kanban dashboard
/factory rework <change-name> <note>  route feedback to the agents responsible for the affected tasks
/factory approve <change-name>        approve and merge a job that was escalated to a human
```

These are the intended factory commands. Use the OpenSpec skills or CLI commands actually installed in the target repository for proposing, applying, validating, verifying, and archiving changes.

`/factory dashboard` starts the local dashboard server if it is not already running and opens the board in the available browser. It should not start duplicate servers. If browser opening is unavailable, it should still start the dashboard server and print the local URL.

### 4. Verify and archive

Before closing a change, confirm that all applicable OpenSpec tasks are complete, the implementation meets the change requirements, and factory reviews have passed. Then use the repository's configured OpenSpec or git workflow to archive the change.

## For full stack and AI agent applications

The factory should select roles from the actual change instead of spawning every possible agent for every feature. A full stack feature may need frontend, backend/API, data/platform, and QA agents depending on the OpenSpec tasks.

For AI behavior, include relevant checks for tool permissions, input validation, prompt injection boundaries, retries and timeouts, cost limits, output validation, model/tool failures, evaluations, and safe rollback paths when needed.

## References

- [OpenSpec setup](https://openspec.dev/docs/setup)
- [OpenSpec quickstart](https://openspec.dev/docs/quickstart)
- [OpenSpec skills](https://openspec.dev/docs/skills)
- [OpenSpec CLI](https://openspec.dev/docs/cli)

