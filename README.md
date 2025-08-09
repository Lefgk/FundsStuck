# Stuck Funds Analysis Report - RAKE Protocol

## Executive Summary

We had a project that was a fork of AutoFarm that now has **$1.43 million** of Vtokens stuck due to a approval bug inherited from AutoFarm's strategy contract. This bug permanently set approvals to 0 once the contract was paused. The contracts cannot perform any Venus lending operations due to zero token approvals.

## Root Cause Analysis

**Bug Origin**: 
- `pause()` correctly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)`
- `unpause()` incorrectly sets: `IERC20(wantAddress).safeApprove(vTokenAddress, 0)` 
- **Should be**: `IERC20(wantAddress).safeApprove(vTokenAddress, uint256(-1))`

This permanently locks approval at zero, preventing all Venus Protocol interactions.

## Strategy Contract Overview

Automated leveraged yield farming on Venus that:

- Takes user deposits, borrows against them with leverage, claims Venus rewards, compounds everything back into positions
- `deposit()` → `_farm()` → `_leverage()` → loops `_supply()` + `_borrow()` for 3x leverage
- `withdraw()` → `_deleverage()` → `_removeSupply()` + `_repayBorrow()` to unwind positions
- `earn()` → `claimVenus()` → `swapExactTokensForTokens()` → compounds rewards
- `updateBalance()` → tracks `supplyBal`/`borrowBal` via `balanceOfUnderlying()` + `borrowBalanceCurrent()`
- `deleverageOnce()` + `deleverageUntilNotOverLevered()` for risk management
- `rebalance()` → adjusts `borrowRate`/`borrowDepth` strategy parameters

## Affected Contracts

**RAKE Token Address**: `0xbDa8D53fe0F164915b46cd2EcfFD94254b6086a2`

| Contract Address | vToken | Total vTokens | Underlying Asset | Supplied | Borrowed | Supplied USD | Borrowed USD | Net USD |
|------------------|--------|---------------|------------------|----------|----------|--------------|--------------|---------|
| 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 | vBTC | 450,000 | BTCB | 9.217613 | 5.24 | $1,084,372 | $616,279 | $468,093 |
| 0x568254EAAC1476faf9A908e577faa3Ab96029801 | vDOT | 55,000 | DOT | 1,250 | 616 | $4,825 | $2,379 | $2,446 |
| 0x7485704d8b6cebd1411f5aafeb063e6f2816069a | vUSDC | 21,000,000 | USDC | 544,220 | 328,750 | $544,220 | $328,750 | $215,470 |
| 0xc6f470da6c284d16647cee2230522a85b0818f6d | vUSDT | 10,000,000 | USDT | 257,160 | 156,250 | $257,160 | $156,250 | $100,910 |
| 0x0b09Efc9458c00354414D2A560aA6EDa19490169 | vLINK | 230,000 | LINK | 4,730 | 2,850 | $83,354 | $50,239 | $33,115 |
| 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C | vETH | 11,000 | ETH | 232.68 | 135.84 | $875,958 | $511,532 | $364,426 |
| 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C | vBUSD | 9,000,000 | BUSD | 206,750 | 0 | $206,750 | $0 | $206,750 |
| 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122 | vBNB | 7,000 | BNB | 183.6 | 127.46 | $143,901 | $99,936 | $43,965 |
| **TOTALS** | | | | | | **$3,196,540** | **$1,765,365** | **$1,431,175** |

## Recovery Solutions Attempted

All lending related functions tested and confirmed to fail at the same point using foundry tests in forked environment.

**Functions Tested:**

- `deposit()`
- `withdraw()`
- `farm()`
- `earn()`
- `deleverageOnce()`
- `deleverageUntilNotOverLevered()`
- `rebalance()`

**Failure Point:** All transactions revert with error:
```
BEP20: transfer amount exceeds allowance
```

## RakeFarm LP Analysis

**Contract Address**: `0x7f7Bf15B9c68D23339C31652C8e860492991760d`  
**Block Number**: 1990376681

### Farm Overview

- **Total Pools**: 42
- **Total Allocation Points**: 8,539
- **AUTO Per Block**: 8,000,000,000,000,000
- **Start Block**: 5,195,791
- **Owner**: `0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07`
- **Current Block**: 1,990,376,681

### Pool Details

| PID | Stake Token | Token Symbol | LP Pair | Allocation Points | Allocation % | Strategy Address |
|-----|-------------|--------------|---------|-------------------|--------------|------------------|
| 0 | 0x1Cb667Fe903DBdcbd27D8B35E82FbCEF4Ca0f621 | Cake-LP | WBNB / RAKE | 1,500 | 17% | 0xaE3Cf8A82F9441E38b92a2575a9b73d444a93D75 |
| 1 | 0x889A24725Ce7E867FEB9ea5f1c96b0111c91F39D | Cake-LP | BR34P / RAKE | 1,200 | 14% | 0x014c44f948EB082c52A8c6dC090bA53273A425A5 |
| 2 | 0x70e882efa9beA28262d4873e65d5f65E9B2bABa6 | Cake-LP | BR34P / WBNB | 1,000 | 11% | 0xDF635b131a9eb4B7CA4b50F21c3a4F722F9f3699 |
| 3 | 0xA0718093baa3E7AAE054eED71F303A4ebc1C076f | Cake-LP | sBDO / BUSD | 5 | 0% | 0xF5737944b82A2C9235aFC140c627e66b4496C2e7 |
| 4 | 0xc5b0d73A7c0E4eaF66baBf7eE16A2096447f7aD6 | Cake-LP | BDO / BUSD | 5 | 0% | 0x3576F383Af96F59280bDEd50Dd18d7ecEdAE18D6 |
| 5 | 0x74690f829fec83ea424ee1F1654041b2491A7bE9 | Cake-LP | BDO / WBNB | 2 | 0% | 0xEEA269fe7EF8d7a7AE438A2D31A3B2089242b7D2 |
| 6 | 0x19e7cbECDD23A16DfA5573dF54d98F7CaAE03019 | Cake-LP | BUSD / EGG | 10 | 0% | 0xDE1DAa4BD680056b4F46Dd1602617eFbB29c3378 |
| 7 | 0xd1B59D11316E87C3a0A069E80F590BA35cD8D8D3 | Cake-LP | WBNB / EGG | 10 | 0% | 0x06aC46ad065eeaB1B4C08A8CE0F02649669c00df |
| 8 | 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c | WBNB | WBNB | 150 | 1% | 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122 |
| 9 | 0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56 | BUSD | BUSD | 100 | 1% | 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C |
| 10 | 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c | BTCB | BTCB | 100 | 1% | 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12 |
| 11 | 0x2170Ed0880ac9A755fd29B2688956BD959F933F8 | ETH | ETH | 100 | 1% | 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C |
| 12 | 0xF8A0BF9cF54Bb92F17374d9e9A321E6a111a51bD | LINK | LINK | 20 | 0% | 0x0b09Efc9458c00354414D2A560aA6EDa19490169 |
| 13 | 0x55d398326f99059fF775485246999027B3197955 | USDT | USDT | 100 | 1% | 0xc6f470dA6C284D16647CEE2230522a85b0818f6D |
| 14 | 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d | USDC | USDC | 100 | 1% | 0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A |
| 15 | 0x7083609fCE4d1d8Dc0C979AAb8c869Ea2C873402 | DOT | DOT | 20 | 0% | 0x568254EAAC1476faf9A908e577faa3Ab96029801 |
| 16 | 0x0392957571F28037607C14832D16f8B653eDd472 | Cake-LP | ETH / COMP | 10 | 0% | 0xf94D0de3D7e526099Fe37C48C9fFBD91E08cCb10 |
| 17 | 0x4269e7F43A63CEA1aD7707Be565a94a9189967E9 | Cake-LP | WBNB / UNI | 10 | 0% | 0x5a0E5FA9C561fE714d3A5430F1a18b3718D5Df25 |
| 18 | 0xaeBE45E3a03B734c68e5557AE04BFC76917B4686 | Cake-LP | WBNB / LINK | 10 | 0% | 0x14Df2B006508801D56943f50af010a2a9Be07ebA |
| 19 | 0xfF17ff314925Dff772b71AbdFF2782bC913B3575 | Cake-LP | VAI / BUSD | 10 | 0% | 0x8a4551973867b4DAB73522Ce79C8A77d2F92497F |
| 20 | 0xc15fa3E22c912A276550F3E5FE3b0Deb87B55aCd | Cake-LP | USDT / BUSD | 10 | 0% | 0x43454f1B2aD32641D6e3c3D6DE52cB9069e21CAb |
| 21 | 0x99d865Ed50D2C32c1493896810FA386c1Ce81D91 | Cake-LP | ETH / BETH | 20 | 0% | 0xaCB13Cf5A10702bB8B3c9ee9e601fA8327070538 |
| 22 | 0x70D8929d04b60Af4fb9B58713eBcf18765aDE422 | Cake-LP | ETH / WBNB | 10 | 0% | 0x7E05D7410092cEe59ec3c91d7D166640F5660571 |
| 23 | 0x7561EEe90e24F3b348E1087A005F78B4c8453524 | Cake-LP | BTCB / WBNB | 100 | 1% | 0x40DcB52aB47Ff18B99F4d1586D8dE50f5E763815 |
| 24 | 0x1B96B92314C44b159149f7E0303511fB2Fc4774f | Cake-LP | WBNB / BUSD | 100 | 1% | 0x7E3BB5C284aEa90faF788C1874E8a410a5621E7f |
| 25 | 0xA527a61703D82139F8a06Bc30097cC9CAA2df5A6 | Cake-LP | Cake / WBNB | 100 | 1% | 0x6A5199CF84f4189f017f2993b69C0b69021ce753 |
| 26 | 0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82 | Cake | Cake | 200 | 2% | 0x699Dfc44cfa9CC8B216559C48d76575a987472d1 |
| 27 | 0x4e0f3385d932F7179DeE045369286FFa6B03d887 | Cake-LP | ALPHA / WBNB | 8 | 0% | 0x64f166a200EC8fDe8dFF8C8F71624E7b982E282c |
| 28 | 0xBA51D1AB95756ca4eaB8737eCD450cd8F05384cF | Cake-LP | ADA / WBNB | 10 | 0% | 0x1B709b583ff354e35910020a6E540984b4fa174e |
| 29 | 0xc639187ef82271D8f517de6FEAE4FaF5b517533c | Cake-LP | BAND / WBNB | 10 | 0% | 0xB06372a25D74E6c2793F6f96353e44391b4C550e |
| 30 | 0xC7b4B32A3be2cB6572a1c9959401F832Ce47a6d2 | Cake-LP | XRP / WBNB | 3 | 0% | 0xD930A7dD87fA6201e2644fBD622E0003E40Ee0b1 |
| 31 | 0x9B989A7B8963f4b08eC094710e2966FB3c7F6C43 | Cake-LP | VIKING / BUSD | 0 | 0% | 0x3E0bf5D712B1e1Acdb811320e7337b19A609DF8e |
| 32 | 0xC79173E5f6501D7C1ab2F4E7544B13Fc6562Ce6a | Cake-LP | VIKING / WBNB | 0 | 0% | 0x19f1f2082aBc360706beAeC7Af113F8A212828E7 |
| 33 | 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c | WBNB | WBNB | 632 | 7% | 0x24342BA15dDfC2c8e35D4583E4F208BDA0c84D04 |
| 34 | 0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56 | BUSD | BUSD | 402 | 4% | 0xccFC4ECC31b106a82c405941d81b2b6EDe26E9bc |
| 35 | 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c | BTCB | BTCB | 402 | 4% | 0x402b67C415aeEed1a88a0B8d7A67a1799F1dE46E4 |
| 36 | 0x2170Ed0880ac9A755fd29B2688956BD959F933F8 | ETH | ETH | 402 | 4% | 0x46aB0bc682f59028f827C78D65C4D3Ef74f2A396 |
| 37 | 0x55d398326f99059fF775485246999027B3197955 | USDT | USDT | 402 | 4% | 0x1EA762DF2aBB80B8428dfa5A1d402b6C2Cd82635 |
| 38 | 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d | USDC | USDC | 402 | 4% | 0xA7DE31e6468aAB2D41948686768ac051E976CAB0 |
| 39 | 0xF8A0BF9cF54Bb92F17374d9e9A321E6a111a51bD | LINK | LINK | 57 | 0% | 0xDceFCb8a4C39Df62045ccd4e97842F414b2dd123 |
| 40 | 0x7083609fCE4d1d8Dc0C979AAb8c869Ea2C873402 | DOT | DOT | 57 | 0% | 0xE8038B55D7C16a979e9D588725b89E78C0cbCB48 |
| 41 | 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c | WBNB | WBNB | 750 | 8% | 0x29c089E32Ab620c782fFd4103Ee7ab89d2FaE437 |


Pools 33-40 allocation getting reduced : 

https://bscscan.com/advanced-filter?tadd=0x7f7bf15b9c68d23339c31652c8e860492991760d&fadd=0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07&mtd=0x64482f79%7eSet

<img width="1521" height="646" alt="image" src="https://github.com/user-attachments/assets/3ce9621d-235e-4158-976a-a4b7e8946849" />


"RAKE Protocol vs Stuck Assets: Complete Performance Breakdown"

| Date | BNB Price | RAKE/BNB | RAKE Price | WBNB Balance | Available Liquidity BNB | Available Liquidity USD | Dev Fee Earned | Venus Rewards | Cumulative Rewards | Total Stuck Assets |
|------|-----------|----------|------------|--------------|-------------------------|-------------------------|----------------|---------------|--------------------|--------------------|
| Feb 25, 2021 | $280 | ~447 | $125,160 | 1,000 | 448,000 | $125,440,000 | 31.80 RAKE ($3,980,088) | 31.78 RAKE ($3,977,509) | $7,957,597 | $1,557,670 |
| Feb 26, 2021 | $260 | ~100 | $26,000 | 6,500 | 650,000 | $169,000,000 | 31.80 RAKE ($826,800) | 31.78 RAKE ($826,280) | $8,810,677 | $1,520,000 |
| Feb 27, 2021 | $270 | ~59 | $15,930 | 5,800 | 342,200 | $92,394,000 | 31.80 RAKE ($506,574) | 31.78 RAKE ($506,064) | $9,823,315 | $1,490,000 |
| Feb 28, 2021 | $280 | ~14 | $3,920 | 4,200 | 58,800 | $16,464,000 | 31.80 RAKE ($124,656) | 31.78 RAKE ($124,578) | $10,072,549 | $1,460,000 |
| Mar 01, 2021 | $280 | ~12.5 | $3,500 | 6,200 | 77,500 | $21,700,000 | 31.80 RAKE ($111,300) | 31.78 RAKE ($111,230) | $10,295,079 | $1,458,331 |
| Mar 02, 2021 | $280 | ~12.5 | $3,500 | 6,800 | 85,000 | $23,800,000 | 31.80 RAKE ($111,300) | 31.78 RAKE ($111,230) | $10,517,609 | $1,497,757 |
| Mar 03, 2021 | $280 | ~12.5 | $3,500 | 8,500 | 106,250 | $29,750,000 | 31.80 RAKE ($111,300) | 31.78 RAKE ($111,230) | $10,740,139 | $1,535,008 |
| Mar 04, 2021 | $280 | ~10.7 | $3,000 | 5,000 | 53,500 | $14,980,000 | 31.80 RAKE ($95,400) | 31.78 RAKE ($95,340) | $10,930,879 | $1,508,091 |
| Mar 05, 2021 | $280 | ~6.25 | $1,750 | 5,500 | 34,375 | $9,625,000 | 31.80 RAKE ($55,650) | 18.62 RAKE ($32,585) | $11,019,114 | $1,478,607 |
| Mar 06, 2021 | $280 | ~5.36 | $1,500 | 3,000 | 16,080 | $4,502,400 | 31.80 RAKE ($47,700) | 18.62 RAKE ($27,930) | $11,094,744 | $1,456,891 |
| Mar 07, 2021 | $280 | ~5.0 | $1,400 | 3,200 | 16,000 | $4,480,000 | 31.80 RAKE ($44,520) | 18.62 RAKE ($26,068) | $11,165,332 | $1,435,175 |
| Mar 08, 2021 | $280 | ~4.3 | $1,200 | 2,800 | 12,040 | $3,371,200 | Period Total | Period Total | $11,195,000 | $1,420,000 |
| Mar 09, 2021 | $280 | ~3.9 | $1,100 | 2,600 | 10,140 | $2,839,200 | Period Total | Period Total | $11,220,000 | $1,410,000 |
| Mar 10, 2021 | $280 | ~3.6 | $1,000 | 2,200 | 7,920 | $2,217,600 | Period Total | Period Total | $11,240,000 | $1,400,000 |
| Mar 31, 2021 | $280 | ~2.5 | $700 | 900 | 2,250 | $630,000 | Period Total | Period Total | $11,280,000 | $1,420,000 |
| Apr 30, 2021 | $500 | ~1.2 | $600 | 800 | 960 | $480,000 | Period Total | Period Total | $11,380,000 | $1,650,000 |
| May 31, 2021 | $680 | ~0.9 | $612 | 750 | 675 | $459,000 | Period Total | Period Total | $11,480,000 | $1,850,000 |
| Jun 30, 2021 | $360 | ~1.0 | $360 | 720 | 720 | $259,200 | Period Total | Period Total | $11,540,000 | $1,400,000 |
| Jul 31, 2021 | $320 | ~0.75 | $240 | 700 | 525 | $168,000 | Period Total | Period Total | $11,580,000 | $1,350,000 |
| Aug 31, 2021 | $480 | ~0.5 | $240 | 680 | 340 | $163,200 | Period Total | Period Total | $11,620,000 | $1,500,000 |
| Sep 30, 2021 | $380 | ~0.44 | $168 | 650 | 286 | $108,680 | Period Total | Period Total | $11,650,000 | $1,420,000 |
| Oct 31, 2021 | $500 | ~0.33 | $165 | 620 | 205 | $102,500 | Period Total | Period Total | $11,680,000 | $1,550,000 |
| Nov 30, 2021 | $650 | ~0.15 | $97.5 | 600 | 90 | $58,500 | Period Total | Period Total | $11,700,000 | $1,700,000 |
| Dec 31, 2021 | $530 | ~0.1 | $53 | 62.15 | 6.2 | $3,286 | Period Total | Period Total | $11,710,000 | $1,480,000 |
| Jan 05, 2022 | $350 | ~2.5 | $875 | 180 | 450 | $157,500 | Period Total | Period Total | $11,715,000 | $1,350,000 |
| Jan 15, 2022 | $330 | ~15 | $4,950 | 220 | 3,300 | $1,089,000 | Period Total | Period Total | $11,730,000 | $1,320,000 |
| Jan 25, 2022 | $340 | ~65 | $22,100 | 300 | 19,500 | $6,630,000 | Period Total | Period Total | $11,800,000 | $1,340,000 |
| Jan 31, 2022 | $330 | ~20 | $6,600 | 250 | 5,000 | $1,650,000 | Period Total | Period Total | $11,850,000 | $1,320,000 |
| Feb 15, 2022 | $350 | ~12 | $4,200 | 180 | 2,160 | $756,000 | Period Total | Period Total | $11,880,000 | $1,350,000 |
| Feb 28, 2022 | $360 | ~7.86 | $2,830 | 150 | 1,179 | $424,440 | Period Total | Period Total | $11,900,000 | $1,360,000 |
