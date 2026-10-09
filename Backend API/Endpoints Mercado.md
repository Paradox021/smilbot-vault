# 🏪 Endpoints: Mercado P2P por Servidor

Las ofertas se gestionan como documentos independientes en la colección `market_offers`.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

- Publicar, comprar y cancelar son **Self**: con token de usuario, `body.discordId` debe ser el del propio usuario (si no → `403`). El bot puede actuar en nombre de cualquiera.

---

## 1. Listar Ofertas Activas del Servidor
- **Ruta:** `GET /market/:serverId/offers`
- **Filtro:** Solo ofertas con `status: 'ACTIVE'`.
- **Respuesta:** Array con oferta y datos de la carta poblada.

---

## 2. Publicar Carta en el Mercado
- **Ruta:** `POST /market/:serverId/offers`
- **Body:** `{ "discordId": string, "username": string, "cardName": string, "price": number }`
- **Validaciones y Efecto:**
  1. Verifica que el usuario posea la carta.
  2. Descuenta 1 unidad de la carta del inventario del vendedor (`count -= 1`).
  3. Crea el documento en `market_offers` con `status: 'ACTIVE'`.

---

## 3. Comprar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/buy`
- **Body:** `{ "discordId": string, "username": string }` (comprador)
- **Operación Atómica:**
  1. Comprador no puede ser el vendedor.
  2. Comprador debe tener `balance >= price`.
  3. Descuenta `price` al comprador e inserta `MARKET_BUY` en `transactions`.
  4. Suma `price` al vendedor (`totalCoinsEarned += price`) e inserta `MARKET_SELL` en `transactions`.
  5. Transfiere la carta al inventario del comprador.
  6. Actualiza la oferta a `status = 'SOLD'`, guardando `soldPrice`, `buyerDiscordId` y `soldAt`.

---

## 4. Cancelar Oferta
- **Ruta:** `DELETE /market/:serverId/offers/:offerId`
- **Body:** `{ "discordId": string, "username": string }` (vendedor)
- **Efecto:**
  1. Verifica que la oferta esté `ACTIVE` y pertenezca al vendedor.
  2. Devuelve la carta al inventario del vendedor (`count += 1`).
  3. Marca la oferta como `status = 'CANCELLED'` y guarda `cancelledAt`.
