# ANSA LeadGen Engine — Rebuilt Workflow Package

Rebuilt per ANSA 3-layer architecture from the original monolithic LeadGen_Engine.json.

---

## What's in this package

### Layer 2 — Adapters (import first)

| File | Purpose |
|---|---|
| `adapter.crm.get-contacts.hubspot.json` | Fetch & normalize HubSpot contacts with dynamic filters |
| `adapter.crm.update-contact.hubspot.json` | Update any HubSpot contact property |
| `adapter.email.send.gmail.json` | Send email via Gmail OAuth |
| `adapter.ai.research-prospect.perplexity.json` | Perplexity sonar-pro prospect research |
| `adapter.ai.generate-content.claude.json` | Anthropic Claude API content generation |
| `adapter.error.handler.global.json` | Global error handler → Slack alert |

### Layer 1 — Core Logic (import after adapters)

| File | Trigger | Purpose |
|---|---|---|
| `core.leadgen.discovery.json` | Monday 8AM | Pull unworked leads from HubSpot → digest email |
| `core.leadgen.research.json` | Tuesday 8AM | Perplexity research per lead → save to HubSpot |
| `core.leadgen.outreach.json` | Wednesday 8AM | Claude email gen → Day 0/5/10 sequence |
| `core.leadgen.engagement.json` | Thursday 8AM | LinkedIn engagement checklist email |
| `core.leadgen.pipeline.json` | Friday 8AM | Pipeline report + re-engagement for stale leads |
| `core.leadgen.newsletter.json` | 1st of month 9AM | Claude newsletter draft → reviewer email |

---

## Deployment instructions

### Step 1 — Import adapters
Import each `ADAPTER__*.json` into n8n via **Settings > Import Workflow**.
Import all 6 adapters before importing any core workflows.

### Step 2 — Record adapter workflow IDs
After importing each adapter, note the workflow ID assigned by n8n (visible in the URL when the workflow is open).

### Step 3 — Update adapter references in core workflows
In each `CORE__*.json`, replace these placeholder strings with real workflow IDs before importing:

```
ADAPTER_GET_CONTACTS_HUBSPOT_ID       → ID of get-contacts-hubspot
ADAPTER_UPDATE_CONTACT_HUBSPOT_ID     → ID of update-contact-hubspot
ADAPTER_SEND_EMAIL_GMAIL_ID           → ID of send-email-gmail
ADAPTER_RESEARCH_PERPLEXITY_ID        → ID of research-prospect-perplexity
ADAPTER_GENERATE_CONTENT_CLAUDE_ID    → ID of generate-content-claude
ERROR_HANDLER_WORKFLOW_ID             → ID of error-handler-global
```

### Step 4 — Set up credentials in n8n
Add these to the n8n Credentials store (never in node params):

| Credential name | Type | Used by |
|---|---|---|
| `HubSpot (Client)` | HubSpot OAuth2 | get-contacts, update-contact adapters |
| `Gmail (Client)` | Gmail OAuth2 | send-email adapter |
| `Perplexity API (Client)` | HTTP Header Auth (Bearer token) | research-prospect adapter |
| `Anthropic API (Client)` | HTTP Header Auth (x-api-key) | generate-content adapter |
| `ANSA Slack` | Slack API | error-handler-global |

### Step 5 — Update Client Config Set nodes
In each core workflow, open the **Client Config** Set node and replace all `CLIENT_*_HERE` placeholders:

| Placeholder | Value |
|---|---|
| `CLIENT_EMAIL_HERE` | Client's email address |
| `CLIENT_NAME_HERE` | Client's first name |
| `CLIENT_COMPANY_HERE` | Client's company name |
| `CLIENT_TITLE_HERE` | Client's job title |
| `CLIENT_CALENDLY_LINK_HERE` | Client's Calendly URL |
| `CLIENT_VALUE_PROP_HERE` | 1-sentence value proposition for outreach |
| `CLIENT_ICP_DESCRIPTION_HERE` | Description of ideal customer profile |
| `CLIENT_COMPANY_TAGLINE_HERE` | Short company description for newsletter |
| `CLIENT_TARGET_AUDIENCE_HERE` | Newsletter audience description |

### Step 6 — Set error handler globally
In n8n Settings > **Error Workflow**, select `error-handler-global`.
This covers all workflows on the instance with a single error handler.

### Step 7 — Test before activating
1. Manually trigger each core workflow
2. Inspect execution node-by-node
3. Confirm adapters return correctly shaped data
4. Confirm Claude/Perplexity outputs are quality
5. Confirm emails arrive correctly formatted
6. Activate workflows (toggle to Active)

---

## Key changes from original LeadGen_Engine.json

| Original | Rebuilt |
|---|---|
| Single 39-node monolithic workflow | 12 focused workflows across 2 layers |
| Hardcoded email address (ken@...) | Client Config Set node — no hardcoding |
| Credentials embedded in nodes | All credentials in n8n Credentials store |
| Code nodes doing HubSpot calls directly | Normalized adapter sub-workflows |
| No error handling | Global error handler → Slack alert |
| No idempotency checks | Idempotency guards on email sends and research |
| Emails generated with template strings | Claude API generates all outreach emails |
| Day 5/10 emails hardcoded | Claude generates follow-ups with fresh context |
| Single workflow for all 6 triggers | Separate core per day — independent failures |

---

## HubSpot custom properties required

These properties must exist on HubSpot Contact records for the workflows to function:

| Property name | Type | Purpose |
|---|---|---|
| `icp_score` | Text/Number | ICP qualification score |
| `research_notes` | Multi-line text | Perplexity research output |
| `buying_trigger` | Text | Identified buying signal |
| `outreach_sequence_status` | Dropdown | Not Started / Day 0 Sent / Day 5 Sent / Complete |
| `outreach_day0_sent` | Boolean | Idempotency flag |
| `outreach_day5_sent` | Boolean | Idempotency flag |
| `outreach_day0_date` | Date | Timestamp of Day 0 send |
| `research_completed_at` | Date | Timestamp of research completion |
| `sequence_completed_at` | Date | Timestamp of sequence completion |

---

*ANSA Solutions · LeadGen Engine v2.0 · March 2026*
