<!-- solve-problem:abdebcaf-2275-428a-8959-794c0a697b46 -->
# Tenfold risk-adjusted liquidity efficiency

A [Tangle Solve problem](https://solve.tangle.tools/problems/abdebcaf-2275-428a-8959-794c0a697b46).

## What is the problem?

The problem is a capital-efficiency moonshot for automated market makers (AMMs): construct a liquidity-provision mechanism (or prove one impossible) that provides **ten times the executable depth per unit of loss-absorbing capital**, relative to a baseline such as Uniswap v2 uniform liquidity, while holding tail-risk tolerance fixed and accounting for hedging and rebalancing costs.

Formalize as follows. Let an LP commit capital $K$ into a market-making strategy on a volatile asset pair. Define:

- **Executable depth** $D(p)$: the marginal-cost-adjusted trade size the pool absorbs before price impact/slippage exceeds a threshold $\varepsilon$ at spot price $p$ — i.e., effectively the local liquidity reserves $x \cdot y$ derivatives, integrated over a price band.
- **Loss-absorbing capital** $K$: the capital the LP must post that is genuinely at risk (not just nominally staked), i.e., the amount that can be lost to adverse selection, inventory risk, and LVR (loss-versus-rebalancing) over the risk horizon.
- **Tail-risk constraint**: $\Pr[\text{LP loss over horizon } T > \alpha K] \le \delta$, for fixed $(\alpha, \delta)$.

The target: a construction with $D(p)/K \ge 10 \cdot (D(p)/K)_{\text{v2 baseline}}$ for all relevant $p$, under the same $(\alpha, \delta, T)$, where $K$ includes expected hedging and rebalancing costs (e.g., the cost of maintaining a delta-hedged position via periodic portfolio rebalancing).

Ambiguity note: "tenfold" and "loss-absorbing capital" are not formally defined in the census record; the interpretation above (economic capital at risk, not nominal stake) is the most defensible reading, since concentrated liquidity (Uniswap v3) already achieves arbitrarily large *nominal* depth ratios and the record explicitly says nominal depth doesn't count.

## Why does it matters?

Liquidity is the dominant capital cost of exchanges; if DEXs could deliver the same executable depth with one-tenth the at-risk capital, market-maker inventory costs would collapse, narrowing spreads and letting DEXs compete with centralized exchanges on deep instruments. Resilient (tail-risk-bounded) liquidity also matters systemically: during volatility spikes, undercapitalized liquidity vanishes exactly when it's needed, causing cascading slippage and liquidation spirals in DeFi lending. A principled answer — construction or impossibility — would also settle whether the LVR/adverse-selection losses fundamental to passive AMM provision are an inherent tax or an artifact of design.

## What's already known?

- **Uniswap v3 concentrated liquidity** (Fan et al. 2023 and the underlying design) lets LPs allocate capital to price ranges, increasing nominal depth per staked dollar by unbounded factors; but capital concentrated near spot carries high inventory risk and LVR, so *risk-adjusted* efficiency gains are far smaller. Fan et al. study strategic range placement and find equilibrium liquidity concentrates near the fee-optimal ranges — the game-theoretic reason naive concentration doesn't deliver the moonshot.
- **LVR / loss-versus-rebalancing** (Milionis, Moallemi, Roughgarden, Zhang 2022, "Automated Market Making and Arbitrage Profits in the Presence of Fees" and follow-up LVR work) establishes that a passive LP is effectively short a replication portfolio, and that a delta-hedged LP's losses are a computable function of volatility; this gives the correct definition of loss-absorbing capital and shows fees must cover LVR plus hedging costs.
- **Dynamic/active AMM designs** (e.g., Lobster, market-maker–style AMMs à la Avellaneda–Stoikov adapted on-chain) hedge inventory and reprice risk, improving risk-adjusted returns but at rebalancing costs (gas, execution slippage, discrete hedging) that currently eat most of the gain.
- No published construction achieves 10× risk-adjusted depth including hedging costs; no impossibility result rules it out either. (Verify — the literature is fast-moving.)

## What would count as solving it?

Any of:

1. **Construction (upper bound direction)**: an explicit mechanism/strategy — contract design, dynamic fee schedule, inventory-hedging policy — with a proof (analytic or high-fidelity simulation under a specified stochastic price model, e.g., GBM or jump-diffusion, with fee and gas costs) that $D/K \ge 10\times$ the v2 baseline at matched $(\alpha, \delta, T)$, hedging and rebalancing costs included in $K$.
2. **Lower bound / impossibility**: a theorem showing $\sup$ over all admissible strategies of risk-adjusted depth per economic capital is bounded below $10\times$ (e.g., an LVR-based no-go: executable depth generates arbitrage flow whose hedged losses scale with depth, pinning $D/K$).
3. **Partial**: any nontrivial constant-factor improvement with rigorous accounting (e.g., proven $c\times$ for $c > 2$ under realistic cost models), or resolution of the definitional question (a canonical, referee-accepted definition of loss-absorbing capital for AMMs).

## How to start

1. **Fix the accounting model first.** Read Milionis et al.'s LVR framework and derive the v2 baseline $D/K$ under tail-risk constraint $(\alpha,\delta)$ with hedging costs; everything hinges on this denominator. This is your reference point — get it airtight before designing anything.
2. **Read Fan et al. (2023), § strategic LP behavior**, to understand why equilibrium concentration erodes efficiency: any mechanism rewarding deep ranges attracts adverse-selection-exposed capital that withdraws under stress.
3. **Natural approach**: Avellaneda–Stoikov–style utility-based quoting with on-chain dynamic fees and discrete delta-hedging; compute the depth-per-capital ratio as a function of hedge frequency and cost, and see how far the 10× gap is. The main obstacle is that hedging costs scale with the very depth you're adding — the ratio may be self-limiting, which is also the route to an impossibility result.
4. **Sanity-check tool**: Monte Carlo under jump-diffusion with volatility spikes; check tail losses $\delta$ realistically, since tail-risk constraints are where naive constructions fail.

## Key references

- Fan, Moinari, et al. (verify author list), *Strategic Liquidity Provision in Uniswap V3*, AFT 2023, DOI:10.4230/LIPIcs.AFT.2023.25. — Models LPs choosing liquidity ranges strategically in Uniswap v3 and characterizes equilibrium placement/fee capture of concentrated liquidity. https://doi.org/10.4230/LIPIcs.AFT.2023.25
- Milionis, Moallemi, Roughgarden, Zhang (2022/2023), *Automated Market Making and Arbitrage Profits in the Presence of Fees* / LVR papers. — Derives the loss-versus-rebalancing bound showing passive LP losses equal the cost of replicating the pool with a continuously rebalanced portfolio; the key tool for defining loss-absorbing capital. (No DOI recorded; verify.)
- Avellaneda & Stoikov (2008), *High-frequency trading in a limit order book*, Quantitative Finance 8(3). — The canonical inventory-risk-based market-making model; the natural template for risk-adjusted active liquidity provision. (verify link)
- Uniswap v3 whitepaper (Adams, Zinsmeister, Salem, Keefer, 2021). — Defines concentrated liquidity and the nominal depth multiplication this problem must beat on a risk-adjusted basis.

## Warnings

- **Nominal depth is a trap**: the census explicitly notes concentrated liquidity's large *nominal* depth ratios don't satisfy the target; the denominator is economic capital at risk, including hedging/rebalancing costs. Any claimed 10× using nominal stake as $K$ is invalid.
- **Definitional instability**: "tenfold," "executable depth," and "tail-risk tolerance" are informal here; a referee will attack your definitions as much as your math. State your formalization up front and check sensitivity to reasonable alternatives.
- **Baseline ambiguity**: 10× relative to what — v2 at the same tail risk, or best-known v3 strategies? Compute both.
- **Ignoring adversarial flow**: backtests under benign GBM overstate efficiency; arbitrageurs and toxic flow must be modeled (this is exactly what Fan et al.'s strategic setting captures).
- **Gas/rebalancing costs are regime-dependent**: results can invert between low- and high-cost regimes; report both.

---
*`BCGT-000358` · flagged for review*

## Acceptance guidance

Any of:

1. **Construction (upper bound direction)**: an explicit mechanism/strategy — contract design, dynamic fee schedule, inventory-hedging policy — with a proof (analytic or high-fidelity simulation under a specified stochastic price model, e.g., GBM or jump-diffusion, with fee and gas costs) that $D/K \ge 10\times$ the v2 baseline at matched $(\alpha, \delta, T)$, hedging and rebalancing costs included in $K$.
2. **Lower bound / impossibility**: a theorem showing $\sup$ over all admissible strategies of risk-adjusted depth per economic capital is bounded below $10\times$ (e.g., an LVR-based no-go: executable depth generates arbitrage flow whose hedged losses scale with depth, pinning $D/K$).
3. **Partial**: any nontrivial constant-factor improvement with rigorous accounting (e.g., proven $c\times$ for $c > 2$ under realistic cost models), or resolution of the definitional question (a canonical, referee-accepted definition of loss-absorbing capital for AMMs).

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/abdebcaf-2275-428a-8959-794c0a697b46?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000358.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000358)
