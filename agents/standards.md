# Agent Standards

Every agent in .claude/agents/ must have:
- A clear Purpose
- Defined Triggers (what invokes it)
- Decision Authority (GO / GO WITH WARNINGS / NO-GO)
- Standard output format (see below)
- Reference to which governance policies it enforces

Agents must NOT:
- Make production changes without CoreGuardian GO
- Skip governance policy checks
- Call external APIs without declaring it in their definition
- Produce output that bypasses the standard GO/NO-GO format

## Token Efficiency Rules (mandatory in every agent)

Every agent must include these 3 rules at the very top of its instructions,
before any other steps:

1. Read ONLY the policy files listed as required for your role — nothing else
2. If the developer provides logs, files, or code — read max 50 lines only
3. Respond in one turn using the standard output format
   Ask at most one clarifying question, only if a critical piece of info is completely missing

---

## Standard Output Format

Every agent must produce output in this exact structure:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
AGENT: {Agent Name}
DECISION: GO | GO WITH WARNINGS | NO-GO
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SUMMARY:
{2-3 sentences. Plain English. A non-technical person must understand this.}

WARNINGS: (only shown for GO WITH WARNINGS)
- {warning}

ISSUES: (only shown for NO-GO)
- [{policy_id}] {violation} ({severity})

RECOMMENDED ACTION:
{One clear sentence telling the developer exactly what to do next.}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## Decision Criteria

| Decision | When |
|---|---|
| GO | All policies pass, no concerns |
| GO WITH WARNINGS | No blocking violations, but risks worth noting |
| NO-GO | One or more BLOCK-level policy violations found |
