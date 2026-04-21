# How to Verify This Investigation (5-Minute Guide)

No crypto expertise needed. Just click links and check what Etherscan shows you.

---

## Claim 1: "Risk Labs deployed the voting contracts"

**Verify:**
1. Go to: https://etherscan.io/address/0x9a8f92a830a5cb89a3816e3d267cb7791c16b04d
2. Look at the address label at the top. Etherscan officially labels it: **"Risk Labs: Deployer"**
3. Click "Internal Txns" tab → you'll see it creating contracts via the factory

That's it. Etherscan itself confirms who deployed these.

---

## Claim 2: "89 contracts are all owned by 1 Safe"

**Verify:**
1. Go to any DV2 contract, e.g.: https://etherscan.io/address/0x0afc9a935960948ed35e50ac7639cbb38e840b38#readProxyContract
2. Click "Read as Proxy" tab
3. Scroll to function `holdsRole`
4. Enter: `roleId` = `0` and `account` = `0x8180d59b7175d4064bdfa8138a58e9babffda44a`
5. Click "Query"
6. Result: **true**

Try a second one to be sure: https://etherscan.io/address/0x487a5bbdc6091be0e27c0e8ea283047d94862538#readProxyContract
Same query. Same result: **true**.

---

## Claim 3: "Hart Lambur (CEO) signs the Safe"

**Verify:**
1. Go to: https://etherscan.io/address/0x8180d59b7175d4064bdfa8138a58e9babffda44a#readProxyContract
2. Click "Read as Proxy"
3. Find and click `getOwners()` → shows 4 signer addresses
4. First signer: `0xcc400c09ecbac3e0033e4587bdfaabb26223e37d`
5. Go to that address: https://etherscan.io/address/0xcc400c09ecbac3e0033e4587bdfaabb26223e37d
6. In the "More Info" section, you'll see: **"Funded By: hal2001.eth"**
7. hal2001.eth is Hart Lambur's public ENS name (CEO of Risk Labs/UMA/Across)

---

## Claim 4: "Kevin Chan is maxodds.eth (the ACX scandal wallet)"

**Verify:**
1. Go to: https://etherscan.io/address/0xd2a78bb82389d30075144d17e782964918999f7f
2. Look at the ENS name associated with this address
3. Check Ogle's public investigation: https://cryptonews.com/news/across-protocol-token-crashes-10-today-amid-23m-team-misappropriation-allegations/
4. The article names "maxodds.eth" as Kevin Chan's voting wallet in the Across Protocol $23M scandal
5. Same address = same person = now the #1 UMA voter

---

## Claim 5: "BornTooLate was never banned"

**Verify:**
1. Go to: https://etherscan.io/address/0xfc204fcfd2a579157898a212ea25ac98de2b1e1c
2. Click "Token Transfers (ERC-20)" tab
3. Filter by UMA token if possible, or search for recent transfers
4. Most recent UMA transfer: Feb 18, 2026 (375,000 UMA sent out)
5. Wallet is still active. Never restricted. Never blacklisted.

Cross-check with Polymarket's March 2025 announcement calling it "unprecedented":
- The Block: https://www.theblock.co/post/348171/polymarket-says-governance-attack-by-uma-whale-to-hijack-a-bets-resolution-is-unprecedented

---

## Claim 6: "BornTooLate was funded through a single-use intermediary"

**Verify:**
1. Go to: https://etherscan.io/address/0xceccfb4a15ad7afbb957bf3af72939db0ccd6ac5
2. Look at total transaction count: **2**
3. Tx 1: Received 1 ETH from 0xedc7001e
4. Tx 2: Sent 0.9956 ETH to BornTooLate (32 seconds later)
5. This wallet has done nothing else in 5 years. Classic single-use intermediary.

---

## Claim 7: "The Safe holds $14.9M in assets"

**Verify:**
1. Go to: https://etherscan.io/address/0x8180d59b7175d4064bdfa8138a58e9babffda44a
2. Check "Token Holdings" — you'll see millions in UMA + ACX tokens
3. The exact dollar value fluctuates but is consistently in the $10-20M range

---

## Claim 8: "Hart Lambur operates oracles on Polygon"

**Verify:**
1. Go to: https://polygonscan.com/address/0xcc400c09ecbac3e0033e4587bdfaabb26223e37d
2. Check transactions — you'll see `proposePrice()` calls
3. These are proposals to UMA's OptimisticOracleV2 on Polygon
4. The CEO personally proposes prices while his Safe controls the voters who verify them

---

## Claim 9: "They all vote in the same second"

**Verify (requires subgraph query — slightly more technical):**

Go to the UMA V2 Subgraph playground:
https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn

Paste this query:
```graphql
{
  revealedVotes(
    where: {
      voter_in: [
        "0x0afc9a935960948ed35e50ac7639cbb38e840b38",
        "0x487a5bbdc6091be0e27c0e8ea283047d94862538"
      ]
      roundId: "10280"
    }
  ) {
    voter { id }
    time
    price
  }
}
```

You'll see both wallets have the EXACT same `time` value = same Ethereum block. Same `price` = same vote.

---

## Data Source for Voting Stats

**UMA Voting V2 Subgraph (free, public, no API key needed):**
https://api.goldsky.com/api/public/project_clus2fndawbcc01w31192938i/subgraphs/mainnet-voting-v2/0.1.1/gn

Query any voter's stats:
```graphql
{
  voter(id: "0xd2a78bb82389d30075144d17e782964918999f7f") {
    id
    stake
    countReveals
    countCorrectVotes
    countWrongVotes
    withdrawnRewards
  }
}
```

---

## News Sources for Disputed Markets

| Market | Source | URL |
|--------|--------|-----|
| Israel-Hezbollah ceasefire | NPR | https://www.npr.org/2026/04/10/nx-s1-5780569/ |
| Ceasefire rules | US State Dept | state.gov "Cessation Between Israel and Lebanon" |
| Hezbollah rejection | Xinhua | english.news.cn |
| Ukraine mineral deal | The Defiant | https://thedefiant.io/news/defi/polymarket-s-usd7m-ukraine-mineral-deal-debacle-traced-to-oracle-whale |
| UFO declassification | CryptoSlate | https://cryptoslate.com/polymarket-faces-major-credibility-crisis-after-whales-forced-a-yes-ufo-vote-without-evidence/ |
| Zelenskyy suit | CoinDesk | https://www.coindesk.com/markets/2025/07/07/polymarket-embroiled-in-usd160m-controversy-over-whether-zelensky-wore-a-suit-at-nato |
| Across $23M scandal | CryptoNews | https://cryptonews.com/news/across-protocol-token-crashes-10-today-amid-23m-team-misappropriation-allegations/ |
| Across scandal | CoinTelegraph | https://cointelegraph.com/news/across-protocol-founders-accused-of-funneling-23m-to-own-company |
| UMIP-189 whitelist | The Block | https://www.theblock.co/post/366507/polymarket-uma-oracle-update |
| Iran ceasefire insider | Bloomberg | bloomberg.com/news/articles/2026-04-08/ |
| Regulatory pressure | NPR | https://www.npr.org/2026/04/10/nx-s1-5780569/ |
| Columbia/Haifa study (Mitts & Ofir) | Harvard Law Forum | https://corpgov.law.harvard.edu/2026/03/25/from-iran-to-taylor-swift-informed-trading-in-prediction-markets/ |

---

*Every claim above is publicly verifiable. If any step doesn't match what we described, we want to know.*
