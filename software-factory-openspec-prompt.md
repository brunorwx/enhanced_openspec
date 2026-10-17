# Build request

Build a simple, visual "software factory" in this repo using a Claude Code skill, subagents, OpenSpec, and Markdown files. Keep it small: a viewer should understand the whole thing in a few minutes. Inspect the repo first so the agents know its language, test command, conventions, and existing OpenSpec setup.

Use OpenSpec as the source of truth for feature planning and implementation tasks. The factory must not create a parallel backlog, spec, or task checklist. The factory's main job after OpenSpec creates a change is to route its tasks to a coordinated team of implementation subagents, integrate their work, and run reviews. The team must adapt to the repository and change, including full stack products and applications built around AI agents.

## The orchestrator

The factory is one skill, `/factory`. Whichever Claude Code session runs it becomes the orchestrator: it runs the loop below, coordinates OpenSpec and the implementation/review subagents, integrates their work, and is the only writer of `factory/board.json`.

Before implementing the factory, inspect the repository for OpenSpec's configuration, schema, active changes, installed skills or commands, and CLI version. Follow the workflow already configured in the repository. Use the available OpenSpec skills when present; otherwise use the installed CLI's `openspec instructions`, status, and validation commands. Do not assume skill names, schemas, or command options that are not installed. If OpenSpec is not set up, document the setup needed and use the supported OpenSpec setup flow rather than inventing a parallel format.

## The loop (edit this to change the factory)

```mermaid
flowchart TD
    feature --> plan["OpenSpec proposal<br/>specs · design · tasks"]
    plan --> planReview["Review OpenSpec plan<br/>before implementation"]
    planReview --> dispatch["Map OpenSpec tasks<br/>to specialist agents"]
    dispatch --> implement["Parallel implementation<br/>when tasks are independent"]
    implement --> integrate["Integration lead<br/>resolve contracts and run app"]
    integrate --> security["security review"]
    integrate --> ux["UX and accessibility review"]
    integrate --> ui["UI design review"]
    integrate --> code["code and architecture review"]
    integrate --> aiReview["AI quality and safety review<br/>if AI behavior changes"]

    security --> wait["WAIT FOR ALL<br/>applicable reviews in"]
    ux --> wait
    ui --> wait
    code --> wait
    aiReview --> wait

    wait -->|all PASS| verify["verify OpenSpec requirements<br/>and task completion"]
    verify --> approver["approver<br/>opus"]
    approver -->|APPROVE| merge["merge to main"]
    approver -->|ESCALATE| human["needs-human"]

    wait -->|any CHANGES:<br/>route fixes to component owners<br/>(max 2 rounds)| implement
    human -. "/factory approve" .-> human
```

OpenSpec owns the change proposal and implementation checklist. The plan stage creates or updates the active OpenSpec change using the repository's configured workflow (normally proposal, spec deltas, optional design, and tasks). Review those artifacts before implementation. After the plan is accepted, the orchestrator reads the OpenSpec apply instructions, maps each OpenSpec task to a capable implementation agent, and records only task references, owners, and execution status in the dashboard state. Specialist agents implement the assigned tasks. The integration lead coordinates interfaces between frontend, backend, data, platform, and AI components, then brings the changes together and runs the app and relevant checks. Mark tasks complete in the OpenSpec task file only after the assigned work is integrated and verified. Factory review rounds add feedback; they do not replace or duplicate OpenSpec tasks.

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

Do not add `factory/backlog.md`, `factory/jobs/<change>/spec.md`, or a second implementation checklist. New work is proposed through OpenSpec. `/factory next` selects an existing active OpenSpec change with unfinished tasks and dispatches those tasks to implementation subagents, using the configured OpenSpec apply instructions as the contract for context and task tracking.

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

Each agent reads its context from the OpenSpec change and repository files. Implementation agents change only the files or components assigned to them and report the OpenSpec task IDs they completed. The integration lead owns shared contracts, conflict resolution, end-to-end wiring, and final checks. The orchestrator alone updates `factory/board.json` and the OpenSpec task checkboxes, after verifying integrated work. Reviewers write only their own review file; the approver writes only `decision.md`.

## Implementation team

Create a small set of reusable specialist subagents based on the stack and repository. Do not spawn every role for every change. The orchestrator selects only the roles needed by the OpenSpec tasks; it may use a generalist implementation agent when the change does not need a specialist.

Typical roles include:

- **Integration lead**: owns the feature branch, task dispatch, API and data contracts, shared-file coordination, integration, and end-to-end validation. This can be the orchestrating session if that is safer and simpler.
- **Frontend**: web or mobile UI, client state, routing, accessibility, and frontend tests.
- **Backend/API**: services, endpoints, business logic, authorization, and API tests.
- **Data/platform**: schemas, migrations, persistence, infrastructure, deployment, and observability.
- **AI/agent systems**: model and provider adapters, prompts, tool definitions, orchestration, memory or retrieval, structured outputs, evaluations, tracing, and failure handling.
- **QA/reliability**: test strategy, regression coverage, integration scenarios, and reliability checks.

For AI or agent features, explicitly consider evaluation datasets and success criteria, tool permissions and input validation, prompt injection boundaries, retries and timeouts, token/cost limits, structured output validation, observability, and graceful behavior when a model or tool fails. Add only the safeguards and evaluations relevant to the OpenSpec requirements and the existing product architecture.

Split work by OpenSpec task IDs and clear file/component ownership. Parallelize only tasks whose dependencies and write sets do not conflict. If the environment supports isolated worktrees or equivalent isolation, use them for conflicting work and integrate the results. Otherwise, assign non-overlapping files and have the integration lead sequence shared-file changes. Agents must agree on interfaces before parallel work begins. Keep the task IDs as references in `board.json`; never turn that state into a second editable checklist.

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

The tasks in `tasks.md` are the implementation checklist. Do not copy them into a factory-owned file. The factory can display task IDs, assigned agents, and mirrored progress, but OpenSpec remains the only place where task completion is recorded. Update its checkboxes after implementation is integrated and verified, following the OpenSpec workflow.

## Rules

- Inspect the repo and its OpenSpec setup before creating or changing a change proposal.
- OpenSpec artifacts are the source of truth for proposal, requirements, design, and implementation tasks. Never create duplicate factory specs, backlogs, or task lists.
- Use the configured OpenSpec schema and installed workflow. Follow its artifact instructions and validation; do not hard-code assumptions about optional artifacts.
- Review the proposal, specs, design when present, and tasks before implementation. Do not dispatch implementation agents until the plan is accepted for implementation.
- Every implementation task must be assigned to an accountable specialist agent or the integration lead. Agents get context from OpenSpec and repository files, not undocumented conversation context.
- Define interfaces and file ownership before parallel work. Avoid overlapping edits; sequence dependent work and integrate subagent changes before marking OpenSpec tasks complete.
- Reviewers write only their own review file; the approver writes only `decision.md`. Implementation agents change assigned code and report results; they do not edit the dashboard or mark OpenSpec tasks complete.
- Each review starts with `VERDICT: PASS` or `CHANGES`; `decision.md` starts with `APPROVE` or `ESCALATE`.
- The integration lead creates or uses a feature branch (`factory/<change-name>`). Implementation subagents work only on their assigned OpenSpec task IDs and owned files/components.
- Validate the full stack across its boundaries: for example, client-to-API behavior, API-to-data behavior, migrations, auth, and the built application. Use the repository's documented test command and the OpenSpec acceptance scenarios.
- For AI/agent changes, validate the specified behavior with relevant deterministic tests or evaluations, including failure paths and tool boundaries where applicable. Do not treat a successful model response in one example as sufficient coverage.
- The orchestrator updates OpenSpec task checkboxes only after integration and verification. Never mark a task complete unless its work is done.
- WAIT FOR ALL: a review round is done only when every applicable reviewer's file exists in that round folder. Never start a rework cycle mid-round.
- Give each implementation agent a clear name tied to the change and role (for example, `frontend-<change-name>` or `ai-systems-<change-name>`). Keep the same agents for follow-up fixes when possible, and route each review finding to the agent that owns the affected component.
- Allow at most two rework rounds. If issues remain, escalate to a human.
- After all reviews pass, verify that the implementation matches the OpenSpec requirements and all applicable tasks are complete. Do not archive the OpenSpec change until implementation and reviews are complete; use the configured OpenSpec archive workflow so spec deltas are synchronized correctly.
- Only `/factory approve <change-name>` may merge a job in `needs-human` into `main`.

## Commands

```text
/factory <feature>                   create or continue an OpenSpec change, then dispatch its tasks to the factory agents
/factory next                         dispatch unfinished tasks from the first eligible active OpenSpec change
/factory dashboard                   start and open the live factory Kanban dashboard
/factory approve <change-name>        merge a needs-human job into main
/factory rework <change-name> <note>  route feedback to the implementation agent(s) that own the affected tasks
```

`/factory dashboard` starts the dashboard's local server using the repository's supported mechanism and opens the dashboard in the available browser. If automatic browser opening is unavailable, print the local address and the one-step command to open it. It must not start duplicate servers when one is already running. The dashboard is read-only: links may open change artifacts, but dashboard interaction must not silently change OpenSpec files or approve, archive, or merge changes.

Use the actual installed OpenSpec skills or CLI commands for proposing, applying, verifying, validating, and archiving. If the requested factory command would conflict with an active OpenSpec change, show the matching changes and ask which one to use rather than creating a duplicate.

## Dashboard

Build a Kanban-style dashboard in `factory/dashboard.html` that shows both live active changes and past archived changes. Use columns that match the implemented factory flow, such as Planning, Ready, Building, Review, Needs human, and Done/Archived. Each active card shows its OpenSpec change name, current stage, task progress, assigned implementation agents, and applicable review verdicts. Archived changes appear in a past-changes view or the Done/Archived column, with a way to inspect their proposal, completed task list, reviews, and decision.

The dashboard must refresh automatically while open so changes made by `/factory` appear without a manual reload. Use a lightweight polling or file-watch mechanism compatible with the repository's environment. The orchestrator updates `factory/board.json` as a display snapshot when stages, agent assignments, or review verdicts change; it also refreshes the active and archived change index from OpenSpec. OpenSpec artifacts remain authoritative for requirements, task completion, and archive status. The dashboard must identify stale or unavailable data rather than presenting an old snapshot as current.

`/factory dashboard` starts the local dashboard server if needed and opens it in the available browser. Keep server startup idempotent and document how to stop it. If the environment cannot open a browser, return the local URL. Keep the dashboard read-only: it may link to Markdown artifacts but must not edit, approve, archive, or merge changes. Do not expose the local server to the public network. Keep the UI clean, minimal, and usable for both a quick status check and browsing past work.
