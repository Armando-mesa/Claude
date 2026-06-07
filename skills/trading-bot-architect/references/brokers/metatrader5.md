# MetaTrader 5 (MT5) — Python Integration

MetaTrader es el ecosistema más grande del mundo para forex, CFDs y futuros retail.

## Restricción importante

**La librería oficial de Python requiere Windows** — MT5 debe estar instalado y corriendo en la misma máquina. No funciona en Linux/Mac directamente.

```bash
pip install MetaTrader5
```

```python
import MetaTrader5 as mt5

if not mt5.initialize():
    print(f"Error: {mt5.last_error()}")
    quit()

mt5.initialize(login=123456, password="tu_password", server="ICMarkets-Demo")
```

## Datos de mercado

```python
import pandas as pd

rates = mt5.copy_rates_from_pos("EURUSD", mt5.TIMEFRAME_H1, 0, 500)
df = pd.DataFrame(rates)
df['time'] = pd.to_datetime(df['time'], unit='s')

tick = mt5.symbol_info_tick("EURUSD")
print(f"Bid: {tick.bid}, Ask: {tick.ask}")
```

## Órdenes

```python
request = {
    "action": mt5.TRADE_ACTION_DEAL,
    "symbol": "EURUSD",
    "volume": 0.1,
    "type": mt5.ORDER_TYPE_BUY,
    "price": mt5.symbol_info_tick("EURUSD").ask,
    "sl": 1.0800,
    "tp": 1.0950,
    "deviation": 20,
    "magic": 12345,
    "comment": "mi_bot_v1",
    "type_time": mt5.ORDER_TIME_GTC,
    "type_filling": mt5.ORDER_FILLING_IOC,
}

result = mt5.order_send(request)
if result.retcode != mt5.TRADE_RETCODE_DONE:
    print(f"Error: {result.comment}")
```

**`magic`** identifica qué órdenes pertenecen a qué bot (crítico si hay varias estrategias en la misma cuenta).

## Posiciones

```python
positions = mt5.positions_get()
for pos in positions:
    print(f"{pos.symbol}: {pos.volume} lotes, PnL: {pos.profit}")
```

## Brokers compatibles

IC Markets, Pepperstone, FP Markets, AvaTrade, XM, Exness, Admiral Markets.
