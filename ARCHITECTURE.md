```mermaid
flowchart TD

TG[Telegram<br/>/start /status /stop /positions /analysis] --> ORCH

ORCH[OpenClaw Main<br/>- routing & auth(optional)<br/>- trade_enabled flag<br/>- state + db query] --> K_TICK
ORCH --> B_TICK

K_TICK[Kiwoom Tick Scheduler<br/>15/day · 08:00~15:30<br/>trade window: 09:00~15:30] --> K_DATA
B_TICK[Bybit Tick Scheduler<br/>60/day · every 24m · 24/7] --> B_DATA

K_DATA[Kiwoom Data<br/>OHLCV daily/minute · cache 1Y] --> UNI
B_DATA[Bybit Data<br/>kline/ticker · cache] --> UNI

UNI[Universe<br/>Kiwoom: 20D value>=20억, price 1,500~200,000<br/>exclude 관리/우선/ETF/스팩, sector priority<br/>Bybit: volume/volatility top-N] --> IND

IND[Signals<br/>BUY gate: close>SMA240<br/>Entry: crossup(5,20)+vol1.5 OR 20D breakout+vol1.5 OR pullback(-2~-4% & above SMA20)<br/>Exit: -4% SL, +7% TP(half)+trail -3%, time exit 10D] --> DEC

DEC[Decision<br/><b>BUY / SELL / HOLD (forced)</b><br/>no position: BUY else HOLD<br/>has position: exit->SELL else HOLD] --> RISK

RISK[Risk Guard<br/>max 8 positions · sector cap 40%<br/>daily -3% => block NEW BUY (SELL allowed)<br/>MDD -10% => size 50% down] --> SIZE

SIZE[Position Sizer<br/>Kiwoom: 1,500,000 KRW, 75 splits => 20,000 KRW/slot<br/>Bybit: 150 quote, 75 splits => slot sizing] --> PRE

PRE[Pre-Trade Checks<br/>cash/min order/lot size<br/>max pos/sector cap/daily stop] --> ORDER

ORDER[Execution<br/>place/cancel/replace] --> K_BROK
ORDER --> B_BROK

K_BROK[Kiwoom Adapter<br/>paper/live] --> DB
B_BROK[Bybit Adapter<br/>paper/live] --> DB

DB[(SQLite<br/>signals, orders, fills, positions,<br/>daily_metrics, events/errors)] --> REP
REP[Reporting/Alerts<br/>telegram notify + EOD report + CSV] --> TG

PM2[PM2 Monitor<br/>restart + heartbeat + notify] -.-> ORCH 
```

