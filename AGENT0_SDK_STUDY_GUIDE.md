# Agent0 SDK Study Guide

**A Comprehensive Guide to Understanding Agent0 SDK and ERC-8004**

---

## Table of Contents

1. [Introduction](#introduction)
2. [What is Agent0 SDK?](#what-is-agent0-sdk)
3. [What is an Agent?](#what-is-an-agent)
4. [A2A Protocol](#a2a-protocol)
5. [Sandbox & Testnet Environments](#sandbox--testnet-environments)
6. [How It Works on Blockchain/Web3](#how-it-works-on-blockchainweb3)
7. [IPFS & The Graph](#ipfs--the-graph)
8. [Infrastructure Costs](#infrastructure-costs)
9. [ERC-8004 Standard](#erc-8004-standard)
10. [Versioning & Updates](#versioning--updates)
11. [Quick Reference](#quick-reference)

---

## Introduction

Agent0 SDK is a Python SDK for building **agentic economies** - enabling AI agents to register, discover each other, and build reputation using blockchain infrastructure (ERC-8004) and decentralized storage.

**Key Concept**: Permissionless discovery and trust for AI agents without relying on proprietary catalogues or intermediaries.

---

## What is Agent0 SDK?

### Overview

Agent0 SDK enables you to:

- ✅ **Create and manage agent identities** - Register AI agents on-chain with unique identities
- ✅ **Advertise agent capabilities** - Publish MCP and A2A endpoints with automated capability extraction
- ✅ **OASF taxonomies** - Advertise standardized skills and domains
- ✅ **Enable permissionless discovery** - Make agents discoverable by other agents and platforms
- ✅ **Build reputation** - Give and receive feedback with cryptographic authentication
- ✅ **Cross-chain operations** - Deploy and manage agents across multiple blockchain networks

### Architecture

```
┌──────────────────────────────────────────┐
│           Agent0 SDK Architecture        │
├──────────────────────────────────────────┤
│                                          │
│  ┌──────────────┐      ┌──────────────┐  │
│  │    IPFS      │      │  Blockchain  │  │
│  │  (Storage)   │◄────►│  (On-chain)  │  │
│  └──────────────┘      └──────────────┘  │
│         │                      │         │
│         │                      │         │
│         ▼                      ▼         │
│  ┌────────────────────────────────────┐  │
│  │         The Graph (Subgraph)       │  │
│  │         (Indexing & Search)        │  │
│  └────────────────────────────────────┘  │
│                                          │
└──────────────────────────────────────────┘
```

### Core Components

1. **SDK Class** - Main entry point for all operations
2. **Agent Class** - Represents individual agents with registration data
3. **Web3Client** - Handles smart contract interactions
4. **IPFSClient** - Manages decentralized storage
5. **SubgraphClient** - Queries indexed data
6. **FeedbackManager** - Handles reputation system
7. **AgentIndexer** - Provides search and discovery

---

## What is an Agent?

### Definition

An **agent** in Agent0 SDK is any entity that can be registered on-chain. Technically, any application can qualify as an agent, but the SDK is designed for **AI agents and agentic systems**.

### Minimum Requirements

To register an agent, you only need:

- ✅ A `name` (string)
- ✅ A `description` (string)

That's it! The SDK is very permissive.

### What Makes It Useful?

To be discoverable and useful in the Agent0 ecosystem, an agent should:

1. **Advertise capabilities** via endpoints:
   - MCP endpoint (tools, prompts, resources)
   - A2A endpoint (skills)
   - OASF skills/domains

2. **Be discoverable** through search by capabilities

3. **Accept feedback** to build reputation

4. **Support interoperability** protocols (MCP/A2A)

### Example: Registering an Agent

```python
from agent0_sdk import SDK

# Initialize SDK
sdk = SDK(
    chainId=11155111,  # Ethereum Sepolia testnet
    rpcUrl="https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY",
    signer="YOUR_PRIVATE_KEY",
    ipfs="pinata",
    pinataJwt="YOUR_PINATA_JWT"
)

# Create agent
agent = sdk.createAgent(
    name="My AI Agent",
    description="An intelligent assistant for various tasks"
)

# Configure endpoints (makes it discoverable)
agent.setMCP("https://mcp.example.com/")
agent.setA2A("https://a2a.example.com/agent-card.json")

# Register on-chain
agent.registerIPFS()
print(f"Agent registered: {agent.agentId}")  # e.g., "11155111:123"
```

---

## A2A Protocol

### What is A2A?

**A2A** stands for **"Agent-to-Agent"** - a standardized protocol that enables autonomous agents to communicate and collaborate directly with each other.

### How A2A Works

A2A uses **"agent cards"** (JSON files) that describe an agent's capabilities:

```json
{
  "name": "Data Analysis Agent",
  "description": "An agent specialized in data processing",
  "skills": [
    {
      "name": "Data Processing",
      "tags": ["python", "pandas", "data-analysis", "csv"]
    },
    {
      "name": "API Integration",
      "tags": ["rest-api", "http", "json"]
    }
  ]
}
```

### A2A vs MCP

| Feature | A2A | MCP |
|---------|-----|-----|
| **Full Name** | Agent-to-Agent | Model Context Protocol |
| **Purpose** | Agent discovery & collaboration | Tool/resource access |
| **Capabilities** | Skills (tags) | Tools, Prompts, Resources |
| **Format** | Agent cards (JSON) | JSON-RPC over HTTP |
| **Use Case** | "What can this agent do?" | "What tools/resources does it have?" |

### Using A2A in Agent0 SDK

```python
# Set an A2A endpoint
agent.setA2A("https://a2a.example.com/agent-card.json", version="0.30")

# SDK automatically:
# 1. Fetches the agent card from the endpoint
# 2. Extracts skills from the card
# 3. Stores them as a2aSkills for discovery

# Search by A2A skills
results = sdk.searchAgents(a2aSkills=["python", "data-analysis"])
```

---

## Sandbox & Testnet Environments

### Supported Testnets

Agent0 SDK currently runs on **testnets only** (no mainnet yet):

1. **Ethereum Sepolia** (Chain ID: `11155111`)
   - Identity Registry: `0x8004A818BFB912233c491871b3d84c89A494BD9e` (Updated Jan 2026)
   - Reputation Registry: `0x8004B663056A597Dffe9eCcC1965A193B7388713` (Updated Jan 2026)
   - Validation Registry: `0x8004CB39f29c09145F24Ad9dDe2A108C1A2cdfC5`

2. **Base Sepolia** (Chain ID: `84532`)
   - Identity Registry: `0x8004AA63c570c570eBF15376c0dB199918BFe9Fb`
   - Reputation Registry: `0x8004bd8daB57f14Ed299135749a5CB5c42d341BF`
   - Validation Registry: `0x8004C269D0A5647E51E121FeB226200ECE932d55`

3. **Polygon Amoy** (Chain ID: `80002`)
   - Identity Registry: `0x8004ad19E14B9e0654f73353e8a0B600D46C2898`
   - Reputation Registry: `0x8004B12F4C2B42d00c46479e859C92e39044C930`
   - Validation Registry: `0x8004C11C213ff7BaD36489bcBDF947ba5eee289B`

4. **Linea Sepolia** (Chain ID: `59141`)
   - Contracts deployed, but no subgraph configured yet

### Getting Started on Testnet

**Requirements:**

1. **Testnet ETH** (free from faucets):
   - Ethereum Sepolia: https://sepoliafaucet.com/
   - Base Sepolia: https://www.coinbase.com/faucets/base-ethereum-goerli-faucet
   - Polygon Amoy: https://faucet.polygon.technology/

2. **RPC Endpoint** (free):
   - Alchemy: https://www.alchemy.com/
   - Infura: https://www.infura.io/

3. **IPFS** (optional, free for ERC-8004):
   - Pinata: https://www.pinata.cloud/ (free tier)
   - Filecoin Pin: Free for ERC-8004 agents

**Example Setup:**

```python
from agent0_sdk import SDK
import os

sdk = SDK(
    chainId=11155111,  # Ethereum Sepolia
    rpcUrl=os.getenv("RPC_URL"),
    signer=os.getenv("PRIVATE_KEY"),
    ipfs="pinata",
    pinataJwt=os.getenv("PINATA_JWT")
)
```

---

## How It Works on Blockchain/Web3

### Architecture Overview

Agent0 SDK uses **ERC-8004**, an Ethereum standard built on top of existing Ethereum standards.

### Smart Contracts (On-Chain)

Three main contracts per chain:

#### 1. Identity Registry (ERC-721 NFT)

- Each agent is an **ERC-721 NFT**
- Minting creates a unique `agentId` (token ID)
- Stores:
  - Agent URI (IPFS or HTTP link to registration file)
  - Metadata (wallet address, ENS name, custom data)
  - Ownership (who owns the agent)

**Registration Flow:**

```python
agent.registerIPFS()

# Behind the scenes:
# 1. Uploads registration file to IPFS → gets CID
# 2. Calls IdentityRegistry.register(ipfs://CID, metadata)
# 3. Mints ERC-721 NFT → returns agentId
# 4. Sets tokenURI to ipfs://CID
```

#### 2. Reputation Registry

- Stores feedback on-chain
- Each feedback entry includes:
  - Score (0-100)
  - Tags (2 tags max, stored as bytes32)
  - Feedback URI (IPFS link to full feedback file)
  - Cryptographic authentication

#### 3. Validation Registry

- For third-party verification of high-risk transactions
- Not actively used yet in Agent0 SDK v0.31

### IPFS (Decentralized Storage)

Registration files and feedback files are stored on IPFS:

```json
// Registration file (stored on IPFS)
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My AI Agent",
  "description": "...",
  "endpoints": [...],
  "skills": [...],
  "domains": [...]
}
```

**Benefits:**
- Decentralized (no single point of failure)
- Immutable (CID changes if content changes)
- Cost-effective (cheaper than storing on-chain)

### The Graph (Subgraph Indexing)

The Graph indexes blockchain and IPFS data for fast queries:

- Indexes all agent registrations
- Indexes all feedback
- Provides GraphQL API for search
- Enables complex queries without direct blockchain calls

**Example:**

```python
# Fast search (uses subgraph, not blockchain)
results = sdk.searchAgents(
    name="AI",
    a2aSkills=["python"],
    active=True
)
# This queries The Graph, not the blockchain directly
```

### Registration Flow

```python
# Step 1: Create agent (off-chain, in memory)
agent = sdk.createAgent(name="My Agent", description="...")

# Step 2: Configure agent (off-chain)
agent.setMCP("https://mcp.example.com/")
agent.setA2A("https://a2a.example.com/agent-card.json")

# Step 3: Register on-chain
agent.registerIPFS()

# What happens:
# 1. SDK uploads registration file to IPFS → gets CID
# 2. SDK calls IdentityRegistry.register("ipfs://CID", metadata)
# 3. Smart contract mints ERC-721 NFT → returns tokenId
# 4. SDK extracts agentId from Transfer event
# 5. SDK calls setAgentUri(tokenId, "ipfs://CID")
# 6. Agent is now registered! agentId = "11155111:123"
```

### Blockchain Operations

**Read Operations (No Gas):**
- Query agent data
- Search agents
- Get feedback
- Read metadata

**Write Operations (Requires Gas):**
- Register agent
- Update agent URI
- Set metadata
- Give feedback
- Transfer ownership

All write operations require:
- Private key (signer)
- Testnet ETH for gas
- Transaction confirmation (~15 seconds on Sepolia)

### Multi-Chain Support

Agents can exist on multiple chains with the same identity:

```python
# Agent ID format: chainId:tokenId
agentId = "11155111:123"  # Ethereum Sepolia
agentId = "84532:456"     # Base Sepolia
agentId = "80002:789"     # Polygon Amoy

# Search across all chains
results = sdk.searchAgents(chains="all")
```

---

## IPFS & The Graph

### What is IPFS?

**IPFS** (InterPlanetary File System) is a distributed storage protocol. It stores files across a peer-to-peer network instead of a single server.

**Key Concepts:**

1. **Content Addressing**: Files identified by CID (Content Identifier) - a hash of the file's content
   - Same content = same CID
   - Change content = different CID
   - CID is immutable

2. **Distributed Storage**: Files stored across many nodes in the IPFS network

3. **Retrieval**: Anyone with the CID can fetch the file from any node that has it

### Where is IPFS Storage Located?

IPFS storage is **distributed**. The SDK supports multiple providers:

#### Option 1: Pinata (Most Common)

**Location**: Pinata's servers (centralized service, distributed via IPFS)

```python
sdk = SDK(
    ipfs="pinata",
    pinataJwt="your-jwt-token"
)
```

**How it works:**
- You upload → Pinata's servers store it
- Pinata pins it (keeps it available)
- Pinata's servers join the IPFS network
- Other IPFS nodes can cache it
- **Free for ERC-8004 agents**

**Physical Location**: Pinata's data centers (AWS, etc.)

#### Option 2: Filecoin Pin

**Location**: Filecoin network (decentralized blockchain storage)

```python
sdk = SDK(
    ipfs="filecoinPin",
    filecoinPrivateKey="your-key"
)
```

**How it works:**
- Files stored on Filecoin miners' hardware
- Miners are paid to store data
- Data is replicated across multiple miners
- **Free for ERC-8004 agents**

**Physical Location**: Distributed across Filecoin miners globally

#### Option 3: Local IPFS Node

**Location**: Your own computer/server

```python
sdk = SDK(
    ipfs="node",
    ipfsNodeUrl="http://localhost:5001"
)
```

**Physical Location**: Your computer/server

### IPFS Gateways (For Reading)

When reading IPFS data, the SDK uses public gateways:

```python
# SDK tries these gateways in order:
gateways = [
    "https://gateway.pinata.cloud/ipfs/{cid}",   # Pinata's gateway
    "https://ipfs.io/ipfs/{cid}",                # Protocol Labs gateway
    "https://dweb.link/ipfs/{cid}",              # Cloudflare gateway
    "https://cloudflare-ipfs.com/ipfs/{cid}"     # Cloudflare gateway
]
```

### Where is The Graph Located?

**The Graph** is a decentralized indexing protocol. Subgraphs are hosted on The Graph Network.

**Subgraph URLs:**

```python
DEFAULT_SUBGRAPH_URLS = {
    11155111: "https://gateway.thegraph.com/api/.../subgraphs/id/...",  # Ethereum Sepolia
    84532: "https://gateway.thegraph.com/api/.../subgraphs/id/...",     # Base Sepolia
    80002: "https://gateway.thegraph.com/api/.../subgraphs/id/...",     # Polygon Amoy
}
```

**Where The Graph Runs:**

1. **The Graph Gateway** (`gateway.thegraph.com`):
   - Hosted by Edge & Node (The Graph Foundation)
   - Serves as entry point for queries
   - Routes to indexers

2. **Indexers** (decentralized):
   - Independent operators run indexer nodes
   - They sync blockchain data and index it
   - They serve queries and earn rewards

3. **Data Location**:
   - Indexers store indexed data on their own infrastructure
   - Data is replicated across multiple indexers
   - Gateway routes queries to available indexers

**What The Graph Indexes:**

1. Blockchain events (agent registrations, feedback, transfers)
2. IPFS data (fetches and indexes registration/feedback files)
3. Aggregated data (reputation scores, statistics, search indexes)

---

## Infrastructure Costs

### Who Pays for Pinata's Infrastructure?

**Short Answer**: Pinata pays for the infrastructure (AWS, data transfer, etc.). They offer a free tier for ERC-8004 agents, and they make money from paid plans and enterprise customers.

### Pinata's Business Model

1. **Free Tier** (Limited):
   - Small storage/bandwidth
   - **Free for ERC-8004 agents** (sponsored)
   - Good for testing and small projects

2. **Paid Tiers**:
   - Starter, Pro, Team plans
   - Pay-as-you-go for storage and bandwidth
   - Higher limits and features

3. **Enterprise**:
   - Custom pricing
   - Dedicated infrastructure
   - SLAs and support

### Why Free for ERC-8004?

Pinata sponsors ERC-8004 agents to:
- Support the ecosystem
- Drive adoption
- Build brand awareness
- Attract users who may upgrade later

### Infrastructure Costs Breakdown

When you use Pinata (even on free tier), Pinata pays for:

- **AWS/Cloud Hosting**: Server instances, storage, database
- **Data Transfer**: Ingress (uploading), egress (downloading), IPFS network traffic
- **IPFS Network**: Running IPFS nodes, bandwidth, pinning service
- **Gateway Service**: HTTP gateway servers, CDN, SSL certificates
- **Operations**: Monitoring, maintenance, support, development

### Cost Comparison

| Service | Who Pays | Cost Model |
|---------|----------|------------|
| Pinata (Free Tier) | Pinata (sponsored) | Free for ERC-8004 agents |
| Pinata (Paid) | You | $10-200+/month based on usage |
| Filecoin Pin | Protocol Labs (sponsored) | Free for ERC-8004 agents |
| Local IPFS Node | You | Your server costs |
| IPFS Gateways | Various orgs | Free (public service) |

### Reading Data (Gateways)

When reading IPFS data, the SDK uses public gateways:

- **Pinata Gateway**: Pinata (part of their service)
- **ipfs.io**: Protocol Labs (public service)
- **Cloudflare**: Cloudflare (public service)

These are **free to use**, but the organizations cover the costs.

---

## ERC-8004 Standard

### What is ERC-8004?

**ERC-8004** (EIP-8004) is an Ethereum Improvement Proposal that creates a **trustless framework** for AI agents to:
- Discover each other
- Interact and collaborate
- Build reputation
- Operate across organizations without prior trust

### The Three Registries

ERC-8004 defines three smart contract registries:

#### 1. Identity Registry (ERC-721 NFT)

Each agent gets a unique ERC-721 token (NFT) as its digital identity:

```python
# When you register an agent:
agent.registerIPFS()

# What happens:
# - Mints an ERC-721 NFT → unique agentId (e.g., "11155111:123")
# - Links to "Agent Card" (registration file) stored on IPFS
# - Contains: name, description, capabilities, endpoints, metadata
```

**Registration File Format:**

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "My AI Agent",
  "description": "...",
  "endpoints": [
    {
      "name": "MCP",
      "endpoint": "https://mcp.example.com/",
      "version": "2025-06-18"
    },
    {
      "name": "A2A",
      "endpoint": "https://a2a.example.com/agent-card.json",
      "version": "0.30"
    }
  ],
  "registrations": [
    {
      "agentId": 123,
      "agentRegistry": "eip155:11155111:0x8004a6090Cd10A7288092483047B097295Fb8847"
    }
  ],
  "supportedTrust": ["reputation", "crypto-economic"],
  "active": true,
  "x402support": false
}
```

#### 2. Reputation Registry

A decentralized feedback system:

```python
# Give feedback to an agent
feedback_file = sdk.prepareFeedback(
    agentId="11155111:123",
    score=85,
    tags=["great", "helpful"]
)
feedback = sdk.giveFeedback("11155111:123", feedback_file)
```

**Features:**
- Structured reviews (score, tags, text, context)
- Payment proof linking (x402)
- Cryptographic authentication
- On-chain storage (immutable)

#### 3. Validation Registry

For third-party verification of high-risk transactions:
- TEE (Trusted Execution Environment) oracles
- Staked inference
- Zero-knowledge ML proofs

**Note**: This registry exists in the contracts but isn't actively used yet in Agent0 SDK v0.31.

### What Makes an Agent "ERC-8004 Compliant"?

An agent is ERC-8004 compliant if it:

1. **Has an Identity Registry registration**:
   - Registered as ERC-721 NFT
   - Has a valid registration file
   - Registration file follows ERC-8004 format

2. **Registration file includes**:
   - `type`: `"https://eips.ethereum.org/EIPS/eip-8004#registration-v1"`
   - `name` and `description`
   - `endpoints` (MCP, A2A, etc.)
   - `registrations` array (links to blockchain)

3. **Can receive feedback**:
   - Reputation Registry integration
   - Feedback authentication

### ERC-8004 vs. Regular Agents

| Feature | Regular Agent | ERC-8004 Agent |
|---------|---------------|----------------|
| **Identity** | Centralized database | ERC-721 NFT (on-chain) |
| **Discovery** | Proprietary catalog | Permissionless, decentralized |
| **Reputation** | Platform-specific | Cross-platform, on-chain |
| **Ownership** | Platform-controlled | NFT ownership (transferable) |
| **Interoperability** | Platform-dependent | Standard protocol (ERC-8004) |
| **Trust** | Requires platform trust | Trustless (blockchain) |

### Contract Addresses

The SDK uses these ERC-8004 contract addresses:

**Ethereum Sepolia:**
- Identity Registry: `0x8004A818BFB912233c491871b3d84c89A494BD9e` (Updated Jan 2026)
- Reputation Registry: `0x8004B663056A597Dffe9eCcC1965A193B7388713` (Updated Jan 2026)
- Validation Registry: `0x8004CB39f29c09145F24Ad9dDe2A108C1A2cdfC5`

**Notice**: All addresses start with `0x8004` - a reference to the ERC-8004 standard number!

### Key Benefits of ERC-8004

1. **Permissionless**: No central authority controls registration
2. **Portable**: Identity and reputation follow the agent across platforms
3. **Verifiable**: On-chain proof of identity and reputation
4. **Interoperable**: Standard format works across implementations
5. **Trustless**: No need to trust intermediaries

---

## Versioning & Updates

### Agent ID Immutability

**Yes, the agent ID is immutable.**

The `agentId` is an ERC-721 token ID (NFT) and **cannot change**:

```python
# First registration
agent.registerIPFS()
agentId = agent.agentId  # e.g., "11155111:123"

# This agentId NEVER changes, even after updates
# It's permanently tied to the ERC-721 NFT
```

**Why Immutable:**
- ERC-721 token IDs are permanent
- Reputation/feedback is tied to this ID
- Ownership transfers use the same ID
- Changing it would break references

### Versioning in ERC-8004

There are multiple version concepts:

#### 1. ERC-8004 Registration File Version

The registration file format version is specified in the `type` field:

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1"
}
```

**Who Decides**: The Ethereum community via the EIP process. Currently `registration-v1`. Future versions (e.g., `registration-v2`) would be defined by a new EIP.

#### 2. Endpoint Protocol Versions

Each endpoint has its own version:

```python
agent.setMCP("https://mcp.example.com/", version="2025-06-18")  # MCP protocol version
agent.setA2A("https://a2a.example.com/", version="0.30")        # A2A protocol version
agent.setENS("myagent.eth", version="1.0")                      # ENS version
```

**Who Decides**:
- **MCP version**: MCP protocol maintainers
- **A2A version**: A2A protocol maintainers
- **ENS version**: ENS protocol maintainers

#### 3. Custom Metadata Version

You can track your own version in metadata:

```python
agent.setMetadata({
    "version": "1.0.0",  # Your own versioning scheme
    "category": "ai-assistant"
})
```

**Who Decides**: **You** (the agent owner). Use any scheme you want (semantic versioning, date-based, etc.).

#### 4. Updated Timestamp

The registration file includes an `updatedAt` timestamp:

```python
registration_file.updatedAt = int(time.time())  # Unix timestamp
```

This tracks when the registration file was last modified.

### Update vs Re-Register

**You should UPDATE the existing registration, not create a new one.**

#### How Updates Work

When an agent is already registered, calling `registerIPFS()` **updates** it:

```python
# Agent already registered with agentId = "11155111:123"
agent = sdk.loadAgent("11155111:123")

# Make improvements
agent.updateInfo(description="Improved description with new capabilities")
agent.setMCP("https://new-mcp.example.com/")
agent.setMetadata({"version": "1.1.0"})  # Update your version

# Update existing registration (NOT re-register)
agent.registerIPFS()  # Updates the same agentId
```

**What Happens During Update:**

```python
if self.registration_file.agentId:
    # Agent already registered - UPDATE existing registration
    # 1. Upload NEW registration file to IPFS → gets NEW CID
    ipfsCid = self.sdk.ipfs_client.addRegistrationFile(...)
    
    # 2. Update agent URI on-chain → points to NEW CID
    #    (old CID still exists on IPFS, but new one is active)
    txHash = self.sdk.web3_client.transact_contract(
        self.sdk.identity_registry,
        "setAgentUri",
        agentId,  # SAME agentId
        f"ipfs://{ipfsCid}"  # NEW CID
    )
    
    # 3. Update metadata on-chain (if changed)
    #    Only sends transactions for changed metadata (saves gas)
```

#### Why Update Instead of Re-Register?

1. ✅ **Preserves identity**: Same `agentId` maintains continuity
2. ✅ **Preserves reputation**: Feedback stays linked to the same ID
3. ✅ **Preserves ownership**: NFT ownership remains intact
4. ✅ **More efficient**: Only updates what changed
5. ✅ **Better UX**: Users can track the same agent over time

### Example: Making Improvements

```python
# Initial registration
agent = sdk.createAgent("My Agent", "Initial version")
agent.setMCP("https://mcp.example.com/", version="2025-06-18")
agent.setMetadata({"version": "1.0.0"})
agent.registerIPFS()
# agentId = "11155111:123"

# Later: Make improvements
agent = sdk.loadAgent("11155111:123")

# Update capabilities
agent.updateInfo(description="Improved agent with new features")
agent.setMCP("https://mcp.example.com/", version="2025-06-18")  # Same endpoint
agent.addSkill("new_skill", validate_oasf=True)
agent.setMetadata({"version": "1.1.0"})  # Bump your version

# Update registration (keeps same agentId)
agent.registerIPFS()
# Still agentId = "11155111:123"
# But registration file on IPFS has new CID
# And updatedAt timestamp is updated
```

### Version Tracking Best Practices

#### 1. Use Metadata for Your Version

```python
agent.setMetadata({
    "version": "1.2.3",  # Semantic versioning
    "build": "2024-01-15",
    "changelog": "Added new MCP tools"
})
```

#### 2. Update `updatedAt` Automatically

The SDK updates `updatedAt` whenever you modify the agent:

```python
agent.updateInfo(...)  # Updates updatedAt
agent.setMCP(...)      # Updates updatedAt
agent.addSkill(...)    # Updates updatedAt
```

#### 3. Track Changes in Description

```python
agent.updateInfo(
    description="Agent v1.2.3 - Added data analysis capabilities. Updated: 2024-01-15"
)
```

### What Gets Updated vs What Stays the Same

| Field | Can Change? | How |
|-------|-------------|-----|
| **agentId** | ❌ Never | Immutable ERC-721 token ID |
| **agentURI** | ✅ Yes | Points to new IPFS CID |
| **name** | ✅ Yes | Update via `updateInfo()` |
| **description** | ✅ Yes | Update via `updateInfo()` |
| **endpoints** | ✅ Yes | Update via `setMCP()`, `setA2A()`, etc. |
| **metadata** | ✅ Yes | Update via `setMetadata()` |
| **updatedAt** | ✅ Yes | Auto-updated on changes |
| **owner** | ✅ Yes | Transfer NFT ownership |

### Version History

IPFS provides version history:

```
Initial registration:
  CID: QmABC123... → Registration v1.0.0

First update:
  CID: QmXYZ789... → Registration v1.1.0
  (Old CID QmABC123... still exists on IPFS)

Second update:
  CID: QmDEF456... → Registration v1.2.0
  (Both old CIDs still exist)
```

The blockchain stores the **current URI** (latest CID), but old CIDs remain on IPFS for history.

---

## Quick Reference

### Agent Lifecycle

```python
# 1. Create agent
agent = sdk.createAgent(name="My Agent", description="...")

# 2. Configure endpoints
agent.setMCP("https://mcp.example.com/")
agent.setA2A("https://a2a.example.com/agent-card.json")

# 3. Add OASF skills/domains
agent.addSkill("data_engineering/data_transformation_pipeline", validate_oasf=True)
agent.addDomain("technology/data_science", validate_oasf=True)

# 4. Set metadata
agent.setMetadata({"version": "1.0.0"})
agent.setAgentWallet("0x...", chainId=11155111)
agent.setTrust(reputation=True, cryptoEconomic=True)

# 5. Register on-chain
agent.registerIPFS()
# agentId = "11155111:123"

# 6. Update later
agent = sdk.loadAgent("11155111:123")
agent.updateInfo(description="Updated description")
agent.setMetadata({"version": "1.1.0"})
agent.registerIPFS()  # Updates same agentId
```

### Search & Discovery

```python
# Search agents
results = sdk.searchAgents(
    name="AI",
    a2aSkills=["python"],
    mcpTools=["code_generation"],
    active=True,
    chains="all"  # Search across all chains
)

# Search by reputation
reputation_results = sdk.searchAgentsByReputation(
    minAverageScore=80,
    tags=["great"],
    chains="all"
)
```

### Feedback System

```python
# Prepare feedback
feedback_file = sdk.prepareFeedback(
    agentId="11155111:123",
    score=85,
    tags=["great", "helpful"],
    text="This agent was very helpful!"
)

# Give feedback
feedback = sdk.giveFeedback("11155111:123", feedback_file)

# Search feedback
feedbacks = sdk.searchFeedback(
    agentId="11155111:123",
    minScore=80,
    maxScore=100
)

# Get reputation summary
summary = sdk.getReputationSummary("11155111:123")
print(f"Average score: {summary['averageScore']}")
```

### Key Concepts Summary

| Concept | Description |
|---------|-------------|
| **Agent** | Any entity registered on-chain (designed for AI agents) |
| **Agent ID** | Immutable ERC-721 token ID (format: `chainId:tokenId`) |
| **Registration File** | JSON file stored on IPFS describing the agent |
| **ERC-8004** | Ethereum standard for agent identity and reputation |
| **MCP** | Model Context Protocol (tools, prompts, resources) |
| **A2A** | Agent-to-Agent protocol (skills) |
| **OASF** | Open Agentic Schema Framework (standardized taxonomies) |
| **IPFS** | Decentralized storage for registration/feedback files |
| **The Graph** | Indexing service for fast search and queries |
| **Testnets** | Sandbox environments (Sepolia, Base Sepolia, Polygon Amoy) |

### Important Notes

- ⚠️ **Alpha Release**: Agent0 SDK v0.31 is in alpha - not production ready
- 🔗 **Testnets Only**: Currently only testnets supported (no mainnet)
- 🆓 **Free Storage**: Pinata and Filecoin Pin offer free storage for ERC-8004 agents
- 🔒 **Agent ID Immutable**: Once registered, agent ID never changes
- 🔄 **Update, Don't Re-Register**: Use `registerIPFS()` on existing agents to update
- 📝 **Version Tracking**: Use metadata to track your own versioning scheme

---

## Additional Resources

- **Documentation**: https://sdk.ag0.xyz
- **GitHub**: https://github.com/agent0lab/agent0-py
- **ERC-8004 Spec**: https://eips.ethereum.org/EIPS/eip-8004
- **OASF**: https://github.com/agntcy/oasf
- **MCP Protocol**: https://modelcontextprotocol.io
- **A2A Protocol**: https://a2a-protocol.org

---

**Last Updated**: Based on Agent0 SDK v0.31

**Status**: Alpha Release - Active Development