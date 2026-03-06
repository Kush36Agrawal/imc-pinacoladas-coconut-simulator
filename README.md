# PEARLS Trading Simulator

A browser-based trading simulator inspired by **IMC Prosperity Round 1**.
Students upload historical order book data and their trading strategy to simulate trading against market participants.

The simulator runs entirely in the **browser** — no backend required.

---

## Overview

This project simulates a **limit order book environment** for the product **PEARLS**.

At each timestep:

1. The simulator loads the **order book snapshot** from CSV data.
2. **Bots generate orders** based on their strategies.
3. The **student strategy function** generates bid and ask quotes.
4. All orders enter a **shared order book**.
5. A **matching engine executes trades**.
6. The system updates **cash, position, and PnL**.

PnL is calculated as:

```
PnL = Cash + Position × Fair Value
```

The **fair value model is hidden**, encouraging students to infer it from trading behaviour.

---

## Features

* Full **limit order book simulation**
* **Continuous matching engine**
* **Partial fills supported**
* **Position limit enforcement (±20)**
* **Multiple trading bots**
* **Student order priority at same price level**
* Real-time:

  * PnL chart
  * Position chart
  * Trade log
  * Order book visualization

Runs completely in the **browser using JavaScript**.

---

## Trading Bots

Four automated participants provide liquidity and noise:

### MM Alpha

Market maker quoting around a volume-weighted fair price.

### MM Beta

Market maker that adjusts prices using order flow imbalance.

### Noise Alpha

Random trader producing unpredictable order flow.

### Noise Beta

Mean-reverting trader around an anchor price.

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

## Position Limits

Student positions are constrained:

```
-20 ≤ Position ≤ +20
```

Orders exceeding limits are automatically reduced.

---

## Required Data

Students must upload the provided CSV datasets:

```
prices_round_1_day_-2.csv
prices_round_1_day_-1.csv
prices_round_1_day_0.csv
prices_round_1_day_1.csv
prices_round_1_day_2.csv
```

Each row represents a **snapshot of the order book**.

Example columns:

```
timestamp
product
bid_price_1
bid_volume_1
bid_price_2
bid_volume_2
bid_price_3
bid_volume_3
ask_price_1
ask_volume_1
ask_price_2
ask_volume_2
ask_price_3
ask_volume_3
```

---

## Strategy Interface

Students implement a function:

```python
def strategy(row, position, step):
```

### Parameters

| Parameter | Description                 |
| --------- | --------------------------- |
| row       | Current order book snapshot |
| position  | Current position in PEARLS  |
| step      | Current timestep index      |

### Return format

```python
{
    "bid": int,
    "ask": int,
    "bid_qty": int,
    "ask_qty": int
}
```

Example strategy:

```python
def strategy(row, position, step):
    best_bid = row["bid_price_1"]
    best_ask = row["ask_price_1"]
    mid = (best_bid + best_ask) / 2

    return {
        "bid": int(mid) - 1,
        "ask": int(mid) + 1,
        "bid_qty": 3,
        "ask_qty": 3
    }
```

---

## How to Run

1. Open the simulator website.
2. Upload the CSV price files.
3. Upload your `strategy.py` file.
4. Click **Run Simulation**.

The system will simulate trades and display:

* Order book state
* Trade executions
* Position changes
* PnL evolution

---


