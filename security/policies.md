# Security Policies

## SP-001: No Secrets in Code
- Rule: No API keys, tokens, passwords, or credentials in source code
- Severity: CRITICAL
- Enforcement: BLOCK

## SP-002: Container Image Scanning
- Rule: All container images must pass vulnerability scan before deployment
- Severity: HIGH
- Enforcement: BLOCK on CRITICAL CVEs / WARN on HIGH CVEs

## SP-003: IAM Least Privilege
- Rule: No wildcard (*) IAM actions on production resources
- Severity: HIGH
- Enforcement: BLOCK

## SP-004: Branch Protection
- Rule: Direct pushes to main/master/release branches are prohibited
- Severity: HIGH
- Enforcement: BLOCK

## SP-005: SAST Gate
- Rule: Static analysis must pass before merge to protected branches
- Severity: HIGH
- Enforcement: BLOCK
