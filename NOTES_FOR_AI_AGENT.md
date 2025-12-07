# Notas para el Agente de IA que Revisará Este Código

## 📋 Contexto General

Este es un bot de scalping de alta frecuencia diseñado para operar en Binance USDC Futures aprovechando los pares con 0% de comisiones maker. La estrategia combina análisis técnico tradicional (RSI) con microestructura de mercado (orderbook) para detectar reversiones tras pérdida de momentum.

**Objetivo**: Capturas pequeñas (0.03-0.18%) con alta frecuencia y win rate >65%.

---

## 🔴 PUNTOS CRÍTICOS A REVISAR

### 1. ⚡ Latencia y Race Conditions

**PROBLEMA CRÍTICO**: El edge de esta estrategia depende de entrar y colocar el TP inmediatamente.

**Detalles**:
- Target TP: 0.03% (3 basis points)
- En BTC @ $100,000 → Solo $30 de movimiento
- Si el orderbook se actualiza cada 100-300ms, hay ventana muy estrecha
- **El TP debe colocarse en <100ms tras la entrada o perdemos el edge**

**Qué revisar**:
```python
# En order_manager.py - ¿Se envían órdenes en paralelo?
async def place_entry_with_tp(symbol, side, notional, tp_price):
    # ❌ MAL: Secuencial
    entry_order = await exchange.create_order(...)
    await asyncio.sleep(0.1)  # ¡FATAL! 100ms de delay
    tp_order = await exchange.create_order(...)
    
    # ✅ BIEN: Paralelo
    entry_task = asyncio.create_task(exchange.create_order(...))
    tp_task = asyncio.create_task(exchange.create_order(...))
    entry_orde...