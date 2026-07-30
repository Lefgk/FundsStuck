# RAKE Protocol Venus Recovery Proposal


## The Issue

I'm the founder of a BSC project that operated between 2021-2022 as a fork of AutoFarm. AutoFarm had a critical bug in their strategy contract that permanently set token approvals to zero once contracts were paused. This unfortunately caused our contracts to accumulate significant amounts of vBTC, vETH, vBUSD, and other Venus wrapped tokens that became permanently locked within the contract architecture.

The technical root cause is documented in **[Data Appendix Section A](./data_appendix.md#a-technical-root-cause)**, where the `unpause()` function incorrectly maintains zero approvals instead of restoring unlimited approvals, effectively breaking all Venus protocol interactions.

Our affected contracts currently hold (live on-chain data, verified 24 July 2026) **~$2.11 million in supplied assets** with **~$1.14 million borrowed**, resulting in **~$974 thousand net stuck value** across **seven** Venus markets — vBUSD, vBTC, vETH, vLINK, vUSDT, vUSDC and vDOT (detailed breakdown in **[Data Appendix Section B](./data_appendix.md#b-affected-contract-details)**). *USD figures track spot prices; the fixed on-chain token quantities are the authoritative measure and are fully verifiable via the commands in the appendix.*

> **Note on scope — our eighth Venus strategy is deliberately excluded.** We also operate a vBNB strategy carrying the same code, but the bug zeroes an **ERC-20 allowance**, and the vBNB market accepts **native BNB** via a `payable mint()` and has no `underlying()` ERC-20 at all — so no `transferFrom` and no allowance is ever consulted there. The approval bug therefore cannot be what stopped that strategy, and we are not asking Venus to act on it. We have removed it from every figure in this request. **Only the 7 ERC-20 strategies are frozen by this bug.**
>
> To be straight about it: that strategy is *also* stuck, by an unrelated fault of our own — over five years its borrow interest outran its supply interest until the position passed its own leverage ceiling, and the unwind path now reverts on a SafeMath underflow. We are not presenting it as recovered or as trivially recoverable. We exclude it because it is a different problem and **not one Venus needs to solve.**

> **⚠️ The stuck funds are now being actively eroded.** Venus's own market-deprecation process (VIP-634/635 and the multi-chain wind-down) set the vDOT market's collateral factor and liquidation threshold to zero, which made our vDOT strategy liquidatable through no fault of ours. Between 12–22 July 2026 it was liquidated in **21 transactions**, and vLINK is one governance step away from the same fate. Because the approval bug means our strategies **cannot repay, deleverage, or exit**, they cannot defend themselves — the deprecation engine (punitive ~300% borrow APR, 100% reserve factor) transfers our equity to Venus reserves and liquidators over time. This makes a cooperative recovery urgent, not merely desirable. Full record in **[Data Appendix Section F](./data_appendix.md#f-deprecation-and-liquidation-record)**.

We are requesting assistance to recover roughly **$974 thousand** in Venus protocol tokens that are permanently stuck in our smart contracts due to an inherited approval bug from AutoFarm's strategy implementation. As the verified owner of both the deployer wallet and affected contracts, we seek Venus protocol team collaboration to unlock these assets and resume normal operations.

## Why Now?

My development partner Lef was working with me on a new project that involves Venus contracts and recognized that a contract owned by me had a significant amount of Venus tokens. This initiated our collaboration to develop a solution for recovering these tokens, as I still maintain full control and ownership of both the deployer wallet and the affected contracts.

Our exploration of various recovery methods is detailed in the technical appendix (see **[Data Appendix Section C](./data_appendix.md#c-info-on-attempts-to-recover-funds)**), where we systematically tested multiple contract functions to identify the exact failure point.


## Historical Context

In 2021, when this issue first occurred, it coincided with Venus's depegging event, which was ultimately resolved through Binance's intervention. I had reached out at the time, but due to the depegging crisis and the overwhelming volume of support tickets, the Venus team was unable to provide assistance with my specific technical issue, resulting in minimal progress toward resolution.

## User Refund and Impact Mitigation

The bug affected our platform very shortly after launch. Fortunately, I had other functioning products, so the stuck Venus vault assets represented approximately 4% of our total value locked (TVL). Since these funds were unable to generate yield for users or the platform, I took immediate action to protect our community.

The affected pools kept paying RAKE emissions to their depositors after the strategies were bricked, exactly as if the deposits were still working, and their allocation points were never reduced — not then, not since. In the first seven days the seven frozen pools paid their depositors **165.188 RAKE**, and **486.975 RAKE** by 25 March 2021.

What that was worth is measured from the swap receipts, not estimated: for each day, the BNB those wallets were actually paid divided by the RAKE they actually sold, applied to that day's emissions. RAKE fell from $43,300 to $1,300 in ten days, so a single blended rate would distort it either way; each day is valued at its own realized rate instead.

- By **day five — 2 March 2021** — they had received **4,467.2 BNB**, worth **$1,020,056** at the BNB price of the time. That is **1.05x** the $972,437 of principal still frozen, and the point at which the emissions passed it.
- By **day seven** it was **5,050.1 BNB — $1,158,700, or 1.19x**. By 25 March, **5,884.1 BNB — $1,354,993, or 1.39x**.
- At **day-of VWAP** the first week comes to **$1,278,224 — 1.31x**.

Both sides of that comparison are in Feb–Mar 2021 dollars, so it mixes no price epochs. **96 of the 103 affected wallets sold**, and every fill is an on-chain swap with a matching transfer — this was real money, not paper.

Two things I want to state plainly rather than have someone else point out. It is an **aggregate**: coverage was not universal, and seven of the 103 wallets never sold their RAKE at all. And it was **not a discretionary act** — allocation points were never changed and no airdrop was made. The pools continued paying their pre-existing schedule, which is why the coverage accrues over days rather than arriving in one transaction. What I claim is that the compensation was substantial, that it was real rather than promised, and that it exceeded the frozen principal within the first week.

Liquidity was not the constraint. Over $4.7M was swapped for BNB on day one (the WBNB side of the trades; see Data Appendix Table 2), so any depositor who wanted to sell and leave could do so (see **[Data Appendix Table 2](./data_appendix.md#table-2-complete-daily-trading-and-liquidity-data)**).

What I can show without qualification is that I did not sell into that. Of the **9,702.43 RAKE** minted to the dev-fee wallet, it still holds **8,933.66 RAKE — 92.08%** of it, and its only market interactions in the window were liquidity *adds* and a burn. The dev fee was accrued and never sold, and that is verifiable in a single call.

After the compensation was paid out, I shifted focus to other development priorities, as initial attempts to engage Venus support yielded limited progress.

## Current Development and Recovery Plans

For the past 18 months, I have self-funded development of a unique trading and strategy management platform that will launch on BSC, bringing high-demand tools to BSC users that are currently unavailable on any EVM chain. This development specifically includes integration with Venus pools and lending protocols.

The recovered funds would be allocated toward:

- Complete platform audits, including smart contracts, economic, risk analysis and more
- Marketing and user acquisition
- Continued platform development and feature expansion

To make this worthwhile for Venus directly, I am offering a recovery fee paid to the Venus treasury — a percentage of the ~$974k recovered — and I will cover all audit and engineering costs associated with the recovery. Beyond that, the platform launch is expected to drive additional volume and users for Venus, but the treasury fee and covered costs stand on their own regardless.

## Proposed Recovery Solution

### Key Finding: External Debt Repayment Works

We verified on a BSC fork that `repayBorrowBehalf()` successfully repays strategy debt from an external address. This works **despite the approval bug**, because it draws on the *repayer's* own allowance, not the strategy's broken one. Repaying the debt is the essential first step of the recovery: once debt is zero, moving the collateral out of the strategy creates **zero bad debt** for Venus. On its own, however, `repayBorrowBehalf()` recovers **$0** — it only clears the liability; the ~$2.11M of supplied collateral remains stranded behind the same broken approval. A second step is therefore required to move the collateral.

### Recommended Recovery Path — Governance vToken Implementation Swap (Full Recovery)

This is the **recommended** approach. It recovers the full net equity (~$974k) with **no liquidation penalty, no bad debt, and no stranded equity**, in three steps:

1. **Owner repays all strategy debt** via `repayBorrowBehalf()` (~$1.14M; verified working despite the bug). With debt = 0, moving the collateral creates **zero bad debt** for Venus.
2. **The VIP.** Venus governance temporarily points each affected vToken delegator (proxy) at a patched implementation that adds a single admin-only function `rescueTransfer(strategy, to)`, which moves the strategy's vTokens to the RAKE governance address `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, then restores the original implementation. All of this happens **atomically within one VIP transaction**, so no external actor can interleave between the swap and the restore.
3. **Owner redeems** the ~$2.11M of underlying normally.

**Result:** full ~$974k net equity recovered — no liquidation penalty, no bad debt, no stranded equity.

**Feasibility verified on-chain:** the 7 ERC-20 vTokens are delegator proxies whose admin is the Venus Normal Timelock `0x939bD8d64c0A9583A7Dcea9933f7b21697ab6396`; they share the implementation `0xCDfea50f7CECCB24Fe804657DB8E6c93b689941e`, and the `accountTokens` mapping lives at storage slot 14. Our vBNB strategy is a native market with a separate admin (`0x9A7890534d9d91d473F28cB97962d176e2B65f1d`), is not affected by the approval bug, and is **not part of this request** — no Venus action is needed on it.

### Proof of Concept

We have reproduced this recovery end-to-end against a BSC mainnet fork (block `111859565`), impersonating the Venus Normal Timelock. The proof-of-concept confirms **full recovery — `redeemed == supplied` for all 7 ERC-20 markets** — after clearing debt with `repayBorrowBehalf()` and executing the atomic implementation-swap. The complete Foundry test suite and a runnable `forge` script are available to the Venus team on request.

### Venus Admin Functions Analysis

| Function | Can Help? | Notes |
|----------|-----------|-------|
| `repayBorrowBehalf()` | **YES** | Clears debt despite the bug (step 1 of the recovery) |
| vToken implementation swap | **YES** | Moves the collateral out of the strategy — full recovery |
| `sweepToken()` | No | Cannot sweep underlying token |
| `reduceReserves()` | No | Only for protocol reserves |

### Venus Governance Precedent

Venus governance has repeatedly authorized targeted, position-specific actions and routinely upgrades vToken delegate implementations — the delegator pattern is built for exactly this. To be candid: reassigning a live vToken balance from one account to another **is** a novel action for Venus, and we do not want to understate that. It is acceptable here because (1) all debt is repaid first, so there is **zero bad debt**; (2) the recipient is the proven on-chain owner (`govAddress()`); (3) the scope is hard-locked to the specific strategy → owner pairs; (4) the mechanism is independently audited; and (5) a recovery fee is paid to Venus.

Concretely, the swap uses a dedicated, **independently-audited `RescueDelegate`** that is **timelock-only** and **hard-restricted** to the specific (strategy → owner) pairs. It moves only the two account balances involved — conserving `totalSupply` and leaving all other market accounting **byte-identical** — and the storage layout is proven per-market before any VIP is filed. The audit is funded by the requester.

| VIP | Description |
|-----|-------------|
| **VIP-79** | Whitelisted BNB Chain as the **sole** liquidator of a specific exploiter account (Nov 2022); wallet pre-funded with $30M. Anchor precedent — governance authorized a single designated party to act on one position. [ref](https://www.coindesk.com/tech/2023/08/21/bnb-chain-exploiter-liquidated-for-30m-on-venus-protocol) |
| **VIP-161** | BUSD market deprecation plan — governance-directed market intervention. [ref](https://community.venus.io/t/risk-analysis-of-busd-and-potential-deprecation-plan/3353) |
| **Delegate upgrades** | Venus routinely upgrades vToken delegate implementations via VIP — the delegator pattern is the same mechanism the audited, timelock-only `RescueDelegate` in this recovery uses. [ref](https://docs-v4.venus.io/governance/decentralization) |

### Deprecation Urgency

Venus's own market-deprecation process has already **liquidated the vDOT strategy 21 times** (12–22 July 2026), and **vLINK is next** (collateral factor 0, liquidation threshold 0.63 vs. our 0.618 ratio — only a **~$488 buffer**). The deprecated markets charge a punitive **~300% borrow APR with a 100% reserve factor**, and because the approval bug prevents the strategy from repaying or exiting, this steadily bleeds our equity into Venus reserves. This makes a cooperative recovery **urgent, not merely desirable**. Full record in **[Data Appendix Section F](./data_appendix.md#f-deprecation-and-liquidation-record)**.

## On-Chain Verification

Anyone can verify the stuck funds using these commands:

```bash
# vBTC Strategy - Check supply
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "balanceOfUnderlying(address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org

# vBTC Strategy - Check borrow
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "borrowBalanceCurrent(address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org

# Check approval (returns 0 - the bug)
cast call 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c \
  "allowance(address,address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  --rpc-url https://bsc-dataseed.binance.org
```

Full verification commands for all 7 affected strategies available in **[Data Appendix](./data_appendix.md)**.

## Verification and Next Steps

I can provide complete proof of ownership and control over the contracts containing the stuck tokens. Crucially, this control is **verifiable on-chain right now** — every one of the 7 affected strategy contracts returns `govAddress()` = `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, proving I control the strategies themselves rather than merely having deployed them. Anyone, including Venus risk reviewers, can confirm this without trusting me:

```bash
cast call 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 "govAddress()(address)" --rpc-url https://bsc-dataseed.binance.org
```

On request I will also provide a signed message and an on-chain transaction from that same address. The main contract addresses are in **[Data Appendix Section B](./data_appendix.md#b-affected-contract-details)**, and technical architecture is documented in **[Data Appendix Section D](./data_appendix.md#d-strategy-contract-architecture)**.

We have conducted comprehensive testing in forked environments to confirm the exact nature of the approval bug and its impact on all lending-related functions (**[Data Appendix Section C](./data_appendix.md#c-info-on-attempts-to-recover-funds)**).

I would like to move forward with the recovery process as soon as possible and am available to provide any additional documentation, proof of ownership, or technical details required by the Venus protocol team.
For further contact :

- Discord -> 5ubse7en
- Email -> f0r3x_shark@protonmail.com

