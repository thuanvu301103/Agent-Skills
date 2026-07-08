# PR Review Checklist

Use these categories to drive Step 3 of the review. Not every category applies to every diff — skip what's irrelevant rather than forcing a finding.

## 1. Correctness

- Off-by-one errors, wrong comparison operators, inverted booleans
- Null/undefined/None not handled where the type allows it
- Incorrect handling of empty collections, zero, negative numbers
- Async/await misuse: missing `await`, unhandled promise rejection, race conditions between concurrent calls
- State mutated when the caller expects immutability (or vice versa)
- Error paths that swallow exceptions silently or return a misleading success value

## 2. Security

- Unsanitized input reaching a shell command, SQL query, HTML render, or file path (injection, XSS, path traversal)
- Secrets, tokens, or credentials committed in code, logs, or config
- Missing authorization check on a new endpoint/action (authentication alone is not authorization)
- Deserializing untrusted input without validation
- New dependency added without checking it's from a trustworthy source

## 3. Performance

- N+1 queries or repeated network/DB calls inside a loop that could be batched
- Unbounded loops or recursion over user-controlled input
- Loading a full dataset into memory when pagination/streaming is available
- Redundant re-computation of a value already available in scope

## 4. Concurrency & shared state

- Shared mutable state accessed without synchronization
- Resources (files, connections, locks) not released on early return or exception
- Idempotency: does retrying this operation cause duplicate side effects?

## 5. Error handling

- Errors caught and logged but the failure is not actually surfaced to the caller/user
- Overly broad catch blocks that hide unrelated bugs
- Missing handling for a failure mode the new code explicitly introduces (e.g. a new external call with no timeout/retry story)

## 6. Tests

- New behavior with no corresponding test
- Tests that only cover the happy path, missing the edge case the PR was actually meant to fix
- Assertions that would pass even if the implementation were wrong (weak/tautological assertions)
- Mocked-out dependency that hides a real integration risk (e.g. mocking a DB call that could fail in ways the mock can't represent)

## 7. API & backward compatibility

- Public function/endpoint signature changed in a way that breaks existing callers
- Removed or renamed field in a serialized format (API response, DB schema, config file) without a migration path
- Default behavior changed for existing callers who didn't opt in

## 8. Readability & simplification (secondary priority)

- Duplicated logic that could reuse an existing helper
- Abstraction introduced for a single use case (premature generalization)
- Naming that misrepresents what the code does (not just "could be nicer")

Only raise items in this category as **Nits** — they should never block a merge on their own.
