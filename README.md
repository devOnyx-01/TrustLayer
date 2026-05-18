# TrustLayer
**Decentralized Insurance Protocol on Stellar**

TrustLayer is a decentralized insurance protocol built on **Stellar using Soroban smart contracts**. It enables individuals and communities to pool capital and collectively underwrite coverage against DeFi risks — including smart contract exploits, protocol hacks, exchange/custody failures, stablecoin depegs, and validator slashing events — with all claims, payouts, and governance logic enforced fully on-chain.

The project solves the problem of opaque, centralized insurance providers in the DeFi ecosystem by replacing them with a member-owned, DAO-governed risk pool. Instead of a traditional insurance company, members pool capital together and govern the system collectively. TrustLayer is designed for developers, risk underwriters, and DeFi communities interested in building open, composable coverage infrastructure using low-fee, fast-finality blockchain primitives.

---

## 🚀 Core Features

- Non-custodial capital pooling via Soroban smart contracts
- Coverage for smart contract hacks and protocol exploits
- Protection against exchange and custody failures
- Stablecoin depeg coverage with oracle-verified triggers
- Validator slashing risk underwriting
- On-chain claims submission and community voting
- Staking-based underwriting — members stake capital to earn premiums
- DAO governance with TLR token for protocol decisions
- Risk marketplace for listing and pricing coverage products
- Native USDC-based pools on Stellar testnet

---

## 🏗 Architecture Overview

- **Frontend (`apps/web`)**
  Next.js application for interacting with TrustLayer smart contracts. Provides user interface for purchasing coverage, submitting claims, staking capital, and participating in governance votes.

- **Backend (`apps/api`)**
  Node.js API for off-chain services such as indexing contract events, monitoring risk triggers, sending claim notifications, managing user metadata, and aggregating pool analytics.

- **Smart Contracts (`contracts/`)**
  Soroban smart contracts written in Rust that manage all insurance logic, capital pool custody, premium calculations, claims adjudication, payout disbursement, and DAO governance.

- **Oracle Layer (`oracle/`)**
  Off-chain oracle services that feed verified price data, exploit detection signals, and slashing event feeds into the on-chain contracts to trigger automatic coverage assessments.

---

## 📁 Repository Structure

```text
/
├── apps/
│   ├── web/              # Next.js frontend
│   └── api/              # Node.js backend API
├── contracts/            # Soroban smart contracts (Rust)
├── oracle/               # Off-chain oracle services
├── packages/             # Shared utilities and types
├── scripts/              # Deployment and automation scripts
├── tests/                # Integration and E2E tests
└── README.md
```

---

## 🛠 Setup Instructions

### Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** (v18 or higher) - [Download](https://nodejs.org/)
- **npm** or **yarn** - Comes with Node.js
- **Rust** (stable toolchain) - [Install](https://rustup.rs/)
- **Soroban CLI** - Instructions below
- **Stellar testnet account** - We'll create this in setup

### Installation Overview

1. Clone the repository
2. Set up smart contracts
3. Set up backend API
4. Set up the oracle layer
5. Set up frontend
6. Run tests

---

## 📦 1. Clone the Repository

```bash
git clone https://github.com/your-org/trustlayer.git
cd trustlayer
```

---

## 🔗 2. Smart Contracts Setup (Soroban)

### Install Soroban CLI

```bash
cargo install --locked stellar-cli --features opt
```

Or use the install script:

```bash
curl -fsSL https://github.com/stellar/stellar-cli/raw/main/install.sh | sh
```

Verify installation:

```bash
stellar --version
```

### Configure Stellar Testnet

```bash
stellar network add --global testnet \
  --rpc-url https://soroban-testnet.stellar.org:443 \
  --network-passphrase "Test SDF Network ; September 2015"
```

### Generate Identity & Fund Account

```bash
stellar keys generate --global deployer --network testnet
```

Get your address:

```bash
stellar keys address deployer
```

Fund your account using Friendbot:

```bash
curl "https://friendbot.stellar.org?addr=$(stellar keys address deployer)"
```

Verify balance:

```bash
stellar account balance --id deployer --network testnet
```

### Build Contracts

```bash
cd contracts
cargo build --target wasm32-unknown-unknown --release
```

### Deploy Contracts

Deploy the core insurance pool contract:

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/trustlayer_pool.wasm \
  --source deployer \
  --network testnet
```

Deploy the governance contract:

```bash
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/trustlayer_governance.wasm \
  --source deployer \
  --network testnet
```

Save both contract IDs — you'll need them for the frontend, backend, and oracle setup.

### Initialize Contracts

Initialize the pool contract:

```bash
stellar contract invoke \
  --id YOUR_POOL_CONTRACT_ID \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin $(stellar keys address deployer) \
  --token CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA
```

Initialize the governance contract:

```bash
stellar contract invoke \
  --id YOUR_GOVERNANCE_CONTRACT_ID \
  --source deployer \
  --network testnet \
  -- initialize \
  --admin $(stellar keys address deployer) \
  --pool YOUR_POOL_CONTRACT_ID
```

---

## 🖥 3. Backend Setup (Node.js API)

```bash
cd apps/api
npm install
```

### Create Environment File

Create `.env` in `apps/api/`:

```env
PORT=3001
NODE_ENV=development

# Stellar Network
STELLAR_NETWORK=testnet
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
HORIZON_URL=https://horizon-testnet.stellar.org

# Contracts
POOL_CONTRACT_ID=YOUR_DEPLOYED_POOL_CONTRACT_ID
GOVERNANCE_CONTRACT_ID=YOUR_DEPLOYED_GOVERNANCE_CONTRACT_ID

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/trustlayer

# Optional
REDIS_URL=redis://localhost:6379
```

### Run Database Migrations

```bash
npm run migrate
```

### Start Backend Server

```bash
npm run dev
```

Backend should now be running at `http://localhost:3001`

### Verify Backend

```bash
curl http://localhost:3001/health
```

---

## 🔮 4. Oracle Layer Setup

```bash
cd oracle
npm install
```

### Create Environment File

Create `.env` in `oracle/`:

```env
PORT=3002
NODE_ENV=development

# Stellar
SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
ORACLE_SIGNER_SECRET=YOUR_ORACLE_ACCOUNT_SECRET_KEY

# Contracts
POOL_CONTRACT_ID=YOUR_DEPLOYED_POOL_CONTRACT_ID

# Price Feeds
COINGECKO_API_URL=https://api.coingecko.com/api/v3
DEPEG_THRESHOLD_PERCENT=2

# Exploit Monitoring (optional)
FORTA_API_KEY=YOUR_FORTA_API_KEY
```

### Start Oracle Service

```bash
npm run dev
```

Oracle service should now be running at `http://localhost:3002`

---

## 🌐 5. Frontend Setup (Next.js)

```bash
cd apps/web
pnpm install
```

### Create Environment File

Create `.env.local` in `apps/web/`:

```env
NEXT_PUBLIC_BASE_URL=https://trustlayer.app
NEXT_PUBLIC_HORIZON_PUBLIC_URL=https://horizon.stellar.org
NEXT_PUBLIC_HORIZON_TESTNET_URL=https://horizon-testnet.stellar.org
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_POOL_CONTRACT_ID=YOUR_DEPLOYED_POOL_CONTRACT_ID
NEXT_PUBLIC_GOVERNANCE_CONTRACT_ID=YOUR_DEPLOYED_GOVERNANCE_CONTRACT_ID
NEXT_PUBLIC_COINGECKO_API_URL=https://api.coingecko.com/api/v3
NEXT_PUBLIC_DISCORD_URL=https://discord.gg/trustlayer
NEXT_PUBLIC_TELEGRAM_URL=https://t.me/trustlayer
NEXT_PUBLIC_GITHUB_URL=https://github.com/your-org/trustlayer
```

### Run Development Server

```bash
pnpm dev
```

Frontend should now be running at `http://localhost:3000`

### Build for Production

```bash
npm run build
npm start
```

---

## 🧪 6. Running Tests

### Contract Tests

```bash
cd contracts
cargo test
```

### Backend Tests

```bash
cd apps/api
npm test
```

Run with coverage:

```bash
npm run test:coverage
```

### Oracle Tests

```bash
cd oracle
npm test
```

### Frontend Tests

```bash
cd apps/web
npm test
```

Run E2E tests (requires running backend, oracle, and deployed contracts):

```bash
npm run test:e2e
```

### Integration Tests

From project root:

```bash
npm run test:integration
```

---

## 🌍 Network Configuration

### Testnet

- **Network Passphrase:** `Test SDF Network ; September 2015`
- **RPC URL:** `https://soroban-testnet.stellar.org:443`
- **Horizon URL:** `https://horizon-testnet.stellar.org`
- **Friendbot:** `https://friendbot.stellar.org`

### Contract Addresses (Testnet)

- **Pool Contract:** `CXXXXXX...` *(Update after deployment)*
- **Governance Contract:** `CXXXXXX...` *(Update after deployment)*
- **USDC Token:** `CBIELTK6YBZJU5UP2WWQEUCYKLPU6AUNZ2BQ4WWFEIE3USCIHMXQDAMA`

---

## 🛡 Coverage Types

| Coverage Type | Trigger Mechanism | Payout Source |
|---|---|---|
| Smart Contract Exploit | On-chain exploit signal + DAO vote | Capital pool |
| Protocol Hack | Oracle-verified loss report + DAO vote | Capital pool |
| Exchange / Custody Failure | Verified insolvency event + DAO vote | Capital pool |
| Stablecoin Depeg | Price oracle breach of peg threshold | Capital pool |
| Validator Slashing | On-chain slashing event detection | Capital pool |

---

## 🗳 Governance & Claims Flow

1. **Coverage Purchase** — User pays a premium to the pool contract to cover a specified asset or protocol for a defined period.
2. **Claim Submission** — In the event of a loss, the user submits a claim with supporting evidence.
3. **DAO Review** — TLR token holders vote to approve or reject the claim within the voting window.
4. **Oracle Verification** — For trigger-based events (depegs, slashing), oracle data is submitted on-chain to validate the claim automatically.
5. **Payout Disbursement** — Approved claims trigger automatic payout from the pool to the claimant's wallet.
6. **Premium Distribution** — Premiums from active policies are periodically distributed to stakers proportional to their stake.

---

## 🐛 Troubleshooting

### Contract Deployment Fails

**Error:** `insufficient balance`

**Solution:** Fund your account using Friendbot:

```bash
curl "https://friendbot.stellar.org?addr=$(stellar keys address deployer)"
```

### Frontend Can't Connect to Wallet

**Error:** `Failed to connect wallet`

**Solution:**
1. Ensure you have Freighter wallet installed
2. Switch wallet to Testnet network
3. Confirm `NEXT_PUBLIC_STELLAR_NETWORK=testnet` is set in `.env.local`

### Oracle Not Pushing Data

**Error:** `Oracle feed timeout`

**Solution:**
1. Verify `SOROBAN_RPC_URL` is correct in `oracle/.env`
2. Confirm the oracle signer account is funded on testnet
3. Check Stellar testnet status: https://status.stellar.org

### Backend Can't Index Events

**Error:** `RPC connection timeout`

**Solution:**
1. Verify RPC URL is correct in `apps/api/.env`
2. Check Stellar testnet status: https://status.stellar.org
3. Try alternative RPC: `https://soroban-testnet.stellar.org:443`

### Contract Build Fails

**Error:** `wasm32-unknown-unknown target not found`

**Solution:** Add wasm target:

```bash
rustup target add wasm32-unknown-unknown
```

### Tests Failing

**Error:** `Network connection error`

**Solution:** Ensure contracts are deployed and all environment variables are set correctly across `apps/api/.env`, `oracle/.env`, and `apps/web/.env.local`.

---

## 📚 Documentation & Resources

- **Stellar Documentation:** [developers.stellar.org](https://developers.stellar.org/docs/build/smart-contracts)
- **Soroban Docs:** [soroban.stellar.org/docs](https://soroban.stellar.org/docs)
- **Contract Architecture:** [contracts/README.md](./contracts/README.md)
- **Oracle Design:** [oracle/README.md](./oracle/README.md)
- **Governance Guide:** [docs/governance.md](./docs/governance.md)
- **Soroban Examples:** [github.com/stellar/soroban-examples](https://github.com/stellar/soroban-examples)

---

## 🤝 Contributing

See our detailed [CONTRIBUTING.md](CONTRIBUTING.md) for coding standards (Rust/Soroban, TypeScript), Git workflow, naming conventions, and the full PR process.

---

## 🗺 Roadmap

### Current Phase (Q2 2026)
- ✅ Core pool and governance contracts
- ✅ Basic coverage purchase and claims flow
- 🚧 Oracle integration for depeg and slashing triggers
- 🚧 DAO voting interface

### Next Phase (Q3 2026)
- TLR token launch and staking rewards
- Risk marketplace UI
- Claims history and analytics dashboard
- Mobile app (Flutter)
- Mainnet deployment

### Future
- Cross-chain coverage pools
- Parametric insurance products
- Automated claims via oracle-only triggers
- Advanced risk pricing models
- Reinsurance layer

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Stellar Development Foundation for the Soroban platform
- Nexus Mutual for pioneering decentralized insurance concepts
- Drips Wave for grants and support
- Open-source contributors and community testers

---

## 📞 Support

Need help? Here's how to get support:

1. Check the [Troubleshooting](#-troubleshooting) section
2. Search [existing issues](https://github.com/your-org/trustlayer/issues)
3. Open a [new issue](https://github.com/your-org/trustlayer/issues/new) with detailed information
4. Join our [Discord community](https://discord.gg/trustlayer) *(if available)*

---

**Built with ❤️ on Stellar**
