# Oanda — Forex & CFDs

```bash
pip install oandapyV20
```

```python
import oandapyV20
import os

client = oandapyV20.API(
    access_token=os.getenv('OANDA_TOKEN'),
    environment="practice"  # "practice" = demo, "live" = producción
)
ACCOUNT_ID = os.getenv('OANDA_ACCOUNT_ID')
```

## Datos

```python
from oandapyV20.endpoints.instruments import InstrumentsCandles

r = InstrumentsCandles(instrument="EUR_USD", params={"count": 200, "granularity": "H1"})
client.request(r)
candles = r.response['candles']
```

## Streaming

```python
from oandapyV20.endpoints.pricing import PricingStream

def stream_prices(instruments):
    r = PricingStream(accountID=ACCOUNT_ID, params={"instruments": instruments})
    for response in client.request(r):
        if response['type'] == 'PRICE':
            print(f"{response['instrument']}: {response['bids'][0]['price']}/{response['asks'][0]['price']}")
```

## Órdenes

```python
import oandapyV20.endpoints.orders as orders_ep

data = {
    "order": {
        "type": "MARKET",
        "instrument": "EUR_USD",
        "units": "10000",
        "timeInForce": "FOK",
        "stopLossOnFill": {"price": "1.0800"},
        "takeProfitOnFill": {"price": "1.0950"},
    }
}
r = orders_ep.OrderCreate(accountID=ACCOUNT_ID, data=data)
client.request(r)
```

## Instrumentos

- Forex: 70+ pares (EUR_USD, GBP_USD, USD_JPY...)
- Índices: US30, US500, DE30
- Commodities: XAU_USD, XAG_USD, BCO_USD

Nomenclatura: `BASE_QUOTE` — `EUR_USD`, no `EURUSD`.

Ventajas: demo gratuita sin límite, datos históricos hasta 2001, regulado FCA/CFTC/ASIC.
