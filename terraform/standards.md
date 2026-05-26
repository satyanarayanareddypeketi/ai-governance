# Terraform Standards

These are structural standards for how Terraform projects must be organised and written.
They guide how infrastructure code should be built — not runtime enforcement rules.

---

## Module Design

## TF-S-001: Reusable Infrastructure Must Be a Module
- Standard: All reusable infrastructure patterns must be encapsulated as Terraform
  modules — copy-pasting resource blocks across environments is not permitted
- Severity: MEDIUM
- Enforcement: WARN

## TF-S-002: Required Provider and Version Constraints
- Standard: Every module must declare `required_providers` and `required_version`
  constraints in a `versions.tf` file — floating version constraints are not permitted
  in production modules
- Severity: HIGH
- Enforcement: WARN
- Example:
  ```hcl
  terraform {
    required_version = ">= 1.5.0, < 2.0.0"
    required_providers {
      aws = {
        source  = "hashicorp/aws"
        version = "~> 5.0"
      }
    }
  }
  ```

## TF-S-003: Typed and Described Module Inputs
- Standard: All module input variables must declare an explicit `type` constraint
  and a `description` — use of `type = any` is not permitted in shared/public modules
- Severity: MEDIUM
- Enforcement: WARN

## TF-S-004: Modules Must Expose Outputs
- Standard: Modules must declare outputs for every resource attribute that consumers
  may need (e.g. ARNs, IDs, endpoints) — consumers must not use
  `module.x.resource.y.attribute` to reach into module internals
- Severity: MEDIUM
- Enforcement: WARN

## TF-S-005: Root Modules Are Wiring Only
- Standard: Root modules (environment entrypoints) must only call child modules and
  set variable values — they must not define reusable resource logic directly
- Severity: MEDIUM
- Enforcement: WARN

---

## State Management

## TF-S-006: Remote State Required for Production
- Standard: All production and staging Terraform state must use a remote backend
  (S3 + DynamoDB locking, or Terraform Cloud) — local state is only permitted for
  ephemeral development environments
- Severity: HIGH
- Enforcement: BLOCK

## TF-S-007: State Isolation Per Environment and Workload
- Standard: State files must be isolated per environment (dev/staging/prod) and per
  workload — sharing state across environments or unrelated workloads is not permitted
- Severity: HIGH
- Enforcement: WARN
- Example S3 key pattern: `{project}/{environment}/{workload}/terraform.tfstate`

## TF-S-008: State Bucket Security Requirements
- Standard: The S3 bucket used for Terraform state must have all of the following:
  - Versioning enabled
  - Server-side encryption (SSE-KMS or SSE-S3)
  - MFA-delete enabled
  - Block all public access settings enabled
- Severity: HIGH
- Enforcement: BLOCK

---

## Directory and File Structure

## TF-S-009: Required Files in Every Module
- Standard: Every Terraform root module and reusable module must contain these files:
  - `main.tf` — primary resource definitions
  - `variables.tf` — all input variable declarations
  - `outputs.tf` — all output declarations
  - `versions.tf` — provider and Terraform version constraints
  - `README.md` — module purpose, inputs, outputs, and usage example
- Severity: MEDIUM
- Enforcement: WARN

## TF-S-010: Environment Separation via Directories
- Standard: Each environment must be a separate directory under `envs/`:
  ```
  envs/
    dev/
    staging/
    prod/
  ```
  No single root module may manage multiple environments via workspace switching
- Severity: HIGH
- Enforcement: WARN
- Reason: Workspace-based environment separation shares state backend config and
  increases the risk of accidental cross-environment applies

## TF-S-011: Variable Values in .tfvars Files
- Standard: All environment-specific variable values must be declared in
  `{environment}.tfvars` files — no hardcoded literal values inside `.tf` resource
  blocks or module calls
- Severity: MEDIUM
- Enforcement: WARN

---

## Naming and Tagging

## TF-S-012: Mandatory Resource Tags
- Standard: All AWS resources that support tagging must include these tags at minimum:
  | Tag | Description |
  |---|---|
  | `Environment` | dev / staging / prod |
  | `Owner` | team or individual responsible |
  | `CostCenter` | billing allocation code |
  | `ManagedBy` | must be set to `terraform` |
  | `Project` | project or product name |
- Severity: HIGH
- Enforcement: WARN
- Recommendation: Declare a `local.common_tags` map in each root module and merge
  it into every resource's `tags` argument

## TF-S-013: Consistent Resource Naming Convention
- Standard: Resource names must follow the pattern:
  `{project}-{environment}-{resource-type}-{descriptor}`
  Example: `payments-prod-rds-primary`, `auth-dev-lambda-authorizer`
- Severity: MEDIUM
- Enforcement: WARN

---

## Testing

## TF-S-014: Modules Must Include Examples
- Standard: Every reusable module must have an `examples/` directory containing at
  least one runnable configuration that demonstrates the module's primary use case
- Severity: LOW
- Enforcement: INFO

## TF-S-015: Automated Module Tests Required
- Standard: Reusable modules must have automated tests using Terratest or native
  `terraform test` that cover at minimum: plan, apply, assertion of outputs, and destroy
- Severity: MEDIUM
- Enforcement: WARN

---

## CI/CD Integration

## TF-S-016: Formatting and Validation in CI
- Standard: CI pipeline must run `terraform fmt -check` and `terraform validate`
  on every pull request — formatting or validation failures must block merge
- Severity: HIGH
- Enforcement: BLOCK

## TF-S-017: Plan Artifact Saved and Reused for Apply
- Standard: CI pipeline must save the `terraform plan` output as a binary plan file
  (`.tfplan`) and the apply step must use that saved plan — apply must never re-run
  plan independently
- Severity: HIGH
- Enforcement: WARN
- Reason: Re-running plan at apply time can produce a different plan if upstream
  state changed between CI steps

## TF-S-018: Security Scan Before Apply
- Standard: `tfsec` or `checkov` must run against all Terraform code in CI and
  must produce no HIGH or CRITICAL findings before an apply is permitted
- Severity: HIGH
- Enforcement: BLOCK
