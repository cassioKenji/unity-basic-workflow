# Claude Instructions for solo game developing

Projeto Unity 6 e C#. Siga todas as regras abaixo ao gerar qualquer código.

## Sempre avalie game patterns para resolver problemas conhecidos
- Observer Pattern
- Singleton
- State Pattern
- Object Pooling Pattern
- Factory Pattern


## Estrutura de documentação

@docs/README.md


# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative. This is a solo dev project, do not overengineering things**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

# Hard Constraints (never-break rules)

## Data Safety
- Never delete production data without explicit user confirmation in the same message
- Never store secrets (API keys, passwords, tokens) in code files or commit them
- Never run DROP, TRUNCATE, or DELETE without WHERE on production databases

## Code Safety
- Never catch errors silently (empty catch blocks, swallowed promises)

## Process Safety
- Never force-push to main/master
- Never skip pre-commit hooks (no --no-verify)
- Never amend a commit that has been pushed to a shared branch

## Scope Safety
- Never modify files outside the directories specified in the current task
- Never add dependencies without stating the reason and checking bundle size impact
- Never refactor code that isn't part of the current task (note it for later instead)

# Anti-Patterns to Detect

## Code Smells to Flag
- Feature envy: a function that references another module's internals more than its own
- Primitive obsession: passing >3 related primitives instead of a typed object
- Temporal coupling: functions that must be called in a specific order without enforcement

## Process Smells to Flag
- Yak shaving: spending >3 tool calls on something tangential to the task
- Shotgun surgery: a single change requiring edits in >5 files (suggests missing abstraction)
- Copy-paste programming: duplicating >5 lines instead of extracting a function

## AI-Specific Anti-Patterns
- Hallucinated APIs: calling a method that doesn't exist in the current version
- Stale context: referencing file contents from earlier in the conversation that may have changed
- Over-apology: spending tokens on apologies instead of fixing the issue

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

