---
name: requirement-analysis
description: Analyze a business/software requirement (BRD, ticket, feature request, or change request) into Functional Requirements, Non-Functional Requirements, Scope, Business Logic (old vs new flow traced to actual code/functions, old vs new data model, glossary of abbreviations), and Business Rules/Edge Cases with concrete examples. Use before implementation or design. Also matches Vietnamese requests like "phân tích yêu cầu", "phân tích BRD", "so sánh flow cũ và mới".
license: MIT
compatibility: opencode
metadata:
  category: business-analysis
---

## What I do

Turn a raw requirement into a structured analysis a developer can implement from and a reviewer can verify against — grounded in the actual codebase, not just the requirement text. Output always has 6 sections: **Scope**, **Functional Requirements**, **Non-Functional Requirements**, **Business Logic (old vs new)**, **Data Model (old vs new)**, **Business Rules & Edge Cases**. See [references/template.md](references/template.md) for the exact output format with a worked example.

## When to use me

- User shares a requirement/BRD/ticket and asks for analysis or breakdown
- User asks to compare old vs new flow before making a change
- Before writing a technical design or implementation plan for a non-trivial feature/change

Not for: implementing the feature itself (this produces the analysis, not the code), and not for trivial bug fixes with no ambiguity in scope or behavior.

## Step 1 — Gather input, classify NEW vs CHANGE

Collect the requirement text and any linked docs/tickets. Determine:
- **NEW feature** — no prior behavior exists.
- **CHANGE** — modifies existing behavior. You MUST locate the current implementation in the codebase (search by domain keyword, API route, or table name) before writing anything. Do not analyze a "change" from the requirement text alone.

If the requirement is missing, contradictory, or too vague to scope, ask the user — do not invent scope to fill the gap.

## Step 2 — Scope

- **In-scope**: what this requirement covers, stated as concrete capabilities
- **Out-of-scope**: things that sound related but are explicitly excluded — call these out even if the requirement doc doesn't mention them, whenever you can reasonably foresee the reader asking "does this also cover X?"
- **Assumptions**: anything you filled in due to ambiguity. Flag every one for the user to confirm — never resolve ambiguity silently in a way the user didn't ask for.

## Step 3 — Functional Requirements (FR)

List discrete, testable requirements with stable IDs (`FR-01`, `FR-02`, ...) so later docs/PRs can reference them:
- Actor (who/what triggers it)
- Precondition → action → expected outcome
- Acceptance criteria, Given/When/Then style

## Step 4 — Non-Functional Requirements (NFR)

Only cover categories relevant to this requirement — for each irrelevant one, write "Not applicable" rather than omitting it silently (this documents that it was considered):
- Performance (latency/throughput, hot-path impact)
- Security & authorization (who can/can't trigger this, data sensitivity)
- Reliability (retry, idempotency, failure/rollback behavior)
- Scalability / data volume
- Compliance & audit (logging, regulatory retention — relevant for retail/finance domains)
- Backward compatibility (existing API/data consumers)
- Observability (what must be logged/monitored to detect issues in production)

## Step 5 — Business Logic: old flow vs new flow

The most code-grounded section — every step must cite a real `file:line` and function/method name, found by actually reading the code, never fabricated.

1. Trace the **current** flow step by step, each step tagged with its `file:line — Function.method()`.
2. Trace the **proposed/new** flow the same way. For code that doesn't exist yet, mark the step `(new)` and propose where it should live based on the existing module structure (for consistency with neighboring code).
3. Present as a table: `Step | Old (file:function) | New (file:function) | What changes`.
4. For a pure NEW feature (no old flow), document the proposed flow to the same level of code-grounded detail, modeled on how a similar existing flow is implemented in this codebase.

## Step 6 — Data Model: old vs new

For each entity/table/schema touched:
- Old: fields, types, constraints, relationships — cite the schema/model file
- New: added/removed/changed fields and migration implications
- Backward-compatibility concerns: existing rows, nullable vs required, default values

## Step 7 — Glossary

Collect every abbreviation and domain term used in the requirement or the touched code area, each with a one-line definition (e.g. `SKU`, `GMV`, `COD`). Domain acronyms mean specific things per company/context — never assume the reader already knows them.

## Step 8 — Business Rules & Edge Cases

For every rule implied or stated by the requirement — state it explicitly even if only implied — give a concrete example, then enumerate edge cases derived from it:

| Rule | Example input → outcome | Edge case | Expected outcome |
|---|---|---|---|

Cover at minimum: boundary values (zero, negative, max/overflow), invalid/missing input, duplicate/concurrent actions, and permission edge cases. A rule listed without a concrete example is incomplete — go back and add one.

## Step 9 — Output

One structured markdown document, sections in the order above. Anything you couldn't resolve: mark `TBD — needs clarification: <specific question>` instead of guessing.

## Guardrails

- Every code reference must come from actually reading the codebase (Grep/Read/Explore) — never fabricate a file path, line number, or function name.
- Every business rule or edge case must ship with a concrete example.
- Surface assumptions and ambiguities explicitly; don't silently pick one interpretation of an ambiguous requirement.
- Keep FR/NFR IDs stable across revisions of the same analysis so they stay referenceable from later design docs and PRs.
