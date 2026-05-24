# Cost Policies

## CP-001: Budget Threshold
- Rule: AWS spend >80% of monthly budget triggers warning; >95% blocks non-critical deploys
- Severity: MEDIUM (80%) / HIGH (95%)
- Enforcement: WARN / BLOCK

## CP-002: Large Resource Provisioning
- Rule: Instances larger than xlarge require cost justification in IaC metadata
- Severity: MEDIUM
- Enforcement: WARN

## CP-003: Idle Resources
- Rule: Resources unused >7 days flagged for review
- Severity: LOW
- Enforcement: INFO
