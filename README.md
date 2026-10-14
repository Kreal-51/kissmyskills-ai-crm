# KissMySkills AI CRM

**A CRM you can run from your AI assistant: Claude (or another MCP client) reads, updates and reconfigures your pipeline through tools, and the web app shows the same data.**

Live: [app.kissmyskills.com](https://app.kissmyskills.com) (sign in with a KissMySkills account) · Product site: [kissmyskills.com](https://kissmyskills.com)

---

## The problem

Small teams already "talk" to an AI assistant all day, but their customer data sits in a separate CRM that the assistant cannot see or change. So they copy-paste between the two, and the CRM drifts out of date. Classic CRMs also ask for setup work (stages, fields, views) that a non-technical owner never gets round to.

## Who it's for

Freelancers, small agencies and small businesses (sales, recruiting, support, real estate, trades and similar) who use an AI assistant daily and want their pipeline to be something the assistant can work on directly.

## Key features

- **MCP-first CRM.** More than 80 `crm_*` tools let a connected AI search and update deals, add notes, reorganise stages and fields, import data, and summarise "what needs attention today".
- **Setup changes by conversation, with a safety net.** Board setup (stages, fields, reminders, wording) is versioned. Batch changes support a dry run, a diff and a confirm step, and any earlier version can be rolled back.
- **Several boards in one workspace.** AI CRM, Lead generation, Receptionist leads, Customer support, Hiring, plus user-defined boards, with cards that can move between boards.
- **Projects mode.** Jira-style spaces, records, epics/subtasks, links, comments and history, with an importer for Jira issue exports.
- **Saved views.** Board, table and calendar views, a shared filter grammar (used by views, queries and export), colour rules, formula and rollup fields, CSV/JSON export.
- **Automations and webhooks.** "When X, if the card matches, then Y" rules with a run log, plus signed outbound webhooks.
- **Find leads.** Saved ideal-customer profiles; the user's own AI finds companies, the CRM stores them, enriches contact details from public company pages (respecting robots.txt) and keeps outreach drafts.
- **Hiring.** Public job pages with an application form, resume upload, resume-to-text for AI screening, and fit scores saved on the candidate card.
- **In-app "Ask AI".** A chat panel that uses the same CRM tools. It runs on Cloudflare Workers AI by default, or on Claude with the user's own Anthropic key (encrypted at rest). Destructive actions are never executed by the model: they come back as a Confirm card.
- **18 industry templates**, team workspaces with roles and invites, daily digest e-mail, interface in 10 languages.

## How it works

```mermaid
flowchart LR
  subgraph Clients
    AI["Claude / other MCP client"]
    WEB["Web app (browser)"]
    CAND["Job applicants (public job pages)"]
  end

  subgraph Cloudflare
    MCP["KissMySkills MCP Worker<br/>OAuth 2.1 + tool registry"]
    CRM["CRM Worker<br/>(API, UI, CrmRPC entrypoint)"]
    D1[("D1 (SQLite)<br/>workspaces, boards, cards,<br/>records, views, rules")]
    KV[("KV<br/>OAuth state, session revocation,<br/>runtime feature flags")]
    R2[("R2<br/>resumes, nightly D1 backups")]
    WAI["Workers AI<br/>(Ask AI default models)"]
    CRON["Cron triggers<br/>(hourly jobs, nightly backup)"]
    TOOLS["Other KissMySkills tool Workers<br/>(service bindings)"]
  end

  ANT["Anthropic API<br/>(user's own key, optional)"]
  RES["Resend (e-mail)"]
  PH["PostHog (product analytics)"]

  AI -->|MCP over HTTPS| MCP
  MCP -->|service binding RPC:<br/>tools() / call()| CRM
  WEB --> CRM
  CAND --> CRM
  CRM --> D1
  CRM --> KV
  CRM --> R2
  CRM -->|Ask AI| WAI
  CRM -.->|Ask AI, BYO key| ANT
  CRM --> RES
  CRM -.-> PH
  CRM --> TOOLS
  CRON --> CRM
```

- **The CRM owns its tool list.** The MCP Worker fetches `tools()` from the CRM over a Cloudflare service binding and forwards calls with the already-authenticated user. If the CRM is slow or down, the CRM tools simply disappear from that session and the rest of the connector keeps working.
- **One code path for every caller.** MCP tools, the in-app assistant and the web UI run through the same operation layer, so permissions, plan limits and validation live in one place.
- **Most AI work runs in the user's own assistant.** Lead search, outreach copy and candidate screening are done by the user's AI; the CRM stores results and enforces rules. The only server-side LLM calls are the optional in-app assistant (Workers AI or the user's own Claude key).
- **Environments.** Local dev (no production bindings), a separate staging Worker with its own D1/KV/R2, and production. Releases go staging first, then production by an approved commit, with a `/health` version check after each deploy.

## Tech stack

- **Runtime:** Cloudflare Workers (JavaScript, ES modules), cron triggers, service bindings
- **Data:** Cloudflare D1 (27 SQL migrations), KV, R2
- **AI:** Model Context Protocol (tools served via the KissMySkills MCP connector), Cloudflare Workers AI (Llama / Qwen / Mistral function-calling models), Anthropic SDK for bring-your-own-key Claude
- **Auth:** "Sign in with KissMySkills" (OAuth via `@cloudflare/workers-oauth-provider`), workspace roles
- **Other:** Resend for e-mail, PostHog for product analytics, `unpdf` for resume text, Node-based unit and API tests (70+ test files), i18n and template checks in the test script

## My role

I'm the product manager and the builder. I built this with Claude Code as my engineering pair:

- **Framing problems.** I started from my own sales and hiring work: what I kept copying between my assistant and my CRM.
- **Writing specs.** Each change started as a short written spec: user story, acceptance checks, rollout flag, rollback.
- **Prioritising.** I picked what to ship next from real usage (PostHog events, support feedback) and dropped or postponed the rest.
- **Shipping in small branches.** One feature or fix per branch. The repo has 100+ short-lived branches and dozens of tagged production releases.
- **Verifying on staging.** Every release went to a staging copy first, and I checked it there before production. New features stayed behind runtime flags until I had looked at them.

## What I learned / product decisions

- **MCP-first, UI second.** Designing tools for an AI caller forced clear contracts: every tool returns structured errors with hints and a link back to the right screen in the app. That made the web UI simpler too, because both use the same operations.
- **Guardrails for AI writes.** Letting an AI change the CRM's setup is powerful and risky. Setup is versioned with dry runs, diffs, confirm tokens and rollback. Bulk and destructive actions need an explicit human confirm, and demo data is clearly separated from real data.
- **Cost-aware AI by default.** The in-app assistant uses a free tier of Workers AI with per-plan limits and AI cost caps, and power users can bring their own Claude key. The heavier AI work stays in the user's own assistant, so the product doesn't pay for it.
- **Release discipline for a one-person team.** Staging with its own data, runtime feature flags, nightly D1 backups to R2, and a scripted staging-to-production flow made it safe to ship several times a day without a QA team.

More detail: [data model sketch](docs/data-model.md) · [API and MCP surface](docs/api-surface.md)

---

Source is private; happy to walk through it in an interview.
