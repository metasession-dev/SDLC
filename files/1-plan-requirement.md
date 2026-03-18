---
description: Define a new requirement in the RTM, classify risk, generate test scope, and prepare for implementation
---

# Plan Requirement

**Pipeline Stage:** 1 of 5
**Next:** `2-implement-and-test.md`
**References:** Test Policy (risk classification, AI governance), Test Strategy (risk matrix, testing depth, AI documentation)

---

## When to Use

- Starting a new feature, enhancement, or significant change
- Work that needs formal traceability (security, payments, RBAC, data handling)
- Any change a stakeholder or auditor might ask "was this tested?"

**Skip this workflow** for trivial changes (typo fixes, formatting, dependency bumps) — go straight to `2-implement-and-test.md`.

## Steps

### Step 1: Determine the Next Requirement ID

```bash
grep -oP 'REQ-\d+' compliance/RTM.md | sort -t- -k2 -n | tail -1
```

The next ID is one higher (e.g., if the last is REQ-007, use REQ-008).

### Step 2: Classify Risk Level

| Risk Level | Criteria |
|---|---|
| **Low** | Internal tools, no regulated data, no auth changes |
| **Medium** | Touches PII, user-facing features, API changes, new dependencies |
| **High** | Security, payments, RBAC, data handling, authentication |

AI involvement raises risk by one level when touching Medium or High categories. See Test Policy for the full risk matrix.

### Step 3: Add Entry to RTM

Open `compliance/RTM.md`, Part B:

```markdown
| REQ-XXX | [Brief description] | [LOW/MEDIUM/HIGH] | TBD | TBD | DRAFT | -- | -- |
```

### Step 4: Create Evidence Directory

```bash
mkdir -p compliance/evidence/REQ-XXX
```

### Step 5: Generate Test Scope

Create a test scope document proportional to the assessed risk level. This must exist **before implementation begins** — it is the evidence that testing was planned, not retroactively documented.

The AI generates this based on the risk classification and the Test Strategy's risk-based testing depth matrix.

**For LOW risk:**

```bash
cat > compliance/evidence/REQ-XXX/test-scope.md << 'EOF'
# Test Scope — REQ-XXX

**Risk Level:** LOW
**Requirement:** [Brief description]
**Date:** [YYYY-MM-DD]

## Test Approach

Standard gates apply. No additional testing beyond universal exit criteria.

- TypeScript compilation: 0 errors
- SAST scan: 0 high/critical findings
- Dependency audit: 0 high/critical vulnerabilities
- E2E suite: all pass
- CI independent verification: all PR checks pass
- Human code review via PR

## Acceptance Criteria

- [x] [Criterion 1 — what "done" looks like]
- [x] [Criterion 2]
EOF
```

**For MEDIUM risk:**

```bash
cat > compliance/evidence/REQ-XXX/test-scope.md << 'EOF'
# Test Scope — REQ-XXX

**Risk Level:** MEDIUM
**Requirement:** [Brief description]
**Date:** [YYYY-MM-DD]

## Test Approach

Standard gates plus targeted verification.

**Universal gates (mandatory — verified locally AND in CI):**
- TypeScript compilation: 0 errors
- SAST scan: 0 high/critical findings
- Dependency audit: 0 high/critical vulnerabilities
- E2E suite: all pass (full suite local, unauthenticated subset in CI)
- Human code review via PR

**Additional testing required by risk level:**
- [ ] Access control: [which endpoints/roles need verification]
- [ ] Audit logging: [which actions must produce log entries]
- [ ] Dependency review: [if new packages, verify real/current/no CVEs]
- [ ] [Any other area-specific testing]

## Validation Approach

How we confirm this meets the business requirement:
- [e.g., "Verify public page displays new content correctly"]
- [e.g., "Confirm edits are visible to users within expected time"]

## Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All additional testing items above pass
EOF
```

**For HIGH risk:**

```bash
cat > compliance/evidence/REQ-XXX/test-scope.md << 'EOF'
# Test Scope — REQ-XXX

**Risk Level:** HIGH
**Requirement:** [Brief description]
**Date:** [YYYY-MM-DD]

## Test Approach

Full verification and validation per Test Strategy high-risk requirements.

**Universal gates (mandatory — verified locally AND in CI):**
- TypeScript compilation: 0 errors
- SAST scan: 0 high/critical findings
- Dependency audit: 0 high/critical vulnerabilities
- E2E suite: all pass
- Human code review via PR

**Security testing (mandatory for HIGH):**
- [ ] Access control: [specific endpoints, roles, expected behaviors]
- [ ] Audit logging: [specific actions that must produce entries]
- [ ] Input validation: [boundary/injection testing needed]
- [ ] Error handling: [verify no sensitive data in error responses]

**Additional high-risk testing:**
- [ ] Independent review: [who will provide secondary review]
- [ ] Penetration testing consideration: [warranted? justification]
- [ ] Performance impact: [load/performance concerns]
- [ ] Regression scope: [areas needing manual verification beyond E2E]

## Validation Approach

How we confirm this meets the business requirement:
- [Specific user workflow to test end-to-end]
- [Business rule to validate]
- [Stakeholder sign-off needed? From whom?]

## AI Involvement (if applicable)

- AI tool: [tool name / none]
- Code categories AI will generate: [list]
- Elevated review required for: [security-sensitive files]
- Regeneration protocol: [will any components be regenerated?]

## Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] All security testing items pass
- [ ] All validation items confirmed
- [ ] Independent review completed (if required)
EOF
```

### Step 6: Update Requirements Document (If Applicable)

If the requirement modifies a documented feature, update the requirements document to reflect the intended change.

### Step 7: Document AI Use Intent (If Applicable)

If AI will generate code (Medium/High risk):

```bash
cat > compliance/evidence/REQ-XXX/ai-use-note.md << 'EOF'
# AI Use Record — REQ-XXX

**AI Tool:** [tool name]
**Planned AI Use:** [implementation / test generation / both / none]
**Risk Classification Impact:** [note if risk was raised due to AI involvement]
EOF
```

For Low risk, the `Co-Authored-By` commit tag is sufficient.

### Step 8: Commit

```bash
git add compliance/RTM.md compliance/evidence/REQ-XXX docs/REQUIREMENTS.md
git commit -m "compliance: [REQ-XXX] define requirement and test scope - [description] [RISK: LOW/MEDIUM/HIGH]"
```

## Output

- REQ-XXX in RTM with `DRAFT` and risk classification
- Evidence directory with test scope (exists before implementation)
- AI use note (if applicable)

## Next Step

Proceed to `2-implement-and-test.md`. Refer back to `test-scope.md` during implementation.
