# Ways of Working: Product @ Silent Labs

## Context

Product is a new function at SL. Until now, product decisions were made by the founding/engineering team (and that worked — it got us here). Product's role is not to replace that judgment, but to **add structure, prioritization, and customer context** so the team ships faster and with more clarity.

---

## What Product Does (and Doesn't Do)

| Product DOES | Product DOES NOT |
|-------------|-----------------|
| Own the "what" and "why" — what we build and for whom | Own the "how" — engineering decides architecture and implementation |
| Prioritize the backlog based on customer + business input | Make decisions without eng/biz input |
| Write specs, acceptance criteria, success metrics | Write code or design UIs (unless needed) |
| Run discovery — talk to customers, analyze competitors | Replace founder intuition — we augment it |
| Say "not now" to good ideas that don't fit the current phase | Say "no" permanently — everything gets captured |

**The mental model:** Product is the **editor**, not the **author**. The team has been writing the story. Product helps structure it, cut what's not working, and sequence what's next.

---

## Rituals

### 1. Daily Standup (15 min, async-first)

**Format:** Slack/async by default. Sync only if needed.

Each person posts by 10:30am:
```
Done: [what I shipped/completed yesterday]
Today: [what I'm working on]
Blocked: [anything I need help with]
```

**Sync standup** (2-3x/week, 15 min max) only when:
- Active sprint with dependencies
- Multiple people blocked
- Launch week

**Product's role in standup:** Listen. Unblock. Don't manage.

### 2. Weekly Product Sync (30 min, Mondays)

**Who:** Product + Eng Lead + Research Lead + Founder(s)
**Purpose:** Align on the week's priorities

**Agenda:**
```
1. What shipped last week (5 min)
2. What's planned this week (10 min)
3. Any priority changes / new inputs (10 min)
4. Blockers / decisions needed (5 min)
```

**Product's role:** Come with a proposed priority list. Get buy-in, not sign-off. If founder disagrees, discuss — don't override.

**Output:** Updated weekly priorities shared in Slack/Notion within 1 hour.

### 3. Bi-Weekly Backlog Review (45 min, alternate Wednesdays)

**Who:** Product + Eng Lead (+ founders optional)
**Purpose:** Groom, reprioritize, estimate

**Agenda:**
```
1. Review new requests/ideas added since last review (15 min)
2. Reprioritize top 10 backlog items (15 min)
3. Estimate upcoming items (15 min)
```

**Product's role:** Every item in the backlog has: a one-line problem statement, who it's for, and a priority tag (P0/P1/P2).

### 4. Monthly Roadmap Check-in (60 min, first Friday of month)

**Who:** Full team (eng + biz + product + founders)
**Purpose:** Zoom out. Are we building the right things?

**Agenda:**
```
1. What we shipped this month (10 min)
2. Customer/market signals (10 min)
3. Roadmap review — next 4-8 weeks (20 min)
4. Open floor — what's on your mind? (20 min)
```

**Product's role:** Present a 1-pager: what we learned, what changed, what's next. Keep it visual — not a 40-slide deck.

---

## How Decisions Get Made

### Decision Framework (RACI-lite)

| Decision Type | Who Decides | Who's Consulted | Who's Informed |
|--------------|------------|----------------|---------------|
| What to build next | Product + Founder | Eng Lead, Research | Team |
| Technical architecture | Eng Lead | Product, Research | Founder |
| Protocol design / crypto primitives | Research | Eng Lead, Product | Founder |
| Feasibility of a customer use case | Research + Eng Lead | Product | Biz |
| Customer-facing messaging | Founder + Biz | Product | Eng, Research |
| Timeline / scope tradeoffs | Product + Eng Lead | Founder, Research | Team |
| Pricing / GTM | Founder + Biz | Product | Eng |
| Hiring / resourcing | Founder | Product + Eng Lead | Team |

### The "Disagree and Commit" Rule

At seed stage, speed > consensus. If we disagree:
1. Discuss for max 15 minutes
2. If no alignment — the person closest to the problem decides
3. Everyone else commits fully
4. We revisit in 2-4 weeks with data

---

## How Product Takes Over Gracefully

### Month 1: Listen and Map (Weeks 1-4)

- Sit in every meeting. Don't change anything yet.
- Map: who makes what decisions today? What's the current process?
- 1:1 with every team member: "What's working? What's frustrating? What do you wish someone would own?"
- Document the current roadmap, backlog, and decision-making process as-is
- **Output:** Current state doc. Share with team: "Here's what I've understood. What am I missing?"

### Month 2: Add Structure (Weeks 5-8)

- Introduce the weekly product sync (start with founders + eng lead only)
- Create a single backlog (consolidate scattered lists/tickets/Slack threads)
- Start writing lightweight specs for the next 2-3 features
- **Don't say:** "We need a process for this"
- **Do say:** "I've put together a backlog — can we review it together?"

### Month 3: Own It (Weeks 9-12)

- Run the weekly sync and backlog review
- Start saying "not now" to low-priority requests (with reasoning)
- Introduce the monthly roadmap check-in
- **Don't say:** "This is now my decision"
- **Do say:** "Based on what I'm hearing from customers and the team, here's my recommendation"

---

## Communication Norms

| Channel | Use For | Response Time |
|---------|---------|--------------|
| Slack (channel) | Daily updates, quick questions, FYIs | Same day |
| Slack (DM) | Urgent blockers, sensitive topics | Within 2 hours |
| Notion/Linear | Specs, backlog, roadmap, documentation | Async |
| Meeting | Decisions that need debate, alignment | Scheduled |
| Email | External communication only | 24 hours |

**Rule:** If it takes >3 Slack messages to resolve, hop on a 5-min call. If a call runs >15 min, schedule a proper meeting.

---

## What Product Needs From Each Team

### From Engineering
- Flag technical risks early — don't surprise me at sprint end
- Push back on specs that don't make sense
- Estimate honestly — I'd rather plan for 2 weeks than discover it at week 3
- Tell me when product is over-speccing — "this doesn't need a spec, it's a 2-hour fix"

### From Business / Founders
- Share customer conversations — even rough notes
- Tell me when priorities shift — I'd rather know early
- Give me context on why a request matters — "client X asked for Y" is more useful than "we need Y"
- Trust that "not now" doesn't mean "never"

### From Research / Cryptography
- Proactive heads-up on what's feasible vs what's research-grade — before product commits to a timeline
- Flag when a customer use case requires a new protocol (vs can be built on existing primitives)
- Rough effort estimates: "this is a known problem, 4 weeks" vs "this is open research, 3-6 months"
- Share paper reviews / competitive intel — if a competitor publishes something relevant, product should know

---

## What Teams Can Expect From Product

- A clear, prioritized backlog — always current, never stale
- Specs before eng starts building (even lightweight ones)
- A weekly written update on what's happening and why
- A "no surprises" policy — if priorities change, you'll know first
- Credit where it's due — product doesn't ship anything, the team does

---

## Tools (Suggested)

| Purpose | Tool | Why |
|---------|------|-----|
| Backlog + tickets | Linear | Fast, eng-friendly, good for seed stage |
| Specs + docs | Notion | Flexible, easy to share, low overhead |
| Communication | Slack | Already using it |
| Roadmap | Notion or spreadsheet | Don't over-tool at seed stage |
| Customer feedback | Notion DB or Airtable | One place to capture all signals |

---

## Working With Research

### Why Research Is Different

SL is a cryptography company. Unlike a typical SaaS startup where eng can build whatever product specs, here some features depend on **protocols that don't exist yet** — they need to be designed, proven secure, and optimized before eng can implement them. Research is not a support function; it's a core dependency in the build pipeline.

**The pipeline:**
```
Customer need → Product spec → Research feasibility check → Protocol design → Eng implementation → Ship
                                    ↑
                          This step can take 2 weeks or 6 months.
                          Product MUST know which one before committing.
```

### When Research Must Be Consulted

| Trigger | What to Ask Research | Expected Output |
|---------|---------------------|-----------------|
| New use case from a customer or prospect | "Can we do X with our current protocols?" | Yes / Yes with modifications / No, needs new protocol / Not feasible |
| Product spec for a new feature | "What's the crypto overhead? Latency? Trust assumptions?" | Feasibility memo: what's possible, what's hard, what's a dealbreaker |
| Customer asks about security guarantees | "What can we formally claim? What are the caveats?" | Clear language for what we can and can't promise |
| Competitor ships something new | "Is their approach better? Should we adapt?" | Competitive analysis from a protocol perspective |
| Eng hits a performance bottleneck | "Can the protocol be optimized, or is this a fundamental limit?" | Optimization path or "this is the theoretical floor" |

### Research Rituals

#### Bi-Weekly Research ↔ Product Sync (30 min)

**Who:** Product + Research Lead
**Purpose:** Keep product informed on what's coming out of research, and keep research informed on what customers need.

**Agenda:**
```
1. Research updates — what's progressing, what's stuck (10 min)
2. Product pipeline — upcoming features that may need research input (10 min)
3. Feasibility requests — new items product needs assessed (10 min)
```

**This is the most important meeting for Product at SL.** If you skip this, you'll either promise customers something research can't deliver, or miss something research already solved.

#### Research Review (Monthly, 45 min)

**Who:** Research + Eng Lead + Product + Founder
**Purpose:** Deep dive on 1-2 research topics that are approaching production-readiness.

**Format:**
- Research presents: what they built, what it does, performance benchmarks, limitations
- Eng asks: how do we implement this? What's the API surface?
- Product asks: which customer use cases does this unlock? What can we sell?
- Founder asks: does this change our competitive position?

### The Research ↔ Product Contract

| Product commits to | Research commits to |
|-------------------|-------------------|
| Never promising customers a timeline without checking with research first | Giving honest effort estimates — "2 weeks" vs "3 months" vs "we don't know yet" |
| Writing use case briefs that explain the *customer problem*, not the *protocol needed* — let research figure out the how | Flagging early when something is harder than expected — don't surface it at deadline |
| Not treating research as a blocker to complain about — they're solving hard problems | Translating crypto concepts into plain language for product/biz — no jargon walls |
| Including research in roadmap planning — their work IS the roadmap for future features | Proactively sharing when a breakthrough unlocks new use cases — don't wait to be asked |

### How to Handle the "Research Timeline" Problem

The hardest part of working with a research team: **some things can't be estimated.**

A protocol design might take 2 weeks or 2 months. A security proof might need 3 iterations. This is normal — not a process failure.

**How product handles this:**

1. **Classify every feature as "eng-ready" or "research-dependent"**
   - Eng-ready: protocol exists, just needs implementation. Normal sprint planning.
   - Research-dependent: needs new protocol work. Goes into a separate "research pipeline" with wider time ranges.

2. **Use ranges, not deadlines, for research-dependent work**
   - "This feature needs 4-8 weeks of research + 3 weeks of eng" — not "shipping in 6 weeks"
   - Communicate ranges to customers and biz. Underpromise.

3. **Create a "research graduation" checklist**
   - Before eng picks up a research-dependent feature, research must deliver:
     - [ ] Protocol spec (what it does, inputs, outputs, trust model)
     - [ ] Security proof or argument (even informal)
     - [ ] Performance benchmarks (latency, compute cost)
     - [ ] Known limitations / edge cases
   - If any of these are missing, the feature stays in research. No exceptions.

4. **Parallel-track when possible**
   - Eng can build the SDK, API wrapper, and integration layer while research finalizes the protocol
   - Product identifies what can be built in parallel to avoid idle eng time

---

## Anti-Patterns to Avoid

1. **Don't introduce too many rituals at once.** Start with the weekly sync. Add others as the team feels the need.
2. **Don't gate engineering.** If an engineer knows what to build and it's aligned with priorities — they don't need to wait for a spec.
3. **Don't create a "product approval" bottleneck.** Product's job is to enable speed, not control it.
4. **Don't present the roadmap as "product's roadmap."** It's the team's roadmap. Product just writes it down.
5. **Don't compete with the founder's vision.** At seed stage, the founder IS the product visionary. Your job is to translate that vision into shippable increments.
6. **Don't treat research like eng with slower timelines.** Research work is non-linear. A breakthrough can collapse 3 months into 1 week, and a dead-end can extend 2 weeks into 3 months. Plan accordingly.
7. **Don't commit customer timelines on research-dependent features.** Always check with research first. "Let me confirm feasibility and get back to you" is always the right answer.
8. **Don't skip the feasibility check.** The most expensive mistake at a crypto company: promising a feature, building the wrapper, then discovering the underlying protocol can't support it.

---

## The One-Liner

**Product at SL exists to make the team ship faster, with more clarity, and with fewer "wait, why are we building this?" moments.**
