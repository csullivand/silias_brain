# Claude Sessions Index

Master index of all Claude Code work sessions.
Organized by project > topic/ticket > dated sessions.

---

## Projects

### silia
Repo: `/Users/sulli/Projects/silia/06-11-25/silia/`

#### Initial Obsidian Setup
- **Branch:** develop
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/Initial Obsidian Setup/2026-04-09|2026-04-09]] — Set up full Obsidian session tracking system with project folders, auto-save, startup logs

#### Close Conversation Endpoint
- **Branch:** feat/SL-1144-close-conversation-endpoint
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/Close Conversation Endpoint/2026-04-13|2026-04-13]] — Created PUT /conversation/{id}/close for n8n, local testing, Postman collection

#### SL-1162 Template Security
- **Branch:** fix/SL-1162-template-security
- **Status:** Complete
- **Sessions:**
  - [[Claude Sessions/silia/SL-1162 Template Security/2026-04-14|2026-04-14]] — Fixed 7 API Gateway model mismatches, domainTopic validation, security lint

#### SL-669 Account Status Stripe Pause/Resume
- **Branch:** feat/SL-669-account-status-change
- **Status:** Implementation done, pending build/deploy/test
- **Sessions:**
  - [[Claude Sessions/silia/SL-669 Account Status Stripe/2026-04-15|2026-04-15]] — Added Stripe pause/resume to AccountStatusService + dunning process planning

#### Close Conversation by ChannelId
- **Branch:** feat/SL-1155-close-conversation-api-main
- **Status:** Complete — cherry-picked to main
- **Sessions:**
  - [[Claude Sessions/silia/Close Conversation by ChannelId/2026-04-21|2026-04-21]] — Changed close endpoint to use channelId instead of conversationId

#### SL-677 Dunning Process
- **Branch:** feat/SL-677-dunnig-process
- **Status:** In progress — code review fixes applied, pending commit
- **Sessions:**
  - [[Claude Sessions/silia/SL-677 Dunning Process/2026-04-22|2026-04-22]] — Eslint fixes, Stripe type improvements, dunning constants, structured logging

#### SL-678 Billing Retrocompatibility
- **Branch:** feat/SL-678-suspension-no-payment
- **Status:** In progress — implemented, pending test/commit
- **Sessions:**
  - [[Claude Sessions/silia/SL-678 Suspension No Payment/2026-04-23|2026-04-23]] — Auto-subscribe logic for pre-billing accounts in chatbot PUT handler

#### SL-1143 Minutes Card RTA
- **Branch:** feat/SL-1143-minutes-card-rta
- **Status:** Complete (continued as SL-1146)
- **Sessions:**
  - [[Claude Sessions/silia/SL-1143 Minutes Card RTA/2026-04-28|2026-04-28]] — RTA agent card in Billing view with minutes, unit cost, usage cost

#### SL-1146 Metering Minutes
- **Branch:** feat/SL-1146-metering-minutes
- **Status:** In progress — infra complete, pending commit/test
- **Sessions:**
  - [[Claude Sessions/silia/SL-1146 Metering Minutes/2026-05-04|2026-05-04]] — Added IAM for BillingRateAudit table, completing infra setup

#### SL-1149 RTA Metered Scheduling
- **Branch:** fix/SL-1149-dunning-rta
- **Status:** In progress — code complete, pending deploy/test
- **Sessions:**
  - [[Claude Sessions/silia/SL-1149 RTA Metered Scheduling/2026-05-11|2026-05-11]] — Fixed RTA metered subscriptions for future dates + activation metered item saving

#### Billing Suspension Audit
- **Branch:** develop
- **Status:** In progress — audit complete, implementation pending
- **Sessions:**
  - [[Claude Sessions/silia/Billing Suspension Audit/2026-05-21|2026-05-21]] — Full codebase audit of suspension feature: backend complete, 4 frontend gaps identified

#### SL-682 Billing Audit + Tax + RTA Sync
- **Branch:** feat/SL-682-audit-logs
- **Status:** Complete — PRs created, CI passing
- **Sessions:**
  - [[Claude Sessions/silia/billing-audit-tax/2026-05-25|2026-05-25]] — Audit log PR fixes, US-TAX-01 automatic_tax, RTA syncRtaStatus HTTP→DynamoDB, checkov baseline

---

### loteria
Repo: `/Users/sulli/Documents/PersonalWork/Loteria/loteria-suerte/`
Sistema de gestión para bancas de lotería en Centroamérica.

#### Setup Session Tracking
- **Status:** In progress
- **Sessions:**
  - [[Claude Sessions/loteria/Setup Session Tracking/2026-05-13|2026-05-13]] — Set up Obsidian session tracking for loteria project