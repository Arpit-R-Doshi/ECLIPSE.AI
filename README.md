# ECLIPSE.AI

**A Decentralized AI Model Marketplace on Polygon**

ECLIPSE.AI is a full-stack Web3 platform that enables AI model owners to publish, monetize, and serve their models through a trustless blockchain network. Users can discover models, subscribe with ECL tokens, and run inference -- all without intermediaries. Every transaction is recorded on-chain, ensuring transparent revenue distribution and verifiable usage.

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Smart Contracts](#smart-contracts)
- [Token Economics](#token-economics)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Smart Contract Deployment](#smart-contract-deployment)
- [Running the Backend](#running-the-backend)
- [Running the Frontend](#running-the-frontend)
- [API Reference](#api-reference)
- [License](#license)

---

## Architecture Overview

```
                         Polygon Amoy (Testnet)
                    +--------------------------+
                    |  SYN3RGYToken (ERC-20)   |
                    |  PaymentManager          |
                    |  ModelRegistry           |
                    |  PromptExecution         |
                    +-----------+--------------+
                                |
          +---------------------+---------------------+
          |                                           |
+---------v----------+                     +----------v---------+
|   React Frontend   |  <--- REST API ---> |   Node.js Backend  |
|   (Vite + wagmi)   |                     |   (Express + SQLite)|
+--------------------+                     +----------+---------+
                                                      |
                                           +----------v---------+
                                           |   Ollama Runtime   |
                                           |   (LLM Inference)  |
                                           +--------------------+
```

The frontend communicates with the backend over REST. Wallet interactions (connect, subscribe, approve tokens) go directly from the browser to Polygon via MetaMask and RainbowKit. The backend maintains a SQLite read-index that mirrors on-chain state for fast querying, and proxies inference requests to an Ollama compute node.

---

## Tech Stack

### Frontend
| Technology | Version | Purpose |
|---|---|---|
| React | 19.x | UI framework |
| Vite | 8.x | Build tool and dev server |
| wagmi | 2.19.x | React hooks for Ethereum |
| viem | 2.38.x | TypeScript Ethereum interface |
| RainbowKit | 2.2.x | Wallet connection UI |
| TanStack Query | 5.x | Async state management |
| Framer Motion | 12.x | Animations |
| React Router | 7.x | Client-side routing |
| react-hot-toast | 2.x | Notifications |

### Backend
| Technology | Version | Purpose |
|---|---|---|
| Node.js | 22.x | Runtime |
| Express | 5.x | HTTP framework |
| better-sqlite3 | 12.x | Embedded database |
| ethers.js | 6.x | Blockchain interaction |
| Axios | 1.x | HTTP client (Ollama, IPFS) |
| Multer | 2.x | File upload handling |
| CryptoJS | 4.x | AES-256 encryption |
| Razorpay | 2.x | Fiat-to-token payments |

### Smart Contracts
| Technology | Version | Purpose |
|---|---|---|
| Solidity | 0.8.20 | Contract language |
| Foundry | latest | Build and test framework |
| OpenZeppelin | 5.x | Audited contract libraries |

### Infrastructure
| Technology | Purpose |
|---|---|
| Polygon Amoy | Layer-2 testnet |
| Pinata / IPFS | Decentralized model storage |
| Ollama | Local LLM inference runtime |
| MetaMask | Browser wallet |

---

## Project Structure

```
ECLIPSE.AI/
|
|-- contracts/                  # Solidity smart contracts
|   |-- SYN3RGYToken.sol        # ERC-20 token (ECL)
|   |-- PaymentManager.sol      # Subscription & pay-per-use payments
|   |-- ModelRegistry.sol       # On-chain model registration
|   |-- PromptExecution.sol     # On-chain prompt logging
|
|-- backend/
|   |-- src/
|   |   |-- server.js           # Express entry point
|   |   |-- db/sqlite.js        # Database schema and queries
|   |   |-- routes/
|   |   |   |-- models.js       # CRUD for AI models
|   |   |   |-- inference.js    # Ollama inference proxy
|   |   |   |-- execution.js    # Prompt execution with on-chain logging
|   |   |   |-- subscriptions.js# Subscription sync and queries
|   |   |   |-- payments.js     # Razorpay + ECL token purchases
|   |   |   |-- wallet.js       # Wallet sync, balance, faucet
|   |   |   |-- apikeys.js      # API key management
|   |   |   |-- history.js      # Prompt history queries
|   |   |-- services/
|   |       |-- blockchain.js   # Ethers.js contract interactions
|   |       |-- compute.js      # Ollama node management
|   |       |-- encryption.js   # AES-256 encryption utilities
|   |       |-- ipfs.js         # Pinata IPFS upload/retrieval
|   |       |-- vision.js       # Image analysis for vision models
|
|-- frontend/
|   |-- src/
|   |   |-- main.jsx            # App entry, wagmi + RainbowKit providers
|   |   |-- App.jsx             # Router, wallet state, global context
|   |   |-- index.css           # Global design system
|   |   |-- pages/
|   |   |   |-- Landing.jsx     # Marketing landing page
|   |   |   |-- RoleSelect.jsx  # User / Owner role selection
|   |   |   |-- Marketplace.jsx # Browse and search models
|   |   |   |-- ModelDetail.jsx # Model info, subscribe, chat interface
|   |   |   |-- Dashboard.jsx   # User dashboard with analytics
|   |   |   |-- OwnerDashboard.jsx # Owner revenue and model management
|   |   |   |-- UploadModel.jsx # Model upload with IPFS and encryption
|   |   |   |-- ChatHistory.jsx # Past prompt/response history
|   |   |-- components/
|   |       |-- Navbar.jsx      # Navigation with RainbowKit ConnectButton
|   |       |-- BuyECLModal.jsx # Razorpay-powered token purchase
|   |       |-- CashoutModal.jsx# ECL to fiat withdrawal
|   |       |-- WebGLShader.jsx # Animated background shader
|
|-- scripts/
|   |-- deploy.js               # Hardhat deployment script
|
|-- .env.example                # Environment variable template
|-- foundry.toml                # Foundry configuration
```

---

## Smart Contracts

All contracts are deployed to **Polygon Amoy** testnet.

### SYN3RGYToken (ECL)
ERC-20 token with an initial supply of 1,000,000 ECL minted to the deployer. Used as the native payment currency across the marketplace.

### PaymentManager
Handles both subscription and pay-per-use payments with an automated revenue split:
- **85%** to the model owner
- **10%** to the compute node (pay-per-use only)
- **5%** to the platform treasury

### ModelRegistry
On-chain registry for AI models. Stores model metadata, IPFS CIDs, and ownership. Supports ownership transfers.

### PromptExecution
Immutable on-chain log of inference requests. Records the model used, input hash, output hash, cost, and timestamp for auditability.

---

## Token Economics

| Action | Subscriber | Model Owner | Platform |
|---|---|---|---|
| Subscription | -price ECL | +85% of price | +15% of price |
| Pay-per-use | -cost ECL | +85% of cost | +5% of cost |
| Faucet (testnet) | +100 ECL | -- | -- |
| Buy ECL (Razorpay) | +purchased amount | -- | -- |

---

## Prerequisites

Before running the project, ensure the following are installed:

- **Node.js** v22 or later -- [nodejs.org](https://nodejs.org/)
- **npm** v10 or later (bundled with Node.js)
- **Foundry** (for smart contract compilation) -- [getfoundry.sh](https://getfoundry.sh/)
- **Ollama** (for local LLM inference) -- [ollama.com](https://ollama.com/)
- **MetaMask** browser extension -- [metamask.io](https://metamask.io/)
- A **WalletConnect Project ID** -- [cloud.reown.com](https://cloud.reown.com/) (free)

---

## Environment Setup

1. Clone the repository:

```bash
git clone https://github.com/Arpit-R-Doshi/ECLIPSE.AI.git
cd ECLIPSE.AI
```

2. Copy the environment template and fill in your values:

```bash
cp .env.example .env
```

3. Edit `.env` with your configuration:

```env
# Blockchain
DEPLOYER_PRIVATE_KEY=your_private_key_here
POLYGON_AMOY_RPC=https://rpc-amoy.polygon.technology/

# Contract Addresses (filled after deployment)
SYN3RGY_TOKEN_ADDRESS=
MODEL_REGISTRY_ADDRESS=
PAYMENT_MANAGER_ADDRESS=
PROMPT_EXECUTION_ADDRESS=

# IPFS (Pinata)
PINATA_API_KEY=your_pinata_api_key
PINATA_SECRET_KEY=your_pinata_secret_key
PINATA_JWT=your_pinata_jwt

# Ollama Compute Node
OLLAMA_URL=http://localhost:11434

# Server
PORT=3001
FRONTEND_URL=http://localhost:5173

# Encryption
MASTER_ENCRYPTION_KEY=generate_a_random_32_byte_hex_string
```

---

## Smart Contract Deployment

1. Install Foundry dependencies:

```bash
forge install
```

2. Compile contracts:

```bash
forge build
```

3. Deploy to Polygon Amoy:

```bash
npx hardhat run scripts/deploy.js --network amoy
```

4. Copy the deployed contract addresses from the console output into your `.env` file under the `Contract Addresses` section.

---

## Running the Backend

1. Navigate to the backend directory and install dependencies:

```bash
cd backend
npm install
```

2. Start the development server:

```bash
npm run dev
```

The backend will start on `http://localhost:3001`. On first run, it automatically creates the SQLite database.

3. Seed the demo models (one-time):

Open a browser or run:

```bash
curl http://localhost:3001/api/seed
```

This creates two demo models (Gemma 2B and Llama 3 8B) in the database.

4. Ensure Ollama is running with at least one model pulled:

```bash
ollama pull gemma:2b
ollama pull llama3:8b
```

---

## Running the Frontend

1. Navigate to the frontend directory and install dependencies:

```bash
cd frontend
npm install
```

2. Start the development server:

```bash
npm run dev
```

The frontend will start on `http://localhost:5173`.

3. Open `http://localhost:5173` in a browser with MetaMask installed.

4. Connect your wallet, select a role (User or Model Owner), and begin interacting with the marketplace.

---

## API Reference

### Health and Seed
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/health` | Health check |
| GET | `/api/config` | Frontend configuration (ABIs, addresses) |
| GET | `/api/seed` | Seed demo models into the database |

### Models
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/models` | List all models |
| GET | `/api/models/:id` | Get model by ID |
| POST | `/api/models/upload` | Upload a new model (multipart) |
| DELETE | `/api/models/:id` | Delete a model (owner only) |

### Inference
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/inference` | Run inference on a model |

### Subscriptions
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/subscriptions/sync` | Sync on-chain subscription to local index |
| GET | `/api/subscriptions/check/:address/:modelId` | Check subscription status |
| GET | `/api/subscriptions/user/:address` | List user subscriptions |
| GET | `/api/subscriptions/owner/:address` | Owner subscription stats |

### Wallet
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/wallet/sync` | Sync wallet address to backend |
| GET | `/api/wallet/balance/:address` | Get platform balance |
| POST | `/api/wallet/faucet` | Claim 100 ECL (testnet) |

### Payments
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/payments/create-order` | Create Razorpay order |
| POST | `/api/payments/verify` | Verify payment and credit ECL |
| POST | `/api/payments/cashout` | Withdraw ECL to fiat |

---

## License

This project is licensed under the ISC License.
