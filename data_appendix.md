# Data Appendix - RAKE Protocol Stuck Funds Analysis

> **📋 Related Documents:** [Recovery Proposal](./recovery_proposal.md)

## Quick Reference - Table Index

- **[Table 1: Venus Strategy Contract Positions](#table-1-venus-strategy-contract-positions-live-on-chain-data---july-2026)** - Detailed breakdown of all affected contracts (live on-chain data)
- **[Why vBNB (pid 8) is excluded](#why-vbnb-pid-8-is-excluded)** - vBNB is also stuck, but from a separate defect Venus cannot fix
- **[Table 2: Complete Daily Trading and Liquidity Data](#table-2-complete-daily-trading-and-liquidity-data)** - Complete Daily Trading and Liquidity Data
- **[Section E: On-Chain Verification Commands](#e-on-chain-verification-commands)** - Commands to verify stuck funds
- **[Proof of Ownership (on-chain)](#proof-of-ownership-on-chain)** - Every strategy's `govAddress()` maps to the requester's gov address
- **[Section F: Deprecation and Liquidation Record](#f-deprecation-and-liquidation-record)** - Venus market deprecation, the vDOT liquidation, and the vLINK risk
- **[Section G: Proof of Concept (Foundry)](#g-proof-of-concept-foundry)** - Runnable BSC-fork test suite proving recovery

---

## A. Technical Root Cause

**Bug Description**: AutoFarm strategy contract inheritance flaw

- `pause()` function correctly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- `unpause()` function incorrectly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- **Correct implementation should be**: `IERC20(wantAddress).safeApprove(vTokenAddress, uint256(-1))`

**Impact**: Permanently locks the `want → vToken` approval at zero on the **seven ERC-20 strategies** (pids 9–15), so every strategy operation that moves the underlying into or out of Venus reverts `BEP20: transfer amount exceeds allowance` (Section C). The affected code path is guarded by `if (!wantIsWBNB)` and so never executes on the native-BNB strategy.

### Two independent locks on the ERC-20 pools

A user attempting to exit one of these pools hits **two** distinct locks, and both must be cleared:

1. **Venus's own action pause on the deprecated market (hit first).** `withdraw(9, 1e18)` on the vBUSD pool reverts with **`action is paused`**. That is the Venus **Comptroller's** error string — it is not the strategy's `Pausable`, which reverts with `Pausable: paused`. A Venus engineer can confirm the string is theirs. This lock is a consequence of the market deprecation described in Section F, and it is Venus-side parameter state.
2. **The zero allowance from `unpause()` (independent).** Even with the action pause lifted, every path that moves the underlying still reverts on the allowance — the strategy has no function that can restore it, because it is not upgradeable and `inCaseTokensGetStuck()` protects `wantAddress` (Section C).

This appendix and the recovery request concern lock 2, which the strategy cannot repair on its own at any price. Lock 1 is stated here so the picture is complete and so the two error strings are not confused.

### Why the approval bug cannot reach the native-BNB market (reproducible)

A zero `want → vToken` allowance only matters if the vToken pulls the underlying with `transferFrom`. That is true exactly when the market has an ERC-20 `underlying()`. The native-BNB market has none — its `mint()` is `payable` — so no allowance is ever consulted and no allowance can be set wrong. The distinguishing test is one call per market:

```bash
RPC=https://bsc-dataseed.binance.org

# a FROZEN market — vBTC has an ERC-20 underlying, so mint() pulls via transferFrom
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B "underlying()(address)" --rpc-url $RPC
# -> 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c   (BTCB)  => needs an allowance, which is 0 => bricked

# vBNB has NO underlying() function at all — it is the native-BNB market
cast call 0xA07c5b74C9B40447a954e1466938b865b6BBea36 "underlying()(address)" --rpc-url $RPC
# -> reverts / no such function.  mint() is payable; no transferFrom; no allowance is ever consulted
```

Confirmed live on BSC (chain id 56) for the vBNB strategy `0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122`: its `wantAddress()` is `0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c` (WBNB), and its WBNB → vBNB allowance *is* zero — but that approval is a **no-op**, because vBNB never calls `transferFrom`. The same zero allowance that bricks the seven ERC-20 strategies is inert there.

**This does not mean the vBNB pool is fine.** It is also stuck, for an unrelated reason — an arithmetic underflow in RAKE's own strategy code. See "Why vBNB (pid 8) is excluded" below. The scope of this request is the seven pools that share the approval root cause; vBNB is a separate defect and is pursued separately.

## B. Affected Contract Details

**Deployer/Owner**: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`

**Farm Contract Address**: `0x7f7Bf15B9c68D23339C31652C8e860492991760d`



### Table 1: Venus Strategy Contract Positions (Live On-Chain Data - July 2026)

**Verified on-chain:** 24 July 2026 (BSC mainnet)
**Prices Used (spot):** BTC $64,076.26 | ETH $1,895.19 | BNB $566.50 | LINK $8.29 | DOT $0.76 | BUSD $1.00 | USDT $0.998773 | USDC $0.999746

| Contract Address | vToken | Underlying | Supplied | Borrowed | Net | Supplied USD | Borrowed USD | Net USD |
|------------------|--------|------------|----------|----------|-----|--------------|--------------|---------|
| [0xB3bC...3c12](https://bscscan.com/address/0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12) | vBTC | BTCB | 9.2324 | 5.3029 | 3.9295 | $599,210 | $344,174 | $255,036 |
| [0x9E3e...d19C](https://bscscan.com/address/0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C) | vETH | ETH | 234.44 | 138.07 | 96.37 | $441,232 | $259,845 | $181,386 |
| [0x7485...069A](https://bscscan.com/address/0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A) | vUSDC | USDC | 560,872 | 345,364 | 215,508 | $560,828 | $345,335 | $215,493 |
| [0xDF3d...859C](https://bscscan.com/address/0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C) | vBUSD | BUSD | 206,755 | 0 | 206,755 | $206,298 | $0 | $206,298 |
| [0xc6f4...8f6D](https://bscscan.com/address/0xc6f470dA6C284D16647CEE2230522a85b0818f6D) | vUSDT | USDT | 264,646 | 163,973 | 100,673 | $264,450 | $163,851 | $100,600 |
| [0x0b09...0169](https://bscscan.com/address/0x0b09Efc9458c00354414D2A560aA6EDa19490169) | vLINK | LINK | 4,736.05 | 2,926.23 | 1,809.82 | $40,067 | $24,756 | $15,311 |
| [0x5682...9801](https://bscscan.com/address/0x568254EAAC1476faf9A908e577faa3Ab96029801) † | vDOT | DOT | 424.69 | 0.41 | 424.28 | $341 | $0 | $340 |
| **TOTAL** | | | | | | **~$2.11M** ($2,107,606) | **~$1.14M** ($1,135,169) | **~$972K** ($972,437) |

† vDOT was liquidated by Venus's market deprecation in July 2026 — see Section F.

*Per-row USD is rounded to the nearest dollar; the TOTAL is computed from unrounded values.*

### Why vBNB (pid 8) is excluded

The farm ran **eight** Venus leveraged strategies (pids 8–15). Table 1 covers the **seven ERC-20 strategies** (pids 9–15), which share the single approval root cause in Section A. The eighth, vBNB (`0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122`, pid 8), is excluded because it is stuck for a **different reason**, not because it is unaffected.

**The vBNB position is still stuck.** 184.493698 BNB supplied / 129.373745 borrowed, **55.119953 BNB net (~$31,225)** across **69 positions**. Verified on-chain:

- `RakeFarm.withdraw(8, 1)` from a real holder reverts **`SafeMath: subtraction overflow`**.
- Simulated from the gov wallet, `deleverageOnce()` and `deleverageUntilNotOverLevered()` revert with the **same underflow**. The deleverage path cannot clear it unaided. The deleverage path is unusable; unbricking it requires paying down the Venus borrow from outside the contract, which is a separate exercise and not part of this request.
- Only `withdraw(8, 0)` (harvest) succeeds.

**Cause — an over-leverage arithmetic underflow in RAKE's own strategy, not the approval bug.** The strategy targets `borrowRate` 585 (0.585) with `BORROW_RATE_MAX` 595, but the live position sits at 129.373745 / 184.493698 = **0.7012**, above its own ceiling. It drifted there over five years because borrow interest outran supply interest (**+29.25 BNB** of debt against **+20.62 BNB** of supply). The unwind path then computes the required supply as 129.373745 / 0.585 = **221.15 BNB**, subtracts it from the actual 184.49, goes negative, and SafeMath reverts. There is no branch handling a position that is *already* over the limit.

Venus cannot fix an arithmetic underflow in a third-party contract, so this pool is not part of the Venus request and is excluded from every total in this document. It is being pursued separately.

## Summary
- **Total Supplied:** ~$2.11M ($2,107,606)
- **Total Borrowed:** ~$1.14M ($1,135,169)
- **Net Position:** ~$972K ($972,437) at current spot prices

*Note: USD values fluctuate with market prices, so the net figure floats with the market. Token quantities are on-chain and accrue interest every block, and because these are deprecated markets carrying a punitive borrow curve, the debt compounds faster than the supply: both legs drift upward, but the borrowed leg outruns the supplied leg, so **net underlying drifts slightly downward** over time.*

*An earlier revision of this note cited a net of ~$1.26M at January 2026 prices. That figure was computed on the **eight**-pool basis, including vBNB, and has not been restated for the seven pools — it is left here unrevised rather than re-estimated, and should not be compared against the $972,437 above.*

## C. Info on attempts to recover funds

We performed various tests on bsc fork chains using foundry. Most scripts ended with `BEP20: transfer amount exceeds allowance` error.

Tested functions (among others):
- `deposit()`
- `withdraw()`
- `farm()`
- `earn()`
- `deleverageOnce()`
- `deleverageUntilNotOverLevered()`
- `rebalance()`

Some scenarios we tested  :

### **Scenario 1: Emergency Token Extraction**

Tested `inCaseTokensGetStuck()` function to extract various tokens:

- **vBTC extraction**: Failed with `!safe` error (protected as vTokenAddress)
- **Venus token extraction**: Failed with `!safe` error (protected as earnedAddress)

**Result**: Core protocol tokens (vBTC, Venus, want tokens) remain stuck due to safety checks.

### **Scenario 2: Complete Position Unwinding**

Attempted forced deleveraging via `rebalance(0, 0)`:

- **Rebalance execution**: Failed with `BEP20: transfer amount exceeds allowance`
- **Position closure**: Unable to close leveraged positions due to approval issues
- **Fund liberation**: Leveraged funds remain locked in Venus protocol

**Result**: Complete position unwinding impossible due to broken token approvals preventing Venus interactions.

### **Scenario 3: Manual Deleveraging Steps**

Attempted step-by-step deleveraging approach:

- **`deleverageUntilNotOverLevered()`**: Failed with allowance error
- **`deleverageOnce()`**: Failed with allowance error on Venus interactions
- **Manual position reduction**: All Venus-dependent operations blocked

**Result**: Manual deleveraging fails at the same point as automated methods - Venus protocol interactions are blocked by insufficient token approvals.

**Root Cause**: The `unpause()` function bug that sets `wantAddress` approval to 0 instead of unlimited has rendered all Venus protocol interactions impossible, effectively locking all leveraged funds.

## D. Strategy Contract Architecture

**Core Operations**:
* Takes user deposits, borrows against them with leverage, claims Venus rewards, compounds everything back into positions
* `deposit()` → `_farm()` → `_leverage()` → loops `_supply()` + `_borrow()` for 3x leverage
* `withdraw()` → `_deleverage()` → `_removeSupply()` + `_repayBorrow()` to unwind positions
* `earn()` → `claimVenus()` → `swapExactTokensForTokens()` → compounds rewards
* `updateBalance()` → tracks `supplyBal`/`borrowBal` via `balanceOfUnderlying()` + `borrowBalanceCurrent()`
* `deleverageOnce()` + `deleverageUntilNotOverLevered()` for risk management
* `rebalance()` → adjusts `borrowRate`/`borrowDepth` strategy parameters

**Venus Protocol Integration Details**:
* `enterMarkets([vTokenAddress])` called during deployment to enable collateral usage
* Strategy holds single vToken position (e.g., vBTC) as both collateral and debt asset
* Leverages Venus lending/borrowing to amplify exposure to underlying asset
* `IVToken.mint(amount)` - supplies underlying to Venus, receives vTokens
* `IVToken.borrow(amount)` - borrows underlying against vToken collateral
* `IVToken.redeemUnderlying(amount)` - withdraws underlying, burns vTokens
* `IVToken.repayBorrow(amount)` - repays borrowed underlying
* `IVToken.balanceOfUnderlying(address)` - returns supplied balance + accrued interest
* `IVToken.borrowBalanceCurrent(address)` - returns borrowed balance + accrued interest
* `IVenusDistribution.claimVenus(address)` - claims XVS rewards

**The Critical Bug Impact on Venus Integration**:
* `unpause()` incorrectly sets `IERC20(wantAddress).approve(vTokenAddress, 0)`
* This breaks ALL Venus operations requiring underlying token transfers:
 - `IVToken.mint()` fails - cannot supply more collateral
 - `IVToken.repayBorrow()` fails - cannot repay debt
 - `_leverage()` fails - cannot build positions
 - `_deleverage()` fails - cannot unwind positions

## E. On-Chain Verification Commands

Anyone can verify the stuck funds using these commands:

```bash
# ========== vBTC Strategy ==========
# Supply
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "balanceOfUnderlying(address)(uint256)" 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org
# Borrow
cast call 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  "borrowBalanceCurrent(address)(uint256)" 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 \
  --rpc-url https://bsc-dataseed.binance.org
# Approval (returns 0 - THE BUG)
cast call 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c \
  "allowance(address,address)(uint256)" 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 0x882C173bC7Ff3b7786CA16dfeD3DFFfb9Ee7847B \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vETH Strategy ==========
cast call 0xf508fCD89b8bd15579dc79A6827cB4686A3592c8 \
  "balanceOfUnderlying(address)(uint256)" 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0xf508fCD89b8bd15579dc79A6827cB4686A3592c8 \
  "borrowBalanceCurrent(address)(uint256)" 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vUSDC Strategy ==========
cast call 0xecA88125a5ADbe82614ffC12D0DB554E2e2867C8 \
  "balanceOfUnderlying(address)(uint256)" 0x7485704d8b6cebd1411f5aafeb063e6f2816069a \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0xecA88125a5ADbe82614ffC12D0DB554E2e2867C8 \
  "borrowBalanceCurrent(address)(uint256)" 0x7485704d8b6cebd1411f5aafeb063e6f2816069a \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vUSDT Strategy ==========
cast call 0xfD5840Cd36d94D7229439859C0112a4185BC0255 \
  "balanceOfUnderlying(address)(uint256)" 0xc6f470da6c284d16647cee2230522a85b0818f6d \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0xfD5840Cd36d94D7229439859C0112a4185BC0255 \
  "borrowBalanceCurrent(address)(uint256)" 0xc6f470da6c284d16647cee2230522a85b0818f6d \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vLINK Strategy ==========
cast call 0x650b940a1033B8A1b1873f78730FcFC73ec11f1f \
  "balanceOfUnderlying(address)(uint256)" 0x0b09Efc9458c00354414D2A560aA6EDa19490169 \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0x650b940a1033B8A1b1873f78730FcFC73ec11f1f \
  "borrowBalanceCurrent(address)(uint256)" 0x0b09Efc9458c00354414D2A560aA6EDa19490169 \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vDOT Strategy ==========
cast call 0x1610bc33319e9398de5f57B33a5b184c806aD217 \
  "balanceOfUnderlying(address)(uint256)" 0x568254EAAC1476faf9A908e577faa3Ab96029801 \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0x1610bc33319e9398de5f57B33a5b184c806aD217 \
  "borrowBalanceCurrent(address)(uint256)" 0x568254EAAC1476faf9A908e577faa3Ab96029801 \
  --rpc-url https://bsc-dataseed.binance.org

# ========== vBUSD Strategy ==========
cast call 0x95c78222B3D6e262426483D42CfA53685A67Ab9D \
  "balanceOfUnderlying(address)(uint256)" 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C \
  --rpc-url https://bsc-dataseed.binance.org
cast call 0x95c78222B3D6e262426483D42CfA53685A67Ab9D \
  "borrowBalanceCurrent(address)(uint256)" 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C \
  --rpc-url https://bsc-dataseed.binance.org
```

### Proof of Ownership (on-chain)

Every one of the 7 frozen ERC-20 strategy contracts returns `govAddress()` equal to the requester's gov address `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, which proves control of the strategies. Anyone can confirm this:

```bash
for s in 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C 0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C 0xc6f470dA6C284D16647CEE2230522a85b0818f6D 0x0b09Efc9458c00354414D2A560aA6EDa19490169 0x568254EAAC1476faf9A908e577faa3Ab96029801; do
  cast call $s "govAddress()(address)" --rpc-url https://bsc-dataseed.binance.org
done
# each returns 0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07
```

The 7 strategy addresses:
- `0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12` (vBTC)
- `0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C` (vETH)
- `0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A` (vUSDC)
- `0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C` (vBUSD)
- `0xc6f470dA6C284D16647CEE2230522a85b0818f6D` (vUSDT)
- `0x0b09Efc9458c00354414D2A560aA6EDa19490169` (vLINK)
- `0x568254EAAC1476faf9A908e577faa3Ab96029801` (vDOT)

The same call returns the same gov address for the eighth (vBNB) strategy `0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122`, which is **not** part of this request — see "Why vBNB (pid 8) is excluded" above.

### Table 2: Complete Daily Trading and Liquidity Data

| Date (2021) | BNB Price | RAKE Price (VWAP) | Liq Add $ | Liq Removed $ | Swapped RAKE $ | Swapped BNB $ | Dev Fee RAKE | Daily Comp | Cumulative Comp | Stuck Assets ‡ |
|-------------|-----------|-------------------|-----------|-----------|----------------|---------------|--------------|------------|-----------------|--------------|
| Feb 25 | $245 | $43.3K | $1.24M | $541K | $5.18M | $4.74M | 32 ($1.4M) | 32 ($1.4M) | $1.4M | $1.56M |
| Feb 26 | $228 | $22.7K | $4.71M | $1.64M | $19.8M | $19.1M | 32 ($727K) | 32 ($727K) | $2.1M | $1.52M |
| Feb 27 | $224 | $10.5K | $3.15M | $2.24M | $8.14M | $7.49M | 32 ($335K) | 32 ($335K) | $2.4M | $1.49M |
| Feb 28 | $218 | $5.6K | $797K | $693K | $2.47M | $2.10M | 32 ($178K) | 32 ($178K) | $2.6M | $1.46M |
| Mar 01 | $233 | $4.3K | $1.62M | $672K | $4.99M | $4.88M | 32 ($136K) | 32 ($136K) | $2.8M | $1.46M |
| Mar 02 | $247 | $3.5K | $815K | $368K | $1.88M | $1.98M | 32 ($111K) | 32 ($111K) | $2.9M | $1.50M |
| Mar 03 | $240 | $3.6K | $1.01M | $357K | $2.28M | $2.20M | 32 ($116K) | 32 ($116K) | $3.0M | $1.54M |
| Mar 04 | $235 | $3.1K | $698K | $1.64M | $3.32M | $2.98M | 32 ($101K) | 32 ($101K) | $3.1M | $1.51M |
| Mar 05 | $228 | $2.3K | $512K | $198K | $1.39M | $1.21M | 32 ($73K) | 19 ($43K) | $3.1M | $1.48M |
| Mar 06 | $226 | $1.6K | $167K | $888K | $1.49M | $1.41M | 32 ($50K) | 19 ($30K) | $3.2M | $1.46M |
| Mar 07 | $233 | $1.3K | $197K | $79K | $896K | $905K | 32 ($42K) | 19 ($25K) | $3.2M | $1.44M |

**Column Methodology:**
- **Liq Add $**: Total USD value of liquidity added = (RAKE added × RAKE price) + (WBNB added × BNB price)
- **Liq Rem $**: Total USD value of liquidity removed = (RAKE removed × RAKE price) + (WBNB removed × BNB price)
- **Swapped RAKE $**: USD value of RAKE tokens traded = RAKE traded × RAKE VWAP price
- **Swapped BNB $**: USD value of BNB tokens traded = WBNB traded × BNB price
- **Dev Fee RAKE**: Platform fees earned in RAKE tokens (never sold, preserving liquidity for users)
- **Daily Comp**: RAKE tokens distributed as compensation to users with stuck funds
- **Cumulative Comp**: Running total of compensation distributed at daily RAKE prices
- **Stuck Assets ‡**: **UNDOCUMENTED LEGACY ESTIMATE — NOT REPRODUCED.** See the note below before citing it.

**‡ On the "Stuck Assets" column.** These values carry no derivation. They arrived with an earlier revision of this appendix, have **not** been reproduced from any source data in this repository, and no methodology for them was ever recorded. Three specific problems:

- They are denominated in **2021 dollars**, whereas every other valuation in this pack is at the 24 July 2026 spot prices listed under Table 1.
- They do **not** reconcile with the frozen-principal figures used elsewhere in this pack, and are materially larger than them.
- They cover the **eight**-pool group, so they cannot be compared against the seven-pool totals in Table 1 even after converting the basis.

The column is retained here only so that nothing is lost, and it is **not relied on anywhere in the recovery request**. Do not cite it as evidence. Reconstructing a defensible 2021-basis figure would require archival on-chain state that this repository does not contain — it has deliberately not been estimated.

*Scope of the Daily Comp column:* the per-day rows are not split by pool, so they are stated here only as the raw source series. The figure this pack actually uses is **165.188 RAKE** — the amount the **seven frozen ERC-20 pools** paid to the **103 external wallets that are still frozen** over the first seven days. The derivation from the source series, for traceability: of the 237.111 RAKE the eight strategy pools earned in that period, pid 8 (vBNB) took 48.634, leaving 188.477 across the seven pools in this request; of that, 23.288 went to the operator's own gov wallet `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`, whose only remaining position was pid-8 dust. Excluding the operator's own emissions leaves **165.188 RAKE**.

Valued from the swap receipts — each day's emissions at the BNB-per-RAKE rate those wallets actually achieved that day — the 165.188 RAKE came to **5,050.1 BNB by day seven**, having passed the $972,437 of frozen principal on **day five, 2 March 2021** at 4,467.2 BNB. By 25 March 2021 the seven pools had paid **486.975 RAKE**, worth **5,884.1 BNB**. No 8-pool figure is relied on anywhere in this pack.

---

## F. Deprecation and Liquidation Record

Venus's "Multi-Chain Deprecated and Off-boarded Markets" wind-down (VIP-634 / VIP-635) executed in two steps:

- **Step 1** — supply and borrow caps set to 0, reserve factor set to 100%, and a punitive interest curve installed (~300% borrow APR).
- **Step 2** — collateral factor and liquidation threshold set to 0.

The combination made the vDOT strategy liquidatable with **no price move involved** — the borrow position exceeded a collateral threshold that governance had forced to zero.

**Verified on-chain today:**
- vDOT reserve factor = 100%
- vDOT realized borrow rate ≈ 297% APR (debt grew from 721 to ~768 DOT in ~8 days)
- vBUSD reserve factor also 100%

**The vDOT liquidation.** The vDOT strategy (`0x568254EAAC1476faf9A908e577faa3Ab96029801`) was liquidated in **21 transactions between 12–22 July 2026** by liquidator EOA `0x06b7f3ee2321c5a19558ab55ae7e3062f75d0ab1`, routed through the Venus official Liquidator contract `0x0870793286aaDA55D39CE7f82fb2766e8004cF43`.

| Date/Time (UTC) | Block | Transaction Hash | Note |
|-----------------|-------|------------------|------|
| 2026-07-12 11:43 | 109549615 | [0x77db59f0ac11fd17ea29e31d649285d65fe63e85c9ffc16de4802cfd163192a9](https://bscscan.com/tx/0x77db59f0ac11fd17ea29e31d649285d65fe63e85c9ffc16de4802cfd163192a9) | First; repaid 316.83 DOT, seized 15,191.40 vDOT |
| 2026-07-12 | 109549624 | [0xfd60e98cb0ae5af4bea51a9513f62742ec80a742c2b8d43d0a02fd2effd95bfe](https://bscscan.com/tx/0xfd60e98cb0ae5af4bea51a9513f62742ec80a742c2b8d43d0a02fd2effd95bfe) | |
| 2026-07-12 | 109549626–109549672 | *(16 further liquidations, same day)* | |
| 2026-07-14 09:24 | 109914957 | [0xe3b685ec14e3365299b6309fdeb3b4afe47d7b36aa359293837a60e9ca1b9ef9](https://bscscan.com/tx/0xe3b685ec14e3365299b6309fdeb3b4afe47d7b36aa359293837a60e9ca1b9ef9) | |
| 2026-07-22 13:09 | 111480318 | [0x5a25572f8eaf27b816109bae5d2959e21beb0931032941a965ce69830668f0c0](https://bscscan.com/tx/0x5a25572f8eaf27b816109bae5d2959e21beb0931032941a965ce69830668f0c0) | |
| 2026-07-22 23:38 | 111564102 | [0x2a47c279c227a2da5de4cf207603dded15c75c9245fac21eb82e16661cd3fbd6](https://bscscan.com/tx/0x2a47c279c227a2da5de4cf207603dded15c75c9245fac21eb82e16661cd3fbd6) | Last |

**Net effect on vDOT:** ~768 DOT of debt was repaid by the liquidator, ~845 DOT of collateral was seized (10% Venus liquidation incentive; the Liquidator contract's treasury keeps 50% of that bonus), and net equity fell ~124 DOT — the ~77 DOT liquidation-incentive haircut (845 seized − 768 repaid) plus ~47 DOT of punitive interest accrued on the debt — with **424.69 DOT remaining stranded** in the strategy.

**vLINK is next.** Its collateral factor is already 0 and its liquidation threshold is 0.63, versus the strategy's borrow/supply ratio of 0.618 (2,926.23 / 4,736.05) — only ~$488 of buffer remains. LINK is being off-boarded via the May 2026 risk update.

**Forum references:**
- Step 1: https://community.venus.io/t/multi-chain-deprecated-and-off-boarded-markets-parameter-update-step-1-of-2/5833
- Step 2: https://community.venus.io/t/multi-chain-deprecated-and-off-boarded-markets-step-2-of-2-oracle-feed-update/5858
- LINK off-boarding: https://community.venus.io/t/may-2026-risk-parameter-update-asset-off-boarding/5785
- BUSD deprecation precedent: https://community.venus.io/t/busd-deprecation-forced-liquidations/3784

Anyone can independently verify the vDOT liquidation by inspecting the transactions above on https://bscscan.com or with `cast receipt <tx> --rpc-url <bsc-rpc>`.

## G. Proof of Concept (Foundry)

We have reproduced the recovery against a BSC mainnet fork (block `111859565`), impersonating the Venus Normal Timelock (`0x939bD8d64c0A9583A7Dcea9933f7b21697ab6396`). The proof-of-concept demonstrates:

- **(a)** `repayBorrowBehalf` clears all strategy debt (despite the approval bug).
- **(b)** The governance implementation-swap achieves **full recovery** — redeemed underlying equals supplied for all 7 ERC-20 markets.

The complete Foundry test suite and a runnable `forge` script are available to the Venus team on request.

---

## H. Data Sources

**** BNB/USD price: [Binance API](https://api.binance.com) - Daily (Open + Close) / 2

**** RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/) - VWAP

**** RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)

**** Liquidity/Swap info: Events from LP token (Burn/Mint and Swap) using scripts

**** Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)

