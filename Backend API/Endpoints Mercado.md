# 🏪 Endpoints: Mercado P2P por Servidor

Las ofertas se gestionan como documentos independientes en la colección `market_offers` (modelo en [Mercado y Transacciones](../Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md)).
Rutas en `routers/marketRouter.js`, `controllers/marketController.js` y `services/marketService.js`. `:serverId` es el ID del servidor (guild) de Discord.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

- Publicar, comprar y cancelar son **Self**: con token de usuario, `body.discordId` debe ser el del propio usuario (si no → `403`). El bot puede actuar en nombre de cualquiera.
- Publicar, comprar y cancelar **crean el usuario** si el `discordId` del body no existe (con el `username` del body).
- ⚠️ El body usa siempre `discordId` (no `sellerDiscordId` ni `buyerDiscordId`). Si falta, no se valida: el backend intenta crear un usuario sin `discordId` y responde `500`.

---

## 1. Listar Ofertas Activas del Servidor
- **Ruta:** `GET /market/:serverId/offers`
- **Filtro:** Ofertas con `status: 'ACTIVE'` (y las antiguas sin `status` con `active: true`). Orden: más reciente primero.
- **Respuesta:** Array de ofertas con `cardId` poblado (la `Card` completa) y `seller` poblado (el documento `User` completo del vendedor).
- No existe `GET /market/:serverId/offers/:offerId` para una sola oferta.

---

## 2. Publicar Carta en el Mercado
- **Ruta:** `POST /market/:serverId/offers`
- **Body:** `{ "discordId": string, "username": string, "cardName": string, "price": number }`
- **Efecto:**
  1. Busca en el inventario del vendedor una carta con ese **nombre exacto** (`cardName`) y `count > 0`.
  2. Crea el documento en `market_offers` con `status: 'ACTIVE'`.
  3. Descuenta 1 unidad de la carta del inventario del vendedor (escrow).
- **Respuesta (`200`):** `{ cardId, price, seller, sellerDiscordId, active, status }`. ⚠️ No incluye el `_id` de la oferta creada.
- **Errores:** si no tiene la carta → `500 { "error": "Card not found in your collection" }`. No se valida que `price` sea positivo.

---

## 3. Comprar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/buy`
- **Body:** `{ "discordId": string, "username": string }` (comprador)
- **Efecto (en este orden):**
  1. Marca la oferta como `status = 'SOLD'` (guarda `buyer`, `buyerDiscordId`, `soldPrice`, `soldAt`).
  2. Añade la carta al inventario del comprador.
  3. Descuenta `price` al comprador (`totalCoinsSpent += price`) e inserta `MARKET_BUY` en `transactions`.
  4. Suma `price` al vendedor (`totalCoinsEarned += price`) e inserta `MARKET_SELL` en `transactions`.
- ⚠️ **No es atómico:** los pasos se ejecutan uno tras otro sin sesión transaccional de MongoDB. Si uno falla a mitad, los anteriores no se deshacen.
- **Respuesta (`200`):** la oferta actualizada (`status: 'SOLD'`).
- **Errores:**
  - `404 { "error": "Offer not found" }` / `404 { "error": "Seller not found" }`
  - `400 { "error": "Offer is not active" }`
  - `400 { "error": "You don't have enough money" }`
  - `400 { "error": "You can't buy your own offer" }`

---

## 4. Cancelar Oferta
- **Ruta:** `DELETE /market/:serverId/offers/:offerId`
- **Body:** `{ "discordId": string, "username": string }` (vendedor)
- **Efecto:**
  1. Verifica que la oferta exista, pertenezca al vendedor y esté `ACTIVE`.
  2. Marca la oferta como `status = 'CANCELLED'` y guarda `cancelledAt`.
  3. Devuelve la carta al inventario del vendedor (`count += 1`).
- **Respuesta (`200`):** la oferta actualizada.
- **Errores:** todos como `500`: `Offer not found`, `You can't remove an offer that is not yours!`, `Offer is not active`.
