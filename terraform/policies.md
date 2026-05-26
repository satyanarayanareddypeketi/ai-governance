# Terraform Policies

These are enforceable rules for Terraform projects — violations result in WARN or BLOCK verdicts.

---

## Security

## TF-P-001: No Sensitive Values in Version Control
- Rule: `.tfvars` files containing sensitive values (passwords, tokens, keys) must
  not be committed to version control — use `.gitignore` to exclude them and
  supply values via CI secrets or a secrets manager
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-002: Sensitive Outputs Must Be Marked
- Rule: Any Terraform output that exposes a secret, password, token, or private key
  must declare `sensitive = true`
- Severity: HIGH
- Enforcement: BLOCK

## TF-P-003: S3 Buckets Must Block All Public Access
- Rule: Every `aws_s3_bucket` resource must have an associated
  `aws_s3_bucket_public_access_block` resource with all four settings set to `true`:
  - `block_public_acls = true`
  - `block_public_policy = true`
  - `ignore_public_acls = true`
  - `restrict_public_buckets = true`
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-004: No Unrestricted Ingress on Non-Web Ports
- Rule: Security group rules must not allow `0.0.0.0/0` or `::/0` ingress on any
  port other than 80 or 443 — all other ingress must be scoped to specific CIDRs
  or security group IDs
- Severity: HIGH
- Enforcement: BLOCK

## TF-P-005: Encryption at Rest Required
- Rule: All stateful AWS resources must have encryption enabled:
  | Resource | Required Setting |
  |---|---|
  | `aws_s3_bucket` | `server_side_encryption_configuration` |
  | `aws_db_instance` | `storage_encrypted = true` |
  | `aws_dynamodb_table` | `server_side_encryption` enabled |
  | `aws_ebs_volume` | `encrypted = true` |
  | `aws_sqs_queue` | `kms_master_key_id` set |
- Severity: HIGH
- Enforcement: BLOCK

## TF-P-006: KMS Key Rotation Must Be Enabled
- Rule: All `aws_kms_key` resources must set `enable_key_rotation = true`
- Severity: MEDIUM
- Enforcement: WARN

## TF-P-007: No Wildcard IAM Policies via Terraform
- Rule: `aws_iam_policy` and `aws_iam_role_policy` resources must not use `*` for
  both `Action` and `Resource` in the same statement
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-008: RDS Must Not Be Publicly Accessible
- Rule: `aws_db_instance` and `aws_rds_cluster` resources must set
  `publicly_accessible = false`
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-009: CloudTrail Must Be Enabled in Root Account
- Rule: Every AWS account managed by Terraform must have an `aws_cloudtrail`
  resource with multi-region enabled and log file validation enabled
- Severity: HIGH
- Enforcement: WARN

---

## State and Plan Safety

## TF-P-010: Plan Required Before Production Apply
- Rule: A `terraform plan` must be reviewed and approved before any `terraform apply`
  targeting a production environment — applying without a reviewed plan is not permitted
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-011: No Destroy on Production Root Module
- Rule: `terraform destroy` must not be run against a production root module without
  explicit `--target` scoping and documented approval — whole-environment destroy
  is always blocked in production
- Severity: CRITICAL
- Enforcement: BLOCK

## TF-P-012: prevent_destroy on Stateful Production Resources
- Rule: The following resource types in production environments must declare
  `lifecycle { prevent_destroy = true }`:
  - `aws_db_instance`
  - `aws_rds_cluster`
  - `aws_dynamodb_table`
  - `aws_s3_bucket`
  - `aws_elasticache_replication_group`
- Severity: HIGH
- Enforcement: BLOCK

---

## CI/CD

## TF-P-013: fmt and validate Must Pass in CI
- Rule: `terraform fmt -check` and `terraform validate` must pass on every pull
  request — failures block merge
- Severity: HIGH
- Enforcement: BLOCK

## TF-P-014: Security Scan Must Pass Before Apply
- Rule: `tfsec` or `checkov` must produce zero HIGH or CRITICAL findings before
  a Terraform apply is permitted in CI
- Severity: HIGH
- Enforcement: BLOCK

## TF-P-015: Saved Plan Must Be Used for Apply
- Rule: The CI apply step must consume the `.tfplan` binary artifact produced by
  the plan step — re-running plan at apply time is not permitted
- Severity: HIGH
- Enforcement: WARN

---

## Drift and Compliance

## TF-P-016: No Manual Changes to Terraform-Managed Resources
- Rule: AWS resources declared in Terraform must not be modified directly via the
  AWS Console, CLI, or SDK outside of a Terraform apply — all changes must go
  through the IaC pipeline
- Severity: HIGH
- Enforcement: WARN
- Reason: Manual changes create state drift that causes unpredictable behaviour on
  the next plan/apply cycle

## TF-P-017: Drift Detection Must Run on a Schedule
- Rule: Automated `terraform plan` drift detection must run at least weekly against
  all production environments — any detected drift must create an alert or ticket
- Severity: MEDIUM
- Enforcement: WARN

---

## Module Governance

## TF-P-018: Only Approved Module Sources Permitted
- Rule: Root modules must only reference modules from approved sources:
  - Internal module registry
  - `git::https://github.com/{org}/` (organisation-owned repos)
  - Official HashiCorp Registry modules (`registry.terraform.io/hashicorp/`)
  Third-party public registry modules require security review before use
- Severity: HIGH
- Enforcement: WARN

## TF-P-019: Module Versions Must Be Pinned
- Rule: All module source references must pin to a specific version tag or commit
  SHA — floating references (`?ref=main` or no version) are not permitted in
  production
- Severity: HIGH
- Enforcement: BLOCK
