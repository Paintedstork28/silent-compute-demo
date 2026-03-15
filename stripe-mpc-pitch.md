# MPC for Stripe — The Pitch

**Silence Laboratories × Stripe**
**Last updated:** March 2026

---

## Start Here: What Stripe Already Has

Stripe has Radar (cross-merchant fraud ML), Stripe Data, and access to $1.4T in annual transaction flow. They see more payment behavior than almost anyone. They've built clean room tools and have legal data-sharing arrangements with merchants.

So the honest question is: **why would Stripe need MPC?**

The answer is not regulation — at least not primarily. It's three things Stripe cannot do today with any amount of encryption or clean rooms.

---

## The Three Things Stripe Can't Do Today

### 1. Sell fraud intelligence to banks — without becoming a data liability

Stripe's fraud detection (Radar) works because Stripe sees transactions across millions of merchants. That cross-merchant signal is enormously valuable to banks and financial institutions.

The problem: to sell that signal to a bank, Stripe has to either share raw transaction data (banks won't accept it) or build a clean room where the bank connects their data (banks won't trust Stripe with it — they're competitors in payments).

**With MPC:** Stripe offers a fraud intelligence network. Banks connect their fraud signals. Stripe contributes its transaction signals. Neither side sees the other's raw data. The output is a better fraud score — computed privately across both datasets.

Stripe becomes the network operator of cross-institutional fraud intelligence. That's a new revenue line. No data custody, no liability, no trust problem.

---

### 2. Enable two Stripe merchants to collaborate — without Stripe in the middle

Two large Stripe merchants — say, a bank and a retailer — want to find their shared customer base for a co-marketing campaign. Today, they can't. Stripe can see both datasets but can't legally hand one merchant the other's customer list. The merchants can't share directly. A clean room requires both parties on the same platform with a trusted operator.

**With MPC:** The two merchants' nodes compute the overlap directly. Stripe facilitates the connection but never holds either dataset. The output: "You have 2.3M customers in common" — with no raw data exchanged.

Stripe monetizes the connection, not the data. This opens a whole category of merchant-to-merchant collaboration that doesn't exist today.

---

### 3. Verify without collecting — the DPDP / GDPR unlock

Every time Stripe processes a payment that requires eligibility verification — age, creditworthiness, identity — someone collects raw data. Stripe Identity collects documents. Lending partners collect bank statements. This creates compliance exposure under GDPR, India's DPDP Act, and increasingly strict data minimization rules.

**With MPC:** The verification is a boolean. "Is this person over 18?" — yes. "Are they creditworthy for this EMI?" — yes. No document, no statement, no PII stored anywhere. The raw data never leaves the user's environment.

This matters more as agentic commerce scales. Millions of autonomous agent transactions, each requiring an eligibility check — you can't store that data, and you can't afford the liability of holding it.

---

## What Stripe Gets

| Capability | What It Unlocks | Revenue Model |
|---|---|---|
| Cross-institution fraud network | Sell fraud intelligence to banks without data custody | Per query fee to banks |
| Merchant-to-merchant MPC | New category: private collaboration between merchants | Platform fee per computation |
| Boolean eligibility verification | Compliant checkout for regulated products at scale | Premium Identity tier |
| Threshold signing for agents | Agent payments without credentials (ACP upgrade) | Per-signing fee |

None of these require Stripe to replace anything they have. They're additive — new surfaces, new revenue, new moat.

---

## Why Now

**India:** DPDP Act enforcement begins 2025-2026. Any cross-party data computation involving Indian user data needs to demonstrate data minimization. Clean rooms don't satisfy this — data still moves to the clean room environment. MPC does.

**Agentic commerce:** Stripe is building ACP (Agentic Commerce Protocol) to handle AI agent payments. As agent transaction volume scales, credential-based auth (Shared Payment Tokens) becomes a systemic risk. MPC threshold signing is the upgrade path — same developer experience, mathematically stronger security.

**Competitive:** Visa and Mastercard are both exploring MPC for cross-network fraud intelligence. The network that builds this first owns the standard.

---

## The One-Line Pitch

> Stripe's clean rooms and encryption expand what you can do with data you're allowed to share. MPC expands what you can compute on data that can never be shared — competitors, regulated data, cross-border. That's a different problem, and it's the one that's getting harder.

---

## What We're Asking For

A 30-minute conversation with Stripe's product team on two specific surfaces:

1. **Radar + MPC** — cross-institution fraud intelligence network. What would it take to pilot this with 3 banks on Stripe's network?
2. **ACP + threshold signing** — upgrading SPTs for high-volume agent payments. Can we run a benchmark against Stripe's latency requirements?

We're not asking Stripe to rebuild anything. We're asking where the boundaries of Radar and Identity are — and whether MPC closes the gap.

---

*Technical detail on ACP integration: see `stripe-acp-mpc-positioning.md`*
*Silent Compute platform: platform.silencelaboratories.com*
