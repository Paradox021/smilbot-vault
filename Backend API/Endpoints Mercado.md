# 🏪 Endpoints: Mercado P2P por Servidor

Las ofertas se gestionan como documentos independientes en la colección `market_offers` (modelo en [Mercado y Transacciones](../Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md)).
Rutas en `routers/marketRouter.js`, `controllers/marketController.js` y `services/marketService.js`. `:serverId` es el ID del servidor (guild) de Discord.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

- Publicar, comprar y cancelar son **Self**: con token de usuario, `body.discordId` debe ser el del propio usuario (si no → `403`). El bot puede actuar en nombre de cualquiera.
- **Atómicas:** publicar, comprar y cancelar se ejecutan dentro de una **transacción de MongoDB**. Si un paso falla, no se aplica ninguno. Dos compras simultáneas de la misma oferta no pueden tener éxito las dos. (Requiere que MongoDB sea un replica set, como Atlas.)
- **Body común** de publicar, comprar y cancelar: `discordId` (obligatorio, string) y `username` (string, opcional). Publicar y comprar **crean el usuario** si no existe; en ese caso `username` es obligatorio (`400` si falta).
- Todos los errores tienen la forma `{ "error": "<mensaje>" }`.

---

## 1. Listar Ofertas Activas del Servidor
- **Ruta:** `GET /market/:serverId/offers`
- **Filtro:** ofertas con `status: 'ACTIVE'` (y las antiguas sin `status` con `active: true`). Orden: más reciente primero.
- **Respuesta (`200`):** array de ofertas con `cardId` poblado (la `Card` completa) y `seller` reducido a `{ _id, discordId, username }`.
```json
[
  {
    "_id": "6702...",
    "serverId": "111222333444555666",
    "seller": { "_id": "66bc...", "discordId": "123456789012345678", "username": "alice" },
    "sellerDiscordId": "123456789012345678",
    "cardId": { "_id": "642d...", "name": "SwimmingLarry", "type": 4, "imageUrl": "https://...", "description": "...", "author": "..." },
    "price": 300,
    "status": "ACTIVE",
    "active": true,
    "buyer": null,
    "buyerDiscordId": null,
    "soldPrice": null,
    "soldAt": null,
    "cancelledAt": null,
    "createdAt": "2026-10-10T12:00:00.000Z",
    "updatedAt": "2026-10-10T12:00:00.000Z"
  }
]
```

---

## 2. Obtener una Oferta
- **Ruta:** `GET /market/:serverId/offers/:offerId`
- **Respuesta (`200`):** la oferta con el mismo formato que en el listado, **en cualquier estado** (`ACTIVE`, `SOLD`, `CANCELLED`). Comprueba `status` antes de mostrarla como disponible.
- **Errores:** `404 Offer not found` si no existe, no pertenece a ese servidor o el id no es válido.

---

## 3. Publicar Carta en el Mercado
- **Ruta:** `POST /market/:serverId/offers`
- **Body:** `{ "discordId": string, "username"?: string, "cardId"?: string, "cardName"?: string, "price": number }`
  - La carta se indica con **`cardId`** (recomendado) o, si no se envía, con **`cardName`** (nombre exacto).
  - `price`: entero positivo.
- **Efecto:** retira 1 copia de la carta del inventario del vendedor (escrow) y crea la oferta con `status: 'ACTIVE'`.
- **Respuesta (`201`):** la oferta creada, con su `_id` (`cardId` y `seller` sin poblar, como ids).
- **Errores:**
  - `400`: `discordId is required`, `cardId or cardName is required`, `cardId is not valid`, `price must be a positive integer`, `username is required for new users`.
  - `404 Card not found in your collection`: la carta no existe o el vendedor no tiene copias.

---

## 4. Comprar Oferta
- **Ruta:** `POST /market/:serverId/offers/:offerId/buy`
- **Body:** `{ "discordId": string, "username"?: string }` (comprador)
- **Efecto (en una transacción):**
  1. Marca la oferta como `status = 'SOLD'` (guarda `buyer`, `buyerDiscordId`, `soldPrice`, `soldAt`).
  2. Descuenta `price` al comprador (`totalCoinsSpent += price`) y le añade la carta.
  3. Suma `price` al vendedor (`totalCoinsEarned += price`).
  4. Inserta `MARKET_BUY` (comprador) y `MARKET_SELL` (vendedor) en `transactions`.
- **Respuesta (`200`):** la oferta actualizada (`status: 'SOLD'`, sin poblar).
- **Errores:**
  - `400`: `Offer is not active`, `You don't have enough money`, `You can't buy your own offer`, errores de validación del body.
  - `404`: `Offer not found`, `Seller not found`.

---

## 5. Cancelar Oferta
- **Ruta:** `DELETE /market/:serverId/offers/:offerId`
- **Body:** `{ "discordId": string }` (vendedor)
- **Efecto (en una transacción):** marca la oferta como `status = 'CANCELLED'` (guarda `cancelledAt`) y devuelve la carta al inventario del vendedor (`count += 1`).
- **Respuesta (`200`):** la oferta actualizada.
- **Errores:**
  - `400`: `Offer is not active`, errores de validación del body.
  - `403 You can't remove an offer that is not yours!`
  - `404 Offer not found`
- Una oferta también puede pasar a `CANCELLED` sin esta ruta si un admin borra la carta del catálogo (`DELETE /card/:id`). En ese caso la copia en escrow **no** se devuelve. Ver [Endpoints Cartas](./Endpoints%20Cartas.md#6-eliminar-carta).
