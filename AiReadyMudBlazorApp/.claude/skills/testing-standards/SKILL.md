---
name: testing-standards
description: 'Use when writing or reviewing unit tests, integration tests, or bUnit component tests. Covers test project selection, naming, setup patterns, behavior-first assertions, UI verification, and required build/test checks.'
argument-hint: 'Testing task, affected project, or changed behavior'
---

# Testing Standards

## When to Use

- Add or update automated tests for .NET code changes.
- Decide whether a change needs unit, integration, or component coverage.
- Review a test for brittleness, missing coverage, or implementation-detail coupling.
- Verify completion for spec-driven work or UI changes.

## Procedure

1. Identify the behavior that changed and choose the smallest test level that can verify it reliably.
2. Select the matching test project and mirror the source structure when practical.
3. Write behavior-focused tests using the repo's libraries and naming conventions.
4. Run the relevant verification commands before considering the work complete.
5. If the change affects anything in the UI, run the app and navigate the relevant user flows with the `#browser` tool, or with Playwright if `#browser` is not sufficient, to confirm the UI both works and looks as expected.
6. If the work came from a spec in `.specs/` (repo root), update the acceptance checkboxes and status as the testing work is completed.

## Choosing the Test Type

- Use unit tests for one behavior in isolation, especially branching, transformations, and explicit error handling.
- Use integration tests for multiple units working together, end-to-end flows, or file input/output behavior.
- Use bUnit component tests in the repository's web test project for component rendering, interaction, and UI states.
- Do not add tests for pure delegation, configuration wiring, or default framework behavior unless the code adds its own logic.

## bUnit Component Test Philosophy

bUnit renders the full component tree by default, making parent-component tests inherently integration-style. There is no need for a separate unit + integration split.

**Test observable outcomes, not invocations.**
Prefer asserting that a message appeared, a component rendered or disappeared, or UI state changed over asserting that a method was called. Use `Received()` only when a service call with specific arguments is genuinely the behavior under test—e.g., when the service owns the mutable state and the parent has no other way to observe the result.

**Avoid `DidNotReceive()` assertions.**
If a visible outcome proves the operation didn't happen (a no-op message is shown, the modal is closed, etc.), assert that outcome instead. A `DidNotReceive()` check is redundant alongside an assertion that already proves the same thing.

**Don't test child component internals from the parent.**
Each child component has its own tests. The parent's tests should only verify what the *parent* is responsible for: wiring services, showing or hiding sections, passing correct data. Do not assert on CSS state, internal properties, or render logic that belongs entirely to a child component.

**Avoid tests for static, always-rendered informational copy.**
If text is unconditional and carries no state or behavior, a test usually only proves that a literal string exists in markup. That adds brittleness when wording changes without protecting meaningful behavior. Prefer browser verification for this kind of content, and reserve automated tests for state changes, conditional rendering, user interactions, and data-driven output.

**Keep shared setup minimal.**
The constructor or class-level setup should only contain what's needed for the component to render without crashing—service registrations, JSInterop stubs, and fixed return values for state the component reads on init. Tests that need specific mock behavior should set it up in the test body. Default NSubstitute return values (null / zero / empty collections) are already implicit; don't repeat them in setup.

## Choosing the Test Project

- The repository's test project is `AiReadyMudBlazorApp.UnitTests\`.
- Keep test files aligned with the source area they cover. For example, tests for `AiReadyMudBlazorApp\Components\` live under `AiReadyMudBlazorApp.UnitTests\Components\`.

## Test Conventions

- Use xUnit with AwesomeAssertions, NSubstitute, AutoFixture, and bUnit where appropriate.
- Do not create custom fake classes in tests when NSubstitute can express the behavior. Prefer `Substitute.For(...)` test doubles over handwritten fakes.
- Name tests `When_<action>_Then_<expected_result>`, with every word separated by an underscore. The only exception is a codebase identifier (method name, class name, property name) used verbatim — treat that as a single unit (e.g., `When_ComputeHash_Called_Then_Returns_Sha256_Hex_String` where `ComputeHash` is a method being tested). Never merge plain English words without a separating underscore.
- Use Arrange, Act, Assert structure without section comments.
- Prefer collection expressions in test setup and assertions when the target type is clear instead of `new List<T>()` or `new List<T> { ... }`.
- If the repository later adds database-involved tests, use the real provider behavior over `Microsoft.EntityFrameworkCore.InMemory`. NEVER use SQLite or the in-memory provider.
- When using NSubstitute `Returns(first, second, ...)` for sequential values, do not pass a bare empty collection expression like `[]` as a later argument. Assign it to a typed local first so NSubstitute does not interpret it as an empty set of additional return values.
- Prefer class-level test doubles and initialize the SUT in the constructor when setup is shared. xUnit creates a new test class instance for every test, so constructor initialization is safe and exactly equivalent to constructing the SUT at the top of each test body.
- Assert behavior, returned values, state changes, or rendered output instead of implementation details.
- When asserting multiple properties on the same returned object or DTO, prefer a single `BeEquivalentTo(...)` assertion over separate per-property assertions when it keeps the expectation clear.
- Avoid `Received()` and `DidNotReceive()` unless the interaction itself is the behavior under test.
- Do not write tests that only prove an uncaught exception bubbles through unchanged.
- Do not make production members more visible solely to unit test them; extract a service or helper when needed.
- Never call external services in tests.
- For simple entity persistence tests, one test that inserts a record and asserts all fields are correct is sufficient. Do not multiply tests across individual fields, optional fields, or enum values.

## Verification

- Run `dotnet build AiReadyMudblazorApp.slnx` for code changes.
- Whenever you add or change automated tests, run `dotnet test AiReadyMudblazorApp.slnx` to verify the changes worked.
- If `dotnet test` fails because of your change, keep making focused fixes and re-running it until it passes.
- For UI changes, use the `#browser` tool or Playwright MCP (whichever you have) to navigate the affected user flows. Use Playwright MCP if `#browser` is not available (or sufficient). Keep iterating until the UI works and looks right.
- Do not finish while change-related build or test failures remain.

## Completion Checklist

- The changed behavior is covered by the right automated tests.
- The selected test project matches the source area.
- Assertions verify behavior rather than collaborator calls.
- Build and relevant tests pass.
- UI changes are verified with the `#browser` tool, or with Playwright when `#browser` is not available (or sufficient), and the affected flows both work and look right.
- Spec files are updated when the work implements acceptance criteria.
