# Deployment Policies

## DP-001: Deployment Windows
- Rule: Production deployments only Mon–Fri 09:00–17:00 UTC
- Severity: HIGH
- Enforcement: BLOCK

## DP-002: Canary Requirement
- Rule: Deployments affecting >10% traffic must use canary or blue-green strategy
- Severity: HIGH
- Enforcement: BLOCK

## DP-003: Rollback Plan
- Rule: Every deployment must declare a rollback procedure
- Severity: MEDIUM
- Enforcement: WARN

## DP-004: Approval Gates
- Rule: Production deployments require 2 human approvals + CoreGuardian GO
- Severity: CRITICAL
- Enforcement: BLOCK

## DP-005: Dependency Freeze
- Rule: No new third-party dependencies within 48h of a scheduled release
- Severity: MEDIUM
- Enforcement: WARN
