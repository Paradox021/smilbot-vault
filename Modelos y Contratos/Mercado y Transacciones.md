# 🏪 Mercado Libre y Transacciones

---

## 1. Ciclo de Vida de una Oferta (`MarketOfferStatus`)

- `ACTIVE`: Oferta publicada en el servidor, disponible para compra.
- `SOLD`: Compra realizada exitosamente. Las monedas van al vendedor y la carta al comprador.
- `CANCELLED`: Oferta retirada por el vendedor; la carta regresa a su inventario.

---

## 2. Reglas de Validación Contable
1. El comprador no puede ser el mismo vendedor (`buyerId !== sellerId`).
2. El comprador debe tener saldo suficiente (`balance >= price`).
3. El vendedor debe ser el propietario legítimo de la carta en el momento de crear la oferta.
4. Toda operación registra un asiento contable en la colección `transactions`.

---

## 3. Tipos de Transacción en Ledger
- `DAILY_CLAIM`: +100 monedas.
- `CARD_BUY`: -100 monedas (compra de sobre gacha).
- `MARKET_BUY`: -precio (asiento de compra).
- `MARKET_SELL`: +precio (asiento de venta).
- `ADMIN_ADJUST`: Ajustes administrativos.

Véase también:
- [[Backend API/Endpoints Mercado|Endpoints Mercado]]
