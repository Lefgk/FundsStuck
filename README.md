# RAKE Protocol — Venus Stuck-Funds Recovery

Documentation for recovering ~$1M of Venus vTokens permanently stuck in RAKE Protocol
strategy contracts, caused by an approval bug inherited from the AutoFarm fork
(`unpause()` sets the vToken approval to zero, breaking all Venus interactions).
The proposal asks Venus governance to help recover the funds via a vToken
implementation-swap that returns the full balance with no penalty or bad debt.

## Contents

- **`recovery_proposal.md`** — the recovery proposal (narrative: the issue, history, and the requested solution).
- **`VENUS_RECOVERY_REQUEST.md`** — the detailed request to Venus: affected contracts, root cause, recommended recovery, precedents, and proof.
- **`data_appendix.md`** — supporting data: live on-chain positions, verification commands, and the deprecation/liquidation record.
- **`raw_data/`** — raw stuck-funds analysis and source figures.
