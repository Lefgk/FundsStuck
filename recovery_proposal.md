# RAKE Protocol Venus Recovery Proposal


## The Issue

I'm the founder of a BSC project that operated between 2021-2022 as a fork of AutoFarm. AutoFarm had a critical bug in their strategy contract that permanently set token approvals to zero once contracts were paused. This unfortunately caused our contracts to accumulate significant amounts of vBTC, vETH, vBUSD, and other Venus wrapped tokens that became permanently locked within the contract architecture.

The technical root cause is documented in **[Data Appendix Section A](./data_appendix.md#a-technical-root-cause)**, where the `unpause()` function incorrectly maintains zero approvals instead of restoring unlimited approvals, effectively breaking all Venus protocol interactions.

Our affected contracts currently hold **$3.2 million in supplied assets** with **$1.77 million borrowed**, resulting in **$1.43 million net stuck value** across eight different Venus markets 
(detailed breakdown in **[Data Appendix Section A](./data_appendix.md#b-affected-contract-details)**).

We are requesting assistance to recover **$1.43 million** in Venus protocol tokens that are permanently stuck in our smart contracts due to an inherited approval bug from AutoFarm's strategy implementation. As the verified owner of both the deployer wallet and affected contracts, we seek Venus protocol team collaboration to unlock these assets and resume normal operations.

## Why Now?

My development partner Lef was working with me on a new project that involves Venus contracts and recognized that a contract owned by me had a significant amount of Venus tokens. This initiated our collaboration to develop a solution for recovering these tokens, as I still maintain full control and ownership of both the deployer wallet and the affected contracts.

Our exploration of various recovery methods is detailed in the technical appendix (see **Data Appendix Section C**), where we systematically tested multiple contract functions to identify the exact failure point.


## Historical Context

In 2021, when this issue first occurred, it coincided with Venus's depegging event, which was ultimately resolved through Binance's intervention. We had reached out at the time, but due to the depegging crisis and the overwhelming volume of support tickets, the Venus team was unable to provide assistance with our specific technical issue, resulting in minimal progress toward resolution.

The historical performance data shows our protocol managed substantial assets during its operational period, with peak individual asset values reaching over $1.1 million in ETH holdings alone (**Data Appendix Section F**).

## User Refund and Impact Mitigation

The bug affected our platform very shortly after launch. Fortunately, we had other functioning products, so the stuck Venus vault assets represented approximately 4% of our total value locked (TVL). Since these funds were unable to generate yield for users or the platform, we took immediate action to protect our community.

We successfully refunded all affected stuck users and paid them a premium by diverting team profits earned from our other operational products. This ensured no users suffered financial losses due to the technical issue. After completing user refunds, I shifted focus to other development priorities, as initial attempts to engage Venus support yielded limited progress.

## Current Development and Recovery Plans

For the past 18 months, I have self-funded development of a unique trading and strategy management platform that will launch on BSC, bringing high-demand tools to BSC users that are currently unavailable on any EVM chain. This development specifically includes integration with Venus pools and lending protocols.

The recovered funds would be allocated toward:

- Professional smart contract audits
- Marketing and user acquisition
- Continued platform development and feature expansion

This platform launch will drive increased volume and expand the user base for Venus and other key BSC ecosystem contributors we're partnering with.

## Verification and Next Steps

I can provide complete proof of ownership and control over the contracts containing the stuck tokens. The contract addresses, transaction histories, and technical architecture are fully documented in **Data Appendix Sections B, D, and E**.

We have conducted comprehensive testing in forked environments to confirm the exact nature of the approval bug and its impact on all lending-related functions (**Data Appendix Section C**).

I would like to move forward with the recovery process as soon as possible and am available to provide any additional documentation, proof of ownership, or technical details required by the Venus protocol team.

---

_For complete technical specifications, contract addresses, historical data, and testing results, please refer to the accompanying Data Appendix document._
