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



## Extended version :# RAKE Protocol Timeline Analysis (Feb 25 - Mar 7, 2021)

| Date         | BNB Price | RAKE Price | WBNB Balance | RAKE Balance | Available Liquidity USD | Dev Fee Earned      | Venus Rewards       | Cumulative Rewards | Total Stuck Assets |
|--------------|-----------|------------|--------------|--------------|-------------------------|---------------------|---------------------|--------------------|--------------------|
| Feb 25, 2021 | $280      | $125,160   | 1025         | 10.93        | $287,000                | 31.80 RAKE ($3,980,088) | 31.78 RAKE ($3,977,509) | $3,977,509         | $1,557,670         |
| Feb 26, 2021 | $260      | $26,000    | 6501         | 95           | $1,690,260              | 31.80 RAKE ($826,800)   | 31.78 RAKE ($826,280)   | $4,803,789         | $1,520,000         |
| Feb 27, 2021 | $270      | $15,930    | 5604         | 209          | $1,513,080              | 31.80 RAKE ($506,574)   | 31.78 RAKE ($506,064)   | $5,309,853         | $1,490,000         |
| Feb 28, 2021 | $280      | $3,920     | 4159         | 309          | $1,164,520              | 31.80 RAKE ($124,656)   | 31.78 RAKE ($124,578)   | $5,434,431         | $1,460,000         |
| Mar 01, 2021 | $280      | $3,500     | 6197         | 417          | $1,735,160              | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,545,661         | $1,458,331         |
| Mar 02, 2021 | $280      | $3,500     | 6736         | 508          | $1,886,080              | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,656,891         | $1,497,757         |
| Mar 03, 2021 | $280      | $3,500     | 8642         | 553          | $2,419,760              | 31.80 RAKE ($111,300)   | 31.78 RAKE ($111,230)   | $5,768,121         | $1,535,008         |
| Mar 04, 2021 | $280      | $3,000     | 4803         | 596          | $1,344,840              | 31.80 RAKE ($95,400)    | 31.78 RAKE ($95,340)    | $5,863,461         | $1,508,091         |
| Mar 05, 2021 | $280      | $1,750     | 5595         | 628          | $1,566,600              | 31.80 RAKE ($55,650)    | 18.62 RAKE ($32,585)    | $5,896,046         | $1,478,607         |
| Mar 06, 2021 | $280      | $1,500     | 2989         | 560          | $836,920                | 31.80 RAKE ($47,700)    | 18.62 RAKE ($27,930)    | $5,923,976         | $1,456,891         |
| Mar 07, 2021 | $280      | $1,400     | 3146         | 613          | $880,880                | 31.80 RAKE ($44,520)    | 18.62 RAKE ($26,068)    | $5,950,044         | $1,435,175         |


---

# RAKE Protocol Timeline Analysis - Complete Trading Data (Feb 25 - Mar 7, 2021)

| Date         | BNB Price | RAKE Price | WBNB Balance | RAKE Balance | Available Liquidity USD | Liquidity Added (RAKE) | Liquidity Added ($) | Liquidity Removed (RAKE) | Liquidity Removed ($) | Swapped RAKE | Swapped RAKE ($) | Swapped WBNB | Swapped WBNB ($) | Net Liquidity Change | Total Stuck Assets |
|--------------|-----------|------------|--------------|--------------|-------------------------|------------------------|--------------------|--------------------------|-----------------------|--------------|------------------|--------------|------------------|---------------------|-------------------|
| Feb 25, 2021 | $280      | $125,160   | 1025         | 10.93        | $287,000                | 2,654.98              | $332,331,149       | 1,145.10                 | $143,391,504          | 21,836.95    | $2,733,344,502   | 146.80       | $41,104          | +1,509.89 RAKE      | $1,557,670        |
| Feb 26, 2021 | $260      | $26,000    | 6501         | 95           | $1,690,260              | 10,428.20             | $271,132,800       | 3,442.98                 | $89,517,480           | 83,307.43    | $2,165,993,180   | 872.38       | $226,819         | +6,985.22 RAKE      | $1,520,000        |
| Feb 27, 2021 | $270      | $15,930    | 5604         | 209          | $1,513,080              | 6,727.03              | $107,147,507       | 4,577.04                 | $72,892,063           | 32,793.07    | $522,354,425     | 794.67       | $214,661         | +2,149.99 RAKE      | $1,490,000        |
| Feb 28, 2021 | $280      | $3,920     | 4159         | 309          | $1,164,520              | 1,441.62              | $5,651,151          | 1,360.95                 | $5,335,124            | 9,009.98     | $35,319,128      | 442.49       | $123,897         | +80.67 RAKE         | $1,460,000        |
| Mar 01, 2021 | $280      | $3,500     | 6197         | 417          | $1,735,160              | 3,328.80              | $11,650,800        | 1,387.22                 | $4,855,270            | 20,373.94    | $71,308,790      | 1,121.66     | $314,065         | +1,941.58 RAKE      | $1,458,331        |
| Mar 02, 2021 | $280      | $3,500     | 6736         | 508          | $1,886,080              | 1,693.61              | $5,927,635         | 761.18                   | $2,664,130            | 8,053.10     | $28,185,850      | 543.42       | $152,158         | +932.43 RAKE        | $1,497,757        |
| Mar 03, 2021 | $280      | $3,500     | 8642         | 553          | $2,419,760              | 2,079.41              | $7,277,935         | 690.44                   | $2,416,540            | 9,307.58     | $32,576,530      | 641.49       | $179,617         | +1,388.96 RAKE      | $1,535,008        |
| Mar 04, 2021 | $280      | $3,000     | 4803         | 596          | $1,344,840              | 1,528.27              | $4,584,810         | 3,264.67                 | $9,794,010            | 13,037.43    | $39,112,290      | 1,134.79     | $317,741         | -1,736.40 RAKE      | $1,508,091        |
| Mar 05, 2021 | $280      | $1,750     | 5595         | 628          | $1,566,600              | 896.88                | $1,569,540         | 407.85                   | $713,738              | 4,967.02     | $8,692,289       | 557.57       | $156,120         | +489.03 RAKE        | $1,478,607        |
| Mar 06, 2021 | $280      | $1,500     | 2989         | 560          | $836,920                | 373.67                | $560,505           | 1,834.29                 | $2,751,435            | 6,057.11     | $9,085,665       | 923.33       | $258,532         | -1,460.62 RAKE      | $1,456,891        |
| Mar 07, 2021 | $280      | $1,400     | 3146         | 613          | $880,880                | 206.99                | $289,786           | 27.14                    | $38,002               | 1,796.00     | $2,514,400       | 319.92       | $89,578          | +179.85 RAKE        | $1,435,175        |

## Key Trading Insights

**Price Volatility**: RAKE token experienced extreme volatility, dropping from $125,160 on Feb 25 to $1,400 on Mar 7 - a 98.9% decline in just 10 days.

**Trading Activity**: 
- Peak trading volume occurred on Feb 26: 83,307 RAKE ($2.17B) and 872 WBNB ($227K)
- Trading activity declined significantly from Mar 4 onwards as price collapsed
- Consistent buyer preference: 66-80% of swaps were WBNB→RAKE across most days

**Liquidity Dynamics**:
- Major liquidity additions on Feb 25-26: 13,083 RAKE worth $603M combined
- Significant liquidity removal events on Mar 4 and Mar 6, indicating panic exits
- Net liquidity remained positive on most days despite price decline

**Market Behavior**:
- Available pool liquidity peaked at $2.4M on Mar 3
- Large outflows coincided with major price drops (Mar 4: -1,736 RAKE, Mar 6: -1,460 RAKE)
- Despite massive trading volumes, stuck assets remained stable around $1.4-1.5M

**Technical Issue**: The AutoFarm strategy contract bug prevented recovery of $1.4M+ in assets across multiple Venus Protocol positions, regardless of market conditions.

---

# RAKE Protocol Trading Activity Analysis (Feb 25 - Mar 7, 2021)

| Date         | **SWAPS** |           |           |             |             | **LIQUIDITY** |            |              |              |          |          |
|--------------|-----------|-----------|-----------|-------------|-------------|---------------|------------|--------------|--------------|----------|----------|
|              | Total     | RAKE→WBNB | WBNB→RAKE | Volume RAKE | Volume WBNB | RAKE Added   | WBNB Added | RAKE Removed | WBNB Removed | Net RAKE | Net WBNB |
| Feb 25, 2021 | 4,300     | 1,447     | 2,853     | 21,836.95   | 146.80      | 2,654.98     | 16.05      | 1,145.10     | 7.84         | +1,509.88| +8.21    |
| Feb 26, 2021 | 8,657     | 2,436     | 6,221     | 83,307.43   | 872.38      | 10,428.20    | 108.11     | 3,442.98     | 38.72        | +6,985.22| +69.39   |
| Feb 27, 2021 | 4,246     | 936       | 3,310     | 32,793.07   | 794.67      | 6,727.03     | 161.26     | 4,577.04     | 115.15       | +2,149.99| +46.11   |
| Feb 28, 2021 | 3,412     | 1,409     | 2,003     | 9,009.98    | 442.49      | 1,441.62     | 68.78      | 1,360.95     | 67.93        | +80.67   | +0.85    |
| Mar 01, 2021 | 3,829     | 1,649     | 2,180     | 20,373.94   | 1,121.66    | 3,328.80     | 193.84     | 1,387.22     | 79.52        | +1,941.58| +114.32  |
| Mar 02, 2021 | 2,997     | 1,389     | 1,608     | 8,053.10    | 543.42      | 1,693.61     | 114.66     | 761.18       | 52.76        | +932.43  | +61.90   |
| Mar 03, 2021 | 2,148     | 608       | 1,540     | 9,307.58    | 641.49      | 2,079.41     | 142.67     | 690.44       | 50.17        | +1,388.96| +92.50   |
| Mar 04, 2021 | 2,366     | 684       | 1,682     | 13,037.43   | 1,134.79    | 1,528.27     | 134.10     | 3,264.67     | 282.26       | -1,736.40| -148.16  |
| Mar 05, 2021 | 1,534     | 314       | 1,220     | 4,967.02    | 557.57      | 896.88       | 101.17     | 407.85       | 47.81        | +489.03  | +53.37   |
| Mar 06, 2021 | 1,534     | 302       | 1,232     | 6,057.11    | 923.33      | 373.67       | 53.91      | 1,834.29     | 293.97       | -1,460.62| -240.06  |
| Mar 07, 2021 | 547       | 109       | 438       | 1,796.00    | 319.92      | 206.99       | 37.24      | 27.14        | 4.79         | +179.85  | +32.45   |

## G. Data Sources

- RAKE/BNB price: [CoinMarketCap DEX](https://dex.coinmarketcap.com/token/bsc/0xbda8d53fe0f164915b46cd2ecffd94254b6086a2/)
- BNB/USD price: [CoinMarketCap](https://coinmarketcap.com/currencies/bnb/)
- RAKE/WBNB liquidity: [BSCScan Token Analytics](https://bscscan.com/token/0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c?a=0x1cb667fe903dbdcbd27d8b35e82fbcef4ca0f621#tokenAnalytics)
- Pool allocation changes: [BSCScan Advanced Filter](https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet)
