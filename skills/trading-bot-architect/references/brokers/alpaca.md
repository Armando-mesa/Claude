# Alpaca — US Stocks & ETFs

Alpaca es el broker preferido para bots de acciones US por su API moderna, trading comission-free y paper trading integrado.

## Librería oficial

```bash
pip install alpaca-py
```

```python
from alpaca.trading.client import TradingClient
from alpaca.trading.requests import MarketOrderRequest, LimitOrderRequest
from alpaca.trading.enums import OrderSide, TimeInForce
from alpaca.data.historical import StockHistoricalDataClient
from alpaca.data.requests import StockBarsRequest
from alpaca.data.timeframe import TimeFrame
import os

client = TradingClient(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY'),
    paper=True
)
```

## Datos históricos

```python
data_client = StockHistoricalDataClient(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)

from datetime import datetime, timedelta

request_params = StockBarsRequest(
    symbol_or_symbols='AAPL',
    timeframe=TimeFrame.Hour,
    start=datetime.now() - timedelta(days=30)
)

bars = data_client.get_stock_bars(request_params)
df = bars.df
```

## Streaming en tiempo real

```python
from alpaca.data.live import StockDataStream

stream = StockDataStream(
    api_key=os.getenv('ALPACA_API_KEY'),
    secret_key=os.getenv('ALPACA_SECRET_KEY')
)

async def on_bar(bar):
    print(f"{bar.symbol}: O={bar.open} H={bar.high} L={bar.low} C={bar.close}")

stream.subscribe_bars(on_bar, 'AAPL', 'MSFT')
stream.run()
```

## Órdenes

```python
market_order_data = MarketOrderRequest(
    symbol='AAPL',
    qty=10,
    side=OrderSide.BUY,
    time_in_force=TimeInForce.DAY
)
order = client.submit_order(order_data=market_order_data)

# Bracket order
from alpaca.trading.requests import TakeProfitRequest, StopLossRequest

bracket_order = MarketOrderRequest(
    symbol='AAPL',
    qty=10,
    side=OrderSide.BUY,
    time_in_force=TimeInForce.DAY,
    order_class='bracket',
    take_profit=TakeProfitRequest(limit_price=185.00),
    stop_loss=StopLossRequest(stop_price=170.00)
)

client.cancel_orders()
client.close_all_positions(cancel_orders=True)
```

## PDT Rule — Pattern Day Trader

**Crítico para bots con acciones US:**

```python
# Si la cuenta tiene < $25,000 en equity:
# Máximo 3 day trades en 5 días hábiles (rolling)
account = client.get_account()
if int(account.daytrade_count) >= 3 and float(account.equity) < 25000:
    pass  # No hacer day trade
```

## Horario de mercado

```python
clock = client.get_clock()
if clock.is_open:
    print(f"Mercado abierto. Cierra en: {clock.next_close}")
else:
    print(f"Mercado cerrado. Abre en: {clock.next_open}")
```
