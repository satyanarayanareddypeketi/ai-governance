# Token Efficiency Policies

These policies govern how agents must behave to minimize LLM token consumption.
Every agent must follow these rules before making any AI call.

## TE-001: Check Cache Before Fetching
- Rule: Agents must always check .claude/governance-cache/last-updated.txt before
  fetching any governance policy. Never re-fetch if cache is from today.
- Severity: HIGH
- Enforcement: BLOCK (agent must not proceed to fetch if cache is current)

## TE-002: Read Only What You Need
- Rule: Agents must read only the policy files relevant to their task.
  Do not load all policy files unless you are CoreGuardian.
  - DeployAssure: security + deployment + compliance only
  - TestSentinel: compliance only
  - PipelineAssist: deployment + github-actions only
  - IncidentTracer: security + compliance only
  - CoreGuardian: all files (only exception)
- Severity: MEDIUM
- Enforcement: WARN

## TE-003: Summarize Before Reasoning
- Rule: When passing context to the LLM for reasoning, always summarize first.
  Never pass raw log files, full stack traces, or entire file contents.
  Extract only: error message, failed step, relevant lines (max 50 lines of logs).
- Severity: HIGH
- Enforcement: WARN

## TE-004: Deterministic First, AI Second
- Rule: Agents must attempt to resolve decisions using policy rules alone before
  invoking any AI reasoning. Only use AI reasoning if policy checks alone
  cannot produce a clear verdict.
- Severity: HIGH
- Enforcement: WARN

## TE-005: No Redundant Policy Re-reads
- Rule: Within a single conversation session, do not re-read a policy file
  you have already read. Reference previously read content instead.
- Severity: MEDIUM
- Enforcement: WARN

## TE-006: Concise Output Only
- Rule: Agent output must follow the standard GO/GO WITH WARNINGS/NO-GO format.
  No extra commentary, no restating the question, no padding.
  Output must be scannable in under 30 seconds.
- Severity: MEDIUM
- Enforcement: WARN

## TE-007: Maximum One Clarifying Question Per Invocation
- Rule: Agents must make a decision based on available context.
  Only ask the developer a question if a critical piece of information is
  completely missing and cannot be inferred (e.g. target environment unknown).
  Maximum one clarifying question per invocation.
- Severity: LOW
- Enforcement: INFO

## TE-008: Limit Context Passed to CoreGuardian
- Rule: When escalating to CoreGuardian, pass only:
  - Your verdict (GO / GO WITH WARNINGS / NO-GO)
  - The specific policy violation or concern
  - The relevant context snippet (not the full conversation)
  Do not re-send the entire thread.
- Severity: MEDIUM
- Enforcement: WARN
