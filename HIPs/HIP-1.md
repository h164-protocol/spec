# HIP-1: Node Schema Specification

**H.164 Improvement Proposal 1**
**Title:** Service Node, Request Node, and Interaction Event Schema Definitions
**Author:** Dustin Goodwin
**Status:** Draft
**Created:** 2026-02-23
**Requires:** HIP-0

---

## Abstract

This document defines the core data schemas for the H.164 protocol: the **Service Node** (provider endpoints), the **Request Node** (consumer/requestor endpoints), and the **Interaction Event** chain (query → match → activation → outcome). It also formalizes the **Availability Tier** model and the **Trust Score** model referenced in HIP-0.

These schemas are the canonical definitions against which implementations MUST validate. All fields, types, constraints, and semantics are normative. Minimal viable node definitions are included to enable immediate bootstrapping with public data.

**Scope note:** This HIP defines data schemas (what nodes and events look like). It does not define the query API, write semantics, authentication model, or real-time subscription interface. Those are the subject of HIP-2 (API Specification, forthcoming).

---

## Table of Contents

1. [Conventions](#1-conventions)
2. [Service Node Schema](#2-service-node-schema)
3. [Request Node Schema](#3-request-node-schema)
4. [Interaction Event Schema](#4-interaction-event-schema)
5. [Availability Tier Definitions](#5-availability-tier-definitions)
6. [Trust Score Model](#6-trust-score-model)
7. [Example Nodes Across Verticals](#7-example-nodes-across-verticals)

---

## 1. Conventions

- All schemas are defined in [JSON Schema](https://json-schema.org/) (Draft 2020-12).
- Phone numbers MUST conform to [E.164](https://www.itu.int/rec/T-REC-E.164) format (e.g., `+14155551234`).
- Timestamps MUST be ISO 8601 in UTC (e.g., `2026-02-23T14:30:00Z`).
- Geographic coordinates use WGS 84 (decimal degrees).
- Trust scores are decimal values in the range `[0.0, 1.0]`.
- Confidence scores are decimal values in the range `[0.0, 1.0]`.
- All monetary values use [ISO 4217](https://www.iso.org/iso-4217-currency-codes.html) currency codes. Amounts are expressed in standard currency units (e.g., 450 = $450 USD, not cents) unless otherwise noted.
- Any `enum` field marked "Extensible via HIP process" may be expanded in future HIPs without a breaking schema version change.
- The key words "MUST", "MUST NOT", "SHOULD", "OPTIONAL" follow [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## 2. Service Node Schema

A **Service Node** represents a wrapped service provider endpoint — a phone number enriched with structured metadata that enables agent-driven discovery, evaluation, and activation.

### 2.1 Top-Level Structure

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schema.h164.io/service-node/v1",
  "title": "H.164 Service Node",
  "type": "object",
  "required": ["version", "endpoint", "entity", "capabilities", "coverage", "availability", "trust", "activation", "listing_basis", "last_verified"],
  "properties": {
    "version": { "type": "string", "const": "1.0-draft", "description": "H.164 schema version for forward compatibility." },
    "endpoint": { "$ref": "#/$defs/E164Endpoint" },
    "entity": { "$ref": "#/$defs/ServiceEntity" },
    "capabilities": { "$ref": "#/$defs/Capabilities" },
    "coverage": { "$ref": "#/$defs/Coverage" },
    "availability": { "$ref": "#/$defs/Availability" },
    "trust": { "$ref": "#/$defs/ServiceTrustScore" },
    "response_sla": { "$ref": "#/$defs/ResponseSLA" },
    "activation": { "$ref": "#/$defs/Activation" },
    "verification": { "$ref": "#/$defs/ServiceVerification" },
    "listing_basis": {
      "type": "string",
      "enum": ["public_listing", "provider_opted_in", "provider_claimed", "provider_unaware"],
      "description": "Basis on which this node was created. Tracks consent status for GDPR/privacy compliance."
    },
    "last_verified": { "type": "string", "format": "date-time" },
    "metadata": { "type": "object", "additionalProperties": true }
  }
}
```

### 2.2 Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | ✅ | Schema version identifier. MUST be `"1.0-draft"` for this version. |
| `endpoint` | E164Endpoint | ✅ | The E.164 phone number that is being wrapped. The root identifier. |
| `entity` | ServiceEntity | ✅ | Identity information about the service provider. |
| `capabilities` | Capabilities | ✅ | What this provider can do. Structured taxonomy of service categories. |
| `coverage` | Coverage | ✅ | Where this provider operates. Geospatial boundaries. |
| `availability` | Availability | ✅ | When and whether this provider is reachable. Tiered confidence model. |
| `trust` | ServiceTrustScore | ✅ | Composite trust score derived from interaction history and verification. |
| `response_sla` | ResponseSLA | No | Expected response time when activated. |
| `activation` | Activation | ✅ | How an agent reaches this node. Supported contact methods. |
| `verification` | ServiceVerification | No | Verified credentials: licenses, insurance, certifications. |
| `listing_basis` | string | ✅ | How this node was created: `public_listing` (seeded from published directory), `provider_opted_in` (provider registered directly), `provider_claimed` (provider verified and claimed an existing node), `provider_unaware` (seeded but not yet claimed). Tracks consent for GDPR/privacy compliance. |
| `last_verified` | ISO 8601 datetime | ✅ | Timestamp of the most recent signal confirming this node's data. |
| `metadata` | Object | No | Freeform extension point for vertical-specific data. |

### 2.3 Minimal Viable Node (Seed Schema)

The full service node schema above represents the target state. At bootstrap, most nodes will be seeded from public data sources (business directories, state license databases, review platforms) and will carry only a subset of fields. The **Minimal Viable Node** defines the lowest bar for index inclusion — the six fields required to make a node queryable and activatable by an agent.

**Required fields for index inclusion:**

| Field | Example | Source |
|-------|---------|--------|
| `endpoint` | `"+14155551234"` | Public listing |
| `entity.name` | `"Mike's Plumbing"` | Public listing |
| `capabilities.primary.category` | `"plumbing"` | Inferred from listing category |
| `coverage.zones[0]` | `{ "type": "postal_codes", "postal_codes": ["94110"] }` | Listing address |
| `availability.tier` | `4` | Default for unverified seed nodes |
| `activation.methods[0]` | `{ "type": "voice", "endpoint": "+14155551234" }` | Public listing |
| `listing_basis` | `"public_listing"` | How node was created |

A seed node enters at **Tier 4** with a cold-start trust score derived solely from credential signals (see Section 6.5). The node becomes useful immediately — an agent can discover it, evaluate it against alternatives, and attempt activation. Every successful interaction enriches the node, adding trust signals and triggering tier upgrades.

This is the bootstrap path: seed 50,000+ nodes from public data at Tier 3–4, prove resolution utility, then enrich toward Tier 1–2 through real interactions and provider integrations.

```json
{
  "version": "1.0-draft",
  "endpoint": "+14155551234",
  "entity": { "name": "Mike's Plumbing", "type": "individual" },
  "capabilities": { "primary": { "category": "plumbing", "subcategory": ["emergency_repair", "drain_cleaning"] } },
  "coverage": { "zones": [{ "type": "postal_codes", "postal_codes": ["94110", "94112"] }], "mobile": true },
  "availability": { "tier": 4, "confidence": 0.15, "signal_sources": ["directory_listing", "carrier_active"] },
  "trust": { "composite": 0.20, "interaction_count": 0, "verification_score": 0.20, "decay_rate": 0.005, "last_updated": "2026-02-23T00:00:00Z" },
  "activation": { "methods": [{ "type": "voice", "endpoint": "+14155551234", "priority": 1, "ai_receptionist": false }] },
  "listing_basis": "public_listing",
  "last_verified": "2026-02-23T00:00:00Z"
}
```

Implementations SHOULD accept nodes with only the minimal fields above. Implementations MUST NOT reject a node solely because optional fields (`verification`, `response_sla`, `metadata`) are absent.

### 2.4 Component Definitions

#### E164Endpoint

```json
{
  "$defs": {
    "E164Endpoint": {
      "type": "string",
      "pattern": "^\\+[1-9]\\d{1,14}$",
      "description": "Phone number in E.164 format. Globally unique. The root identifier of this node."
    }
  }
}
```

#### ServiceEntity

```json
{
  "ServiceEntity": {
    "type": "object",
    "required": ["name", "type"],
    "properties": {
      "name": {
        "type": "string",
        "description": "Display name of the service provider."
      },
      "type": {
        "type": "string",
        "enum": ["individual", "business", "franchise_location", "government_agency"],
        "description": "Entity classification."
      },
      "legal_name": {
        "type": "string",
        "description": "Registered legal name, if different from display name."
      },
      "identifiers": {
        "type": "object",
        "properties": {
          "ein": { "type": "string", "description": "Employer Identification Number (US)." },
          "duns": { "type": "string", "description": "D-U-N-S Number." },
          "npi": { "type": "string", "description": "National Provider Identifier (US healthcare)." },
          "state_license": { "type": "string", "description": "State-issued license number." },
          "bar_number": { "type": "string", "description": "Bar association number (legal)." }
        },
        "additionalProperties": true,
        "description": "External identifiers. Extensible by vertical."
      }
    }
  }
}
```

#### Capabilities

```json
{
  "Capabilities": {
    "type": "object",
    "required": ["primary"],
    "properties": {
      "primary": {
        "type": "object",
        "required": ["category", "subcategory"],
        "properties": {
          "category": {
            "type": "string",
            "description": "Top-level service category. Controlled vocabulary (e.g., 'plumbing', 'dental', 'legal', 'towing', 'pet_services')."
          },
          "subcategory": {
            "type": "array",
            "items": { "type": "string" },
            "description": "Specific services within the category (e.g., ['emergency_repair', 'drain_cleaning', 'water_heater'])."
          }
        }
      },
      "secondary": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["category"],
          "properties": {
            "category": { "type": "string" },
            "subcategory": { "type": "array", "items": { "type": "string" } }
          }
        },
        "description": "Additional service categories this provider handles."
      },
      "languages": {
        "type": "array",
        "items": { "type": "string" },
        "description": "ISO 639-1 language codes (e.g., ['en', 'es'])."
      },
      "specializations": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Freeform specialization tags (e.g., ['tankless_water_heaters', 'commercial_plumbing'])."
      }
    }
  }
}
```

#### Coverage

```json
{
  "Coverage": {
    "type": "object",
    "required": ["zones"],
    "properties": {
      "zones": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["type"],
          "properties": {
            "type": {
              "type": "string",
              "enum": ["radius", "polygon", "postal_codes", "region"],
              "description": "How the coverage zone is defined."
            },
            "center": {
              "type": "object",
              "properties": {
                "lat": { "type": "number", "minimum": -90, "maximum": 90 },
                "lng": { "type": "number", "minimum": -180, "maximum": 180 }
              },
              "description": "Center point for radius-based coverage."
            },
            "radius_km": {
              "type": "number",
              "minimum": 0,
              "description": "Service radius in kilometers from center point."
            },
            "polygon": {
              "type": "array",
              "items": {
                "type": "object",
                "properties": {
                  "lat": { "type": "number" },
                  "lng": { "type": "number" }
                }
              },
              "description": "Ordered list of coordinates defining coverage polygon."
            },
            "postal_codes": {
              "type": "array",
              "items": { "type": "string" },
              "description": "List of postal/ZIP codes served."
            },
            "region": {
              "type": "string",
              "description": "Named region (e.g., 'Manhattan', 'Greater London')."
            },
            "label": {
              "type": "string",
              "description": "Human-readable label for this zone (e.g., 'Primary service area')."
            }
          }
        },
        "minItems": 1,
        "description": "One or more geographic zones this provider covers."
      },
      "mobile": {
        "type": "boolean",
        "default": false,
        "description": "Whether this provider travels to the customer (true) or operates from a fixed location (false)."
      },
      "headquarters": {
        "type": "object",
        "properties": {
          "lat": { "type": "number" },
          "lng": { "type": "number" },
          "address": { "type": "string" }
        },
        "description": "Primary business location, if applicable."
      }
    }
  }
}
```

#### Availability

```json
{
  "Availability": {
    "type": "object",
    "required": ["tier", "confidence"],
    "properties": {
      "tier": {
        "type": "integer",
        "enum": [0, 1, 2, 3, 4],
        "description": "Availability tier (0=suspended, 1=live wrapped, 2=verified active, 3=inferred, 4=stale). See Section 5."
      },
      "confidence": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Confidence that this node is currently available and reachable."
      },
      "signal_sources": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "live_endpoint",
            "recent_interaction",
            "fsm_integration",
            "ai_receptionist",
            "published_hours",
            "review_activity",
            "website_active",
            "directory_listing",
            "carrier_active"
          ]
        },
        "description": "Signals contributing to the current availability assessment."
      },
      "hours": {
        "type": "object",
        "properties": {
          "timezone": { "type": "string", "description": "IANA timezone (e.g., 'America/New_York')." },
          "regular": {
            "type": "object",
            "properties": {
              "mon": { "$ref": "#/$defs/DayHours" },
              "tue": { "$ref": "#/$defs/DayHours" },
              "wed": { "$ref": "#/$defs/DayHours" },
              "thu": { "$ref": "#/$defs/DayHours" },
              "fri": { "$ref": "#/$defs/DayHours" },
              "sat": { "$ref": "#/$defs/DayHours" },
              "sun": { "$ref": "#/$defs/DayHours" }
            }
          },
          "emergency_available": {
            "type": "boolean",
            "default": false,
            "description": "Whether this provider accepts emergency/after-hours requests."
          }
        },
        "description": "Published operating hours."
      },
      "last_signal": {
        "type": "string",
        "format": "date-time",
        "description": "Timestamp of the most recent availability signal."
      },
      "next_refresh": {
        "type": "string",
        "format": "date-time",
        "description": "When the next availability check is scheduled."
      }
    }
  }
}
```

#### DayHours (helper)

```json
{
  "DayHours": {
    "type": "object",
    "properties": {
      "open": { "type": "string", "pattern": "^\\d{2}:\\d{2}$", "description": "Opening time in HH:MM (24hr)." },
      "close": { "type": "string", "pattern": "^\\d{2}:\\d{2}$", "description": "Closing time in HH:MM (24hr)." },
      "closed": { "type": "boolean", "default": false }
    }
  }
}
```

#### ServiceTrustScore

```json
{
  "ServiceTrustScore": {
    "type": "object",
    "required": ["composite"],
    "properties": {
      "composite": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Overall trust score. Weighted aggregate of all trust dimensions."
      },
      "interaction_count": {
        "type": "integer",
        "minimum": 0,
        "description": "Total number of completed interactions through H.164."
      },
      "resolution_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of activations that resulted in confirmed service delivery."
      },
      "response_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of activations answered (not missed/ignored)."
      },
      "dispute_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of interactions resulting in disputes."
      },
      "repeat_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of requestors who activated this node more than once."
      },
      "price_accuracy": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "How closely final charges match initial quotes."
      },
      "verification_score": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Score derived from verified credentials (licenses, insurance, certifications)."
      },
      "decay_rate": {
        "type": "number",
        "description": "Daily decay factor applied when no new signals are received. See Section 6."
      },
      "last_updated": {
        "type": "string",
        "format": "date-time",
        "description": "When the trust score was last recalculated."
      },
      "onchain_attestation": {
        "type": "object",
        "properties": {
          "chain": { "type": "string", "enum": ["ethereum", "base", "arbitrum"], "description": "Network where attestation is anchored. Extensible via HIP process." },
          "contract": { "type": "string", "description": "Attestation contract address." },
          "token_id": { "type": "string", "description": "Token or attestation ID." },
          "attested_at": { "type": "string", "format": "date-time" }
        },
        "description": "OPTIONAL. On-chain attestation of trust score for cryptographic verifiability."
      }
    }
  }
}
```

#### ResponseSLA

```json
{
  "ResponseSLA": {
    "type": "object",
    "properties": {
      "expected_seconds": {
        "type": "integer",
        "description": "Expected time in seconds for this node to respond to an activation."
      },
      "p50_seconds": {
        "type": "integer",
        "description": "Median observed response time."
      },
      "p95_seconds": {
        "type": "integer",
        "description": "95th percentile observed response time."
      },
      "sample_size": {
        "type": "integer",
        "description": "Number of activations in the SLA calculation window."
      }
    }
  }
}
```

#### Activation

```json
{
  "Activation": {
    "type": "object",
    "required": ["methods"],
    "properties": {
      "methods": {
        "type": "array",
        "items": {
          "type": "object",
          "required": ["type"],
          "properties": {
            "type": {
              "type": "string",
              "enum": ["voice", "sms", "dispatch_api", "webhook", "email"],
              "description": "Activation method. Extensible via HIP process."
            },
            "endpoint": {
              "type": "string",
              "description": "The endpoint for this method. For voice/sms, this is the E.164 number (may differ from the primary node endpoint). For dispatch_api/webhook, this is the URL."
            },
            "priority": {
              "type": "integer",
              "minimum": 1,
              "description": "Preferred order. 1 = most preferred."
            },
            "ai_receptionist": {
              "type": "boolean",
              "default": false,
              "description": "Whether this endpoint is handled by an AI receptionist capable of structured interaction."
            },
            "capabilities": {
              "type": "array",
              "items": {
                "type": "string",
                "enum": ["schedule", "dispatch", "quote", "triage", "transfer_to_human"]
              },
              "description": "What can be accomplished through this activation method."
            }
          }
        },
        "minItems": 1
      },
      "preferred_channel": {
        "type": "string",
        "enum": ["voice", "sms", "dispatch_api", "webhook", "email"],
        "description": "The provider's preferred activation channel."
      }
    }
  }
}
```

#### ServiceVerification

```json
{
  "ServiceVerification": {
    "type": "object",
    "properties": {
      "licenses": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "type": { "type": "string", "description": "License type (e.g., 'plumbing_contractor', 'medical_license')." },
            "number": { "type": "string" },
            "issuer": { "type": "string", "description": "Issuing authority (e.g., 'State of California CSLB')." },
            "jurisdiction": { "type": "string" },
            "status": { "type": "string", "enum": ["active", "expired", "suspended", "unverified"] },
            "verified_at": { "type": "string", "format": "date-time" },
            "expires_at": { "type": "string", "format": "date-time" }
          }
        }
      },
      "insurance": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "type": { "type": "string", "description": "Insurance type (e.g., 'general_liability', 'malpractice')." },
            "carrier": { "type": "string" },
            "coverage_amount": { "type": "number" },
            "coverage_currency": { "type": "string", "default": "USD" },
            "status": { "type": "string", "enum": ["active", "expired", "unverified"] },
            "verified_at": { "type": "string", "format": "date-time" },
            "expires_at": { "type": "string", "format": "date-time" }
          }
        }
      },
      "certifications": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "name": { "type": "string", "description": "Certification name." },
            "issuer": { "type": "string" },
            "verified_at": { "type": "string", "format": "date-time" },
            "expires_at": { "type": "string", "format": "date-time" }
          }
        }
      },
      "carrier_verified": {
        "type": "boolean",
        "description": "Whether the phone number has been verified as active with the carrier."
      },
      "identity_verified": {
        "type": "boolean",
        "description": "Whether the entity behind this number has been identity-verified."
      }
    }
  }
}
```

---

## 3. Request Node Schema

A **Request Node** represents a wrapped consumer/requestor endpoint — a phone number enriched with structured metadata that enables trust evaluation, consent enforcement, and interaction history tracking.

The Request Node is the 8-billion-person side of H.164. It wraps the human whose agent acts on their behalf.

### 3.1 Top-Level Structure

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schema.h164.io/request-node/v1",
  "title": "H.164 Request Node",
  "type": "object",
  "required": ["version", "endpoint", "entity", "agent_identity", "trust", "consent_thresholds"],
  "properties": {
    "version": { "type": "string", "const": "1.0-draft", "description": "H.164 schema version for forward compatibility." },
    "endpoint": { "$ref": "#/$defs/E164Endpoint" },
    "entity": { "$ref": "#/$defs/RequestEntity" },
    "agent_identity": { "$ref": "#/$defs/AgentIdentity" },
    "trust": { "$ref": "#/$defs/RequestTrustScore" },
    "preferences": { "$ref": "#/$defs/RequestPreferences" },
    "consent_thresholds": { "$ref": "#/$defs/ConsentThresholds" },
    "reachability": { "$ref": "#/$defs/Reachability" },
    "interaction_history": { "$ref": "#/$defs/RequestInteractionHistory" },
    "verification": { "$ref": "#/$defs/RequestVerification" },
    "registration_basis": {
      "type": "string",
      "enum": ["user_opted_in", "agent_registered", "imported"],
      "description": "How this request node was created. Tracks consent for privacy compliance."
    },
    "metadata": { "type": "object", "additionalProperties": true }
  }
}
```

### 3.2 Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | ✅ | Schema version identifier. MUST be `"1.0-draft"` for this version. |
| `endpoint` | E164Endpoint | ✅ | The E.164 phone number of the requestor. |
| `entity` | RequestEntity | ✅ | Identity of the human (or organization) behind the request. |
| `agent_identity` | AgentIdentity | ✅ | Which agent platform is acting for this human. |
| `trust` | RequestTrustScore | ✅ | Composite trust score based on interaction history as a requestor. |
| `preferences` | RequestPreferences | No | Service preferences, budget, quality expectations. |
| `consent_thresholds` | ConsentThresholds | ✅ | Rules governing who and what can reach this endpoint. |
| `reachability` | Reachability | No | How and when this requestor can be contacted. |
| `interaction_history` | RequestInteractionHistory | No | Summary statistics of past interactions. |
| `verification` | RequestVerification | No | Verification signals: carrier, payment, identity. |
| `registration_basis` | string | No | How this node was created: `user_opted_in` (user explicitly registered), `agent_registered` (agent platform created on user's behalf with permission), `imported` (migrated from external system). |
| `metadata` | Object | No | Freeform extension point. |

### 3.3 Component Definitions

#### RequestEntity

```json
{
  "RequestEntity": {
    "type": "object",
    "required": ["type"],
    "properties": {
      "type": {
        "type": "string",
        "enum": ["individual", "organization"],
        "description": "Whether this is a person or an organization making requests."
      },
      "name": {
        "type": "string",
        "description": "Display name. OPTIONAL — privacy-preserving by default."
      },
      "location": {
        "type": "object",
        "properties": {
          "lat": { "type": "number" },
          "lng": { "type": "number" },
          "locality": { "type": "string", "description": "City or neighborhood (coarse location for matching)." },
          "region": { "type": "string" },
          "country": { "type": "string" }
        },
        "description": "General location for service matching. NOT precise address."
      }
    }
  }
}
```

#### AgentIdentity

```json
{
  "AgentIdentity": {
    "type": "object",
    "required": ["platform"],
    "properties": {
      "platform": {
        "type": "string",
        "description": "The agent platform acting on behalf of this human (e.g., 'openai', 'anthropic', 'google', 'custom')."
      },
      "agent_id": {
        "type": "string",
        "description": "Unique identifier for the specific agent instance."
      },
      "agent_version": {
        "type": "string",
        "description": "Version of the agent platform."
      },
      "authorization": {
        "type": "object",
        "properties": {
          "scope": {
            "type": "array",
            "items": { "type": "string" },
            "description": "What actions this agent is authorized to perform (e.g., ['query', 'activate', 'schedule', 'pay'])."
          },
          "granted_at": { "type": "string", "format": "date-time" },
          "expires_at": { "type": "string", "format": "date-time" }
        },
        "description": "Authorization scope the human has granted to this agent."
      }
    }
  }
}
```

#### RequestTrustScore

```json
{
  "RequestTrustScore": {
    "type": "object",
    "required": ["composite"],
    "properties": {
      "composite": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Overall requestor trust score."
      },
      "interaction_count": {
        "type": "integer",
        "minimum": 0,
        "description": "Total interactions initiated through H.164."
      },
      "completion_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of initiated requests that resulted in completed service."
      },
      "payment_reliability": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of interactions with timely, undisputed payment."
      },
      "dispute_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of interactions resulting in disputes initiated by this requestor."
      },
      "no_show_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Percentage of dispatched services where requestor was unavailable."
      },
      "outcome_reporting_rate": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "How often this requestor's agent reports outcomes after service completion."
      },
      "decay_rate": { "type": "number" },
      "last_updated": { "type": "string", "format": "date-time" },
      "onchain_attestation": {
        "type": "object",
        "properties": {
          "chain": { "type": "string", "enum": ["ethereum", "base", "arbitrum"], "description": "Network where attestation is anchored. Extensible via HIP process." },
          "contract": { "type": "string", "description": "Attestation contract address." },
          "token_id": { "type": "string", "description": "Token or attestation ID." },
          "attested_at": { "type": "string", "format": "date-time" }
        },
        "description": "OPTIONAL. On-chain attestation of requestor trust score."
      }
    }
  }
}
```

#### RequestPreferences

```json
{
  "RequestPreferences": {
    "type": "object",
    "properties": {
      "service_categories": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Service categories this requestor is likely to need."
      },
      "location_preference": {
        "type": "object",
        "properties": {
          "max_radius_km": { "type": "number" },
          "preferred_locality": { "type": "string" }
        }
      },
      "budget_ranges": {
        "type": "object",
        "additionalProperties": {
          "type": "object",
          "properties": {
            "min": { "type": "number" },
            "max": { "type": "number" },
            "currency": { "type": "string", "default": "USD" }
          }
        },
        "description": "Budget ranges keyed by service category."
      },
      "quality_threshold": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Minimum provider trust score this requestor is willing to accept."
      },
      "language_preference": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Preferred languages (ISO 639-1)."
      }
    }
  }
}
```

#### ConsentThresholds

```json
{
  "ConsentThresholds": {
    "type": "object",
    "required": ["inbound_policy"],
    "properties": {
      "inbound_policy": {
        "type": "string",
        "enum": ["open", "trust_gated", "allowlist_only", "closed"],
        "description": "Overall policy for inbound contact. 'open' = any provider can reach this node. 'trust_gated' = only providers above minimum score. 'allowlist_only' = only pre-approved providers. 'closed' = no inbound contact."
      },
      "min_provider_trust": {
        "type": "number",
        "minimum": 0.0,
        "maximum": 1.0,
        "description": "Minimum provider trust score required for inbound contact (when policy is 'trust_gated')."
      },
      "allowed_categories": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Service categories from which this node accepts inbound contact."
      },
      "blocked_categories": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Service categories from which inbound contact is always blocked."
      },
      "allowlist": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Specific service node endpoints (E.164) pre-approved for contact."
      },
      "blocklist": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Specific service node endpoints (E.164) permanently blocked."
      },
      "time_restrictions": {
        "type": "object",
        "properties": {
          "timezone": { "type": "string" },
          "allowed_hours": {
            "type": "object",
            "properties": {
              "start": { "type": "string", "pattern": "^\\d{2}:\\d{2}$" },
              "end": { "type": "string", "pattern": "^\\d{2}:\\d{2}$" }
            }
          },
          "emergency_override": {
            "type": "boolean",
            "default": false,
            "description": "Whether emergency-flagged contacts can bypass time restrictions."
          }
        }
      },
      "max_daily_inbound": {
        "type": "integer",
        "default": 50,
        "description": "Maximum inbound agent queries per day. Prevents even trusted agents from overwhelming providers. Set to 0 for unlimited."
      }
    }
  }
}
```

#### Reachability

```json
{
  "Reachability": {
    "type": "object",
    "properties": {
      "preferred_channel": {
        "type": "string",
        "enum": ["voice", "sms", "push_notification", "in_app", "email"],
        "description": "How this requestor prefers to be contacted."
      },
      "channels": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "type": { "type": "string", "enum": ["voice", "sms", "push_notification", "in_app", "email"] },
            "endpoint": { "type": "string" },
            "active": { "type": "boolean" }
          }
        }
      },
      "do_not_disturb": {
        "type": "boolean",
        "default": false,
        "description": "Global DND flag. When true, no contact except emergencies."
      }
    }
  }
}
```

#### RequestInteractionHistory

```json
{
  "RequestInteractionHistory": {
    "type": "object",
    "properties": {
      "total_requests": { "type": "integer", "minimum": 0 },
      "total_completions": { "type": "integer", "minimum": 0 },
      "total_disputes": { "type": "integer", "minimum": 0 },
      "total_no_shows": { "type": "integer", "minimum": 0 },
      "categories_used": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Service categories this requestor has previously engaged."
      },
      "first_interaction": { "type": "string", "format": "date-time" },
      "last_interaction": { "type": "string", "format": "date-time" }
    }
  }
}
```

#### RequestVerification

```json
{
  "RequestVerification": {
    "type": "object",
    "properties": {
      "carrier_verified": {
        "type": "boolean",
        "description": "Phone number confirmed active with carrier."
      },
      "payment_method_on_file": {
        "type": "boolean",
        "description": "Whether a verified payment method is associated with this node."
      },
      "identity_confidence": {
        "type": "string",
        "enum": ["none", "carrier_only", "basic_kyc", "full_kyc"],
        "description": "Level of identity verification completed."
      },
      "verified_at": { "type": "string", "format": "date-time" }
    }
  }
}
```

### 3.4 Minimal Viable Request Node (Seed Schema)

At bootstrap, request nodes need only four fields to participate in the protocol:

| Field | Example | Source |
|-------|---------|--------|
| `endpoint` | `"+14155550101"` | User's phone number |
| `agent_identity.platform` | `"anthropic"` | Agent platform registering the node |
| `consent_thresholds.inbound_policy` | `"trust_gated"` | User preference (default: trust_gated) |
| `trust.composite` | `0.30` | Cold-start score from verification signals |

All other fields — preferences, reachability, interaction history — enrich over time as the requestor's agent interacts with the index. The minimal request node establishes identity, consent, and baseline trust. Everything else compounds through use.

```json
{
  "version": "1.0-draft",
  "endpoint": "+14155550101",
  "entity": { "type": "individual" },
  "agent_identity": { "platform": "anthropic", "agent_id": "claude-agent-abc123", "authorization": { "scope": ["query", "activate"] } },
  "trust": { "composite": 0.30, "interaction_count": 0, "decay_rate": 0.005, "last_updated": "2026-02-23T00:00:00Z" },
  "consent_thresholds": { "inbound_policy": "trust_gated", "min_provider_trust": 0.40 }
}
```

Implementations SHOULD accept request nodes with only the minimal fields above. Implementations MUST NOT reject a request node solely because optional fields (`preferences`, `reachability`, `interaction_history`) are absent.

---

## 4. Interaction Event Schema

Every interaction between a request node and the H.164 index produces a chain of events. These events form the **Interaction Log** — the protocol's core data asset.

### 4.1 Event Chain

An interaction follows a four-stage event chain:

```
query_event → match_event → activation_event → outcome_event
```

All events in a single interaction share a `session_id`. Each event has a unique `event_id` and a `timestamp`.

### 4.2 Common Event Fields

Every event includes:

```json
{
  "EventBase": {
    "type": "object",
    "required": ["event_id", "session_id", "event_type", "timestamp"],
    "properties": {
      "event_id": {
        "type": "string",
        "format": "uuid",
        "description": "Unique identifier for this event."
      },
      "session_id": {
        "type": "string",
        "format": "uuid",
        "description": "Links all events in a single interaction resolution chain."
      },
      "event_type": {
        "type": "string",
        "enum": ["query", "match", "activation", "outcome"]
      },
      "timestamp": {
        "type": "string",
        "format": "date-time"
      },
      "request_node": {
        "type": "string",
        "description": "E.164 endpoint of the requesting node."
      },
      "agent_identity": {
        "type": "object",
        "properties": {
          "platform": { "type": "string" },
          "agent_id": { "type": "string" }
        }
      }
    }
  }
}
```

### 4.3 Query Event

Emitted when an agent queries the index to resolve a need.

```json
{
  "QueryEvent": {
    "allOf": [{ "$ref": "#/$defs/EventBase" }],
    "properties": {
      "event_type": { "const": "query" },
      "query": {
        "type": "object",
        "required": ["need", "location"],
        "properties": {
          "need": {
            "type": "object",
            "properties": {
              "category": { "type": "string" },
              "subcategory": { "type": "string" },
              "description": { "type": "string", "description": "Freeform description of the need." },
              "urgency": {
                "type": "string",
                "enum": ["emergency", "same_day", "scheduled", "flexible"],
                "description": "How urgently the service is needed."
              }
            }
          },
          "location": {
            "type": "object",
            "properties": {
              "lat": { "type": "number" },
              "lng": { "type": "number" },
              "radius_km": { "type": "number" }
            }
          },
          "constraints": {
            "type": "object",
            "properties": {
              "min_trust_score": { "type": "number" },
              "max_response_seconds": { "type": "integer" },
              "required_verification": {
                "type": "array",
                "items": { "type": "string", "enum": ["licensed", "insured", "certified", "identity_verified"] }
              },
              "budget_max": { "type": "number" },
              "languages": { "type": "array", "items": { "type": "string" } }
            }
          },
          "max_results": {
            "type": "integer",
            "default": 5,
            "description": "Maximum number of matching nodes to return."
          }
        }
      }
    }
  }
}
```

### 4.4 Match Event

Emitted when the index returns ranked service nodes.

```json
{
  "MatchEvent": {
    "allOf": [{ "$ref": "#/$defs/EventBase" }],
    "properties": {
      "event_type": { "const": "match" },
      "results": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "service_node_endpoint": { "type": "string" },
            "rank": { "type": "integer" },
            "match_score": { "type": "number", "minimum": 0.0, "maximum": 1.0 },
            "trust_score_at_match": { "type": "number" },
            "availability_confidence_at_match": { "type": "number" },
            "distance_km": { "type": "number" },
            "estimated_response_seconds": { "type": "integer" }
          }
        }
      },
      "total_candidates": {
        "type": "integer",
        "description": "Total nodes evaluated before filtering."
      },
      "filters_applied": {
        "type": "array",
        "items": { "type": "string" },
        "description": "Which constraint filters reduced the candidate set."
      }
    }
  }
}
```

### 4.5 Activation Event

Emitted when an agent activates (contacts) a matched service node.

```json
{
  "ActivationEvent": {
    "allOf": [{ "$ref": "#/$defs/EventBase" }],
    "properties": {
      "event_type": { "const": "activation" },
      "service_node_endpoint": {
        "type": "string",
        "description": "E.164 endpoint of the activated service node."
      },
      "activation_method": {
        "type": "string",
        "enum": ["voice", "sms", "dispatch_api", "webhook", "email"]
      },
      "activation_result": {
        "type": "string",
        "enum": [
          "answered_human",
          "answered_ai",
          "voicemail",
          "no_answer",
          "busy",
          "disconnected",
          "api_success",
          "api_error",
          "webhook_delivered",
          "webhook_failed"
        ]
      },
      "response_time_seconds": {
        "type": "number",
        "description": "Seconds between activation attempt and response."
      },
      "dispatch_confirmed": {
        "type": "boolean",
        "description": "Whether service dispatch was confirmed during this activation."
      },
      "quoted_price": {
        "type": "number",
        "description": "Price quoted during activation, if applicable."
      },
      "quoted_currency": {
        "type": "string",
        "default": "USD"
      },
      "estimated_arrival_minutes": {
        "type": "integer",
        "description": "Estimated minutes to arrival, if applicable (dispatch services)."
      },
      "scheduled_time": {
        "type": "string",
        "format": "date-time",
        "description": "Scheduled service time, if applicable (appointment services)."
      }
    }
  }
}
```

### 4.6 Outcome Event

Emitted after service completion (or failure). This is the signal that feeds back into trust scores.

```json
{
  "OutcomeEvent": {
    "allOf": [{ "$ref": "#/$defs/EventBase" }],
    "properties": {
      "event_type": { "const": "outcome" },
      "service_node_endpoint": { "type": "string" },
      "outcome_hash": {
        "type": "string",
        "description": "SHA-256 hash of the canonical outcome payload. Cryptographic commitment enabling future zero-knowledge outcome proofs and on-chain attestation."
      },
      "outcome_status": {
        "type": "string",
        "enum": [
          "completed",
          "completed_with_issues",
          "no_show_provider",
          "no_show_requestor",
          "cancelled_by_provider",
          "cancelled_by_requestor",
          "disputed",
          "unknown"
        ]
      },
      "verification_signals": {
        "type": "object",
        "properties": {
          "transaction_confirmed": {
            "type": "boolean",
            "description": "Payment was processed."
          },
          "final_price": {
            "type": "number",
            "description": "Actual amount charged."
          },
          "price_currency": { "type": "string", "default": "USD" },
          "price_variance": {
            "type": "number",
            "description": "Percentage difference between quoted and final price."
          },
          "temporal_chain_valid": {
            "type": "boolean",
            "description": "Event timestamps form a plausible sequence."
          },
          "counter_call_consistent": {
            "type": "boolean",
            "description": "Provider and requestor independently reported consistent outcomes."
          },
          "requestor_reported": {
            "type": "boolean",
            "description": "Requestor's agent submitted an outcome report."
          },
          "provider_reported": {
            "type": "boolean",
            "description": "Provider's node submitted an outcome report."
          }
        }
      },
      "satisfaction_signal": {
        "type": "object",
        "properties": {
          "score": {
            "type": "number",
            "minimum": 0.0,
            "maximum": 1.0,
            "description": "Normalized satisfaction indicator."
          },
          "source": {
            "type": "string",
            "enum": ["requestor_explicit", "agent_inferred", "behavioral"],
            "description": "How the satisfaction signal was obtained."
          }
        }
      },
      "dispute": {
        "type": "object",
        "properties": {
          "filed": { "type": "boolean" },
          "filed_by": { "type": "string", "enum": ["requestor", "provider"] },
          "reason": { "type": "string" },
          "resolved": { "type": "boolean" },
          "resolution": { "type": "string" }
        }
      },
      "completion_timestamp": {
        "type": "string",
        "format": "date-time",
        "description": "When the service was completed."
      }
    }
  }
}
```

---

## 5. Availability Tier Definitions

The availability tier system classifies service nodes by signal quality, determining how much confidence an agent can place in the node's current reachability and responsiveness.

### 5.1 Tier Definitions

| Tier | Name | Description | Confidence Range | % of Index | Refresh Rate |
|------|------|-------------|------------------|------------|--------------|
| **0** | Suspended | Number confirmed disconnected, fraudulent, or flagged for abuse. Auto-filtered from all query results. Nodes at Tier 0 are never returned in match events. Requires manual review or fresh verification signal to re-enter the index. | 0.00 | Variable | On-trigger only |
| **1** | Live Wrapped | Real-time integration. AI receptionist, live API, or FSM platform connected. Full structured signal. The node actively participates in the protocol. | 0.90 – 1.00 | < 5% | Continuous / Real-time |
| **2** | Verified Active | Recent agent interaction or periodic verification confirms this node is active and responsive. Signal is fresh but not real-time. | 0.70 – 0.89 | 5 – 15% | Every 24 – 72 hours |
| **3** | Inferred Available | No direct integration or recent interaction. Availability is inferred from proxy signals: published hours, recent review activity, active website, directory listings, carrier status. Probabilistic. | 0.30 – 0.69 | 80%+ | Weekly / On-demand |
| **4** | Stale Node | No recent signal of any kind. The phone number may still be active, but the index has no confidence in current availability, capabilities, or responsiveness. Listed but effectively unverified. | 0.00 – 0.29 | Variable | Monthly / On-trigger |

### 5.2 Tier Transition Rules

**Upward transitions** (lower → higher tier):

| From | To | Trigger |
|------|----|---------|
| 4 → 3 | Stale → Inferred | New proxy signal detected (review, listing update, website activity). |
| 3 → 2 | Inferred → Verified | Successful agent interaction with confirmed outcome. Periodic verification call answered. |
| 2 → 1 | Verified → Live | Provider integrates AI receptionist, connects FSM platform, or registers dispatch API. |

**Downward transitions** (higher → lower tier):

| From | To | Trigger |
|------|----|---------|
| 1 → 2 | Live → Verified | Integration endpoint becomes unresponsive for > 24 hours. API errors exceed threshold. |
| 2 → 3 | Verified → Inferred | No successful interaction in 30 days. Verification attempt fails. |
| 3 → 4 | Inferred → Stale | No proxy signal update in 90 days. Carrier reports number disconnected. |
| Any → 0 | Any → Suspended | Carrier confirms permanently disconnected. Fraud detected. TCPA violation flagged. Multiple consecutive failed activations with disconnected signal. |

### 5.3 Signal Sources by Tier

| Signal Source | Tier 1 | Tier 2 | Tier 3 | Tier 4 |
|---------------|--------|--------|--------|--------|
| Live API / AI receptionist | ✅ Required | — | — | — |
| FSM platform integration | ✅ Qualifies | ✅ Qualifies | — | — |
| Recent agent interaction | ✅ | ✅ Required | — | — |
| Published business hours | ✅ | ✅ | ✅ Primary | — |
| Recent review activity | ✅ | ✅ | ✅ Primary | — |
| Active website | ✅ | ✅ | ✅ Supporting | — |
| Directory listing | ✅ | ✅ | ✅ Supporting | ✅ Only signal |
| Carrier status (active) | ✅ | ✅ | ✅ | ✅ |

---

## 6. Trust Score Model

### 6.1 Overview

Trust scores are composite metrics derived from two signal classes:

1. **Credential signals** — verified licenses, insurance, certifications, identity, carrier status. These are binary or categorical: the credential exists or it doesn't.
2. **Interaction signals** — outcomes from real H.164 interactions. Resolution rates, response rates, dispute rates, price accuracy, repeat activation. These are continuous and decay over time.

Trust scores are calculated separately for service nodes and request nodes, using different weights appropriate to each role.

### 6.2 Service Node Trust Score

```
composite = (w1 × verification_score) +
            (w2 × resolution_rate) +
            (w3 × response_rate) +
            (w4 × (1 - dispute_rate)) +
            (w5 × repeat_rate) +
            (w6 × price_accuracy)
```

**Recommended initial weights:**

| Component | Weight | Rationale |
|-----------|--------|-----------|
| `verification_score` | 0.20 | Baseline credibility from verified credentials. |
| `resolution_rate` | 0.25 | Did the service actually get delivered? Core signal. |
| `response_rate` | 0.15 | Does the provider answer when activated? |
| `1 - dispute_rate` | 0.15 | Absence of disputes. |
| `repeat_rate` | 0.15 | Revealed preference — strongest behavioral signal. |
| `price_accuracy` | 0.10 | Does the provider charge what they quote? |

### 6.3 Request Node Trust Score

```
composite = (w1 × verification_score) +
            (w2 × completion_rate) +
            (w3 × payment_reliability) +
            (w4 × (1 - dispute_rate)) +
            (w5 × (1 - no_show_rate)) +
            (w6 × outcome_reporting_rate)
```

**Recommended initial weights:**

| Component | Weight | Rationale |
|-----------|--------|-----------|
| `verification_score` | 0.15 | Carrier verification, payment method, identity. |
| `completion_rate` | 0.25 | Does this requestor follow through on dispatches? |
| `payment_reliability` | 0.25 | Does this requestor pay reliably? |
| `1 - dispute_rate` | 0.15 | Low dispute frequency. |
| `1 - no_show_rate` | 0.10 | Requestor is available when provider arrives. |
| `outcome_reporting_rate` | 0.10 | Requestor's agent reports outcomes (feeds the log). |

### 6.4 Decay Model

Trust scores decay when no new interaction signals are received. This prevents stale high scores from persisting indefinitely.

```
score_effective = score_base × e^(-λt)
```

Where:
- `score_base` = most recently calculated composite score
- `λ` = decay constant (recommended: 0.005 per day)
- `t` = days since last interaction signal

At `λ = 0.005`:
- 30 days without signal: score retains ~86% of its value
- 90 days: ~64%
- 180 days: ~41%
- 365 days: ~16%

This ensures that active, engaged nodes maintain their scores while dormant nodes naturally decline, triggering tier downgrades and signaling reduced confidence to querying agents.

### 6.5 Cold Start

New nodes with zero interaction history receive an **initial trust score** derived solely from credential signals:

- Carrier-verified phone number: +0.10
- Active business license (verified): +0.15
- Insurance on file (verified): +0.10
- Identity verified (basic KYC): +0.10
- Identity verified (full KYC): +0.20
- Payment method on file (request nodes): +0.15
- Active directory listings: +0.05
- Recent positive reviews (external): +0.05 to +0.10 (scaled)

Maximum cold-start score is capped at **0.60** for service nodes and **0.50** for request nodes. Full trust can only be earned through interaction history.

**Bootstrap note:** During network bootstrap, most service nodes will have cold-start scores in the 0.15–0.40 range. Agent implementations SHOULD default `quality_threshold` to 0.0 (accept all) and present low-trust nodes to users with appropriate confidence indicators rather than hard-filtering them. This allows early interactions to occur — generating the outcome signals that bootstrap trust scores network-wide. Hard-filtering seed nodes before any interactions take place creates a dead-start problem. The default thresholds in Section 6.6 (0.0) are set deliberately to avoid this.

### 6.6 Threshold Mechanics

Nodes can set minimum counterparty trust scores:

- A **service node** sets `min_requestor_trust`: queries from request nodes below this score are filtered before they reach the provider. Default: 0.0 (accept all).
- A **request node** sets `quality_threshold` in preferences and `min_provider_trust` in consent thresholds: only service nodes at or above this score are returned in match results. Default: 0.0 (accept all).

When both sides set thresholds, both must be satisfied for a match to occur. This creates a bilateral trust market where high-trust nodes on both sides naturally match with each other.

---

## 7. Example Nodes Across Verticals

### 7.1 Emergency Plumber — Tier 1 (Live Wrapped)

```json
{
  "version": "1.0-draft",
  "endpoint": "+14155559876",
  "entity": {
    "name": "Bay Area Emergency Plumbing",
    "type": "business",
    "legal_name": "BAEP Services LLC",
    "identifiers": {
      "state_license": "CA-PLB-987654"
    }
  },
  "capabilities": {
    "primary": {
      "category": "plumbing",
      "subcategory": ["emergency_repair", "leak_detection", "drain_cleaning", "water_heater", "sewer_line"]
    },
    "languages": ["en", "es"],
    "specializations": ["24_hour_emergency", "commercial_plumbing", "tankless_water_heaters"]
  },
  "coverage": {
    "zones": [
      {
        "type": "radius",
        "center": { "lat": 37.7749, "lng": -122.4194 },
        "radius_km": 40,
        "label": "SF Bay Area"
      }
    ],
    "mobile": true,
    "headquarters": {
      "lat": 37.7749,
      "lng": -122.4194,
      "address": "123 Mission St, San Francisco, CA 94105"
    }
  },
  "availability": {
    "tier": 1,
    "confidence": 0.97,
    "signal_sources": ["ai_receptionist", "recent_interaction", "fsm_integration"],
    "hours": {
      "timezone": "America/Los_Angeles",
      "regular": {
        "mon": { "open": "00:00", "close": "23:59" },
        "tue": { "open": "00:00", "close": "23:59" },
        "wed": { "open": "00:00", "close": "23:59" },
        "thu": { "open": "00:00", "close": "23:59" },
        "fri": { "open": "00:00", "close": "23:59" },
        "sat": { "open": "00:00", "close": "23:59" },
        "sun": { "open": "00:00", "close": "23:59" }
      },
      "emergency_available": true
    },
    "last_signal": "2026-02-23T10:15:00Z",
    "next_refresh": "2026-02-23T10:20:00Z"
  },
  "trust": {
    "composite": 0.92,
    "interaction_count": 347,
    "resolution_rate": 0.94,
    "response_rate": 0.98,
    "dispute_rate": 0.02,
    "repeat_rate": 0.41,
    "price_accuracy": 0.89,
    "verification_score": 0.95,
    "decay_rate": 0.005,
    "last_updated": "2026-02-23T08:00:00Z"
  },
  "response_sla": {
    "expected_seconds": 15,
    "p50_seconds": 8,
    "p95_seconds": 45,
    "sample_size": 347
  },
  "activation": {
    "methods": [
      {
        "type": "dispatch_api",
        "endpoint": "https://api.baeplumbing.com/h164/dispatch",
        "priority": 1,
        "ai_receptionist": true,
        "capabilities": ["dispatch", "quote", "triage"]
      },
      {
        "type": "voice",
        "endpoint": "+14155559876",
        "priority": 2,
        "ai_receptionist": true,
        "capabilities": ["dispatch", "quote", "triage", "transfer_to_human"]
      },
      {
        "type": "sms",
        "endpoint": "+14155559876",
        "priority": 3,
        "capabilities": ["schedule"]
      }
    ],
    "preferred_channel": "dispatch_api"
  },
  "verification": {
    "licenses": [
      {
        "type": "plumbing_contractor",
        "number": "CA-PLB-987654",
        "issuer": "State of California CSLB",
        "jurisdiction": "CA",
        "status": "active",
        "verified_at": "2026-01-15T00:00:00Z",
        "expires_at": "2027-01-15T00:00:00Z"
      }
    ],
    "insurance": [
      {
        "type": "general_liability",
        "carrier": "State Farm",
        "coverage_amount": 2000000,
        "coverage_currency": "USD",
        "status": "active",
        "verified_at": "2026-01-15T00:00:00Z",
        "expires_at": "2027-01-15T00:00:00Z"
      }
    ],
    "carrier_verified": true,
    "identity_verified": true
  },
  "listing_basis": "provider_opted_in",
  "last_verified": "2026-02-23T10:15:00Z",
  "metadata": {
    "average_job_price_usd": 450,
    "fleet_size": 12,
    "years_in_business": 18,
    "accepts_crypto": true
  }
}
```

### 7.2 Emergency Plumber — Tier 3 (Inferred Available)

Same provider category, minimal data. Demonstrates the thin node.

```json
{
  "version": "1.0-draft",
  "endpoint": "+14155551234",
  "entity": {
    "name": "Mike's Plumbing",
    "type": "individual"
  },
  "capabilities": {
    "primary": {
      "category": "plumbing",
      "subcategory": ["emergency_repair", "drain_cleaning"]
    }
  },
  "coverage": {
    "zones": [
      {
        "type": "postal_codes",
        "postal_codes": ["94110", "94112", "94114", "94131"]
      }
    ],
    "mobile": true
  },
  "availability": {
    "tier": 3,
    "confidence": 0.45,
    "signal_sources": ["published_hours", "directory_listing", "carrier_active"],
    "hours": {
      "timezone": "America/Los_Angeles",
      "regular": {
        "mon": { "open": "08:00", "close": "18:00" },
        "tue": { "open": "08:00", "close": "18:00" },
        "wed": { "open": "08:00", "close": "18:00" },
        "thu": { "open": "08:00", "close": "18:00" },
        "fri": { "open": "08:00", "close": "18:00" },
        "sat": { "open": "09:00", "close": "14:00" },
        "sun": { "closed": true }
      },
      "emergency_available": false
    },
    "last_signal": "2026-02-10T00:00:00Z"
  },
  "trust": {
    "composite": 0.35,
    "interaction_count": 0,
    "verification_score": 0.35,
    "decay_rate": 0.005,
    "last_updated": "2026-02-10T00:00:00Z"
  },
  "activation": {
    "methods": [
      {
        "type": "voice",
        "endpoint": "+14155551234",
        "priority": 1,
        "ai_receptionist": false
      }
    ],
    "preferred_channel": "voice"
  },
  "listing_basis": "public_listing",
  "last_verified": "2026-02-10T00:00:00Z"
}
```

### 7.3 Tow Truck Operator — Tier 1 (Dispatch-Heavy)

```json
{
  "version": "1.0-draft",
  "endpoint": "+12125559999",
  "entity": {
    "name": "Metro Tow NYC",
    "type": "business",
    "legal_name": "Metro Towing Services Inc."
  },
  "capabilities": {
    "primary": {
      "category": "towing",
      "subcategory": ["emergency_tow", "flatbed", "motorcycle_tow", "long_distance"]
    },
    "secondary": [
      {
        "category": "roadside_assistance",
        "subcategory": ["jump_start", "tire_change", "lockout", "fuel_delivery"]
      }
    ],
    "specializations": ["heavy_duty", "ev_towing", "exotic_vehicles"]
  },
  "coverage": {
    "zones": [
      {
        "type": "polygon",
        "polygon": [
          { "lat": 40.9176, "lng": -74.2591 },
          { "lat": 40.9176, "lng": -73.7004 },
          { "lat": 40.4774, "lng": -73.7004 },
          { "lat": 40.4774, "lng": -74.2591 }
        ],
        "label": "NYC Metro"
      }
    ],
    "mobile": true
  },
  "availability": {
    "tier": 1,
    "confidence": 0.95,
    "signal_sources": ["live_endpoint", "fsm_integration", "recent_interaction"],
    "hours": {
      "timezone": "America/New_York",
      "regular": {
        "mon": { "open": "00:00", "close": "23:59" },
        "tue": { "open": "00:00", "close": "23:59" },
        "wed": { "open": "00:00", "close": "23:59" },
        "thu": { "open": "00:00", "close": "23:59" },
        "fri": { "open": "00:00", "close": "23:59" },
        "sat": { "open": "00:00", "close": "23:59" },
        "sun": { "open": "00:00", "close": "23:59" }
      },
      "emergency_available": true
    },
    "last_signal": "2026-02-23T14:00:00Z"
  },
  "trust": {
    "composite": 0.88,
    "interaction_count": 1203,
    "resolution_rate": 0.91,
    "response_rate": 0.96,
    "dispute_rate": 0.03,
    "repeat_rate": 0.28,
    "price_accuracy": 0.92,
    "verification_score": 0.85,
    "decay_rate": 0.005,
    "last_updated": "2026-02-23T12:00:00Z"
  },
  "response_sla": {
    "expected_seconds": 10,
    "p50_seconds": 6,
    "p95_seconds": 30,
    "sample_size": 1203
  },
  "activation": {
    "methods": [
      {
        "type": "dispatch_api",
        "endpoint": "https://api.metrotownyc.com/dispatch",
        "priority": 1,
        "ai_receptionist": true,
        "capabilities": ["dispatch", "quote", "triage"]
      },
      {
        "type": "voice",
        "endpoint": "+12125559999",
        "priority": 2,
        "ai_receptionist": true,
        "capabilities": ["dispatch", "quote", "transfer_to_human"]
      }
    ],
    "preferred_channel": "dispatch_api"
  },
  "verification": {
    "licenses": [
      {
        "type": "towing_operator",
        "number": "NY-TOW-55432",
        "issuer": "NYC TLC",
        "jurisdiction": "NY",
        "status": "active",
        "verified_at": "2026-01-20T00:00:00Z"
      }
    ],
    "insurance": [
      {
        "type": "general_liability",
        "carrier": "Progressive Commercial",
        "coverage_amount": 5000000,
        "status": "active",
        "verified_at": "2026-01-20T00:00:00Z"
      },
      {
        "type": "garage_keepers",
        "carrier": "Progressive Commercial",
        "coverage_amount": 1000000,
        "status": "active",
        "verified_at": "2026-01-20T00:00:00Z"
      }
    ],
    "carrier_verified": true,
    "identity_verified": true
  },
  "listing_basis": "provider_opted_in",
  "last_verified": "2026-02-23T14:00:00Z",
  "metadata": {
    "fleet_size": 22,
    "average_eta_minutes": 18,
    "accepts_insurance_claims": true
  }
}
```

### 7.4 Dentist Office — Tier 2 (Appointment-Based)

```json
{
  "version": "1.0-draft",
  "endpoint": "+13105557777",
  "entity": {
    "name": "Bright Smile Dental",
    "type": "business",
    "legal_name": "Bright Smile Dental PC",
    "identifiers": {
      "npi": "1234567890"
    }
  },
  "capabilities": {
    "primary": {
      "category": "dental",
      "subcategory": ["general_dentistry", "cleaning", "fillings", "crowns", "emergency_dental"]
    },
    "secondary": [
      {
        "category": "dental",
        "subcategory": ["cosmetic", "whitening", "invisalign"]
      }
    ],
    "languages": ["en", "ko"]
  },
  "coverage": {
    "zones": [
      {
        "type": "radius",
        "center": { "lat": 34.0522, "lng": -118.2437 },
        "radius_km": 15,
        "label": "West Los Angeles"
      }
    ],
    "mobile": false,
    "headquarters": {
      "lat": 34.0522,
      "lng": -118.2437,
      "address": "456 Wilshire Blvd, Los Angeles, CA 90024"
    }
  },
  "availability": {
    "tier": 2,
    "confidence": 0.78,
    "signal_sources": ["recent_interaction", "published_hours", "review_activity"],
    "hours": {
      "timezone": "America/Los_Angeles",
      "regular": {
        "mon": { "open": "09:00", "close": "17:00" },
        "tue": { "open": "09:00", "close": "17:00" },
        "wed": { "open": "09:00", "close": "17:00" },
        "thu": { "open": "09:00", "close": "19:00" },
        "fri": { "open": "09:00", "close": "15:00" },
        "sat": { "open": "10:00", "close": "14:00" },
        "sun": { "closed": true }
      },
      "emergency_available": true
    },
    "last_signal": "2026-02-21T00:00:00Z"
  },
  "trust": {
    "composite": 0.79,
    "interaction_count": 42,
    "resolution_rate": 0.93,
    "response_rate": 0.88,
    "dispute_rate": 0.00,
    "repeat_rate": 0.52,
    "price_accuracy": 0.95,
    "verification_score": 0.90,
    "decay_rate": 0.005,
    "last_updated": "2026-02-21T00:00:00Z"
  },
  "activation": {
    "methods": [
      {
        "type": "voice",
        "endpoint": "+13105557777",
        "priority": 1,
        "ai_receptionist": true,
        "capabilities": ["schedule", "triage", "transfer_to_human"]
      },
      {
        "type": "sms",
        "endpoint": "+13105557777",
        "priority": 2,
        "capabilities": ["schedule"]
      }
    ],
    "preferred_channel": "voice"
  },
  "verification": {
    "licenses": [
      {
        "type": "dental_license",
        "number": "DDS-CA-45678",
        "issuer": "Dental Board of California",
        "jurisdiction": "CA",
        "status": "active",
        "verified_at": "2026-01-10T00:00:00Z"
      }
    ],
    "insurance": [
      {
        "type": "malpractice",
        "carrier": "Dentists Insurance Company",
        "coverage_amount": 3000000,
        "status": "active",
        "verified_at": "2026-01-10T00:00:00Z"
      }
    ],
    "carrier_verified": true,
    "identity_verified": true
  },
  "listing_basis": "provider_claimed",
  "last_verified": "2026-02-21T00:00:00Z",
  "metadata": {
    "accepts_insurance": true,
    "insurance_networks": ["Delta Dental", "Cigna", "Aetna"],
    "new_patient_available": true,
    "average_wait_days_new_patient": 5
  }
}
```

### 7.5 Criminal Defense Attorney — Tier 2 (Consultation-Based)

```json
{
  "version": "1.0-draft",
  "endpoint": "+12125553333",
  "entity": {
    "name": "Law Office of Sarah Chen",
    "type": "individual",
    "legal_name": "Sarah Chen, Esq.",
    "identifiers": {
      "bar_number": "NY-BAR-123456"
    }
  },
  "capabilities": {
    "primary": {
      "category": "legal",
      "subcategory": ["criminal_defense", "dui", "drug_offenses", "assault", "white_collar"]
    },
    "languages": ["en", "zh"]
  },
  "coverage": {
    "zones": [
      {
        "type": "region",
        "region": "New York City"
      },
      {
        "type": "region",
        "region": "Westchester County"
      }
    ],
    "mobile": false,
    "headquarters": {
      "lat": 40.7580,
      "lng": -73.9855,
      "address": "789 Broadway, New York, NY 10003"
    }
  },
  "availability": {
    "tier": 2,
    "confidence": 0.75,
    "signal_sources": ["recent_interaction", "published_hours", "website_active"],
    "hours": {
      "timezone": "America/New_York",
      "regular": {
        "mon": { "open": "09:00", "close": "18:00" },
        "tue": { "open": "09:00", "close": "18:00" },
        "wed": { "open": "09:00", "close": "18:00" },
        "thu": { "open": "09:00", "close": "18:00" },
        "fri": { "open": "09:00", "close": "17:00" },
        "sat": { "closed": true },
        "sun": { "closed": true }
      },
      "emergency_available": true
    },
    "last_signal": "2026-02-20T00:00:00Z"
  },
  "trust": {
    "composite": 0.82,
    "interaction_count": 28,
    "resolution_rate": 0.96,
    "response_rate": 0.85,
    "dispute_rate": 0.00,
    "repeat_rate": 0.18,
    "price_accuracy": 0.97,
    "verification_score": 0.95,
    "decay_rate": 0.005,
    "last_updated": "2026-02-20T00:00:00Z"
  },
  "activation": {
    "methods": [
      {
        "type": "voice",
        "endpoint": "+12125553333",
        "priority": 1,
        "ai_receptionist": false,
        "capabilities": ["triage", "schedule"]
      },
      {
        "type": "sms",
        "endpoint": "+12125553333",
        "priority": 2,
        "capabilities": ["schedule"]
      }
    ],
    "preferred_channel": "voice"
  },
  "verification": {
    "licenses": [
      {
        "type": "bar_admission",
        "number": "NY-BAR-123456",
        "issuer": "New York State Bar Association",
        "jurisdiction": "NY",
        "status": "active",
        "verified_at": "2026-02-01T00:00:00Z"
      }
    ],
    "insurance": [
      {
        "type": "malpractice",
        "carrier": "LawPro Insurance",
        "coverage_amount": 5000000,
        "status": "active",
        "verified_at": "2026-02-01T00:00:00Z"
      }
    ],
    "carrier_verified": true,
    "identity_verified": true
  },
  "listing_basis": "public_listing",
  "last_verified": "2026-02-20T00:00:00Z",
  "metadata": {
    "consultation_fee_usd": 0,
    "consultation_type": "free_initial",
    "years_practicing": 14,
    "notable_cases": true
  }
}
```

### 7.6 Mobile Dog Groomer — Tier 3 (Coverage-Area-Based)

```json
{
  "version": "1.0-draft",
  "endpoint": "+15125558888",
  "entity": {
    "name": "Pampered Paws Mobile Grooming",
    "type": "individual"
  },
  "capabilities": {
    "primary": {
      "category": "pet_services",
      "subcategory": ["dog_grooming", "dog_bathing", "nail_trimming", "deshedding"]
    },
    "specializations": ["large_breeds", "anxious_dogs", "puppy_first_groom"]
  },
  "coverage": {
    "zones": [
      {
        "type": "postal_codes",
        "postal_codes": ["78701", "78702", "78703", "78704", "78731", "78745", "78748", "78749"],
        "label": "Central Austin"
      }
    ],
    "mobile": true
  },
  "availability": {
    "tier": 3,
    "confidence": 0.52,
    "signal_sources": ["published_hours", "review_activity", "website_active"],
    "hours": {
      "timezone": "America/Chicago",
      "regular": {
        "mon": { "open": "08:00", "close": "17:00" },
        "tue": { "open": "08:00", "close": "17:00" },
        "wed": { "open": "08:00", "close": "17:00" },
        "thu": { "open": "08:00", "close": "17:00" },
        "fri": { "open": "08:00", "close": "17:00" },
        "sat": { "open": "09:00", "close": "15:00" },
        "sun": { "closed": true }
      },
      "emergency_available": false
    },
    "last_signal": "2026-02-15T00:00:00Z"
  },
  "trust": {
    "composite": 0.41,
    "interaction_count": 0,
    "verification_score": 0.40,
    "decay_rate": 0.005,
    "last_updated": "2026-02-15T00:00:00Z"
  },
  "activation": {
    "methods": [
      {
        "type": "voice",
        "endpoint": "+15125558888",
        "priority": 1,
        "ai_receptionist": false
      },
      {
        "type": "sms",
        "endpoint": "+15125558888",
        "priority": 2
      }
    ],
    "preferred_channel": "voice"
  },
  "verification": {
    "certifications": [
      {
        "name": "Certified Professional Groomer",
        "issuer": "National Dog Groomers Association of America",
        "verified_at": "2026-01-05T00:00:00Z"
      }
    ],
    "carrier_verified": true,
    "identity_verified": false
  },
  "listing_basis": "public_listing",
  "last_verified": "2026-02-15T00:00:00Z",
  "metadata": {
    "price_range_usd": { "min": 60, "max": 150 },
    "booking_lead_time_days": 3,
    "mobile_unit_type": "custom_van"
  }
}
```

### 7.7 Example Request Node

```json
{
  "version": "1.0-draft",
  "endpoint": "+14155550101",
  "entity": {
    "type": "individual",
    "location": {
      "locality": "San Francisco",
      "region": "CA",
      "country": "US"
    }
  },
  "agent_identity": {
    "platform": "anthropic",
    "agent_id": "claude-agent-abc123",
    "agent_version": "2026.2",
    "authorization": {
      "scope": ["query", "activate", "schedule", "pay"],
      "granted_at": "2026-01-15T00:00:00Z"
    }
  },
  "trust": {
    "composite": 0.73,
    "interaction_count": 12,
    "completion_rate": 0.92,
    "payment_reliability": 0.92,
    "dispute_rate": 0.00,
    "no_show_rate": 0.08,
    "outcome_reporting_rate": 0.83,
    "decay_rate": 0.005,
    "last_updated": "2026-02-22T00:00:00Z"
  },
  "preferences": {
    "service_categories": ["plumbing", "electrical", "dental", "legal"],
    "location_preference": {
      "max_radius_km": 25,
      "preferred_locality": "San Francisco"
    },
    "budget_ranges": {
      "plumbing": { "min": 100, "max": 800, "currency": "USD" },
      "dental": { "min": 0, "max": 2000, "currency": "USD" }
    },
    "quality_threshold": 0.30,
    "language_preference": ["en"]
  },
  "consent_thresholds": {
    "inbound_policy": "trust_gated",
    "min_provider_trust": 0.50,
    "allowed_categories": ["plumbing", "electrical", "dental"],
    "blocked_categories": ["telemarketing"],
    "time_restrictions": {
      "timezone": "America/Los_Angeles",
      "allowed_hours": { "start": "08:00", "end": "21:00" },
      "emergency_override": true
    }
  },
  "reachability": {
    "preferred_channel": "push_notification",
    "channels": [
      { "type": "push_notification", "endpoint": "agent://anthropic/abc123", "active": true },
      { "type": "sms", "endpoint": "+14155550101", "active": true },
      { "type": "voice", "endpoint": "+14155550101", "active": true }
    ],
    "do_not_disturb": false
  },
  "interaction_history": {
    "total_requests": 12,
    "total_completions": 11,
    "total_disputes": 0,
    "total_no_shows": 1,
    "categories_used": ["plumbing", "dental", "towing"],
    "first_interaction": "2026-01-20T00:00:00Z",
    "last_interaction": "2026-02-22T00:00:00Z"
  },
  "verification": {
    "carrier_verified": true,
    "payment_method_on_file": true,
    "identity_confidence": "basic_kyc",
    "verified_at": "2026-01-15T00:00:00Z"
  },
  "registration_basis": "user_opted_in",
  "metadata": {}
}
```

---

## Appendix A: Schema Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0-draft | 2026-02-23 | Initial draft. Service Node, Request Node, Interaction Event, Availability Tiers, Trust Model. |

---

## Appendix B: Category Taxonomy (Initial)

The capabilities taxonomy is extensible. The initial set is seeded from NAICS codes and major service marketplaces. A future HIP-6 will formalize governance and community additions. The following categories represent the initial controlled vocabulary:

| Category | Example Subcategories |
|----------|----------------------|
| `plumbing` | emergency_repair, drain_cleaning, water_heater, sewer_line, fixture_install |
| `electrical` | emergency_repair, panel_upgrade, wiring, lighting, ev_charger |
| `hvac` | emergency_repair, ac_install, furnace_repair, duct_cleaning |
| `towing` | emergency_tow, flatbed, motorcycle_tow, long_distance |
| `roadside_assistance` | jump_start, tire_change, lockout, fuel_delivery |
| `dental` | general_dentistry, emergency_dental, cosmetic, orthodontics |
| `medical` | primary_care, urgent_care, specialist_referral |
| `legal` | criminal_defense, family_law, personal_injury, immigration, estate_planning |
| `pet_services` | dog_grooming, veterinary, pet_sitting, dog_walking |
| `automotive` | repair, maintenance, body_shop, inspection |
| `cleaning` | residential, commercial, deep_clean, move_out |
| `landscaping` | lawn_care, tree_service, hardscaping, irrigation |
| `pest_control` | general, termite, wildlife_removal |
| `locksmith` | emergency_lockout, rekey, lock_install, safe |
| `moving` | local, long_distance, packing, storage |

Additional categories are added via HIP process.

---

*H.164 is an open protocol licensed under Apache 2.0.*
*Specification repository: [github.com/h164-protocol](https://github.com/h164-protocol)*
*Contact: Dustin@h164.io*
