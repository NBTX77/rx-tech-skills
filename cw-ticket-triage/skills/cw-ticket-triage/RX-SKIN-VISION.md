# RxTechApp — Project Vision & Goals

> **Owner:** Travis Brown / RX Technology
> **Last updated:** 2026-04-28
> **Status:** v2 in active build at mcp.rxtech.app · v0.17.0 · 1352 tests · v1 frozen at rxtech.app
> **History:** v1 (RxSkin) was the first attempt; v2 (RxTechApp) is a complete green-field rebuild started April 2026.

---

## What Is RxTechApp

RxTechApp is an AI-powered operations platform that replaces ConnectWise Manage's frontend and unifies the entire MSP service delivery lifecycle into a single modern interface.

It serves every role at RX Technology — Account Managers, Service Integration, IT, Leadership, and Finance/Procurement. The platform tracks work from the moment a client says "yes" through scoping, execution, procurement, deployment, and billing. Every department sees what matters to their role without digging through ConnectWise or chasing people down.

**RxTechApp's data model is vendor-neutral.** ConnectWise is the primary PSA today, but the architecture preserves the option to replace, supplement, or run alongside an alternative ticketing system without a schema migration.

**Today RxTechApp serves a single MSP (RX Technology). The active product is multi-customer — RX Tech sees all client organizations.** Architecture preserves the option to onboard additional MSPs as separate tenants in the future, but commercialization is not committed today.

The AI layer is what makes it different from a better-looking CW frontend. The platform watches how the team actually works — what they click, where they spend time, where tickets stall, where handoffs fail. It learns the operation and helps improve it over time. **External AI agents can also operate the platform via MCP** — the AI layer is both internal observer and external surface.

---

## Why It Exists

ConnectWise Manage is a database with a bad interface. The problems it creates for RX Technology:

- **Technicians** juggle 10+ browser tabs to check security tools, network monitors, credentials, and email while working a ticket.
- **Account Managers** can't easily see where their clients' work stands without digging through boards and tickets.
- **Leadership** has no real-time view of utilization, margin, or bottlenecks — they have to ask around or build reports.
- **Finance/Procurement** tracks POs, materials, and billing in spreadsheets because CW doesn't connect those dots.
- **Everyone** loses work at department handoffs because there's no visibility into who owns what after a transition.

RX Skin fixes all of this by pulling everything into one place and making the invisible visible.

---

## Core Goals

### 1. One app for every role
Every department at RX Technology should be able to do their daily work inside RX Skin without opening ConnectWise, spreadsheets, or chasing people for status updates. Each role gets a view tailored to what they need — not a generic ticket dashboard.

### 2. Cross-department visibility
The full lifecycle of every engagement — from AM handoff through ops execution, procurement, deployment, and billing — should be visible to anyone who needs to see it. When leadership asks "where is the Smith Office buildout?" the answer should take one click.

### 3. Eliminate context-switching for operations
Technicians should never leave RX Skin to check Duo auth logs, SentinelOne threats, Passportal credentials, Umbrella DNS blocks, Auvik network status, or Proofpoint quarantine. Every MSP tool the team uses daily should surface contextual data and actions directly on the ticket.

### 4. AI that learns the operation
The platform collects behavioral analytics (clicks, navigation, scroll depth, dead zones), workflow data (ticket lifecycle events, bottleneck detection), and team feedback. The AI uses this data to:
- Surface bottleneck alerts and handoff failures to leadership
- Suggest workflow improvements based on observed patterns
- Eventually propose and automate digital workflows through the Nexus middleware engine
- Help the team iterate the interface and processes based on real usage, not guesswork

External AI agents can also operate the platform via MCP — the AI layer is both internal observer (watching how the team works) and external surface (callable by Claude, ChatGPT, and other agentic clients via the MCP server at `mcp.rxtech.app`).

### 5. Bridge digital and human workflows
Not everything happens in software. Someone creates a PO in QuickBooks, someone physically installs equipment, someone calls a vendor. RX Skin should be able to define, visualize, and track these real-world steps alongside digital ones — so the full picture of a job is always visible regardless of which steps are automated and which are manual.

### 6. Self-improving platform
The feedback loop is built into the architecture: the team uses the platform (or an external AI agent operates it via MCP) → analytics collect behavioral data → the AI identifies patterns and problems → improvements are made → the cycle repeats. Every new feature gets the same treatment. The platform gets smarter about itself over time. MCP-driven agentic operations are one input into the loop, not a separate one.

---

## Who Uses It and What They Need

Every user in RxTechApp has **two identity axes**: a **permission role** (`admin` / `manager` / `technician` / `viewer`) that determines what actions they can take, and **zero or more department memberships** (IT, SI, AM, GA, LT) that determine which views and tailored frontends they see. A user can belong to multiple departments. Travis Brown is the canonical dual-role example — `admin` permission tier + `LT` department membership.

The five real departments map 1:1 to the five user-role views below. Department slugs `LT` (= Leadership Team) and `GA` (= Finance & Procurement) are stored as legacy short codes; display names are friendly. A sixth meta-role, `admin`, is not a department.

### AM — Account Managers
- Which clients have open work and what's the status of each engagement
- What's overdue or at risk
- Client health scores and renewal status
- Ability to hand off work to operations with full context

### SI — Service Integration
- Active projects with resource allocation and timelines
- Project financial rollup (hours vs. budget, materials, margin)
- Ticket and project boards with department-appropriate views
- MSP tool data in context (network, security, licensing)
- Physical-security install/integration tools (UniFi, Hanwha, DW Spectrum, DNA Fusion, Verkada, Genea, PDK) — gated on SI frontend ship in PHASE-11+

### IT — Technicians
- Ticket list with search, filters, and mobile support
- Ticket detail with notes, time entries, and remote tools
- MSP tool overlays: Duo, Auvik, SentinelOne, M365, Passportal, Meraki, Webex (the PHASE-10 7-integration floor)
- AI triage, auto-categorization, draft notes, and the RX Assistant chatbot
- M365 mail, Teams, and calendar without leaving the app
- Webex calling, voicemail, and helpdesk queue

### LT — Leadership Team
- Utilization rates and throughput by department
- Margin by client, project, and engagement type
- Bottleneck summary — where work stalls and why
- AI-generated insights and improvement suggestions

### GA — Finance & Procurement
- Open POs and materials tracking
- Unbilled time and invoice queue
- License correlation — coverage gaps and billing reconciliation across all platforms
- Project P&L: time logged, materials cost, billable vs. non-billable, margin

### admin (meta-role)
- Full platform access regardless of department
- MCP server management, integration health, credential vault, kill switches
- Per-role grant administration, alias system, audit log review
- Today: Travis only. Multi-user transition expands `admin` to additional Travis-designated users (cluster 8).

---

## Design Principles

- **BFF pattern is sacred.** No external API calls from the browser. All credentials server-side only.
- **Overlay cards, not side-by-side panels.** Collapsible floating 520px bubble cards for contextual data (like fleet map sidebar pattern).
- **Feature flags gate everything.** Every integration and AI feature can be toggled per-tenant without code changes.
- **Document first, automate later.** New workflows start as human checklists. Deep links come next. API automation comes last.
- **The team's behavior shapes the product.** Analytics and AI feedback drive prioritization, not roadmap assumptions.

---

## What Exists Today (v2 — v0.17.0, 2026-04-28)

v2 status as of MCP-MGMT shipment. v1 inventory (rxtech.app, RxSkin codebase) is frozen — see sibling `Projects-Archive/CW-RxSkin-V1-Archive/v1-archive/` (moved out of this project 2026-05-13) for historical record.

- **Production:** mcp.rxtech.app (Vercel, auto-deploy from `main`)
- **Repo:** github.com/NBTX77/RxSkin (v2 in `RxTechApp/`); v1 frozen at commit `53a3baa`
- **Test count:** 1352 (1186 → 1352 across MCP-MGMT)
- **Phases complete:** Phase 0–6 (infrastructure, observability, queue, schema, integrations, AI layer) + MCP-1 + MCP-2 + MCP-COWORK-INSTALL + MCP-MGMT
- **Live integrations:** 13 (CW, M365, Auvik, Meraki, Duo, SentinelOne, Passportal, Samsara, SmileBack, Umbrella, ScalePad, Webex auth-only, Datto creds-pending). Roster total: 37 planned across 3 tiers (15 carry-forward + 9 new + 9 distributors + 3 AI providers + 1 QB).
- **MCP server:** 196 kebab-case tools across all 13 integrations + RX platform tools, exposed at `mcp.rxtech.app`. Full MCP-MGMT management UX (kill switch, per-role grants, alias system, Access tab, permissions matrix, simulator).
- **Specs:** 27 domain specs in `docs/specs/`. Master decisions in `RX-TECHAPP-V2-ARCHITECTURE-DECISIONS.md`.
- **Hard Rules:** 26 enforced (BFF sacred, RLS on every Postgres table, theme tokens only, AI cost governance, WCAG 2.1 AA, etc.).
- **AI layer:** Anthropic Claude live; OpenAI + xAI configurable when keys entered.
- **Auth posture:** single-user gate (Travis only) until PHASE-10 mid-phase gate flip.

## What's Next

Current v2 phase pipeline (full version: `RX-TECHAPP-V2-ARCHITECTURE-DECISIONS.md §5` and Notion roadmap):

1. **DOC-REFRESH** (in progress) — clear the 48-cluster goal review punch list across vision, architecture, dashboard, and memory.
2. **MCP-HOTFIX-001** — 12 P0/P1/P2 findings from 2026-04-27 exploratory POC. Travis-paced, one fix per session.
3. **STABILIZE** — 7 sub-milestones (ST-1 to ST-7) covering integration completion, cred backfill, AI deepening, documentation, Redis chattiness, test gap-fill, passkeys/WebAuthn. Sub-milestones scoped after MCP-HOTFIX-001 closes.
4. **PHASE-10 — IT Frontend MVP** — first department frontend ships. Full IT toolset (7-integration overlay floor + full AI suite), multi-user gate flips mid-phase.
5. **PHASE-11+** — AM, SI, GA, LT frontends incrementally. Long-range items below land per-department:
   - Department-tailored landing pages (per dept route)
   - Ticket lifecycle timeline (cross-department visibility)
   - Handoff alerts (stalled work detection)
   - Human workflow engine (templates + instances + 3 visualization views)
   - Project financial rollup (unified P&L per engagement)

---

*This document is the north star for all development decisions. When in doubt about how to implement a feature, check whether the approach serves the goals above. If it only serves IT, consider whether it could serve the whole team instead.*
