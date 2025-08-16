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

### F. Timeline Summary

| Date         | BNB Price | RAKE/BNB | RAKE Price | Total Stuck Assets |
| ------------ | --------- | -------- | ---------- | ------------------ |
| Feb 25, 2021 | $280      | ~447     | $125,160   | $1,557,670         |
| Mar 07, 2021 | $280      | ~5.0     | $1,400     | $1,435,175         |
| Dec 31, 2021 | $530      | ~0.1     | $53        | $1,480,000         |
| Feb 28, 2022 | $360      | ~7.86    | $2,830     | $1,360,000         |

## Extended version :

| Date         | BNB<br/>Price | RAKE/<br/>BNB | RAKE<br/>Price | WBNB<br/>Balance | Available<br/>Liquidity RAKE | Available<br/>Liquidity USD | Dev Fee<br/>Earned      | Venus<br/>Rewards       | Cumulative<br/>Rewards | Total Stuck<br/>Assets |
| ------------ | ------------- | ------------- | -------------- | ---------------- | ---------------------------- | --------------------------- | ----------------------- | ----------------------- | ---------------------- | ---------------------- |
| Feb 25, 2021 | $280          | ~447          | $125,160       | 1,000            | 448,000                      | $280,000                    | 31.80 RAKE ($3,980,088) | 31.78 RAKE ($3,977,509) | $3,977,509             | $1,557,670             |
| Feb 26, 2021 | $260          | ~100          | $26,000        | 6,500            | 650,000                      | $1,690,000                  | 31.80 RAKE ($826,800)   | 31.78 RAKE ($826,280)   | $4,803,789             | $1,520,000             |
| Feb 27, 2021 | $270          | ~59           | $15,930        | 5,800            | 342,200                      | $1,566,000                  | 31.80 RAKE ($506,574)   | 31.78 RAKE ($506,064)   | $5,309,853             | $1,490,000             |
| Feb 28, 2021 | $280          | ~14           | $3,920         | 4,200            | 58,800                       | $1,176,000                  | 31.80 RAKE ($124,656)   | 31.78 RAKE ($124,578)   | $5,434,431             | $1,460,000             |
| Mar 01, 2021 | $280          | ~12.5         | $3,500         | 6,200            | 77,500                       | $1,736,000                  | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,545,661             | $1,458,331             |
| Mar 02, 2021 | $280          | ~12.5         | $3,500         | 6,800            | 85,000                       | $1,904,000                  | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,656,891             | $1,497,757             |
| Mar 03, 2021 | $280          | ~12.5         | $3,500         | 8,500            | 106,250                      | $2,380,000                  | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,768,121             | $1,535,008             |
| Mar 04, 2021 | $280          | ~10.7         | $3,000         | 5,000            | 53,500                       | $1,400,000                  | 31.80 RAKE ($95,400)    | 31.78 RAKE ($95,340)    | $5,863,461             | $1,508,091             |
| Mar 05, 2021 | $280          | ~6.25         | $1,750         | 5,500            | 34,375                       | $1,540,000                  | 31.80 RAKE ($55,650)    | 18.62 RAKE ($32,585)    | $5,896,046             | $1,478,607             |
| Mar 06, 2021 | $280          | ~5.36         | $1,500         | 3,000            | 16,080                       | $840,000                    | 31.80 RAKE ($47,700)    | 18.62 RAKE ($27,930)    | $5,923,976             | $1,456,891             |
| Mar 07, 2021 | $280          | ~5.0          | $1,400         | 3,200            | 16,000                       | $896,000                    | 31.80 RAKE ($44,520)    | 18.62 RAKE ($26,068)    | $5,950,044             | $1,435,175             |
| Mar 08, 2021 | $280          | ~4.3          | $1,200         | 2,800            | 12,040                       | $784,000                    | 31.80 RAKE ($38,160)    | 18.62 RAKE ($22,344)    | $5,972,388             | $1,420,000             |
| Mar 09, 2021 | $280          | ~3.9          | $1,100         | 2,600            | 10,140                       | $728,000                    | 31.80 RAKE ($34,980)    | 18.62 RAKE ($20,482)    | $5,992,870             | $1,410,000             |
| Mar 10, 2021 | $280          | ~3.6          | $1,000         | 2,200            | 7,920                        | $616,000                    | 31.80 RAKE ($31,800)    | 18.62 RAKE ($18,620)    | $6,011,490             | $1,400,000             |
| Mar 31, 2021 | $280          | ~2.5          | $700           | 900              | 2,250                        | $252,000                    | 31.80 RAKE ($467,460)   | 18.62 RAKE ($273,714)   | $6,285,204             | $1,420,000             |
| Apr 30, 2021 | $500          | ~1.2          | $600           | 800              | 960                          | $400,000                    | 31.80 RAKE ($572,400)   | 18.62 RAKE ($335,160)   | $6,620,364             | $1,650,000             |
| May 31, 2021 | $680          | ~0.9          | $612           | 750              | 675                          | $510,000                    | 31.80 RAKE ($603,324)   | 18.62 RAKE ($353,324)   | $6,973,688             | $1,850,000             |
| Jun 30, 2021 | $360          | ~1.0          | $360           | 720              | 720                          | $259,200                    | 31.80 RAKE ($343,440)   | 18.62 RAKE ($201,090)   | $7,174,778             | $1,400,000             |
| Jul 31, 2021 | $320          | ~0.75         | $240           | 700              | 525                          | $224,000                    | 31.80 RAKE ($236,592)   | 18.62 RAKE ($138,538)   | $7,313,316             | $1,350,000             |
| Aug 31, 2021 | $480          | ~0.5          | $240           | 680              | 340                          | $326,400                    | 31.80 RAKE ($236,592)   | 18.62 RAKE ($138,538)   | $7,451,854             | $1,500,000             |
| Sep 30, 2021 | $380          | ~0.44         | $168           | 650              | 286                          | $247,000                    | 31.80 RAKE ($160,272)   | 18.62 RAKE ($93,840)    | $7,545,694             | $1,420,000             |
| Oct 31, 2021 | $500          | ~0.33         | $165           | 620              | 205                          | $310,000                    | 31.80 RAKE ($162,657)   | 18.62 RAKE ($95,233)    | $7,640,927             | $1,550,000             |
| Nov 30, 2021 | $650          | ~0.15         | $97.5          | 600              | 90                           | $390,000                    | 31.80 RAKE ($93,030)    | 18.62 RAKE ($54,450)    | $7,695,377             | $1,700,000             |
| Dec 31, 2021 | $530          | ~0.1          | $53            | 62.15            | 6.2                          | $32,940                     | 31.80 RAKE ($52,235)    | 18.62 RAKE ($30,587)    | $7,725,964             | $1,480,000             |
| Jan 05, 2022 | $350          | ~2.5          | $875           | 180              | 450                          | $63,000                     | 31.80 RAKE ($139,125)   | 18.62 RAKE ($81,463)    | $7,807,427             | $1,350,000             |
| Jan 15, 2022 | $330          | ~15           | $4,950         | 220              | 3,300                        | $72,600                     | 31.80 RAKE ($1,574,100) | 18.62 RAKE ($921,690)   | $8,729,117             | $1,320,000             |
| Jan 25, 2022 | $340          | ~65           | $22,100        | 300              | 19,500                       | $102,000                    | 31.80 RAKE ($7,027,800) | 18.62 RAKE ($4,115,020) | $12,844,137            | $1,340,000             |
| Jan 31, 2022 | $330          | ~20           | $6,600         | 250              | 5,000                        | $82,500                     | 31.80 RAKE ($1,259,280) | 18.62 RAKE ($737,352)   | $13,581,489            | $1,320,000             |
| Feb 15, 2022 | $350          | ~12           | $4,200         | 180              | 2,160                        | $63,000                     | 31.80 RAKE ($1,869,840) | 18.62 RAKE ($1,094,856) | $14,676,345            | $1,350,000             |
| Feb 28, 2022 | $360          | ~7.86         | $2,830         | 150              | 1,179                        | $54,000                     | 31.80 RAKE ($1,169,922) | 18.62 RAKE ($684,884)   | $15,361,229            | $1,360,000             |

## G. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
