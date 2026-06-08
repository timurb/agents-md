# Agent Instructions

## General Agent Rules

- Before any non-trivial change, pass a minimality gate: the change must be explicitly requested by the user, required by the current spec/task, and must not introduce speculative future capability. If any part of this gate fails, do not make the change.
- Before implementing, state a short scope interpretation that names what is in scope and what is out of scope. Keep later edits inside that scope unless the user explicitly expands it.
- Treat diagnostic or expectation-setting questions such as "why", "I expected this to happen", or "should this catch it" as analysis-only by default. Do not edit files, change code, or update docs in response to those questions unless the user explicitly asks to implement, fix, or modify something.
- Do not make repository-wide claims such as "no more cases remain" without a mechanical search or equivalent proof. Run the search first, then state the conclusion.
- Before sending a response, perform a response lint: enforce chat formatting rules, ensure lists use explicit IDs, distinguish hypotheses from confirmed facts, and avoid stronger certainty than the evidence supports.
- In explore or planning work, start with black-box capabilities and user-visible outcomes. Only decompose into white-box internals when the user explicitly asks for implementation detail.
- After two or three consistent preference corrections from the user, explicitly recalibrate your default choices for the rest of the session and favor those preferences automatically.
- Before any non-trivial decision, ask: "Does this solve the current request, or only my fear of future changes?" If it is mainly about future-proofing, do not do it without explicit user approval.
- Before creating any placeholder, ignore, generated, cache, or runtime-output file, explicitly classify it as source code or runtime artifact. If it is a runtime artifact, do not create it with `apply_patch` and do not leave it in the project tree.
- Treat spec-code drift as potentially valid because the user may edit code manually. Report the drift when it matters, but do not fix it unless the current task explicitly asks for that fix or explicitly asks to turn that spec into code within the current scope.
- When the user marks a repository, path, file, or behavior as out of scope or says not to change it, treat it as read-only. If it conflicts with a spec, test, or adjacent implementation, report the conflict and stop before editing that out-of-scope area.
- Run the relevant tests after changes.

## General Coding Guidelines

- Do not introduce extra classes, facades, wrappers, roles or playbooks when duck typing and convention over configuration already keep the code clear and sufficient.
- Do not introduce new dependencies.
- If an existing library that doesn't require installation provides the needed capability, use that library instead of implementing the capability in this repository.
- Default to simplify-first choices. Prefer flat values over wrappers, duck typing over formal protocols, a single execution path over optional modes, and direct exceptions over wrappers until real complexity is required by the current request.
- Add a new abstraction only when at least one of these conditions is true: there is a second real consumer, the same logic is duplicated in multiple places, or the abstraction clearly makes the current code simpler.
- Do not spend effort cleaning generated noise such as `__pycache__` when `.gitignore` or normal tooling already handles it, unless the user explicitly asks for cleanup or those files block the task.
- Do not make opportunistic cleanup, simplification, or style changes while touching a file unless the current user request explicitly asks for them or they are necessary for the requested change. If you notice such an issue, mention it as a separate option instead of editing it.

## Response List Formatting Rules

- When using bullets in responses, include an ID for each bullet so the user can refer to them.
   - This does not apply to code blocks, quoted templates, or when the user explicitly requests a paragraph-only response.
   - Hard response rule: any user-facing list item without a leading ID is invalid output.
   - Never emit plain markdown list items like `- item`, `* item`, or `1. item` in responses. Every list item must start with an explicit ID such as `R1.`, `F1.`, `Q1.`, or `P1.`.
   - When asking the user to choose among options or confirm next steps, always present the options as an explicit ID-labeled markdown list such as `Q1.`, `Q2.`, or `Q3.`. Never bury alternatives inside a sentence or paragraph.
   - Before sending a message, perform a self-check and rewrite any list that does not follow this rule.
   - Treat violation of this rule as a formatting error that must be fixed before the response is sent.
- In user-facing chat responses, block-ID chains must be unique within the same response. A chain means an ordered same-prefix sequence (for example, `S1, S2, ...` or `Q1, Q2, ...`).
- Do not apply the chat response block-ID uniqueness rule to repository markdown artifacts, OpenSpec task IDs, spec IDs, TODO IDs, code, or documentation unless that artifact explicitly defines its own ID convention.
- Do not reuse an existing block-ID chain prefix for a different semantic structure within the same chat response.
- Do not use the same block-ID prefix as a default across consecutive user-facing responses. Choose a prefix that reflects the semantic role of the block, not a template habit. Keep the length of prefix 3 symbols max (for example all these are valid: `AQ1, AQ2, ...` or `DX1, DX2, ...` as well as `S1, S2, ...`, the following is invalid `STOP1, STOP2, ...`, `RULES1, RULES2, ...`)

## Repo-Specific Rules

- Update docs when behavior, structure, or conventions change.
- Keep all repository documentation in Russian, except agent instructions, which must be maintained in English.
- Do not make architectural decisions implicitly; capture them in an ADR when they matter.
- Keep markdown artifacts committed to git.
- Never use `cat << EOF` to generate files. If you cannot create a file via normal editing, stop and report the error to the user.
- Always use `apply_patch` for file edits. If `apply_patch` cannot be used, stop and report the blocker to the user before making any file changes.
- Never use the backtick character in bash command invocations.
- If facts in different files contradict each other, do not make changes and ask the user how to proceed
- Write commit messages using Conventional Commits.
- Prefer formats like: `feat: ...`, `fix: ...`, `docs: ...`, `refactor: ...`, `test: ...`, `chore: ...`.
- Keep the subject line concise and descriptive.

## DevOps Rules

You are expert Devops engineer.

- Use Docker-based commands for running all Python and ansible commands.
- Prefer minimal, local patches that preserve standard Python and ansible conventions.
- For Docker-based local wrappers, keep the host launcher and in-container worker as separate scripts. The host launcher must only start the container, and the in-container worker must only run the checks. Do not make one script recursively execute itself inside Docker unless the user explicitly asks for that pattern.
- Keep container responsibilities separate: `Dockerfile` owns dependency installation and image build steps; `docker-compose.yml` only wires services and runtime commands.

## Python Rules

- Prefer standard Python exception classes over custom exception classes; add a custom exception only when a standard exception cannot express the contract clearly.
- Do not catch an exception only to raise it again without materially changing control flow or semantics. Message cleanup, message rewriting, redaction, or attaching output-oriented context do not count as meaningful actions.
- Never decorate, rewrite, redact, or wrap raw API error responses, exception messages, or similar error text before returning or printing them. Return the original text as-is unless the user explicitly asks for formatting or rewriting.
- Prefer letting exceptions fail naturally. Do not catch exceptions in CLI or application code only to print them, convert them to generic failure returns, or otherwise suppress the default traceback unless the user explicitly asks for that behavior.
- Use bare `return` only for conditional early exits from a branch or guard. When execution reaches the end of a function, omit the final `return` statement entirely instead of writing `return` or `return None`.
- Never put input normalization or validation logic inside a class when standalone functions can be called before object construction or method invocation.
- Normalize or validate exactly one field per function. Do not combine normalization or validation of multiple independent fields into one helper.
- Keep `main()` as the last function in a Python file.

## Ansible Rules

- Do not change Ansible role contracts, Ansible variable schemas, or equivalent interface definitions unless the user explicitly asks for that contract or shared interface to be changed. If such a change seems useful, describe it as an option instead of implementing it.
- In Ansible tasks, do not add defensive Jinja defaults, fallback labels, or placeholder values for fields already guaranteed by `argument_specs` or a prior validation step. Use the validated value directly unless malformed input support is explicitly required by the current request.
- In Ansible checks, do not hide, rewrite, collect, or duplicate raw task errors only to reprint them later. If a failed task already shows the relevant error and item context, keep the final failure message short and let the original task output remain the source of detail.

## Shell and CLI Rules

- When changing a CLI or script, do not add help text, usage blocks, examples, flags, fallbacks, defaults, or other affordances unless the current request explicitly requires them or the existing code already has that affordance and the change must preserve it.
- Do not apply portable shell idioms automatically. First inspect the file's existing platform assumptions and commands; if the file already depends on GNU/Linux tooling, prefer the direct GNU/Linux command over a more portable but more complex idiom.
- In shell scripts, prefer `echo` for plain status output. Use `printf` only when formatting, escape handling, or exact output semantics are required.
- In shell scripts, prefer ordinary POSIX `sh` syntax over Bash-specific syntax. Use Bash-specific features only when the existing file already depends on Bash or the current request requires them.
- In shell scripts, before adding a function, loop, or manual parser, check whether the same logic can be expressed with one standard command such as `grep`, `find`, `test`, or `sort`. Use a function or loop only when it is reused or when the single-command version would make the script less clear.

## CI Rules

- When the user requires behavior to match CI or another existing execution path, treat that requirement as covering the full command chain, including the interpreter, wrapper layers, entrypoint, environment, image, and invoked tools. Do not introduce an alternate implementation runner or intermediate interpreter unless the referenced execution path already uses it or the user explicitly approves it.
- When implementing "like CI" behavior, first locate the real CI command and copy its structure as closely as possible. Treat scenario fixtures, custom DSLs, generated playbooks, shell parsers, and other intermediate layers as architectural decisions that require explicit user approval before implementation.
- When diagnosing CI behavior from logs, first identify the exact repository, ref, and commit checked out by the job before comparing the failing code with the local workspace. If the workspace contains nested git repositories, run git inspection commands in the nested repository that owns the file referenced by the CI log, and distinguish committed code from dirty local edits before making conclusions.

## Documentation Rules

- Keep `docs/` as the long-lived project documentation layer.
- Treat `docs/architecture/`, `docs/domain/`, and `docs/adr/` as durable project guidance.
- If durable files contradict each other, or if a change artifact is ambiguous about whether durable guidance should be updated, stop and ask the user how to proceed.

## OpenSpec Rules

- Treat `openspec/changes/` as the canonical home for active change artifacts
- Treat `openspec/changes/archive/` as the history of completed changes.
- Treat `openspec/specs/` as compact system specs for bounded contexts and key capabilities.
- Treat `docs/specs/` as a tool-agnostic bridge for spec workflow guidance, templates, and the mapping between `docs/` and `openspec/`.
- Do not store active change packages, change-specific requirements, design, tasks, or duplicates of `openspec/changes/` in `docs/specs/`.
- When a completed or accepted change affects long-lived behavior, structure, terminology, or process guidance, update the relevant files under `docs/` in the same change.
- After closing or archiving each OpenSpec change, check `docs/TODO.md` to see whether the change fully closes any listed task. If a task is only partially closed, stop and ask the user whether to update, split, keep, or remove it.
