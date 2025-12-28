# PancakeSwap Prediction Viewer

This is a **view-only monitoring tool** for the PancakeSwap prediction market - it watches and reports, but doesn't place bets.

## Core Purpose
Monitors prediction rounds in real-time and sends Telegram alerts for important events like winning streaks and big price movements.

## Key Components

### 1. **Data Collection & Caching**

**Round History Tracking:**
- Fetches and caches last 24 completed rounds
- Stores: prices, winners, payouts, max bets
- Saves to `rounds_cache.json` to avoid redundant blockchain queries
- Only fetches new rounds, uses cache for existing data

**Live Monitoring:**
- Tracks current round's betting activity
- Shows bull/bear percentages, pool size, whale counts
- **Smart RPC optimization**: Only fetches bet data in last 25 seconds to save API calls

### 2. **Price Tracking**

- **Live BNB Price**: Updates from Chainlink oracle every 10 seconds (cached)
- **Price Direction**: Shows change from last round's close price
  - 🟩 Green for gains
  - 🟥 Red for losses
  - Shows exact USDT difference

### 3. **Telegram Notification System**

The bot sends automatic alerts for two scenarios:

#### **Streak Alerts (4+ consecutive wins)**
Triggers when same side wins 4+ rounds in a row:
```
🚨 STREAK ALERT! 🚨
🟢 BULL STREAK: 6 wins in a row!

💰 Current BNB: $645.2340 📈 +0.8750
📊 24-Round Summary: BULL: 15 | BEAR: 9
📊 Last 10 rounds: [shows recent results]
⚠️ Consider the streak when analyzing patterns!
```

#### **Big Price Movement Alerts (>$1.75)**
Triggers when price moves more than $1.75 in one round:
```
🚨 BIG PRICE MOVEMENT! 🚨
📈 Round 12345: BNB moved UP by $2.45!

💰 Current BNB: $647.6890 📈 +3.3250
📊 Details:
Open: $645.2400
Close: $647.6900
Winner: 🟩 BULL
💰 Payouts: Bull 1.85x | Bear 2.30x
```

**Smart Features:**
- 3-second delay before sending to avoid spam
- Includes 24-round summary in every notification
- Tracks last notified streak to avoid duplicates
- Only notifies when streak increases or changes direction

### 4. **Display Features**

**24-Round History Table:**
Shows comprehensive data for last 24 rounds:
- Open/Close prices
- Price changes with direction emoji
- Payouts for both sides
- Max bets on each side
- Winner with color coding
- Total pool size

**ADHD-Friendly Summary Box:**
Quick stats at a glance:
- Bull wins vs Bear wins
- Overall price trend (first → last)
- Current streak information

**Real-Time Display:**
Two-line updating display:
```
🟢 TIME: 237s | 💰 Live BNB: $645.2340 🟩 +0.8750
📊 Bull: 52.3% | Bear: 47.7% | Ratio: 1.10 | Pool: 8.4523 | ML: 0.235
```

Color-coded timer:
- 🟢 Green: >30 seconds
- 🟡 Yellow: 10-30 seconds  
- 🔴 Red: <10 seconds

### 5. **ML Prediction Score**

Calculates a prediction score (-1 to 1) based on:
- Bet ratio patterns
- Pool volatility
- Time-based patterns (hourly bull win rates)
- Whale activity
- Total pool size

**Not used for betting**, just displayed for reference.

### 6. **Optimization Features**

**RPC Call Reduction:**
- Only fetches bet data in last 25 seconds of round
- Before 25s: Shows only timer and price
- Caches price data (10-second refresh)
- Limits blockchain queries to recent blocks

**Efficient Updates:**
- Uses ANSI escape codes to update same lines (no scrolling)
- Single-line cursor movement for clean display
- Minimal screen refresh

### 7. **Main Loop Flow**

```
Start → Fetch 24 rounds history → Display table →
Wait for round → Monitor live (update every 1s) →
Round ends → Check for alerts → Send notifications →
Wait 170s → Repeat
```

**Round Monitoring Phases:**
1. **Early Phase** (>25s remaining): Show timer + price only
2. **Active Phase** (≤25s remaining): Show full betting stats
3. **End Phase**: Round completes, fetch new data, check alerts

### 8. **Alert Detection Logic**

**Streak Detection:**
```python
if current_streak >= 4 and current_streak > last_notified_streak:
    send_telegram_alert()
    last_notified_streak = current_streak
```

**Price Movement Detection:**
```python
if abs(price_change_usdt) > 1.75:
    send_telegram_alert()
```

## Key Differences from Betting Bot

| Feature | Betting Bot | Viewer Bot |
|---------|-------------|------------|
| **Purpose** | Places bets | Monitors only |
| **Wallet** | Required | Not needed |
| **Private Key** | Required | Not needed |
| **Transactions** | Sends txs | Read-only |
| **RPC Usage** | Heavy | Optimized |
| **Notifications** | Bet confirmations | Streaks & movements |
| **Data Frequency** | Continuous | Last 25s only |

## Use Cases

1. **Pattern Analysis**: Study winning patterns without risking funds
2. **Streak Awareness**: Get alerted to hot/cold streaks
3. **Price Volatility**: Monitor unusual price movements
4. **Strategy Research**: Observe bet ratios and whale behavior
5. **Multi-Bot Coordination**: Share alerts with multiple betting bots

## Example Output

```
🚀 PANCAKESWAP PREDICTION VIEWER
📊 LAST 24 ROUNDS HISTORY
🔥 CURRENT STREAK: 🟢 BULL x5
============================================
Round    Open USDT    Close USDT   Change    Winner   Pool
52345    $645.2340   $645.8920    📈+0.66   🟩BULL   12.453
52346    $645.8920   $644.1250    📉-1.77   🟥BEAR   15.234
...
============================================
🎯 24-ROUND SUMMARY:
🟢 BULL: 14 wins | 🔴 BEAR: 10 wins
📈 PRICE TREND: $640.1234 → $645.8920 (+5.77)
```

This viewer is perfect for monitoring the market, studying patterns, and getting real-time alerts without the complexity or risk of active trading.
