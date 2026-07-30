# RAKE Protocol — Venus Stuck-Funds Recovery

Documentation for recovering **~$974k** of Venus vTokens permanently stuck across **7 ERC-20
Venus markets** in RAKE Protocol strategy contracts, caused by an approval bug inherited from the
AutoFarm fork (`unpause()` sets the vToken approval to zero, breaking all Venus interactions).
**$2,107,606 supplied against $1,135,169 borrowed — $972,437 of net stuck equity**, held on-chain
by **103 wallets across 146 positions**. The proposal asks Venus governance to help recover the
funds via a vToken implementation-swap that returns the full balance with no penalty or bad debt.

The bug zeroes an **ERC-20 allowance**, so it can only affect the 7 ERC-20 strategies (pids 9–15).
The eighth Venus strategy, **vBNB (pid 8), is not part of this request** — the vBNB market takes
native BNB through a `payable mint()` and has no ERC-20 underlying, so no allowance is ever
consulted and the approval bug cannot be what stopped it. **vBNB is nonetheless also stuck**, by an
unrelated fault: years of borrow interest outrunning supply interest pushed it past its own
leverage ceiling, and its unwind path reverts on a SafeMath underflow. It is excluded from this
request because it is a different bug and needs no Venus action — not because the position is
empty. See `RECOVERY_ANALYSIS.md`.

## Contents

- **`recovery_proposal.md`** — the recovery proposal (narrative: the issue, history, and the requested solution).
- **`VENUS_RECOVERY_REQUEST.md`** — the detailed request to Venus: affected contracts, root cause, recommended recovery, precedents, and proof.
- **`data_appendix.md`** — supporting data: live on-chain positions, verification commands, and the deprecation/liquidation record.
- **`raw_data/`** — raw stuck-funds analysis and source figures.
