# Vault Operations

Once your vault is live with initialized strategies and allocated funds, you're responsible for ongoing operations. This section covers monitoring, automation, and cost management.

{% hint style="danger" %}
**Voltr/Ranger does not provide hosting services.** You are responsible for running and maintaining your own infrastructure for bots, scripts, and monitoring. There is no managed service for vault operations.
{% endhint %}

## Operational Responsibilities

As a vault operator, you need to handle:

| Responsibility | Frequency | Automation Recommended |
|---------------|-----------|----------------------|
| **Monitor vault health** | Continuous | Yes |
| **Rebalance allocations** | As needed (daily/weekly) | Yes |
| **Claim protocol rewards** | Per protocol schedule | Yes |
| **Swap rewards to base asset** | After each claim | Yes |
| **Respond to market conditions** | As needed | Depends on strategy |
| **Maintain SOL balance** | Check weekly | Optional |
| **Monitor strategy performance** | Daily | Yes |

## Section Contents

{% content-ref url="monitoring-and-api.md" %}
[Monitoring & API](monitoring-and-api.md)
{% endcontent-ref %}

{% content-ref url="running-bots-and-scripts.md" %}
[Running Bots & Scripts](running-bots-and-scripts.md)
{% endcontent-ref %}

{% content-ref url="gas-fees-and-ata-costs.md" %}
[Gas Fees & ATA Costs](gas-fees-and-ata-costs.md)
{% endcontent-ref %}
