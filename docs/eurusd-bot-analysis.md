# EURUSD Trading Bot Analysis Report

**Date:** 2026-01-26
**Analyst:** BMAD Business Analyst
**Subject:** Lot Size Upgrade Analysis (0.02 → 0.03) with €50 Capital Addition

---

## Executive Summary

After analyzing your trading bot code and MT5 trade history, I discovered a **critical calculation error** in your code comments. The code states "€10.00 profit" for the profit lock, but the **actual profit is approximately €1.00** (or $1.00). This explains why your MT5 history shows profits of €0.85 to €2.84 instead of the expected €10.

---

## Part 1: The Calculation Discrepancy Explained

### Why PowerShell Says "€10" But MT5 Shows €0.85-€2.84

The issue is in the code comment - it's mathematically wrong:

```python
# FROM YOUR CODE (INCORRECT):
MIN_BE_POINTS = 100            # Profit Lock trigger: 100 points (10 pips)
PROFIT_LOCK_POINTS = 50        # Lock 50 points (5 pips = €10 @ 0.02 lot) ← WRONG!
```

### Correct Pip Value Calculation for EURUSD

| Component | Value |
|-----------|-------|
| 1 point | 0.00001 |
| 1 pip | 0.0001 (10 points) |
| 10 points | 1 pip |
| 50 points | 5 pips |
| 100 points | 10 pips |

### Pip Value Per Lot Size (EURUSD)

For EURUSD (quote currency USD):
```
Pip Value = 0.0001 × Lot Size × 100,000 (contract size)
```

| Lot Size | Units | Pip Value (USD) | Pip Value (EUR @ 1.19) |
|----------|-------|-----------------|------------------------|
| 0.01 | 1,000 | $0.10 | €0.084 |
| 0.02 | 2,000 | $0.20 | €0.168 |
| 0.03 | 3,000 | $0.30 | €0.252 |
| 0.04 | 4,000 | $0.40 | €0.336 |
| 0.05 | 5,000 | $0.50 | €0.420 |

---

## Part 2: Actual Profit Calculations (Current 0.02 Lot)

### What Your Bot ACTUALLY Locks/Earns

| Event | Points | Pips | Actual Profit (0.02 lot) |
|-------|--------|------|--------------------------|
| **Profit Lock Trigger** | 100 pts | 10 pips | Triggers at $2.00 / €1.68 profit |
| **SL Moved to Lock** | 50 pts | 5 pips | Guarantees $1.00 / €0.84 |
| **Trailing Start** | 150 pts | 15 pips | Trailing begins at $3.00 / €2.52 |
| **Min TP (200 pts)** | 200 pts | 20 pips | Target: $4.00 / €3.36 |

### Verification Against Your MT5 History

Looking at your attached screenshot trades:

| Trade Time | Profit (€) | Estimated Pips | Analysis |
|------------|------------|----------------|----------|
| 2026.01.23 20:11:18 | 0.85 | ~4.3 pips | Stopped before BE trigger |
| 2026.01.23 20:33:00 | 0.85 | ~4.3 pips | Stopped before BE trigger |
| 2026.01.26 10:16:26 | 2.51 | ~12.6 pips | Profit Lock + some trailing |
| 2026.01.26 10:42:04 | -1.70 | ~-8.5 pips | Stop Loss hit |
| 2026.01.26 11:46:15 | -1.69 | ~-8.5 pips | Stop Loss hit |
| 2026.01.26 12:11:21 | 2.35 | ~11.8 pips | Profit Lock active |
| 2026.01.26 15:16:48 | 2.34 | ~11.7 pips | Profit Lock active |
| 2026.01.26 15:55:36 | 2.29 | ~11.5 pips | Profit Lock active |
| 2026.01.26 16:45:45 | 2.84 | ~14.2 pips | Best trade, trailing worked |
| 2026.01.26 17:39:03 | -1.94 | ~-9.7 pips | Stop Loss hit |
| 2026.01.26 18:08:02 | -1.70 | ~-8.5 pips | Stop Loss hit |
| 2026.01.26 18:17:32 | 3.26 | ~16.3 pips | Total profit shown |

**Conclusion:** Your actual results MATCH the correct calculation ($0.20/pip), NOT the €10 claim.

---

## Part 3: Upgrade Analysis (0.02 → 0.03 Lot)

### Proposed Setup

| Parameter | Current (0.02) | Proposed (0.03) | Change |
|-----------|----------------|-----------------|--------|
| Capital | €103.26 | €153.26 (+€50) | +48.4% |
| Lot Size | 0.02 | 0.03 | +50% |
| Pip Value | $0.20 / €0.168 | $0.30 / €0.252 | +50% |

### New Profit/Loss Calculations at 0.03 Lot

| Event | Points | Pips | Profit at 0.03 lot |
|-------|--------|------|---------------------|
| **Profit Lock Trigger** | 100 pts | 10 pips | $3.00 / €2.52 |
| **Guaranteed Lock** | 50 pts | 5 pips | $1.50 / €1.26 |
| **Trailing Start** | 150 pts | 15 pips | $4.50 / €3.78 |
| **Min TP (200 pts)** | 200 pts | 20 pips | $6.00 / €5.04 |
| **Stop Loss (100 pts)** | -100 pts | -10 pips | -$3.00 / -€2.52 |

---

## Part 4: Simulated Trade Examples (0.03 Lot)

### Scenario A: Perfect Trade (TP Hit)

```
ENTRY: BUY EURUSD @ 1.18500
SL: 1.18400 (100 points / 10 pips below)
TP: 1.18700 (200 points / 20 pips above)

Timeline:
├─ [ENTRY] Price: 1.18500 | P/L: €0.00
│
├─ Price rises to 1.18550 (+50 pts / +5 pips)
│  P/L: +€1.26 (unrealized)
│  Status: Watching...
│
├─ Price rises to 1.18600 (+100 pts / +10 pips) ← PROFIT LOCK TRIGGERS
│  P/L: +€2.52 (unrealized)
│  ✅ SL MOVED to 1.18550 (locking 50 pts / 5 pips)
│  Guaranteed minimum: €1.26
│
├─ Price rises to 1.18650 (+150 pts / +15 pips) ← TRAILING STARTS
│  P/L: +€3.78 (unrealized)
│  📈 Trailing SL follows price
│
├─ Price hits 1.18700 (+200 pts / +20 pips) ← TP HIT!
│  ✅ TRADE CLOSED
│  💰 FINAL P/L: +€5.04 (+$6.00)
```

### Scenario B: Profit Lock Saves the Day

```
ENTRY: BUY EURUSD @ 1.18500
SL: 1.18400 | TP: 1.18700

Timeline:
├─ [ENTRY] Price: 1.18500
│
├─ Price rises to 1.18600 (+100 pts) ← PROFIT LOCK TRIGGERS
│  ✅ SL MOVED to 1.18550 (locking €1.26)
│
├─ Price rises to 1.18650 (+150 pts)
│  📈 Trailing SL moves to 1.18570
│
├─ ⚠️ NEWS EVENT - Price reverses!
│
├─ Price drops to 1.18580
│  Trailing SL still at 1.18570
│
├─ Price drops to 1.18570 ← TRAILING SL HIT
│  ✅ TRADE CLOSED
│  💰 FINAL P/L: +€1.76 (+$2.10)
│  (70 points profit captured)

WITHOUT PROFIT LOCK: Would have lost -€2.52 at original SL!
```

### Scenario C: Stop Loss Hit (Full Loss)

```
ENTRY: BUY EURUSD @ 1.18500
SL: 1.18400 | TP: 1.18700

Timeline:
├─ [ENTRY] Price: 1.18500
│
├─ Price drops to 1.18450 (-50 pts)
│  P/L: -€1.26 (unrealized)
│  Status: Holding...
│
├─ Price drops to 1.18400 (-100 pts) ← SL HIT
│  ✅ TRADE CLOSED
│  💸 FINAL P/L: -€2.52 (-$3.00)
```

---

## Part 5: Risk Profile Matrix (BMAD Methodology)

### Risk Assessment (Probability × Impact)

| Risk ID | Description | Probability | Impact | Score | Priority |
|---------|-------------|-------------|--------|-------|----------|
| FIN-001 | Increased loss per trade (-€2.52 vs -€1.68) | High (3) | Medium (2) | 6 | HIGH |
| FIN-002 | Margin call risk (higher margin required) | Low (1) | High (3) | 3 | LOW |
| FIN-003 | Faster capital depletion on losing streak | Medium (2) | High (3) | 6 | HIGH |
| OPS-001 | Code comment confusion on actual values | High (3) | Low (1) | 3 | LOW |
| BUS-001 | Psychological impact of larger losses | Medium (2) | Medium (2) | 4 | MEDIUM |

### Margin Requirement Analysis

For XM Ultra Low Standard (30:1 leverage on EURUSD):

```
Margin = (Lot Size × Contract Size × Price) / Leverage
```

| Lot Size | Contract Value | Margin Required (30:1) | % of €153.26 |
|----------|----------------|------------------------|--------------|
| 0.01 | €1,000 | €33.33 | 21.7% |
| 0.02 | €2,000 | €66.67 | 43.5% |
| 0.03 | €3,000 | €100.00 | 65.2% |
| 0.04 | €4,000 | €133.33 | 87.0% |

**Analysis:** At 0.03 lot with €153.26 capital, you'll use ~65% margin per trade. This is aggressive but manageable for a single position.

---

## Part 6: Profit/Loss Comparison Table

### Daily Scenario Projections

Assuming 10 trades per day with your current win rate (~60% from screenshot):

| Metric | Current (0.02) | Proposed (0.03) | Difference |
|--------|----------------|-----------------|------------|
| Win Rate | 60% | 60% | - |
| Avg Win | €2.00 | €3.00 | +50% |
| Avg Loss | €1.70 | €2.55 | +50% |
| 6 Wins | €12.00 | €18.00 | +€6.00 |
| 4 Losses | -€6.80 | -€10.20 | -€3.40 |
| **Daily P/L** | €5.20 | €7.80 | +€2.60 |
| **Weekly P/L** | €26.00 | €39.00 | +€13.00 |

### Worst Case: 5 Consecutive Losses

| Scenario | Current (0.02) | Proposed (0.03) |
|----------|----------------|-----------------|
| 1 Loss | -€1.70 | -€2.55 |
| 2 Losses | -€3.40 | -€5.10 |
| 3 Losses | -€5.10 | -€7.65 |
| 4 Losses | -€6.80 | -€10.20 |
| 5 Losses | -€8.50 | -€12.75 |
| Remaining Capital | €94.76 | €140.51 |
| % Drawdown | 8.2% | 8.3% |

---

## Part 7: Code Correction Recommendation

The print statement in your code should be corrected:

### Current (Incorrect):
```python
log_msg(f"🛡️ PROFIT LOCK: Secured €10.00 ({PROFIT_LOCK_POINTS} pts) @ {new_sl:.5f}")
```

### Corrected Version:
```python
# Calculate actual locked profit
pip_value = VOLUME * 100000 * 0.0001  # $0.20 for 0.02, $0.30 for 0.03
locked_profit = (PROFIT_LOCK_POINTS / 10) * pip_value  # Convert points to pips
log_msg(f"🛡️ PROFIT LOCK: Secured ${locked_profit:.2f} ({PROFIT_LOCK_POINTS} pts) @ {new_sl:.5f}")
```

---

## Part 8: Final Recommendation

### Go/No-Go Decision Matrix

| Factor | Assessment | Recommendation |
|--------|------------|----------------|
| Capital Adequacy | €153.26 vs €100 margin | ✅ ADEQUATE |
| Win Rate (60%) | Profitable | ✅ GOOD |
| Risk per Trade | 1.7% of capital | ✅ ACCEPTABLE |
| Margin Usage | 65% per trade | ⚠️ MODERATE |
| Psychological | Can handle €2.55 losses? | ❓ DEPENDS |

### Final Verdict: CONDITIONAL GO

**Proceed with 0.03 lot IF:**
1. You understand the actual profits are €1.26-€5.04 range, NOT €10
2. You're comfortable with €2.55 max loss per trade
3. You keep DAILY_MAX_LOSS_EUR at €40 or increase to €50
4. You understand 5 consecutive losses = €12.75 drawdown

### Suggested Parameter Adjustments for 0.03 Lot

```python
VOLUME = 0.03                  # Upgraded lot size
DAILY_MAX_LOSS_EUR = 50.0      # Increase daily limit proportionally
DAILY_MAX_TRADES = 15          # Allow more trades for recovery
```

---

## Appendix: Quick Reference Card

### At 0.03 Lot Size

| Event | Points | Pips | USD | EUR (~) |
|-------|--------|------|-----|---------|
| Profit Lock Trigger | 100 | 10 | $3.00 | €2.52 |
| **Guaranteed Min Profit** | 50 | 5 | **$1.50** | **€1.26** |
| Trailing Activation | 150 | 15 | $4.50 | €3.78 |
| Target TP | 200 | 20 | $6.00 | €5.04 |
| Max Stop Loss | -100 | -10 | -$3.00 | -€2.52 |

---

*Report generated using BMAD-METHOD™ Risk Analysis Framework*
