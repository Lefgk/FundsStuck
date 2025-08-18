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



### Table 1: Venus Strategy Contract Positions

| Contract Address                           | vToken | Total vTokens | Underlying Asset | Supplied | Borrowed | Supplied USD   | Borrowed USD   | Net USD        |
| ------------------------------------------ | ------ | ------------- | ---------------- | -------- | -------- | -------------- | -------------- | -------------- |
| 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 | vBTC   | 450,000       | BTCB             | 9        | 5        | $1,084,372     | $616,279       | $468,093       |
| 0x568254EAAC1476faf9A908e577faa3Ab96029801 | vDOT   | 55,000        | DOT              | 1,250    | 616      | $4,825         | $2,379         | $2,446         |
| 0x7485704d8b6cebd1411f5aafeb063e6f2816069a | vUSDC  | 21,000,000    | USDC             | 544,220  | 328,750  | $544,220       | $328,750       | $215,470       |
| 0xc6f470da6c284d16647cee2230522a85b0818f6d | vUSDT  | 10,000,000    | USDT             | 257,160  | 156,250  | $257,160       | $156,250       | $100,910       |
| 0x0b09Efc9458c00354414D2A560aA6EDa19490169 | vLINK  | 230,000       | LINK             | 4,730    | 2,850    | $83,354        | $50,239        | $33,115        |
| 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C | vETH   | 11,000        | ETH              | 233      | 136      | $875,958       | $511,532       | $364,426       |
| 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C | vBUSD  | 9,000,000     | BUSD             | 206,750  | 0        | $206,750       | $0             | $206,750       |
| 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122 | vBNB   | 7,000         | BNB              | 184      | 127      | $143,901       | $99,936        | $43,965        |
| **TOTAL**                                 |        |               |                  |          |          | **$3,196,540** | **$1,765,365** | **$1,431,175** |

## C. Info on attempts to recover funds

**All functions tested fail with error**: `BEP20: transfer amount exceeds allowance`

Tested functions:

- `deposit()`
- `withdraw()`
- `farm()`
- `earn()`
- `deleverageOnce()`
- `deleverageUntilNotOverLevered()`
- `rebalance()`

## D. Strategy Contract Architecture

**Core Operations**:

- Takes user deposits, borrows against them with leverage, claims Venus rewards, compounds everything back into positions
- `deposit()` → `_farm()` → `_leverage()` → loops `_supply()` + `_borrow()` for 3x leverage
- `withdraw()` → `_deleverage()` → `_removeSupply()` + `_repayBorrow()` to unwind positions
- `earn()` → `claimVenus()` → `swapExactTokensForTokens()` → compounds rewards
- `updateBalance()` → tracks `supplyBal`/`borrowBal` via `balanceOfUnderlying()` + `borrowBalanceCurrent()`
- `deleverageOnce()` + `deleverageUntilNotOverLevered()` for risk management
- `rebalance()` → adjusts `borrowRate`/`borrowDepth` strategy parameters

## E. RakeFarm LP Pool Analysis

### Table 2: Complete Daily Trading and Liquidity Data




| Date | BNB Price | RAKE Price | Liq Add $ | Liq Rem $ | Swapped RAKE $ | Swapped WBNB $ | Dev Fee | Venus Rewards to stuck Vaults | Cumulative Rewards | Assets Stuck  |
|------|-------|--------|-----------|-----------|-------------|-------------|---------|---------------|-------------|--------------|
| Feb 25 | $280 | $125K | $332M | $143M | $2.73B | $41K | 32 RAKE ($4.0M) | 32 RAKE ($4.0M) | $4.0M | $1.56M |
| Feb 26 | $260 | $26K | $271M | $89.5M | $2.17B | $227K | 32 RAKE ($827K) | 32 RAKE ($826K) | $4.8M | $1.52M |
| Feb 27 | $270 | $15.9K | $107M | $72.9M | $522M | $215K | 32 RAKE ($507K) | 32 RAKE ($506K) | $5.3M | $1.49M |
| Feb 28 | $280 | $3.9K | $5.7M | $5.3M | $35.3M | $124K | 32 RAKE ($125K) | 32 RAKE ($125K) | $5.4M | $1.46M |
| Mar 01 | $280 | $3.5K | $11.7M | $4.9M | $71.3M | $314K | 32 RAKE ($111K) | 32 RAKE ($111K) | $5.5M | $1.46M |
| Mar 02 | $280 | $3.5K | $5.9M | $2.7M | $28.2M | $152K | 32 RAKE ($111K) | 32 RAKE ($111K) | $5.7M | $1.50M |
| Mar 03 | $280 | $3.5K | $7.3M | $2.4M | $32.6M | $180K | 32 RAKE ($111K) | 32 RAKE ($111K) | $5.8M | $1.54M |
| Mar 04 | $280 | $3K | $4.6M | $9.8M | $39.1M | $318K | 32 RAKE ($95K) | 32 RAKE ($95K) | $5.9M | $1.51M |
| Mar 05 | $280 | $1.8K | $1.6M | $714K | $8.7M | $156K | 32 RAKE ($56K) | 19 RAKE ($33K) | $5.9M | $1.48M |
| Mar 06 | $280 | $1.5K | $561K | $2.8M | $9.1M | $259K | 32 RAKE ($48K) | 19 RAKE ($28K) | $5.9M | $1.46M |
| Mar 07 | $280 | $1.4K | $290K | $38K | $2.5M | $90K | 32 RAKE ($45K) | 19 RAKE ($26K) | $6.0M | $1.44M |

---

## G. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
