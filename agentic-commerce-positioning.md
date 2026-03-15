# MPC in Agentic Commerce: Use Case Positioning

**Author:** Ambika Pande, Product Lead — Silent Compute
**Last updated:** March 2026
**Purpose:** Standalone positioning doc for Silent Compute + Silent Shard in agentic commerce

---

## The Core Problem

In agentic commerce, AI agents act on your behalf — browsing, comparing, negotiating, buying. To do this well, they need access to sensitive data they should not have:

- Your bank balance (to know your budget)
- Your spending history (to know your preferences)
- Your credentials (to execute payments)
- Your location, health data, identity (for personalized offers)

Today, the options are: (a) give the agent full access (privacy nightmare), or (b) limit the agent's capabilities (bad experience). MPC creates option (c): the agent can compute on your data without seeing it.

---

## Use Case 1: Agent-to-Agent Price Negotiation

### Scenario
Your shopping agent negotiates with a merchant's pricing agent.

- Your agent knows your max budget is ₹15,000
- Merchant's agent knows their floor price is ₹11,000
- Neither should reveal their number to the other

### What MPC Does
Both agents input their constraints into an MPC protocol. The output is either "deal at ₹X" (if overlap exists) or "no deal" — without either side learning the other's limit.

### Why This Matters
Today, negotiation requires revealing intent. "I'll pay up to 15K" weakens your position. MPC enables negotiation where neither party reveals their hand — like a sealed-bid auction, but for every transaction.

### Target Customers
- AI commerce platforms (Amazon, Flipkart AI agents)
- B2B procurement platforms
- Travel/hospitality booking agents

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Two-party sealed-bid protocol (compare + compute midpoint) | 4-6 weeks | P0 |
| Multi-round negotiation protocol (iterative MPC) | 8-10 weeks | P1 |
| Agent SDK integration (API for AI frameworks) | 4 weeks | P1 |
| Latency optimization (<100ms per round) | 6 weeks | P2 |

---

## Use Case 2: Creditworthiness Verification Without Data Exposure

### Scenario
You want to buy a ₹2L appliance on EMI. The merchant's agent needs to verify you can afford it. Today, this means sharing bank statements or running a credit check that the merchant (or their agent) sees.

### What MPC Does
- Your agent holds your financial data (income, existing EMIs, balance)
- Merchant's agent holds the eligibility criteria (min income, max DTI ratio)
- MPC outputs: "eligible" or "not eligible" — nothing else

The merchant never sees your salary. You never see their underwriting model. Both get what they need.

### Why This Matters
This is the unlock for instant EMI at checkout by AI agents — without the current flow of sharing Aadhaar, bank statements, and PAN with every merchant.

### Target Customers
- BNPL/EMI platforms (Simpl, LazyPay, ZestMoney successors)
- E-commerce checkout providers (Razorpay, Cashfree)
- Digital lenders embedding in merchant flows
- Account Aggregators (Finvu, OneMoney)

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Boolean eligibility MPC protocol (multi-field threshold check) | 4-5 weeks | P0 |
| Standardized eligibility schema (income, DTI, credit score bands) | 2-3 weeks | P0 |
| AA/FIP integration for encrypted financial data input | 4-5 weeks | P1 |
| Merchant checkout SDK (plug into existing payment flows) | 4 weeks | P1 |
| Audit trail for compliance (who checked, what was the output) | 3 weeks | P2 |

---

## Use Case 3: Personalized Pricing Without Revealing Preferences

### Scenario
A travel agent AI is booking your flight. Airlines want to price dynamically based on your willingness to pay, travel history, and loyalty status. But you don't want the airline to know you're a frequent business traveler (they'll charge more).

### What MPC Does
- Your agent holds: travel history, budget, flexibility
- Airline's agent holds: pricing model, seat inventory, dynamic pricing rules
- MPC outputs: the best price for you, computed jointly — without the airline seeing your profile or you seeing their pricing algorithm

### Why This Matters
Today, personalized pricing works against the consumer (you pay more if you look rich). MPC flips this — personalization happens, but without information asymmetry. The airline can optimize revenue, the consumer gets a fair deal, and neither side reveals their strategy.

### Target Customers
- Airlines and travel aggregators (MakeMyTrip, Cleartrip)
- Insurance platforms (dynamic premium pricing)
- SaaS platforms with usage-based pricing
- Any platform with dynamic/personalized pricing models

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Secure function evaluation protocol (price = f(user_data, pricing_model)) | 8-10 weeks | P1 |
| Privacy-preserving ML inference (model stays private, data stays private) | 12-16 weeks | P2 |
| Real-time latency (<200ms for checkout-speed pricing) | 6-8 weeks | P2 |
| A/B testing framework (prove fair pricing vs. traditional) | 4 weeks | P3 |

---

## Use Case 4: Multi-Agent Basket Optimization

### Scenario
Your agent is assembling a grocery order across 3 platforms (BigBasket, Blinkit, Zepto) to minimize total cost. Each platform has different prices, offers, and delivery slots — but they don't want to share pricing data with each other (or with a comparison agent).

### What MPC Does
- Each platform's agent inputs: their prices for the items in your basket
- Your agent inputs: the basket + delivery preferences
- MPC outputs: the optimal split (buy X from BigBasket, Y from Blinkit) — without any platform seeing the others' prices

### Why This Matters
This enables true cross-platform optimization without requiring platforms to share competitive data. Today, comparison shopping is done by scraping — crude and adversarial. MPC makes it cooperative.

### Target Customers
- AI shopping assistants (Google Shopping, Amazon Rufus, independent agents)
- Price comparison platforms
- B2B procurement platforms (multi-vendor RFQ optimization)
- Insurance aggregators (multi-insurer quote optimization)

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Multi-party minimum/optimization protocol (3+ parties) | 8-10 weeks | P1 |
| Platform integration SDK (API for price input) | 4-5 weeks | P1 |
| Basket schema standardization (product IDs, delivery slots) | 3 weeks | P1 |
| Result routing (split order instructions to each platform) | 3 weeks | P2 |
| Platform incentive design (why would Blinkit participate?) | Strategy, not eng | P0 |

---

## Use Case 5: Payment Authorization Without Credential Sharing

### Scenario
Your AI agent wants to pay ₹5,000 to a merchant. Today, it either has your UPI PIN / card details (dangerous), or it bounces you back to approve every transaction manually (defeats the purpose of an agent).

### What MPC Does
Using threshold signatures (Silent Shard):
- Your phone holds key shard 1
- The agent holds key shard 2
- The payment processor holds key shard 3
- Any 2 of 3 can authorize a transaction

So the agent + processor can execute payments up to a pre-set limit without your active involvement — but the agent alone can never authorize anything. Your phone can revoke the agent's shard at any time.

### Why This Matters
This is the authentication layer for agentic payments. It solves the fundamental tension: agents need to pay on your behalf, but you can't give them your credentials. Threshold signatures let you delegate authority without delegating keys.

This directly uses Silent Shard — SL's existing product. It's the most natural extension.

### Target Customers
- AI platforms building payment agents (OpenAI, Google, startups)
- Payment service providers / PSPs (Stripe, Razorpay, Adyen)
- Neobanks building AI-first experiences (Jupiter, Fi)
- Wallet providers integrating agentic flows
- UPI ecosystem (NPCI, if agent-initiated UPI becomes a standard)

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Agent key shard provisioning (user delegates shard to agent) | 4-5 weeks | P0 |
| 2-of-3 threshold signing for payment authorization | 3-4 weeks | P0 (Silent Shard exists) |
| Spending limit / policy engine (agent can spend up to ₹X per day) | 4 weeks | P0 |
| Shard revocation flow (user revokes agent access instantly) | 2-3 weeks | P0 |
| PSP/gateway integration SDK (Stripe, Razorpay adapters) | 4-6 weeks | P1 |
| Transaction audit trail (what the agent spent, when, where) | 3 weeks | P1 |
| Multi-agent support (different agents for different spend categories) | 4 weeks | P2 |

---

## Use Case 6: KYC/Identity Verification Across Agents

### Scenario
You're buying age-restricted goods (alcohol, medicine) via an agent. The merchant needs to verify you're 21+ but your agent shouldn't share your Aadhaar or date of birth.

### What MPC Does
- Your identity provider (DigiLocker / eKYC) holds your DOB
- Merchant's agent holds the rule: "age >= 21"
- MPC outputs: "yes" or "no"

No date of birth shared. No Aadhaar exposed. Just the boolean the merchant needs.

### Why This Matters
Every agent-mediated transaction that requires identity verification (age, nationality, accreditation for financial products) can use this. It's the zero-knowledge identity layer for commerce.

### Target Customers
- DigiLocker / India Stack ecosystem
- Age-gated e-commerce (alcohol, pharma, financial products)
- Cross-border commerce (nationality verification without passport sharing)
- Regulated product sales (accredited investor checks, prescription verification)

### What SL Needs to Build

| Component | Effort | Priority |
|---|---|---|
| Boolean attribute verification protocol (age >= X, country == Y) | 3-4 weeks | P0 |
| DigiLocker / eKYC integration (encrypted attribute input) | 4-5 weeks | P1 |
| Attribute schema (age, nationality, accreditation status, prescription validity) | 2 weeks | P1 |
| Merchant verification SDK (plug into checkout flow) | 3-4 weeks | P1 |
| Multi-attribute verification (age + nationality + accreditation in one call) | 3 weeks | P2 |

---

## Where SL Fits: The Product Map

| Layer of Agentic Commerce | What's Needed | SL Product |
|---|---|---|
| Authentication | Agent proves it acts for you, without holding your keys | Silent Shard (threshold signatures) |
| Authorization | Agent pays on your behalf within rules | Silent Shard (2-of-3 signing) |
| Data access | Agent computes on your data without seeing it | Silent Compute (MPC) |
| Negotiation | Agents transact without revealing private inputs | Silent Compute (MPC) |
| Identity | Agent proves attributes without revealing data | Silent Compute (ZK-adjacent) |

### The Thesis in One Line

Silent Shard is the authentication layer for agentic payments. Silent Compute is the data layer. Together, they're the privacy infrastructure for agentic commerce.

### Competitive Positioning

No other company has both pieces:
- Wallet MPC companies (Fireblocks, Fordefi) have the signing but not the compute
- Data clean rooms (Snowflake, AWS) have the compute but not the signing
- ZK companies (Polygon, zkSync) are blockchain-native, not commerce-native
- SL has both — and can offer them as a unified platform

---

## Feasibility & Timing

| Use Case | Feasibility | Reasoning |
|---|---|---|
| Payment authorization (threshold sigs) | Ready now | Direct extension of Silent Shard |
| KYC/identity verification | 6 months | Simple boolean MPC; low protocol complexity |
| Creditworthiness check | 6-9 months | Structured MPC; needs schema + partner integration |
| Agent-to-agent negotiation | 12 months | Multi-round MPC; protocol complexity + latency |
| Multi-platform basket optimization | 18 months | Multi-party MPC at scale; needs 3+ party support + platform adoption |
| Dynamic pricing without info asymmetry | 18+ months | Research-grade; needs secure ML capabilities |

---

## Recommended GTM Sequence

### Phase 1: Authentication (Now — Month 3)
Start with what's already built. Silent Shard for agent payment authorization.

- Partner with 1 PSP (Razorpay or Stripe) to offer threshold-signed agent payments
- Target: AI agent startups building commerce flows
- Message: "Let your AI agent pay without holding credentials"
- Revenue model: per-signing-operation fee

### Phase 2: Identity + Credit (Month 4-9)
Layer Silent Compute for simple boolean verifications.

- KYC verification for agent-mediated purchases
- Creditworthiness check for instant EMI via agents
- Partner with AA ecosystem for encrypted financial data input
- Message: "Verify without revealing"
- Revenue model: per-verification fee

### Phase 3: Negotiation + Optimization (Month 10-18)
Advanced MPC for multi-agent interactions.

- Agent-to-agent price negotiation
- Multi-platform basket optimization
- Partner with AI commerce platforms
- Message: "The negotiation layer for AI commerce"
- Revenue model: per-computation fee + platform subscription

### Phase 4: Intelligence (Month 18+)
Secure ML for privacy-preserving personalization.

- Dynamic pricing without information asymmetry
- Preference learning without data sharing
- This is the long-term moat
- Message: "AI that personalizes without surveillance"

---

## Key Metrics to Track

| Metric | Phase 1 Target | Phase 2 Target | Phase 3 Target |
|---|---|---|---|
| Partner integrations | 1-2 PSPs | 3-5 (PSPs + AAs + merchants) | 10+ (platforms + agents) |
| Monthly signing operations | 10K | 100K | 1M+ |
| Monthly MPC computations | — | 50K | 500K+ |
| Avg latency (signing) | <50ms | <50ms | <50ms |
| Avg latency (MPC verification) | — | <200ms | <500ms |
| Revenue | Pilot stage | First paid contracts | $1M ARR |

---

## Open Questions

1. **Agent identity standard:** Is there an emerging standard for how AI agents identify themselves in commerce? (OAuth for agents? Agent-specific certificates?)
2. **UPI + agents:** Will NPCI allow agent-initiated UPI transactions? If yes, threshold-signed UPI is a massive opportunity. If no, we route through card networks.
3. **Platform adoption incentive:** Why would BigBasket/Blinkit join a multi-party MPC for basket optimization? Need to design win-win incentive (access to demand signal without revealing supply).
4. **Regulatory stance:** RBI's position on AI agents making financial decisions on behalf of users is unclear. Need to track this.
5. **Latency budget:** Agentic commerce needs sub-second responses. Can MPC protocols hit <200ms for verification use cases? What's the current benchmark?

---

*This is a living document. Update as the agentic commerce ecosystem matures and SL's protocol capabilities expand.*
