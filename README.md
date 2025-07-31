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
