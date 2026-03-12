# Black '76 Commodity Options Pricing Engine

A Python implementation of the Black '76 model for pricing options on commodity futures.
Built and tested against live MCX options data.

---

## What This Does

This notebook prices European options on commodity futures, including:
- Call and Put option pricing using the Black '76 model
- Implied Volatility solver using Brent's method
- First-order Greeks: Delta, Gamma, Vega, Theta
- Payoff profile visualisation
- Put-Call parity verification

---

## Why Black '76 — Not Black-Scholes

Black-Scholes prices options on spot assets like stocks. For commodities, we trade
**futures contracts**, not the spot price directly.

A futures price already has the cost of carry built into it — storage costs,
interest rates, convenience yield. This means futures have **zero drift** under
the risk-neutral measure. There is no need to add the risk-free rate `r` inside d1.

Black '76 accounts for this by removing `r` from d1:

$$d_1 = \frac{\ln(F/K) + \frac{\sigma^2}{2} T}{\sigma \sqrt{T}}$$

Applying Black-Scholes to commodity futures would be incorrect because it would
add drift that does not exist in futures pricing.

---

## Why Brent's Method — Not Newton-Raphson

Implied Volatility is the only input to Black '76 not directly observable from
the market. We back it out by finding the sigma that makes model price = market price.

Newton-Raphson is the standard root-finding method, but it divides by Vega at
each iteration:

$$\sigma_{n+1} = \sigma_n - \frac{Black76(\sigma_n) - C_{market}}{Vega(\sigma_n)}$$

For deep ITM or deep OTM options, **Vega approaches zero**. The denominator blows
up, causing Newton-Raphson to produce unstable or non-convergent results.

**Brent's method** is derivative-free. It brackets the root between
`[sigma_low, sigma_high]` and is guaranteed to converge as long as the root lies
within the bracket. This makes it robust where Newton-Raphson fails.

---

## Put-Call Parity Verification

For Black '76, Put-Call parity states:

$$C - P = (F - K) \cdot e^{-rT}$$

If this relationship holds, the model is **arbitrage-free** — no trader can
extract risk-free profit from our prices. If parity were violated, a trader
could simultaneously buy and sell the call, put, and futures contract to lock
in guaranteed profit, which is impossible in an efficient market.

Our implementation passes this check, confirming the model is internally consistent.

---

## Sample Output

```
===================================
  BLACK '76 COMMODITY ANALYSIS
===================================
Underlying (F):       5819.00
Market Price:          274.20
Implied Vol (IV):       55.57%
-----------------------------------
Delta:                 0.6063
Gamma:               0.000723
Vega (1% move):        3.6711
Theta (per day):     -10.3875
===================================

========================================
  PUT-CALL PARITY CHECK
========================================
  C - P           : 118.8395
  (F-K)*e^(-rT)   : 118.8395
  Error           : 4.83e-13
  PASSED — Model is correct
========================================
```

---

## Tech Stack

- Python 3.x
- `numpy` — numerical computation
- `scipy.stats.norm` — normal distribution CDF/PDF
- `scipy.optimize.brentq` — Brent's root-finding method
- `matplotlib` — payoff profile visualisation

---

## Model Assumptions & Limitations

- **European options only** — does not handle early exercise
- **Constant volatility** — real markets show volatility smile/skew
- **Log-normal futures returns** — fat tails and jumps are not captured
- **No transaction costs** — real execution involves slippage and commissions

---

## Related Projects

- [Black-Scholes Options Pricing](https://github.com/devil0026/black_scholes)
- Pairs Trading — Walk-Forward Statistical Arbitrage Engine

---

## Author

**Rakshit Gandhi**
B.Sc. Information Technology | GLSU Ahmedabad
Focused on quantitative finance, derivatives pricing, and systematic strategy development.
[rakshitgandhi184@gmail.com](mailto:rakshitgandhi184@gmail.com)
