# Silent Compute — Demo Platform

**Owner:** Ambika Pande, Product Lead
**Last updated:** March 2026
**Purpose:** Reference doc for the Silent Compute demo platform — what it is, how it's hosted, what to build next

---

## What the Demo Proves

One thing: *"We computed a result across two parties' data, and neither party saw the other's raw data."*

Three beats, in order:

| Beat | What you show | What the audience feels |
| --- | --- | --- |
| **1. The problem** | Party A has applicants. Party B has a watchlist. Today, someone has to share raw data. | They know this pain. |
| **2. The magic** | Hit a button. Watch encrypted messages fly. Result: "7 matches." | "It worked without sharing data?" |
| **3. The proof** | Click "View Party B's data" → Access Denied. Show audit trail + encrypted payloads. | "This is real." |

---

## How the Demo Works

**SL hosts the sandbox.** Prospects get a URL. No installs, no setup, no IT approval needed.

- SL runs two MPC nodes and the coordination layer on its own infrastructure
- The sandbox comes pre-loaded with dummy data for all four entities
- Anyone can log in with `admin / admin` and start running computations immediately
- The demo can be shared as a link — for sales calls, async follow-ups, inbound leads

This is intentionally honest: the sandbox node is managed by SL so prospects can evaluate the platform without any infrastructure commitment. Production nodes always run in the customer's own environment.

---

## Demo Use Case

**Watchlist screening (AML)** — easiest to understand, fastest to build, highest urgency.

Lender checks if its applicants appear on a bank's fraud/sanctions watchlist — without the lender seeing the watchlist, and without the bank seeing the applicant list.

---

## Platform: Two Modes

### Demo Mode (default)

- Fully hosted and managed by SL — prospect just opens a URL
- Pre-loaded with dummy data across 4 fictional entities
- Login: `admin / admin`
- No setup required
- Node badge: `NODE-DEMO-001 · Managed by SL · Demo`
- Honest callout shown to users: "This demo node runs on SL's infrastructure. When you go live, your node runs in your own environment."

### Live Mode

- Customer deploys their own SC Node in their infrastructure
- 5-step guided setup wizard inside the platform
- Works for evaluation (sample table, 500+ rows) or full production
- Copy in wizard: *"Starting with an evaluation? Point the connector at a sample table — even 500 rows is enough to prove the protocol works."*
- SL handholding available — same-day support

---

## Dummy Data Design

**4 dummy entities on the network:**

| Entity | Type | Role in demo |
| --- | --- | --- |
| Demo AA | Account Aggregator | Data provider |
| Demo Bureau | Credit Bureau | Watchlist holder |
| Demo Bank | Commercial Bank | Fraud data holder |
| Demo Publisher | Adtech | Audience data holder |

**Dataset specs:**

- ~10,000 records per entity
- ~22% deliberate overlap between party pairs — demo finds matches without either side seeing the other's list
- Realistic but clearly fictional — no real PAN numbers, no real names
- Same schema reused across use cases where possible

---

## Platform Screens

### Login

Two modes:

- **Demo** — "Active" badge, `admin / admin` hint shown on click
- **Live** — "Production" badge, prompts for company credentials

### Demo Onboarding (first login)

Short onboarding shown before the dashboard. Tells the user what SL has set up for them.

- "Your Demo Environment"
- "We've provisioned a demo node on SL's infrastructure, pre-loaded with dummy data across four fictional entities."
- Steps shown: Provisioning → TLS certs → Registering with Silent Network → Loading dummy data → Online
- Callout: "Ready to use your own data? Deploy your own node in ~15 min — start with a sample table or go straight to production."

### Live Onboarding (5-step wizard)

Shown to live mode users after login. Guides them through deploying their own node. Works for evaluation customers as well as production — framed as: *"Node setup · 5 steps · ~15 min · Works for evaluation or production."*

| Step | What happens |
| --- | --- |
| 1. Generate install token | Paste token; pre-fills CLI commands in later steps |
| 2. Install SC Node | One-line install script — pulls node image, configures service |
| 3. Connect database | CLI command — data stays in the customer's environment |
| 4. Join the network | Node registers identity; SL attestation handshake |
| 5. Verify & go live | Health check; node shows Active in dashboard |

- Skip option: "Explore platform first"
- SL handholding CTA on every step: "Book a call → we'll walk you through this live"

### Dashboard: 4 Tabs

**Use Cases**
Cards for each use case. Click through to entity directory filtered by use case.

**Network Directory**
All entities on the SC network — name, type, online status, capabilities. "Request Computation" per entity.

**Request Flow**
Select computation type → configure both parties' inputs → accept / decline flow. Clear statement: "The other party will NOT see your raw data."

**Computation Status**
Animated protocol visualizer — encrypted packets flying between nodes, round progress, latency counter.

**Results & Analysis**
- Summary: records checked, matches found, time, proof hash
- Results table: shows only your own matched records
- "View other party's data" → **Access Denied** (this is the proof)
- Audit trail: every step timestamped, consent records, cryptographic proof hash
- Download CSV, Verify Proof, Download Audit Certificate

---

## What SL Manages vs. What the Customer Manages

| | Demo | Live |
| --- | --- | --- |
| Node location | SL's infrastructure | Customer's own infra (cloud, on-prem, or SL-delegated) |
| Node management | SL | Customer (SL provides tooling + support) |
| Data | SL's dummy data | Customer's own database (start with a sample table if evaluating) |
| Network registration | Pre-done by SL | Customer runs one CLI command |
| Purpose | Show how it works | Real computations on real data — evaluation or production |

---

## Build Checklist

| Priority | Component | Effort | Status |
| --- | --- | --- | --- |
| P0 | SL-hosted sandbox environment (two nodes + coordinator) | Depends on infra | — |
| P0 | Shareable URL — no login required for demo | 1–2 days | — |
| P0 | Interactive prototype (all tabs, all flows) | Done | ✅ |
| P0 | Sandbox onboarding flow | Done | ✅ |
| P0 | Live mode login + node setup wizard | Done | ✅ |
| P0 | Real SC protocol wired to dashboard (Run button → results) | Depends on SDK | — |
| P1 | Protocol visualizer (encrypted message animation) | 3–5 days | ✅ (prototype) |
| P1 | Audit trail tab | 2–3 days | ✅ (prototype) |
| P2 | 4 dummy entities with pre-loaded datasets | 3–5 days | — |
| P3 | Per-use-case demo flows (AML, credit, enrichment) | 1–2 weeks | — |

---

## Demo Don'ts

- Don't show raw data from both sides — it undermines the privacy claim
- Don't use technical jargon in the demo narrative — lead with the problem
- Don't skip the "Access Denied" moment — it's the proof
- Don't demo multi-party before two-party is solid
- Don't require any install from the prospect — the sandbox URL should just work

---

## Open Questions

1. **Real MPC backend:** How long to wire the prototype to a real SC protocol? What's the current SDK state?
2. **Shareable link:** Do we want a public demo URL (no login) for async sharing, or always require `admin / admin`?
3. **Multi-entity demo:** When do we demo 3-party computation? After two-party is proven in the field.
4. **Pricing on the dashboard:** Show it now or keep off until GTM?

---

*Prototype file: `silence-labs/silent-compute-prototype.html` — open in browser, no build step.*
