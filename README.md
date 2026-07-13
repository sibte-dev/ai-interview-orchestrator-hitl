# AI Interview Orchestrator (Human-in-the-Loop Demo)

> **Status:** Demo architecture built with synthetic candidate data. No real candidate PII, employer data, or production credentials are included in this repository.

An n8n workflow that automates interview-stage decisioning while keeping a recruiter in control of every advance/reject decision — built around 2026-era agentic automation practices: multi-model task routing, human-in-the-loop approval gates, and a feedback loop for continuous improvement.

## Why human-in-the-loop, not full autonomy

Fully autonomous hiring decisions carry real fairness, legal, and reputational risk. This workflow deliberately stops at a **Wait Node** before any candidate is advanced or rejected, and requires explicit recruiter sign-off. Nothing here auto-rejects a candidate.

## Architecture

```
Webhook (candidate intake)
        │
        ▼
Set Node (synthetic candidate payload)
        │
        ▼
Agent A — Skills-Based Scorer (Claude)
   scores candidate on demonstrated skills/competencies,
   not job titles or degrees
        │
        ├──────────────────────────────┐
        ▼                               ▼
Agent B — Question Generator      IF: score >= 70
   (GPT-4o-mini)                       │
   generates targeted competency   ┌───┴────┐
   interview questions             ▼        ▼
                          HITL Approval   Log outcome
                          (Wait Node)     (feedback loop)
                                │
                    ┌───────────┴───────────┐
                    ▼                       ▼
            Slack: notify scheduling   Log outcome
                    │                       │
                    └───────────┬───────────┘
                                ▼
                      Respond to Webhook
```

## Components

| Node | Purpose |
|---|---|
| `Webhook: Candidate Intake` | Entry point — would connect to an ATS or application form in production |
| `Set: Synthetic Candidate Payload` | Demo data substitute for real intake data |
| `Agent A: Skills-Based Scorer` | Claude call scoring the candidate against job requirements on skills, not keywords |
| `Agent B: Question Generator` | Separate model (GPT-4o-mini) generates interview questions targeting the candidate's weak areas — deliberately split from scoring to demonstrate multi-model orchestration |
| `IF: Score >= 70` | Routing logic; low scores skip straight to logging rather than the approval gate |
| `HITL Gate: Recruiter Approval` | Wait node — workflow pauses until a recruiter submits a decision |
| `Log Outcome` | Writes recruiter decisions back to a sheet, forming a feedback dataset for future scoring-prompt tuning |
| `Slack: Notify Scheduling Approved` | Downstream action once a human approves |

## Setup (to run this yourself)

1. Self-hosted n8n via Docker, or n8n Cloud free tier
2. Import `workflow.json` via **Workflows → Import from File**
3. Add credentials for Anthropic and OpenAI (free-tier keys work for demo volume)
4. Replace `SYNTHETIC_SHEET_ID` with your own Google Sheet, or swap the node for Postgres/Airtable
5. Trigger the webhook with a sample payload to see the flow execute end-to-end

## What this repo intentionally does not include

- Real candidate data or resumes
- Production API keys or credentials
- A live ATS integration (would require client-specific configuration)

## Roadmap / not yet built

- Compliance/audit-log sub-workflow (GDPR / EU AI Act style logging)
- Bias-check step on Agent A's scoring output
- Multi-candidate batch mode
