# Kraken, Bybit y OKX — Crypto Exchanges

Los tres soportan ccxt — el código es prácticamente idéntico.

```python
import ccxt.async_support as ccxt

exchange = ccxt.kraken({'apiKey': ..., 'secret': ...})
exchange = ccxt.bybit({'apiKey': ..., 'secret': ..., 'options': {'defaultType': 'linear'}})
exchange = ccxt.okx({'apiKey': ..., 'secret': ..., 'password': os.getenv('OKX_PASSPHRASE')})
```

Fetch, órdenes y posiciones son idénticos a binance.md.

## Kraken
- El más regulado (FCA, FinCEN). Bueno para bots US/EU.
- Testnet: `beta.futures.kraken.com`
- Rate limits: 15 tokens, recupera 3/s

## Bybit
- Mayor exchange de derivados tras Binance. API muy documentada.
- Linear (USDT-margined) es el más común para bots.
- Testnet: `https://testnet.bybit.com` — API key separada.

```python
await exchange.set_leverage(10, 'BTC/USDT:USDT')
await exchange.set_position_mode(True, 'BTC/USDT')  # hedge mode
```

## OKX
- El más completo en variedad: spot, perpetuos, futuros, options, DeFi.
- Requiere passphrase adicional obligatorio.
- Paper trading nativo: `headers: {'x-simulated-trading': '1'}`

## Comparativa

| Exchange | Mejor para | Testnet |
|---|---|---|
| Kraken | Spot regulado US/EU | Sí (futuros) |
| Bybit | Perpetuos, derivatives | Sí |
| OKX | Variedad de productos | Sí (simulated) |
