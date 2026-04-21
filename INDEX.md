# UMA Investigation — File Index

## Core Evidence

| File | Purpose | Status |
|------|---------|--------|
| `evidence-db.json` | Structured database of ALL findings with confidence levels | COMPLETE |
| `FINAL-NARRATIVE.md` | Full investigation document (conservative claims only, with inline verify links) | REVISED v2 |
| `investigation.html` | Shareable HTML page with all evidence | COMPLETE |
| `VERIFY-GUIDE.md` | Step-by-step verification guide for non-technical readers (URLs for every claim) | COMPLETE |
| `APPENDIX-queries.md` | ALL reproducible queries (curl commands, GraphQL, Etherscan links) | COMPLETE |

## Publication-Ready

| File | Purpose | Status |
|------|---------|--------|
| `X-ARTICLE.md` | **Full X/Twitter article** — all 10 findings + all reproducible queries inline | COMPLETE |
| `TWEETS-CORRECTED.md` | 16-tweet thread (v2.0, all retracted claims removed) | COMPLETE |
| `POST-telegram-corrected.md` | 4 Telegram posts (corrected) | COMPLETE |
| `DISCORD-verify-corrected.md` | 7 Discord messages for community verification | COMPLETE |

## Old/Deprecated (DO NOT USE — contains retracted claims)

| File | Issue |
|------|-------|
| `TWEET-all-evidence.md` | Contains "shared aggregator" claim (RETRACTED) |
| `DISCORD-verify-finding1.md` | Based on "shared aggregator" (RETRACTED) |
| `POST-telegram.md` | Missing corrected data |
| `finding-1-shared-aggregator.html` | ENTIRE PREMISE RETRACTED |
| `ALL-POSTS.html` | Contains retracted claims |

## Key Findings Summary

### PROVEN (on-chain verified)
1. 86 of 89 DesignatedVotingV2 contracts owned by same Safe (6.76M UMA, 38.3%)
2. Hart Lambur (CEO) signs the Safe (hal2001.eth → 0xcc400c09)
3. All 5 top DV2 contracts vote in same second, 100% of rounds
4. Kevin Chan votes in same second as DV2 bloc, 100% of rounds
5. Total Risk Labs control: 11.37M UMA = 64.4% of stake
6. BornTooLate: 1.35M UMA sent to intermediary, never banned, still active
7. Single-use funding intermediary (2 txns total, March 2021)
8. Rewards: $665K/year, $1.44M total extracted
9. **Kevin Chan (maxodds.eth) = SAME wallet from $23M ACX scandal = #1 UMA voter**
10. Hart Lambur: 5 proposePrice() calls on Polygon OptimisticOracleV2 ($200K bonds)

### VERIFIED (subgraph/multiple sources)
1. 98.52% vote agreement within DV2 bloc (2,233 rounds)
2. 99.81% agreement Kevin vs DV2 (1,045 rounds, 2 disagreements)
3. Compounding to 54.9% within 5 years
4. $438M in disputed markets (media-documented)

### RETRACTED
1. "Shared aggregator" = actually Coinbase 10
2. "Safe labeled Risk Labs Treasury" = actually "Smart Account by Safe"
3. "10 contracts" = actually 89 (undercount, not overcount)

## Analysis Scripts (generated during research)

| File | Purpose |
|------|---------|
| `tmp/uma_vote_compare.py` | Vote reveal comparison across DV2 contracts |
| `tmp/uma_vote_deep.py` | Deep analysis with commit timing |
