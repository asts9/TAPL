# TAPL — Real-Time Tap Trading Platform

Backend, smart-contract, and Chainlink CRE infrastructure for the **TAPL BTC/USDT tap-trading platform**.

TAPL combines an offchain trading/application backend with EVM smart contracts and Chainlink Runtime Environment (CRE) workflows for verification, settlement commitments, solvency reporting, LP reserve management, and strategy updates.

---

## 🚀 Project Overview

TAPL is a BTC/USDT tap-trading platform designed around fast offchain gameplay with blockchain-based verification and settlement commitments.

The system is divided into three major layers:

1. **TAPL Backend**

   * NestJS + TypeScript
   * Authentication and account management
   * Order and settlement processing
   * Payments and distributions
   * Price, risk, and strategy modules
   * Realtime socket communication
   * Blockchain integrations through ethers.js

2. **Smart Contract System**

   * Solidity / EVM
   * Price integrity reporting
   * Settlement batch commitments
   * Pool reserve management
   * LP distribution
   * Strategy parameter management

3. **Chainlink CRE Workflows**

   * Scheduled and HTTP-triggered workflows
   * Application API integration
   * Chainlink reference data
   * Onchain reads and writes
   * Signed report submission

---

# 🏗️ High-Level Architecture

```mermaid
flowchart LR
    A[Client Apps] --> B[NestJS API]

    B --> C[PostgreSQL]
    B --> D[Redis]
    B --> E[MinIO]
    B --> F[Kafka]
    B --> G[EVM RPC / Smart Contracts]
    B --> I[Socket Gateway]

    H[Worker Service] --> C
    H --> D
    H --> F
    H --> G

    I --> A

    B --> J[Tapl App API]
    J --> K[Chainlink CRE Workflows]

    K --> L[PriceIntegrity.sol]
    K --> M[Settlement.sol]
    K --> N[PoolReserve.sol]
    K --> O[LPDistributor.sol]
    K --> P[StrategyManager.sol]

    L --> Q[Backend Indexer]
    M --> Q
    N --> Q
    O --> Q
    P --> Q
```

---

# 📦 Backend Architecture

The main TAPL backend is built using **NestJS and TypeScript**.

### Backend responsibilities

* Authentication and authorization
* User account management
* Order lifecycle
* Settlement processing
* Payment flows
* Distribution and allocation
* Price ingestion
* Risk checks
* Strategy configuration
* Realtime communication
* Background workers
* Blockchain-facing integrations

### Backend infrastructure

| Component      | Responsibility                   |
| -------------- | -------------------------------- |
| NestJS         | Main API and business logic      |
| PostgreSQL     | Relational application data      |
| Redis          | Caching and runtime coordination |
| MinIO          | Object storage                   |
| Kafka          | Event-driven communication       |
| EVM / RPC      | Blockchain integration           |
| Socket Gateway | Realtime updates                 |

---

# 🧩 Backend Domains

| Domain         | Responsibility                                    |
| -------------- | ------------------------------------------------- |
| `auth`         | Authentication, authorization, and access control |
| `account`      | User account management                           |
| `order`        | Order lifecycle handling                          |
| `settlement`   | Settlement processing and related jobs            |
| `payment`      | Payment-related business flows                    |
| `distribution` | Distribution and allocation flows                 |
| `price`        | Price ingestion and processing                    |
| `risk`         | Risk checks and policy logic                      |
| `strategy`     | Strategy configuration and execution support      |
| `socket`       | Realtime communication                            |
| `worker`       | Background processing and listeners               |

---

# 📁 Backend Repository Structure

```text
.
├── src/
│   ├── adapters/
│   ├── config/
│   ├── libs/
│   ├── migrations/
│   ├── modules/
│   ├── scripts/
│   └── utils/
├── system-design/
├── benchmark-results/
├── docker-compose.yml
├── docker.env.example
├── README.example.md
└── README.md
```

---

# ⛓️ TAPL x Chainlink

The blockchain layer provides smart contracts and Chainlink Runtime Environment infrastructure for the TAPL BTC tap-trading proof of concept.

This layer is responsible for:

* Onchain verification
* Settlement batching
* Solvency reporting
* LP reserve management
* Strategy updates
* Chainlink CRE workflows

The blockchain layer does **not** perform the primary order-matching engine.

Fast gameplay and bet resolution remain offchain, while blockchain infrastructure provides verification and canonical batch commitments.

---

# 📊 Product Capability Status

| Domain    | Feature                       | Status | Notes                                                         |
| --------- | ----------------------------- | ------ | ------------------------------------------------------------- |
| Contracts | Price integrity reporting     | Done   | 15m batch comparison, pass/fail flags, hashes, report storage |
| Contracts | Settlement batch commitment   | Done   | `commitSettlementBatch(...)` stores batch metadata            |
| Contracts | Pool reserve vault            | Done   | LP deposit/withdraw, trader deposit/claim, solvency reporting |
| Contracts | Strategy parameter management | Done   | Volatility regime updates onchain                             |
| CRE       | Price integrity workflow      | Done   | 15m cron, app API + Chainlink reference comparison            |
| CRE       | Settlement workflow           | Done   | 15m cron, app API batch → onchain commit                      |
| CRE       | Pool solvency workflow        | Done   | Daily cron, app API liability + onchain balance read          |
| CRE       | Strategy rebalance workflow   | Done   | HTTP trigger, app API state + regime payload                  |

---

# 🔗 Chainlink Integration

TAPL uses Chainlink in two major ways.

## 1. Chainlink Runtime Environment

CRE workflows live under:

```text
cre/
```

Each workflow uses the CRE SDK for:

* Cron or HTTP triggers
* Deterministic workflow execution
* EVM onchain reads
* Signed report submission

### Workflow entrypoints

```text
cre/price-integrity/main.ts
cre/settlement/main.ts
cre/pool-solvency/main.ts
cre/strategy-rebalance/main.ts
```

---

## 2. CRE-Compatible Consumer Contracts

The consumer contracts receive reports/actions generated by the Chainlink CRE workflows.

Key contracts:

```text
contracts/src/PriceIntegrity.sol
contracts/src/Settlement.sol
contracts/src/PoolReserve.sol
contracts/src/StrategyManager.sol
```

---

# 📜 Smart Contract System

| Contract               | Purpose                                                                         |
| ---------------------- | ------------------------------------------------------------------------------- |
| `Roles.sol`            | Central role registry for owner, reporter, settler, strategist, and distributor |
| `PriceIntegrity.sol`   | Stores 15m batch comparison reports, pass/fail results, and failure flags       |
| `Settlement.sol`       | Stores settlement batch commitments and payout markers                          |
| `PoolReserve.sol`      | USDT vault, LP shares, trader flows, and solvency reports                       |
| `LPDistributor.sol`    | Distribution queue and reserve allocation signaling                             |
| `StrategyManager.sol`  | Onchain volatility regime parameters                                            |
| `ReceiverTemplate.sol` | CRE-compatible consumer receiver base                                           |

---

# 🔄 CRE Workflow System

| Workflow           | Trigger      | Input Source                         | Contract Write                          |
| ------------------ | ------------ | ------------------------------------ | --------------------------------------- |
| Price Integrity    | 15m cron     | App API + Chainlink reference data   | `PriceIntegrity.sol`                    |
| Settlement         | 15m cron     | App API settlement batch             | `Settlement.sol`                        |
| Pool Solvency PoR  | Daily cron   | App API liability + ERC20 balance    | `PoolReserve.sol`                       |
| LP Distribution    | Daily cron   | App API LP distribution batch        | `LPDistributor.sol` + `PoolReserve.sol` |
| Strategy Rebalance | HTTP trigger | App API strategy state + risk engine | `StrategyManager.sol`                   |

Detailed workflow specifications are available under:

```text
specs/cre-workflows/
```

---

# 🔐 Core Architecture Flows

## 1. Price Integrity Flow

Every 15 minutes, the CRE workflow:

1. Fetches the last closed 15-minute OHLC window from the application API.
2. Fetches Chainlink-aligned reference data.
3. Canonicalizes the data.
4. Computes metrics and flags.
5. Submits the report to `PriceIntegrity.sol`.
6. The backend indexer processes the emitted events.

```mermaid
sequenceDiagram
    participant API as Tapl App API
    participant CL as Chainlink Reference Data
    participant CRE as CRE Price Integrity Workflow
    participant PI as PriceIntegrity.sol
    participant IDX as Backend Indexer

    CRE->>API: Fetch internal 1s OHLC batch
    CRE->>CL: Fetch reference 1s OHLC batch
    CRE->>CRE: Canonicalize and score
    CRE->>PI: Submit batch report
    PI-->>IDX: PriceIntegrityBatchReported
    PI-->>IDX: BatchSubmitted
```

---

## 2. Settlement Flow

The application API assembles deposits, withdrawals, and settled win/lose orders into 15-minute batches.

CRE then:

1. Reads the pending settlement batch.
2. Canonicalizes the records.
3. Computes the deterministic settlement root.
4. Commits the settlement batch onchain.
5. The backend indexer processes the event.

```mermaid
sequenceDiagram
    participant API as Tapl App API
    participant CRE as CRE Settlement Workflow
    participant SET as Settlement.sol
    participant IDX as Backend Indexer

    API-->>CRE: Pending 15m settlement batch
    CRE->>CRE: Canonicalize records
    CRE->>CRE: Compute settlement root
    CRE->>SET: commitSettlementBatch(...)
    SET-->>IDX: SettlementBatchCommitted
```

---

# 💹 Market and Settlement Model

* Product mode: **BTC/USDT tap-trading**
* Trading window: **5 seconds**
* Grid bands: **$20**
* Fast gameplay remains offchain
* Bet resolution remains offchain
* Blockchain is responsible for verification and batch commitments
* Price integrity is stored every 15 minutes
* Settlement is batch committed every 15 minutes
* Proof-of-reserve reporting is performed daily in the current PoC design

---

# 📡 Application → Chainlink → Blockchain

```mermaid
flowchart LR
    A["TAPL App API"] --> B["Chainlink CRE Workflows"]
    C["Chainlink Reference Data"] --> B

    B --> D["PriceIntegrity.sol"]
    B --> E["Settlement.sol"]
    B --> F["PoolReserve.sol"]
    B --> G["LPDistributor.sol"]
    B --> H["StrategyManager.sol"]

    F --> I["USDT / ERC20 Asset"]
    G --> F
    E --> F

    D --> J["Backend Indexer"]
    E --> J
    F --> J
    G --> J
    H --> J
```

---

# 📍 Deployed Contracts

Current deployment summary:

| Contract                  | Address                                      |
| ------------------------- | -------------------------------------------- |
| `Roles`                   | `0x23B8F26A359C036C69BfaF28AE765D3d975f055d` |
| `PriceIntegrity`          | `0x60430364eBC71ac11720f012756ceA2c294c50de` |
| `PoolReserve`             | `0x0b7400e10fd916A7B9eEe8981532aF9f38255bAf` |
| `Settlement`              | `0xEDD391FDa28993287Df301485ABF72865dee5050` |
| `StrategyManager`         | `0x1CB5b9fc0F24D26366aA8F2aeFE875fD356f4616` |
| `Asset (USDT on Sepolia)` | `0x779877A7B0D9E8603169DdbD7836e478b4624789` |
| `Forwarder`               | `0x15fC6ae953E024d975e77382eEeC56A9101f9F88` |

### Network

```text
Network: Sepolia
RPC: https://eth-sepolia.api.onfinality.io/public
```

---

# 🗂️ Blockchain Repository Structure

```text
.
├── contracts/
│   ├── src/
│   ├── script/
│   ├── test/
│   └── foundry.toml
├── cre/
│   ├── price-integrity/
│   ├── settlement/
│   ├── pool-solvency/
│   ├── strategy-rebalance/
│   ├── test/
│   └── project.yaml
├── specs/
│   ├── cre-workflows/
│   ├── smart-contract-build-checklist.md
│   ├── smart-contract-vertical-slides.md
│   ├── cre-event-indexing-spec.md
│   └── frontend-pool-reserve-integration.md
├── deployments-v1.txt
└── README.example.md
```

---

# 🛠️ Backend Setup

## Prerequisites

Install:

* Node.js `23.7.0+` recommended
* Yarn
* Docker
* Docker Compose
* TypeScript

---

# ⚙️ Environment Variables

Create a `.env` file:

```env
NODE_ENV=production
PORT=3001
WORKER_PORT=3002
NETWORK=mainnet

# PostgreSQL
POSTGRES_URL=postgres://root:1@localhost:5432/rwa

# Redis
REDIS_URL=redis://default:foobared@localhost:6379/0

# MinIO
MINIO_ACCESS_KEY=development
MINIO_SECRET_KEY=123456789
BUCKET_NAME=development
MINIO_HOST=localhost
MINIO_PORT=32126

# Kafka
KAFKA_BROKER=localhost:39092
KAFKA_TOPIC_PREFIX=local-rwa
KAFKA_RUNNING_FLAG=true

# RPC
RPC=

# Authentication
JWT_SECRET=1

# External services
APIFY_KEY=
PRIVY_APP_ID=
PRIVY_APP_SECRET=

# Admin private key
ADMIN_PRIVATE_KEY=
```

> Never commit real private keys, API keys, passwords, or production secrets to GitHub.

---

# 🐳 Docker Environment

Create `docker.env`:

```env
POSTGRES_USER=root
POSTGRES_PASSWORD=1
POSTGRES_DB=viral_bot
POSTGRES_PORT=5432

REDIS_PASSWORD=foobared
REDIS_PORT=6379

MINIO_ROOT_USER=development
MINIO_ROOT_PASSWORD=123456789
MINIO_PORT=32126
MINIO_CONSOLE_PORT=9001

ZOOKEEPER_PORT=2181

KAFKA_PORT=39092
KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:39092

CLICKHOUSE_DB=viral_bot
CLICKHOUSE_USER=default
CLICKHOUSE_PASSWORD=1
CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT=1
CLICKHOUSE_PORT_HTTP=8123
CLICKHOUSE_PORT_TCP=9000
```

---

# ▶️ Running the Backend

### 1. Start infrastructure

```bash
docker compose --env-file docker.env up -d
```

### 2. Install dependencies

```bash
yarn install
```

### 3. Generate TypeScript bindings

```bash
yarn typechain:gen
```

### 4. Generate migrations

```bash
yarn migration:generate
```

### 5. Apply migrations

```bash
yarn migration:up
```

### 6. Start the main application

```bash
yarn dev
```

### 7. Start the worker

Open another terminal:

```bash
yarn dev:worker
```

---

# ⛓️ Smart Contract Development

## Prerequisites

* Foundry

### Build

```bash
cd contracts
forge build
```

### Test

```bash
forge test -vv
```

### Deploy

```bash
cd contracts

forge script script/DeployHackathon.s.sol \
  --rpc-url $RPC_URL \
  --broadcast

forge script script/SeedDemoData.s.sol \
  --rpc-url $RPC_URL \
  --broadcast
```

---

# 🔗 CRE Development

## Prerequisites

* Bun
* Chainlink CRE CLI

### Install dependencies

```bash
cd cre
bun install
```

### Build

```bash
bun run build
```

### Test

```bash
bun test
```

---

# 🧪 CRE Simulation

```bash
cd cre

cre workflow simulate price-integrity --target local-simulation

cre workflow simulate settlement --target local-simulation

cre workflow simulate pool-solvency --target local-simulation
```

---

# 📚 Integration Specifications

Important specifications are located under:

```text
specs/
```

### Backend indexing

```text
specs/cre-event-indexing-spec.md
```

### Pool reserve frontend integration

```text
specs/frontend-pool-reserve-integration.md
```

### CRE workflow behavior

```text
specs/cre-workflows/README.md
```

### Smart contract checklist

```text
specs/smart-contract-build-checklist.md
```

---

# 🧱 Engineering Maturity

| Area                 | Status        | Notes                                                      |
| -------------------- | ------------- | ---------------------------------------------------------- |
| Smart contract scope | Hackathon PoC | Minimal scope optimized for delivery                       |
| CRE workflow scope   | Hackathon PoC | Deterministic and idempotent, app API driven               |
| Test coverage        | Good for PoC  | Foundry + Bun tests                                        |
| Security posture     | PoC           | Trusted app API / CRE assumptions remain                   |
| Production readiness | Not targeted  | Mocked CCIP, simplified trader accounting, no upgrade path |

---

# 🚫 Current Scope Limitations

This repository currently does **not** include:

* Frontend UI
* Main application backend services inside the blockchain repository
* Market engine
* Real Chainlink CCIP bridge execution

The application backend and blockchain/CRE infrastructure are integrated through defined APIs, workflows, events, and indexing specifications.

---

# 🔮 System Responsibility

```text
                    TAPL
                     │
        ┌────────────┴────────────┐
        │                         │
   OFFCHAIN SYSTEM            ONCHAIN SYSTEM
        │                         │
   NestJS Backend             Solidity
        │                         │
   Orders                    Verification
   Gameplay                  Settlement
   Payments                  Reserves
   Risk                      Strategy
   Pricing                   Reports
   Accounts                  Events
        │                         │
        └────────────┬────────────┘
                     │
                Chainlink CRE
                     │
          Verification / Automation
```

---

# 📖 References

* Chainlink CRE Documentation: https://docs.chain.link/cre
* TAPL Product Specification: `./specs/spec.md`
* CRE Workflow Specifications: `./specs/cre-workflows/README.md`

---

# 👨‍💻 Development

TAPL is structured as a modular system combining:

```text
NestJS
TypeScript
PostgreSQL
Redis
Kafka
MinIO
EVM / Solidity
Foundry
Chainlink CRE
Bun
```

The architecture separates high-frequency application gameplay from blockchain verification and settlement commitments, allowing the backend and blockchain layers to have clearly defined responsibilities.
