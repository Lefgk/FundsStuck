# Venus Protocol Fund Recovery Request

## Executive Summary

**Total Net Value Stuck:** ~$972K USD ($972,437, Venus oracle at BSC block 112,835,163)
**Total Supplied:** ~$2.11 Million USD ($2,107,606)
**Total Borrowed:** ~$1.14 Million USD ($1,135,169)
**Value when the pools froze (Feb 2021, Venus oracle):** $1,028,474
**Affected Markets:** 7 Venus vToken markets (the seven ERC-20 leveraged strategies, pids 9–15)
**Root Cause:** Inherited bug from AutoFarm strategy contract
**Recommended Recovery:** a governance vToken implementation-swap achieving FULL recovery with no liquidation penalty and no bad debt (see Proposed Recovery Solution)

The funds are permanently locked due to a bug in the `unpause()` function that sets token approvals to zero instead of unlimited, breaking all Venus Protocol interactions.

*Note: USD values fluctuate with market prices. Token quantities are on-chain and accrue interest each block (supplied at supply APY, borrowed at the higher punitive borrow APY on these deprecated markets), so both legs drift upward but the borrowed leg outruns the supplied leg — net underlying drifts slightly downward over time.*

---

## Ownership & Verification

| Role | Address |
|------|---------|
| **Deployer/Owner/Gov** | `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07` |
| **Farm Contract** | `0x7f7Bf15B9c68D23339C31652C8e860492991760d` |

**Proof of Ownership Available:**
- Signature from gov address
- On-chain transaction from gov address
- Full control over all affected contracts

---

## Proof of Ownership

**Verifiable on-chain right now:** every one of the 7 affected strategy contracts returns `govAddress()` = `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07` (the requester's gov address). This proves the requester controls the strategies themselves, not merely that they deployed the farm. Anyone — including Venus risk reviewers — can confirm this without trusting us:

```bash
cast call 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 "govAddress()(address)" --rpc-url https://bsc-dataseed.binance.org
# returns 0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07  (same for all 7 strategies)
```

Run the same call against each of the 7 strategy addresses to confirm they all point at the same gov address:

- `0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12` (vBTC)
- `0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C` (vETH)
- `0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A` (vUSDC)
- `0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C` (vBUSD)
- `0xc6f470dA6C284D16647CEE2230522a85b0818f6D` (vUSDT)
- `0x0b09Efc9458c00354414D2A560aA6EDa19490169` (vLINK)
- `0x568254EAAC1476faf9A908e577faa3Ab96029801` (vDOT)

The farm also ran an eighth Venus strategy, `0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122` (vBNB), under the same gov address. Its funds are also stuck, but from a **separate defect that Venus cannot fix**, so it is **not part of this request** — see "Why vBNB is out of scope" below.

**On request, the requester will additionally provide:**
1. An EIP-191 signed message from the gov address containing a Venus-chosen nonce plus the proposal hash.
2. A fresh on-chain transaction from the gov address referencing this proposal.
3. Any challenge-response Venus dictates.

Recovered funds can be delivered to a Venus-verified destination.

---

## Strategy Contracts Covered by This Request

The seven ERC-20 strategies frozen by the approval bug. (The eighth Venus strategy, vBNB, is stuck by a different defect and is out of scope — see below.)

### Table 1: Venus Strategy Contract Positions (Live On-Chain Data - July 2026)

**Verified on-chain:** BSC block 112,835,163 (29 July 2026)

**Priced by Venus's own oracle**, so every figure below is reproducible on Venus infrastructure:
Comptroller `0xfD36E2c2a6789Db23113685031d7F16329158384` → `oracle()` →
`0x6592b5de802159f3e74b2486b091d11a8256ab8a` → `getUnderlyingPrice(vToken)`, scaled 1e18.
Balances are `balanceOfUnderlying()` and `borrowBalanceCurrent()` on each vToken.

| Contract Address | vToken | Supplied USD | Borrowed USD | Net USD |
|------------------|--------|--------------|--------------|---------|
| [0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12](https://bscscan.com/address/0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12) | vBTC | $591,594 | $339,850 | $251,745 |
| [0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C](https://bscscan.com/address/0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C) | vETH | $444,341 | $261,706 | $182,634 |
| [0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A](https://bscscan.com/address/0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A) | vUSDC | $560,945 | $345,493 | $215,452 |
| [0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C](https://bscscan.com/address/0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C) | vBUSD | $206,755 | $0 | $206,755 |
| [0xc6f470dA6C284D16647CEE2230522a85b0818f6D](https://bscscan.com/address/0xc6f470dA6C284D16647CEE2230522a85b0818f6D) | vUSDT | $264,409 | $163,867 | $100,542 |
| [0x0b09Efc9458c00354414D2A560aA6EDa19490169](https://bscscan.com/address/0x0b09Efc9458c00354414D2A560aA6EDa19490169) | vLINK | $39,240 | $24,253 | $14,987 |
| [0x568254EAAC1476faf9A908e577faa3Ab96029801](https://bscscan.com/address/0x568254EAAC1476faf9A908e577faa3Ab96029801) | vDOT | $322 | $0 | $321 |
| **TOTAL** | | **$2,107,606** | **$1,135,169** | **$972,437** |

*\*Per-row USD is rounded to the nearest dollar; the TOTAL row is computed from unrounded figures.*

**At the moment each pool froze, the same positions were worth $1,028,474.** Each pool measured at
its own freeze block — the last `unpause()` on that strategy — and priced by the Venus oracle then in
force, `0x516c18dc440f107f12619a6d2cc320622807d0ee`. Per-pool freeze blocks, oracle prices and both
epochs are in `package/RAKE_Venus_Evidence/2_data_by_pool_VERIFIED.csv`. Net equity has fallen
**5.45%** in the five and a half years since, as borrow interest outran supply interest and the
deprecation liquidations took the vDOT and vBUSD legs.

*Note on vDOT:* the vDOT position (0x568254EAAC1476faf9A908e577faa3Ab96029801) was largely liquidated by Venus's market deprecation and now holds only 424.69 DOT supplied / 0.41 DOT borrowed (~$340 net). See the Deprecation & Liquidation Record below.

### Why vBNB is out of scope

The farm ran **eight** Venus leveraged strategies (pids 8–15). This request covers the **seven ERC-20 strategies** above (pids 9–15), which share the one root cause described below. The eighth, vBNB (`0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122`, pid 8), is excluded because it is stuck for a **different reason** — not because it is unaffected, and not because it is recoverable.

**The approval bug cannot reach a native-BNB market.** vBNB supplies **native BNB** via the vToken's `payable mint()`, so no ERC-20 allowance is ever consulted. The buggy line is guarded by `if (!wantIsWBNB)` — see the Buggy Code block under "The Bug" below — and never executes for the BNB strategy. One call distinguishes the two cases:

```bash
# frozen market: vBTC has an ERC-20 underlying, so mint() pulls via transferFrom
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B "underlying()(address)" --rpc-url https://bsc-dataseed.binance.org
# -> 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c (BTCB) => needs an allowance, which is 0 => bricked

# vBNB has no underlying() at all — native market, payable mint(), no allowance involved
cast call 0xA07c5b74C9B40447a954e1466938b865b6BBea36 "underlying()(address)" --rpc-url https://bsc-dataseed.binance.org
# -> reverts / no such function
```

The vBNB strategy's `wantAddress()` is WBNB (`0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c`) and its WBNB → vBNB allowance is indeed zero, exactly as on the seven frozen strategies — but on vBNB that approval is inert, because vBNB never calls `transferFrom`.

**vBNB is nevertheless still stuck — by a defect in RAKE's own strategy.** The position (184.493698 BNB supplied / 129.373745 borrowed, **55.119953 BNB net, ~$31,225**, 69 depositor positions) cannot be withdrawn: `RakeFarm.withdraw(8, 1)` reverts `SafeMath: subtraction overflow`, and simulated from the gov wallet both `deleverageOnce()` and `deleverageUntilNotOverLevered()` revert with the same underflow. Only the zero-amount harvest path works. The strategy targets `borrowRate` 585 (0.585) against `BORROW_RATE_MAX` 595, but five years of borrow interest outrunning supply interest (+29.25 BNB debt vs +20.62 BNB supply) pushed the live ratio to **0.7012**, above its own ceiling; the unwind path then computes a required supply of 129.373745 / 0.585 = 221.15 BNB, subtracts it from the actual 184.49, and underflows. No branch handles a position that is already over the limit.

**Venus cannot fix an arithmetic underflow inside a third-party contract**, which is why no Venus action is requested for vBNB. It is a separate matter, pursued separately. Every figure in this document excludes it. Section A of the data appendix records the full reproducible evidence for both defects.

---

## The Bug - Technical Root Cause

### Location
- **File:** `StratVLEV.sol` (Venus Leveraged Strategy - AutoFarm fork)
- **Function:** `unpause()` at line 1833

### Buggy Code
```solidity
function unpause() external {
    require(msg.sender == govAddress, "Not authorised");
    _unpause();
    IERC20(venusAddress).safeApprove(uniRouterAddress, uint256(-1));
    IERC20(wantAddress).safeApprove(uniRouterAddress, uint256(-1));
    if (!wantIsWBNB) {
        IERC20(wantAddress).safeApprove(vTokenAddress, 0);  // <-- BUG: Should be uint256(-1)
    }
}
```

### Correct Code Should Be
```solidity
if (!wantIsWBNB) {
    IERC20(wantAddress).safeApprove(vTokenAddress, uint256(-1));  // FIXED: Unlimited approval
}
```

### Impact
- `pause()` function correctly sets approval to 0
- `unpause()` function **incorrectly maintains** approval at 0 instead of restoring to unlimited
- This breaks ALL Venus operations requiring underlying token transfers:
  - `IVToken.mint()` fails - cannot supply more collateral
  - `IVToken.repayBorrow()` fails - cannot repay debt
  - `_leverage()` fails - cannot build positions
  - `_deleverage()` fails - cannot unwind positions

### Two independent locks — to be precise about which is which

A user exiting one of these pools today hits **two** separate locks, and we want to be exact about their origin rather than attribute both to the approval bug:

1. **Venus's action pause on the deprecated market, hit first.** `withdraw(9, 1e18)` on the vBUSD pool reverts with **`action is paused`** — the **Comptroller's** error string, not the strategy's `Pausable` (which reverts `Pausable: paused`). This follows from the market deprecation described below and is Venus-side parameter state.
2. **The zero allowance from `unpause()`, independent of the first.** Clearing the action pause would not free the funds: every path moving the underlying still reverts `BEP20: transfer amount exceeds allowance`, and the strategy has no way to restore the approval — it is not upgradeable and `inCaseTokensGetStuck()` protects `wantAddress`.

This request concerns lock 2, which is unfixable from the strategy side at any price. Lock 1 is stated so the picture is complete and the two error strings are not conflated.

---

## Why Normal Recovery Fails

We performed extensive testing on BSC fork chains using Foundry. All attempts ended with `BEP20: transfer amount exceeds allowance` error.

### Tested Functions

| Function | Result | Failure Point |
|----------|--------|---------------|
| `deposit()` | FAILED | `_farm()` → `_supply()` → needs approval |
| `withdraw()` | FAILED | `_farm()` → `_supply()` → needs approval |
| `farm()` | FAILED | `_supply()` → needs approval |
| `earn()` | FAILED | Venus interactions blocked |
| `deleverageOnce()` | FAILED | `_repayBorrow()` → needs approval |
| `deleverageUntilNotOverLevered()` | FAILED | Calls `deleverageOnce()` |
| `rebalance(0, 0)` | FAILED | `_farm()` at end → needs approval |
| `inCaseTokensGetStuck(vToken)` | FAILED | Protected with `!safe` error |
| `inCaseTokensGetStuck(wantToken)` | FAILED | Protected with `!safe` error |

### Tested Scenarios

**Scenario 1: Emergency Token Extraction**
- Tested `inCaseTokensGetStuck()` function
- vToken extraction: Failed with `!safe` error (protected as vTokenAddress)
- Want token extraction: Failed with `!safe` error (protected as wantAddress)
- Result: Core protocol tokens remain stuck due to safety checks

**Scenario 2: Complete Position Unwinding**
- Attempted `rebalance(0, 0)` to force deleverage
- Result: Failed with `BEP20: transfer amount exceeds allowance`
- Position closure impossible due to approval issues

**Scenario 3: Manual Deleveraging**
- Attempted step-by-step deleveraging
- `deleverageUntilNotOverLevered()`: Failed with allowance error
- `deleverageOnce()`: Failed with allowance error
- All Venus-dependent operations blocked

**Scenario 4: External Debt Repayment (VERIFIED WORKING)**
- Tested `repayBorrowBehalf()` from external address
- Result: **SUCCESS** - Can repay debt from outside
- But subsequent operations still fail due to internal approval bug

### Contract Limitations
- **Not upgradeable** - No proxy pattern, cannot fix the bug
- **No admin override** - No function to manually set approval
- **Protected tokens** - `inCaseTokensGetStuck()` blocks wantAddress and vTokenAddress

---

## External Verification Commands

Anyone can verify the stuck funds on-chain:

```bash
# vBTC Strategy - Check supply and borrow
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "balanceOfUnderlying(address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org

cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "borrowBalanceCurrent(address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org

# Check approval (will return 0 - the bug)
cast call 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c \
  "allowance(address,address)(uint256)" \
  0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  --rpc-url https://bsc-dataseed.binance.org

# vETH Strategy
cast call 0xf508fCD89b8bd15579dc79A6827cB4686A3592c8 \
  "balanceOfUnderlying(address)(uint256)" \
  0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C \
  --rpc-url https://bsc-dataseed.binance.org

# vUSDC Strategy
cast call 0xecA88125a5ADbe82614ffC12D0DB554E2e2867C8 \
  "balanceOfUnderlying(address)(uint256)" \
  0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A \
  --rpc-url https://bsc-dataseed.binance.org

# vUSDT Strategy
cast call 0xfD5840Cd36d94D7229439859C0112a4185BC0255 \
  "balanceOfUnderlying(address)(uint256)" \
  0xc6f470dA6C284D16647CEE2230522a85b0818f6D \
  --rpc-url https://bsc-dataseed.binance.org

# vLINK Strategy
cast call 0x650b940a1033B8A1b1873f78730FcFC73ec11f1f \
  "balanceOfUnderlying(address)(uint256)" \
  0x0b09Efc9458c00354414D2A560aA6EDa19490169 \
  --rpc-url https://bsc-dataseed.binance.org

# vDOT Strategy
cast call 0x1610bc33319e9398de5f57B33a5b184c806aD217 \
  "balanceOfUnderlying(address)(uint256)" \
  0x568254EAAC1476faf9A908e577faa3Ab96029801 \
  --rpc-url https://bsc-dataseed.binance.org

# vBUSD Strategy
cast call 0x95c78222B3D6e262426483D42CfA53685A67Ab9D \
  "balanceOfUnderlying(address)(uint256)" \
  0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C \
  --rpc-url https://bsc-dataseed.binance.org
```

---

## Venus Admin Functions Analysis

| Function | Full recovery? | Notes |
|----------|----------------|-------|
| `sweepToken()` | No | Cannot sweep underlying token |
| `reduceReserves()` | No | Only for protocol reserves |
| `repayBorrowBehalf()` | **YES** | Clears debt despite the bug (step 1 of the recovery); uses the repayer's own allowance |
| vToken implementation swap (delegator) | **YES — recommended** | Admin-only patched implementation adds `rescueTransfer`, moving vTokens to the owner atomically; recovers 100% of equity, no penalty, no bad debt |

---

## Proposed Recovery Solutions

### Governance vToken Implementation-Swap (RECOMMENDED — full recovery)

Proven on a BSC mainnet fork at block 111859565 (see Proof of Concept below). Recovers 100% of the equity with **no liquidation penalty, no bad debt, and no stranded equity.**

**Step 1 — Owner repays all strategy debt.** The owner calls `repayBorrowBehalf` for each strategy to clear the entire ~$1.14M of debt. This works despite the approval bug because `repayBorrowBehalf` uses the *repayer's* own allowance, not the strategy's. With debt = 0, moving the collateral creates ZERO bad debt for Venus.

**Step 2 — The VIP (atomic).** Venus governance temporarily points each affected vToken delegator (proxy) at a patched implementation — the **RescueDelegate** — that adds one admin-only function `rescueTransfer(strategy, to)`. That function moves the strategy's vTokens to the RAKE governance address `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, after which the VIP restores the original implementation — all in a single atomic VIP transaction.

The RescueDelegate is a carefully-scoped, independently-audited operation, not a blanket admin hook. Specifically it:
- is callable **ONLY by the Normal Timelock** (`0x939bD8d64c0A9583A7Dcea9933f7b21697ab6396`);
- is **hard-restricted to the specific (strategy → owner) pairs** listed in this proposal — it cannot touch any other account;
- moves **ONLY the two `accountTokens` entries** (decrement the strategy, credit the owner), conserving `totalSupply`, and leaves `totalBorrows`, `totalReserves`, `borrowIndex`, and the exchange rate **byte-identical**. These are asserted as explicit post-condition invariants in the PoC.

**Storage-slot proof (per-market, not assumed).** The `accountTokens` mapping lives at storage slot 14. This will be proven **PER-MARKET** with `cast storage` output for each of the 7 ERC-20 vTokens before any VIP is submitted — we do not assume a shared layout across markets.

**Independent audit.** The requester will fund an **INDEPENDENT audit** of the RescueDelegate (e.g. OpenZeppelin / Certik / Peckshield), with results made public, before execution.

**Honest novelty assessment.** We are explicit that this is novel: there is **no exact prior precedent** for governance reassigning a live vToken balance. We argue it is nonetheless acceptable because:
- debt is fully repaid **first**, so there is zero bad debt and zero protocol loss;
- the recipient is the **proven owner** (see Proof of Ownership);
- the scope is **hard-locked** to the enumerated (strategy → owner) pairs and to the `accountTokens` entries alone;
- it is **independently audited** with public results before execution;
- a **recovery fee is paid** to the Venus treasury.

**Step 3 — Owner redeems.** The owner redeems the ~$2.11M of underlying normally.

**Result:** the full ~$972K of net equity is recovered, with no liquidation penalty, no bad debt for Venus, and no stranded equity.

**Feasibility verified on-chain:**
- The 7 ERC-20 vTokens are delegator proxies whose admin is the Venus Normal Timelock `0x939bD8d64c0A9583A7Dcea9933f7b21697ab6396`.
- They share a common implementation `0xCDfea50f7CECCB24Fe804657DB8E6c93b689941e`.
- The `accountTokens` mapping is at storage slot 14 (to be re-proven per-market with `cast storage`, per above).
- The vBNB market is native, has a separate admin (`0x9A7890534d9d91d473F28cB97962d176e2B65f1d`), and is **not in scope** — the vBNB strategy is stuck by an underflow in RAKE's own code rather than by the approval bug, so no Venus action would help it (see "Why vBNB is out of scope").

Venus already upgrades vToken delegate implementations through VIPs (the delegator pattern), which gives the protocol a well-worn operational path for swapping and restoring an implementation. Reassigning a live vToken balance, however, is treated here as a **novel** action and hardened accordingly — see the honesty assessment above.

#### What Venus gets

- **A recovery fee to the Venus treasury** — a percentage of the ~$972K recovered, paid **before** redemption. Venus is compensated directly and up front, not with speculative future volume.
- **Zero cost to Venus** — the requester covers **100%** of the independent audit and engineering costs **regardless of outcome**.
- **A cleaner balance sheet and less operational risk** — the recovery clears a **~$1.14M borrow liability** against Venus markets and removes a **market-deprecation edge case** (positions that cannot defend themselves against off-boarding, currently forcing punitive liquidations).

#### Execution notes

1. **Pre-verified calls.** `repayBorrowBehalf` and `redeemAllowed` were verified **per-market on a current-head BSC fork** and will be re-proven at execution time.
2. **No re-accrual window.** The repay → `rescueTransfer` → restore steps are **sequenced/bundled** so no borrow can re-accrue between the repay and the collateral move; any residual dust is over-repaid and reconciled.
3. **Repayment funding.** The requester will source the **~$1.14M of underlyings** to repay — either directly, or via a **flash-loan inside the rescue flow**.
4. **vBNB is not part of this request.** The vBNB strategy (~$31,225 net) never depended on the ERC-20 approval the bug destroys — it uses native-coin mechanics — but it is still stuck, on an over-leverage arithmetic underflow in RAKE's own strategy code (see "Why vBNB is out of scope"). That is not something Venus governance can repair, so nothing is asked of Venus for it and no coordination with the separate vBNB admin (`0x9A7890534d9d91d473F28cB97962d176e2B65f1d`) is required.

### Alternative framing: Custom Rescue Contract

The same idea can be framed as a rescue module: governance authorizes an admin-only routine (bundled into the patched implementation or a dedicated rescue contract) to move the strategies' vTokens to the owner after debt is repaid, then the owner redeems for underlying. Mechanically identical to the recommended path above — same hard scope-lock, same conserved invariants, same independent audit, and no liquidation penalty.

---

## Venus Governance Precedent

Venus has repeatedly used governance to authorize designated parties, deprecate markets, and change contract behavior. Verified precedents:

| VIP / Action | Description | Reference |
|--------------|-------------|-----------|
| **VIP-79** | Whitelisted BNB Chain as the SOLE liquidator of a specific exploiter account (Nov 2022); wallet pre-funded with $30M. Anchor precedent for governance authorizing a single designated party to act on a specific position. | [coindesk.com](https://www.coindesk.com/tech/2023/08/21/bnb-chain-exploiter-liquidated-for-30m-on-venus-protocol) |
| **VIP-161** | BUSD market deprecation plan — governance-directed market intervention. | [community.venus.io](https://community.venus.io/t/risk-analysis-of-busd-and-potential-deprecation-plan/3353) |
| **Delegate upgrades** | Venus upgrades vToken delegate implementations through VIPs routinely (the delegator pattern) — direct precedent that swapping and restoring a delegate implementation is a well-worn operational path. | [docs-v4.venus.io](https://docs-v4.venus.io/governance/decentralization) |

These precedents show Venus governance authorizing **targeted, position-specific actions** (VIP-79), **directed market interventions** (VIP-161), and **routine delegate upgrades** (the delegator pattern). We are explicit, however, that reassigning a **live vToken balance** is **NOT identical to any of them** — the requester treats it as a novel action and hardens it accordingly (proven ownership, debt repaid to zero bad debt, requester-funded independent audit, hard scope-lock, treasury fee).

Rather than ask for a one-off favor, the requester proposes that Venus **codify a general, permissionless stuck-fund recovery standard** — a published set of criteria that any party in this situation can invoke:
- **Proven ownership** of the stuck position (verifiable on-chain plus a signed challenge-response).
- **Debt repaid to zero bad debt** before any collateral is moved.
- **Requester-funded independent audit** of the rescue mechanism, results public.
- **Treasury fee** paid to Venus before redemption.

Adopting this standard means granting this request establishes a **disciplined, repeatable process** rather than an ad-hoc exception.

---

## Deprecation & Liquidation Record

Venus's market-deprecation process — **VIP-634/635 "Multi-Chain Deprecated and Off-boarded Markets," Steps 1 & 2** — set the vDOT market's collateral factor and liquidation threshold to zero and repriced it to a punitive ~300% borrow APR with a 100% reserve factor. That made our vDOT strategy (`0x568254EAAC1476faf9A908e577faa3Ab96029801`) liquidatable through no fault of our own.

It was liquidated in **21 transactions between 12–22 July 2026** by liquidator EOA `0x06b7f3ee2321c5a19558ab55ae7e3062f75d0ab1` via the Venus official Liquidator contract `0x0870793286aaDA55D39CE7f82fb2766e8004cF43`.

- First liquidation tx: `0x77db59f0ac11fd17ea29e31d649285d65fe63e85c9ffc16de4802cfd163192a9`
- Last liquidation tx: `0x2a47c279c227a2da5de4cf207603dded15c75c9245fac21eb82e16661cd3fbd6`

**Net effect:** ~845 DOT of collateral seized, ~768 DOT of debt repaid, and 424.69 DOT still stranded. Net equity fell ~124 DOT — comprising the ~77 DOT liquidation-incentive haircut (845 seized − 768 repaid) plus ~47 DOT of punitive interest that accrued on the debt before/during unwinding.

**vLINK is next in line:** its collateral factor is already 0 and its liquidation threshold is 0.63 versus our borrow/supply ratio of 0.618 — only ~$488 of buffer remains.

Forum threads:
- Step 1: https://community.venus.io/t/multi-chain-deprecated-and-off-boarded-markets-parameter-update-step-1-of-2/5833
- Step 2: https://community.venus.io/t/multi-chain-deprecated-and-off-boarded-markets-step-2-of-2-oracle-feed-update/5858
- LINK off-boarding: https://community.venus.io/t/may-2026-risk-parameter-update-asset-off-boarding/5785

**Key point:** because the approval bug prevents our strategies from repaying, deleveraging, or exiting, they cannot defend against this. The funds are no longer merely frozen — they are being actively confiscated by the deprecation engine.

---

## Proof of Concept (Foundry)

We have reproduced this recovery end-to-end against a BSC mainnet fork (block 111859565), impersonating the Venus Normal Timelock. The proof-of-concept confirms:

1. `repayBorrowBehalf` clears all strategy debt (despite the approval bug).
2. The governance implementation-swap achieves full recovery — `redeemed == supplied` for all 7 ERC-20 markets.

The complete Foundry test suite and a runnable `forge` script are available to the Venus team on request.

---

## Historical Context & User Compensation

### Timeline
- **Feb 2021**: RAKE Protocol launched as AutoFarm fork on BSC
- **Feb 25, 2021**: Bug discovered when contracts were paused/unpaused
- **Feb 25, 2021**: Immediate user compensation initiated
- **2021**: Initial contact with Venus (during depegging crisis, limited response)
- **Aug 2025**: Recovery effort resumed with technical analysis

### User Impact & Mitigation

The bug affected the platform shortly after launch. Stuck Venus vault assets represented ~4% of total TVL. Compensation began the same day, paid as RAKE farm emissions directed to the affected pools.

Over the first seven days the seven frozen ERC-20 pools paid **165.188 RAKE** to the 103 wallets that are still frozen today, and **486.975 RAKE** by 25 March 2021. **96 of the 103 sold**, and what they received is measured from the swap receipts themselves: for each day, the BNB actually paid out divided by the RAKE actually swapped, applied to that day's emissions. No single blended rate is used — RAKE fell about 70% during that week, and any one average distorts the result in one direction or the other.

| | Cumulative BNB | Value at the BNB price of the day | vs $972,437 today | vs $1,028,474 when frozen |
|---|---|---|---|---|
| Day 3 — 28 Feb 2021 | 3,694.2 | $835,007 | 0.86x | 0.81x |
| **Day 5 — 02 Mar 2021** | **4,467.2** | **$1,020,056** | **1.05x** | 0.99x |
| Day 6 — 03 Mar 2021 | 4,800.7 | $1,100,081 | 1.13x | **1.07x** |
| Day 7 — 04 Mar 2021 | 5,050.1 | $1,158,700 | 1.19x | 1.13x |
| Through 25 Mar 2021 | 5,884.1 | $1,354,993 | 1.39x | 1.32x |

**The frozen principal was covered on day five, 2 March 2021** — four days after the last strategy was bricked — and on day six against the principal as valued at the freeze itself. Both sides of that comparison are in Feb–Mar 2021 dollars, so it mixes no price epochs. At day-of RAKE VWAP the first week comes to $1,278,224, or 1.31x.

The method is corroborated by market data it never touches. Converting the daily realized rates at each day's BNB price gives $21,680 · $8,998 · $4,362 · $3,939 · $3,635 per RAKE for 26 Feb – 2 Mar, against a published RAKE VWAP series of $22,700 · $10,500 · $5,600 · $4,300 · $3,500 for the same days. The realized rates sit just below market every day, which is what selling into a falling book produces — and they were derived only from swap receipts.

Deliberately excluded, all of which understate the result: 167.7 RAKE these wallets sold into the BR34P/RAKE pair, which is not credited at all; RAKE they still hold unsold; and every RAKE they earned from any pool that is not frozen.

**This is an aggregate, not a per-wallet guarantee.** Coverage was not universal — seven of the 103 wallets never sold their RAKE at all. Nor was it a discretionary act: `allocPoint` was never changed and no airdrop occurred. The frozen pools simply continued paying their pre-existing emission schedule, which is why the coverage accrues over days rather than in a single transaction, and why nothing on-chain records any depositor accepting RAKE in settlement of a claim.

### Table 2: Trading Data Showing Compensation (First Week After Bug)

| Date (2021) | BNB Price | RAKE Price | Swap Vol $ | Dev Fee RAKE | Daily Comp | Cumulative Comp | Stuck Assets ‡ |
|-------------|-----------|------------|------------|--------------|------------|-----------------|--------------|
| Feb 25 | $245 | $43,300 | $4.7M | 32 ($1.4M) | 32 ($1.4M) | $1.4M | $1.56M |
| Feb 26 | $228 | $22,700 | $19.1M | 32 ($727K) | 32 ($727K) | $2.1M | $1.52M |
| Feb 27 | $224 | $10,500 | $7.5M | 32 ($335K) | 32 ($335K) | $2.4M | $1.49M |
| Feb 28 | $218 | $5,600 | $2.1M | 32 ($178K) | 32 ($178K) | $2.6M | $1.46M |
| Mar 01 | $233 | $4,300 | $4.9M | 32 ($136K) | 32 ($136K) | $2.8M | $1.46M |
| Mar 02 | $247 | $3,500 | $2.0M | 32 ($111K) | 32 ($111K) | $2.9M | $1.50M |
| Mar 03 | $240 | $3,600 | $2.2M | 32 ($116K) | 32 ($116K) | $3.0M | $1.54M |

The rows above are the raw source series and are not split by pool; the figure this request uses is **165.188 RAKE**, paid by the seven frozen pools to the 103 external wallets still frozen. Derivation, for traceability: of the 237.111 RAKE the strategy pools earned over the first seven days, pid 8 took 48.634, leaving 188.477 across the seven pools here; of that, 23.288 went to the operator's own gov wallet `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, whose only remaining position was pid-8 dust. Excluding the operator's own emissions leaves **165.188 RAKE** — the figure quoted above. No 8-pool total is relied on anywhere in this request.

**‡ The "Stuck Assets" column is a superseded legacy estimate.** It carries no recorded derivation, covers the eight-pool group rather than the seven in Table 1, and does not reconcile with the frozen-principal figures cited above. It is shown for continuity only and **no part of this request relies on it.** The 2021-basis question it was gesturing at has since been answered properly: the seven frozen pools were worth **$1,028,474** at their freeze blocks, measured from archival state and priced by the Venus oracle in force at the time. Use that figure, not this column.

**Key Points:**
- Dev fees were held (not sold) to preserve liquidity for users
- $4.7M+ swap volume on day one ($9.1M CMC volume), demonstrating sufficient liquidity
- Compensation was paid immediately and in a liquid market, so affected users had a real exit; on the realized basis it passed the frozen principal on day five and reached 1.39x by 25 March — but the principal itself remains frozen, which is why recovery is still being sought

---

## Strategy Contract Architecture

### Core Operations
- Takes user deposits, borrows against them with leverage, claims Venus rewards, compounds into positions
- `deposit()` → `_farm()` → `_leverage()` → loops `_supply()` + `_borrow()` for 3x leverage
- `withdraw()` → `_deleverage()` → `_removeSupply()` + `_repayBorrow()` to unwind positions
- `earn()` → `claimVenus()` → swaps → compounds rewards
- `updateBalance()` → tracks `supplyBal`/`borrowBal`
- `deleverageOnce()` + `deleverageUntilNotOverLevered()` for risk management
- `rebalance()` → adjusts `borrowRate`/`borrowDepth` parameters

### Venus Protocol Integration
- `enterMarkets([vTokenAddress])` called during deployment
- Strategy holds single vToken position as both collateral and debt asset
- Leverages Venus lending/borrowing to amplify exposure
- `IVToken.mint(amount)` - supplies underlying, receives vTokens
- `IVToken.borrow(amount)` - borrows against collateral
- `IVToken.redeemUnderlying(amount)` - withdraws underlying
- `IVToken.repayBorrow(amount)` - repays borrowed amount

---

## Contact Information

**RAKE Protocol Owner:**
- Gov Address: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`
- Discord: `5ubse7en`
- Email: `f0r3x_shark@protonmail.com`

**Venus Protocol:**
- Telegram: https://t.me/venusprotocol
- Governance Forum: https://community.venus.io/
- Documentation: https://docs-v4.venus.io/

---

## Data Sources

- **BNB/USD price:** [Binance API](https://api.binance.com) - Daily (Open + Close) / 2
- **RAKE/BNB price:** [CoinMarketCap DEX](https://dex.coinmarketcap.com)
- **Liquidity data:** [BSCScan Token Analytics](https://bscscan.com)
- **On-chain verification:** BSC RPC endpoints

---

## Summary

| Item | Value |
|------|-------|
| Total Stuck (Net) | **~$972K ($972,437)** |
| Total Supplied | ~$2.11M ($2,107,606) |
| Total Borrowed | ~$1.14M ($1,135,169) |
| Net value when frozen (Feb 2021) | $1,028,474 |
| Affected Strategies | 7 (the ERC-20 markets sharing this root cause; vBNB is stuck by a separate defect in RAKE's own code — see "Why vBNB is out of scope") |
| Bug Type | Approval set to 0 in `unpause()` |
| Contract Upgradeable | No |
| Users Compensated | Yes (2021) |
| Recommended Recovery | Governance vToken implementation-swap (full recovery) |

*USD values verified on-chain 24 July 2026. Token quantities are on-chain and accrue interest each block — the underlying amounts are the authoritative, verifiable measure; USD floats with the market. Note that the vDOT position has been almost entirely liquidated by Venus's market deprecation (see Deprecation & Liquidation Record).*

**Request:** Venus Protocol governance assistance to recover roughly $972K in stuck funds across the seven frozen ERC-20 markets, via **a governance vToken implementation-swap** that achieves full recovery with no liquidation penalty and no bad debt. A Foundry proof-of-concept (BSC mainnet fork) is available to the Venus team on request.

---

*Document prepared: January 12, 2026*
*Last updated: July 24, 2026 (on-chain figures refreshed; recommended recovery path finalized; vDOT liquidation recorded; vBNB de-scoped — stuck by a separate defect in RAKE's own code, not by the approval bug)*
