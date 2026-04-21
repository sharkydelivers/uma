# APPENDIX: Reproducible Queries & Raw Evidence

*Every claim in this investigation can be verified by running the queries below. No API keys needed for subgraph queries. Etherscan queries need a free API key (get one at etherscan.io/apis).*

---

## 1. How Many DV2 Contracts Were Deployed?

### Query (Etherscan — count internal transactions from factory)

```bash
curl "https://api.etherscan.io/v2/api?chainid=1&module=account&action=txlistinternal&address=0x6b46a05f7f9f73d927abd99f5cb5c5652944c94f&startblock=0&endblock=99999999&page=1&offset=1000&sort=asc&apikey=YOUR_KEY"
```

### Result
- **89 total contract creations** (all `type: "create"`)
- First: 2023-02-25 (Block 16,705,595)
- Last: 2026-02-18 (Block 24,481,311)

### Who Called the Factory?

```bash
curl "https://api.etherscan.io/v2/api?chainid=1&module=account&action=txlist&address=0x6b46a05f7f9f73d927abd99f5cb5c5652944c94f&startblock=0&endblock=99999999&page=1&offset=1000&sort=asc&apikey=YOUR_KEY"
```

**IMPORTANT CORRECTION**: The factory is **permissionless**. 36 unique addresses called it:
- `0x9a8f92a830a5cb89a3816e3d267cb7791c16b04d` ("Risk Labs: Deployer" — Etherscan-labeled): **27 transactions → 52 contracts** (via multicall batches)
- **35 other addresses**: 1-2 transactions each → **37 contracts**

**What this means**: 52 contracts were deployed by Risk Labs. The other 37 were deployed by independent users. The factory function `newDesignatedVoting(address owner, address voter)` lets the caller specify the owner — so we cannot assume all 89 are Safe-owned.

### Verify "Risk Labs: Deployer" Label

Go to: https://etherscan.io/address/0x9a8f92a830a5cb89a3816e3d267cb7791c16b04d
You will see Etherscan's official label: "Risk Labs: Deployer"

---

## 2. Does the Safe Own a Specific DV2 Contract?

### Query (Etherscan — Read Contract)

Go to any DV2 contract → "Read as Proxy" tab → function `holdsRole`:
- `roleId`: 0 (means "Owner")
- `account`: 0x8180d59b7175d4064bdfa8138a58e9babffda44a

**Direct link (example — top DV2 contract):**
https://etherscan.io/address/0x0afc9a935960948ed35e50ac7639cbb38e840b38#readProxyContract

### Result for tested contracts

| Contract | Deployed By | holdsRole(0, Safe) | Stake (UMA) |
|----------|------------|-------------------|-------------|
| 0x0afc9a935960948ed35e50ac7639cbb38e840b38 | Risk Labs: Deployer | **TRUE** | 831,304 |
| 0x487a5bbdc6091be0e27c0e8ea283047d94862538 | Risk Labs: Deployer | **TRUE** | 678,803 |
| 0x326c8193b3e45e6a3faae69606092e131028e257 | Risk Labs: Deployer | **TRUE** | 449,359 |
| 0x6564c6c4601ee8610b54086daa3647a01e44e8c1 | Risk Labs: Deployer | **TRUE** | 440,185 |
| 0x66ca133ed3ed3e7b6b03e2cccb6265ead8cd8b73 | Risk Labs: Deployer | **TRUE** | 415,169 |
| 0xf659a672f0c8b6ead04fc06e6c1c2e7ab3d29e1e | Risk Labs: Deployer | **TRUE** | 281,786 |
| 0x840bf8f0cf24e99cb66cdf402e73ebc1fcea1674 | Risk Labs: Deployer | **TRUE** | 253,090 |
| 0xbd9e1684e95ec7b1ffa1ff2345f62ede1b38dae0 | Risk Labs: Deployer | **TRUE** | 3.95 (emptied) |

**UPDATE (April 21, 2026): EXHAUSTIVE VERIFICATION COMPLETE.**

We called `holdsRole(0, 0x8180d59b7175d4064bdfa8138a58e9babffda44a)` on **ALL 89 factory contracts** using the correct function selector `0x7cdc1cb9` (keccak256 of `holdsRole(uint256,address)`).

**Result: 86 TRUE, 3 FALSE.**

The 3 FALSE contracts:
| Contract | Owner (via getMember(0)) | Deployed By |
|----------|------------------------|-------------|
| 0x2ba1e904... | 0x9a8f92a830a5cb89a3816e3d267cb7791c16b04d (Risk Labs: Deployer) | Risk Labs: Deployer |
| 0xdd781a04... | 0xb51d0852... (independent user) | 0xb51d0852... |
| 0xdef630ea... | 0x1076188b... (independent user) | 0x1076188b... |

**Bottom line: 86 of 89 (96.6%) are Safe-owned. Plus 1 more owned by Risk Labs deployer directly = 87/89 (97.8%) in the Risk Labs ecosystem.**

### How to reproduce the exhaustive check

```bash
# For each of the 89 contract addresses, call:
curl -s "https://api.etherscan.io/v2/api?chainid=1&module=proxy&action=eth_call&to=CONTRACT_ADDRESS&data=0x7cdc1cb900000000000000000000000000000000000000000000000000000000000000000000000000000000000000008180d59b7175d4064bdfa8138a58e9babffda44a&tag=latest&apikey=YOUR_KEY"

# Result ending in ...0001 = TRUE (Safe owns it)
# Result ending in ...0000 = FALSE (Safe does NOT own it)
```

**IMPORTANT**: The correct function selector is `0x7cdc1cb9` (NOT `0xe24b8e2d` which some tools compute incorrectly). Verify: `keccak256("holdsRole(uint256,address)")[:4] = 0x7cdc1cb9`.

---

## 3. Who Signs the Safe?

### Query (Etherscan — Read as Proxy → getOwners)

**Direct link:**
https://etherscan.io/address/0x8180d59b7175d4064bdfa8138a58e9babffda44a#readProxyContract

Call `getOwners()`. Returns:
```
[
  "0xcc400c09ecbac3e0033e4587bdfaabb26223e37d",
  "0x363605c0bde9f1f5053ada30618d95dbfc109bf5",
  "0x837219d7a9c666f5542c4559bf17d7b804e5c5fe",
  "0x72b32c1a6a75cbafae36c0ca8e763946d370e766"
]
```

Call `getThreshold()`. Returns: `2`

### Proving Signer #1 = Hart Lambur

Go to: https://etherscan.io/address/0xcc400c09ecbac3e0033e4587bdfaabb26223e37d
In "More Info" section: **"Funded By: hal2001.eth"**

Verify hal2001.eth = Hart Lambur:
- https://etherscan.io/name-lookup-search?id=hal2001.eth
- Hart Lambur's public ENS name (UMA/Across Protocol co-founder, ex-Goldman Sachs)

---

## 4. Vote Correlation (Subgraph Proof)

### Query — Same-round vote reveals for 4 wallets

```bash
curl -X POST \
  "https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ revealedVotes(where: { voter_in: [\"0x0afc9a935960948ed35e50ac7639cbb38e840b38\", \"0x487a5bbdc6091be0e27c0e8ea283047d94862538\", \"0x326c819380b8842e3a55e5f97f6011ee91844205\", \"0xd2a78bb82389d30075144d17e782964918999f7f\"], roundId: \"10275\" }, first: 100) { voter { id } time price roundId identifier } }"}'
```

### Result (Round 10275 — cleanest proof)

10 unique price requests in this round. All 4 wallets voted the EXACT same `price` value on all 10:

| Request | 0x0afc (DV2) | 0x487a (DV2) | 0x326c (DV2) | Kevin Chan | Match? |
|---------|-------------|-------------|-------------|-----------|--------|
| 1 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | YES |
| 2 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | YES |
| 3 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | YES |
| ... | ... | ... | ... | ... | ... |
| 10 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | 1000000000000000000 | YES |

**100% identical votes across all 4 wallets in all 10 price requests.**

### Additional rounds tested

| Round | Shared Requests | 4-wallet Agreement | Disagreements |
|-------|----------------|-------------------|---------------|
| 10275 | 10 | 10/10 (100%) | 0 |
| 10278 | 20+ | ~95%+ | 1-2 IGNORE divergences |
| 10279 | 20+ | ~95%+ | 1-2 IGNORE divergences |
| 10280 | 22 | 18/22 (82%) | 4 (partial participation) |

**NOTE on "same-second" claim**: The subgraph `time` field is the price request timestamp, NOT the block time of the reveal transaction. For same-block proof, Etherscan transaction data is needed. What the subgraph DOES prove is **100% vote value agreement** — they always choose the same answer. The "same-second" commit/reveal claim requires checking Etherscan block data for the multicall transactions.

---

## 5. Kevin Chan / maxodds.eth Stats

### Query

```bash
curl -X POST \
  "https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ voter(id: \"0xd2a78bb82389d30075144d17e782964918999f7f\") { id stake countReveals countCorrectVotes countWrongVotes withdrawnRewards } }"}'
```

### Response (raw)

```json
{
  "data": {
    "voter": {
      "id": "0xd2a78bb82389d30075144d17e782964918999f7f",
      "stake": "1520241039633693643505873",
      "countReveals": 3101,
      "countCorrectVotes": 2965,
      "countWrongVotes": 37,
      "withdrawnRewards": "927785000000000000000000"
    }
  }
}
```

**Interpretation**:
- Stake: 1,520,241 UMA (divide by 10^18)
- 3,101 vote reveals
- 95.6% correct (2,965/3,101)
- 927,785 UMA in rewards withdrawn

---

## 6. Top DV2 Contract Stats

### Query

```bash
curl -X POST \
  "https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ voter(id: \"0x0afc9a935960948ed35e50ac7639cbb38e840b38\") { id stake countReveals countCorrectVotes countWrongVotes withdrawnRewards } }"}'
```

### Response (raw)

```json
{
  "data": {
    "voter": {
      "id": "0x0afc9a935960948ed35e50ac7639cbb38e840b38",
      "stake": "831303982948209473827394",
      "countReveals": 3069,
      "countCorrectVotes": 2953,
      "countWrongVotes": 35,
      "withdrawnRewards": "383581000000000000000000"
    }
  }
}
```

---

## 7. BornTooLate.eth Reward Flows

### Query (Etherscan — ERC-20 transfers)

```bash
curl "https://api.etherscan.io/v2/api?chainid=1&module=account&action=tokentx&address=0xfc204fcfd2a579157898a212ea25ac98de2b1e1c&contractaddress=0x04fa0d235c4abf4bcf4787af4cf447de572ef828&startblock=0&endblock=99999999&sort=asc&apikey=YOUR_KEY"
```

**Direct Etherscan link:**
https://etherscan.io/address/0xfc204fcfd2a579157898a212ea25ac98de2b1e1c#tokentxns

Filter by UMA token (0x04fa0d235c4abf4bcf4787af4cf447de572ef828).

### Key outgoing transfers to intermediary 0xa99ef3fb

| Date | Amount (UMA) | To |
|------|-------------|-----|
| 2023-08-20 | 83,684 | 0xa99ef3fb |
| 2024-01-15 | 95,338 | 0xa99ef3fb |
| 2024-05-22 | 102,445 | 0xa99ef3fb |
| 2024-09-18 | 115,901 | 0xa99ef3fb |
| 2025-02-14 | 130,282 | 0xa99ef3fb |
| 2025-07-10 | 142,098 | 0xa99ef3fb |
| 2025-11-05 | 158,722 | 0xa99ef3fb |
| 2026-02-18 | 375,000 | 0xa99ef3fb |
| ... | ... | ... |
| **TOTAL** | **~1,354,020** | |

---

## 8. BornTooLate Single-Use Funding Intermediary

### Verify

Go to: https://etherscan.io/address/0xceccfb4a15ad7afbb957bf3af72939db0ccd6ac5

**You will see:**
- Total transactions: **2**
- Tx 1: Received 1.0 ETH from 0xedc7001e (March 12, 2021)
- Tx 2: Sent 0.9956 ETH to 0xfc204fcf (BornTooLate) — 32 seconds later

The wallet has done nothing else in 5+ years.

---

## 9. Hart Lambur Polygon Activity

### Verify

Go to: https://polygonscan.com/address/0xcc400c09ecbac3e0033e4587bdfaabb26223e37d

You will see 9 transactions, including proposePrice() calls to OptimisticOracleV2.

---

## 10. Kevin Chan = maxodds.eth = ACX Scandal Wallet

### Verify

1. Address: https://etherscan.io/address/0xd2a78bb82389d30075144d17e782964918999f7f
2. ENS lookup: https://etherscan.io/name-lookup-search?id=maxodds.eth
3. Ogle's investigation naming this wallet: https://cryptonews.com/news/across-protocol-token-crashes-10-today-amid-23m-team-misappropriation-allegations/
4. CoinTelegraph confirmation: https://cointelegraph.com/news/across-protocol-founders-accused-of-funneling-23m-to-own-company

---

## 11. Total Staked UMA in System

### Query

```bash
curl -X POST \
  "https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ votingV2S { cumulativeStake } }"}'
```

This returns the total UMA staked in the VotingV2 contract.

---

## 12. List All Voters (Top Stakers)

### Query

```bash
curl -X POST \
  "https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn" \
  -H "Content-Type: application/json" \
  -d '{"query":"{ voters(first: 50, orderBy: stake, orderDirection: desc) { id stake countReveals } }"}'
```

This returns the top 50 voters by stake. You can verify our named wallets appear at the top.

---

## Raw Evidence Files (Local)

| File | Contents |
|------|----------|
| `tmp-dv2-internal-txs.json` | All 89 factory internal transactions (contract creations) |
| `tmp-dv2-normal-txs.json` | All 64 normal transactions to the factory |
| `tmp-dv2-evidence.json` | Structured summary with first/last entries |
| `tmp/uma-subgraph-round-10275.json` | Raw vote reveals for round 10275 (4 wallets) |
| `tmp/uma-subgraph-round-10278.json` | Raw vote reveals for round 10278 |
| `tmp/uma-subgraph-round-10279.json` | Raw vote reveals for round 10279 |
| `tmp/uma-subgraph-round-10280.json` | Raw vote reveals for round 10280 |
| `tmp/uma-subgraph-kevin-chan-stats.json` | Kevin Chan voter entity (raw) |
| `tmp/uma-subgraph-top-dv2-stats.json` | Top DV2 voter entity (raw) |
| `tmp/uma-subgraph-proof-appendix.md` | Full analysis with all queries |

---

*All queries above are reproducible by anyone. No authentication needed for subgraph. Free Etherscan API key at etherscan.io/apis.*
