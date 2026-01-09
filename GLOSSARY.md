# Agent0 & ERC-8004 Glossary

A comprehensive glossary of terms used in the Agent0 SDK, ERC-8004 standard, and the broader agentic economy ecosystem.

---

## Table of Contents

- [Blockchain & Web3 Fundamentals](#blockchain--web3-fundamentals)
- [ERC-8004 Core Concepts](#erc-8004-core-concepts)
- [Agent Protocols](#agent-protocols)
- [Identity & Trust](#identity--trust)
- [Storage & Indexing](#storage--indexing)
- [SDK Components](#sdk-components)
- [Cryptography](#cryptography)
- [Data Formats](#data-formats)

---

## Blockchain & Web3 Fundamentals

### ABI (Application Binary Interface)

The **ABI** is a JSON specification that defines how to interact with a smart contract. It describes:
- Function names and their parameters
- Parameter types (uint256, address, string, bytes32, etc.)
- Return types
- Events the contract emits

**Example:**
```json
{
  "inputs": [
    { "name": "agentId", "type": "uint256" },
    { "name": "score", "type": "uint8" }
  ],
  "name": "giveFeedback",
  "outputs": [],
  "type": "function"
}
```

**Why it matters:** The SDK uses ABIs to encode function calls and decode responses when interacting with ERC-8004 contracts.

---

### Address

A 20-byte (40 hex character) identifier for an account on Ethereum. There are two types:
- **EOA (Externally Owned Account):** Controlled by a private key (e.g., `0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb`)
- **Contract Address:** Points to a deployed smart contract

**Format:** `0x` followed by 40 hexadecimal characters.

---

### Block

A batch of transactions grouped together and added to the blockchain. Each block contains:
- A list of transactions
- A reference to the previous block (hash)
- A timestamp
- The miner/validator who created it

---

### Chain ID

A unique numeric identifier for a blockchain network. Used to prevent replay attacks across chains.

| Chain                      | Chain ID |
|----------------------------|----------|
| Ethereum Mainnet           | 1        |
| Ethereum Sepolia (Testnet) | 11155111 |
| Base Sepolia               | 84532    |
| Polygon Amoy               | 80002    |
| Linea Sepolia              | 59141    |

---

### Contract (Smart Contract)

A program deployed on the blockchain that executes automatically when called. ERC-8004 defines three contracts:
- **Identity Registry** - Manages agent NFTs
- **Reputation Registry** - Stores feedback
- **Validation Registry** - Records third-party verifications

---

### EOA (Externally Owned Account)

An Ethereum account controlled by a private key (as opposed to a smart contract). When you create a wallet, you get an EOA.

---

### ERC (Ethereum Request for Comments)

A standard proposal for Ethereum. Important ERCs include:
- **ERC-20:** Fungible tokens (e.g., USDC)
- **ERC-721:** Non-fungible tokens (NFTs) - used by ERC-8004 for agents
- **ERC-1271:** Standard for smart contract signatures
- **ERC-8004:** The agent identity and reputation standard

---

### Gas

The unit measuring computational effort on Ethereum. Every operation costs gas:
- Simple transfer: ~21,000 gas
- Contract deployment: 100,000+ gas
- Complex function call: varies

**Gas Price:** How much ETH you pay per unit of gas (measured in Gwei).
**Gas Limit:** Maximum gas you're willing to spend on a transaction.

---

### NFT (Non-Fungible Token)

A unique, non-interchangeable token on the blockchain. In ERC-8004, **each agent is an NFT** (ERC-721), giving it:
- Unique identity (tokenId = agentId)
- Ownership that can be transferred
- Metadata (the registration file)

---

### Private Key

A 256-bit secret number that controls an Ethereum account. **Never share it.**

**Format:** 64 hex characters, often prefixed with `0x`
```
0x1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef
```

From a private key, you derive:
- Public key → Address

---

### Provider

A connection to an Ethereum node that allows reading blockchain data and submitting transactions.

**Types:**
- **RPC Provider:** HTTP/WebSocket connection (e.g., Alchemy, Infura)
- **Injected Provider:** Browser wallet (e.g., MetaMask)

---

### RPC (Remote Procedure Call)

The protocol used to communicate with Ethereum nodes. You send JSON-RPC requests to an RPC URL.

**Example RPC URLs:**
- `https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY`
- `https://sepolia.infura.io/v3/YOUR_KEY`

---

### Signer

An object that can sign transactions and messages. In ethers.js:
- **Wallet:** A signer created from a private key
- **JsonRpcSigner:** A signer from a provider (e.g., MetaMask)

---

### Testnet

A blockchain network for testing that uses valueless tokens. Used to develop without spending real money.

**Agent0 supported testnets:**
- Ethereum Sepolia
- Base Sepolia
- Polygon Amoy
- Linea Sepolia

---

### Transaction

An operation that changes blockchain state. Requires:
- Sender address
- Recipient (contract or EOA)
- Data (for contract calls)
- Gas limit and gas price
- Signature from the sender's private key

---

### Wallet

Software or hardware that manages private keys and can sign transactions. Examples:
- MetaMask (browser extension)
- Hardware wallets (Ledger, Trezor)
- Programmatic wallets (ethers.js Wallet class)

---

## ERC-8004 Core Concepts

### Agent

In ERC-8004, an **agent** is any entity registered on-chain. Technically, any application can be an agent, but the standard is designed for **AI agents and agentic systems**.

**Minimum requirements:**
- A `name` (string)
- A `description` (string)

**What makes an agent useful:**
- Advertises capabilities via endpoints (MCP, A2A)
- Is discoverable through search
- Can receive feedback to build reputation

---

### Agent ID

A unique identifier for an agent. Format: `chainId:tokenId`

**Examples:**
- `11155111:123` - Agent #123 on Ethereum Sepolia
- `84532:456` - Agent #456 on Base Sepolia

The tokenId is the ERC-721 NFT token ID assigned by the Identity Registry.

---

### Agent Registry

The full identifier for an Identity Registry contract. Format: `{namespace}:{chainId}:{contractAddress}`

**Example:**
```
eip155:11155111:0x8004a6090Cd10A7288092483047B097295Fb8847
```

- `eip155` - EVM chain namespace
- `11155111` - Ethereum Sepolia chain ID
- `0x8004...` - Identity Registry contract address

---

### Agent URI

The URI pointing to an agent's registration file. Can be:
- **IPFS:** `ipfs://QmXyz...`
- **HTTPS:** `https://example.com/agent.json`
- **Data URI:** `data:application/json;base64,eyJ0eXBl...`

---

### Agent Wallet

The on-chain verified address where an agent receives payments.

**v1.0 rules:**
- Reserved metadata key (cannot be set via `setMetadata()`)
- Initially set to owner's address
- Requires EIP-712 signature to change
- Resets to zero address on ownership transfer

---

### Capability

What an agent can do. Capabilities are advertised through endpoints:

| Protocol | Capability Types |
|----------|------------------|
| **MCP** | Tools, Prompts, Resources, Completions |
| **A2A** | Skills, Tasks |
| **OASF** | Standardized Skills, Domains |

---

### Endpoint

A way to interact with an agent. Defined in the registration file:

```json
{
  "name": "MCP",
  "endpoint": "https://mcp.agent.com/",
  "version": "2025-06-18"
}
```

**Common endpoint types:**
- `MCP` - Model Context Protocol server
- `A2A` - Agent-to-Agent protocol
- `OASF` - Open Agentic Schema Framework
- `ENS` - Ethereum Name Service
- `DID` - Decentralized Identifier
- `web` - Human-facing website
- `email` - Contact email

---

### Feedback

A reputation signal given by a client (reviewer) to an agent. On-chain fields:
- `score` - 0-100 rating (required)
- `tag1`, `tag2` - Categorization strings (optional)
- `endpoint` - Which endpoint was used (optional)
- `feedbackURI` - Link to off-chain details (optional)
- `feedbackHash` - Hash of off-chain file (optional)

---

### Feedback Index

A sequential number tracking feedback from a specific client to a specific agent. Starts at 1 and increments with each feedback submission.

**Example:** If client `0xABC...` gives 3 feedbacks to agent `11155111:42`, they have indexes 1, 2, and 3.

---

### Identity Registry

The ERC-721 smart contract that manages agent identities. Functions:
- `register()` - Mint a new agent NFT
- `setAgentURI()` - Update the registration file location
- `setMetadata()` - Set on-chain key-value metadata
- `setAgentWallet()` - Update payment address (with signature)
- `transferFrom()` - Transfer agent ownership

---

### Operator

An address authorized to manage an agent without owning it. Set via ERC-721's `setApprovalForAll()`.

---

### Owner

The address that owns an agent NFT. Has full control including:
- Updating the agent URI
- Setting metadata
- Transferring ownership
- Delegating to operators

---

### Registration File

The JSON document describing an agent's identity and capabilities. Stored off-chain (IPFS/HTTP), linked via `agentURI`.

**Required fields:**
- `type` - Schema identifier
- `name` - Agent name
- `description` - What the agent does

**Optional fields:**
- `image` - Agent avatar URL
- `endpoints` - Communication protocols
- `registrations` - On-chain registrations
- `supportedTrust` - Trust models supported
- `active` - Is the agent operational
- `x402Support` - Accepts x402 payments

---

### Reputation Registry

The smart contract storing feedback and reputation data. Functions:
- `giveFeedback()` - Submit a review
- `revokeFeedback()` - Retract your feedback
- `appendResponse()` - Add a response to feedback
- `readFeedback()` - Get a specific feedback entry
- `getSummary()` - Get aggregated score

---

### Validation Registry

The smart contract for third-party verification (TEE, zkML, staked inference). Functions:
- `validationRequest()` - Request verification
- `validationResponse()` - Submit verification result

---

## Agent Protocols

### A2A (Agent-to-Agent Protocol)

A protocol enabling agents to discover and communicate with each other. Key concepts:

**Agent Card:** A JSON file describing an agent's capabilities:
```json
{
  "name": "Data Agent",
  "skills": [
    { "name": "Analysis", "tags": ["python", "pandas"] }
  ]
}
```

**Discovery paths:**
- `/.well-known/agent-card.json` (recommended)
- `/.well-known/agent.json`
- `/agentcard.json`

---

### MCP (Model Context Protocol)

A protocol for AI models to access external tools and resources. Defines:

**Tools:** Functions the agent can execute
```json
{ "name": "search_web", "description": "Search the internet" }
```

**Prompts:** Pre-defined prompt templates

**Resources:** Data sources the agent can access

**Communication:** JSON-RPC over HTTP

---

### OASF (Open Agentic Schema Framework)

A standardized taxonomy for agent capabilities. Provides:
- **Skills:** 136 standardized skill categories
- **Domains:** 204 domain categories

**Example skills:**
- `natural_language_processing/summarization`
- `data_engineering/data_transformation_pipeline`

**Example domains:**
- `finance_and_business/investment_services`
- `technology/data_science`

---

### x402

A payment protocol for AI services. When `x402Support: true`, the agent accepts programmatic payments. Proof of payment can be included in feedback.

---

## Identity & Trust

### DID (Decentralized Identifier)

A self-sovereign identity standard. Format: `did:{method}:{identifier}`

**Example:** `did:ethr:0x742d35...`

---

### ENS (Ethereum Name Service)

Human-readable names for Ethereum addresses. Like DNS for blockchain.

**Example:** `vitalik.eth` → `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045`

---

### EIP-712

A standard for typed structured data signing. Creates human-readable signature requests instead of raw hex.

**Used in v1.0 for:**
- `setAgentWallet()` - Proving control of new wallet address

---

### ERC-1271

A standard for smart contract signature verification. Allows smart contract wallets to "sign" messages.

---

### Reputation

An agent's track record based on accumulated feedback. Aggregated as:
- `count` - Number of feedback entries
- `averageScore` - Mean of all scores (0-100)

Can be filtered by:
- Tags
- Reviewer addresses
- Capabilities/skills

---

### Trust Model

How trust is established with an agent. ERC-8004 supports:

| Model             | Description                                |
|-------------------|--------------------------------------------|
| `reputation`      | Based on accumulated feedback              |
| `crypto-economic` | Staked collateral at risk                  |
| `tee-attestation` | Trusted Execution Environment verification |

---

## Storage & Indexing

### CID (Content Identifier)

An IPFS content address. A hash of the file content.

**Example:** `QmXoypizjW3WknFiJnKLwHCnL72vedxjQkDDP1mXWo6uco`

**Property:** Same content always produces the same CID.

---

### Gateway

An HTTP server that provides access to IPFS content. Examples:
- `https://gateway.pinata.cloud/ipfs/{cid}`
- `https://ipfs.io/ipfs/{cid}`
- `https://dweb.link/ipfs/{cid}`

---

### IPFS (InterPlanetary File System)

A distributed storage network. Files are addressed by content hash (CID), not location.

**Benefits:**
- Decentralized (no single point of failure)
- Immutable (content can't be changed)
- Verifiable (hash proves authenticity)

---

### Pinning

Keeping IPFS content available by storing it on a node. Without pinning, content may be garbage collected.

**Services:**
- Pinata
- Filecoin
- Local IPFS node

---

### Subgraph

A GraphQL API that indexes blockchain and IPFS data. Created using The Graph protocol.

**What it indexes:**
- Agent registrations
- Feedback events
- Registration file contents

**Why use it:**
- Fast queries (vs. slow blockchain reads)
- Complex filtering
- Full-text search

---

### The Graph

A decentralized indexing protocol. Indexers run nodes that:
1. Watch blockchain events
2. Fetch linked IPFS data
3. Build queryable indexes
4. Serve GraphQL queries

---

## SDK Components

### Agent Class

The SDK class managing an individual agent's lifecycle:
- Configuration (endpoints, metadata)
- Registration (IPFS upload, on-chain minting)
- Updates (re-registration with same agentId)
- Transfer (ownership change)

---

### EndpointCrawler

SDK component that automatically extracts capabilities from endpoints:
- **MCP:** JSON-RPC calls to `tools/list`, `prompts/list`, `resources/list`
- **A2A:** HTTP fetch of agent card, extract skill tags

---

### FeedbackManager

SDK component handling reputation operations:
- Preparing feedback files
- Submitting feedback on-chain
- Searching feedback
- Aggregating reputation summaries

---

### IPFSClient

SDK component for decentralized storage:
- Upload registration/feedback files
- Retrieve files by CID
- Supports Pinata, Filecoin, local nodes

---

### SDK Class

The main entry point orchestrating all components:
- Initializes Web3, IPFS, Subgraph clients
- Creates and loads agents
- Provides search functionality
- Manages feedback operations

---

### SubgraphClient

SDK component for fast queries via GraphQL:
- Search agents by attributes
- Filter by capabilities
- Get feedback history
- Multi-chain queries

---

### Web3Client

SDK component for blockchain interactions:
- Contract instantiation
- Read operations (view functions)
- Write operations (transactions)
- Signature creation

---

## Cryptography

### Hash

A fixed-size output from a one-way function. Used for:
- Content verification (IPFS CIDs)
- Commitment schemes (feedback hash)
- Addresses (keccak256 of public key)

**Keccak-256:** Ethereum's hash function (similar to SHA-3)

---

### Signature

A cryptographic proof that a message was created by the holder of a private key.

**Types:**
- **ECDSA:** Standard Ethereum signature (65 bytes: r, s, v)
- **EIP-712:** Typed structured data signature
- **ERC-1271:** Smart contract signature

**Use cases:**
- Signing transactions
- Proving wallet ownership (setAgentWallet)
- Message authentication

---

### bytes32

A 32-byte (256-bit) fixed-size data type in Solidity. Previously used for feedback tags (now `string` in v1.0).

**Encoding example:**
```
"great" → 0x6772656174000000000000000000000000000000000000000000000000000000
```

---

## Data Formats

### CAIP (Chain Agnostic Improvement Proposal)

Standards for cross-chain identifiers:
- **CAIP-2:** Chain ID format (`eip155:1`)
- **CAIP-10:** Account ID format (`eip155:1:0x742...`)

---

### Data URI

A way to embed data inline in a URI. Format:
```
data:{mediatype};{encoding},{data}
```

**Example (base64 JSON):**
```
data:application/json;base64,eyJ0eXBlIjoiaHR0cHM6Ly9laXBzLmV0aGVyZXVtLm9yZy9FSVBzL2VpcC04MDA0I3JlZ2lzdHJhdGlvbi12MSJ9
```

---

### GraphQL

A query language for APIs. Used by The Graph subgraphs.

**Example query:**
```graphql
query {
  agents(where: { active: true }, first: 10) {
    id
    name
    mcpTools
  }
}
```

---

### JSON-RPC

A protocol for remote procedure calls using JSON. Used by:
- Ethereum nodes (eth_call, eth_sendTransaction)
- MCP servers (tools/list, prompts/list)

**Format:**
```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "params": {},
  "id": 1
}
```

---

### URI vs URL

- **URI (Uniform Resource Identifier):** Any identifier (`ipfs://Qm...`, `did:ethr:0x...`)
- **URL (Uniform Resource Locator):** A locatable address (`https://example.com`)

All URLs are URIs, but not all URIs are URLs.

---

## Quick Reference Tables

### Common Abbreviations

| Abbreviation | Full Name                       |
|--------------|---------------------------------|
| ABI          | Application Binary Interface    |
| A2A          | Agent-to-Agent Protocol         |
| CID          | Content Identifier              |
| DID          | Decentralized Identifier        |
| ENS          | Ethereum Name Service           |
| EOA          | Externally Owned Account        |
| ERC          | Ethereum Request for Comments   |
| IPFS         | InterPlanetary File System      |
| MCP          | Model Context Protocol          |
| NFT          | Non-Fungible Token              |
| OASF         | Open Agentic Schema Framework   |
| RPC          | Remote Procedure Call           |
| TEE          | Trusted Execution Environment   |
| URI          | Uniform Resource Identifier     |
| zkML         | Zero-Knowledge Machine Learning |

### ERC-8004 Contracts

| Contract            | Purpose      | Key Functions                            |
|---------------------|--------------|------------------------------------------|
| Identity Registry   | Agent NFTs   | register, setAgentURI, setMetadata       |
| Reputation Registry | Feedback     | giveFeedback, revokeFeedback, getSummary |
| Validation Registry | Verification | validationRequest, validationResponse    |

### Agent ID Format

```
{chainId}:{tokenId}

Examples:
11155111:123    → Ethereum Sepolia, token #123
84532:456       → Base Sepolia, token #456
```

### Agent Registry Format

```
{namespace}:{chainId}:{contractAddress}

Example:
eip155:11155111:0x8004a6090Cd10A7288092483047B097295Fb8847
```

---

*Last updated: January 2026 - Based on ERC-8004 v1.0*
