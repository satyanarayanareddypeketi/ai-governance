# Compliance Policies

## CMP-001: Audit Log Retention
- Rule: All deployment events must log actor, timestamp, environment, outcome
- Severity: CRITICAL
- Enforcement: BLOCK

## CMP-002: PII in Logs
- Rule: Log outputs must not contain PII (email, SSN, credit card patterns)
- Severity: HIGH
- Enforcement: BLOCK

## CMP-003: Encryption at Rest
- Rule: All S3 buckets and RDS instances must have encryption enabled
- Severity: HIGH
- Enforcement: BLOCK

## CMP-004: Test Coverage Threshold
- Rule: Code coverage must not drop below 70% on protected branches
- Severity: MEDIUM
- Enforcement: WARN (below 70%) / BLOCK (below 50%)
