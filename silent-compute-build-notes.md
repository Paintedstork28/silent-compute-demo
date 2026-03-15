# Silent Compute: Build Strategy Notes

**Author:** Ambika Pande, Product Lead — Silent Compute
**Last updated:** March 2026
**Purpose:** Reference notes for Silent Compute product build strategy

---

## 1. What is Silent Compute

### ELI5
Two banks want to check if the same person owes money to both — but neither wants to show the other its customer list. Silent Compute lets them get the answer ("yes, 47 people overlap") without either side ever seeing the other's data. The data never moves. It stays encrypted, at its source, the entire time.

### Technical Definition
Cryptographic computing platform built on Multi-Party Computation (MPC) protocols. Enables multiple entities to jointly compute functions over their private inputs — without revealing those inputs to each other, to Silence Labs, or to any intermediary.

### Core Guarantees
- **No data movement** — raw data never leaves its source environment
- **End-to-end encryption** — data remains encrypted even during computation
- **Purpose-bound execution** — only pre-agreed computations run
- **Auditability** — every computation is tracked, verifiable, and compliant by design

### How It Differs from Alternatives

| Approach | Data moves? | Trusted third party? | Regulatory risk |
|---|---|---|---|
| Data sharing via API | Yes | Yes (recipient) | High |
| Clean rooms (Snowflake, AWS) | Yes (copied) | Yes (cloud provider) | Medium |
| Federated learning | Model moves, not data | Partial | Medium |
| **Silent Compute (MPC)** | **No** | **No** | **Low** |

---

## 2. Architecture: Three Layers

### Layer 1: MPC Node (Deployed at Each Company)

Server-side package (Docker container or native binary) installed on each participating company's own infrastructure.

**Deployment options:**

| Option | How | Best for |
|---|---|---|
| Company's own cloud (AWS/GCP/Azure) | Docker container in their VPC | Most companies |
| On-premise server | Binary install on bare metal / VM | Banks, regulated entities |
| SL-managed cloud (delegated) | SL spins up an isolated node for the company | Small companies, NBFCs without infra |

**What it does:**
- Connects to company's internal database/data source (read-only)
- Encrypts data locally using MPC secret-sharing
- Executes MPC protocol rounds with other nodes (peer-to-peer)
- Reports status and metadata to the SL coordination layer
- Never sends raw data anywhere

**What gets installed:**
```
sc-node/
├── sc-engine          # MPC protocol runtime
├── data-connector     # Plugs into company's DB (Postgres, MySQL, API, CSV)
├── config.yaml        # Node ID, network registration token, data source config
├── tls-certs/         # Mutual TLS certs for node-to-node communication
└── sc-agent           # Background service that talks to SL coordination layer
```

**Installation flow:**
1. Company signs up on dashboard → gets a REGISTRATION TOKEN
2. Company runs: `sc-node install --token=<TOKEN>`
3. sc-agent starts → registers with SL network → node appears on dashboard
4. Company configures data source: `sc-node connect --db=postgres://...`
5. Node runs health check → reports "online" to dashboard

### Layer 2: SL Coordination Layer (The Network)

Hosted by SL. Does NOT touch data. Think of it like a phone exchange: it connects calls, it doesn't listen to them.

| Function | Detail |
|---|---|
| **Node registry** | Tracks which companies have installed nodes, their status (online/offline), version |
| **Network directory** | Lets companies discover other companies on the network |
| **Computation catalog** | Lists available MPC computations (secure match, secure aggregate, joint scoring, etc.) |
| **Handshake broker** | When Party A wants to compute with Party B, the coordination layer brokers the initial connection. After handshake, nodes talk directly peer-to-peer. |
| **Consent ledger** | Records consent: "Party A requested Secure Match with Party B. Party B accepted at 10:42 AM." |
| **Result routing** | Receives computation result metadata (not raw results) for dashboard display |
| **Audit log** | Immutable record of all network activity |

**What it does NOT do:**
- Never receives raw data
- Never receives unencrypted computation inputs
- Never participates in MPC protocol execution
- Cannot reconstruct any party's data from what it sees

### Layer 3: Dashboard (Web App)

Web application hosted by SL (or self-hosted for enterprise). See Section 5 for full screen specs.

### How Dashboard Talks to Node

```
Dashboard (browser) ←→ SL Coordination Layer (cloud) ←→ Company's MPC Node (on-prem)
      HTTPS                                      gRPC/WSS
```

The `sc-agent` daemon on the company's server:

| Function | Direction | Protocol |
|---|---|---|
| Sends heartbeat ("I'm alive") | Node → SL Cloud | gRPC / WebSocket |
| Reports metadata (record count, capabilities) | Node → SL Cloud | gRPC |
| Receives computation requests | SL Cloud → Node | gRPC (push) |
| Reports computation status/results metadata | Node → SL Cloud | gRPC |
| Talks to other nodes during MPC | Node ↔ Node | Direct TLS (peer-to-peer) |

**Critical:** sc-agent initiates the outbound connection to SL cloud. Company doesn't need to open inbound ports. Makes firewall/IT approval much easier.

---

## 3. Data Flow (Step by Step)

1. **Integration** — Each entity deploys SC SDK within their own infra
2. **Schema agreement** — Parties agree on computation type (match, score, aggregate) and input schema
3. **Encryption at source** — SDK encrypts data locally using MPC secret-sharing
4. **Protocol execution** — Encrypted shares exchanged via secure channels; joint computation runs across parties
5. **Result delivery** — Only agreed-upon output (overlap count, risk score, aggregate stat) is revealed

### Deployment Models
- **Two-party:** Most common. Bank ↔ Bureau, Lender ↔ AA, Platform ↔ Regulator
- **Multi-party:** 3+ entities collaborating (e.g., consortium fraud detection)
- **Delegated:** One party delegates compute to a trusted node running SC SDK

### What SL Ships

| Component | Description |
|---|---|
| **SC SDK / Libraries** | Embeddable libraries (Python, Java, Go) for on-prem integration |
| **CCVM (Cryptographic Computing VM)** | Managed runtime for executing MPC protocols |
| **Silent Network** | Infrastructure layer for deploying and managing MPC networks |
| **Dashboard** | Admin panel for computation management, audit logs, consent tracking |

---

## 4. Entity Isolation

| Layer | Mechanism |
|---|---|
| **Data isolation** | Raw data never leaves entity's environment; only encrypted shares transit |
| **Compute isolation** | Each entity runs its own SC SDK instance; no shared runtime |
| **Key isolation** | Each entity holds its own key material; threshold schemes prevent single-party reconstruction |
| **Network isolation** | Point-to-point encrypted channels between parties; no shared bus |
| **Logical isolation** | Each computation is scoped to specific entities and specific data fields |

### What SL Servers See vs. Don't See

| SL servers see | SL servers DON'T see |
|---|---|
| Which entities are registered | Any party's raw data |
| Which computations were configured | Input values to any computation |
| Computation metadata (time, status, latency) | The actual encrypted shares (party-to-party) |
| Results (if dashboard-routed) | Anything beyond the agreed output |

---

## 5. Dashboard: Screen-by-Screen Spec

### Screen 0: Onboarding / Node Setup

First-time company sign-up flow:
1. Create organization
2. Get registration token
3. Install MPC node (Docker or binary)
4. Configure data source (via CLI on company's server)
5. Go live on the network

Data source config done via CLI (not dashboard, for security):
```bash
$ sc-node connect \
    --type=postgres \
    --host=internal-db.company.com \
    --database=customers \
    --table=applicants \
    --fields=name,pan,phone,loan_amount
```

Dashboard then shows: data source connected, record count, fields exposed for computation.

### Screen 1: Network Directory

Shows all companies on the SC network:
- Name, type (Bank/Bureau/AA/NBFC), online status
- Capabilities advertised (Watchlist Match, KYC Verify, Credit Score, etc.)
- "Request Computation" button per entity
- Filters by entity type, search

Companies choose what capabilities to advertise. They don't see each other's data — only name, type, capabilities, status.

### Screen 2: Request Computation

Initiated by the requesting party:
- Select computation type (Secure Match, Secure Aggregate, Joint Credit Score)
- Shows what each party provides and what the output will be
- Configure input: select data source, records (all or filtered), fields to use
- Consent and terms: purpose declaration, output scope agreement
- Submit request to the other party

### Screen 3: Incoming Request (Other Party's View)

Receiving party sees:
- Who is requesting, computation type, what input they need to provide
- Configure their data source and fields
- Clear statement: "Requestor will NOT see your raw data"
- Accept / Decline / Ask Questions

### Screen 4: Computation Running (Both Parties)

Live view:
- Progress bar with percentage
- Real-time encrypted protocol messages (hex blobs) — visual proof of encryption
- MPC round progress (Round 2/4, etc.)
- Records processed counter, elapsed time, estimated remaining
- Protocol type displayed (Garbled Circuit, OT, etc.)

### Screen 5: Results

- Summary: total records checked, matches found, computation time, proof hash
- Matched records table showing ONLY the requesting party's own data (not the other party's)
- "View other party's data" → **Access Denied** (this is the proof)
- Download CSV, view audit trail
- Option to "Request Detail on Matched Records" (triggers new scoped computation)

### Screen 6: Audit Trail

Per-computation immutable log:
- Every step timestamped: request, acceptance, protocol initiation, completion
- Consent records for both parties
- Cryptographic proof hash (independently verifiable)
- "Verify Proof" and "Download Audit Certificate" actions

### Screen 7: My Node (Node Management)

- Node status, ID, version, uptime
- Connected data sources with record counts, last sync time
- Capabilities advertised to the network (toggle on/off)
- Computation history (date, counterparty, type, result)

---

## 6. Use Cases

### UC1: Data Enrichment for Lenders

**Problem:** Lenders need richer borrower profiles but can't access data held by other lenders, merchants, or telecom providers.

**How SC solves it:** Multiple data holders run joint MPC computations to produce enriched attributes (DTI ratio, spending scores) without sharing raw data.

**Target customers:**
- Digital lenders (KreditBee, MoneyTap, Navi)
- Payment networks (NPCI, Visa)
- Account Aggregators (Finvu, OneMoney, Sahamati ecosystem)
- Merchant platforms (Razorpay, PhonePe)

**GTM:** Partner with 1-2 AAs as distribution channel. Pilot with mid-size digital lender. Position as "AA 2.0."

**Build required:**

| Component | Effort | Priority |
|---|---|---|
| Standardized enrichment schema (income, obligations, spending) | 2-3 weeks | P0 |
| Two-party SC protocol for lender ↔ data provider | 4-6 weeks | P0 |
| AA ecosystem integration (FIP/FIU adapters) | 4 weeks | P1 |
| Pre-built enrichment templates (DTI, cashflow score) | 3 weeks | P1 |
| Dashboard: computation logs, consent management | 3-4 weeks | P2 |

### UC2: AML / Fraud Detection

**Problem:** Fraud patterns span multiple institutions. No single bank sees the full picture. Sharing customer data across banks is illegal.

**How SC solves it:** Banks jointly run anomaly detection, graph analysis, watchlist matching on encrypted transaction data.

**Target customers:**
- Tier 1 banks (SBI, HDFC, ICICI)
- RBI / FIU-IND
- Payment aggregators (Paytm, PhonePe)
- Regtech providers (partnership channel)

**GTM:** Lead with regulatory angle ("RBI wants inter-bank fraud detection; here's how without violating privacy laws"). Partner with IBA or FIU-IND for consortium pilot. Start with secure watchlist matching (simplest, highest urgency).

**Build required:**

| Component | Effort | Priority |
|---|---|---|
| Secure match protocol (watchlist/sanctions screening) | 3-4 weeks | P0 |
| Multi-party protocol support (3+ banks) | 6-8 weeks | P0 |
| Encrypted graph analysis (transaction network patterns) | 8-10 weeks | P1 |
| Alerting framework (privacy-preserving alert generation) | 4 weeks | P1 |
| Compliance audit trail & reporting | 3 weeks | P1 |
| Integration adapters for core banking systems | 6 weeks | P2 |

### UC3: Agentic Payments Privacy

**Problem:** AI agents executing transactions need payment credentials and financial data — massive privacy and security risk.

**How SC solves it:** AI agents interact with payment systems through MPC. Agent initiates payments without seeing plaintext credentials, balances, or transaction details.

**Target customers:**
- AI platforms (OpenAI, Google, startups)
- PSPs (Stripe, Razorpay, Adyen)
- Neobanks (Jupiter, Fi)
- Wallet providers

**GTM:** Position as "privacy rail for agentic commerce." Partner with 1-2 PSPs. First-mover advantage is significant. Frontier use case — invest in R&D now, GTM later.

**Build required:**

| Component | Effort | Priority |
|---|---|---|
| Agent-to-payment MPC protocol (credential-less authorization) | 6-8 weeks | P0 |
| Threshold signature integration (multi-party tx approval) | 4-5 weeks | P0 |
| PSP/gateway integration SDK (Stripe, Razorpay adapters) | 4-6 weeks | P1 |
| Consent & delegation framework (user → agent permissions) | 3-4 weeks | P1 |
| Agent observability (what was computed, audit logs) | 3 weeks | P2 |

### UC4: Regulatory Reporting / Compliance

**Problem:** Regulators need aggregated data from financial institutions. Centralizing raw data creates concentration risk. Current reporting is manual and slow.

**How SC solves it:** Institutions run joint MPC to produce aggregated regulatory metrics (total exposure, NPAs, liquidity ratios) — regulator receives results without any institution revealing raw books.

**Target customers:**
- RBI (systemic risk monitoring)
- SEBI (market surveillance)
- NBFCs (high compliance burden, low infra)
- SROs (MFIN, Sa-Dhan)

**GTM:** Regulator-first (RBI Innovation Hub, IFSCA sandbox). Compliance-cost reducer for NBFCs. Partner with regtech firms.

**Build required:**

| Component | Effort | Priority |
|---|---|---|
| Secure aggregation protocol (sum, average, distribution) | 3-4 weeks | P0 |
| Pre-built regulatory templates (RBI returns, CRAR, NPA ratios) | 4-5 weeks | P0 |
| Multi-party support (regulator + N institutions) | 5-6 weeks | P1 |
| Scheduled/automated computation runs | 3 weeks | P1 |
| Audit trail with cryptographic proof of correctness | 4 weeks | P1 |
| NBFC-friendly deployment (lightweight, cloud-hosted option) | 3 weeks | P2 |

### UC5: Credit Underwriting Without Data Sharing

**Problem:** Accurate credit decisions need data from multiple sources. Every data hop is a privacy leak.

**How SC solves it:** Lender, bureau, and alt-data providers jointly compute a credit score using MPC — lender gets the decision, never sees raw data from other providers.

**Target customers:**
- Account Aggregators (Finvu, OneMoney, Perfios)
- Credit bureaus (CIBIL, Experian, CRIF — as participants)
- Digital lenders (thin-file / new-to-credit segments)
- Alt-data providers (telecom, utility)

**GTM:** Position as "privacy layer for AA." Target thin-file/NTC use case. Pilot with AA + lender pair.

**Build required:**

| Component | Effort | Priority |
|---|---|---|
| Joint scoring MPC protocol (multi-source → single score) | 6-8 weeks | P0 |
| AA integration layer (FIP/FIU protocol adapters) | 4-5 weeks | P0 |
| Bureau integration adapter (CIBIL/Experian encrypted query) | 5-6 weeks | P1 |
| Alternative data connectors (telecom, UPI, utility) | 4-5 weeks | P1 |
| Consent management (borrower approval flow) | 3 weeks | P1 |
| Model validation toolkit (prove accuracy ≈ raw-data model) | 4 weeks | P2 |

---

## 7. Prioritization Matrix

| Use Case | Market Readiness | Technical Complexity | Revenue Potential | Regulatory Tailwind | **Priority** |
|---|---|---|---|---|---|
| **AML / Fraud Detection** | High | Medium-High | High | Very Strong (RBI push) | **1 — Start here** |
| **Credit Underwriting** | High (AA ready) | Medium | High | Strong (DPDP Act) | **2** |
| **Data Enrichment** | Medium-High | Medium | Medium-High | Medium | **3** |
| **Regulatory Reporting** | Medium | Low-Medium | Medium | Strong | **4** |
| **Agentic Payments** | Low (nascent) | High | Very High (long-term) | Emerging | **5 — Invest early, ship later** |

**Rationale:**
- AML/Fraud #1: Regulatory pressure creating active demand. RBI pushing for inter-bank fraud intelligence. SC is only legal way. Large deal sizes.
- Credit Underwriting #2: AA ecosystem = ready distribution. NTC segment underserved. Strong PMF.
- Agentic Payments last for now: Highest long-term TAM. Invest in protocol R&D + thought leadership. GTM when agent ecosystem matures (6-12 months).

---

## 8. Roadmap

### Phase 1: Foundation (Months 1-3)
- Build core two-party MPC protocol (secure match + secure aggregation)
- Build entity onboarding + key management flow
- Build audit trail service
- **Pilot: Secure watchlist matching with 2 banks**
- Deliver MVP dashboard (entity management + audit logs)

### Phase 2: Expand (Months 4-6)
- Build multi-source joint scoring protocol
- AA ecosystem integration (FIP/FIU adapters)
- Multi-party protocol support (3+ entities)
- **Pilot: Credit underwriting with 1 AA + 1 lender**
- Expand dashboard (consent tracker, computation catalog)

### Phase 3: Scale (Months 7-9)
- Pre-built regulatory templates (RBI returns)
- Data enrichment protocol templates
- Scheduled/automated computation runs
- Billing & metering infrastructure
- **Pilot: NBFC regulatory reporting consortium**

### Phase 4: Frontier (Months 10-12)
- Agent-to-payment MPC protocol
- Threshold signature integration (leverage Silent Shard)
- PSP integration SDK
- **POC with 1 AI platform / PSP**
- Publish "Privacy Rail for Agentic Commerce" whitepaper

---

## 9. Key Metrics (Year 1 Targets)

| Metric | Target |
|---|---|
| Production deployments | 3-5 |
| Entities onboarded | 10-15 |
| Computations executed | 100K+ |
| Avg. computation latency | <500ms (two-party match) |
| Revenue | First 2-3 paid contracts |
| Regulatory engagement | 1 sandbox/pilot with RBI or SEBI |

---

## 10. Demo / Sandbox Setup

### What the Demo Proves
One thing: "We computed a result across two parties' data, and neither party saw the other's raw data."

### Demo Use Case
Watchlist screening (AML) — easiest to understand, fastest to build, highest urgency.

### Sandbox Architecture (Single Machine)

```
Docker network: sc-sandbox
├── container: party-a-node     (simulates Lender's server)
│   ├── sc-engine
│   ├── sc-agent
│   └── mock-data: applicants.csv
├── container: party-b-node     (simulates Bank's server)
│   ├── sc-engine
│   ├── sc-agent
│   └── mock-data: watchlist.csv
├── container: sc-coordinator   (simulates SL cloud)
│   ├── node registry
│   ├── handshake broker
│   └── audit log
└── container: sc-dashboard     (the web UI)
    ├── React/Next.js app
    └── connects to sc-coordinator API
```

Runs with `docker-compose up`, fits on a laptop.

### Demo Narrative (Three Beats)

| Beat | Show | Audience feels |
|---|---|---|
| **1. The problem** | Party A has applicants. Party B has a watchlist. Today, someone has to share raw data. | They know this pain. |
| **2. The magic** | Hit button. Watch encrypted messages fly. Result: "7 matches." | "It worked without sharing data?" |
| **3. The proof** | Click "View Party B's data" → Access Denied. Show audit trail + encrypted payloads. | "This is real." |

### Demo Components to Build

| Priority | Component | Effort |
|---|---|---|
| P0 | Two-node SC protocol (secure match) on one machine | Depends on SDK |
| P0 | Dashboard with Run button + results + Access Denied proof | 1-2 weeks |
| P1 | Protocol visualizer (encrypted message animation) | 3-5 days |
| P1 | Audit trail tab | 2-3 days |
| P2 | Docker Compose packaging | 1-2 days |
| P3 | Cloud-hosted sandbox with shareable URL | 1 week |

### Mock Data Design

**Party A (Lender — applicants):** 100 records with name, PAN, phone, loan amount
**Party B (Bank — watchlist):** 50 records with name, PAN, flag reason

Some records deliberately overlap. Demo finds matches without either party seeing the other's full list.

### Demo Don'ts
- Don't show raw data from both sides (undermines the privacy claim)
- Don't use abstract/technical language
- Don't skip the "Access Denied" moment
- Don't demo multi-party before two-party works
- Don't over-engineer mock data (50-100 records per side is enough)

---

## 11. Open Questions

1. **Build vs. partner for bureau integration?** CIBIL/Experian adapters are high-effort.
2. **Pricing model:** Per-computation, per-entity subscription, or hybrid?
3. **Multi-party protocol readiness:** How far is current MPC stack from 5+ party production latency?
4. **Dashboard investment:** Build in-house or use existing observability platform (Grafana etc.) with custom plugins?
5. **Agentic payments timing:** When to move from R&D to active GTM? What's the trigger signal?

---

## 12. Silence Labs Existing Stack (Reference)

### Products
- **Silent Shard** — MPC-as-a-service for decentralized auth and key management (TSS-based)
- **Silent Shard Snap** — MetaMask integration for distributed self-custodial wallet
- **Silent Compute** — Privacy-preserving data computation platform
- **CCVM** — Cryptographic Computing Virtual Machine (secrets management + data privacy engine)
- **Silent Network** — SDK/API for rapid MPC network setup

### Key Tech
- Threshold Signature Schemes (TSS) using DKLs23, Lindell17 algorithms
- Distributed signatures in <10ms, key gen in <10 seconds (3-party)
- Supports asynchronous operations, weighted thresholds, offline key refresh
- Partnerships: BitGo, MetaMask, EigenLayer, Biconomy, EasyCrypto

---

*This is a living document. Update as assumptions are validated through customer conversations and technical spikes.*
