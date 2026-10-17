<!-- Shared guidelines for consistent agent behavior across codebases. -->

# Global Code Guidelines

## Communication

Be direct and concise. State results, decisions, tradeoffs, and problems without praise, filler, apologies, or conversation/progress narration.

## Thread and Workspace Routing

If a message appears intended for another chat, keep discovery read-only. Match paths, PRs, branches, or CI logs against `git worktree list`, then confirm ownership through chat metadata and recent context. A repository match alone is insufficient.

Warn the user promptly with the exact chat title, link if available, and supporting workspace/task evidence. State uncertainty or unavailable discovery rather than guessing. After warning, stop: do not act on the misrouted prompt or resume prior work in that turn.

If the user then says to continue, resume this thread’s work from before the misrouted prompt. “Continue” does not authorize accepting another thread’s task. Take on that task only when explicitly instructed to do so; do not change workspaces or edit another thread’s checkout on its behalf without that authorization. Forwarding requires explicit authorization.

## Project Context and Workflow

Before substantial planning or editing, read relevant project documentation. Follow repository conventions and more specific `AGENTS.md` instructions over these defaults. Where the project uses ADRs or research questions, consult and update relevant records using its significance criteria, formats, and lifecycle conventions. Record significant decisions in ADRs and unresolved questions/open work in research questions; skip routine implementation choices unless project criteria require a record. Skip unrelated documentation.

Use repository commands and this order:

```text
understand → edit → auto-fix/format → compile/test/lint → investigate remaining failures
```

Run configured formatting, import cleanup, lint fixes, and safe mechanical rewrites before manual debugging. Use format-all when customary; repository-wide formatting-only changes are allowed unless concurrency or documented constraints make them unsafe.

## Correctness

- Fix root causes, not symptoms.
- Handle errors explicitly and idiomatically, including library, network, and external-call failures. Verify libraries signal failure as expected; never disguise failure as a successful empty/null result or another value also representing success.
- Preserve meaningful distinctions between missing, null/nil, and empty values.
- Avoid non-trivial duplication; explain tradeoffs when duplication is preferable to reuse or refactoring.
- Prefer named structures when returned values have distinct meanings.
- Explain meaningful tradeoffs: what is gained and sacrificed.
- Build structured output with native data structures and serialization, not string concatenation.

## Long-Running Jobs

Design benchmark harnesses, batch runners, and other long-running parameter-matrix tools to resume from a specified cell or checkpoint without rerunning completed work. Persist progress and intermediate results as execution proceeds so they remain available for inspection after interruption or failure. Record parameter combinations and completion status explicitly to distinguish complete, partial, and failed cells.

## Tools and Shell

Use project linters, type checkers, static analyzers, and language-aware tools to catch errors statically. Explicitly search the full tool inventory for task-relevant capabilities, including deferred tools; tools visible in context are not the complete inventory. Reuse explicit discovery results within the task unless availability changes. For code edits, discover the semantic editor (`semedit`) and use its callable tools where supported. Fall back to ordinary editing only when no applicable semantic tool is callable, or it fails, is unsupported, or is ambiguous.

Prefer LSP for navigation/refactoring/diagnostics, AST tools for structural changes outside semantic-editor coverage, and `rg`/`fd` for text/file searches. Use text replacement only when syntax structure is irrelevant. Retrieve only needed information.

Write shell, Makefiles, and CI defensively: quote paths/expansions, preserve argument boundaries, handle arbitrary filenames and option boundaries, and fail explicitly on errors.

Consult these files if present, only for the relevant work:

- `~/.codex/docs/TOOLING.md`: tooling selection, installation, replacement, or workflow setup; skip for established routine workflows.
- `~/.codex/docs/SHELL.md`: non-trivial shell implementation.
- `~/.codex/RTK.md`: Codex/RTK workflows.

## Testing

Test observable behavior, not private internals. Avoid shallow mocks that let broken behavior pass; prefer fewer high-confidence tests.

Use property-based tests where useful for behavioral invariants across varied inputs. Name or describe properties using framework support so failures identify the violated behavior.

Reuse expensive setup and execution across related checks. Group assertions against shared setup/results into individually named checks, such as Go `t.Run` subtests or JUnit 5 `assertAll` with descriptive messages. Isolate setups when state, lifecycle, or failure behavior requires it.

Unless instructed otherwise, use nonfatal assertions to collect independent failures. Stop early only when a failed prerequisite invalidates subsequent checks or makes them unsafe.

## Comments

Every file needs a brief opening comment explaining its purpose, why it belongs there, and why it contains what it does. Document every function’s purpose and reason to exist using the language’s standard documentation format where available.

Purpose documentation must be meaningful, never boilerplate or a restatement of names/implementation. If a function lacks an explainable purpose, reconsider its design or necessity: clarify, reshape, inline, or remove it rather than inventing justification.

Otherwise, comment only non-obvious constraints, invariants, workarounds, or surprising behavior. Explain why, not what. Keep comments short; no decorative banners or em dashes (`—`).

## Source Control and Vendor Neutrality

Never add AI assistants as co-authors or other source-control metadata.

Do not introduce, recommend, or default to vendors/providers/services unless selected by the project or user. Prefer capability-based names except in intentionally provider-specific code.

## Text Diagrams

Use aligned Unicode box-drawing characters (`┌ ┐ └ ┘ ─ │`) with continuous walls and outer-container labels in the top edge.

## Model Usage and Delegation

- Default: GPT-6 Luna (`gpt-6-luna`) for routine, well-scoped work with clear requirements and easy verification.
- Sol always means GPT-6.1 Sol (`gpt-6.1-sol`), including workers/reviews; never GPT-6 Sol (`gpt-6-sol`). Never use any GPT-5.6 model. If a required model is unavailable, report it and ask how to proceed.
- Assess reasoning risks before substantive work; keep the assessment internal unless it changes the approach. Use Sol when Luna struggles, retries, or is uncertain, or proactively for subtle external semantics, contracts/lifecycles, concurrency/security, unclear integration boundaries, or false-confidence risks. For tests, escalate when determining the property or validating the test requires stronger reasoning; routine property-based tests alone do not warrant escalation.
- Delegate only bounded work with material benefit; use a focused Sol worker when an independent semantic check is needed. Do not create workers solely to satisfy the model hierarchy when the active model can resolve the question directly. Supply the question, constraints, relevant repository state/files, and prior findings, not accumulated history. Return routine work to Luna after resolving the difficult question.
- Start reasoning effort low for routine work, medium for substantive reasoning; increase only as complexity warrants. Size alone does not justify higher effort or Astra.

### Astra Permission Gate

Every GPT-6 Astra (`gpt-6-astra`) use, including workers/reviews, requires fresh explicit user permission. Request approval for the exact question, why Sol is insufficient, the scope, and creation of a separate thread. Wait for affirmative authorization naming Astra for that use. General task approval, urgency, complexity, permission for another use, or “continue” never suffice.

Run approved work only in a fresh, separate thread: never switch the current thread, fork full history, or reuse accumulated Astra context. If unavailable, report the limitation without invoking Astra.

Start every Astra task with an explicit, self-contained handover containing:

- User objective, exact question, authorized scope/limits, constraints, and decisions.
- Repository path, worktree, branch, and revision where applicable.
- Necessary files/excerpts and their relevance.
- Prior attempts, evidence, failures, and uncertainties; distinguish facts from assumptions.
- Expected output, acceptance criteria, and edit versus analysis-only permission.

Keep context focused but complete; assume no access to the original conversation. Retrieve concise conclusions, evidence, and unresolved issues into the original thread. Resume on Luna or GPT-6.1 Sol; further Astra work requires fresh permission.
