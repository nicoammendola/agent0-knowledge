# Watchtower: Trust-as-a-Service for Agents

*Continuous monitoring infrastructure for agent health, compliance, and performance*

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [The Core Concept](#the-core-concept)
3. [What Watchtower Should (and Shouldn't) Measure](#what-watchtower-should-and-shouldnt-measure)
4. [Tiered Monitoring System](#tiered-monitoring-system)
5. [API Design](#api-design)
6. [Integration with Trust Score](#integration-with-trust-score)
7. [Challenges & Mitigations](#challenges--mitigations)
8. [Business Model](#business-model)
9. [Implementation Considerations](#implementation-considerations)
10. [Roadmap](#roadmap)

---

## Executive Summary

**Watchtower** is a continuous monitoring system that provides objective, real-time health signals for registered agents. Think of it as **Core Web Vitals for the agent economy**.

### The Value Proposition

| Problem | Watchtower Solution |
|---------|---------------------|
| "Is this agent actually online?" | Real-time reachability checks |
| "Is this agent reliable?" | Historical uptime tracking |
| "Does this agent follow standards?" | Protocol compliance verification |
| "How fast is this agent?" | Latency benchmarking |
| Reputation can be gamed | Watchtower metrics are objective and hard to fake |

### Key Insight

> **Watchtower provides objective, measurable, hard-to-fake signals that complement subjective reputation data.**

Reputation tells you: "Users liked this agent in the past."
Watchtower tells you: "This agent is working right now."

---

## The Core Concept

### The Google Analogy

| Google Search | Agent0 Watchtower |
|---------------|-------------------|
| Crawls websites | Monitors agent endpoints |
| Indexes content | Indexes capabilities (tools, skills) |
| Core Web Vitals (LCP, FID, CLS) | Agent Vitals (uptime, latency, compliance) |
| PageRank (link-based ranking) | Trust Score (verification + reputation + health) |
| Serves search results | Serves curated lists API |
| Doesn't evaluate "is this website good?" | Doesn't evaluate "does this agent work well?" |

**Key principle:** Google doesn't try to answer "is this a good website?" They measure objective signals and let users decide. Watchtower should do the same for agents.

### What Makes Watchtower Valuable

| Strength | Why It Matters |
|----------|----------------|
| **Objective data** | Can't be gamed like reviews; either you're online or you're not |
| **Real-time signals** | Know if an agent is down RIGHT NOW, not after bad reviews |
| **Automated & scalable** | Doesn't require human curation to function |
| **Hard to fake** | You can't Sybil-attack your response latency |
| **Fills a real gap** | "Is this agent actually working?" is unaddressed |
| **Complements reputation** | Reputation = historical quality; Watchtower = current health |

---

## What Watchtower Should (and Shouldn't) Measure

### ✅ SHOULD Measure: Objective, Hard-to-Fake Metrics

| Metric | Difficulty | Why It Works |
|--------|------------|--------------|
| Endpoint reachability (HTTP 200?) | Easy | Binary—either up or down |
| Response latency (p50, p95, p99) | Easy | Measurable, comparable |
| SSL/TLS certificate validity | Easy | Security baseline |
| DNS resolution time | Easy | Infrastructure quality signal |
| Uptime percentage (24h, 7d, 30d) | Easy | Historical reliability |
| MCP `tools/list` returns valid JSON | Medium | Protocol compliance |
| A2A agent card is valid schema | Medium | Protocol compliance |
| Registration file matches on-chain | Medium | Data integrity |
| Error rate under normal load | Medium | Stability signal |

### ❌ SHOULD NOT Measure: Subjective, Domain-Specific Quality

| Metric | Difficulty | Why It Doesn't Work |
|--------|------------|---------------------|
| "Does the agent write good code?" | **HARD** | Requires code quality evaluation—a product in itself |
| "Is the agent's advice accurate?" | **HARD** | Requires domain expertise and ground truth |
| "Does it do what the description claims?" | **HARD** | Requires semantic understanding of claims |
| "Is the output safe/ethical?" | **HARD** | Requires safety evaluation infrastructure |
| "Is it worth the price?" | **HARD** | Subjective, depends on use case |

### The Boundary

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WATCHTOWER MEASUREMENT BOUNDARY                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  WATCHTOWER MEASURES                    │  LEAVE TO OTHERS                  │
│  ───────────────────                    │  ────────────────                 │
│                                         │                                   │
│  • Is it online?                        │  • Is it good at its job?         │
│  • Is it fast?                          │  • Is the output accurate?        │
│  • Is it reliable?                      │  • Is it safe?                    │
│  • Does it follow protocols?            │  • Is it worth the money?         │
│  • Is the schema valid?                 │  • Does it match my needs?        │
│                                         │                                   │
│  = OBJECTIVE HEALTH                     │  = SUBJECTIVE QUALITY             │
│                                         │                                   │
│  Watchtower handles this                │  Reputation + Auditors handle     │
│                                         │  this                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Tiered Monitoring System

### Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      WATCHTOWER MONITORING TIERS                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TIER 1: HEALTH CHECK                                                       │
│  Cost: Free | Frequency: Every 5 min | Scope: All registered agents         │
│                                                                             │
│  TIER 2: PROTOCOL COMPLIANCE                                                │
│  Cost: Free | Frequency: Hourly | Scope: Opted-in agents                    │
│                                                                             │
│  TIER 3: PERFORMANCE BENCHMARKS                                             │
│  Cost: Paid | Frequency: Daily | Scope: Opted-in, paying agents             │
│                                                                             │
│  TIER 4: CAPABILITY VERIFICATION                                            │
│  Cost: Varies | Frequency: On-demand | Scope: Third-party marketplace       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

### Tier 1: Health Check (Free, Automated, All Agents)

**Purpose:** Basic liveness monitoring for all registered agents.

**What We Check:**

| Check | Method | Output |
|-------|--------|--------|
| Endpoint reachability | HTTP HEAD/GET request | UP / DOWN / DEGRADED |
| Response latency | Measure time-to-first-byte | Milliseconds (p50, p95, p99) |
| SSL/TLS validity | Certificate chain verification | VALID / EXPIRING / INVALID |
| DNS resolution | DNS lookup timing | Milliseconds |
| HTTP status code | Check for 2xx response | Status code |

**Frequency:** Every 5 minutes

**Output:**
- Health status badge (🟢 Healthy / 🟡 Degraded / 🔴 Down)
- Uptime percentage (24h, 7d, 30d)
- Average latency

**Implementation Notes:**
- Run from multiple geographic regions to detect regional outages
- Store historical data for trend analysis
- Alert on status changes (optional webhook for agent owners)

---

### Tier 2: Protocol Compliance (Free, Automated, Opted-In)

**Purpose:** Verify that agents correctly implement advertised protocols.

**What We Check:**

| Protocol | Check | Output |
|----------|-------|--------|
| **MCP** | `tools/list` returns valid JSON-RPC | COMPLIANT / NON-COMPLIANT |
| **MCP** | `prompts/list` returns valid response | COMPLIANT / NON-COMPLIANT |
| **MCP** | `resources/list` returns valid response | COMPLIANT / NON-COMPLIANT |
| **MCP** | Listed tools have valid schemas | Schema validation score |
| **A2A** | Agent card accessible at well-known path | FOUND / NOT FOUND |
| **A2A** | Agent card schema is valid | VALID / INVALID |
| **A2A** | Advertised skills are properly formatted | Validation score |
| **General** | Registration file matches on-chain URI | MATCH / MISMATCH |
| **General** | CORS headers allow discovery | ENABLED / DISABLED |

**Frequency:** Hourly

**Output:**
- Protocol compliance badges (MCP ✓, A2A ✓)
- Compliance score (0-100)
- List of validation errors/warnings

**Why Opt-In:**
- Some compliance checks require deeper probing
- Agents can prepare for compliance (no surprise "gotchas")
- Creates incentive to opt-in for better trust score

---

### Tier 3: Performance Benchmarks (Paid, Opted-In)

**Purpose:** Detailed performance testing for agents that want certification.

**What We Check:**

| Benchmark | Method | Output |
|-----------|--------|--------|
| Throughput | Sequential requests over 1 minute | Requests/second |
| Concurrency | Parallel requests (10, 50, 100) | Success rate under load |
| Error rate | Track 4xx/5xx responses | Error percentage |
| Consistency | Same input → same output? | Consistency score |
| Rate limiting | Detect and document limits | Rate limit details |
| Cold start | First request after idle | Cold start latency |
| Warm latency | Subsequent requests | Warm latency |

**Frequency:** Daily (or on-demand)

**Output:**
- Performance certification badge
- Detailed benchmark report
- Comparison to category averages
- Historical performance trends

**Why Paid:**
- Resource-intensive testing
- Creates revenue stream
- Self-selection for serious agents

---

### Tier 4: Capability Verification (Third-Party Marketplace)

**Purpose:** Domain-specific quality evaluation by specialized auditors.

**NOT done by Agent0 directly.** Instead, we enable a marketplace:

| Auditor Type | What They Test | Example |
|--------------|----------------|---------|
| **Code Quality** | Coding agents' output quality | "CodeEval" runs coding challenges |
| **Safety** | Harmful content generation | "SafetyFirst" runs red-team tests |
| **Accuracy** | Factual correctness | "FactCheck" verifies claims |
| **Domain Expert** | Industry-specific quality | "LegalReview" for legal agents |

**How It Works:**

1. Third-party auditors register with Agent0
2. Auditors define their evaluation methodology
3. Agents request evaluation (and pay auditor's fee)
4. Results are published to Agent0's registry
5. Users can filter by auditor certifications

**Agent0's Role:**
- Provide infrastructure for auditor registration
- Standardize result format
- Display auditor certifications in trust data
- NOT responsible for auditor quality (marketplace dynamics)

---

## API Design

### Health Endpoints

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         WATCHTOWER API                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  HEALTH STATUS                                                              │
│  ─────────────                                                              │
│                                                                             │
│  GET /api/v1/health/{agentId}                                               │
│  Returns: Current health status                                             │
│  {                                                                          │
│    "status": "healthy",           // healthy | degraded | down              │
│    "latency": { "p50": 120, "p95": 250, "p99": 500 },                       │
│    "ssl": "valid",                                                          │
│    "lastCheck": "2026-01-09T12:00:00Z",                                     │
│    "region": "us-east-1"                                                    │
│  }                                                                          │
│                                                                             │
│  GET /api/v1/health/{agentId}/history                                       │
│  Returns: Historical health data                                            │
│  {                                                                          │
│    "uptime": { "24h": 99.9, "7d": 99.5, "30d": 98.2 },                      │
│    "incidents": [ { "start": "...", "end": "...", "type": "outage" } ],     │
│    "latencyTrend": [ { "date": "...", "p50": 120 }, ... ]                   │
│  }                                                                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  COMPLIANCE STATUS                                                          │
│  ─────────────────                                                          │
│                                                                             │
│  GET /api/v1/compliance/{agentId}                                           │
│  Returns: Protocol compliance details                                       │
│  {                                                                          │
│    "score": 95,                                                             │
│    "protocols": {                                                           │
│      "mcp": { "compliant": true, "version": "2025-06-18" },                 │
│      "a2a": { "compliant": true, "version": "0.3.0" }                       │
│    },                                                                       │
│    "issues": [ { "severity": "warning", "message": "..." } ],               │
│    "lastCheck": "2026-01-09T11:00:00Z"                                      │
│  }                                                                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  PERFORMANCE BENCHMARKS (Tier 3)                                            │
│  ───────────────────────────────                                            │
│                                                                             │
│  GET /api/v1/benchmarks/{agentId}                                           │
│  Returns: Performance benchmark results                                     │
│  {                                                                          │
│    "certified": true,                                                       │
│    "certifiedAt": "2026-01-08T00:00:00Z",                                   │
│    "throughput": { "rps": 50, "percentile": 85 },                           │
│    "concurrency": { "max": 100, "successRate": 99.2 },                      │
│    "coldStart": 1200,                                                       │
│    "warmLatency": 150                                                       │
│  }                                                                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ALERTS & INCIDENTS                                                         │
│  ──────────────────                                                         │
│                                                                             │
│  GET /api/v1/alerts/{agentId}                                               │
│  Returns: Recent alerts and incidents                                       │
│  {                                                                          │
│    "activeAlerts": [ { "type": "high_latency", "since": "..." } ],          │
│    "recentIncidents": [ { "type": "outage", "duration": 300, ... } ],       │
│    "incidentCount": { "24h": 0, "7d": 1, "30d": 3 }                         │
│  }                                                                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  LEADERBOARDS (Data-Driven Curated Lists)                                   │
│  ─────────────────────────────────────────                                  │
│                                                                             │
│  GET /api/v1/leaderboard/uptime                                             │
│  Returns: Top agents by uptime                                              │
│                                                                             │
│  GET /api/v1/leaderboard/latency                                            │
│  Returns: Fastest agents by response time                                   │
│                                                                             │
│  GET /api/v1/leaderboard/compliance                                         │
│  Returns: Highest protocol compliance scores                                │
│                                                                             │
│  GET /api/v1/leaderboard/overall                                            │
│  Returns: Best combined health score                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Webhook Notifications (For Agent Owners)

```typescript
// Agent owners can register webhooks to receive alerts

POST /api/v1/webhooks
{
  "agentId": "11155111:42",
  "url": "https://myagent.com/webhook",
  "events": ["status_change", "compliance_failure", "latency_spike"]
}

// Webhook payload example
{
  "event": "status_change",
  "agentId": "11155111:42",
  "timestamp": "2026-01-09T12:00:00Z",
  "data": {
    "previousStatus": "healthy",
    "currentStatus": "down",
    "reason": "HTTP 503"
  }
}
```

---

## Integration with Trust Score

Watchtower data becomes one of three pillars in the overall Trust Score:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TRUST SCORE COMPOSITION                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  TRUST SCORE = weighted combination of:                                     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  VERIFICATION (30%)                                                 │    │
│  │  • Tier 0-4 verification status                                     │    │
│  │  • Domain ownership proof                                           │    │
│  │  • Social verification                                              │    │
│  │  • Stake amount                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  REPUTATION (40%)                                                   │    │
│  │  • Weighted feedback score                                          │    │
│  │  • Number of reviews                                                │    │
│  │  • Review quality signals (x402 proof, reviewer trust)              │    │
│  │  • Recency of reviews                                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  WATCHTOWER HEALTH (30%)                                            │    │
│  │  • Uptime percentage (50% of health score)                          │    │
│  │  • Response latency vs. category average (25%)                      │    │
│  │  • Protocol compliance score (25%)                                  │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  Example calculation:                                                       │
│  ─────────────────────                                                      │
│  • Verification: Tier 2 verified → 70/100                                   │
│  • Reputation: 4.2/5 weighted score → 84/100                                │
│  • Watchtower: 99.5% uptime, 150ms (fast), 95% compliance → 95/100          │
│                                                                             │
│  Trust Score = 0.3(70) + 0.4(84) + 0.3(95) = 21 + 33.6 + 28.5 = 83.1        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Health Score Calculation

```typescript
interface HealthScore {
  overall: number;        // 0-100
  components: {
    uptime: number;       // 0-100 (e.g., 99.5% = 99.5)
    latency: number;      // 0-100 (relative to category)
    compliance: number;   // 0-100
  };
}

function calculateHealthScore(agent: AgentHealthData): HealthScore {
  // Uptime: Direct percentage
  const uptimeScore = agent.uptime30d; // e.g., 99.5
  
  // Latency: Compared to category average
  // Fast = 100, Average = 50, Slow = 0
  const categoryAvg = getCategoryAverageLatency(agent.category);
  const latencyRatio = categoryAvg / agent.latencyP50;
  const latencyScore = Math.min(100, latencyRatio * 50);
  
  // Compliance: Direct score
  const complianceScore = agent.complianceScore;
  
  // Weighted combination
  const overall = (uptimeScore * 0.5) + (latencyScore * 0.25) + (complianceScore * 0.25);
  
  return {
    overall,
    components: { uptime: uptimeScore, latency: latencyScore, compliance: complianceScore }
  };
}
```

---

## Challenges & Mitigations

### Challenge 1: Gaming/Manipulation

**Risk:** Agents detect watchtower requests and respond differently.

```typescript
// Malicious agent code
if (request.ip in KNOWN_WATCHTOWER_IPS) {
  return FAST_CACHED_RESPONSE;
} else {
  return SLOW_REAL_COMPUTATION;
}
```

**Mitigations:**

| Mitigation | Effectiveness | Cost |
|------------|---------------|------|
| Rotate monitoring IPs | Medium | Medium |
| Use residential proxies | High | High (ethically gray) |
| Random timing | Medium | Low |
| Community-reported discrepancies | Medium | Low |
| Cross-reference with user latency reports | High | Low |

**Best approach:** Accept that sophisticated gaming is possible, but:
1. Most agents won't bother
2. Discrepancies will surface in user feedback
3. Gaming the system is itself a reputation signal if detected

---

### Challenge 2: Cost at Scale

**Risk:** Continuous monitoring is expensive.

**Cost Breakdown:**

| Component | Cost Driver | Estimate |
|-----------|-------------|----------|
| Compute (checkers) | Request volume | $0.001 per check |
| Storage (history) | Data retention | $0.02 per GB/month |
| Multi-region | Redundancy | 3x multiplier |
| Bandwidth | Response size | Minimal |

**Mitigations:**

| Mitigation | Approach |
|------------|----------|
| Tiered frequency | Check popular agents more often |
| Sampling | Don't check every agent every time |
| Edge caching | Cache results for non-real-time queries |
| Paid tier | Offset costs with Tier 3 revenue |

---

### Challenge 3: Permission & Consent

**Risk:** Monitoring without consent could be problematic.

**Approach:**

| Tier | Consent Model |
|------|---------------|
| Tier 1 (Basic Health) | Implicit—public endpoints, like Google crawling |
| Tier 2 (Compliance) | Explicit opt-in |
| Tier 3 (Benchmarks) | Explicit opt-in + payment |
| Tier 4 (Capability) | Explicit request to auditor |

**Terms of Service:** Agents registering on ERC-8004 agree to basic health monitoring as part of being listed.

---

### Challenge 4: False Positives

**Risk:** Transient issues cause unfair penalties.

**Mitigations:**

| Mitigation | Implementation |
|------------|----------------|
| Grace periods | Don't flag until 3 consecutive failures |
| Multi-region consensus | Require 2+ regions to agree on outage |
| Historical smoothing | Use rolling averages, not point-in-time |
| Agent notification | Alert owners before public status change |

---

## Business Model

### Revenue Streams

| Stream | Description | Pricing |
|--------|-------------|---------|
| **Tier 1-2 (Free)** | Loss leader; drives adoption | $0 |
| **Tier 3 Certification** | Performance benchmarking | $50-200/month per agent |
| **API Access** | Access to health/leaderboard data | Tiered by volume |
| **Webhooks** | Real-time alerts for agent owners | $10/month |
| **White-label** | Watchtower for enterprise | Custom |
| **Auditor Marketplace** | Transaction fee on Tier 4 | 10-20% of auditor fee |

### Cost Structure

| Cost | Monthly Estimate (1000 agents) |
|------|-------------------------------|
| Compute (AWS Lambda) | $100-300 |
| Storage (TimescaleDB/InfluxDB) | $50-100 |
| Multi-region redundancy | $100-200 |
| Operations/maintenance | $500-1000 |
| **Total** | **$750-1600/month** |

### Break-Even Analysis

At $100/month for Tier 3 certification:
- Break-even: 8-16 paying agents
- Target: 50+ Tier 3 agents for healthy margin

---

## Implementation Considerations

### Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      WATCHTOWER ARCHITECTURE                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                    │
│  │  Scheduler  │────▶│   Workers   │────▶│  Agents     │                    │
│  │  (Cron)     │     │  (Lambda)   │     │  (Targets)  │                    │
│  └─────────────┘     └──────┬──────┘     └─────────────┘                    │
│                             │                                               │
│                             ▼                                               │
│                      ┌─────────────┐                                        │
│                      │  Time-Series│                                        │
│                      │  Database   │                                        │
│                      │  (InfluxDB) │                                        │
│                      └──────┬──────┘                                        │
│                             │                                               │
│            ┌────────────────┼────────────────┐                              │
│            ▼                ▼                ▼                              │
│     ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                       │
│     │  REST API   │  │  Webhooks   │  │  Subgraph   │                       │
│     │  (Public)   │  │  (Alerts)   │  │  (Index)    │                       │
│     └─────────────┘  └─────────────┘  └─────────────┘                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Existing Watchtower Service

The `watchtower/` folder in the repo already has:
- AWS Lambda deployment (`deploy/template.yaml`)
- Reachability checks (`src/checks/reachability.ts`)
- Feedback posting (`src/feedback/post.ts`)
- Subgraph integration (`src/subgraph/agents.ts`)

**Expansion needed:**
- Protocol compliance checks (MCP, A2A validation)
- Performance benchmarking suite
- Public API layer
- Historical data storage
- Alerting system

### Technology Choices

| Component | Recommended | Alternatives |
|-----------|-------------|--------------|
| Scheduler | AWS EventBridge | CloudWatch Events, Cron |
| Workers | AWS Lambda | Cloud Functions, Containers |
| Time-series DB | InfluxDB / TimescaleDB | Prometheus, CloudWatch |
| API | Express / Hono | FastAPI, tRPC |
| Alerting | SNS + Webhooks | PagerDuty, Opsgenie |

---

## Roadmap

### Phase 1: Foundation (Month 1-2)

| Task | Priority | Effort |
|------|----------|--------|
| Expand existing watchtower for Tier 1 checks | 🔴 P0 | Medium |
| Add multi-region monitoring | 🔴 P0 | Medium |
| Implement time-series storage | 🔴 P0 | Medium |
| Build basic health API | 🔴 P0 | Medium |
| Integrate health into Trust Score | 🟡 P1 | Low |

### Phase 2: Compliance (Month 2-3)

| Task | Priority | Effort |
|------|----------|--------|
| MCP compliance checker | 🔴 P0 | Medium |
| A2A compliance checker | 🔴 P0 | Medium |
| Compliance API endpoints | 🟡 P1 | Low |
| Opt-in mechanism for Tier 2 | 🟡 P1 | Low |

### Phase 3: Performance (Month 3-4)

| Task | Priority | Effort |
|------|----------|--------|
| Benchmark test suite | 🟡 P1 | High |
| Payment integration for Tier 3 | 🟡 P1 | Medium |
| Certification badge system | 🟡 P1 | Low |
| Performance leaderboards | 🟢 P2 | Low |

### Phase 4: Ecosystem (Month 4-6)

| Task | Priority | Effort |
|------|----------|--------|
| Webhook notification system | 🟡 P1 | Medium |
| Auditor marketplace foundation | 🟢 P2 | High |
| Public documentation | 🟡 P1 | Medium |
| SDK integration | 🟡 P1 | Medium |

---

## Summary

### What Watchtower Is

- Continuous health monitoring for agents
- Objective, hard-to-fake metrics
- Protocol compliance verification
- One of three pillars in Trust Score

### What Watchtower Is NOT

- Quality/capability evaluation (leave to reputation + auditors)
- A replacement for user feedback
- A guarantee of agent safety or accuracy

### The Value Proposition

> **Watchtower answers: "Is this agent working right now?"**
> 
> Reputation answers: "Did users like this agent in the past?"
> 
> Together, they tell a complete trust story.

---

*Last updated: January 2026*
