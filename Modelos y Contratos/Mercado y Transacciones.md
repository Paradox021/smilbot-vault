# 🏪 Mercado P2P, Transacciones e Índices

Especificación detallada de la arquitectura del mercado libre y el libro mayor inmutable (Ledger).

---

## 1. Colección Independiente: `MarketOffer`

Para evitar documentos monolíticos gigantes que sobrepasen el límite de 16 MB en MongoDB, cada oferta vive como un documento independiente en `market_offers`:

```typescript
export type MarketOfferStatus = 'ACTIVE' | 'SOLD' | 'CANCELLED';

interface MarketOfferSchema {
  _id: ObjectId | string;
  serverId: string;                // ID del servidor de Discord (Guild ID)
  seller: ObjectId | string;       // Referencia a 'User'
  sellerDiscordId: string;         // DiscordId del vendedor
  cardId: ObjectId | string;       // Referencia a 'Card' en venta
  price: number;                   // Precio en monedas
  status: MarketOfferStatus;       // 'ACTIVE' | 'SOLD' | 'CANCELLED' (default: 'ACTIVE')
  
  // Metadatos de compra/cancelación
  buyer?: ObjectId | string | null;// Referencia a 'User' (comprador)
  buyerDiscordId?: string | null;  // DiscordId del comprador
  soldPrice?: number | null;       // Precio final acordado
  soldAt?: Date | null;            // Fecha de compra
  cancelledAt?: Date | null;       // Fecha de cancelación
  createdAt: Date;
  updatedAt: Date;
}
```

### Índices de Alto Rendimiento (MongoDB)
```javascript
marketOfferSchema.index({ serverId: 1, status: 1, createdAt: -1 });
marketOfferSchema.index({ seller: 1, status: 1 });
marketOfferSchema.index({ buyer: 1, status: 1 });
```

---

## 2. Colección Inmutable de Auditoría: `Transaction` (Ledger)

```typescript
export type TransactionType =
  | 'DAILY_CLAIM'      // Reclamo de dailybalance (+100)
  | 'CARD_BUY'         // Compra de sobre gacha (-100)
  | 'MARKET_BUY'       // Compra en el mercado (-price)
  | 'MARKET_SELL'      // Ganancia por venta en el mercado (+price)
  | 'TRADE'            // Intercambio directo
  | 'ADMIN_ADJUST';    // Ajuste manual por moderador

export interface TransactionSchema {
  _id: ObjectId | string;
  discordId: string;           // Usuario afectado
  type: TransactionType;
  amount: number;              // Variación (+/-)
  balanceBefore: number;       // Saldo previo
  balanceAfter: number;        // Saldo posterior
  metadata?: {
    cardId?: string;
    cardType?: number;         // 0 a 4
    roll?: number;             // Tirada RNG 0-999
    sellerDiscordId?: string;
    buyerDiscordId?: string;
    offerId?: string;
    streakAtClaim?: number;
    previousStreak?: number;
    previousMaxStreak?: number;
    isNewRecord?: boolean;
    streakBroken?: boolean;
  };
  createdAt: Date;
}
```

### Índices de Alto Rendimiento (MongoDB)
```javascript
transactionSchema.index({ discordId: 1, createdAt: -1 });
transactionSchema.index({ type: 1 });
```

---

## 3. Escrow y Atomicidad en el Mercado
1. **Al publicar (`POST /market/:id/offers`):** La carta se descuenta inmediatamente del inventario del vendedor (`count -= 1`) para actuar como depósito en garantía (*escrow*), evitando que la venda dos veces o la intercambie mientras está en el mercado.
2. **Al cancelar (`POST .../cancel`):** La carta se devuelve automáticamente al inventario del vendedor (`count += 1`).
3. **Al comprar (`POST .../buy`):** En una sesión transaccional de base de datos se transfiere el dinero, se transfiere la carta al comprador, y se emiten los dos registros contables (`MARKET_BUY` y `MARKET_SELL`).
