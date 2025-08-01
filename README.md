# Stuck Funds Analysis Report - RAKE Protocol

## Executive Summary

We had a project that was a fork of AutoFarm that now has $1.43 million of Vtokens stuck due to a approval bug inherited from AutoFarm's strategy contract. This bug permanently set approvals to 0 once the contract was paused. The contracts cannot perform any Venus lending operations due to zero token approvals.

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

deposit()
withdraw()
farm()
earn()
deleverageOnce()
deleverageUntilNotOverLevered()
rebalance()

**Failure Point:** All transactions revert with error:
```
BEP20: transfer amount exceeds allowance
```


RakeFarm LP : 
=== RakeFarm Detailed Pool Analysis ===
Contract Address: 0x7f7Bf15B9c68D23339C31652C8e860492991760d
Block Number: 1990376681

^C
lefgiak@Lefs-MacBook-Pro SCALE % 
lefgiak@Lefs-MacBook-Pro SCALE % npx hardhat run scripts/poolAnalysis.js --network localhost --no-compile
WARNING: You are currently using Node.js v24.4.0, which is not supported by Hardhat. This can lead to unexpected behavior. See https://hardhat.org/nodejs-versions


=== RakeFarm Detailed Pool Analysis ===
Contract Address: 0x7f7Bf15B9c68D23339C31652C8e860492991760d
Block Number: 1990376681

=== Farm Overview ===
Total Pools: 42
Total Allocation Points: 8539
AUTO Per Block: 8000000000000000
Start Block: 5195791
Owner: 0x16c7C45725A977ae6530e8dEE73F6da9aE2e7E07
Current Block: 1990376681

=== Pool Details ===
----------------------------------------
PID: 0
Stake Token: 0x1Cb667Fe903DBdcbd27D8B35E82FbCEF4Ca0f621
Token Symbol: Cake-LP
LP Pair: WBNB / RAKE
Allocation Points: 1500
Allocation % (bp): 17
Strategy Address: 0xaE3Cf8A82F9441E38b92a2575a9b73d444a93D75

----------------------------------------
PID: 1
Stake Token: 0x889A24725Ce7E867FEB9ea5f1c96b0111c91F39D
Token Symbol: Cake-LP
LP Pair: BR34P / RAKE
Allocation Points: 1200
Allocation % (bp): 14
Strategy Address: 0x014c44f948EB082c52A8c6dC090bA53273A425A5

----------------------------------------
PID: 2
Stake Token: 0x70e882efa9beA28262d4873e65d5f65E9B2bABa6
Token Symbol: Cake-LP
LP Pair: BR34P / WBNB
Allocation Points: 1000
Allocation % (bp): 11
Strategy Address: 0xDF635b131a9eb4B7CA4b50F21c3a4F722F9f3699

----------------------------------------
PID: 3
Stake Token: 0xA0718093baa3E7AAE054eED71F303A4ebc1C076f
Token Symbol: Cake-LP
LP Pair: sBDO / BUSD
Allocation Points: 5
Allocation % (bp): 0
Strategy Address: 0xF5737944b82A2C9235aFC140c627e66b4496C2e7

----------------------------------------
PID: 4
Stake Token: 0xc5b0d73A7c0E4eaF66baBf7eE16A2096447f7aD6
Token Symbol: Cake-LP
LP Pair: BDO / BUSD
Allocation Points: 5
Allocation % (bp): 0
Strategy Address: 0x3576F383Af96F59280bDEd50Dd18d7ecEdAE18D6

----------------------------------------
PID: 5
Stake Token: 0x74690f829fec83ea424ee1F1654041b2491A7bE9
Token Symbol: Cake-LP
LP Pair: BDO / WBNB
Allocation Points: 2
Allocation % (bp): 0
Strategy Address: 0xEEA269fe7EF8d7a7AE438A2D31A3B2089242b7D2

----------------------------------------
PID: 6
Stake Token: 0x19e7cbECDD23A16DfA5573dF54d98F7CaAE03019
Token Symbol: Cake-LP
LP Pair: BUSD / EGG
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0xDE1DAa4BD680056b4F46Dd1602617eFbB29c3378

----------------------------------------
PID: 7
Stake Token: 0xd1B59D11316E87C3a0A069E80F590BA35cD8D8D3
Token Symbol: Cake-LP
LP Pair: WBNB / EGG
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x06aC46ad065eeaB1B4C08A8CE0F02649669c00df

----------------------------------------
PID: 8
Stake Token: 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c
Token Symbol: WBNB
Allocation Points: 150
Allocation % (bp): 1
Strategy Address: 0xf498e4C06CcE3bFeaD5f32a69Db3d39af401E122

----------------------------------------
PID: 9
Stake Token: 0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56
Token Symbol: BUSD
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0xDF3df3EE9Fb6D5c9B4fdcF80A92D25d2285A859C

----------------------------------------
PID: 10
Stake Token: 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c
Token Symbol: BTCB
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0xB3bCA2C1c2C4DF2C903BE3F341C96732Ac8b3c12

----------------------------------------
PID: 11
Stake Token: 0x2170Ed0880ac9A755fd29B2688956BD959F933F8
Token Symbol: ETH
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0x9E3e8878B53c5762Ec6F461701f02ee6d1D9d19C

----------------------------------------
PID: 12
Stake Token: 0xF8A0BF9cF54Bb92F17374d9e9A321E6a111a51bD
Token Symbol: LINK
Allocation Points: 20
Allocation % (bp): 0
Strategy Address: 0x0b09Efc9458c00354414D2A560aA6EDa19490169

----------------------------------------
PID: 13
Stake Token: 0x55d398326f99059fF775485246999027B3197955
Token Symbol: USDT
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0xc6f470dA6C284D16647CEE2230522a85b0818f6D

----------------------------------------
PID: 14
Stake Token: 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d
Token Symbol: USDC
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0x7485704d8B6cEBd1411F5AAfEB063e6f2816069A

----------------------------------------
PID: 15
Stake Token: 0x7083609fCE4d1d8Dc0C979AAb8c869Ea2C873402
Token Symbol: DOT
Allocation Points: 20
Allocation % (bp): 0
Strategy Address: 0x568254EAAC1476faf9A908e577faa3Ab96029801

----------------------------------------
PID: 16
Stake Token: 0x0392957571F28037607C14832D16f8B653eDd472
Token Symbol: Cake-LP
LP Pair: ETH / COMP
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0xf94D0de3D7e526099Fe37C48C9fFBD91E08cCb10

----------------------------------------
PID: 17
Stake Token: 0x4269e7F43A63CEA1aD7707Be565a94a9189967E9
Token Symbol: Cake-LP
LP Pair: WBNB / UNI
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x5a0E5FA9C561fE714d3A5430F1a18b3718D5Df25

----------------------------------------
PID: 18
Stake Token: 0xaeBE45E3a03B734c68e5557AE04BFC76917B4686
Token Symbol: Cake-LP
LP Pair: WBNB / LINK
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x14Df2B006508801D56943f50af010a2a9Be07ebA

----------------------------------------
PID: 19
Stake Token: 0xfF17ff314925Dff772b71AbdFF2782bC913B3575
Token Symbol: Cake-LP
LP Pair: VAI / BUSD
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x8a4551973867b4DAB73522Ce79C8A77d2F92497F

----------------------------------------
PID: 20
Stake Token: 0xc15fa3E22c912A276550F3E5FE3b0Deb87B55aCd
Token Symbol: Cake-LP
LP Pair: USDT / BUSD
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x43454f1B2aD32641D6e3c3D6DE52cB9069e21CAb

----------------------------------------
PID: 21
Stake Token: 0x99d865Ed50D2C32c1493896810FA386c1Ce81D91
Token Symbol: Cake-LP
LP Pair: ETH / BETH
Allocation Points: 20
Allocation % (bp): 0
Strategy Address: 0xaCB13Cf5A10702bB8B3c9ee9e601fA8327070538

----------------------------------------
PID: 22
Stake Token: 0x70D8929d04b60Af4fb9B58713eBcf18765aDE422
Token Symbol: Cake-LP
LP Pair: ETH / WBNB
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x7E05D7410092cEe59ec3c91d7D166640F5660571

----------------------------------------
PID: 23
Stake Token: 0x7561EEe90e24F3b348E1087A005F78B4c8453524
Token Symbol: Cake-LP
LP Pair: BTCB / WBNB
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0x40DcB52aB47Ff18B99F4d1586D8dE50f5E763815

----------------------------------------
PID: 24
Stake Token: 0x1B96B92314C44b159149f7E0303511fB2Fc4774f
Token Symbol: Cake-LP
LP Pair: WBNB / BUSD
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0x7E3BB5C284aEa90faF788C1874E8a410a5621E7f

----------------------------------------
PID: 25
Stake Token: 0xA527a61703D82139F8a06Bc30097cC9CAA2df5A6
Token Symbol: Cake-LP
LP Pair: Cake / WBNB
Allocation Points: 100
Allocation % (bp): 1
Strategy Address: 0x6A5199CF84f4189f017f2993b69C0b69021ce753

----------------------------------------
PID: 26
Stake Token: 0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82
Token Symbol: Cake
Allocation Points: 200
Allocation % (bp): 2
Strategy Address: 0x699Dfc44cfa9CC8B216559C48d76575a987472d1

----------------------------------------
PID: 27
Stake Token: 0x4e0f3385d932F7179DeE045369286FFa6B03d887
Token Symbol: Cake-LP
LP Pair: ALPHA / WBNB
Allocation Points: 8
Allocation % (bp): 0
Strategy Address: 0x64f166a200EC8fDe8dFF8C8F71624E7b982E282c

----------------------------------------
PID: 28
Stake Token: 0xBA51D1AB95756ca4eaB8737eCD450cd8F05384cF
Token Symbol: Cake-LP
LP Pair: ADA / WBNB
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0x1B709b583ff354e35910020a6E540984b4fa174e

----------------------------------------
PID: 29
Stake Token: 0xc639187ef82271D8f517de6FEAE4FaF5b517533c
Token Symbol: Cake-LP
LP Pair: BAND / WBNB
Allocation Points: 10
Allocation % (bp): 0
Strategy Address: 0xB06372a25D74E6c2793F6f96353e44391b4C550e

----------------------------------------
PID: 30
Stake Token: 0xC7b4B32A3be2cB6572a1c9959401F832Ce47a6d2
Token Symbol: Cake-LP
LP Pair: XRP / WBNB
Allocation Points: 3
Allocation % (bp): 0
Strategy Address: 0xD930A7dD87fA6201e2644fBD622E0003E40Ee0b1

----------------------------------------
PID: 31
Stake Token: 0x9B989A7B8963f4b08eC094710e2966FB3c7F6C43
Token Symbol: Cake-LP
LP Pair: VIKING / BUSD
Allocation Points: 0
Allocation % (bp): 0
Strategy Address: 0x3E0bf5D712B1e1Acdb811320e7337b19A609DF8e

----------------------------------------
PID: 32
Stake Token: 0xC79173E5f6501D7C1ab2F4E7544B13Fc6562Ce6a
Token Symbol: Cake-LP
LP Pair: VIKING / WBNB
Allocation Points: 0
Allocation % (bp): 0
Strategy Address: 0x19f1f2082aBc360706beAeC7Af113F8A212828E7

----------------------------------------
PID: 33
Stake Token: 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c
Token Symbol: WBNB
Allocation Points: 632
Allocation % (bp): 7
Strategy Address: 0x24342BA15dDfC2c8e35D4583E4F208BDA0c84D04

----------------------------------------
PID: 34
Stake Token: 0xe9e7CEA3DedcA5984780Bafc599bD69ADd087D56
Token Symbol: BUSD
Allocation Points: 402
Allocation % (bp): 4
Strategy Address: 0xccFC4ECC31b106a82c405941d81b2b6EDe26E9bc

----------------------------------------
PID: 35
Stake Token: 0x7130d2A12B9BCbFAe4f2634d864A1Ee1Ce3Ead9c
Token Symbol: BTCB
Allocation Points: 402
Allocation % (bp): 4
Strategy Address: 0x402b67C415aeEd1a88a0B8d7A67a1799F1dE46E4

----------------------------------------
PID: 36
Stake Token: 0x2170Ed0880ac9A755fd29B2688956BD959F933F8
Token Symbol: ETH
Allocation Points: 402
Allocation % (bp): 4
Strategy Address: 0x46aB0bc682f59028f827C78D65C4D3Ef74f2A396

----------------------------------------
PID: 37
Stake Token: 0x55d398326f99059fF775485246999027B3197955
Token Symbol: USDT
Allocation Points: 402
Allocation % (bp): 4
Strategy Address: 0x1EA762DF2aBB80B8428dfa5A1d402b6C2Cd82635

----------------------------------------
PID: 38
Stake Token: 0x8AC76a51cc950d9822D68b83fE1Ad97B32Cd580d
Token Symbol: USDC
Allocation Points: 402
Allocation % (bp): 4
Strategy Address: 0xA7DE31e6468aAB2D41948686768ac051E976CAB0

----------------------------------------
PID: 39
Stake Token: 0xF8A0BF9cF54Bb92F17374d9e9A321E6a111a51bD
Token Symbol: LINK
Allocation Points: 57
Allocation % (bp): 0
Strategy Address: 0xDceFCb8a4C39Df62045ccd4e97842F414b2dd123

----------------------------------------
PID: 40
Stake Token: 0x7083609fCE4d1d8Dc0C979AAb8c869Ea2C873402
Token Symbol: DOT
Allocation Points: 57
Allocation % (bp): 0
Strategy Address: 0xE8038B55D7C16a979e9D588725b89E78C0cbCB48

----------------------------------------
PID: 41
Stake Token: 0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c
Token Symbol: WBNB
Allocation Points: 750
Allocation % (bp): 8
Strategy Address: 0x29c089E32Ab620c782fFd4103Ee7ab89d2FaE437

=== Summary ===
Total Pools: 42
Total Allocation Points: 8539
