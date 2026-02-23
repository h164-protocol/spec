# HIP-0: Protocol Vision & Manifesto

**H.164 Improvement Proposal 0**
**Title:** The Intelligence Layer for Every Phone Number on Earth
**Author:** Dustin Goodwin
**Status:** Draft
**Created:** 2026-02-23

---

## TL;DR

Every AI agent will need to reach a human service provider — a plumber, a lawyer, a tow truck, a dentist. There is no structured, trust-scored, machine-readable way to do this today. H.164 wraps E.164 phone numbers with capabilities, availability, trust, and activation methods, creating the resolution layer between agents and the physical world. Both sides — providers and consumers — carry reputation and set consent thresholds. The interaction log compounds with every resolution, becoming an unforkable intelligence asset. The protocol is open (Apache 2.0) because it has to be. The company builds the dominant commercial infrastructure on top — the Red Hat model applied to telephony's missing intelligence layer.

---

## Abstract

There are over 8.3 billion active mobile subscriptions in the world. Each one points to a human. None of them carry structured information about what that human can do, when they're available, whether they can be trusted, or how to activate them.

H.164 changes this. It is an open protocol that wraps E.164 phone numbers with machine-readable metadata — capabilities, availability, trust, and activation methods — creating the first structured resolution layer between AI agents and the humans they need to reach.

Both Service Nodes and Request Nodes carry reputation, enabling consent-gated, spam-resistant resolution at planet scale.

This document is not the schema. It is not the technical specification. It is the intellectual and architectural case for why H.164 must exist, why it must be open, and what it becomes at scale.

---

## 1. The 8 Billion Number Vision

There are over 8.3 billion active mobile subscriptions worldwide. The initial use case for H.164 is obvious and immediate: approximately 400 million of those are business phone numbers that AI agents currently cannot resolve against. When an agent needs a plumber, a tow truck, a dentist, a lawyer — it hits a wall. There is no structured, queryable, trust-scored index of service providers that an agent can call programmatically. The agent either falls back to web search (built for humans, not machines) or gives up entirely.

This is the beachhead. It is not the vision.

The full scope of H.164 is 8 billion humans, each with their own agent, each rooted to a phone number. Every interaction between agents — whether requesting a service or providing one — flows through phone number endpoints. Both sides of every interaction are phone numbers. Both sides carry reputation. Both sides set consent thresholds. Both sides are wrapped.

H.164 defines two node types: **Service Nodes** (the providers) and **Request Nodes** (the consumers). The service node wraps a plumber's phone number with what they do, where they work, when they're available, and whether they're trustworthy. The request node wraps a consumer's phone number with who their agent is, what they need, how they've behaved in past interactions, and what their consent boundaries are.

Every agent-mediated interaction becomes bilateral: a request node (consumer + agent) negotiates with a service node (provider). This creates the first native consent layer for the agent economy — exactly the kind of decentralized human-AI coordination infrastructure that the next wave of protocol builders will be built on.

This is not a business directory protocol. It is the identity and trust layer for every agent-mediated human interaction on earth.

Phase 1 bootstraps with a thin index of 50,000+ trade-service providers — plumbing, electrical, HVAC, towing — in high-density markets. Public data and manual outreach seed Tier 3–4 nodes. Real agent interactions then enrich them toward Tier 1–2, proving resolution utility before scaling request nodes and additional verticals. The thin index proves the protocol works. The interaction log proves it compounds.

At full deployment, H.164 becomes the substrate on which agents negotiate, transact, verify, and resolve — not with servers, not with APIs, not with wallet addresses, but with the phone numbers that already represent every human alive.

---

## 2. Why Phone Numbers, Not Any Other Primitive

The choice of E.164 phone numbers as the root primitive is not arbitrary. It is the only choice that works.

Consider the alternatives:

**Email addresses** are globally distributed but fundamentally broken as identity primitives. They are trivially spoofable — anyone can register any address at any provider. They are disposable — a user can create and abandon addresses at zero cost, meaning reputation cannot compound. They require no verification of humanness. An email address proves that someone controls a string at a domain. A phone number proves that a carrier has verified a human endpoint.

**Wallet addresses** are cryptographically strong but not universal. Fewer than 500 million humans hold cryptocurrency wallets. Wallet addresses are pseudonymous by design — they are optimized for privacy, not for activation. You cannot call a wallet address. You cannot send it a dispatch. It is an identity primitive for financial transactions, not for human service resolution.

**Social handles** (X, Discord, Telegram) are increasingly used by agents as quasi-identifiers, but they fail every test: not carrier-verified, trivially disposable, platform-dependent, and not activation endpoints. An agent can DM a plumber on X — but it cannot dispatch one. Social handles are communication channels, not identity primitives.

**URLs** point to servers, not to humans. A URL is a location on the internet. It has no inherent connection to a human being, no carrier-verified identity behind it, and no activation capability. You can serve a webpage from a URL. You cannot dispatch a tow truck.

**Government-issued IDs** (SSNs, passport numbers, national IDs) are verified and unique, but they are private by design, jurisdiction-dependent, and structurally inappropriate for machine-to-machine resolution. No human publishes their SSN as a service endpoint. These identifiers exist for regulatory compliance, not for activation.

The phone number is unique across all alternatives because it satisfies four requirements simultaneously:

1. **Universal.** There are more phone numbers than humans. Every country, every carrier, every demographic.
2. **Carrier-verified.** A phone number is issued and maintained by a telecommunications carrier that has verified the holder's identity (to varying degrees by jurisdiction, but always more than email).
3. **Portable.** Numbers transfer between carriers and devices. The number persists even as the underlying infrastructure changes.
4. **The activation endpoint itself.** This is the critical distinction. A phone number is not a pointer to some other system — it IS the system. You resolve a need, you get a number, you call that number, and a human answers. The identifier and the activation mechanism are the same object.

A URL points to a server. A phone number points to a human.

H.164 builds on the only primitive that is simultaneously an identity, a trust anchor, and an activation endpoint.

A note on verification: carrier verification is a baseline, not a guarantee. Prepaid phones, VoIP numbers, and SIM swaps mean that phone number identity has real attack surfaces. H.164 treats carrier verification as the floor, not the ceiling. The trust score, interaction history, and six-layer verification stack (HIP-0 Section 7) are what establish real reliability. A phone number gets you into the index. Behavior keeps you there.

---

## 3. The Four Locks That Kept Phone Numbers Unwrapped

If phone numbers are so obviously the right primitive, why hasn't anyone wrapped them before? Because four structural locks prevented it — and AI agents have broken all four simultaneously.

### Lock 1: Regulatory Protection

Phone numbers exist inside a fortress of telecommunications regulation. TCPA in the United States. GDPR in Europe. STIR/SHAKEN across carriers. These frameworks were built to protect human-to-human communication from spam, fraud, and abuse. They work. They also make programmatic access to phone number endpoints enormously constrained. You cannot legally robocall 10,000 plumbers to ask if they're available. The regulatory stack assumes that the entity calling a phone number is a human — or a bad actor pretending to be one.

AI agents are neither. They are authorized representatives of humans, making legitimate requests on their behalf. The regulatory frameworks are beginning to accommodate this reality, but more importantly, the interaction model is fundamentally different. An agent doesn't need to call 10,000 numbers. It needs to query an index, resolve against structured metadata, and call one number — the right one. H.164 makes the call legitimate by making it informed.

### Lock 2: Privacy as Feature

Phone numbers are personally identifiable information that humans have historically controlled the sharing of. You give your number to people you trust. The unlisted number was a feature, not a bug. Building a structured, queryable index of phone numbers sounds like a privacy nightmare.

But service providers already publish their numbers. Businesses want to be found. The privacy concern is real for the request node side — consumer phone numbers — and H.164 addresses this directly through consent thresholds, reachability controls, and minimum trust score requirements. The protocol doesn't expose private numbers. It wraps numbers that are already public (service nodes) and protects numbers that should remain private (request nodes) behind consent infrastructure.

### Lock 3: Voice as Unstructured Signal

For the entire history of telephony, the payload of a phone call has been unstructured audio. A human speaks, another human listens. There was no way to programmatically process what happened during a call, extract structured outcomes, or feed interaction data back into an index. The phone call was a black box.

AI changes this completely. Voice AI can conduct calls, transcribe them in real time, extract structured data from conversations, and report outcomes. The phone call is no longer a black box — it is a structured data source. This means interaction outcomes can flow back into the protocol, updating trust scores, confirming availability, and building the interaction log that makes H.164's index increasingly intelligent.

### Lock 4: Carrier Economics

Telecommunications carriers have operated as dumb pipes for decades. They transmit calls. They don't understand calls. They have no economic incentive to add intelligence to the phone number layer because their business model is metered minutes and data plans, not resolution quality.

AI agents create new economic incentives. When agents need to resolve real-world needs against phone number endpoints, the value of a structured, trust-scored, availability-aware index becomes enormous. The carrier doesn't need to build this — in fact, the carrier can't build this, because the intelligence comes from interaction outcomes, not from network topology. H.164 sits alongside the carrier infrastructure and adds the value layer that carriers structurally cannot provide.

All four locks broke at the same time because they were all held in place by the same assumption: that phone calls are conversations between humans. AI agents invalidate that assumption. H.164 is the protocol that builds on the new reality.

---

## 4. The Dumb Rail Problem

The global telephony stack runs on signaling protocols designed in the 1970s. SS7 — Signaling System 7 — is the backbone. When a call is placed, SS7 transmits three pieces of information: the calling number, the called number, and sometimes a display name (CNAM). That's it.

No intent. No context. No trust signal. No capability declaration. No urgency indicator. No availability status. No service category. No outcome history.

A 2026 AI agent attempting to resolve "I need an emergency plumber within 15 minutes in a 5-mile radius who is licensed, insured, and has a trust score above 0.8" must work with a signaling infrastructure that can express exactly none of those constraints. The agent can search the web, find a phone number, and place a call — reverting to the same resolution process a human would use, minus the human judgment about which result to trust.

This is the dumb rail problem. The rails exist. They work. They carry 500 billion minutes of voice traffic per year globally. But they carry that traffic with zero intelligence about what the traffic means.

H.164 does not replace SS7. It does not replace SIP. It does not compete with the telecommunications stack. It sits alongside it and adds the structured intelligence that telephony was never designed to carry.

The analogy is precise: HTTP did not replace TCP/IP. TCP/IP handles packet routing — getting data from point A to point B. HTTP added the application layer — defining what the data means and how to interact with it. H.164 adds the application layer to telephony. SS7 routes the call. H.164 defines whether the call should be placed, to whom, with what expectations, and how to evaluate the outcome.

---

## 5. The Protocol Gap

The AI agent ecosystem is building its protocol stack in real time. Three major protocols have emerged:

| Protocol | Layer | Function |
|----------|-------|----------|
| **MCP** (Model Context Protocol) | Agent ↔ Tool | How an agent connects to software tools, databases, and APIs |
| **A2A** (Agent-to-Agent) | Agent ↔ Agent | How agents communicate, delegate, and coordinate with each other |
| **ACP** (Agent Commerce Protocol) | Agent ↔ Commerce | How agents discover and transact for products (Stripe's layer) |

All three protocols share a structural assumption: both endpoints are digital. Tool APIs. Agent endpoints. Product catalogs. Digital-to-digital.

But the real world is not digital. When an agent needs to dispatch a tow truck, book a dentist appointment, or retain a defense attorney, one side of the interaction is a digital agent and the other side is a human being reachable at a phone number. This layer — **agent to human service provider** — has no protocol.

| Protocol | Layer | Function |
|----------|-------|----------|
| MCP | Agent ↔ Tool | Digital ↔ Digital |
| A2A | Agent ↔ Agent | Digital ↔ Digital |
| ACP | Agent ↔ Commerce | Digital ↔ Digital |
| **H.164** | **Agent ↔ Human Service** | **Digital ↔ Physical** |

H.164 composes cleanly with the existing stack: MCP provides the tool interface (agent queries H.164 as a tool), A2A enables delegation (agent asks another agent to resolve), ACP handles payment (settlement after activation). The full chain: **query → resolve → activate → pay.**

H.164 fills the gap. It is the protocol that bridges the digital agent stack and the physical world of human service providers, using the phone number as the universal endpoint.

This gap is not optional. It is the bottleneck. Every agent platform — ChatGPT, Claude, Gemini, vertical AI agents in insurance, healthcare, legal, home services — will need to resolve real-world needs against human providers. Today, each platform builds its own brittle, proprietary, incomplete solution. H.164 provides the shared infrastructure.

---

## 6. Bidirectional Trust as Consent Infrastructure

Without H.164, the agent economy has a spam problem that makes email spam look quaint.

Consider: if every AI agent on every platform can query every phone number on earth to resolve needs for their users, every phone number becomes an unfiltered endpoint for millions of automated requests. A plumber's phone number would receive agent queries from every platform, every geography, every urgency level — with no mechanism to filter, prioritize, or refuse. The plumber's phone becomes unusable.

This is not hypothetical. It is the inevitable consequence of AI agents interacting with unstructured phone number endpoints at scale.

H.164 solves this by making trust bidirectional and consent structural.

**Service nodes** set minimum trust thresholds for incoming queries. A plumber can declare: "I only accept queries from agents whose request nodes have a trust score above 0.7, a verified payment method, and a completion rate above 85%." Agents representing unreliable requestors — those who don't follow through on dispatches, dispute charges frivolously, or waste provider time — are filtered before they ever reach the provider.

**Request nodes** set consent boundaries for outbound contact. A consumer can declare: "Only contact me for service categories I've opted into, only during these hours, only through these channels, and only if the provider's trust score exceeds my threshold." This prevents providers (or their agents) from initiating unwanted contact.

This is **STIR/SHAKEN extended to agent-mediated calls with pre-call trust attestation**. STIR/SHAKEN verifies that a human caller is who they claim to be. H.164 goes further: it verifies that an agent is trustworthy, authorized, and operating within consent boundaries on both sides of the interaction — before the call is ever placed.

The protocol is fundamentally pro-human. It does not exist to help machines reach people more efficiently — it exists to ensure that when machines reach people, the interaction is wanted, qualified, and accountable. Every interaction is logged, every outcome is recorded, and both sides build reputation based on real behavior.

Without this layer, the agent economy becomes a machine-to-human spam network. With it, the agent economy becomes a trust-mediated, consent-governed resolution system. H.164 is the difference.

---

## 7. The Oracle Problem

In 2017, the decentralized finance ecosystem faced an existential question: how does a smart contract know the price of ETH? The contract lives on-chain. The price exists off-chain. There was no trustworthy mechanism to bring real-world data into the blockchain environment.

Chainlink solved this with decentralized oracle networks — systems that aggregate real-world data from multiple sources, verify it through consensus mechanisms, and deliver it on-chain with cryptographic guarantees. The oracle problem was the bottleneck for DeFi. Chainlink was the unlock.

H.164 faces the same structural challenge in a different domain: **how does the index know the plumber actually showed up?**

An agent queries the H.164 index. The index returns a ranked set of service nodes. The agent activates one — calls the plumber. The plumber says they'll be there in 30 minutes. Did they show up? Did they fix the problem? Did they charge what they quoted? Did the customer pay? Was the outcome satisfactory?

If the index cannot answer these questions with verified data, trust scores are meaningless. They become Yelp stars — manipulable, stale, and disconnected from real outcomes. The index becomes a directory, not an intelligence layer.

H.164 defines a six-layer verification stack for outcome signals:

1. **Transaction confirmation.** Was payment processed? What was the amount relative to the quote? This is the hardest signal to fake and the strongest indicator of real service delivery.

2. **Temporal chain.** Does the timeline make sense? Query at 2:00 PM, dispatch confirmed at 2:05, arrival signal at 2:35, completion signal at 3:45. Temporal consistency across the interaction event chain validates that real activity occurred.

3. **Counter-call verification.** The provider's node and the requestor's node each independently report interaction outcomes. When both sides report consistent data, confidence is high. Discrepancies trigger review.

4. **Absence of negative signal.** No dispute filed. No chargeback initiated. No complaint registered. Absence of negative signal is not proof of quality, but persistent absence across many interactions is a strong probabilistic indicator.

5. **Repeat activation.** The same requestor uses the same provider again. Revealed preference is the strongest trust signal available — people don't hire the same plumber twice if the first experience was bad.

6. **Cross-node reputation.** The provider's trust score across all interactions, not just this one. A provider with 500 verified positive outcomes who has one disputed interaction is evaluated differently than a provider with 3 total interactions and 1 dispute.

These layers compound. No single layer is sufficient. Together, they produce a confidence-weighted outcome score that the index uses to update trust scores, adjust availability tiers, and improve future resolution quality.

These layers activate progressively as network maturity grows. At bootstrap, temporal chain consistency and absence of negative signal are immediately available. Transaction confirmation and counter-call verification require payment integration and provider awareness, which arrive with Tier 1-2 nodes. Repeat activation and cross-node reputation require interaction volume that compounds over months. The stack is designed to start useful and grow stronger — not to require full deployment before producing value.

This is the oracle layer for the physical world. Chainlink answers "what is the price of ETH?" H.164 answers "did the service happen, and was it good?"

H.164's oracle stack is blockchain-native by design. Outcome events can be attested on-chain — via Ethereum attestations, EAS, or zero-knowledge proofs — turning physical service resolutions into verifiable, composable assets. A completed plumbing repair becomes a cryptographically signed outcome that any protocol can reference. Chainlink proved that oracles unlock DeFi. H.164 unlocks the physical economy.

---

## 8. Why Open

H.164 must be an open protocol. This is not idealism. It is the only architecture that works.

**Google is structurally misaligned.** Google's business model is advertising. A trust-scored, transparent resolution layer that routes agents directly to the best provider — bypassing search results, bypassing ads, bypassing the entire attention economy — creates tension with Google's revenue model. Google will build proprietary agent resolution, but it will be optimized for monetization, not for trust. The index will be opaque because transparency undermines the ad model.

**OpenAI is building the agent, not the infrastructure beneath it.** The major AI labs are focused on the agent layer — models, reasoning, and tool use. The protocol stack forming around them (Anthropic's MCP for agent-tool integration, Google's A2A for agent-agent communication) is digital-to-digital. The resolution index that connects agents to physical-world humans is not their layer, just as Google Search was not Netscape's layer. Agent platforms will consume H.164. They will not build it.

**Stripe's ACP solves a different problem.** Stripe's Agent Commerce Protocol handles product discovery and transactions — digital goods with SKUs and prices. It does not cover services, which are inherently local, availability-dependent, trust-sensitive, and phone-number-activated. Buying a product and dispatching a tow truck are fundamentally different interactions. ACP handles the first. H.164 handles the second.

**A closed index gets zero ecosystem buy-in.** If H.164 were proprietary, every agent platform would build its own competing index rather than depend on a single company's infrastructure. The result would be fragmentation — ten incomplete indexes instead of one comprehensive one. Network effects would never compound. The interaction log would never achieve critical mass.

The protocol must be open for the same reason HTTP had to be open. The web could not have been built on a proprietary hypertext protocol controlled by a single company. The value of the protocol is proportional to the number of participants, and participants only join if they trust that the rules won't change to disadvantage them.

The model is Linux and Red Hat. Linus Torvalds released the kernel as open source. Thousands of contributors built on it. Red Hat became a $34 billion company not by owning Linux, but by being the most trusted commercial infrastructure built on top of it — enterprise support, packaging, certification, the operational layer that made the open kernel useful to Fortune 500 companies. The protocol created the market. The company captured value by being the indispensable builder in that market.

H.164 follows the same architecture. The specification is open, governed by the community, licensed under Apache 2.0. The commercial entity — the company — builds the reference implementation, operates the canonical index, provides enterprise APIs, and delivers the tooling that makes the protocol useful at scale. At launch, the team builds both the spec and the reference implementation. As adoption grows, spec governance separates from the commercial entity — the same path Linux took from Torvalds' personal project to the Linux Foundation. Linux is the kernel. Red Hat is the company. H.164 is the spec. We are the Red Hat.

This is the only model that produces both ecosystem adoption and commercial defensibility. Open enough that every agent platform integrates. Commercial enough that the canonical index, the enterprise tooling, and the interaction log create a business worth building.

The Ethereum Attestation Service (EAS) and zero-knowledge outcome proofs are first-class citizens in the reference implementation. Concretely: an insurance protocol can verify on-chain that a tow was dispatched and completed before auto-processing a roadside claim — eliminating manual adjudication. A lending protocol can verify that a licensed contractor completed a home repair before releasing escrow. Every resolved service becomes a cryptographically signed outcome that other protocols can reference without trusting a centralized intermediary.

---

## 9. The Interaction Log as Moat

Protocols can be forked. Schemas can be copied. Index data can be scraped. But the interaction log cannot be replicated.

The interaction log is the record of every query, every match, every activation, and every outcome that flows through the H.164 index. It exists only because real agents made real queries, real providers were activated, real services were delivered, and real outcomes were recorded.

A competitor can copy the H.164 schema on day one. They can populate an index with business listings scraped from public sources. They can even replicate the trust score algorithm. What they cannot replicate is the millions of interaction events that have flowed through the canonical index — the outcomes that make trust scores meaningful, the patterns that make availability predictions accurate, the cross-vertical intelligence that makes resolution quality compound over time.

The interaction log is to H.164 what edit history is to Wikipedia. You can fork Wikipedia's content today. By tomorrow, your fork is already behind — because real editors are making real edits to the canonical version, and those edits are informed by the accumulated context of millions of previous edits. The fork is a snapshot. The original is a living system.

Every interaction that flows through H.164 makes the index smarter. Every outcome signal refines trust scores. Every resolution pattern improves matching algorithms. Every cross-vertical interaction enriches the network. The log compounds daily, and the compound interest is the moat.

This is not a defensibility argument based on secrecy or lock-in. The protocol is open. The schema is public. The log is valuable precisely because it is comprehensive, verified, and continuously growing. A competitor must not only build the technology — they must generate the interaction volume. And interaction volume follows from ecosystem adoption, which follows from trust, which follows from being open and first.

The first comprehensive interaction log wins. It wins because every subsequent participant has more reason to join the network with the most data than to start a new one with none.

The canonical log is initially operated by the H.164 reference implementation team, with a clear migration path to community governance via future HIP. Decentralization of the log is a feature of maturity, not a prerequisite for launch.

---

## 10. The Network Effect Across Verticals

Traditional service marketplaces are vertical. Angi for home services. Zocdoc for healthcare. Avvo for legal. Each builds its own data within its category, and none of it transfers.

H.164 is horizontal by design. A single protocol and a single index serve every vertical, and the intelligence compounds across all of them.

When a dental office's agent handles appointment scheduling through H.164, the interaction data — response times, completion rates, trust signals — feeds back into the same index that tow truck operators, defense attorneys, dog groomers, and emergency plumbers are scored against. The cross-vertical intelligence is not in the per-node trust scores (those are node-specific). It is in the infrastructure that sits beneath them: the availability prediction models learn response-time patterns across all verticals, the fraud detection systems learn manipulation signatures from heterogeneous data that is harder to game than single-vertical patterns, and the trust decay calibration becomes more accurate with diverse signal volume. The models improve. Every vertical benefits.

This cross-pollination is the structural network effect. It means:

- **One protocol wins, not ten.** A vertical-specific protocol for plumbers and a different one for dentists fragment the network effect. A single protocol that handles both concentrates it. Every node added to any vertical increases the value of the entire index.

- **Gaming is harder.** In a single-vertical marketplace, a provider can learn the scoring patterns and optimize for them. In a cross-vertical index, the scoring models incorporate signal patterns from dozens of service categories, making manipulation exponentially more difficult.

- **Cold start is solvable.** A new vertical added to H.164 immediately benefits from the trust models, availability algorithms, and verification systems built on data from existing verticals. A standalone vertical marketplace starts from zero.

- **Agent platforms integrate once.** An agent platform (ChatGPT, Claude, Gemini, vertical agents) integrates with H.164 once and gains resolution capability across all verticals. The alternative — integrating with a separate marketplace API for each service category — is untenable at scale.

The endgame is a single protocol through which any agent can resolve any real-world need against any human service provider, with trust, consent, and outcome verification built in. The phone number is the universal endpoint. H.164 is the universal resolution layer.

---

## Conclusion: The One-Liner

The schema wraps the number. The API resolves the need. The phone number completes the action. The log captures what happened. Everything else is distribution.

H.164 is the protocol that was always going to exist. The only question was when and who. AI agents have broken the locks. The phone number is ready to be wrapped. The protocol gap is undeniable. The timing is now.

We are building this in the open because it is the only way it works. We are building it first because the interaction log only compounds forward. We are building it to last because the phone number is the most durable identifier in human history — and it is about to become the most important one.

First contributors and seed investors shape the canonical index. Dustin@h164.io.

---

*H.164 is an open protocol licensed under Apache 2.0.*
*Specification repository: [github.com/h164-protocol](https://github.com/h164-protocol)*
*Contact: Dustin@h164.io*
