# Key Participants

## Users

Users are the primary participants who deposit assets into vaults to earn yield:

* Deposit assets into any vault of their choice
* Receive LP tokens representing their share of the vault
* Withdraw their assets plus earned yields at any time
* Track their positions and yields through LP token value

{% content-ref url="../for-users/user-interface-guide.md" %}
[user-interface-guide.md](../for-users/user-interface-guide.md)
{% endcontent-ref %}

## Vault Owners

Vault owners are responsible for the overall vault configuration and security:

* Create new vaults with custom parameters
* Add or remove adaptors to determine available strategies
* Set fee structures (performance and management fees)
* Appoint fund managers to handle allocation
* Configure vault-specific parameters like deposit caps

{% content-ref url="../for-vault-owners/vault-initialization-guide/" %}
[vault-initialization-guide](../for-vault-owners/vault-initialization-guide/)
{% endcontent-ref %}

{% content-ref url="../for-vault-owners/frontend-integration-guide/" %}
[frontend-integration-guide](../for-vault-owners/frontend-integration-guide/)
{% endcontent-ref %}

## Vault Managers

Vault managers are appointed by vault owners to handle capital allocation across different strategies:

* Rebalance assets between different DeFi protocols
* Monitor strategy performance
* Adjust allocation based on market conditions
* Optimize for maximum yield while managing risk
* Execute strategy changes

{% content-ref url="../for-vault-owners/fund-allocation-guide/" %}
[fund-allocation-guide](../for-vault-owners/fund-allocation-guide/)
{% endcontent-ref %}

## DeFi Protocol Teams

Protocol teams can extend Voltr's functionality by creating custom adaptors:

* Develop custom adaptors for their protocols
* Define protocol-specific interaction logic
* Implement yield generation strategies
* Integrate with the vault standard interface
* Update adaptor implementations

{% content-ref url="../for-defi-protocols/adaptor-creation-guide/" %}
[adaptor-creation-guide](../for-defi-protocols/adaptor-creation-guide/)
{% endcontent-ref %}

## Interaction Flow

1. Protocol Teams create adaptors
2. Vault Owners set up vaults, appoint managers, and connect adaptors
3. Users deposit into vaults and receive yields
4. Vault Managers handle allocation between adaptors

## Roles and Permissions

<table><thead><tr><th width="226">Action</th><th>Users</th><th>Owners</th><th width="128">Managers</th><th>Protocols</th></tr></thead><tbody><tr><td>Deposit/Withdraw</td><td>✓</td><td></td><td></td><td></td></tr><tr><td>Create Vault</td><td></td><td>✓</td><td></td><td></td></tr><tr><td>Add/Remove Adaptors</td><td></td><td>✓</td><td></td><td></td></tr><tr><td>Manage Allocations</td><td></td><td>✓</td><td>✓</td><td></td></tr><tr><td>Create Adaptors</td><td></td><td></td><td></td><td>✓</td></tr></tbody></table>
