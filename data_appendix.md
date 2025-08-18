# Data Appendix - RAKE Protocol Stuck Funds Analysis

> **📋 Related Documents:** [Recovery Proposal](./recovery_proposal.md) | [Main README](./README.md)

## Quick Reference - Table Index

- **[Table 1: Venus Protocol Contract Positions](#table-1-venus-protocol-contract-positions)** - Detailed breakdown of all affected contracts
- **[Table 2: Key Timeline Milestones](#table-2-key-timeline-milestones)** - Major events and price movements
- **[Table 3: Daily Protocol Metrics and Rewards](#table-3-daily-protocol-metrics-and-rewards)** - Daily tracking data
- **[Table 4: Complete Daily Trading and Liquidity Data](#table-4-complete-daily-trading-and-liquidity-data)** - Full trading analysis

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

### Table 3: Complete Daily Trading and Liquidity Data


| Date         | BNB Price | RAKE Price | Liquidity Added ($) | Liquidity Removed ($) | Swapped RAKE ($) | Swapped WBNB ($) | Dev Fee Earned       | Venus Rewards to stuck Vaults        | Cumulative Rewards | Total Stuck Assets |
| ------------ | --------- | ---------- | ------------------- | --------------------- | ---------------- | ---------------- | -------------------- | -------------------- | ------------------ | ------------------ |
| Feb 25, 2021 | $280      | $125,160   | $332,331,149        | $143,391,504          | $2,733,344,502   | $41,104          | 32 RAKE ($3,980,088) | 32 RAKE ($3,977,509) | $3,977,509         | $1,557,670         |
| Feb 26, 2021 | $260      | $26,000    | $271,132,800        | $89,517,480           | $2,165,993,180   | $226,819         | 32 RAKE ($826,800)   | 32 RAKE ($826,280)   | $4,803,789         | $1,520,000         |
| Feb 27, 2021 | $270      | $15,930    | $107,147,507        | $72,892,063           | $522,354,425     | $214,661         | 32 RAKE ($506,574)   | 32 RAKE ($506,064)   | $5,309,853         | $1,490,000         |
| Feb 28, 2021 | $280      | $3,920     | $5,651,151          | $5,335,124            | $35,319,128      | $123,897         | 32 RAKE ($124,656)   | 32 RAKE ($124,578)   | $5,434,431         | $1,460,000         |
| Mar 01, 2021 | $280      | $3,500     | $11,650,800         | $4,855,270            | $71,308,790      | $314,065         | 32 RAKE ($111,300)   | 32 RAKE ($111,230)   | $5,545,661         | $1,458,331         |
| Mar 02, 2021 | $280      | $3,500     | $5,927,635          | $2,664,130            | $28,185,850      | $152,158         | 32 RAKE ($111,300)   | 32 RAKE ($111,230)   | $5,656,891         | $1,497,757         |
| Mar 03, 2021 | $280      | $3,500     | $7,277,935          | $2,416,540            | $32,576,530      | $179,617         | 32 RAKE ($111,300)   | 32 RAKE ($111,230)   | $5,768,121         | $1,535,008         |
| Mar 04, 2021 | $280      | $3,000     | $4,584,810          | $9,794,010            | $39,112,290      | $317,741         | 32 RAKE ($95,400)    | 32 RAKE ($95,340)    | $5,863,461         | $1,508,091         |
| Mar 05, 2021 | $280      | $1,750     | $1,569,540          | $713,738              | $8,692,289       | $156,120         | 32 RAKE ($55,650)    | 19 RAKE ($32,585)    | $5,896,046         | $1,478,607         |
| Mar 06, 2021 | $280      | $1,500     | $560,505            | $2,751,435            | $9,085,665       | $258,532         | 32 RAKE ($47,700)    | 19 RAKE ($27,930)    | $5,923,976         | $1,456,891         |
| Mar 07, 2021 | $280      | $1,400     | $289,786            | $38,002               | $2,514,400       | $89,578          | 32 RAKE ($44,520)    | 19 RAKE ($26,068)    | $5,950,044         | $1,435,175         |

---

## G. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
