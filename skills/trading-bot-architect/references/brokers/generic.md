# Broker Genérico — Patrón de Abstracción Universal

## Interfaz abstracta BrokerConnector

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import Optional

@dataclass
class Order:
    symbol: str
    side: str          # 'buy' | 'sell'
    quantity: float
    order_type: str    # 'market' | 'limit' | 'stop'
    price: Optional[float] = None
    stop_price: Optional[float] = None

@dataclass
class Position:
    symbol: str
    side: str
    quantity: float
    avg_price: float
    unrealized_pnl: float

@dataclass
class Bar:
    timestamp: int
    open: float
    high: float
    low: float
    close: float
    volume: float

class BrokerConnector(ABC):
    @abstractmethod
    async def connect(self) -> None: pass
    
    @abstractmethod
    async def disconnect(self) -> None: pass
    
    @abstractmethod
    async def get_bars(self, symbol: str, timeframe: str, limit: int) -> list[Bar]: pass
    
    @abstractmethod
    async def subscribe_ticker(self, symbol: str, callback) -> None: pass
    
    @abstractmethod
    async def place_order(self, order: Order) -> str: pass
    
    @abstractmethod
    async def cancel_order(self, order_id: str, symbol: str) -> None: pass
    
    @abstractmethod
    async def get_positions(self) -> list[Position]: pass
    
    @abstractmethod
    async def get_balance(self) -> dict: pass
    
    @abstractmethod
    async def is_connected(self) -> bool: pass
```

## Integración en la arquitectura

```python
class SignalEngine:
    def __init__(self, broker: BrokerConnector, risk_gate: RiskGate):
        self.broker = broker
        self.risk_gate = risk_gate
    
    async def process_signal(self, signal):
        order = Order(symbol=signal.symbol, side=signal.side,
                      quantity=signal.quantity, order_type='market')
        
        approved, reason = await self.risk_gate.check(order)
        if not approved:
            logger.warning(f"Order rejected: {reason}")
            return
        
        order_id = await self.broker.place_order(order)
        logger.info(f"Order placed: {order_id}")

# Swap de broker sin tocar el Signal Engine:
# broker = AlpacaConnector(api_key, secret, paper=True)
# broker = BinanceConnector(api_key, secret, testnet=True)
# engine = SignalEngine(broker=broker, risk_gate=risk_gate)
```

## Checklist para un nuevo conector

```
□ Autenticación (API key, OAuth, etc.)
□ Manejo de errores HTTP (rate limit, auth error, server error)
□ Reconexión automática (websocket drops, timeout)
□ Normalización de símbolos
□ Normalización de cantidad (lotes vs unidades)
□ Manejo de fills parciales
□ Modo sandbox/paper verificado antes de live
□ Rate limits respetados
□ Logs de todas las operaciones
```
