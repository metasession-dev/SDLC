# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repository Is

This is a compliance-grade SDLC template system — a set of 12 documents across two tiers that implement a workflow-driven development pipeline satisfying ISO 29119, ISO 27001, GDPR, SOC 2, and EU AI Act requirements. It is not a software application; it contains markdown templates and an accompanying article.

## Repository Structure

- `article.md` — Long-form article explaining the system's design and rationale
- `files/` — The 12 template documents:
  - **Tier 1 (universal, never project-specific):** `Test_Policy.md`, `Test_Strategy.md`, `Test_Architecture.md`, `Periodic_Security_Review_Schedule.md`
  - **Tier 2 (project-specific, customized per project):** `0-project-setup.md` through `5-deploy-main.md` (workflows), `Test_Plan_TEMPLATE.md`, `README_TEMPLATE.md`

## Key Design Principles

- **Single responsibility per document.** Policy says *what* and *why*, Strategy says *how methodically*, Architecture says *with what tools*. No content should appear in two places.
- **Tier 1 never references a specific project.** Tools are referenced by category in Policy/Strategy; only Architecture names specific products and versions.
- **Risk-based proportionality.** Documentation, testing depth, and review requirements scale with risk level (Low/Medium/High). AI involvement raises risk by one level for Medium/High categories.
- **Workflow-driven compliance.** The six workflows (0-5) are executable procedures with specific commands, not guidelines. Compliance artifacts are natural outputs of following the workflows.

## Workflow Sequence

0. **Project Setup** (run once) — repo, branches, CI, compliance directories
1. **Plan Requirement** — define in RTM, classify risk, generate test scope *before* implementation
2. **Implement and Test** — code on `develop`, all local gates green on every commit
3. **Compile Evidence** — gather artifacts, create release ticket, update RTM
4. **Submit for Review** — PR triggers independent CI verification + human review
5. **Deploy to Main** — merge, verify deployment, finalize compliance artifacts

## Mandatory Gates

All templates assume these gates: TypeScript (0 errors), SAST/Semgrep (0 high/critical), dependency audit (0 high/critical), E2E tests (all pass), human PR approval.

## Conventions in Templates

- **Commit format:** Conventional Commits with `Co-Authored-By` tags for AI, `Ref: REQ-XXX` for tracked requirements
- **Requirement IDs:** `REQ-XXX` format, tracked in `compliance/RTM.md`
- **Status lifecycle:** DRAFT → IN PROGRESS → TESTED - PENDING SIGN-OFF → APPROVED - DEPLOYED
- **Branching:** permanent `develop` branch, protected `main` (production), merge commits to preserve audit trail
- **Evidence model:** local testing (comprehensive) + CI testing (independent verification, tamper-resistant)
- **Placeholders:** Templates use `[BRACKETED_VALUES]` and `# UPDATE` markers for project-specific customization

## When Editing These Templates

- Maintain the separation between Tier 1 and Tier 2 — if content applies universally, it belongs in Tier 1; if project-specific, Tier 2.
- Keep workflow documents as executable procedures with concrete commands, not abstract guidance.
- Risk classification rules and AI governance controls are defined in Test_Policy.md and Test_Strategy.md — don't duplicate them in workflow files.
- The article (`article.md`) should stay consistent with the templates; if you change a process in a template, check whether the article describes the old process.
