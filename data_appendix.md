# Data Appendix - RAKE Protocol Stuck Funds Analysis

> **📋 Related Documents:** [Recovery Proposal](./recovery_proposal.md)

## Quick Reference - Table Index

- **[Table 1: Venus Strategy Contract Positions](#table-1-venus-strategy-contract-positions)** - Detailed breakdown of all affected contracts
- **[Table 2: Complete Daily Trading and Liquidity Data](#table-2-complete-daily-trading-and-liquidity-data)** - Complete Daily Trading and Liquidity Data
---

## A. Technical Root Cause

**Bug Description**: AutoFarm strategy contract inheritance flaw

- `pause()` function correctly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- `unpause()` function incorrectly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- **Correct implementation should be**: `IERC20(wantAddress).safeApprove(vTokenAddress, uint256(-1))`

**Impact**: Permanently locks approval at zero, preventing all Venus Protocol interactions.

## B. Affected Contract Details

**Deployer/Owner**: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`

**Farm Contract Address**: `0x7f7Bf15B9c68D23339C31652C8e860492991760d`



### Table 1: Venus Strategy Contract Positions (data taken 15th August 2025)

| Contract Address | vToken | Total vTokens | Underlying | Supplied | Borrowed | Supplied USD | Borrowed USD | Net USD |
|------------------|--------|---------------|------------|----------|----------|--------------|--------------|---------|
| [0xB3bC...3c12](https://bscscan.com/address/0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12) | vBTC | 450K | BTCB | 9 | 5 | $1.08M | $616K | $468K |
| [0x5682...9801](https://bscscan.com/address/0x568254EAAC1476faf9A908e577faa3Ab96029801) | vDOT | 55K | DOT | 1.25K | 616 | $4.8K | $2.4K | $2.4K |
| [0x7485...069a](https://bscscan.com/address/0x7485704d8b6cebd1411f5aafeb063e6f2816069a) | vUSDC | 21M | USDC | 544K | 329K | $544K | $329K | $215K |
| [0xc6f4...f6d](https://bscscan.com/address/0xc6f470da6c284d16647cee2230522a85b0818f6d) | vUSDT | 10M | USDT | 257K | 156K | $257K | $156K | $101K |
| [0x0b09...0169](https://bscscan.com/address/0x0b09Efc9458c00354414D2A560aA6EDa19490169) | vLINK | 230K | LINK | 4.7K | 2.9K | $83K | $50K | $33K |
| [0x9E3e...d19C](https://bscscan.com/address/0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C) | vETH | 11K | ETH | 233 | 136 | $876K | $512K | $364K |
| [0xDF3d...859C](https://bscscan.com/address/0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C) | vBUSD | 9M | BUSD | 207K | 0 | $207K | $0 | $207K |
| [0xf498...e122](https://bscscan.com/address/0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122) | vBNB | 7K | BNB | 184 | 127 | $144K | $100K | $44K |
| **TOTAL** | | | | | | **$3.20M** | **$1.77M** | **$1.43M** |

## Summary
- **Total Supplied:** $3.20M
- **Total Borrowed:** $1.77M  
- **Net Position:** $1.43M

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

# RakeFarm LP Pool Analysis 

| Date (2021) | BNB Price¹ | RAKE Price² | Liq Add $³ | Liq Rem $³ | Swapped RAKE $⁴ | Swapped WBNB $⁴ | Dev Fee RAKE⁵| Daily Comp⁵ | Cumulative Comp | Stuck Assets⁵ |
|------|-------|--------|-----------|-----------|-------------|-------------|---------|---------------|-------------|--------------|
| Feb 25 | $280 | $125K | $332M | $143M | $2.73B | $41K | 32 ($4.0M) | 32 ($4.0M) | $4.0M | $1.56M |
| Feb 26 | $260 | $26K | $271M | $89.5M | $2.17B | $227K | 32 ($827K) | 32 ($826K) | $4.8M | $1.52M |
| Feb 27 | $270 | $15.9K | $107M | $72.9M | $522M | $215K | 32 ($507K) | 32 ($506K) | $5.3M | $1.49M |
| Feb 28 | $280 | $3.9K | $5.7M | $5.3M | $35.3M | $124K | 32 ($125K) | 32 ($125K) | $5.4M | $1.46M |
| Mar 01 | $280 | $3.5K | $11.7M | $4.9M | $71.3M | $314K | 32 ($111K) | 32 ($111K) | $5.5M | $1.46M |
| Mar 02 | $280 | $3.5K | $5.9M | $2.7M | $28.2M | $152K | 32 ($111K) | 32 ($111K) | $5.7M | $1.50M |
| Mar 03 | $280 | $3.5K | $7.3M | $2.4M | $32.6M | $180K | 32 ($111K) | 32 ($111K) | $5.8M | $1.54M |
| Mar 04 | $280 | $3K | $4.6M | $9.8M | $39.1M | $318K | 32 ($95K) | 32 ($95K) | $5.9M | $1.51M |
| Mar 05 | $280 | $1.8K | $1.6M | $714K | $8.7M | $156K | 32 ($56K) | 19 ($33K) | $5.9M | $1.48M |
| Mar 06 | $280 | $1.5K | $561K | $2.8M | $9.1M | $259K | 32 ($48K) | 19 ($28K) | $5.9M | $1.46M |
| Mar 07 | $280 | $1.4K | $290K | $38K | $2.5M | $90K | 32 ($45K) | 19 ($26K) | $6.0M | $1.44M |

---

## F. Data Sources

**¹** BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)

**²** RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)

**³** RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)

**⁴** Liquidity/Swap info: Events from LP token (Burn/Mint and Swap) using scripts

**⁵** Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)

