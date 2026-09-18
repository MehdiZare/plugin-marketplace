---
name: bina-options
description: Use when the user asks Bina Capital about options on a U.S. stock or ETF. That covers what to sell for premium today, which cash-secured puts or credit spreads rank best, how to read Bina's physical rank_key, edge or agreement fields, whether implied volatility is rich or cheap, how big a move is priced in around earnings, option ideas for a bullish or bearish view, and Bina volatility posts.
---

Use the connected Bina Options MCP tools. Everything they return is model-based
education over delayed (about 15 minutes) listed-options data. It is read-only:
there is no order placement, brokerage or position state. Never present a
result as investment advice, a prediction, or a guaranteed profit.

## Connect and sign in

The plugin connects to `https://mcp.bina.capital/mcp`. Sign in through the
client's browser OAuth flow with a Bina account; the scopes are `options:read`
and `offline_access`. Never ask the user for passwords, API keys or tokens in
chat. An expired or revoked connection returns to the sign-in flow.

## Pick the tool

| The user wants | Call |
|---|---|
| What to sell for premium, best first | `search_premium_structures` (only where the server enables it; if `list_tools` does not show it, say so and do not simulate it) |
| Ideas for a bullish or bearish view on one ticker | `get_enhancements_for_ticker`, then `compare_to_stock_buy` for the buy-100-shares baseline |
| Is implied volatility rich or cheap | `get_iv_context` |
| How big a move is priced in | `get_expected_move`; pass `earnings_date` when you know the print |
| Market-wide volatility or earnings posts | `get_volatility_standouts`, `get_volatility_dispersion`, `get_expected_moves` |

Read the resource `bina://guide` for the full field reference, and
`bina://options/strategy/{name}` to explain any strategy in plain English.

## What can I sell today

1. Get the symbol, the view (`bullish`, `bearish` or `neutral`), the longest
   expiry the user will hold (`dte_max`) and their broker `approval_level`.
   Cash-secured puts need approval 1 or more and a `bullish` view; spreads and
   condors need 3. Ask only for what you are missing.
2. Call `search_premium_structures` and keep the default ranking
   (`rank_by: short_horizon_path`). The list comes back best first by
   `physical.rank_key`: the models' annualised expected return on the capital
   the trade ties up (the strike for a cash-secured put, the widest wing for a
   spread).
3. For each structure you show, give the strikes and expiry, the credit,
   `physical.rank_key` as an annualised percentage, `physical.edge` (dollars
   per share, the most pessimistic model), `physical.edge_high` (the most
   optimistic), `physical.agreement`, and the risk-neutral
   `risk_metrics.prob_profit` beside them.
4. Read `physical.agreement`:
   - `corroborated`: the mean-reversion model and the volatility models agree
     on the direction of the edge.
   - `contested`: they disagree. Say so, and say which model dissents
     (`physical.components`).
   - `single_view`: only the volatility models priced it. They share one
     volatility forecast, so their agreement is not independent evidence.
5. Short expiries produce very large annualised returns. Report them as they
   are, never capped or smoothed, and say whether each one is corroborated. An
   error in the edge is magnified by the same factor.
6. On a put with `physical.wheel` set, the rank assumes a covered call the user
   has not sold. Quote `physical.wheel.rank_key_untilted` for the put alone.
7. Check each structure's `flags`. `assignment_risk` and
   `earnings_before_expiry` matter most to a seller.

## Absence is a state

- A model named in `physical.withheld` did not price the trade. It is unpriced,
  not zero; the reason says why (for example `no_significant_reversion`, which
  is the normal case on most names most days).
- `physical.rank_key` null means no model could price the trade. If
  `physical_context.dividends.status` is `unknown`, nothing on that symbol is
  priced today. Say the models could not price it; do not substitute another
  metric.
- `available: false`, `valid: false`, `earnings_window: "unknown"` and the flag
  `earnings_window_unknown` mean the answer could not be established. They are
  not an all-clear, not "none", and not zero. Only `earnings_window: "clear"`
  is a positive all-clear.

## Presenting results

- Carry every `note`, `caveat` and `disclosures` field verbatim.
- For the market-wide packs, check `valid` and `skip_reason` first, and render
  the `display` strings verbatim instead of reformatting raw numbers.
- Call the physical numbers the models' opinion, not a fair value, mispricing
  or alpha.
- Mirror the tools' neutral `citations`. Do not name the underlying data
  vendors.
- Keep a short table for several trades, one row per structure, in the order
  the tool returned them.
