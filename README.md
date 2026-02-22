# H.164 Protocol

**The Human Endpoint Resolution Layer for the Agent Economy**

---

## The Problem

Every critical endpoint in the digital economy has been wrapped with structured metadata — websites with HTTP, payments with APIs, identity with OAuth. Every endpoint except one: **phone numbers**.

There are 400M+ business phone numbers globally. They remain raw, unstructured E.164 strings — opaque to the AI agents that increasingly need to resolve real-world tasks. For an AI agent, an unwrapped phone number is a dead end. No capabilities. No availability. No trust signal. No way to determine if calling this number will resolve the task at hand.

The agent economy has protocols for agent-to-agent communication (A2A), agent-to-commerce (ACP), and agent-to-tool interaction (MCP). What doesn't exist is the protocol for the last mile: **agent-to-human service provider**.

## What H.164 Is

H.164 is an open protocol that wraps phone numbers with structured metadata, making human service providers **discoverable**, **verifiable**, and **activatable** by AI agents.

The name references [E.164](https://en.wikipedia.org/wiki/E.164), the international telephone numbering standard. H.164 is the human-readable, human-resolved intelligence layer on top of it.

The protocol defines three layers:

### 1. The Schema (Open)
A standard structure for describing a human service endpoint — identity, capabilities, coverage area, availability, trust score, and activation methods. Published under Apache 2.0. Anyone can implement it.

### 2. The Index (Open)
A queryable directory of structured nodes. Free to query. The schema without an index is a whitepaper. The index without the schema is a proprietary database. Together, they are infrastructure.

### 3. The Interaction Log (Proprietary)
The canonical record of every agent-to-human query and outcome. What was needed, who was matched, what happened. This is where network intelligence accumulates and where the protocol becomes smarter with every interaction.

## Node Structure

A wrapped H.164 node transforms a raw phone number into a structured, queryable endpoint:

```json
{
  "endpoint": "+18135550142",
  "format": "E.164",
  "entity": {
    "name": "Mike's Plumbing",
    "type": "service_provider"
  },
  "capabilities": ["drain_clearing", "water_heater_install", "emergency_leak_repair"],
  "coverage": {
    "center": [27.9506, -82.4572],
    "radius_miles": 25
  },
  "availability": {
    "status": "inferred_open",
    "confidence": 0.72,
    "tier": 3,
    "source": "proxy_signals"
  },
  "trust": {
    "score": 0.87,
    "interactions": 142,
    "resolution_rate": 0.91
  },
  "response_sla": 180,
  "last_verified": "2026-02-22T14:30:00Z",
  "activation": {
    "method": "voice",
    "agent_compatible": true
  }
}
```

## Bidirectional Trust

H.164 implements trust scoring on **both sides** of every interaction. Service providers carry reputation scores. Requesting agents carry reputation scores. Providers can set thresholds — "only agents with trust > 0.7 can reach me."

This transforms the protocol from a discovery layer into a **consent and spam prevention infrastructure** for the agent economy. Without bidirectional trust, every phone number becomes an unfiltered endpoint for every agent on earth. H.164 is STIR/SHAKEN for AI agents.

## The Two-Way API

Every API call reads AND writes simultaneously. There is no read-only mode.

- A **query** writes demand signal (what was needed, where, when, urgency)
- A **match** writes which nodes were surfaced
- An **activation** writes response time, answer/no-answer
- An **outcome** writes resolution, pricing, satisfaction

The API is a membrane. Everything passing through it leaves a trace in both directions. You cannot consume the index without contributing to its intelligence.

## Resolution Flow

```
RESOLVE → Agent sends structured need
MATCH   → Index returns ranked nodes
ACTIVATE → Agent calls endpoint
OUTCOME → Resolution recorded
```

## Availability Tiering

Not all nodes carry the same signal strength. H.164 defines four tiers of availability confidence:

| Tier | Description | Signal Source | Confidence |
|------|-------------|---------------|------------|
| 1 | Live wrapped endpoint | AI receptionist, real-time integration | High |
| 2 | Verified active | Recent agent interaction, FSM integration | Medium-High |
| 3 | Inferred available | Proxy signals: hours, reviews, recent activity | Medium |
| 4 | Stale node | No recent signal | Low |

Even Tier 3 is infinitely better than the zero signal agents have today.

## HIPs — H.164 Improvement Proposals

The protocol evolves through **HIPs** (H.164 Improvement Proposals), following the established model of BIPs, EIPs, and CIPs.

| HIP | Title | Status |
|-----|-------|--------|
| HIP-0 | Protocol Vision & Manifesto | Draft |
| HIP-1 | Node Schema Specification | Draft |
| HIP-2 | Interaction Event Schema | Planned |
| HIP-3 | Bidirectional Trust Framework | Planned |
| HIP-4 | Availability Tiering Model | Planned |
| HIP-5 | Two-Way API Specification | Planned |

## Why Open

Closed infrastructure gets zero ecosystem adoption. FSM platforms won't feed data into a competitor's walled garden. Agent companies won't depend on infrastructure one entity controls. The schema must be open for the same reason HTTP had to be open for the web to work.

The schema is the commons. The index is the public good. The interaction log is the moat.

## The Gap

| Protocol | Layer | Status |
|----------|-------|--------|
| MCP (Anthropic) | Agent ↔ Tool | Exists |
| A2A (Google) | Agent ↔ Agent | Exists |
| ACP (Stripe/OpenAI) | Agent ↔ Commerce | Exists |
| **H.164** | **Agent ↔ Human Service Provider** | **This project** |

## Contributing

H.164 is open source under the Apache 2.0 license. We welcome contributions to the schema specification, reference implementations, and HIPs.

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

Apache 2.0 — see [LICENSE](LICENSE) for details.

---

**H.164 — Making every human service provider on earth discoverable, verifiable, and activatable by AI agents.**
