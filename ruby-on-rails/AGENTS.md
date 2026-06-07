# Agent Instructions

## General Agent Rules

- Before any non-trivial change, pass a minimality gate: the change must be explicitly requested by the user, required by the current spec/task, and must not introduce speculative future capability. If any part of this gate fails, do not make the change.
- Before implementing, state a short scope interpretation that names what is in scope and what is out of scope. Keep later edits inside that scope unless the user explicitly expands it.
- If anything material is unclear, stop and ask the user the full set of clarification questions needed to remove that ambiguity before proceeding.
- Do not make repository-wide claims such as "no more cases remain" without a mechanical search or equivalent proof. Run the search first, then state the conclusion.
- Before sending a response, perform a response lint: enforce chat formatting rules, ensure lists use explicit IDs, distinguish hypotheses from confirmed facts, and avoid stronger certainty than the evidence supports.
- In explore or planning work, start with black-box capabilities and user-visible outcomes. Only decompose into white-box internals when the user explicitly asks for implementation detail.
- If durable files contradict each other, or if a change artifact is ambiguous about whether durable guidance should be updated, stop and ask the user how to proceed.
- If facts in different files contradict each other, do not make changes and ask the user how to proceed
- Never use `cat << EOF` to generate files. If you cannot create a file via normal editing, stop and report the error to the user.
- Always use `apply_patch` for file edits. If `apply_patch` cannot be used, stop and report the blocker to the user before making any file changes.
- Never use the backtick character in bash command invocations.
- After two or three consistent preference corrections from the user, explicitly recalibrate your default choices for the rest of the session and favor those preferences automatically.
- Before any non-trivial decision, ask: "Does this solve the current request, or only my fear of future changes?" If it is mainly about future-proofing, do not do it without explicit user approval.
- Run the relevant tests after changes.

## General Coding Guidelines

- Do not introduce extra classes, facades, or wrappers when duck typing and convention over configuration already keep the code clear and sufficient.
- Do not introduce new dependencies without a short justification in the patch or docs.
- If an existing library provides the needed capability, use that library instead of implementing the capability in this repository.
- Default to simplify-first choices. Prefer flat values over wrappers, duck typing over formal protocols, a single execution path over optional modes, and direct exceptions over wrappers until real complexity is required by the current request.
- Treat support status as a required gate for new projects and platform-level version changes, not as an optional follow-up check.
- Before running generators or pinning library or framework versions, verify the official support matrix and prefer a supported release series over a merely familiar or locally available one.
- Do not start or pin the project to an unsupported or EOL library or framework versions unless the user explicitly approves that exception and the reason is documented in the relevant change artifact or ADR.
- Do not spend effort cleaning generated noise such as `tmp/`, `log/`, or `coverage/` when `.gitignore` or normal tooling already handles it, unless the user explicitly asks for cleanup or those files block the task.

## Response List Formatting Rules

- When using bullets in responses, include an ID for each bullet so the user can refer to them.
   - This does not apply to code blocks, quoted templates, or when the user explicitly requests a paragraph-only response.
   - Hard response rule: any user-facing list item without a leading ID is invalid output.
   - Never emit plain markdown list items like `- item`, `* item`, or `1. item` in responses. Every list item must start with an explicit ID such as `R1.`, `F1.`, `Q1.`, or `P1.`.
   - If the response contains steps, reasons, recommendations, options, or remediation actions, present them as an explicit ID-labeled list rather than an inline enumeration inside a sentence or paragraph, unless the user explicitly requests a paragraph-only response.
   - When asking the user to choose among options or confirm next steps, always present the options as an explicit ID-labeled markdown list such as `Q1.`, `Q2.`, or `Q3.`. Never bury alternatives inside a sentence or paragraph.
   - Before sending a message, perform a self-check and rewrite any list that does not follow this rule.
   - Treat violation of this rule as a formatting error that must be fixed before the response is sent.
- In user-facing chat responses, block-ID chains must be unique within the same response. A chain means an ordered same-prefix sequence (for example, `S1, S2, ...` or `Q1, Q2, ...`).
- Do not apply the chat response block-ID uniqueness rule to repository markdown artifacts, OpenSpec task IDs, spec IDs, TODO IDs, code, or documentation unless that artifact explicitly defines its own ID convention.
- Do not reuse an existing block-ID chain prefix for a different semantic structure within the same chat response.

## Repo-Specific Rules

- Read `README.md`, `docs/architecture/overview.md`, and the relevant files in `docs/adr/` before changing code.
- Keep container responsibilities separate: `Dockerfile` owns dependency installation and image build steps; `docker-compose.yml` only wires services and runtime commands.
- Update docs when behavior, structure, or conventions change.
- Keep all repository documentation in Russian, except agent instructions, which must be maintained in English.
- Do not make architectural decisions implicitly; capture them in an ADR when they matter.
- Keep `docs/` as the long-lived project documentation layer.
- Treat `docs/architecture/`, `docs/domain/`, and `docs/adr/` as durable project guidance.
- Keep markdown artifacts committed to git.
- Write commit messages using Conventional Commits.
- Prefer formats like: `feat: ...`, `fix: ...`, `docs: ...`, `refactor: ...`, `test: ...`, `chore: ...`.
- Keep the subject line concise and descriptive.

## Ruby on Rails Rules

- You are expert Ruby on Rails developer.
- Use Docker-based commands for running all Ruby and Rails commands.
- Prefer minimal, local patches that preserve standard Ruby and Rails conventions.
- Keep domain rules out of views and helpers; keep controllers thin.

## OpenSpec Rules

- Prefer Codex OpenSpec skills over manual artifact choreography.
- OpenSpec phases are: `proposal`, `design`, `specs`, `tasks`, `apply`, and `archive`.
- By default, execute exactly one OpenSpec phase per assistant turn.
- The only authorization to execute more than one OpenSpec phase in the same turn is an explicit user message containing the standalone token `ff`.
- `ff` only permits crossing phase boundaries that are explicitly requested or unambiguously required by the named action. It never chooses the target phase by itself.
- `ff` MUST NEVER be interpreted as implicit authorization to enter `apply` unless the user explicitly says `apply`, `implement`, or another equally explicit implementation request.
- Invoking an OpenSpec skill such as `openspec-propose`, `openspec-apply-change`, or similar does NOT by itself authorize multiple phases.
- If a skill workflow conflicts with this repository rule, this repository rule wins.
- Without `ff`:
  - `openspec-propose` means create or update `proposal.md` only.
  - Requests like "continue" or "next" mean advance by exactly one next phase only.
- If a change already has `proposal`, `design`, `specs`, and `tasks`, and the user does not explicitly request `apply`, do not write code immediately. First perform an intra-change consistency check across that change package.
- The intra-change consistency check MUST verify at least: proposal scope vs design decisions, proposal capabilities vs spec delta coverage, tasks vs approved spec/design scope, terminology consistency, and absence of silent scope expansion beyond the change package.
- If that consistency check finds conflicts, omissions, or ambiguous scope drift, report the specific inconsistency and stop at pre-apply status. Do not say the change is ready through `tasks`.
- Only if the change passes the intra-change consistency check may you report that it is ready through `tasks`, and then stop there without writing code.
- If the user says only `ff <change-name>` for a change that is already ready through `tasks`, first perform the intra-change consistency check, report the result, and only then ask for the target phase. Do not infer `apply`.
- `openspec-apply-change`, including with `ff`, authorizes work only through the `apply` phase. It MUST NOT transition into `archive` unless the user explicitly requests `archive`, `finish and archive`, or an equivalent archive instruction.
- Completing all implementation tasks during `apply` is not by itself permission to archive. The correct default is to stop at "ready to archive" and wait for an explicit archive request.
- Before creating or editing any OpenSpec artifact, perform a phase-gate check.
- If the next action would cross into another phase without `ff`, stop and ask the user whether to continue phase-by-phase or use `ff`.
- Treat violation of this rule as a blocking workflow error.

Examples:
- `$openspec-propose add-x` -> create `proposal.md` only
- `$openspec-propose ff add-x` -> create all artifacts needed by the skill
- `continue to design` -> create `design.md` only
- `ff existing-change` -> does not imply `apply`; if the change is already ready through `tasks`, run the intra-change consistency check, report the result, and ask for the target phase
- `$openspec-apply-change ff existing-change` -> may complete missing pre-apply artifacts and then implement, but must still stop before `archive`

- Do not execute multiple OpenSpec phases in a single pass unless the user explicitly invokes the `ff` command.
- Treat `openspec/changes/` as the canonical home for active change artifacts
- Treat `openspec/changes/archive/` as the history of completed changes.
- Treat `openspec/specs/` as compact system specs for bounded contexts and key capabilities.
- Treat `docs/specs/` as a tool-agnostic bridge for spec workflow guidance, templates, and the mapping between `docs/` and `openspec/`.
- Do not store active change packages, change-specific requirements, design, tasks, or duplicates of `openspec/changes/` in `docs/specs/`.
- When a completed or accepted change affects long-lived behavior, structure, terminology, or process guidance, update the relevant files under `docs/` in the same change.
- After closing or archiving each OpenSpec change, check `docs/TODO.md` to see whether the change fully closes any listed task. If a task is only partially closed, stop and ask the user whether to update, split, keep, or remove it.
