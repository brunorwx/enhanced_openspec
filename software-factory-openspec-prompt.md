# Build request

Build a simple, visual "software factory" in this repo using a Claude Code skill, subagents, OpenSpec, and Markdown files. Keep it small: a viewer should understand the whole thing in a few minutes. I[...] 

Use OpenSpec as the source of truth for feature planning and implementation tasks. The factory must not create a parallel backlog, spec, or task checklist. The factory's main job after OpenSpec create[...] 

## The orchestrator

The factory is one skill, `/factory`. Whichever Claude Code session runs it becomes the orchestrator: it runs the loop below, coordinates OpenSpec and the implementation/review subagents, integrates t[...] 

Before implementing the factory, inspect the repository for OpenSpec's configuration, schema, active changes, installed skills or commands, and CLI version. Follow the workflow already configured in t[...] 

## The loop (edit this to change the factory)

```mermaid
flowchart TD
    feature --> plan["OpenSpec proposal\nspecs · design · tasks"]
    plan --> planReview["Review OpenSpec plan\nbefore implementation"]
    planReview --> dispatch["Map OpenSpec tasks\nto specialist agents"]
    dispatch --> implement["Parallel implementation\nwhen tasks are independent"]
    implement --> integrate["Integration lead\nresolve contracts and run app"]
    integrate --> security["security review"]
    integrate --> ux["UX and accessibility review"]
    integrate --> ui["UI design review"]
    integrate --> code["code and architecture review"]
    integrate --> aiReview["AI quality and safety review\nif AI behavior changes"]

    security --> wait["WAIT FOR ALL\napplicable reviews in"]
    ux --> wait
    ui --> wait
    code --> wait
    aiReview --> wait

    wait -->|all PASS| verify["verify OpenSpec requirements\nand task completion"]
    verify --> approver["approver\nopus"]
    approver -->|APPROVE| merge["merge to main"]
    approver -->|ESCALATE| human["needs-human"]

    wait -->|any CHANGES: route fixes to component owners (max 2 rounds)| implement
    human -. "/factory approve" .-> human
```

OpenSpec owns the change proposal and implementation checklist. The plan stage creates or updates the active OpenSpec change using the repository's configured workflow (normally proposal, spec deltas,[...] 

## Target file system

Adapt this layout to the repository's actual OpenSpec configuration and existing conventions:

```text
.claude/
  skills/factory/SKILL.md          # the orchestrator's instructions
  agents/                           # implementation specialists, integration lead, and reviewers
openspec/
  config.yaml                       # existing OpenSpec project configuration, if present
  specs/                            # current system requirements, maintained by OpenSpec
  changes/
    <change-name>/                  # OpenSpec-owned proposal and implementation plan
      proposal.md
      specs/                        # delta requirements
      design.md                     # when the configured workflow calls for it
      tasks.md                      # OpenSpec-owned implementation checklist
    archive/                        # completed changes, archived through OpenSpec
factory/
  board.json                        # dashboard state; never the source of task truth
  dashboard.html                    # live Kanban board, refreshes its board data
  jobs/<change-name>/               # factory-only review and decision records
    round-1/
      review-security.md
      review-ux.md
      review-ui.md
      review-code.md
      review-ai.md                  # only when AI/agent behavior is in scope
    decision.md
```

Do not add `factory/backlog.md`, `factory/jobs/<change>/spec.md`, or a second implementation checklist. New work is proposed through OpenSpec. `/factory next` selects an existing active OpenSpec chang[...] 

## Example agent files

`.claude/agents/<name>.md` (Claude Code subagent syntax):

```yaml
---
name: example-agent
description: One line on when the orchestrator should use this agent.
tools: Read, Grep, Glob, Bash, Write
model: sonnet
---
```

Each agent reads its context from the OpenSpec change and repository files. Implementation agents change only the files or components assigned to them and report the OpenSpec task IDs they completed. [...]

## Implementation team

Create a small set of reusable specialist subagents based on the stack and repository. Do not spawn every role for every change. The orchestrator selects only the roles needed by the OpenSpec tasks; i[...] 

Typical roles include:

- **Integration lead**: owns the feature branch, task dispatch, API and data contracts, shared-file coordination, integration, and end-to-end validation. This can be the orchestrating session if that [...]
- **Frontend**: web or mobile UI, client state, routing, accessibility, and frontend tests.
- **Backend/API**: services, endpoints, business logic, authorization, and API tests.
- **Data/platform**: schemas, migrations, persistence, infrastructure, deployment, and observability.
- **AI/agent systems**: model and provider adapters, prompts, tool definitions, orchestration, memory or retrieval, structured outputs, evaluations, tracing, and failure handling.
- **QA/reliability**: test strategy, regression coverage, integration scenarios, and reliability checks.

For AI or agent features, explicitly consider evaluation datasets and success criteria, tool permissions and input validation, prompt injection boundaries, retries and timeouts, token/cost limits, str[...] 

Split work by OpenSpec task IDs and clear file/component ownership. Parallelize only tasks whose dependencies and write sets do not conflict. If the environment supports isolated worktrees or equivale[...] 

## Example OpenSpec change

Use the schema and templates configured in the repository. With the standard spec-driven workflow, an active change is shaped like this:

```text
openspec/changes/003-rate-limiting/
  proposal.md
  specs/
    rate-limiting/spec.md
  design.md                         # only when needed by the schema
  tasks.md
```

The tasks in `tasks.md` are the implementation checklist. Do not copy them into a factory-owned file. The factory can display task IDs, assigned agents, and mirrored progress, but OpenSpec remains the[...] 

## Rules

- Inspect the repo and its OpenSpec setup before creating or changing a change proposal.
- OpenSpec artifacts are the source of truth for proposal, requirements, design, and implementation tasks. Never create duplicate factory specs, backlogs, or task lists.
- Use the configured OpenSpec schema and installed workflow. Follow its artifact instructions and validation; do not hard-code assumptions about optional artifacts.
- Review the proposal, specs, design when present, and tasks before implementation. Do not dispatch implementation agents until the plan is accepted for implementation.
- Every implementation task must be assigned to an accountable specialist agent or the integration lead. Agents get context from OpenSpec and repository files, not undocumented conversation context.
- Define interfaces and file ownership before parallel work. Avoid overlapping edits; sequence dependent work and integrate subagent changes before marking OpenSpec tas[...]
- Each review starts with `VERDICT: PASS` or `CHANGES`; `decision.md` starts with `APPROVE` or `ESCALATE`.
- The integration lead creates or uses a feature branch (`factory/<change-name>`). Implementation subagents work only on their assigned OpenSpec task IDs and owned files/components.
- Validate the full stack across its boundaries: for example, client-to-API behavior, API-to-data behavior, migrations, auth, and the built application. Use the repository's documented test command an[...]
- For AI/agent changes, validate the specified behavior with relevant deterministic tests or evaluations, including failure paths and tool boundaries where applicable. Do not treat a successful model [...]
- The orchestrator updates OpenSpec task checkboxes only after integration and verification. Never mark a task complete unless its work is done.
- WAIT FOR ALL: a review round is done only when every applicable reviewer's file exists in that round folder. Never start a rework cycle mid-round.
- Give each implementation agent a clear name tied to the change and role (for example, `frontend-<change-name>` or `ai-systems-<change-name>`). Keep the same agents for follow-up fixes when possible,[...]
- Allow at most two rework rounds. If issues remain, escalate to a human.
- After all reviews pass, verify that the implementation matches the OpenSpec requirements and all applicable tasks are complete. Do not archive the OpenSpec change until implementation and reviews ar[...]
- Only `/factory approve <change-name>` may merge a job in `needs-human` into `main`.

## Commands

```text
/factory <feature>                   create or continue an OpenSpec change, then dispatch its tasks to the factory agents
/factory next                         dispatch unfinished tasks from the first eligible active OpenSpec change
/factory dashboard                   start and open the live factory Kanban dashboard
/factory approve <change-name>        merge a needs-human job into main
/factory rework <change-name> <note>  route feedback to the implementation agent(s) that own the affected tasks
```

`/factory dashboard` starts the dashboard's local server using the repository's supported mechanism and opens the dashboard in the available browser. If automatic browser opening is unavailable, print[...] 

Use the actual installed OpenSpec skills or CLI commands for proposing, applying, verifying, validating, and archiving. If the requested factory command would conflict with an active OpenSpec change, [...] 

## Dashboard

Build a Kanban-style dashboard in `factory/dashboard.html` that shows both live active changes and past archived changes. Use columns that match the implemented factory flow, such as Planning, Ready, [...]

The dashboard must refresh automatically while open so changes made by `/factory` appear without a manual reload. Use a lightweight polling or file-watch mechanism compatible with the repository's env[...] 

`/factory dashboard` starts the local dashboard server if it is not already running and opens it in the available browser. Keep server startup idempotent and document how to stop it. If the environment cannot open a bro[...] 

```