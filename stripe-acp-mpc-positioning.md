# Silent Compute x Stripe ACP: MPC Integration Positioning

**Author:** Ambika Pande, Product Lead — Silent Compute
**Last updated:** March 2026
**Purpose:** How Silent Shard + Silent Compute plug into Stripe's Agentic Commerce Protocol (ACP)

---

## What Stripe's ACP Already Solves

Stripe's Agentic Commerce Protocol (co-built with OpenAI, launched May 2025) handles the basic agentic payment flow:

1. Agent discovers product via merchant API
2. Agent shows checkout UI to user
3. User approves → Agent gets Shared Payment Token (SPT)
4. Agent sends SPT to merchant
5. Merchant creates PaymentIntent with SPT
6. Payment processed

### ACP's Key Security Primitives
- **Shared Payment Token (SPT)** — scoped to one merchant + one amount + expiry. Better than raw card numbers.
- **Restricted API keys** — agent gets limited Stripe API access (`rk_*` keys)
- **Virtual cards** — via Stripe Issuing, with spending limits
- **Merchant remains merchant of record** — full control over acceptance

### ACP's Key Components
- **Agent Toolkit** — SDK for LLMs to interact with Stripe APIs. Supports Vercel AI SDK, LangChain, CrewAI.
- **Shared Payment Token (SPT)** — scoped credential for agent-initiated payments.
- **Agentic Commerce Suite (ACS)** — low-code tools for catalog exposure, risk checks, token handling (Dec 2025).
- **REST Endpoints** — `CreateCheckoutRequest`, `UpdateCheckoutRequest`, `CompleteCheckoutRequest`.

---

## Where ACP Falls Short — The MPC Opportunity

| ACP Limitation | What's Missing | MPC Solution |
|---|---|---|
| SPT is still a credential | Agent holds the token. If compromised, token is leaked. Scoping helps but doesn't eliminate risk. | Threshold signing — agent holds a shard, not a token. Can't be used alone. |
| Agent sees everything | Agent sees product, price, user's payment method, merchant identity. Full transaction visibility. | Encrypted computation — agent can execute without seeing all parameters. |
| No negotiation layer | ACP is buy-at-listed-price only. No mechanism for agent-to-agent price discovery or sealed bidding. | MPC negotiation — agents compute deal price without revealing constraints. |
| No private eligibility checks | If checkout requires credit check, KYC, or age verification, the agent or merchant collects raw data. | Boolean MPC — eligible/verified yes or no, zero data shared. |
| Permission model is API-key based | Restricted keys are revocable but not cryptographically enforced. Agent with key can do anything the key permits. | Threshold auth — requires 2-of-3 cryptographic agreement, not just a key. |
| No cross-merchant intelligence | Each merchant is isolated. No collaborative fraud detection, no cross-merchant basket optimization. | Multi-party MPC — merchants collaborate without sharing data. |

---

## The Critical Upgrade: SPT → Threshold Signature

| Shared Payment Token (SPT) | Threshold Signature (Silent Shard) |
|---|---|
| Token = single credential | Shard = one piece of a key |
| Scoped (merchant + amount) | Scoped (rules + limits + whitelist) |
| Agent holds it | Agent holds 1/3, Stripe holds 1/3 |
| Expires after use/time | Revocable instantly by user |
| If leaked: usable within scope | If leaked: useless alone |
| Per-transaction approval | Rules-based auto-approval |
| User clicks "approve" each time | User sets rules once |

**SPT is a scoped credential.** It reduces blast radius but doesn't eliminate it. The agent still holds something usable.

**Threshold signature is a non-credential.** The agent holds something that is literally worthless on its own. Security comes from the math, not from scoping rules.

For a world where agents handle millions of transactions autonomously, this distinction matters. One compromised agent with SPTs can drain accounts within scope limits. One compromised agent with a shard can do nothing.

---

## The Upgraded Flow: ACP + Silent Shard + Silent Compute

### Full User Flow: "Buy best AC under 50K on EMI"

**Step 1: Discovery (standard ACP — no change)**

Agent calls merchant ACP endpoints. Gets product catalog, prices, availability. No MPC needed — public data.

**Step 2: Negotiation (ACP can't do this — MPC adds it)**

Agent asks Flipkart's pricing agent: "Is there a better price?"

- User agent input (encrypted): max budget ₹50K
- Flipkart agent input (encrypted): floor price ₹33K
- MPC output: "Deal possible at ₹34,999. Available in 3 days."
- Neither side revealed their limit.

**Step 3: EMI Eligibility (ACP has no private check — MPC adds it)**

ACP today: Flipkart collects bank statements or redirects to lending partner.

With MPC (3-party: user's AA data × Flipkart's underwriting rules × agent):
- Output: ELIGIBLE, 6 EMIs × ₹5,950
- Not revealed: salary, bank balance, underwriting model

**Step 4: Payment Authorization (ACP uses SPT — MPC upgrades to threshold sig)**

ACP today: Agent gets SPT → sends to merchant → PaymentIntent created.

With Silent Shard:
- Agent shard + Stripe shard = 2-of-3 signature
- Stripe validates: amount ≤ limit, merchant in whitelist, daily cap ok
- Agent NEVER holds a usable credential
- If compromised: shard alone is useless, no replay possible, user revokes instantly

**Step 5: Confirmation (standard ACP — no change)**

Agent sends CompleteCheckoutRequest. Order confirmed. User notified.

---

## Where SL Plugs Into ACP's Architecture

ACP defines three REST endpoints. SL adds three parallel layers:

| ACP Layer (Stripe) | SL Layer (Silent Labs) |
|---|---|
| `CreateCheckoutRequest` — Merchant generates cart | `NegotiatePrice` — MPC sealed-bid between agents → optimized price |
| `UpdateCheckoutRequest` — User makes selections | `VerifyEligibility` — MPC: user's AA data × merchant rules → eligible/not |
| `CompleteCheckoutRequest` — Payment authorized | `AuthorizePayment` — Threshold signature → signed transaction (not SPT) |

SL doesn't replace ACP. It runs alongside it — adding capabilities at each step that ACP's token-based model can't provide.

---

## API Design: Developer Experience

```python
import stripe
from stripe_agent_toolkit import AgentToolkit
from silent_labs import SilentShard, SilentCompute

# ── Setup (one-time) ──

# Standard Stripe agent setup
toolkit = AgentToolkit(
    secret_key="rk_live_...",
    configuration={"actions": {"payment_links": {"create": True}}}
)

# SL: Provision agent shard (replaces SPT model)
agent_auth = SilentShard.provision_agent(
    user_id="cus_abc123",
    agent_id="chatgpt_shopping",
    stripe_key="rk_live_...",
    rules={
        "max_per_txn": 50000_00,
        "daily_limit": 100000_00,
        "merchant_whitelist": ["flipkart", "amazon", "croma"],
        "requires_user_approval_above": 25000_00
    }
)

# ── Checkout Flow ──

# Step 1: Standard ACP — Create checkout
checkout = stripe.acp.CreateCheckoutRequest(
    merchant="flipkart",
    items=[{"name": "Voltas 1.5T AC", "amount": 34999_00}],
    payment_type="emi"
)

# Step 2: SL — Check EMI eligibility (MPC, no data shared)
eligibility = SilentCompute.verify(
    protocol="emi_eligibility",
    user_data_source="finvu_aa",
    merchant_rules=checkout.emi_rules,
    # Returns only: {eligible: true, emi_amount: 5950, tenure: 6}
)

# Step 3: SL — Authorize payment (threshold signature, not SPT)
payment = SilentShard.authorize(
    agent_auth=agent_auth,
    amount=5950_00,
    merchant="flipkart",
    # Agent shard + Stripe shard = 2 of 3 → signed
)

# Step 4: Standard ACP — Complete checkout
result = stripe.acp.CompleteCheckoutRequest(
    checkout_id=checkout.id,
    payment_authorization=payment.signature,
    emi=eligibility.emi_plan
)
```

For the developer, it's 3 extra lines. The MPC complexity is abstracted behind `SilentCompute.verify()` and `SilentShard.authorize()`.

---

## ACP vs. ACP + MPC: Full Comparison

| Step | ACP Today | ACP + Silent Labs MPC |
|---|---|---|
| Discovery | Agent calls merchant APIs | Same (no change needed) |
| Negotiation | Not possible. Buy at listed price only. | MPC enables sealed-bid negotiation between agents |
| Eligibility | Agent/merchant collects raw financial data | MPC: boolean output only. No data shared. |
| Payment auth | SPT — scoped token, but agent holds it | Threshold signature — agent holds shard, can't use alone |
| If agent compromised | SPT can be used within scope until expiry | Shard is useless alone. No replay possible. |
| Identity verification | Stripe Identity collects documents | MPC: boolean attribute check. No PII stored. |
| Cross-merchant fraud | Each merchant isolated (Radar is Stripe-side) | MPC: merchants share fraud signals without sharing data |
| User action required | User approves SPT generation (every purchase) | User sets rules once. Agent operates within them. |

---

## Stripe Product Surfaces Where MPC Creates Value

| SL Product | Stripe Product | What It Enables | New Revenue for Stripe |
|---|---|---|---|
| Silent Shard (threshold signing) | Payment Intents / ACP | Agent payments without credentials | New product: "Stripe Agent Auth" — per-signing fee |
| Silent Compute (negotiation) | Connect | Multi-party price discovery for B2B marketplaces | Higher Connect volume; premium tier |
| Silent Compute (verification) | Identity | ZK attribute verification — verify without collecting PII | Premium Identity tier; GDPR/DPDP compliance moat |
| Silent Compute (metering) | Billing | Private usage-based billing — customer proves tier without revealing exact usage | Enterprise billing upsell |
| Silent Compute (fraud intel) | Radar | Cross-merchant fraud intelligence without sharing transaction data | Premium Radar tier |

---

## Technical Integration Point

Where SL code runs in Stripe's architecture:

```
┌─────────────────────────────────────────────┐
│              STRIPE INFRASTRUCTURE           │
│                                              │
│  ┌──────────┐    ┌──────────┐               │
│  │ Stripe   │    │ Stripe   │               │
│  │ API      │───►│ Core     │               │
│  │ Gateway  │    │ Payment  │               │
│  └──────────┘    │ Engine   │               │
│                  └────┬─────┘               │
│                       │                      │
│              ┌────────▼────────┐             │
│              │  SL MPC Module  │ ← NEW       │
│              │                 │             │
│              │ • Silent Shard  │             │
│              │   (signing)     │             │
│              │ • Silent Compute│             │
│              │   (verification)│             │
│              │ • Audit service │             │
│              └────────┬────────┘             │
│                       │                      │
│              ┌────────▼────────┐             │
│              │  Card Networks  │             │
│              │  / Bank APIs    │             │
│              └─────────────────┘             │
└─────────────────────────────────────────────┘
```

SL's module sits between Stripe's payment engine and the card networks — intercepting payment requests that need threshold signing or MPC verification, processing them cryptographically, and passing the signed/verified result downstream.

---

## The Pitch to Stripe

> **"ACP is the checkout protocol. MPC is the trust protocol."**
>
> ACP solves the flow: how an agent discovers, selects, and completes a purchase.
>
> It doesn't solve the trust: how do you know the agent isn't compromised? How do you verify eligibility without collecting data? How do you negotiate without revealing intent?
>
> SPTs are a smart v1 — scoped, expirable, single-use. But they're still tokens. In a world of millions of autonomous agents making millions of transactions, token security won't scale. Cryptographic security will.
>
> **Silent Shard replaces SPTs with threshold signatures.** Same developer experience, fundamentally stronger security model. The agent never holds anything usable alone.
>
> **Silent Compute adds capabilities ACP can't have** — private eligibility checks, agent-to-agent negotiation, cross-merchant fraud intelligence. These aren't incremental features; they're new product surfaces Stripe can charge for.
>
> We don't compete with ACP. We complete it.

---

## Integration Model

SL ships an SDK that Stripe embeds. Stripe brands it as their own feature. SL gets per-computation revenue share.

| Pricing model | Metric | Estimated range |
|---|---|---|
| Threshold signing | Per signature | $0.001 - $0.01 |
| MPC verification (boolean) | Per verification | $0.01 - $0.05 |
| MPC negotiation (multi-round) | Per negotiation | $0.05 - $0.20 |
| MPC fraud intelligence | Per query | $0.01 - $0.10 |

At Stripe's scale ($1.4T processed in 2024), even 1% adoption of agent payments with MPC = massive volume.

---

## What SL Builds for the Stripe Integration

| Component | Effort | Priority |
|---|---|---|
| Stripe-compatible signing API (drop-in for PaymentIntent) | 6-8 weeks | P0 |
| Agent shard provisioning via Stripe Dashboard | 4 weeks | P0 |
| Spending policy engine (limits, whitelists, time rules) | 4-5 weeks | P0 |
| Stripe Connect MPC adapter (multi-party negotiation) | 8-10 weeks | P1 |
| Stripe Identity ZK verification module | 6 weeks | P1 |
| Stripe Radar collaborative fraud protocol | 8 weeks | P2 |
| Stripe Billing private metering protocol | 6 weeks | P2 |

---

## Open Questions

1. **Stripe's appetite:** Is Stripe building MPC internally, or open to partnership? Their acquisition history (TaxJar, Recko) suggests they buy capabilities.
2. **SPT coexistence:** Can threshold signatures work alongside SPTs as a premium tier, or does it need to replace SPTs entirely?
3. **Latency budget:** ACP checkout needs to be <2 seconds end-to-end. Can threshold signing + MPC verification fit within that? Silent Shard benchmarks: <10ms for signing, but MPC verification may be 100-200ms.
4. **Agent Toolkit integration:** Can SilentShard.authorize() be added as a native action in Stripe's Agent Toolkit alongside existing Payments/Billing/Issuing actions?
5. **Regulatory position:** Are there payment regulations (PCI-DSS, RBI PA guidelines) that explicitly require or prohibit MPC-based authorization? Need legal review.

---

*This is a living document. Update as Stripe evolves ACP and as SL's protocol capabilities mature.*
