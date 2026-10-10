# 🏪 Mercado P2P, Transacciones e Índices

Especificación de la arquitectura del mercado libre y el libro mayor inmutable (Ledger). Endpoints en [Endpoints Mercado](../Backend%20API/Endpoints%20Mercado.md).

---

## 1. Colección Independiente: `MarketOffer`

Para evitar documentos monolíticos que sobrepasen el límite de 16 MB en MongoDB, cada oferta vive como un documento independiente en `market_offers` (`models/marketOffer.js`):

```typescript
export type MarketOfferStatus = 'ACTIVE' | 'SOLD' | 'CANCELLED';

interface MarketOfferSchema {
  _id: ObjectId | string;
  serverId: string;                // ID del servidor de Discord (Guild ID)
  seller: ObjectId | string;       // Referencia a 'User' (obligatorio)
  sellerDiscordId: string | null;  // DiscordId del vendedor
  cardId: ObjectId | string;       // Referencia a 'Card' en venta (obligatorio)
  price: number;                   // Precio en monedas
  status: MarketOfferStatus;       // default: 'ACTIVE'
  active: boolean;                 // Campo heredado (default: true). Se mantiene sincronizado con status

  // Metadatos de compra/cancelación
  buyer: ObjectId | string | null; // Referencia a 'User' (comprador)
  buyerDiscordId: string | null;   // DiscordId del comprador
  soldPrice: number | null;        // Precio final
  soldAt: Date | null;             // Fecha de compra
  cancelledAt: Date | null;        // Fecha de cancelación
  createdAt: Date;
  updatedAt: Date;
}
```

> `models/market.js` conserva el antiguo modelo `Market` (ofertas embebidas en un documento por servidor). Ya no se usa: el servicio solo trabaja con `MarketOffer`.

### Índices (MongoDB)
```javascript
marketOfferSchema.index({ serverId: 1, status: 1, createdAt: -1 });
marketOfferSchema.index({ seller: 1, status: 1 });
marketOfferSchema.index({ buyer: 1, status: 1 });
// además de índices simples en serverId, seller, status y buyer
```

---

## 2. Colección Inmutable de Auditoría: `Transaction` (Ledger)

```typescript
export type TransactionType =
  | 'DAILY_CLAIM'      // Reclamo de dailybalance (+100)
  | 'CARD_BUY'         // Compra de sobre gacha (-100)
  | 'MARKET_BUY'       // Compra en el mercado (-price)
  | 'MARKET_SELL'      // Ganancia por venta en el mercado (+price)
  | 'TRADE'            // Reservado: ningún endpoint lo genera todavía
  | 'ADMIN_ADJUST'     // Ajuste manual (POST/DELETE /user/:id/balance/:amount)
  | 'CARD_REMOVED';    // Un admin borró del catálogo una carta que el usuario tenía o vendía (amount 0)

export interface TransactionSchema {
  _id: ObjectId | string;
  discordId: string;           // Usuario afectado
  type: TransactionType;
  amount: number;              // Variación (+/-)
  balanceBefore: number;       // Saldo previo
  balanceAfter: number;        // Saldo posterior
  metadata: object;            // Depende del tipo (ver tabla). Default: {}
  createdAt: Date;
  updatedAt: Date;
}
```

### `metadata` por tipo (lo que el código escribe realmente)

| Tipo | `metadata` |
| :--- | :--- |
| `DAILY_CLAIM` | `{ streakAtClaim, previousStreak, previousMaxStreak, isNewRecord, streakBroken }` |
| `CARD_BUY` | `{ cardId, cardType, cardName, roll, isPity, pityCountBefore, pityCountAfter }` (`roll: null` si fue por pity) |
| `MARKET_BUY` | `{ cardId, sellerDiscordId, offerId }` |
| `MARKET_SELL` | `{ cardId, buyerDiscordId, offerId }` |
| `ADMIN_ADJUST` | `{}` desde `POST/DELETE /user/:id/balance/:amount`. El snapshot de migración (`scripts/backfillStatsAndLedger.js`) escribe `{ note, migratedAt, legacySnapshot }`. |
| `CARD_REMOVED` | `{ cardId, cardName, cardType, copiesRemoved, offersCancelled }`. `amount: 0` y `balanceBefore = balanceAfter`. |
| `TRADE` | Reservado: ningún código lo escribe. |

En `MARKET_BUY` y `MARKET_SELL`, `cardId` y `offerId` se guardan como string. El comentario de `models/transaction.js` coincide con esta tabla.

### `CARD_REMOVED` (borrado de una carta del catálogo)
`DELETE /card/:id` ([Endpoints Cartas](../Backend%20API/Endpoints%20Cartas.md#6-eliminar-carta)) escribe **uno por usuario afectado**, dentro de la misma transacción que el borrado:
- `copiesRemoved`: copias quitadas de su inventario (puede ser `0` si solo tenía la carta en venta).
- `offersCancelled`: ofertas activas suyas que se cancelaron. La copia en escrow de esas ofertas se pierde.

### Índices (MongoDB)
```javascript
transactionSchema.index({ discordId: 1, createdAt: -1 });
transactionSchema.index({ type: 1 });
```

---

## 3. Escrow y Atomicidad en el Mercado
1. **Al publicar (`POST /market/:serverId/offers`):** la carta se descuenta del inventario del vendedor (`count -= 1`) como depósito en garantía (*escrow*), para que no pueda venderla dos veces.
2. **Al cancelar (`DELETE /market/:serverId/offers/:offerId`):** la carta vuelve al inventario del vendedor (`count += 1`).
3. **Al comprar (`POST .../buy`):** se marca la oferta como vendida, se transfiere la carta, se mueve el dinero y se emiten `MARKET_BUY` y `MARKET_SELL`.
4. **Si un admin borra la carta (`DELETE /card/:id`):** sus ofertas activas pasan a `CANCELLED` (con `active: false` y `cancelledAt`), pero la copia en escrow **no** vuelve al vendedor, porque la carta ya no existe. Se registra un `CARD_REMOVED`.

Publicar, cancelar y comprar se ejecutan dentro de una **transacción de MongoDB** (`mongoose.connection.transaction` en `services/marketService.js`; el borrado de carta usa su propia transacción en `services/cardService.js`):
- Si cualquier paso falla, no se aplica ninguno (oferta, inventarios, saldos y Ledger).
- Ante un conflicto de escritura (p. ej. dos compras simultáneas de la misma oferta) la transacción se reintenta; el segundo intento ve la oferta ya vendida y responde `400 Offer is not active`.
- Requiere que MongoDB sea un **replica set** (Atlas lo es). Un MongoDB *standalone* rechaza las transacciones.
