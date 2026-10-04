# PropHedge

**Hedging Prop Firm Challenges with Static Drawdown: Hedge Ratios, Path-Dependent PnL and Expected Value**

PropHedge is an analytical model for hedging prop firm challenges with an opposing, smaller position on a live brokerage account. It computes the net PnL of every path through a two-stage challenge (evaluation and funded phase), the expected value, and the Sortino-optimal hedge ratios.

This repository contains the code accompanying the working paper by Lennart Rüther (Independent Researcher).

📄 **Paper:** SSRN link will be added after publication.

---

## Idea

Prop firms sell evaluation accounts for a one-time fee. Losses are capped at the fee, while profits on the funded account are paid out. This asymmetric payoff can be hedged:

- Every trade on the prop account is mirrored on a live account in the **opposite direction**.
- The live position is always **smaller** than the prop position, scaled by the hedge ratio $h \in [0,1]$:

$$X^{\text{live}} = -h \cdot X$$

- If the challenge fails, the live account recovers part of the fee. If it passes, the payout on the funded account is meant to offset the hedge loss.

With suitable hedge ratios $(h_E, h_F)$ for the evaluation and funded phase, **all terminal outcomes can become positive**.

## Model

The model assumes a win rate of exactly 50 % with a fixed risk $r$ per trade, so the account balance follows a symmetric random walk with absorbing barriers at the profit target $T$ and the maximum drawdown $D$.

**Pass probabilities** (gambler's ruin):

$$p_E = \frac{D_E}{D_E + T_E}, \qquad p_F = \frac{D_F}{D_F + T_F}$$

**Expected number of trades:**

$$n = \frac{T \cdot D}{r^2}$$

**Path PnL:**

| Path | PnL |
|---|---|
| Evaluation passed (intermediate) | $\Pi_{PE} = -C - h_E T_E A - F_E$ |
| Evaluation failed | $\Pi_{FE} = -C + h_E D_E A - F_E$ |
| Funded account lost | $\Pi_{FF} = \Pi_{PE} + h_F D_F A_F - F_F$ |
| Payout reached | $\Pi_{PO} = \Pi_{PE} - h_F T_F A_F + \varphi T_F A_F - F_F$ |

**Expected value:**

$$\mathbb{E}[\Pi] = (1-p_E)\,\Pi_{FE} + p_E p_F\,\Pi_{PO} + p_E(1-p_F)\,\Pi_{FF}$$

The hedge ratios are optimised by grid search with respect to the **Sortino ratio of the terminal outcomes**, i.e. the model seeks hedge ratios that keep the outcomes of all paths as close together as possible.

## Scope

The model applies **only to challenges with a static drawdown**. With a trailing drawdown, the hedge ratio would have to be adjusted dynamically after every trade, which makes the strategy highly sensitive to bad fills, slippage and execution delays.

## Example

With the default parameters ($50,000 account, $149 fee, 8 % target, 4 % drawdown in both phases, 2 % risk per trade):

| | Value |
|---|---|
| Optimal hedge ratios | $h_E = 0.25$, $h_F = 0.65$ |
| Evaluation failed | +$295 |
| Funded account lost | +$183 |
| Payout reached | +$291 |
| Expected value | ≈ $269.67 |
| Worst running PnL (capital requirement) | −$4,029 |

## Key Findings

- For suitable hedge ratios, all terminal outcomes of the challenge become positive.
- The expected value grows approximately linearly with the maximum drawdown in the practically relevant range.
- The rules of a challenge (drawdown, profit target, profit split) affect the expected value considerably more than its price.
- With a 50 % win rate, daily drawdown limits and consistency rules both reduce to an upper bound on position size. The risk per trade $r$ should be maximised within that bound to minimise transaction and hedging costs.
- Since hedging is prohibited by most prop firms, denied payouts are the central risk. In the example, at least about 44 % of reached payouts must actually be paid for the strategy to be profitable.

## Usage

**Requirements:** Python 3.10+, NumPy, Plotly, Jupyter

```bash
git clone https://github.com/Lennart967/PropHedge.git
cd PropHedge
pip install numpy plotly jupyter
jupyter notebook hedge_simulator.ipynb
```

All parameters (account size, fee, profit targets, drawdowns, risk per trade, fees per trade) are defined at the top of the notebook and can be adjusted to model any static-drawdown challenge.

**Outputs:**

- Heatmap of the Sortino ratio over all hedge ratio combinations with the optimum marked
- Decision tree with the PnL of every path at the optimal hedge ratios
- Worst running PnL along the payout path and the resulting capital requirement

## Assumptions and Limitations

- Win rate of exactly 50 % with symmetric wins and losses; spread and slippage are not modelled.
- No rule other than the maximum drawdown is breached (daily drawdown, consistency rule, minimum trading days).
- The funded account starts at $A_F = A(1 + T_E)$; many prop firms reset it to the original account size instead.
- Only a single payout is considered.
- Contract granularity and margin requirements are only approximated.

## Disclaimer

This project is a purely analytical study. It does not constitute investment, financial or legal advice and is not an invitation to violate the terms of service of prop firms or brokers. Hedging between a prop account and a personal account is prohibited by most providers and may result in account termination and denied payouts. Trading futures and other financial instruments involves substantial risk of loss.

## Acknowledgements

The idea is inspired by the [Prop Arbitrage Simulator](https://proparbitragesimulator.atjresearch.com/) by ATJ Research.

## Citation

```bibtex
@misc{ruether2026prophedge,
  author = {R{\"u}ther, Lennart},
  title  = {PropHedge: Hedging Prop Firm Challenges with Static Drawdown -- Hedge Ratios, Path-Dependent PnL and Expected Value},
  year   = {2026},
  note   = {Working Paper, Version 1.0, SSRN}
}
```

## License

- **Code:** MIT License, see [LICENSE](LICENSE)
- **Paper:** [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)