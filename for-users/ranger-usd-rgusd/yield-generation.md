---
description: 'rgUSD generates yield through two complementary strategies:'
---

# Yield Generation

### Yield Aggregator

Proprietary smart routing algorithm dynamically allocates capital to the highest-yielding USDC lending pools on Solana. It continuously monitors yield across major platforms on Solana and simulates price impact every 30 seconds, rebalancing deposits when simulated returns exceed certain thresholds. If conditions don't warrant immediate rebalancing, the system naturally rebalances every 10 minutes.

This dynamic capital allocation allows rgUSD to capture the highest lending rates on Solana at any point of time as individual pools naturally go through cycles of high and low lending rates depending on utilisation and TVL flows. By routing between platforms, rgUSD captures multiple yield peaks while avoiding the dips, maintaining higher average returns over time.

Currently, rgUSD utilises the following lending protocols:

• Jupiter Lend

• Drift Protocol

• Kamino Finance

These platforms were integrated for their size, liquidity depth and operational resilience. Helping to minimise counterparty and systemic risks.&#x20;

### Delta-Neutral Funding Rate Enhancement

Base lending yields can be boosted by farming funding rates when those returns are expected to exceed what’s available in standard lending markets:

* Funding Rate Farming: Earns funding fees by holding spot positions in major L1 assets (SOL, BTC, and ETH) while simultaneously hedging with matching perpetual positions on Drift Protocol to maintain a delta-neutral profile.

This design allows rgUSD to earn above market returns while minimising risk.&#x20;

Together, both strategies create a robust yield engine: lending provides stable base returns, while funding rate farming adds opportunistic boosts during favourable market conditions.

### Performance Summary (as of 23 Nov 2025)

<table><thead><tr><th width="142.57421875">Metric </th><th align="center" valign="middle">Ranger</th><th align="center">Drift</th><th align="center">Kamino</th><th align="center">Jupiter Len</th><th align="center">Lulo Classic</th></tr></thead><tbody><tr><td><strong>7-Day APY</strong></td><td align="center" valign="middle"><strong>8.56%</strong></td><td align="center">3.51%</td><td align="center">4.20%</td><td align="center">4.82%</td><td align="center">5.52%</td></tr><tr><td><strong>30-Day APY</strong></td><td align="center" valign="middle"><strong>8.89%</strong></td><td align="center">6.06%</td><td align="center">3.83%</td><td align="center">6.61%</td><td align="center">6.67%</td></tr><tr><td><strong>Yield Strategy</strong></td><td align="center" valign="middle"><p>Lending +</p><p>Delta neutral</p></td><td align="center"><p>Lending</p><p>only</p></td><td align="center"><p>Lending</p><p>only</p></td><td align="center"><p>Lending</p><p>only</p></td><td align="center"><p>Lending</p><p>only</p></td></tr></tbody></table>
