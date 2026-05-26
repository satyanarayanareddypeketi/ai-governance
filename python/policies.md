# Python Policies

These are enforceable rules for Python projects — violations result in WARN or BLOCK verdicts.

## PY-P-001: Pinned Dependencies in Production
- Rule: All dependencies in requirements.txt must be pinned to exact versions
  (e.g. requests==2.31.0 not requests>=2.0)
- Severity: HIGH
- Enforcement: BLOCK for production branches / WARN for dev branches
- Reason: Unpinned dependencies cause non-reproducible builds and unexpected breakage

## PY-P-002: Full Dependency Tree Must Be Locked
- Rule: Use pip-compile or poetry lock to ensure the full transitive dependency
  tree is pinned, not just top-level packages
- Severity: MEDIUM
- Enforcement: WARN

## PY-P-003: No Hardcoded Config Values
- Rule: No hardcoded URLs, credentials, environment names, or feature flags in source code
  Use environment variables or a dedicated config module
- Severity: HIGH
- Enforcement: BLOCK

## PY-P-004: No Secrets in Source Code
- Rule: No API keys, passwords, tokens, or connection strings in any .py file
- Severity: CRITICAL
- Enforcement: BLOCK

---

## Lambda-Specific Policies

## PY-P-005: No Direct Event Logging
- Rule: Lambda handler must not log the raw `event` object — PII fields must be
  masked or redacted before any logging
- Severity: CRITICAL
- Enforcement: BLOCK
- Reason: Lambda events often contain user data, tokens, or PII that must not appear in CloudWatch logs

## PY-P-006: No Unsafe Code Execution
- Rule: Use of `eval()`, `exec()`, or `pickle.loads()` on any untrusted or
  externally sourced input is prohibited
- Severity: CRITICAL
- Enforcement: BLOCK

## PY-P-007: IAM Least-Privilege — No Wildcard Actions
- Rule: IAM execution role attached to Lambda must not use `*` for actions or
  resources; every permission must be scoped to specific ARNs and actions
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Overly permissive Lambda roles are the #1 Lambda security finding

## PY-P-008: Secrets via Secrets Manager or SSM Only
- Rule: Sensitive values (DB passwords, API keys, tokens) must be retrieved from
  AWS Secrets Manager or SSM Parameter Store at runtime — not stored as Lambda
  environment variables
- Severity: HIGH
- Enforcement: BLOCK

## PY-P-009: No Direct Public Internet Access Without NAT
- Rule: Lambda functions placed in a VPC must not have a public IP or direct
  internet gateway route — all outbound internet traffic must go through a NAT gateway
- Severity: HIGH
- Enforcement: WARN

## PY-P-010: No Dev Dependencies in Deployment Package
- Rule: Lambda deployment packages must not include development or test dependencies
  (e.g. pytest, mypy, black, ruff) — only runtime dependencies are permitted
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Dev deps bloat cold start package size and expand the attack surface

## PY-P-011: Lambda Layers Must Be Versioned
- Rule: Lambda layer references in production must pin to a specific version ARN —
  mutable `latest` references are not allowed
- Severity: MEDIUM
- Enforcement: WARN

## PY-P-012: No Deprecated Lambda Runtime
- Rule: Lambda functions must use a Python runtime version that is not deprecated
  or end-of-life in AWS (currently python3.8 and below are deprecated)
- Severity: HIGH
- Enforcement: BLOCK

## PY-P-013: Exceptions Must Not Be Swallowed
- Rule: Lambda handler must not catch exceptions silently — all caught exceptions
  must be logged and either re-raised or returned as a structured error response
- Severity: HIGH
- Enforcement: BLOCK

## PY-P-014: DLQ Required for Async Lambdas
- Rule: Any Lambda invoked asynchronously (SNS, S3, EventBridge, async invoke) must
  have a Dead Letter Queue (SQS or SNS) configured
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Without a DLQ, failed async invocations are silently dropped after retries

## PY-P-015: Idempotency Required for Event-Driven Lambdas
- Rule: Lambdas triggered by SQS, SNS, or EventBridge must implement idempotency
  (e.g. via AWS Lambda Powertools Idempotency or a deduplication key check) to
  handle at-least-once delivery
- Severity: HIGH
- Enforcement: WARN

## PY-P-016: Structured JSON Logging Required
- Rule: All log output from Lambda functions must be structured JSON — no
  unstructured string concatenation or bare print() statements
- Severity: HIGH
- Enforcement: WARN

## PY-P-017: Correlation/Trace ID Propagation
- Rule: Every Lambda invocation must attach a correlation ID or X-Ray trace ID to
  all log entries and downstream calls (HTTP, SQS, SNS, SDK)
- Severity: MEDIUM
- Enforcement: WARN

## PY-P-018: Custom CloudWatch Metrics for Business Operations
- Rule: Business-critical Lambda operations (payments, auth, data writes) must emit
  at least one custom CloudWatch metric per operation outcome (success/failure)
- Severity: MEDIUM
- Enforcement: WARN

## PY-P-019: Clients Initialized Outside Handler
- Rule: AWS SDK clients, database connections, and HTTP session objects must be
  initialized at module level (outside the handler function) to enable connection
  reuse across warm Lambda invocations
- Severity: HIGH
- Enforcement: WARN
- Reason: Re-initializing clients inside the handler on every invocation causes
  unnecessary cold-start latency and exhausts connection pools

## PY-P-020: Memory Setting Must Be Benchmarked
- Rule: Production Lambda functions must not use the default 128MB memory setting
  without a documented benchmark justification — memory must be right-sized using
  AWS Lambda Power Tuning or equivalent
- Severity: MEDIUM
- Enforcement: WARN

## PY-P-021: Timeout Must Be Explicitly Set
- Rule: All Lambda functions must declare an explicit timeout — relying on the
  default 3-second timeout in production is not permitted
- Severity: HIGH
- Enforcement: BLOCK
