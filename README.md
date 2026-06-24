# BSM Option Pricer

A Streamlit app that prices US equity options using the Black-Scholes-Merton
model, with implied volatility independently solved from live market quotes
(not Yahoo Finance's pre-computed IV).

## Setup

```bash
pip install -r requirements.txt
streamlit run app.py
```

## How it works

1. **Ticker resolution** — type a company name ("Apple") or ticker ("AAPL").
   The app searches Yahoo Finance and lets you disambiguate if multiple
   matches are found (e.g. different share classes / exchanges).
2. **Market data** — spot price, dividend yield, and listed option
   expirations are pulled live via `yfinance` and cached (`@st.cache_data`)
   to keep repeated lookups fast.
3. **Implied volatility** — for the two listed expirations that bracket your
   requested expiration date, the app solves implied volatility **per
   strike** from real bid/ask quotes using Brent's method against the BSM
   formula (Yahoo's own `impliedVolatility` field is intentionally ignored).
   It then:
   - interpolates within each expiry's smile to your exact strike, then
   - linearly interpolates between the two expiries' smiles to your exact
     requested date.
4. **Pricing** — the interpolated IV, along with spot, strike, time to
   expiry, risk-free rate, and dividend yield, is fed into the closed-form
   Black-Scholes-Merton formula to produce the theoretical price, Greeks,
   the IV smile chart, and a payoff diagram.

## Files

- `app.py` — Streamlit UI (dark theme, input panel, results panel, charts).
- `bsm_engine.py` — all financial math and Yahoo Finance data access,
  independent of Streamlit (testable on its own).

## Error handling

The app distinguishes and gives clear messages for: unresolvable
company/ticker, invalid or implausible strike prices, past/invalid
expiration dates, missing option data (e.g. no listed options, empty chain
for a given expiry, insufficient liquid quotes to solve IV), and
network/API failures — without ever surfacing a raw traceback.

## Disclaimer

For educational/analytical purposes only. Not investment advice.
