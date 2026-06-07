# Interactive Brokers (IBKR) — ib_insync

IBKR es el broker preferido para bots de acciones, futuros y opciones por su API robusta, bajos costos y acceso global.

## Prerequisito: IB Gateway o TWS

```bash
# IB Gateway (recomendado para producción — sin UI)
# Descargar de: https://www.interactivebrokers.com/en/trading/ibgateway-latest.php
# Correr en paper trading primero (puerto 4002)
# Live trading: puerto 4001
```

## Librería recomendada: ib_insync

```bash
pip install ib_insync
```

```python
from ib_insync import *
import asyncio

# Conexión
ib = IB()
await ib.connectAsync('127.0.0.1', 4002, clientId=1)  # 4002=paper, 4001=live

# Verificar conexión
print(ib.isConnected())
```

## Datos de mercado

```python
# Definir contrato
contract = Stock('AAPL', 'SMART', 'USD')
await ib.qualifyContractsAsync(contract)

# Datos históricos
bars = await ib.reqHistoricalDataAsync(
    contract,
    endDateTime='',
    durationStr='30 D',
    barSizeSetting='1 hour',
    whatToShow='TRADES',
    useRTH=True
)
df = util.df(bars)

# Streaming en tiempo real
def onPendingTickers(tickers):
    for t in tickers:
        print(t.contract.symbol, t.last, t.bid, t.ask)

ib.pendingTickersEvent += onPendingTickers
ib.reqMktData(contract)
```

## Órdenes

```python
# Market order
order = MarketOrder('BUY', 100)
trade = ib.placeOrder(contract, order)

# Limit order
order = LimitOrder('BUY', 100, 175.00)
trade = ib.placeOrder(contract, order)

# Stop loss
order = StopOrder('SELL', 100, 170.00)

# Bracket order (entrada + SL + TP en un solo bloque)
bracket = ib.bracketOrder(
    action='BUY',
    quantity=100,
    limitPrice=175.00,
    takeProfitPrice=185.00,
    stopLossPrice=170.00
)
for o in bracket:
    ib.placeOrder(contract, o)

# Monitorear fill
trade.fillEvent += lambda trade, fill: print(f"Fill: {fill}")
```

## Posiciones y cuenta

```python
# Posiciones abiertas
positions = ib.positions()
for pos in positions:
    print(f"{pos.contract.symbol}: {pos.position} @ {pos.avgCost}")

# P&L en tiempo real
ib.reqPnL(ib.managedAccounts()[0])
def onPnL(pnl):
    print(f"Daily PnL: {pnl.dailyPnL}, Unrealized: {pnl.unrealizedPnL}")
ib.pnlEvent += onPnL

# Balance de cuenta
accountValues = ib.accountValues()
```

## Watchdog (reconexión automática)

```python
from ib_insync import IBC, Watchdog

# Para producción: reconexión automática si se cae la conexión
ibc = IBC(978, gateway=True, tradingMode='paper')
watchdog = Watchdog(ibc, ib, port=4002)
watchdog.start()
```

## Rate limits y pacing

IBKR tiene pacing restrictions para datos históricos:
- Máximo 60 requests cada 10 minutos
- Si se excede: error "pacing violation"

```python
# Añadir delay entre requests históricos
import asyncio
for symbol in symbols:
    bars = await ib.reqHistoricalDataAsync(...)
    await asyncio.sleep(1)  # 1 segundo entre requests
```

## Contratos comunes

```python
# Acciones
Stock('AAPL', 'SMART', 'USD')

# Futuros (ES = S&P 500 mini)
Futures('ES', '20240315', 'CME')

# Forex
Forex('EURUSD')

# Opciones
Option('AAPL', '20240315', 175, 'C', 'SMART')  # Call 175 strike

# Crypto (IBKR tiene BTC, ETH)
Crypto('BTC', 'PAXOS', 'USD')
```
