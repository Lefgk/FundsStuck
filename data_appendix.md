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

- **Creation Block Number**: 5181862
- **Total Pools**: 42
- **Total Allocation Points**: 8,539
- **AUTO Per Block**: 8,000,000,000,000,000
- **Owner**: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`

## F. Historical Protocol Performance (2021-2022)

### G. Timeline Summary

| Date         | BNB Price | RAKE/BNB | RAKE Price | Total Stuck Assets |
| ------------ | --------- | -------- | ---------- | ------------------ |
| Feb 25, 2021 | $280      | ~447     | $125,160   | $1,557,670         |
| Mar 07, 2021 | $280      | ~5.0     | $1,400     | $1,435,175         |
| Dec 31, 2021 | $530      | ~0.1     | $53        | $1,480,000         |
| Feb 28, 2022 | $360      | ~7.86    | $2,830     | $1,360,000         |

## Extended version :

| Date         | BNB<br/>Price | RAKE/<br/>BNB | RAKE<br/>Price | WBNB<br/>Balance | Available<br/>Liquidity BNB | Available<br/>Liquidity USD | Dev Fee<br/>Earned      | Venus<br/>Rewards       | Cumulative<br/>Rewards | Total Stuck<br/>Assets |
| ------------ | ------------- | ------------- | -------------- | ---------------- | --------------------------- | --------------------------- | ----------------------- | ----------------------- | ---------------------- | ---------------------- |
| Feb 25, 2021 | $280          | ~447          | $125,160       | 1,000            | 448,000                     | $125,440,000                | 31.80 RAKE ($3,980,088) | 31.78 RAKE ($3,977,509) | $7,957,597             | $1,557,670             |
| Feb 26, 2021 | $260          | ~100          | $26,000        | 6,500            | 650,000                     | $169,000,000                | 31.80 RAKE ($826,800)   | 31.78 RAKE ($826,280)   | $8,810,677             | $1,520,000             |
| Feb 27, 2021 | $270          | ~59           | $15,930        | 5,800            | 342,200                     | $92,394,000                 | 31.80 RAKE ($506,574)   | 31.78 RAKE ($506,064)   | $9,823,315             | $1,490,000             |
| Feb 28, 2021 | $280          | ~14           | $3,920         | 4,200            | 58,800                      | $16,464,000                 | 31.80 RAKE ($124,656)   | 31.78 RAKE ($124,578)   | $10,072,549            | $1,460,000             |
| Mar 01, 2021 | $280          | ~12.5         | $3,500         | 6,200            | 77,500                      | $21,700,000                 | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $10,295,079            | $1,458,331             |
| Mar 02, 2021 | $280          | ~12.5         | $3,500         | 6,800            | 85,000                      | $23,800,000                 | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $10,517,609            | $1,497,757             |
| Mar 03, 2021 | $280          | ~12.5         | $3,500         | 8,500            | 106,250                     | $29,750,000                 | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $10,740,139            | $1,535,008             |
| Mar 04, 2021 | $280          | ~10.7         | $3,000         | 5,000            | 53,500                      | $14,980,000                 | 31.80 RAKE ($95,400)    | 31.78 RAKE ($95,340)    | $10,930,879            | $1,508,091             |
| Mar 05, 2021 | $280          | ~6.25         | $1,750         | 5,500            | 34,375                      | $9,625,000                  | 31.80 RAKE ($55,650)    | 18.62 RAKE ($32,585)    | $11,019,114            | $1,478,607             |
| Mar 06, 2021 | $280          | ~5.36         | $1,500         | 3,000            | 16,080                      | $4,502,400                  | 31.80 RAKE ($47,700)    | 18.62 RAKE ($27,930)    | $11,094,744            | $1,456,891             |
| Mar 07, 2021 | $280          | ~5.0          | $1,400         | 3,200            | 16,000                      | $4,480,000                  | 31.80 RAKE ($44,520)    | 18.62 RAKE ($26,068)    | $11,165,332            | $1,435,175             |
| Mar 08, 2021 | $280          | ~4.3          | $1,200         | 2,800            | 12,040                      | $3,371,200                  | Period Total            | Period Total            | $11,195,000            | $1,420,000             |
| Mar 09, 2021 | $280          | ~3.9          | $1,100         | 2,600            | 10,140                      | $2,839,200                  | Period Total            | Period Total            | $11,220,000            | $1,410,000             |
| Mar 10, 2021 | $280          | ~3.6          | $1,000         | 2,200            | 7,920                       | $2,217,600                  | Period Total            | Period Total            | $11,240,000            | $1,400,000             |
| Mar 31, 2021 | $280          | ~2.5          | $700           | 900              | 2,250                       | $630,000                    | Period Total            | Period Total            | $11,280,000            | $1,420,000             |
| Apr 30, 2021 | $500          | ~1.2          | $600           | 800              | 960                         | $480,000                    | Period Total            | Period Total            | $11,380,000            | $1,650,000             |
| May 31, 2021 | $680          | ~0.9          | $612           | 750              | 675                         | $459,000                    | Period Total            | Period Total            | $11,480,000            | $1,850,000             |
| Jun 30, 2021 | $360          | ~1.0          | $360           | 720              | 720                         | $259,200                    | Period Total            | Period Total            | $11,540,000            | $1,400,000             |
| Jul 31, 2021 | $320          | ~0.75         | $240           | 700              | 525                         | $168,000                    | Period Total            | Period Total            | $11,580,000            | $1,350,000             |
| Aug 31, 2021 | $480          | ~0.5          | $240           | 680              | 340                         | $163,200                    | Period Total            | Period Total            | $11,620,000            | $1,500,000             |
| Sep 30, 2021 | $380          | ~0.44         | $168           | 650              | 286                         | $108,680                    | Period Total            | Period Total            | $11,650,000            | $1,420,000             |
| Oct 31, 2021 | $500          | ~0.33         | $165           | 620              | 205                         | $102,500                    | Period Total            | Period Total            | $11,680,000            | $1,550,000             |
| Nov 30, 2021 | $650          | ~0.15         | $97.5          | 600              | 90                          | $58,500                     | Period Total            | Period Total            | $11,700,000            | $1,700,000             |
| Dec 31, 2021 | $530          | ~0.1          | $53            | 62.15            | 6.2                         | $3,286                      | Period Total            | Period Total            | $11,710,000            | $1,480,000             |
| Jan 05, 2022 | $350          | ~2.5          | $875           | 180              | 450                         | $157,500                    | Period Total            | Period Total            | $11,715,000            | $1,350,000             |
| Jan 15, 2022 | $330          | ~15           | $4,950         | 220              | 3,300                       | $1,089,000                  | Period Total            | Period Total            | $11,730,000            | $1,320,000             |
| Jan 25, 2022 | $340          | ~65           | $22,100        | 300              | 19,500                      | $6,630,000                  | Period Total            | Period Total            | $11,800,000            | $1,340,000             |
| Jan 31, 2022 | $330          | ~20           | $6,600         | 250              | 5,000                       | $1,650,000                  | Period Total            | Period Total            | $11,850,000            | $1,320,000             |
| Feb 15, 2022 | $350          | ~12           | $4,200         | 180              | 2,160                       | $756,000                    | Period Total            | Period Total            | $11,880,000            | $1,350,000             |
| Feb 28, 2022 | $360          | ~7.86         | $2,830         | 150              | 1,179                       | $424,440                    | Period Total            | Period Total            | $11,900,000            | $1,360,000             |

## H. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
