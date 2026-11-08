---
name: code-review
description: Perform a rigorous maintainability review of the current branch, prioritizing structural simplification, sound abstractions, and behavior-preserving improvements.
disable-model-invocation: true
---

# Code Review

## Objective

Review the current branch's changes for correctness and long-term maintainability. Look beyond local cleanup: identify the simplest design that preserves intended behavior.

Prefer eliminating unnecessary concepts, branches, layers, and special cases over merely moving, renaming, or centralizing them. Be demanding, evidence-based, and actionable, but keep every finding proportional to its demonstrated impact.

## Review Principles

1. **Simplify the underlying model first.**
   - Look for a simpler model or flow that removes entire branches, helpers, modes, states, or layers.
   - Prioritize simplifying the underlying model over reorganizing its implementation.
   - Prefer eliminating the source of complexity over centralizing, extracting, or relocating it.
   - When a change introduces complexity, first question whether the requirement or design can be expressed more simply before adding another abstraction.
   - Do not settle for cosmetic cleanup when a meaningful, behavior-preserving redesign is practical.

2. **Preserve behavior.**
   - Treat existing behavior and the change's intended behavior as constraints.
   - Do not recommend a structural change without considering its behavioral impact.
   - Call out behavior changes separately from maintainability concerns.
   - Verify that a proposed simplification preserves relevant edge cases, contracts, and externally observable behavior.

3. **Keep logic in the right place.**
   - Put feature logic in the module or layer that owns the concept.
   - Reuse canonical utilities and established patterns when they are genuinely the right fit.
   - Flag duplicated logic, boundary leaks, and bespoke alternatives to existing helpers.
   - Do not move logic into a shared layer merely to centralize it if doing so weakens ownership or spreads feature-specific concerns.

4. **Prefer explicit, direct designs.**
   - Favor boring, readable control flow over magic, hidden fallback behavior, and ad-hoc special cases.
   - Question abstractions, wrappers, flags, optional parameters, casts, `any`, and `unknown` when they obscure rather than clarify the contract.
   - Introduce an abstraction only when it meaningfully reduces coupling, repetition, or total complexity.
   - Do not introduce abstractions, patterns, or extensibility mechanisms for hypothetical future requirements.
   - Do not replace straightforward code with elaborate patterns unless the resulting design materially reduces complexity.
   - Evaluate the total complexity of a proposed design, including new abstractions, files, dependencies, indirection, and navigation cost.

5. **Control complexity and size.**
   - Treat scattered conditionals and one-off branches as design problems, not isolated style issues.
   - Strongly challenge a change that takes a file from below 1,000 lines to above 1,000 lines. Recommend decomposition unless there is a compelling structural reason.
   - Treat 1000 lines as a warning threshold, not an absolute limit or proof that a file is poorly designed.
   - Also flag substantial file growth below that threshold when it makes a module materially harder to understand or maintain.
   - Do not split files mechanically: decomposition is valuable only when it creates cohesive boundaries and reduces overall comprehension cost.
   - Do not optimize micro-details at the expense of clarity.

6. **Keep orchestration simple and state consistent.**
   - Identify independent work that is needlessly serialized when parallel execution would simplify the flow.
   - Prefer atomic updates when partial state would create inconsistent or difficult-to-recover outcomes.
   - Raise these issues when they materially affect clarity, reliability, or maintainability; do not flag them as speculative micro-optimizations.

## Review Process

1. **Understand the change.** Identify its purpose, affected paths, intended behavior, and relevant existing architecture.
2. **Establish review scope.** Distinguish issues introduced by the change from pre-existing issues. Inspect surrounding code when needed to understand behavior and architecture, but do not attribute unrelated existing problems to the change. Recommend changes outside the diff only when they are necessary to resolve a concrete issue introduced or materially affected by the change.
3. **Trace the design.** Follow the main data and control flows. Check how the change interacts with existing abstractions, contracts, and callers.
4. **Search for simpler alternatives.** Ask whether a clearer model can remove branches, state, indirection, duplication, or special cases. Prefer reducing the source of complexity over moving it elsewhere.
5. **Verify material concerns.** Inspect the relevant implementation and tests. Run appropriate checks when available; never imply a check was run if it was not. Verify the causal link between the implementation and each suspected issue.
6. **Report actionable findings.** Prioritize concrete problems and propose the smallest structural change that resolves each one while preserving intended behavior.

## Finding Criteria

Report a finding only when there is a concrete, meaningful issue, such as:

- A simpler design could remove substantial complexity introduced or materially affected by the change.
- New branching makes an existing flow materially harder to reason about.
- A file crosses the 1,000-line threshold because of the change without compelling structural justification.
- Significant file growth, even below 1,000 lines, makes the module harder to understand or maintain.
- An abstraction, wrapper, or generic mechanism adds indirection without sufficient value.
- Logic is duplicated or placed outside its canonical ownership boundary.
- Types or API contracts hide important invariants through unnecessary optionality, casts, or loose shapes.
- Avoidable sequencing or non-atomic updates make orchestration more fragile.
- A refactor moves complexity without actually reducing it.

For each finding:

- State the problem and its practical consequence.
- Point to the specific file and location.
- Ground the finding in observable code, behavior, tests, or a concrete architectural consequence.
- Explain the causal relationship between the implementation and the reported problem.
- Explain why the current design is problematic, not merely that another style is preferred.
- Recommend a concrete remedy, ideally one that reduces concepts or control-flow paths while preserving behavior.
- Distinguish confirmed defects from design risks or non-blocking improvement opportunities.
- Avoid reporting the same underlying problem in multiple findings.

Do not report speculative concerns, hypothetical future needs, personal style preferences, or low-value nits. Do not present a possible alternative as a defect merely because it could be cleaner. Do not invent issues to make the review appear thorough.

## Priority Order

Rank findings in this order:

1. Correctness or reliability risks introduced by the change.
2. Major, demonstrable opportunities for structural simplification.
3. Increased branching, coupling, or state complexity.
4. Broken abstraction, ownership, or type boundaries.
5. File-size and decomposition problems.
6. Local readability and maintainability issues.

Keep severity proportional to impact. Distinguish blocking issues from non-blocking improvement opportunities. A maintainability preference alone is not sufficient to block approval. A structural simplification should be treated as blocking only when it addresses substantial, demonstrable complexity introduced or materially affected by the change.

## Output

Start with a brief overall assessment.

If issues exist, list findings from highest to lowest priority. Use this structure:

- **[Priority] Short title** — `path/to/file:line`
  - **Problem:** What is wrong, the evidence, and why it matters.
  - **Recommendation:** The concrete behavior-preserving improvement.

Use only priorities that communicate impact (for example, Critical, High, Medium, Low). Do not inflate severity. Use blocking severity only for issues that genuinely warrant blocking the change; label worthwhile but non-blocking improvements accordingly.

If no actionable issues are found, say so plainly and mention any important validation that was not performed. Do not manufacture findings.

## Approval Standard

Do not approve solely because the code works or tests pass. Approval requires:

- No material correctness or reliability issue introduced by the change.
- No obvious, high-value simplification left unaddressed when it removes substantial, demonstrable complexity.
- No unjustified growth in branching, coupling, state, or file size.
- No unnecessary abstraction, duplicated helper, or architecture-boundary leak that materially harms maintainability.
- No avoidable contract ambiguity that makes the implementation harder to understand or maintain.

A maintainability preference, stylistic disagreement, or hypothetical future concern is not by itself a reason to block approval. A proposed simplification is a blocker only when it addresses substantial, demonstrable complexity introduced or materially affected by the change and can preserve intended behavior without disproportionate cost.

When a material issue exists, explain the required change directly. Be rigorous without being arbitrary: every criticism must be grounded in the code, attributable to the change, and proportional to its impact.
