# Python Project Standards

These are structural standards for how Python projects must be organised and written.
They guide how code should be built — not runtime enforcement rules.

## PY-S-001: Required Project Files
- Standard: Every Python project must have these files at root level:
  - pyproject.toml OR setup.py (dependency declaration)
  - requirements.txt OR requirements-lock.txt (pinned dependencies)
  - .python-version OR Pipfile (Python version declaration)
  - README.md
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-002: Python Version Declaration
- Standard: Projects must declare the Python version used
  via .python-version file or python field in pyproject.toml
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-003: Test Directory Structure
- Standard: Tests must live in a /tests directory at project root
  Test files must follow naming convention: test_*.py or *_test.py
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-004: Package Directory Structure
- Standard: Every directory intended as a Python package must contain __init__.py
- Severity: LOW
- Enforcement: INFO

## PY-S-005: Use logging, Not print()
- Standard: Use the logging module instead of print() for any output in non-test code
- Severity: LOW
- Enforcement: INFO

## PY-S-006: Type Hints on Public Functions
- Standard: All public functions (no leading underscore) in src/ must have
  type hints on parameters and return type
- Severity: LOW
- Enforcement: INFO

## PY-S-007: Linting and Formatting Config
- Standard: Projects must include linting config (ruff, flake8, or pylint)
  and formatting config (black or ruff format) declared in pyproject.toml
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-008: Source Code in src/ Layout
- Standard: Application source code should live under src/{package_name}/
  not at the root level, to avoid import confusion during testing
- Severity: LOW
- Enforcement: INFO

---

## Lambda-Specific Standards

## PY-S-009: Separate Handler Entrypoint from Business Logic
- Standard: The Lambda entrypoint file (`handler.py`) must only contain the handler
  function and wiring — all business logic must live in `src/`
- Severity: MEDIUM
- Enforcement: WARN
- Reason: Separating entrypoint from logic makes unit testing possible without
  mocking the Lambda runtime

## PY-S-010: Environment Config via SSM Path Prefixes
- Standard: Environment-specific configuration must use separate SSM Parameter Store
  path prefixes (e.g. `/myapp/prod/`, `/myapp/dev/`) or `.env.{stage}` files —
  no hardcoded `if env == "prod"` stage checks in source code
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-011: IaC Co-located with Function Code
- Standard: Infrastructure-as-Code for the Lambda (SAM template, CDK stack, or
  Terraform module) must live in the same repository as the function source code
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-012: Unit Tests Must Mock AWS SDK Calls
- Standard: Unit tests must mock all AWS SDK interactions using `moto` or
  `pytest-mock` — no real AWS API calls are permitted in unit tests
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-013: Integration Tests Against LocalStack or Dev Account
- Standard: Integration tests must target LocalStack or a dedicated `dev` AWS
  account — never run integration tests against production
- Severity: HIGH
- Enforcement: BLOCK

## PY-S-014: Minimum 80% Test Coverage for src/
- Standard: Code coverage for the `src/` directory must be ≥80% — the handler
  entrypoint file is excluded from this threshold
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-015: Use AWS Lambda Powertools
- Standard: All Lambda functions must use `aws-lambda-powertools` for structured
  logging, metrics, and tracing rather than custom implementations
- Severity: MEDIUM
- Enforcement: WARN
- Reason: Powertools provides battle-tested, low-overhead observability primitives
  built specifically for the Lambda execution model

## PY-S-016: X-Ray Tracing and Instrumentation
- Standard: AWS X-Ray active tracing must be enabled on the Lambda function;
  all outbound HTTP calls and AWS SDK operations must be instrumented via the
  X-Ray SDK or Powertools Tracer
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-017: Lambda-Compatible Build Environment
- Standard: Deployment packages must be built inside a Lambda-compatible Docker
  container (e.g. `public.ecr.aws/lambda/python:3.12`) — not on a developer
  machine or a non-Amazon Linux CI runner
- Severity: HIGH
- Enforcement: WARN
- Reason: Native binary extensions compiled on macOS or Ubuntu are incompatible
  with the Lambda execution environment

## PY-S-018: Deployment Package Size Tracked in CI
- Standard: CI pipeline must measure and record the unzipped deployment package
  size; a warning must be raised if it exceeds 10MB and a block if it exceeds 50MB
- Severity: MEDIUM
- Enforcement: WARN

## PY-S-019: Versioned Deploys with Aliases
- Standard: Every production deployment must publish a new Lambda version;
  traffic must be routed through named aliases (`prod`, `staging`) that point to
  specific versions — aliases must never point to `$LATEST` in production
- Severity: HIGH
- Enforcement: BLOCK
