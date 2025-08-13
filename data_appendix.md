# Data Appendix - RAKE Protocol Stuck Funds Analysis

## A. Technical Root Cause

**Bug Description**: AutoFarm strategy contract inheritance flaw

- `pause()` function correctly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- `unpause()` function incorrectly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- **Correct implementation should be**: `IERC20(wantAddress).safeApprove(vTokenAddress, uint256(-1))`

**Impact**: Permanently locks approval at zero, preventing all Venus Protocol interactions.

## B. Affected Contract Details

**Primary Token**: RAKE Token Address `0xbDa8D53fe0F164915b46cd2EcfFD94254b6086a2`

| Contract Address                           | vToken | Total vTokens | Underlying Asset | Supplied | Borrowed | Supplied USD   | Borrowed USD   | Net USD        |
| ------------------------------------------ | ------ | ------------- | ---------------- | -------- | -------- | -------------- | -------------- | -------------- |
| 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 | vBTC   | 450,000       | BTCB             | 9.217613 | 5.24     | $1,084,372     | $616,279       | $468,093       |
| 0x568254EAAC1476faf9A908e577faa3Ab96029801 | vDOT   | 55,000        | DOT              | 1,250    | 616      | $4,825         | $2,379         | $2,446         |
| 0x7485704d8b6cebd1411f5aafeb063e6f2816069a | vUSDC  | 21,000,000    | USDC             | 544,220  | 328,750  | $544,220       | $328,750       | $215,470       |
| 0xc6f470da6c284d16647cee2230522a85b0818f6d | vUSDT  | 10,000,000    | USDT             | 257,160  | 156,250  | $257,160       | $156,250       | $100,910       |
| 0x0b09Efc9458c00354414D2A560aA6EDa19490169 | vLINK  | 230,000       | LINK             | 4,730    | 2,850    | $83,354        | $50,239        | $33,115        |
| 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C | vETH   | 11,000        | ETH              | 232.68   | 135.84   | $875,958       | $511,532       | $364,426       |
| 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C | vBUSD  | 9,000,000     | BUSD             | 206,750  | 0        | $206,750       | $0             | $206,750       |
| 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122 | vBNB   | 7,000         | BNB              | 183.6    | 127.46   | $143,901       | $99,936        | $43,965        |
| **TOTALS**                                 |        |               |                  |          |          | **$3,196,540** | **$1,765,365** | **$1,431,175** |

## C. Failed Recovery Functions

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

**Contract Address**: `0x7f7Bf15B9c68D23339C31652C8e860492991760d`  
**Block Number**: 1990376681

### Farm Overview

- **Total Pools**: 42
- **Total Allocation Points**: 8,539
- **AUTO Per Block**: 8,000,000,000,000,000
- **Start Block**: 5,195,791
- **Owner**: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`

### Key Pool Allocations

| PID | Token Symbol | LP Pair      | Allocation Points | Allocation % | Strategy Address                           |
| --- | ------------ | ------------ | ----------------- | ------------ | ------------------------------------------ |
| 0   | Cake-LP      | WBNB / RAKE  | 1,500             | 17%          | 0xaE3Cf8A82F9441E38b92a2575a9b73d444a93D75 |
| 1   | Cake-LP      | BR34P / RAKE | 1,200             | 14%          | 0x014c44f948EB082c52A8c6dC090bA53273A425A5 |
| 2   | Cake-LP      | BR34P / WBNB | 1,000             | 11%          | 0xDF635b131a9eb4B7CA4b50F21c3a4F722F9f3699 |
| 8   | WBNB         | WBNB         | 150               | 1%           | 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122 |
| 10  | BTCB         | BTCB         | 100               | 1%           | 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 |

## F. Historical Protocol Performance (2021-2022)

### Timeline Summary

| Date         | BNB Price | RAKE/BNB | RAKE Price | Total Stuck Assets |
| ------------ | --------- | -------- | ---------- | ------------------ |
| Feb 25, 2021 | $280      | ~447     | $125,160   | $1,557,670         |
| Mar 07, 2021 | $280      | ~5.0     | $1,400     | $1,435,175         |
| Dec 31, 2021 | $530      | ~0.1     | $53        | $1,480,000         |
| Feb 28, 2022 | $360      | ~7.86    | $2,830     | $1,360,000         |

### Major Asset Performance

**BTCB (10.40 BTCB)** - Largest single asset (34.0% of portfolio at peak)

- Feb 25, 2021: $530,505 (BTC @ $51,000)
- Oct 31, 2021: $634,400 (BTC @ $61,000) - Peak value
- Feb 28, 2022: $405,600 (BTC @ $39,000)

**ETH (256.64 ETH)** - Best performer

- Feb 25, 2021: $410,621 (ETH @ $1,600)
- Nov 30, 2021: $1,180,544 (ETH @ $4,600) - Peak value
- Feb 28, 2022: $744,256 (ETH @ $2,900)

## G. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
