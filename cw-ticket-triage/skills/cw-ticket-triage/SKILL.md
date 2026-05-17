---
name: cw-ticket-triage
description: "Pull the user's open ConnectWise tickets, summarize by board, status, and priority, and propose triage actions. Use when Travis or another RX Tech user says \"what's on my plate\", \"triage my tickets\", \"open tickets\", \"my queue\", \"what's assigned to me\", or asks about specific board status. Also triggers on direct ticket-number references like \"what's the status of 12345\" or \"look up ticket 98765\"."
---

# ConnectWise Ticket Triage

This skill helps an RX Tech user triage their open ConnectWise tickets. It uses the `cw-*` tools served by the RX Tech MCP server at `mcp.rxtech.app/mcp`.

## When to use

Triggers on any of:

- "what's on my plate", "my queue", "what's assigned to me"
- "triage my tickets", "open tickets", "show my tickets"
- A specific ticket reference: "ticket 12345", "what's the status of 98765", "look up #54321"
- Any board/status query: "what's open on the Service board", "any P1 tickets?"

## Steps

### 1. Identify the user

The user's ConnectWise member id is the same as their authenticated MCP token's `cw_member_identifier`. If you don't know it, call `cw-get-members` with a filter for the current user's email to resolve it. Most asks are about the user running the conversation; only ask whose tickets when explicitly ambiguous.

### 2. Pull open tickets

Call `cw-get-tickets` with these parameters:

- `conditions`: `closedFlag=false AND resources contains "{member_id}"` (use ConnectWise's filter syntax — the tool accepts the conditions string verbatim)
- `pageSize`: 100 (most users have <50 open; cap at 100 to avoid context bloat)
- `orderBy`: `priority/sort asc, lastUpdated desc`
- `fields`: `id, summary, board/name, status/name, priority/name, lastUpdated, dateEntered, contact/name, company/name`

If the user named a specific board or status in their ask, add it to the conditions: e.g., `board/name = "Service"` or `status/name contains "Waiting"`.

If the user referenced a specific ticket number, skip directly to step 4 with that ticket id.

### 3. Render the triage table

Show a compact markdown table sorted by priority (P1 first) then by `lastUpdated desc`. Columns:

| # | Summary | Board | Status | Priority | Stale |
|---|---|---|---|---|---|

The `Stale` column shows days since `lastUpdated`. Format as `3d` or `2w`. Flag any row meeting these criteria with a leading badge:

- `🔥 P1+3d` — Priority 1 + stale 3+ days (immediate attention)
- `⚠️ P2+5d` — Priority 2 + stale 5+ days
- `📥 New` — Created within last 24 hours, not yet status-shifted from "New"

After the table, summarize: `N open tickets across M boards. K need immediate action.`

### 4. Specific-ticket lookup

If the user asked about one ticket by number, call `cw-get-tickets` with `conditions: id = {n}` (or use the singular endpoint if it exists in the surface). Render a detailed card:

```
Ticket {id}: {summary}
  Board: {board.name}
  Status: {status.name}
  Priority: {priority.name}
  Owner: {owner.identifier}
  Contact: {contact.name} ({company.name})
  Created: {dateEntered}
  Last updated: {lastUpdated}

  Latest note (truncated to 280 chars): {latest_note}
```

If there's a recent note, surface its first 280 characters. If not, say "No recent notes."

### 5. Triage proposal

After step 3 or 4, propose actionable next steps. Examples:

- "Ticket 12345 (P1 in Service, 4 days stale) — recommend status flip to 'Waiting Customer' and ping the requester."
- "Two P2 tickets on the LV board both last touched the same day — looks like batch from the same incident; consider grouping."

**Do not write to ConnectWise without explicit user confirmation.** Always describe the proposed change in natural language first ("I'll flip 12345 to 'Waiting Customer' and add an internal note saying X — proceed?"), then act only after the user says yes.

## Write actions (only on user confirmation)

- **Status change:** `cw-update-ticket` with `{id, status: {id: <status_id>}}`. Use `cw-describe-status-mapping` if you need the status_id for a status name; cache the result for the rest of the conversation.
- **Priority change:** `cw-update-ticket` with `{id, priority: {id: <priority_id>}}`. Use `cw-describe-priorities` to resolve names → ids.
- **Internal note:** `cw-create-time-entry` with `internalNotes` field if the user wants a private note; or use the ticket note endpoint if it's in the surface.

For any write action, surface a one-line confirmation message after the call succeeds: "Updated 12345 → Waiting Customer." If the call fails, surface the JSON-RPC error verbatim, then propose the next step.

## Out of scope

- Multi-user team triage (other people's queues) — separate skill if needed
- Creating brand-new tickets — separate skill `cw-ticket-create` if requested
- SLA reporting / time-entry summarization — separate skill `cw-time-tracking` if requested
- Cross-customer aggregation — RX Tech-internal triage only; if MSP-wide views are needed, scope a new skill

## Reference

The ConnectWise tools available on the RX Tech MCP server (kebab-case namespaced):

- Read: `cw-get-tickets`, `cw-get-companies`, `cw-get-contacts`, `cw-get-members`, `cw-get-time-entries`
- Describe (taxonomies): `cw-describe-boards`, `cw-describe-statuses`, `cw-describe-priorities`, `cw-describe-types`, `cw-describe-status-mapping`
- Write: `cw-update-ticket`, `cw-create-ticket`, `cw-create-time-entry`, `cw-update-time-entry`, `cw-delete-time-entry`
- Escape hatch: `cw-raw` for any ConnectWise REST endpoint not yet surfaced as a typed tool

Treat any tool call's error as terminal for that path — do not retry the same call with different args without surfacing the failure to the user first.
