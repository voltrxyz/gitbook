# Fees & Accounting

When the vault realizes a profit, that profit increases the overall asset value. However, not all of this profit is immediately available for fee collection. A portion is “locked” and gradually becomes available (or “unlocked”) over time. Fees are only charged on new profits that exceed a historical peak—this is known as the **high water mark**.

***

### Locked Profit

To prevent sandwich-ing or frontrunning on gains, newly realized profit is initially locked. Over a set duration, this locked profit degrades until it is fully unlocked. The locked profit is calculated as:

$$
\text{Locked Profit} = \left(\frac{\text{Degradation Duration} - \text{Time Elapsed}}{\text{Degradation Duration}}\right) \times \text{Previous Locked Profit}
$$



Once the degradation period has passed, the locked profit becomes zero.

***

### High Water Mark

The high water mark represents the highest asset-per-share value the vault has ever reached. Fees are only applied to the profit that exceeds this mark. If the current asset-per-share value is higher than the high water mark, then the eligible profit is determined by the difference:

$$
\text{Eligible Profit} = \begin{cases} 
0, & \text{if } r_{\text{current}} \leq r_{\text{HWM}} \\
\left(r_{\text{current}} - r_{\text{HWM}}\right) \times \text{Total Shares}, & \text{if } r_{\text{current}} > r_{\text{HWM}}
\end{cases}
$$

$$
\text{Here, } r_{\text{current}} \text{ is the current asset-per-share ratio and } r_{\text{HWM}} \text{ is the high water mark ratio.}
$$

***

### Fee Calculation

Once eligible profit is determined, performance fees are calculated based on a preset fee rate (expressed in basis points). The fee amount is computed as:

$$
\text{Fee Amount} = \frac{\text{Eligible Profit} \times \text{Fee Rate (bps)}}{10000}
$$

If no eligible profit exists (i.e., the current performance does not exceed the high water mark), no fee is charged.

***

### Fee Splitting

The calculated fee amount is split between two parties (for example, an admin and a manager). Suppose the admin’s share is defined by aa basis points and the manager’s by mm basis points. The split is determined as follows:

* **Admin Share:**

$$
\text{Admin Share} = \frac{\text{Fee Amount} \times a}{a + m}
$$

* **Manager Share:**

$$
\text{Manager Share} = \text{Fee Amount} - \text{Admin Share}
$$

***

### Loss Handling

If the vault incurs a loss, the total asset value is reduced by the loss amount. In this scenario, no performance fee is collected, and the loss directly impacts the vault’s asset value.

***

### Summary

* **Profit Handling:** Profits are added to the total asset value, but a portion remains locked and degrades over time.
* **High Water Mark:** Only profits that exceed the historical peak (high water mark) are subject to fees.
* **Fee Calculation:** Fees are calculated as a percentage of the eligible profit and then minted as new tokens proportional to the vault's current state.
* **Fee Splitting:** The fee is divided between parties according to predefined shares.

This approach ensures that fees are only taken on genuine, new gains while protecting investor interests by avoiding fees on previously earned or unrealized profit.
