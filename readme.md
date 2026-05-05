# Citrea State-Diff Footprint Estimate

Assuming “per block” means **per Bitcoin block**, the expected state-diff data is not a fixed number. It scales with the number and type of Citrea transactions included in the batch.

Citrea’s own fee docs give us a way to estimate it. They say every transaction that changes state eventually has those state changes posted to Bitcoin as part of a batch proof, and they expose an `l1DiffSize` field showing the actual state-diff size charged after execution.

Their own table gives rough state-diff sizes:

| Transaction type | Approximate state diff |
|---|---:|
| cBTC transfer to existing recipient | 17 bytes |
| cBTC transfer to new recipient | 33 bytes |
| ERC-20 transfer | 20 bytes |
| Token approval | 30 bytes |
| DEX swap | 70 bytes |
| Lending / borrowing DeFi interaction | 120 bytes |

These figures are based on typical Citrea testnet transaction patterns, not guaranteed mainnet averages.

Source: [Citrea Fee Model](https://docs.citrea.xyz/advanced/fee-model)

## Estimate at 1 Million Citrea Transactions Per Day

At **1 million Citrea transactions per day**, spread evenly across roughly **144 Bitcoin blocks per day**, you get about **6,944 Citrea transactions per Bitcoin block**.

Approximate compressed state-diff data per Bitcoin block:

| Transaction mix | State diff per tx | Per Bitcoin block at 1M tx/day | Per day |
|---|---:|---:|---:|
| Simple cBTC transfers, existing recipients | 17 bytes | ~118 KB | ~17 MB |
| cBTC transfers, new recipients | 33 bytes | ~229 KB | ~33 MB |
| ERC-20 transfers | 20 bytes | ~139 KB | ~20 MB |
| Token approvals | 30 bytes | ~208 KB | ~30 MB |
| DEX swaps | 70 bytes | ~486 KB | ~70 MB |
| DeFi lending / borrowing interactions | 120 bytes | ~833 KB | ~120 MB |

That means **1 million transactions per day is not automatically catastrophic** if most activity is simple transfers. But if the activity is DeFi-heavy, it starts becoming real blockspace consumption quickly.

## Estimate at 10 Million Citrea Transactions Per Day

At **10 million Citrea transactions per day**, multiply those numbers by 10.

| Transaction mix | Per Bitcoin block at 10M tx/day | Per day |
|---|---:|---:|
| Simple cBTC transfers | ~1.18 MB | ~170 MB |
| ERC-20 transfers | ~1.39 MB | ~200 MB |
| DEX swaps | ~4.86 MB | ~700 MB |
| DeFi lending / borrowing | ~8.33 MB | ~1.2 GB |

That is where the design starts looking ugly. A wildly successful Citrea doing millions of DeFi-style transactions per day would require very large recurring Bitcoin data postings.

Citrea’s docs say batch proofs include state differences and that proof size depends on the number and type of Citrea transactions. They also say Citrea chunks proofs to keep each proof transaction under **400 KB**, and that multiple batch proofs can be posted for any Bitcoin block.

Source: [Citrea Architecture and Transaction Lifecycle](https://docs.citrea.xyz/essentials/architecture-and-transaction-lifecycle)

A separate Citrea blog post adds another important limit: after compression, if a state diff is larger than **4 MB**, their prover splits the proof around sequencer commitments, rather than trying to fit the whole thing into one Bitcoin block.

Source: [Citrea Prover: Designed for Bitcoin](https://www.blog.citrea.xyz/citrea-prover-designed-for-bitcoin/)

## Bottom Line

At **1M tx/day**, expected state-diff data might plausibly range from **~100 KB to ~800 KB per Bitcoin block**, depending on transaction mix.

At **10M tx/day**, it could range from **~1 MB to over 8 MB per Bitcoin block**.

The crucial caveat is that Citrea’s own numbers are “indicative,” and the real size depends on how often transactions touch the same accounts or storage slots. Repeated changes to the same storage slots can collapse into smaller state diffs. Lots of unique users, contracts, storage writes, deployments, swaps, and DeFi interactions make the diff much larger.