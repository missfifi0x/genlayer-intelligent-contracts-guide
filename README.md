# Getting Started with GenLayer Intelligent Contracts

> A complete guide to building AI-powered smart contracts on [GenLayer Studio](https://studio.genlayer.com/contracts)

[![GenLayer](https://img.shields.io/badge/GenLayer-Studio-blue?style=for-the-badge)](https://studio.genlayer.com/contracts)
[![Docs](https://img.shields.io/badge/Docs-docs.genlayer.com-green?style=for-the-badge)](https://docs.genlayer.com)
[![Discord](https://img.shields.io/badge/Discord-Join%20Community-7289da?style=for-the-badge&logo=discord)](https://discord.gg/8Jm4v89VAu)
[![GitHub](https://img.shields.io/badge/GitHub-genlayerlabs-black?style=for-the-badge&logo=github)](https://github.com/genlayerlabs/)

---

## Table of Contents

- [What is GenLayer?](#-what-is-genlayer)
- [What are Intelligent Contracts?](#-what-are-intelligent-contracts)
- [Prerequisites](#-prerequisites)
- [Getting Started: GenLayer Studio (Browser)](#-getting-started-genlayer-studio-browser)
- [Getting Started: Local Setup (CLI)](#-getting-started-local-setup-cli)
- [Your First Intelligent Contract](#-your-first-intelligent-contract)
- [Core Concepts](#-core-concepts)
- [Contract Examples](#-contract-examples)
- [Deploying Your Contract](#-deploying-your-contract)
- [Building a dApp Frontend](#-building-a-dapp-frontend)
- [Testing](#-testing)
- [Use Cases](#-use-cases)
- [Resources](#-resources)

---

## What is GenLayer?

GenLayer is an **AI-native blockchain** — a new class of distributed ledger technology that goes beyond deterministic smart contracts. While:

- **Bitcoin** delivers trustless money
- **Ethereum** delivers trustless computation
- **GenLayer** delivers **trustless decision-making**

GenLayer validators are powered by diverse AI/LLM models that reach consensus on complex, subjective decisions. Contracts can natively access the internet, process natural language, and handle non-deterministic logic all without relying on traditional oracle services.

---

## What are Intelligent Contracts?

Intelligent Contracts are an advanced levels of smart contracts that combine:

| Feature | Description |
|---|---|
|  **LLM Integration** | Use GPT-4, Llama, and other models for natural language reasoning |
|  **Web Access** | Fetch live internet data directly on-chain, no oracles needed |
|  **Python-based** | Write in Python, not Solidity |
|  **Optimistic Democracy** | Multiple validators reach consensus via AI-driven voting |
|  **Subjective Decisions** | Handle ambiguity, nuance, and real-world context |

---

## Prerequisites

### For Browser-Based (Studio) — No setup required!
- A modern web browser (Chrome, Firefox, Edge)
- That's it! Go to [studio.genlayer.com/contracts](https://studio.genlayer.com/contracts)

### For Local Development
- **Docker** 26+  [Install Docker](https://docs.docker.com/get-docker/)
- **Node.js** 18+ and npm  [Install Node.js](https://nodejs.org/)
- **Python** 3.11+ (for writing contracts locally)

---

##  Getting Started: GenLayer Studio (Browser)

The fastest way to get started is through the **GenLayer Studio** , a fully browser-based IDE for writing, deploying, and testing Intelligent Contracts.

### Step 1 ; Open the Studio

Visit: **[https://studio.genlayer.com/contracts](https://studio.genlayer.com/contracts)**

You'll see a preloaded environment with:
- A code editor panel
- A contract deployment panel
- A transaction execution panel
- Live validator logs

### Step 2 ; Explore Example Contracts

The Studio comes preloaded with example contracts to help you get started:

- `Storage` ; basic state management
- `LLM Hello World` ; calling an LLM from a contract
- `Wizard of Coin` ; AI decision-making game
- `Fetch Web Content` ; reading live internet data
- `Football Prediction Market` ; a real-world prediction market example

Click any example in the **Contracts** sidebar to load and explore its code.

### Step 3 ; Deploy a Contract

1. Select or write a contract in the editor
2. Click the  **Run and Deploy** tab
3. Fill in any constructor parameters (if needed)
4. Click **Deploy**
5. The contract address will appear upon successful deployment

### Step 4 ; Interact with Your Contract

Once deployed, you can:
- **Read Methods** ; View state without modifying it
- **Write Methods** ; Execute transactions that modify state
- **Node Logs** ; Watch validators reach consensus in real time

---

##  Getting Started: Local Setup (CLI)

For local development with full tooling support:

### Step 1 ; Install the GenLayer CLI

```bash
npm install -g genlayer
```

### Step 2 ; Initialize a New Project

```bash
genlayer init my-intelligent-contract
cd my-intelligent-contract
```

### Step 3 ; Start the Local Studio

```bash
genlayer up
```

This pulls the required Docker images and starts a local GenLayer node. Once running, open your browser at:

```
http://localhost:8080/
```

### Step 4 ; Configure LLM Providers

In the Studio UI, navigate to **Providers** to configure which LLM backends your validators will use (e.g., OpenAI, Ollama, Heurist).

---

##  Your First Intelligent Contract

### Hello World (Simple)

```python
# { "Depends": "py-genlayer:test" }
from genlayer import *

class HelloWorld(gl.Contract):
    greeting: str

    def __init__(self):
        self.greeting = "Hello, GenLayer!"

    @gl.public.view
    def get_greeting(self) -> str:
        """Returns the current greeting (read-only)."""
        return self.greeting

    @gl.public.write
    def set_greeting(self, new_greeting: str):
        """Updates the greeting message."""
        self.greeting = new_greeting
```

### Hello World (With LLM)

```python
# { "Depends": "py-genlayer:test" }
from genlayer import *

class LLMHelloWorld(gl.Contract):
    response: str

    def __init__(self):
        self.response = ""

    @gl.public.write
    def ask_question(self, question: str):
        """Uses an LLM to answer a natural language question."""
        def get_llm_response() -> str:
            result = gl.exec_prompt(
                f"Answer this question in one sentence: {question}"
            )
            return result

        self.response = gl.eq_principle_strict_eq(get_llm_response)

    @gl.public.view
    def get_response(self) -> str:
        return self.response
```

### Fetching Web Data

```python
# { "Depends": "py-genlayer:test" }
from genlayer import *

class WebFetcher(gl.Contract):
    result: str

    def __init__(self):
        self.result = ""

    @gl.public.write
    def fetch_and_analyze(self, url: str, question: str):
        """Fetches a webpage and answers a question about it using an LLM."""
        def analyze() -> str:
            page_content = gl.get_webpage(url, mode="text")
            return gl.exec_prompt(
                f"Based on this content:\n\n{page_content}\n\nAnswer: {question}"
            )

        self.result = gl.eq_principle_strict_eq(analyze)

    @gl.public.view
    def get_result(self) -> str:
        return self.result
```

---

##  Core Concepts

### Optimistic Democracy

GenLayer uses a multi-validator consensus mechanism called **Optimistic Democracy**:

1. A transaction is submitted and enters a `pending` state
2. A randomly selected group of validators is assigned
3. One validator acts as **leader** and proposes an outcome
4. Other validators evaluate the proposal independently
5. Consensus is reached using the **Equivalence Principle**
6. The transaction is finalized

### Equivalence Principle

Because LLMs are non-deterministic (each validator may produce slightly different outputs), GenLayer uses the Equivalence Principle to define what counts as "the same" answer:

```python
# Strict equality — all validators must return the exact same value
self.result = gl.eq_principle_strict_eq(my_non_deterministic_block)

# Prompt-based equivalence , LLM judges whether outputs are equivalent
self.decision = gl.eq_principle_prompt_comparative(
    my_non_deterministic_block,
    "Are these two answers saying the same thing?"
)
```

### Contract Decorators

| Decorator | Description |
|---|---|
| `@gl.public.view` | Read-only method, does not modify state |
| `@gl.public.write` | Modifies contract state, requires consensus |

### Storage Types

```python
class MyContract(gl.Contract):
    # Primitive types
    count: int
    name: str
    active: bool

    # Collections
    items: DynArray[str]
    scores: TreeMap[str, int]

    # Complex types
    user_data: TreeMap[Address, UserInfo]
```

---

##  Contract Examples

### Prediction Market

```python
# { "Depends": "py-genlayer:test" }
from genlayer import *

class PredictionMarket(gl.Contract):
    question: str
    resolved: bool
    outcome: str

    def __init__(self, question: str):
        self.question = question
        self.resolved = False
        self.outcome = ""

    @gl.public.write
    def resolve(self, data_source_url: str):
        """Resolves the market by fetching real-world data."""
        def determine_outcome() -> str:
            page = gl.get_webpage(data_source_url, mode="text")
            result = gl.exec_prompt(
                f"Based on the following data:\n{page}\n\n"
                f"Answer yes or no to: {self.question}\n"
                f"Respond with only 'yes' or 'no'."
            )
            return result.strip().lower()

        self.outcome = gl.eq_principle_strict_eq(determine_outcome)
        self.resolved = True

    @gl.public.view
    def get_outcome(self) -> str:
        return self.outcome if self.resolved else "Not yet resolved"
```

### Dispute Resolution

```python
# { "Depends": "py-genlayer:test" }
from genlayer import *

class DisputeResolver(gl.Contract):
    verdict: str
    reasoning: str

    def __init__(self):
        self.verdict = ""
        self.reasoning = ""

    @gl.public.write
    def resolve_dispute(
        self,
        party_a_claim: str,
        party_b_claim: str,
        evidence_url: str
    ):
        """Uses AI consensus to resolve a dispute between two parties."""
        def analyze_dispute() -> str:
            evidence = gl.get_webpage(evidence_url, mode="text")
            return gl.exec_prompt(
                f"You are an impartial arbitrator. Based on the evidence provided, "
                f"determine who is correct and explain why.\n\n"
                f"Party A claims: {party_a_claim}\n"
                f"Party B claims: {party_b_claim}\n"
                f"Evidence: {evidence}\n\n"
                f"Respond in JSON format: "
                f'{{ "winner": "A or B", "reasoning": "your explanation" }}'
            )

        result = gl.eq_principle_strict_eq(analyze_dispute)
        parsed = json.loads(result)
        self.verdict = parsed["winner"]
        self.reasoning = parsed["reasoning"]
```

---

##  Deploying Your Contract

### Via Studio UI (Recommended for Beginners)

1. Open [studio.genlayer.com/contracts](https://studio.genlayer.com/contracts)
2. Load your contract file
3. Click **Deploy** in the Run & Deploy panel
4. Enter constructor arguments if prompted
5. Copy the deployed contract address

### Via CLI

```bash
# Deploy a single contract
genlayer deploy --contract contracts/my_contract.py --args "Hello World"

# Deploy with a deploy script (for complex multi-contract setups)
genlayer deploy --script deploy/deploy.ts
```

### Network Configuration

Configure your target network in `genlayer.config.ts`:

```typescript
import { createClient, testnet } from "genlayer-js";

export default {
  network: testnet,    // or localnet for local development
  privateKey: process.env.PRIVATE_KEY,
};
```

Available networks:
- `localnet` — Your local Studio instance
- `testnet` — GenLayer Testnet Bradbury (public)

---

##  Building a dApp Frontend

Use **GenLayer JS** to connect a frontend to your Intelligent Contract:

### Installation

```bash
npm install genlayer-js
```

### Reading Contract State

```typescript
import { createClient, testnet } from "genlayer-js";

const client = createClient({ network: testnet });

const result = await client.readContract({
  address: "0xYourContractAddress",
  functionName: "get_greeting",
  args: [],
});

console.log(result); // "Hello, GenLayer!"
```

### Writing to a Contract

```typescript
import { createClient, testnet, simulator } from "genlayer-js";

const client = createClient({
  network: testnet,
  account: privateKeyToAccount(process.env.PRIVATE_KEY),
});

const txHash = await client.writeContract({
  address: "0xYourContractAddress",
  functionName: "set_greeting",
  args: ["Hello from the frontend!"],
});

// Wait for consensus
const receipt = await client.waitForTransactionReceipt({ hash: txHash });
console.log("Transaction finalized:", receipt);
```

---

##  Testing

GenLayer provides a Python testing framework via `pytest`:

```python
import pytest
from genlayer import GenLayer, Account

def test_hello_world_contract():
    account = create_new_account()

    # Deploy
    contract_code = open("contracts/hello_world.py", "r").read()
    contract_address, _ = deploy_intelligent_contract(
        account,
        contract_code,
        '{"greeting": "Hello!"}',
    )

    # Read initial state
    greeting = call_contract_method(contract_address, account, "get_greeting", [])
    assert greeting == "Hello!"

    # Write and verify
    send_transaction(account, contract_address, "set_greeting", ["Updated!"])
    updated = call_contract_method(contract_address, account, "get_greeting", [])
    assert updated == "Updated!"
```

Run tests:
```bash
pytest tests/ -v
```

---

##  Use Cases

GenLayer Intelligent Contracts unlock a new generation of applications:

| Use Case | Description |
|---|---|
|  **Dispute Resolution** | AI validators evaluate evidence and settle disputes autonomously |
|  **Prediction Markets** | Self-settle by fetching real-world outcomes from primary sources |
|  **AI-Governed DAOs** | Natural language proposals analyzed and auto-enforced |
|  **Autonomous Escrow** | Release payments when fulfillment conditions are verified by AI |
|  **Compliance Screening** | Real-time KYC/AML verification against sanctions lists |
|  **Parametric Insurance** | Instant payouts triggered by verifiable real-world events |
| **ADR (Arbitration)** | Decentralized, low-cost alternative to traditional legal arbitration |
| **Contract Automation** | AI interprets ambiguous terms like "force majeure" on-chain |

---

## Project Structure

A typical GenLayer project looks like:

```
my-genlayer-project/
├── contracts/
│   ├── my_contract.py          # Your Intelligent Contract
│   └── helper_contract.py      # Supporting contracts
├── deploy/
│   └── deploy.ts               # Deployment scripts
├── frontend/
│   ├── src/
│   │   └── App.tsx             # Frontend dApp
│   └── package.json
├── tests/
│   └── test_my_contract.py     # Pytest test suite
├── genlayer.config.ts          # Network configuration
└── README.md
```

---

## Resources

| Resource | Link |
|---|---|
|  GenLayer Studio | [studio.genlayer.com/contracts](https://studio.genlayer.com/contracts) |
|  Official Docs | [docs.genlayer.com](https://docs.genlayer.com) |
|  GitHub | [github.com/genlayerlabs](https://github.com/genlayerlabs/) |
|  Discord | [discord.gg/8Jm4v89VAu](https://discord.gg/8Jm4v89VAu) |
|  Twitter/X | [@GenLayer](https://x.com/GenLayer) |
|  Whitepaper | [genlayer.com/whitepaper](https://www.genlayer.com/whitepaper) |
|  Ask GPT (DeepThought) | [ChatGPT GenLayer Assistant](https://chatgpt.com/g/g-67a1d29d11ec81918639203621210149-deepthought-genlayer) |
| 🧪 Testnet Info | [genlayer.com/testnet](https://www.genlayer.com/testnet) |

---

##  Contributing

We welcome contributions! To get involved:

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/my-contract`
3. Add your Intelligent Contract or improvement
4. Submit a pull request

For major changes, you can open an issue first to discuss what you'd like to change.

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details. I had to check YT for this

---

<div align="center">

**Built with  on GenLayer , The Intelligence Layer of the Internet**

[Try the Studio →](https://studio.genlayer.com/contracts)

</div>
