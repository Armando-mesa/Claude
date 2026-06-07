# TradingView — Webhooks como puente universal

```
[Pine Script strategy] → [Alert en TradingView] → [Webhook HTTP POST] → [Tu bot Python]
```

## Servidor webhook (FastAPI)

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import os

app = FastAPI()
WEBHOOK_SECRET = os.getenv('WEBHOOK_SECRET')

class TradingViewAlert(BaseModel):
    action: str          # "buy" | "sell" | "close"
    ticker: str
    price: float
    quantity: float
    secret: str = ""

@app.post("/webhook")
async def receive_alert(alert: TradingViewAlert):
    if alert.secret != WEBHOOK_SECRET:
        raise HTTPException(status_code=401, detail="Unauthorized")
    await execution_engine.process_signal(alert)
    return {"status": "ok"}
```

## Pine Script — JSON en el campo Message

```json
{
  "action": "{{strategy.order.action}}",
  "ticker": "{{ticker}}",
  "price": {{close}},
  "quantity": {{strategy.order.contracts}},
  "secret": "MI_TOKEN_SECRETO"
}
```

Webhook URL en TradingView: `https://tuservidor.com/webhook`

## Signal Adapter

```python
class TradingViewSignalAdapter:
    def normalize(self, alert: TradingViewAlert) -> Signal:
        return Signal(
            symbol=self.normalize_symbol(alert.ticker),
            side=alert.action,
            quantity=alert.quantity,
            price=alert.price,
            source="tradingview"
        )
    
    def normalize_symbol(self, tv_ticker: str) -> str:
        mapping = {"EURUSD": "EUR_USD", "BTCUSDT": "BTC/USDT"}
        return mapping.get(tv_ticker, tv_ticker)
```

## Limitaciones
- Latencia 1-3s (no apto para HFT)
- Webhook solo disponible en plan TradingView de pago (Essential+)
- No hay confirmación de ejecución desde el broker hacia TradingView
- Exposición: ngrok para dev, VPS + HTTPS para producción
