# PAIRS Trading Simulator (COCONUTS & PINA_COLADAS)

A browser-based trading simulator inspired by **IMC Prosperity Round 2**.
Students upload historical order book data and their trading strategy to simulate trading against market participants.

The simulator runs entirely in the **browser** — no backend required.

---

## Overview

This project simulates a **limit order book environment** for two assets: **COCONUTS** and **PINA_COLADAS**.

At each timestep:

1. The simulator loads the **order book snapshot** for both assets from CSV data.
2. **5 Bots** generate orders based on their strategies (including a StatArb bot trading the spread).
3. The **student strategy function** generates bid and ask quotes for both assets.
4. All orders enter their respective **shared order book**.
5. A **matching engine executes trades**.
6. The system updates **cash, positions, and evaluated PnL**.

PnL is continuously evaluated at the current market mid-prices:

```
PnL = Cash + (Coconut Position × Coconut MidPrice) + (Pina Position × Pina MidPrice)
```

---

## Features

* Full **limit order book simulation** for dual assets
* **Continuous matching engine**
* **Position limit enforcement**:
  * COCONUTS: ±600 
  * PINA_COLADAS: ±300
* **Multiple trading bots** including a competitive statistical arbitrageur
* **Student order priority at same price level**
* Real-time:
  * Dual Mid-Price Comparison Chart
  * Dual Position Chart
  * Trade log
  * Dual Order book visualization

Runs completely in the **browser using JavaScript**.

---

## Trading Bots

Five automated participants provide liquidity, noise, and competition:

### MM Alpha
Market maker quoting around a volume-weighted fair price.

### MM Beta
Market maker that adjusts prices using order flow imbalance.

### Noise Alpha
Random trader producing unpredictable order flow.

### Noise Beta
Mean-reverting trader around a short-term moving average.

### StatArb Alpha 
An advanced strategy bot that monitors the cointegrated spread between Coconuts and Pina Coladas. If it spots a pricing inefficiency, it aggressively crosses the spread to capture arbitrage, forcing the student to compete for fills.

Bots trade with **each other, the market book, and the student**.

---

## Matching Engine

All agents interact in a shared order book.

Orders are sorted by:

```
Price
Priority
```

Priority at the same price:

```
Student > Bots > Market
```

Matching rule:

```
while best_bid >= best_ask:
    execute trade
```

Trades support **partial fills**.

---

## Required Data

Students must upload the provided Round 2 CSV datasets:

```
prices_round_2_day_-1.csv
prices_round_2_day_0.csv
prices_round_2_day_1.csv
```

Each row represents a **snapshot of the order book**.

---

## Strategy Interface

Students implement a function:

```python
def strategy(coconut_row, pina_row, pos_coconut, pos_pina, step):
```

### Parameters

| Parameter      | Description |
| -------------- | ----------- |
| coconut_row    | Dictionary representing current order book for Coconuts |
| pina_row       | Dictionary representing current order book for Pina Coladas |
| pos_coconut    | Current position in Coconuts (Max ±600) |
| pos_pina       | Current position in Pina Coladas (Max ±300) |
| step           | Current timestep index |

### Return format

The function MUST return a dictionary with keys `"COCONUTS"` and `"PINA_COLADAS"`. 
(Omit `bid`/`ask` keys to take no action for an asset).

```python
{
    "COCONUTS": {
        "bid": int,
        "ask": int,
        "bid_qty": int,
        "ask_qty": int
    },
    "PINA_COLADAS": {
        "bid": int,
        "ask": int,
        "bid_qty": int,
        "ask_qty": int
    }
}
```

Example strategy (Naive Pairs Trading):

```python
def strategy(coconut_row, pina_row, pos_coconut, pos_pina, step):
    orders = {"COCONUTS": {}, "PINA_COLADAS": {}}
    
    if not coconut_row or not pina_row:
        return orders
        
    c_mid = (coconut_row["bid_price_1"] + coconut_row["ask_price_1"]) / 2
    p_mid = (pina_row["bid_price_1"] + pina_row["ask_price_1"]) / 2
    
    # 15/8 ratio -> 1.875
    spread = p_mid - 1.875 * c_mid 
    
    if spread > 100:
        # Pina is overvalued, Coconuts undervalued
        orders["PINA_COLADAS"] = {"ask": p_row["bid_price_1"], "ask_qty": 30}
        orders["COCONUTS"] = {"bid": c_row["ask_price_1"], "bid_qty": 50}
    elif spread < -50:
        # Pina is undervalued, Coconuts overvalued
        orders["PINA_COLADAS"] = {"bid": p_row["ask_price_1"], "bid_qty": 30}
        orders["COCONUTS"] = {"ask": c_row["bid_price_1"], "ask_qty": 50}

    return orders
```

---

## How to Run

1. Open the simulator website (`index.html`).
2. Upload the CSV price files.
3. Upload your `strategy.py` file.
4. Click **Run Simulation**.

The system will simulate trades and display:

* Dual Order books
* Trade executions
* Position changes
* Mid Price charts
