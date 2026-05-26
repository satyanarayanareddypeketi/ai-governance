# GitHub Actions Pipeline Structure Policies

## GA-001: Workflow File Size Limit
- Rule: No workflow file (.github/workflows/*.yml) should exceed 300 lines
- Severity: MEDIUM
- Enforcement: WARN
- Reason: Large workflows become unmaintainable and hard to debug

## GA-002: Required Workflow Sections
- Rule: Every workflow must have: name, on (trigger), jobs, and at least one step with uses or run
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Incomplete workflows fail silently or behave unexpectedly

## GA-003: Pinned Action Versions
- Rule: All third-party GitHub Actions must be pinned to an exact version (e.g. actions/checkout@v4)
  Floating tags like @main or @master are not allowed in production workflows
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Unpinned actions can introduce breaking changes or supply chain attacks

## GA-004: Secrets Must Use GitHub Secrets
- Rule: No hardcoded credentials, tokens, or API keys in workflow files
  All sensitive values must reference ${{ secrets.SECRET_NAME }}
- Severity: CRITICAL
- Enforcement: BLOCK

## GA-005: Timeout on Every Job
- Rule: Every job must declare a timeout-minutes value (max allowed: 60)
- Severity: MEDIUM
- Enforcement: WARN
- Reason: Jobs without timeouts can run indefinitely and consume billable minutes

## GA-006: Concurrency Control on Production Workflows
- Rule: Workflows that deploy to production must define a concurrency group
  to prevent simultaneous deployments
- Severity: HIGH
- Enforcement: WARN
- Example:
  concurrency:
    group: production-deploy
    cancel-in-progress: false

## GA-007: No Self-Hosted Runners for Public Repos
- Rule: Public repositories must not use self-hosted runners
- Severity: HIGH
- Enforcement: BLOCK
- Reason: Self-hosted runners on public repos are a security risk

## GA-008: Required Permissions Block
- Rule: Every workflow must explicitly declare permissions (do not rely on defaults)
- Severity: MEDIUM
- Enforcement: WARN
- Example:
  permissions:
    contents: read
    pull-requests: write
