# .github Repository

This repository brings together Copilot skills to improve the implementation process with a focus on software quality.

## Objective

Standardize how QA-related activities are requested, including:
- Generation of manual test cases from backlog items.
- Generation of flow-based unit tests (controller, service, and helpers), with execution and coverage.
- Generation of higher-quality code.


## Structure

- copilot-instructions.md
- skills/test-cases/SKILL.md
- skills/unit-tests/SKILL.md

## Available Skills (Current Scope)

### 1. Test Cases

File: skills/test-cases/SKILL.md

Purpose:
- Generate prescriptive test cases from two texts of the same backlog item.
- Treat backlog_original as the source of truth.
- Surface discrepancies and gaps for the PO.
- Produce output in CT format and in Portuguese.

When to use:
- When you have a backlog item and want manual test cases that are executable and convertible to E2E.

### 2. Unit Tests

File: skills/unit-tests/SKILL.md

Purpose:
- Generate full-flow unit tests from an endpoint or controller.
- Mock external dependencies (database, HTTP, queues, cache, and third-party services).
- Cover happy path, edge cases, and errors.
- Run tests and return a coverage report for the tested classes.
- If there is a coverage tooling blocker, request installation of the required tool.

When to use:
- When you need complete flow-based unit tests with infrastructure isolation and coverage visibility.

## Notes

- The more complete the input, the better the quality of generated tests.
- Missing or conflicting requirements must be made explicit as gaps/discrepancies.
- The unit test strategy prioritizes isolation of external dependencies through mocks.
- This repository is not limited to tests; new QA capabilities can be progressively incorporated.
