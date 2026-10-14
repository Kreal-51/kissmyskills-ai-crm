# API and MCP surface

Names and one-line purpose only. All MCP tools take an optional workspace id and run with the identity of the signed-in user. The KissMySkills MCP connector loads the list from the CRM at runtime.

## MCP tools (`crm_*`)

### Pipeline and cards

| Tool | Purpose |
|---|---|
| `crm_today` | What needs attention today: overdue items, reminders, board hygiene |
| `crm_list_boards` | Boards of the workspace with stages, fields and card counts |
| `crm_search_deals` | Find cards on a board by text and/or stage |
| `crm_get_deal` | Full card with attachments and recent journal entries |
| `crm_create_deal` / `crm_update_deal` | Create a card / change fields or stage |
| `crm_add_note` | Add a note or call to a card's journal |
| `crm_move_to_board` | Move a card to another board (e.g. lead to CRM) |
| `crm_bulk_update` | Change many cards or records at once |
| `crm_import_deals` | Import many cards with de-duplication |
| `crm_clear_demo` | Remove template demo cards (real cards untouched) |
| `crm_list_attachments` / `crm_add_attachment` / `crm_remove_attachment` | Link attachments on cards and records |

### Views, queries and analytics

| Tool | Purpose |
|---|---|
| `crm_views` | List, create and update saved views of a board |
| `crm_query` | Search cards with the view filter grammar and return selected fields |
| `crm_export` | Export a board, view or filter as CSV or JSON |
| `crm_analytics` | Performance and pipeline analytics for admins |

### Setup (versioned)

| Tool | Purpose |
|---|---|
| `crm_get_schema` / `crm_apply_schema_ops` | Read the full setup / apply a batch of setup changes as one version |
| `crm_get_config` | Stages, fields and reminders of the current setup |
| `crm_add_stage` / `crm_update_stage` / `crm_rename_stage` / `crm_move_stage` / `crm_remove_stage` | Manage pipeline stages |
| `crm_add_field` / `crm_update_field` / `crm_remove_field` | Manage custom fields |
| `crm_set_lost_reasons` / `crm_set_timer` | Lost reasons and reminder timers |
| `crm_list_templates` / `crm_switch_template` / `crm_apply_template` | Industry templates and migrating live data onto one |
| `crm_history` / `crm_rollback` | Setup version history / restore an earlier version |

### Automations

| Tool | Purpose |
|---|---|
| `crm_rules` | Create and manage "when / if / then" automations |
| `crm_webhooks` | Manage outbound webhook endpoints |

### Workspaces, team and settings

| Tool | Purpose |
|---|---|
| `crm_list_workspaces` / `crm_switch_workspace` / `crm_create_workspace` | Work with several workspaces |
| `crm_accept_invite` | Accept or decline an invitation |
| `crm_list_members` / `crm_invite_member` | Team members, roles and invites |
| `crm_get_settings` / `crm_update_settings` | Personal settings (language, timezone, digest) |

### Find leads

| Tool | Purpose |
|---|---|
| `crm_set_icp` / `crm_get_icp` | Saved ideal-customer profiles |
| `crm_add_leads` | Add companies the user's AI found as lead cards |
| `crm_enrich_lead` | Read public company pages for contact details |
| `crm_draft_outreach` / `crm_list_drafts` | Save and list outreach drafts |

### Hiring

| Tool | Purpose |
|---|---|
| `crm_create_job` / `crm_list_jobs` / `crm_update_job` | Jobs with public pages and application forms |
| `crm_get_resume_text` | Resume text plus the job, for AI screening |
| `crm_score_candidate` | Save the AI's fit score and summary on the candidate |

### Projects mode

| Tool | Purpose |
|---|---|
| `crm_list_spaces` / `crm_create_space` | Project spaces |
| `crm_get_work_config` / `crm_apply_work_template` | Workflow, types, priorities / reset to the default template |
| `crm_search_records` / `crm_get_record` | Find and read records |
| `crm_create_record` / `crm_update_record` / `crm_move_record` | Create, edit and move records |
| `crm_link_records` / `crm_unlink_records` / `crm_add_comment` | Links and comments |
| `crm_add_status` / `crm_update_status` / `crm_move_status` / `crm_remove_status` | Workflow statuses |
| `crm_add_record_type` / `crm_update_record_type` / `crm_remove_record_type` | Record types (epic, story, task...) |
| `crm_add_record_field` / `crm_update_record_field` / `crm_remove_record_field` | Custom record fields |
| `crm_set_link_types` / `crm_set_priorities` / `crm_set_resolutions` | Project vocabularies |
| `crm_import_jira` / `crm_import_report` | Import Jira issue exports and finish the import |

## HTTP API (web app)

Grouped by area; all routes except auth and public job pages need a session.

| Area | Routes (prefix) | Purpose |
|---|---|---|
| Auth | `/login`, `/auth/*`, `/logout` | Sign in with KissMySkills (OAuth) |
| Cards and boards | `/api/deals`, `/api/boards`, `/api/cards/search` | Board data and card edits |
| Setup | `/api/config`, `/api/schema`, `/api/templates` | Versioned setup, schema ops, templates |
| Views and export | `/api/views`, `/api/export` | Saved views, CSV export |
| Automations | `/api/rules`, `/api/webhooks` | Rules, run logs, webhook endpoints |
| Projects | `/api/work/*` | Spaces and records |
| Find leads | `/api/leadgen/*` | Profiles, leads, drafts |
| Hiring | `/api/hiring/*`, `/jobs/*` (public) | Jobs, applications, public job pages |
| Ask AI | `/api/assistant/*` | Chat, confirm destructive actions, own-key management |
| Workspace and account | `/api/workspace*`, `/api/me*`, `/api/account*` | Members, settings, deletion |
| Import | `/api/import` | Bulk import |
| Analytics | `/api/analytics` | Workspace analytics |
| Health | `/health` | Deployed version, used by the release script |

The CRM Worker also exposes a `CrmRPC` service-binding entrypoint (`tools()`, `call()`, session revoke, account deletion) used by the KissMySkills MCP connector.
