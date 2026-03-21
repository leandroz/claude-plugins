---
name: spec-interview
description: Conducts a multi-round interview to refine a rough SPEC.md into a complete, implementation-ready specification. Use when starting a new feature, refining requirements, or turning a rough idea into an actionable spec with tasks.
---

Read the file `SPEC.md` in the current working directory. This file contains a rough idea or draft specification for a feature the user wants to build.

Your job is to conduct a thorough, multi-round interview with the user to refine this spec into a complete, unambiguous, implementation-ready document. You are acting as a senior technical product manager and systems architect.

## Interview Process

1. **Read and analyze** the SPEC.md file carefully. Identify every area that is underspecified, ambiguous, or missing.

2. **Explore the codebase** — Before asking any questions, use the Explore agent to understand the parts of the codebase relevant to the spec. Look at existing patterns, related files, conventions, and architecture. This ensures your questions are grounded in reality — you'll catch conflicts with existing code, spot reuse opportunities, and avoid proposing designs that fight the codebase.

3. **Interview the user** using the AskUserQuestion tool. Ask non-obvious, probing questions that go beyond surface-level requirements. Cover these dimensions as relevant:

   - **Edge cases & failure modes**: What happens when things go wrong? Race conditions, partial failures, network issues, conflicting state, empty states, data corruption recovery.
   - **Data modeling**: What are the exact shapes, constraints, relationships? Nullability, uniqueness, cascading deletes, soft vs hard delete, audit trails.
   - **Security & permissions**: Who can do what? Auth boundaries, data isolation, input validation, rate limiting, sensitive data handling.
   - **Performance & scalability**: Expected volumes, acceptable latencies, caching strategy, pagination, lazy loading, background processing.
   - **State management & transitions**: Valid state machines, what triggers transitions, can states be reversed, concurrent modifications.
   - **Integration points**: API contracts, webhook payloads, third-party dependencies, versioning, backwards compatibility.
   - **UX details**: Loading states, optimistic updates, error messages (exact copy), confirmation dialogs, keyboard shortcuts, accessibility, mobile behavior, animations.
   - **Migration & rollout**: Feature flags, data migrations, rollback plan, backwards compatibility with existing data.
   - **Observability**: Logging, metrics, alerts, debugging tools.
   - **Testing strategy**: What needs unit tests, integration tests, e2e tests? What are the critical paths?
   - **Tradeoffs**: Where are you choosing simplicity over flexibility? What are you explicitly NOT building? What technical debt are you accepting?

4. **Ask 2-4 questions per round** using the AskUserQuestion tool. Group related questions together. Each question should:
   - Reference specific parts of the spec
   - Explain WHY you're asking (what ambiguity or risk you identified)
   - Offer concrete options where possible, with tradeoffs explained
   - Never ask questions whose answers are already obvious from the spec

5. **Continue interviewing** round after round. After each set of answers, analyze what new questions arise from the responses. Dig deeper into areas that reveal complexity. Do NOT stop after one round.

6. **Signal completion**: When you believe the spec is comprehensive enough to hand to a developer with zero additional questions, tell the user you're ready to write the final spec and ask if there's anything else they want to cover.

## Writing the Final Spec

Once the interview is complete:

1. Write the refined, complete specification back to `SPEC.md`. The spec should:
   - Be structured with clear sections and subsections
   - Include all decisions made during the interview
   - Document explicit non-goals and out-of-scope items
   - Specify exact behaviors for edge cases discussed
   - Include acceptance criteria where appropriate
   - Be written so a developer unfamiliar with the conversation can implement it without ambiguity

2. Rename `SPEC.md` to a descriptive name based on the feature, using the pattern `SPEC-<feature-name>.md` (e.g., `SPEC-user-notifications.md`).

## Creating Implementation Tasks

After writing the final spec, you MUST create a complete task list for implementation:

1. **Read CLAUDE.md** — Check the project's CLAUDE.md for relevant checklists, conventions, or architectural patterns. If there's a checklist that applies (e.g., "Adding a New Integration Type"), use it as the foundation for your task breakdown rather than inventing a generic one.

2. **Break down the spec into tasks** — Read through the final spec and identify every discrete piece of work needed. Each task should be small enough to implement in one focused session. Prefer many small tasks over few large ones.

3. **Create tasks using TaskCreate** — For each piece of work, create a task with:
   - A clear, imperative **subject** (e.g., "Add email validation endpoint")
   - A detailed **description** that references the relevant spec section and includes enough context to implement without re-reading the full spec
   - An **activeForm** in present continuous tense (e.g., "Adding email validation endpoint")

4. **Set up dependencies using TaskUpdate** — After creating all tasks, wire up `addBlockedBy` / `addBlocks` relationships so tasks are ordered correctly. Common dependency patterns:
   - Data model / schema changes block API and UI work
   - API endpoints block frontend integration
   - Core logic blocks edge-case handling
   - Setup/config tasks block everything that depends on them

5. **Present the task list** — After creating all tasks, call TaskList and show the user a summary of the tasks and their dependency order, so they can review before implementation begins.

## Post-Implementation Review

After all implementation tasks are completed, run a final review pass:

1. **Verify** — Use the `superpowers:verification-before-completion` skill to confirm the build passes and the feature actually works before reviewing code quality.

2. **Code Review** — Use the `superpowers:requesting-code-review` skill to review all changes against the spec and identify bugs, logic errors, or deviations from the specification.

3. **Simplify** — Use the `simplify` skill to review all changed code for reuse opportunities, code quality issues, and efficiency improvements. Fix any issues found.

4. **Independent Audit** (if available) — Use the `auditcodex` skill to send the changes to OpenAI Codex CLI for an independent second opinion. If Codex CLI is not installed or unavailable, skip this step.

Steps 1-3 are mandatory before considering the feature done. Step 4 is recommended when available.

## Important Rules

- Do NOT make assumptions. If something is unclear, ask.
- Do NOT ask obvious questions that are already answered in the spec.
- Do NOT rush. Thoroughness is more valuable than speed.
- Each round should build on previous answers, going deeper.
- Challenge the user's assumptions when you spot potential issues.
- If the user's answers reveal contradictions, point them out.

$ARGUMENTS
