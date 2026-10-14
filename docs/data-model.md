# Data model (sketch)

A simplified view of the main entities in Cloudflare D1. Columns are omitted on purpose; only the key relations are shown. Every business table is scoped by workspace.

```mermaid
erDiagram
  WORKSPACE ||--o{ MEMBERSHIP : has
  USER ||--o{ MEMBERSHIP : "belongs via"
  USER ||--|| USER_SETTINGS : has
  WORKSPACE ||--o{ WORKSPACE_CONFIG_VERSION : "setup history"

  WORKSPACE ||--o{ CARD : "has (per board)"
  CARD ||--o{ CARD_EVENT : "journal: notes, calls, stage moves"
  CARD ||--o{ CARD_ATTACHMENT : "link attachments"
  CARD ||--o{ OUTREACH_DRAFT : "lead outreach"

  WORKSPACE ||--o{ VIEW : "saved views per board"
  WORKSPACE ||--o{ RULE : "automations per board"
  RULE ||--o{ RULE_RUN : "run log"
  WORKSPACE ||--o{ WEBHOOK_ENDPOINT : has
  WEBHOOK_ENDPOINT ||--o{ WEBHOOK_DELIVERY : logs

  WORKSPACE ||--o{ ICP_PROFILE : "Find leads"
  WORKSPACE ||--o| JOB_BOARD : "careers page"
  WORKSPACE ||--o{ JOB : has
  JOB ||--o{ APPLICATION_FILE : "resumes (R2)"
  APPLICATION_FILE }o--|| CARD : "candidate card"

  WORKSPACE ||--o{ SPACE : "projects mode"
  SPACE ||--o{ RECORD : contains
  RECORD ||--o{ RECORD : "parent / children"
  RECORD ||--o{ RECORD_LINK : "blocks, relates..."
  RECORD ||--o{ COMMENT : has
  RECORD ||--o{ RECORD_EVENT : history
  RECORD ||--o{ ATTACHMENT : has
  PERSON }o--o{ RECORD : "assignee / reporter"

  USER ||--o{ AI_KEY : "own Anthropic key (encrypted)"
```

## Notes

- **Workspace.** A separate CRM. A user can belong to several workspaces with a role (admin or member); there is also a personal workspace per account.
- **Board setup is versioned.** Stages, custom fields, reminders and wording are stored as numbered versions per workspace. That is what makes dry runs, diffs and rollback possible.
- **Cards.** One table holds the cards of every board (AI CRM, Lead generation, Receptionist, Support, Hiring and user-defined boards). Custom field values live with the card. Formula and rollup fields are computed on read.
- **Projects mode** is a separate, Jira-like model: spaces, records with human-readable keys, hierarchy, links, comments and history. Imported records keep their external ids so re-imports stay idempotent.
- **Demo data** from templates is flagged and can be cleared without touching real cards.
- **Files.** Resume files go to a private R2 bucket. The database holds only metadata and extracted text. Nightly D1 backups go to a separate private R2 bucket.
- **Supporting tables** (not shown) cover usage counters, product events, onboarding state and admin audit logs.
