# Supported Integrations

{% content-ref url="../security/deployed-programs.md" %}
[deployed-programs.md](../security/deployed-programs.md)
{% endcontent-ref %}

The Voltr core team has created the following adaptors for easy integration with any Voltr vaults. Adaptors are modular and permissionless — anyone can create their own adaptors to connect Voltr vaults with any protocol.

## Adaptor Overview

| Adaptor Program             | Integrations                            | Sample Scripts                                                                         |
| --------------------------- | --------------------------------------- | -------------------------------------------------------------------------------------- |
| **Generic Lending Adaptor** | Project0 and Save Lending Market        | [lend-scripts](https://github.com/voltrxyz/lend-scripts)                               |
| **Kamino Adaptor**          | Kamino Vaults, Kamino Lending Market    | [kamino-scripts](https://github.com/voltrxyz/kamino-scripts)                           |
| **Drift Adaptor**           | Drift Vaults, Drift Lend, Drift Perps   | [drift-scripts](https://github.com/voltrxyz/drift-scripts)                             |
| **Raydium Adaptor**         | All Raydium CLMM Pools                  | [client-raydium-clmm-scripts](https://github.com/voltrxyz/client-raydium-clmm-scripts) |
| **Jupiter Adaptor**         | SPL-Tokens (Jupiter Swap), Jupiter Lend | [spot-scripts](https://github.com/voltrxyz/spot-scripts)                               |
| **Trustful Adaptor**        | Centralised Exchanges                   | [trustful-scripts](https://github.com/voltrxyz/trustful-scripts)                       |

{% hint style="info" %}
For detailed strategy initialization instructions, see the [Strategy Setup Guide](strategy-setup-guide.md).
{% endhint %}
