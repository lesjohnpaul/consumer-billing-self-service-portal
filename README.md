# Consumer Billing & Self-Service Portal

> Members check bills and track outage repairs from their phones, and the billing database is never exposed to the internet.

**Status:** Live · **Client:** 250K-member electric cooperative · **Role:** Solo architect & full-stack engineer

**Stack:** `Next.js` `Supabase` `Cloudflare Tunnel` `Sync Agent` `MS SQL Server`

> [!NOTE]
> **Showcase repository.** The source code is private and belongs to the client. This repo documents the problem, the architecture, and the results. Screens show demo data. Full case study: https://lesjohnpauloliver.dev/work/consumer-billing-self-service-portal

## Summary

A self-service portal where consumers see their bills, payments, and usage, report outages and follow the repair, while the billing system itself never touches the internet.

## Scale

- Live in production, serving member-consumers across 17 towns and cities
- 370+ test cases across 29 suites
- 13-month usage history per account

## The problem

- Members had to wait for the paper bill or line up at an office to learn what they owed.
- The billing database is live, shared with tellers, and must never be exposed to the internet.
- Outage complaints came by phone and walk-in, with no way for the member or management to follow them.

## Problem → fix

### Members had to wait for the paper bill, or line up at an office, just to learn what they owed.

Balance, due date, and an unpaid-month warning on one screen, for every account the member owns, on any phone.

<img src="images/desktop-home.webp" alt="Overview: balance, due date, unpaid warning, last payment, latest usage, and a severe-weather alert." width="100%"/>
<sub>Overview: balance, due date, unpaid warning, last payment, latest usage, and a severe-weather alert.</sub>
<img src="images/desktop-bills.webp" alt="Every bill for every account the member owns, with paid and unpaid status." width="100%"/>
<sub>Every bill for every account the member owns, with paid and unpaid status.</sub>
<img src="images/mobile-home.webp" alt="Home on a phone: balance and one-tap actions." width="100%"/>
<sub>Home on a phone: balance and one-tap actions.</sub>
<img src="images/mobile-bills.webp" alt="Bills on a phone." width="100%"/>
<sub>Bills on a phone.</sub>

### A higher bill was one number nobody could explain, so members called or walked in to ask why.

Every bill reads line by line (generation, transmission, distribution, taxes) with a plain-language reason it changed, next to the published rate schedule.

<img src="images/desktop-bill-detail.webp" alt="One bill, line by line, with a plain-language reason it changed from last month." width="100%"/>
<sub>One bill, line by line, with a plain-language reason it changed from last month.</sub>
<img src="images/desktop-rates.webp" alt="The published rate schedule, so members can check the math themselves." width="100%"/>
<sub>The published rate schedule, so members can check the math themselves.</sub>

### Members only found out their bill would jump after it arrived.

13 months of usage, comparisons to last month and last year, a next-bill estimate, and an appliance calculator at the member's own rate.

<img src="images/desktop-usage.webp" alt="13-month usage, comparisons to last month and last year, and a next-bill estimate." width="100%"/>
<sub>13-month usage, comparisons to last month and last year, and a next-bill estimate.</sub>
<img src="images/desktop-calculator.webp" alt="Appliance cost estimator at the member's own effective rate per kWh." width="100%"/>
<sub>Appliance cost estimator at the member's own effective rate per kWh.</sub>
<img src="images/mobile-usage.webp" alt="Usage chart on a phone." width="100%"/>
<sub>Usage chart on a phone.</sub>

### A payment made through GCash, Bayad Center, or another office could not be confirmed without a call.

Payment history across every channel, plus a where-to-pay map with the account number ready to copy.

<img src="images/desktop-payments.webp" alt="Payment history across offices, GCash, Bayad Center, and ECPay." width="100%"/>
<sub>Payment history across offices, GCash, Bayad Center, and ECPay.</sub>
<img src="images/desktop-pay.webp" alt="Where to pay: the account number to copy, channels, and a map of offices and mall counters." width="100%"/>
<sub>Where to pay: the account number to copy, channels, and a map of offices and mall counters.</sub>

### Outage complaints came by phone and walk-in, with no record anyone could follow.

Members pin the spot on a map. Each report becomes a numbered job order, routed to the office that serves that town, and the member sees its live status.

<img src="images/desktop-outage.webp" alt="Report an outage or hazard by pinning the spot on a map, then follow its status." width="100%"/>
<sub>Report an outage or hazard by pinning the spot on a map, then follow its status.</sub>
<img src="images/desktop-admin-outage-map.webp" alt="Staff outage map: every open report pinned, with status and next steps." width="100%"/>
<sub>Staff outage map: every open report pinned, with status and next steps.</sub>

### Nobody could answer “where is the lineman on this one?”, and some tickets sat unnoticed.

A dispatch timeline stamps every step with who and when. Unacknowledged tickets escalate to the supervisor, then the manager, and staff get push alerts. Linemen read the same job order on their phones.

<img src="images/desktop-staff-queue.webp" alt="Open job orders across offices, grouped by status, with one-tap actions." width="100%"/>
<sub>Open job orders across offices, grouped by status, with one-tap actions.</sub>
<img src="images/desktop-staff-job-order.webp" alt="A job order's timeline: who acknowledged, who was dispatched, and every crew note." width="100%"/>
<sub>A job order's timeline: who acknowledged, who was dispatched, and every crew note.</sub>
<img src="images/mobile-staff-job-order.webp" alt="The same job order on the lineman's phone." width="100%"/>
<sub>The same job order on the lineman's phone.</sub>

### Management had no view across area offices.

One board shows open tickets per office, the oldest unacknowledged one, escalations, and each lineman's workload.

<img src="images/desktop-admin-ticket-board.webp" alt="All offices at a glance: new, acknowledged, dispatched, oldest new, escalations, and lineman workload." width="100%"/>
<sub>All offices at a glance: new, acknowledged, dispatched, oldest new, escalations, and lineman workload.</sub>

### Most members are on phones over slow mobile data.

The phone layout is designed on its own, not shrunk from desktop, and a device cache paints the last known data before the network answers.

<img src="images/desktop-welcome.webp" alt="Public landing page where members register and sign in." width="100%"/>
<sub>Public landing page where members register and sign in.</sub>
<img src="images/mobile-welcome.webp" alt="Landing page on a phone." width="100%"/>
<sub>Landing page on a phone.</sub>

## Approach

- An on-prem sync agent is the only part that reads the billing database. It mirrors only what the portal needs, pushes outbound, and opens no inbound ports.
- Read-through, not a bulk copy: a member's data is mirrored only after they claim the account, and refreshed on demand.
- Data isolation enforced in SQL: every account-scoped query is guarded against the caller's linked accounts, with row-level security set to default-deny.
- Self-hosted behind a tunnel, with automatic deploys, health checks, and rollback, and a cloud host as a warm fallback.

## What was built

- Bills, line-by-line bill breakdown, payments, and 13-month usage
- Outage and hazard reports with a map pin, tracked as job orders to restoration
- Staff side: ticket board across area offices, dispatch to linemen, account lookup
- Where to pay, rates, appliance calculator, and severe-weather warnings
- On-prem sync agent mirroring billing data to the cloud

## Capabilities

**Member self-service**
- Bill and balance for every account a member owns, with due date and unpaid-month warning
- Line-by-line bill breakdown with a plain-language reason it changed from last month
- 13-month usage, comparisons to last month and last year, and a next-bill estimate
- Payment verification across channels
- Where to pay, with a map of offices and mall counters
- Appliance cost estimator at the member's own effective rate
- Severe-weather warnings for the franchise area

**Outage and job orders**
- Report an outage, hazard, meter, or billing problem with a map pin
- Every report becomes a numbered job order routed to the office that serves that town
- Dispatch timeline: acknowledged, assigned, en route, on site, restored
- Unacknowledged tickets escalate to the supervisor, then the manager, on a schedule
- Members see live status in the app

**Staff tools**
- Ticket board across all area offices, with transfers
- Office tenders file for phone and walk-in callers
- Web Push alerts for new tickets in the staffer's area
- All-offices management view: open tickets, oldest unacknowledged, escalations, lineman workload
- Account lookup and user management

**Security and privacy**
- Account claim with an email code plus a detail from a real bill, with rate limits
- Unauthorized and missing records return the same 404, so accounts cannot be probed
- Role and office scoping re-checked on every admin page and server action
- A lineman sees a caller's number only on a job assigned to them
- Strict CSP, HSTS, database TLS pinned to its CA, bot checks on auth, WAF and rate limits

## Architecture

- Billing system (SQL Server, office LAN) → on-prem Node sync agent → Supabase Postgres mirror with default-deny row-level security → Next.js 16 portal.
- The portal runs on the cooperative's own server behind a Cloudflare Tunnel; a poller deploys the main branch within minutes, health-checks it, and rolls back on failure.
- If a sync fails, the portal keeps serving the last data with an "as of" time instead of going down.
- Charts are hand-built SVG and maps are hand-rolled map tiles, with no chart or map library.

## Engineering

- A test asserts the account guard on every exported data function.
- Moved to a transaction pooler after per-instance session pools ran out of connections under load.
- Replaced a 170 ms auth round-trip per request with local JWT verification.
- A stale-while-revalidate device cache paints the last known data before the first network byte on slow mobile data.
- 370+ test cases across 29 suites in Vitest.

## Where AI does the work

Automation keeps the portal current and the queue moving: the sync agent refreshes billing data on demand, reports route themselves to the right office, and unacknowledged tickets escalate on their own.

## Outcome

- Live in production for members across 17 towns and cities.
- Bill, payment, and usage questions that used to need a call or an office visit are answered in the app.
- Every outage report is a numbered job order with a timeline, so members, dispatchers, and management follow the same record.
- The portal reads from a mirror, so member traffic never reaches the billing database.

---

Built by **[Les John Paul Oliver](https://lesjohnpauloliver.dev)**, AI Engineer in the Philippines: agentic AI, RAG, LLM evals, MCP, and business automation. [Case studies](https://lesjohnpauloliver.dev/#projects) · [Hire me](https://lesjohnpauloliver.dev/ai-engineer-philippines)
