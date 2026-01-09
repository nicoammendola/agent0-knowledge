# Agent0 TypeScript SDK - Technical Discovery

**Version:** Based on agent0-ts SDK v0.31 and ERC-8004 v1.0 (Jan 2026)  
**Last Updated:** January 10, 2026  
**Status:** Technical Analysis - SDK needs updates for ERC-8004 v1.0 compatibility

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Architecture Overview](#architecture-overview)
3. [Core Components Deep Dive](#core-components-deep-dive)
4. [ERC-8004 v1.0 Changes](#erc-8004-v10-changes)
5. [SDK Misalignments with v1.0](#sdk-misalignments-with-v10)
6. [Data Flows](#data-flows)
7. [Smart Contract Interfaces](#smart-contract-interfaces)
8. [Key Design Patterns](#key-design-patterns)
9. [Migration Guide](#migration-guide)

---

## Executive Summary

The **agent0-ts SDK** is a TypeScript implementation for building **agentic economies** based on the **ERC-8004** Ethereum standard. It enables AI agents to:
- Register on-chain as ERC-721 NFTs
- Advertise capabilities via MCP and A2A protocols
- Be discovered permissionlessly through subgraph indexing
- Build reputation through feedback

### ⚠️ Critical: ERC-8004 v1.0 Breaking Changes

The ERC-8004 specification was updated on January 9, 2026 with **breaking changes** that render portions of the current SDK obsolete:

| Change | Impact | Severity |
|--------|--------|----------|
| `feedbackAuth` removed | SDK's entire auth signing system is now unused | 🔴 **CRITICAL** |
| Tags changed from `bytes32` to `string` | SDK encoding/decoding broken | 🔴 **CRITICAL** |
| New `endpoint` field in feedback | SDK missing required parameter | 🟡 **HIGH** |
| `agentWallet` requires EIP-712 signature | SDK's simple metadata approach won't work | 🟡 **HIGH** |
| Parameter renames (`tokenURI` → `agentURI`, etc.) | ABI incompatibility | 🟡 **HIGH** |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SDK (Entry Point)                            │
│                        src/core/sdk.ts                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐   │
│  │    Agent     │  │ FeedbackMgr  │  │     AgentIndexer         │   │
│  │   agent.ts   │  │feedback-mgr  │  │     indexer.ts           │   │
│  │ (Lifecycle)  │  │ (Reputation) │  │ (Search & Discovery)     │   │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────────┘   │
│         │                 │                      │                  │
│  ┌──────┴─────────────────┴──────────────────────┴───────────────┐  │
│  │                    Client Layer                               │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────────────────┐   │  │
│  │  │ Web3Client │  │ IPFSClient │  │   SubgraphClient       │   │  │
│  │  │ (ethers.js)│  │ (Pinata)   │  │   (GraphQL)            │   │  │
│  │  └─────┬──────┘  └─────┬──────┘  └──────────┬─────────────┘   │  │
│  └────────┼───────────────┼────────────────────┼─────────────────┘  │
│           │               │                    │                    │
└───────────┼───────────────┼────────────────────┼────────────────────┘
            │               │                    │
     ┌──────▼──────┐ ┌──────▼──────┐  ┌──────────▼──────────┐
     │ Smart       │ │    IPFS     │  │    The Graph        │
     │ Contracts   │ │  (Pinata/   │  │    (Subgraph)       │
     │ (ERC-8004)  │ │  Filecoin)  │  │                     │
     └─────────────┘ └─────────────┘  └─────────────────────┘
```

### File Structure

```
agent0-ts/
├── src/
│   ├── core/
│   │   ├── sdk.ts              # Main entry point, orchestrates all components
│   │   ├── agent.ts            # Agent lifecycle management
│   │   ├── web3-client.ts      # Blockchain interactions via ethers.js
│   │   ├── ipfs-client.ts      # IPFS storage (Pinata, Filecoin, local)
│   │   ├── subgraph-client.ts  # GraphQL queries to The Graph
│   │   ├── feedback-manager.ts # Reputation system ⚠️ NEEDS UPDATE
│   │   ├── indexer.ts          # Multi-chain search orchestration
│   │   ├── endpoint-crawler.ts # MCP/A2A capability extraction
│   │   ├── oasf-validator.ts   # OASF taxonomy validation
│   │   └── contracts.ts        # ABIs and contract addresses ⚠️ NEEDS UPDATE
│   ├── models/
│   │   ├── interfaces.ts       # Core data structures
│   │   ├── types.ts            # Type aliases
│   │   └── enums.ts            # EndpointType, TrustModel
│   └── utils/
│       ├── constants.ts        # Timeouts, gateways
│       ├── id-format.ts        # AgentId parsing utilities
│       └── validation.ts       # Address normalization
```

---

## Core Components Deep Dive

### 1. SDK Class (`src/core/sdk.ts`)

The main entry point that wires everything together:

```typescript
export class SDK {
  // Internal clients
  private readonly _web3Client: Web3Client;      // Blockchain interactions
  private _ipfsClient?: IPFSClient;              // Decentralized storage
  private _subgraphClient?: SubgraphClient;      // Fast queries via The Graph
  private readonly _feedbackManager: FeedbackManager;  // Reputation system
  private readonly _indexer: AgentIndexer;       // Search & discovery
  
  // Lazy-loaded contract instances
  private _identityRegistry?: ethers.Contract;   // ERC-721 NFT registry
  private _reputationRegistry?: ethers.Contract; // Feedback storage
  private _validationRegistry?: ethers.Contract; // Third-party verification
}
```

**Key Methods:**

| Method | Purpose | V1.0 Status |
|--------|---------|-------------|
| `createAgent(name, desc)` | Creates in-memory Agent with RegistrationFile | ✅ OK |
| `loadAgent(agentId)` | Fetches token URI, loads registration from IPFS | ✅ OK |
| `searchAgents(params)` | Multi-chain subgraph queries | ✅ OK |
| `giveFeedback(agentId, feedbackFile)` | Submit feedback | ⚠️ **NEEDS UPDATE** |
| `signFeedbackAuth(...)` | Sign authorization | ❌ **OBSOLETE** |
| `getReputationSummary(agentId)` | Aggregate feedback scores | ⚠️ **NEEDS UPDATE** |

---

### 2. Agent Class (`src/core/agent.ts`)

Manages individual agent lifecycle:

```typescript
export class Agent {
  private registrationFile: RegistrationFile;  // The agent's "card"
  private _endpointCrawler: EndpointCrawler;   // Auto-extracts capabilities
  private _dirtyMetadata = new Set<string>();  // Tracks what changed
  private _lastRegisteredWallet?: Address;     // For wallet change detection
}
```

**Agent Lifecycle Flow:**

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   createAgent   │───▶│   Configure     │───▶│   registerIPFS  │
│   (in memory)   │    │   Endpoints     │    │   (on-chain)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │                       │
                              ▼                       ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │ setMCP/setA2A   │    │ 1. Upload IPFS  │
                       │ (auto-crawl)    │    │ 2. Call register│
                       └─────────────────┘    │ 3. setAgentUri  │
                                              └─────────────────┘
```

**Key Methods:**

| Method | Purpose | V1.0 Status |
|--------|---------|-------------|
| `setMCP(endpoint, version, autoFetch)` | Configure MCP, auto-crawl tools | ✅ OK |
| `setA2A(agentcard, version, autoFetch)` | Configure A2A, auto-crawl skills | ✅ OK |
| `setAgentWallet(address, chainId)` | Set payment wallet | ⚠️ **NEEDS UPDATE** |
| `addSkill(slug, validateOASF)` | Add OASF skill | ✅ OK |
| `addDomain(slug, validateOASF)` | Add OASF domain | ✅ OK |
| `registerIPFS()` | Register/update on-chain | ⚠️ **PARAM RENAME** |

---

### 3. Web3Client (`src/core/web3-client.ts`)

Handles all Ethereum interactions using ethers.js v6:

```typescript
export class Web3Client {
  public readonly provider: JsonRpcProvider;
  public readonly signer?: Wallet | Signer;
  public chainId: bigint;
  
  // Read operations (free)
  async callContract(contract, methodName, ...args): Promise<any>
  
  // Write operations (requires gas + signer)
  async transactContract(contract, methodName, options, ...args): Promise<string>
  
  // Smart router for register() overloads
  private async registerAgent(contract, options, ...args): Promise<string>
}
```

**Special Feature - Register Function Overload Resolution:**

The contract has 3 `register()` overloads. The SDK intelligently routes:

```typescript
// The registerAgent router selects the correct ABI function
if (args.length === 0) {
  functionName = 'register()';
} else if (args.length === 1 && typeof args[0] === 'string') {
  functionName = 'register(string)';
} else if (args.length === 2) {
  functionName = 'register(string,(string,bytes)[])';
}
// Encodes data directly to bypass ethers function resolution
const data = contractInterface.encodeFunctionData(functionFragment, callArgs);
```

---

### 4. IPFSClient (`src/core/ipfs-client.ts`)

Supports 3 storage providers:

| Provider | Config | How it Works |
|----------|--------|--------------|
| **Pinata** | `ipfs: 'pinata', pinataJwt` | HTTP POST to Pinata v3 API |
| **Filecoin Pin** | `ipfs: 'filecoinPin', filecoinPrivateKey` | (Placeholder - CLI needed) |
| **Local Node** | `ipfs: 'node', ipfsNodeUrl` | ipfs-http-client library |

**Registration File Format (ERC-8004 compliant):**

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My AI Agent",
  "description": "An intelligent assistant",
  "endpoints": [
    {
      "name": "MCP",
      "endpoint": "https://mcp.example.com/",
      "version": "2025-06-18",
      "mcpTools": ["code_gen", "search"],
      "capabilities": []
    },
    {
      "name": "A2A", 
      "endpoint": "https://a2a.example.com/agent-card.json",
      "version": "0.30",
      "a2aSkills": ["python", "data-analysis"]
    },
    {
      "name": "OASF",
      "endpoint": "https://github.com/agntcy/oasf/",
      "version": "v0.8.0",
      "skills": ["data_engineering/data_transformation_pipeline"],
      "domains": ["finance_and_business/investment_services"]
    }
  ],
  "registrations": [
    { "agentId": 123, "agentRegistry": "eip155:11155111:0x8004..." }
  ],
  "supportedTrust": ["reputation"],
  "active": true,
  "x402support": false
}
```

---

### 5. FeedbackManager (`src/core/feedback-manager.ts`) ⚠️ NEEDS MAJOR UPDATE

This component is most affected by ERC-8004 v1.0 changes:

```typescript
export class FeedbackManager {
  // ❌ OBSOLETE in v1.0 - feedbackAuth removed
  async signFeedbackAuth(agentId, clientAddress, indexLimit, expiryHours): Promise<string>
  
  // ⚠️ NEEDS UPDATE - missing 'endpoint' field, tag encoding changed
  async giveFeedback(agentId, feedbackFile, idem?, feedbackAuth?): Promise<Feedback>
  
  // ⚠️ NEEDS UPDATE - tags now return string instead of bytes32
  async getFeedback(agentId, clientAddress, feedbackIndex): Promise<Feedback>
}
```

**Current (BROKEN) Feedback Flow:**

```typescript
// SDK currently does this:
const tag1 = this._stringToBytes32(tag1Str);  // ❌ Wrong - should be string
const tag2 = this._stringToBytes32(tag2Str);  // ❌ Wrong - should be string

await this.web3Client.transactContract(
  this.reputationRegistry,
  'giveFeedback',
  {},
  BigInt(tokenId),
  score,
  tag1,           // ❌ bytes32 - should be string
  tag2,           // ❌ bytes32 - should be string
  feedbackUri,
  feedbackHash,
  ethers.getBytes(authBytes)  // ❌ REMOVED in v1.0
);
```

---

### 6. SubgraphClient (`src/core/subgraph-client.ts`)

GraphQL client for The Graph indexed data:

```typescript
export class SubgraphClient {
  // Search agents with WHERE clause pushed to subgraph
  async searchAgents(params, first, skip): Promise<AgentSummary[]>
  
  // Get single agent by ID
  async getAgentById(agentId): Promise<AgentSummary | null>
  
  // Search feedback with complex filters
  async searchFeedback(params, first, skip, orderBy, orderDirection): Promise<any[]>
}
```

**Filtering Strategy:**

| Filter Type | Where Applied | Efficiency |
|-------------|---------------|------------|
| `active`, `x402support`, `mcp`, `a2a` | Subgraph WHERE | ✅ Efficient |
| `owner`, `operators` | Subgraph WHERE | ✅ Efficient |
| `name` (substring) | Client-side | ⚠️ Less efficient |
| `a2aSkills`, `mcpTools` (array contains) | Client-side | ⚠️ Less efficient |

---

### 7. EndpointCrawler (`src/core/endpoint-crawler.ts`)

Auto-extracts capabilities from MCP and A2A endpoints:

**MCP Capability Extraction:**
```typescript
// Parallel JSON-RPC calls to MCP server
const [tools, resources, prompts] = await Promise.all([
  this._jsonRpcCall(httpUrl, 'tools/list'),
  this._jsonRpcCall(httpUrl, 'resources/list'),
  this._jsonRpcCall(httpUrl, 'prompts/list'),
]);
```

**A2A Skill Extraction:**
```typescript
// Tries multiple well-known paths in order:
const agentcardUrls = [
  endpoint,                                   // Direct URL (ERC-8004 format)
  `${endpoint}/.well-known/agent-card.json`, // Spec-recommended
  `${endpoint}/.well-known/agent.json`,      // Alternative
  `${endpoint}/agentcard.json`,              // Legacy
];
// Extracts tags from skills array in agent card
```

---

## ERC-8004 v1.0 Changes

### Summary of Changes (January 9, 2026)

The ERC-8004 specification received major updates based on community feedback from the testnet phase.

### 1. 🔴 CRITICAL: Feedback Authorization Removed

**What Changed:**
- The entire `feedbackAuth` system has been removed
- Any client can now directly call `giveFeedback()` without pre-authorization
- Spam resistance now relies on filtering by reviewer and off-chain aggregation

**Old Spec:**
```solidity
function giveFeedback(
  uint256 agentId, 
  uint8 score, 
  bytes32 tag1,      // ← bytes32
  bytes32 tag2,      // ← bytes32
  string fileuri, 
  bytes32 filehash, 
  bytes feedbackAuth // ← REMOVED
) external
```

**New Spec (v1.0):**
```solidity
function giveFeedback(
  uint256 agentId, 
  uint8 score, 
  string tag1,       // ← Now string
  string tag2,       // ← Now string
  string endpoint,   // ← NEW
  string feedbackURI,
  bytes32 feedbackHash
) external
```

### 2. 🔴 CRITICAL: Tags Changed from bytes32 to string

All feedback functions now use `string` tags instead of `bytes32`:

| Function          | Old                          | New                        |
|-------------------|------------------------------|----------------------------|
| `giveFeedback`    | `bytes32 tag1, bytes32 tag2` | `string tag1, string tag2` |
| `readFeedback`    | returns `bytes32, bytes32`   | returns `string, string`   |
| `readAllFeedback` | returns `bytes32[]`          | returns `string[]`         |
| `getSummary`      | `bytes32 tag1, bytes32 tag2` | `string tag1, string tag2` |

### 3. 🟡 HIGH: New `endpoint` Field in Feedback

A new optional `endpoint` URI parameter is now part of on-chain feedback:

```solidity
function giveFeedback(
  uint256 agentId, 
  uint8 score, 
  string tag1, 
  string tag2,
  string endpoint,    // ← NEW: URI of the endpoint being reviewed
  string feedbackURI,
  bytes32 feedbackHash
) external
```

### 4. 🟡 HIGH: Agent Wallet Now Requires Signature Verification

**Old Approach (SDK current):**
```typescript
agent.setMetadata({ agentWallet: 'eip155:11155111:0x...' });
```

**New Spec (v1.0):**
- `agentWallet` is a **reserved** metadata key
- Cannot be set via `setMetadata()` or during `register()`
- Initially set to owner's address
- Must use `setAgentWallet()` with EIP-712 signature to change:

```solidity
function setAgentWallet(
  uint256 agentId, 
  address newWallet, 
  uint256 deadline, 
  bytes calldata signature  // EIP-712 or ERC-1271
) external
```

- On transfer, `agentWallet` resets to zero address

### 5. 🟡 HIGH: Parameter Renames

| Old Name | New Name | Used In |
|----------|----------|---------|
| `tokenURI` | `agentURI` | register(), Registered event |
| `fileuri` | `feedbackURI` | giveFeedback() |
| `filehash` | `feedbackHash` | giveFeedback() |
| `index` | `feedbackIndex` | readFeedback(), NewFeedback event |
| `responseUri` | `responseURI` | appendResponse() |

### 6. 🟢 NEW: URIUpdated Event

```solidity
event URIUpdated(uint256 indexed agentId, string newURI, address indexed updatedBy)
```

### 7. 🟢 NEW: NewFeedback Event Expanded

```solidity
event NewFeedback(
  uint256 indexed agentId, 
  address indexed clientAddress, 
  uint64 feedbackIndex,        // ← NEW
  uint8 score, 
  string indexed tag1,         // ← Now string
  string tag2,                 // ← Now string
  string endpoint,             // ← NEW
  string feedbackURI, 
  bytes32 feedbackHash
)
```

### 8. 🟢 NEW: readAllFeedback Returns feedbackIndexes

```solidity
function readAllFeedback(...) external view returns (
  address[] memory clients,
  uint64[] memory feedbackIndexes,  // ← NEW
  uint8[] memory scores,
  string[] memory tag1s,            // ← Now string[]
  string[] memory tag2s,            // ← Now string[]
  bool[] memory revokedStatuses
)
```

### 9. 🟢 NEW: Off-Chain Feedback File Changes

```jsonc
{
  // MUST FIELDS
  "agentRegistry": "eip155:1:{identityRegistry}",
  "agentId": 22,
  "clientAddress": "eip155:1:{clientAddress}",
  "createdAt": "2025-09-23T12:00:00Z",
  "score": 100,

  // REMOVED: "feedbackAuth" is no longer required

  // OPTIONAL FIELDS
  "tag1": "foo",
  "tag2": "bar",
  "endpoint": "https://.../",              // ← NEW
  "skill": "as-defined-by-A2A-or-OASF",
  "domain": "as-defined-by-OASF",          // ← NEW
  "proofOfPayment": { ... },               // ← Renamed from proof_of_payment
  "capability": "tools",
  "name": "foo"
}
```

## Data Flows

### Registration Flow (Current - Still Valid)

```
User Code                   SDK                        External Services
─────────                   ───                        ─────────────────
    │
    │ sdk.createAgent("AI", "desc")
    ├──────────────────────────▶ Creates Agent with empty RegistrationFile
    │
    │ agent.setMCP("https://...")
    ├──────────────────────────▶ EndpointCrawler fetches tools/prompts/resources
    │                           ◀───────────────────────────────── JSON-RPC
    │
    │ agent.setA2A("https://...")
    ├──────────────────────────▶ EndpointCrawler fetches skills
    │                           ◀───────────────────────────────── HTTP GET
    │
    │ agent.registerIPFS()
    ├──────────────────────────▶ 1. IPFSClient.addRegistrationFile()
    │                           ◀───────────────────────────────── Pinata → CID
    │                           2. Web3Client.transactContract("register")
    │                           ◀───────────────────────────────── Blockchain → tokenId
    │                           3. Extract agentId from Transfer event
    │                           4. Web3Client.transactContract("setAgentUri")
    │                           ◀───────────────────────────────── Blockchain → confirmed
    │
    │ Returns: { agentId: "11155111:123", agentURI: "ipfs://Qm..." }
    ◀──────────────────────────
```

### Feedback Flow - CURRENT (BROKEN)

```
User Code                   SDK                        External Services
─────────                   ───                        ─────────────────
    │
    │ sdk.prepareFeedback(agentId, 85, ["great"])
    ├──────────────────────────▶ Creates feedback file object
    │
    │ sdk.giveFeedback(agentId, feedbackFile)
    ├──────────────────────────▶ 1. signFeedbackAuth()      ❌ OBSOLETE
    │                           2. _stringToBytes32(tags)   ❌ WRONG TYPE
    │                           3. Upload to IPFS
    │                           4. transactContract("giveFeedback", ..., auth)
    │                              ▲ FAILS - signature mismatch
```

### Feedback Flow - SHOULD BE (v1.0)

```
User Code                   SDK                        External Services
─────────                   ───                        ─────────────────
    │
    │ sdk.prepareFeedback(agentId, 85, ["great"], null, null, null, null, "https://endpoint")
    ├──────────────────────────▶ Creates feedback file with endpoint
    │
    │ sdk.giveFeedback(agentId, feedbackFile)
    ├──────────────────────────▶ 1. Upload to IPFS (feedbackFile)
    │                           ◀───────────────────────────────── CID
    │                           2. transactContract("giveFeedback",
    │                              agentId, score, tag1, tag2, endpoint,
    │                              feedbackURI, feedbackHash)
    │                           ◀───────────────────────────────── Blockchain
    │
    │ Returns: Feedback object
    ◀──────────────────────────
```

---

## Smart Contract Interfaces

### Identity Registry - v1.0

```solidity
// Registration
function register() external returns (uint256 agentId)
function register(string agentURI) external returns (uint256 agentId)
function register(string agentURI, MetadataEntry[] metadata) external returns (uint256 agentId)

// URI Management
function setAgentURI(uint256 agentId, string newURI) external
// Emits: URIUpdated(agentId, newURI, updatedBy)

// Agent Wallet (NEW in v1.0)
function getAgentWallet(uint256 agentId) external view returns (address)
function setAgentWallet(uint256 agentId, address newWallet, uint256 deadline, bytes signature) external
// Requires EIP-712 or ERC-1271 signature

// Metadata
function getMetadata(uint256 agentId, string metadataKey) external view returns (bytes)
function setMetadata(uint256 agentId, string metadataKey, bytes metadataValue) external
// NOTE: Cannot set "agentWallet" via setMetadata - it's reserved

// Events
event Registered(uint256 indexed agentId, string agentURI, address indexed owner)
event URIUpdated(uint256 indexed agentId, string newURI, address indexed updatedBy)
event MetadataSet(uint256 indexed agentId, string indexed indexedMetadataKey, string metadataKey, bytes metadataValue)
```

### Reputation Registry - v1.0

```solidity
// Feedback Submission (CHANGED in v1.0)
function giveFeedback(
  uint256 agentId,
  uint8 score,           // 0-100
  string tag1,           // ← Changed from bytes32
  string tag2,           // ← Changed from bytes32
  string endpoint,       // ← NEW
  string feedbackURI,    // ← Renamed from fileuri
  bytes32 feedbackHash   // ← Renamed from filehash
) external
// NO feedbackAuth parameter

// Feedback Management
function revokeFeedback(uint256 agentId, uint64 feedbackIndex) external
function appendResponse(uint256 agentId, address clientAddress, uint64 feedbackIndex, 
                       string responseURI, bytes32 responseHash) external

// Read Functions (CHANGED in v1.0)
function readFeedback(uint256 agentId, address clientAddress, uint64 feedbackIndex) 
  external view returns (uint8 score, string tag1, string tag2, bool isRevoked)
  
function readAllFeedback(uint256 agentId, address[] clientAddresses, string tag1, string tag2, bool includeRevoked)
  external view returns (
    address[] clients,
    uint64[] feedbackIndexes,  // ← NEW
    uint8[] scores,
    string[] tag1s,            // ← Changed from bytes32[]
    string[] tag2s,            // ← Changed from bytes32[]
    bool[] revokedStatuses
  )

function getSummary(uint256 agentId, address[] clientAddresses, string tag1, string tag2)
  external view returns (uint64 count, uint8 averageScore)

function getClients(uint256 agentId) external view returns (address[])
function getLastIndex(uint256 agentId, address clientAddress) external view returns (uint64)

// Events (CHANGED in v1.0)
event NewFeedback(
  uint256 indexed agentId,
  address indexed clientAddress,
  uint64 feedbackIndex,        // ← NEW
  uint8 score,
  string indexed tag1,         // ← Changed from bytes32
  string tag2,                 // ← Changed from bytes32
  string endpoint,             // ← NEW
  string feedbackURI,
  bytes32 feedbackHash
)
event FeedbackRevoked(uint256 indexed agentId, address indexed clientAddress, uint64 indexed feedbackIndex)
event ResponseAppended(uint256 indexed agentId, address indexed clientAddress, uint64 feedbackIndex, 
                      address indexed responder, string responseURI, bytes32 responseHash)
```

---

## Key Design Patterns

### 1. Lazy Initialization
Contracts and clients created on first use:
```typescript
getIdentityRegistry(): ethers.Contract {
  if (!this._identityRegistry) {
    this._identityRegistry = this._web3Client.getContract(address, ABI);
  }
  return this._identityRegistry;
}
```

### 2. Soft Fail Pattern
Capability extraction fails gracefully:
```typescript
try {
  const capabilities = await this._endpointCrawler.fetchMcpCapabilities(endpoint);
  if (capabilities) { /* use them */ }
} catch (error) {
  // Soft fail - continue without capabilities
}
```

### 3. Dirty Tracking
Only changed metadata written to chain:
```typescript
private _dirtyMetadata = new Set<string>();
// Only send transactions for changed keys
if (this._dirtyMetadata.has(entry.key)) {
  await transactContract(registry, 'setMetadata', tokenId, key, value);
}
```

### 4. Method Chaining
Fluent API for agent configuration:
```typescript
agent
  .setMCP("https://...")
  .setAgentWallet("0x...", 11155111)
  .setActive(true)
  .setTrust(true, false, false);
```

---

## Appendix: Deployed Contract Addresses

### Current SDK Addresses (May need new deployments)

| Chain | Identity Registry | Reputation Registry | Validation Registry |
|-------|-------------------|---------------------|---------------------|
| ETH Sepolia (11155111) | `0x8004a6090Cd10A7288092483047B097295Fb8847` | `0x8004B8FD1A363aa02fDC07635C0c5F94f6Af5B7E` | `0x8004CB39f29c09145F24Ad9dDe2A108C1A2cdfC5` |
| Base Sepolia (84532) | `0x8004AA63c570c570eBF15376c0dB199918BFe9Fb` | `0x8004bd8daB57f14Ed299135749a5CB5c42d341BF` | `0x8004C269D0A5647E51E121FeB226200ECE932d55` |
| Polygon Amoy (80002) | `0x8004ad19E14B9e0654f73353e8a0B600D46C2898` | `0x8004B12F4C2B42d00c46479e859C92e39044C930` | `0x8004C11C213ff7BaD36489bcBDF947ba5eee289B` |
| Linea Sepolia (59141) | `0x8004aa7C931bCE1233973a0C6A667f73F66282e7` | `0x8004bd8483b99310df121c46ED8858616b2Bba02` | `0x8004c44d1EFdd699B2A26e781eF7F77c56A9a4EB` |

> **Note:** These contracts use the old ABI. New v1.0 contracts may need to be deployed, or existing contracts upgraded if they support UUPS proxy pattern.

---

## Questions for Further Investigation

1. Are the existing contracts UUPS upgradeable to v1.0?
2. Will the subgraph schema need updates for string tags?
3. How should gas sponsorship (EIP-7702) be implemented for frictionless feedback?
4. What's the migration path for existing agents/feedback?

---

*Document generated from technical analysis of agent0-ts SDK and ERC-8004 v1.0 specification.*
