# Live Implied Volatility Surface

A real-time 3D implied volatility surface visualizer built on the **Interactive Brokers TWS API**. The app streams live option market data, computes an IV grid across strikes and expirations, and renders an interactive, continuously updating surface plot alongside a front-month skew chart.

## Features

- **Live data streaming** — connects to TWS/IB Gateway and subscribes to real-time option implied volatility (model computation, tick type 106)
- **Automatic option chain resolution** — resolves the underlying contract, fetches the full option chain, and selects the 6 nearest expirations with strikes within ±2% of spot
- **3D surface plot** — renders the IV surface (strike × expiry × IV) with a dark theme and `magma` colormap, refreshed every 0.5 seconds
- **Front-month skew panel** — plots the nearest-expiry volatility smile with the current spot price marked
- **Interactive controls** — rotate/zoom the 3D surface freely; camera angle is preserved between refreshes. A **Lock Updates** button freezes the plot so you can inspect the surface without it redrawing
- **Smart contract selection** — requests calls above spot and puts below spot (OTM options) for cleaner IV data
- **Linear interpolation** — fills gaps in the IV grid across expirations for a smooth surface

## Preview

The window displays two panels:

| Panel | Description |
|---|---|
| 3D Surface (left) | Live implied volatility surface across strikes and expirations |
| Skew Chart (right) | Front-month volatility smile with spot price reference line |

## Requirements

- Python 3.8+
- [Interactive Brokers TWS](https://www.interactivebrokers.com/en/trading/tws.php) or IB Gateway, running and logged in
- An IBKR account with **market data subscriptions** for the underlying and its options
- Python packages:

```
ibapi
pandas
numpy
matplotlib
```

> **Note:** `ibapi` is distributed by Interactive Brokers. Install it from the [official TWS API download](https://interactivebrokers.github.io/) or via `pip install ibapi`.

## Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/LarinArtem/volatility_surface.git
   cd volatility_surface
   ```

2. **Install dependencies**

   ```bash
   pip install ibapi pandas numpy matplotlib
   ```

3. **Configure TWS / IB Gateway**

   - Open TWS and go to `File → Global Configuration → API → Settings`
   - Enable **"Enable ActiveX and Socket Clients"**
   - Confirm the socket port matches the script (default: **7497** for paper trading; use 7496 for live)
   - Add `127.0.0.1` to trusted IPs if prompted

## Usage

Run the script with TWS open:

```bash
python project_6.py
```

By default it tracks **SPY**. To visualize a different underlying, change the symbol in the entry point:

```python
if __name__ == '__main__':
    app_instance = start_app("AAPL")  # any US stock symbol
```

The app will:

1. Connect to TWS on `127.0.0.1:7497` (client ID 35)
2. Resolve the underlying contract and fetch the live spot price
3. Pull the option chain and subscribe to IV data for near-the-money strikes across the next 6 expirations
4. Launch the live plotting window after a ~10 second warm-up

Press `Ctrl+C` in the terminal to disconnect and close the plot.

## Configuration

Key parameters you can tweak in `project_6.py`:

| Parameter | Location | Default | Description |
|---|---|---|---|
| `symbol` | `start_app()` | `"SPY"` | Underlying ticker |
| Port | `app.connect(...)` | `7497` | TWS API port (7496 = live, 7497 = paper) |
| Expirations | `target_exps` | `[:6]` | Number of expirations to include |
| Strike range | `target_strikes` | `±2%` of spot | Moneyness window for strikes |
| Refresh rate | `plt.pause(.5)` | `0.5s` | Plot update interval |

## How It Works

- **`LiveSurfaceApp`** — combines `EClient` and `EWrapper` from the IB API. Callbacks capture the underlying contract ID, spot price, option chain parameters, and streaming implied volatilities into an in-memory dictionary.
- **`start_app()`** — orchestrates the connection, contract resolution, and market data subscriptions on a background thread.
- **`live_desktop_plot()`** — the main render loop. Pivots the collected IV data into an expiry × strike grid, interpolates missing values, and redraws the 3D surface and skew chart every half second (unless locked).

## Disclaimer

This project is for **educational purposes only** and does not constitute financial advice. Market data usage is subject to your Interactive Brokers data subscriptions and agreements.
