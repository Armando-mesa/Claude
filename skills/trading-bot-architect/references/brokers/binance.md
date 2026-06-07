# Binance — ccxt + websockets

Binance es el exchange más líquido del mundo. Para bots de crypto es la primera opción.

## Librería recomendada: ccxt

```bash
pip install ccxt
```

```python
import ccxt.async_support as ccxt
import os

exchange = ccxt.binance({
    'apiKey': os.getenv('BINANCE_API_KEY'),
    'secret': os.getenv('BINANCE_SECRET'),
    'options': {
        'defaultType': 'spot',  # 'spot', 'future', 'margin'
    }
})

# Para futuros perpetuos (USDT-M)
exchange = ccxt.binance({
    'apiKey': ...,
    'secret': ...,
    'options': {'defaultType': 'future'}
})
```

## Datos de mercado

```python
# OHLCV histórico
ohlcv = await exchange.fetch_ohlcv('BTC/USDT', '1h', limit=500)
import pandas as pd
df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
df['timestamp'] = pd.to_datetime(df['timestamp'], unit='ms')

# Orderbook
orderbook = await exchange.fetch_order_book('BTC/USDT', limit=20)

# Precio actual
ticker = await exchange.fetch_ticker('BTC/USDT')
print(ticker['last'], ticker['bid'], ticker['ask'])
```

## Órdenes

```python
# Market order
order = await exchange.create_market_buy_order('BTC/USDT', 0.001)
order = await exchange.create_market_sell_order('BTC/USDT', 0.001)

# Limit order
order = await exchange.create_limit_buy_order('BTC/USDT', 0.001, 40000)

# Cancelar orden
await exchange.cancel_order(order['id'], 'BTC/USDT')

# Futuros — apalancamiento
await exchange.set_leverage(10, 'BTC/USDT')

# Futuros — market order
order = await exchange.create_order('BTC/USDT', 'market', 'buy', 0.001,
    params={'reduceOnly': False})
```

## Posiciones y balance

```python
# Balance
balance = await exchange.fetch_balance()
usdt_balance = balance['USDT']['free']

# Posiciones abiertas (futuros)
positions = await exchange.fetch_positions(['BTC/USDT'])
for pos in positions:
    if pos['contracts'] > 0:
        print(f"Side: {pos['side']}, Size: {pos['contracts']}, PnL: {pos['unrealizedPnl']}")

# Órdenes abiertas
open_orders = await exchange.fetch_open_orders('BTC/USDT')
```

## WebSocket — datos en tiempo real

```python
import asyncio
import json
import websockets

async def stream_klines(symbol, interval):
    ws_symbol = symbol.replace('/', '').lower()
    url = f"wss://stream.binance.com:9443/ws/{ws_symbol}@kline_{interval}"
    
    async with websockets.connect(url) as ws:
        async for message in ws:
            data = json.loads(message)
            kline = data['k']
            if kline['x']:  # vela cerrada
                print(f"Close: {kline['c']}, Volume: {kline['v']}")
```

## Testnet (paper trading)

```python
exchange = ccxt.binance({
    'apiKey': os.getenv('BINANCE_TESTNET_KEY'),
    'secret': os.getenv('BINANCE_TESTNET_SECRET'),
    'options': {'defaultType': 'future'},
    'urls': {
        'api': {
            'public': 'https://testnet.binancefuture.com/fapi/v1',
            'private': 'https://testnet.binancefuture.com/fapi/v1',
        }
    }
})
```

## Futuros vs Spot — diferencias clave

| Característica | Spot | Futuros |
|---|---|---|
| Apalancamiento | No | Hasta 125x |
| Short directo | No | Sí |
| Financiamiento | No | Funding rate cada 8h |
| Expiración | No | Perpetuos (no expiran) |
| Liquidación | No | Sí (riesgo de pérdida total) |
