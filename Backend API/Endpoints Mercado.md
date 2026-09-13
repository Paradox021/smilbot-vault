# 🏪 Endpoints: Mercado P2P por Servidor

Las ofertas se gestionan como documentos independientes en la colección `market_offers`.

---

## 1. Listar Ofertas Activas del Servidor
- **Ruta:** `GET /market/:serverId/offers`
- **Filtro:** Solo ofertas con `status: 'ACTIVE'`.
- **Respuesta:** Array con oferta y datos de la carta poblada.

---

## 2. Publicar Carta en el Mercado
- **Ruta:** `POST /market/:serverId/offers`
- **Body:** `{ "sellerDiscordId": string, "cardId": string, "price": number }`
- **Validaciones y Efecto:**
  1. Verifica que el usuario posea la carta.
  2. Descuenta 1 unidad de la carta del inventario del vendedor (`count -= 1`).
  3. Crea el documento en `market_offers` con `status: 'ACTIVE'`.

---

## 3. Comprar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/buy`
- **Body:** `{ "buyerId": string }` (o `buyerDiscordId`)
- **Operación Atómica:**
  1. Comprador no puede ser el vendedor.
  2. Comprador debe tener `balance >= price`.
  3. Descuenta `price` al comprador e inserta `MARKET_BUY` en `transactions`.
  4. Suma `price` al vendedor (`totalCoinsEarned += price`) e inserta `MARKET_SELL` en `transactions`.
  5. Transfiere la carta al inventario del comprador.
  6. Actualiza la oferta a `status = 'SOLD'`, guardando `soldPrice`, `buyerDiscordId` y `soldAt`.

---

## 4. Cancelar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/cancel`
- **Body:** `{ "sellerDiscordId": string }`
- **Efecto:**
  1. Verifica que la oferta esté `ACTIVE` y pertenezca al vendedor.
  2. Devuelve la carta al inventario del vendedor (`count += 1`).
  3. Marca la oferta como `status = 'CANCELLED'` y guarda `cancelledAt`.
