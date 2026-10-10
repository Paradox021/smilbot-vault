# 🃏 Endpoints: Cartas y Gacha

Rutas gestionadas por `routers/cardRouter.js` y `controllers/cardController.js`.

> 🔐 Todas las rutas requieren autenticación. Permisos por ruta en [Autenticación](./Autenticacion.md#4-matriz-de-permisos).

- `POST /card` y `DELETE /card/:id` requieren **Admin** (bot o `ADMIN_DISCORD_IDS`).

---

## 1. Catálogo Completo
- **Ruta:** `GET /card`
- **Respuesta:** Array con todas las cartas ordenadas por rareza (`type: 1`).

---

## 2. Obtener Carta por ID
- **Ruta:** `GET /card/:id`
- **Respuesta:** Objeto `Card` individual.
- `404 { "error": "Card not found" }` si no existe o si el `id` no es un ObjectId válido.

---

## 3. Obtener Carta Aleatoria por Rareza
Permite obtener una carta aleatoria de una rareza específica:
- `GET /card/mythic`
- `GET /card/legendary`
- `GET /card/epic`
- `GET /card/rare`
- `GET /card/common`

Si no hay ninguna carta de esa rareza responde `404 { "error": "No cards found for this type" }`.

---

## 4. Tirada de Gacha del Usuario
- **Ruta:** `POST /user/:id/card/random` (`:id` = `discordId`). Es la única ruta de tirada; no existe `/cards/roll`.
- **Coste:** 100 monedas.
- **Errores:** `404` si el usuario no existe; `400 { "error": "No tienes suficientes monedas..." }` si el saldo no llega; `404 { "error": "No cards found for this type" }` si la rareza obtenida no tiene cartas en el catálogo (no se cobra: la compra se hace después de elegir la carta).
- **Acción y Sistema de Pity:**
  1. Valida que `user.balance >= 100`.
  2. Comprueba el estado de pity (`user.pityCount >= 250`):
     - **Si es Pity (`isPity: true`):** Selecciona directamente una carta Mítica aleatoria (`type: 4`), incrementa `pityMythicsCount += 1` y reinicia `pityCount = 0`.
     - **Si es Tirada Regular (`isPity: false`):** Genera un roll (0-999). Si sale Mítica natural (`roll < 5`), reinicia `pityCount = 0`. Si sale otra rareza, incrementa `pityCount += 1`.
  3. Descuenta 100 monedas (`balance -= 100`).
  4. Suma estadísticas: `totalCoinsSpent += 100`, `cardsOpenedCount += 1`.
  5. Añade la carta al inventario `user.cards` (incrementando `count` si ya existe).
  6. Registra la transacción inmutable en el Ledger (`CARD_BUY`) con `metadata: { cardId, cardType, cardName, roll, isPity, pityCountBefore, pityCountAfter }`.
- **Respuesta:** todos los campos de la `Card` obtenida más `roll`, `isPity` y `pity`:
  ```json
  {
    "_id": "642dbd17fc0fd3e62bde6659",
    "name": "SwimmingLarry",
    "description": "Una carta mítica acuática",
    "type": 4,
    "imageUrl": "https://...",
    "roll": null,
    "isPity": true,
    "pity": {
      "current": 0,
      "threshold": 250,
      "remaining": 250
    }
  }
  ```

---

## 5. Crear Nueva Carta (Admin / Multipart)
- **Ruta:** `POST /card`
- **Headers:** `Content-Type: multipart/form-data`
- **Campos:**
  - `data`: JSON string con:
    ```json
    {
      "name": "Nombre de la Carta",
      "type": "mythic",
      "description": "Lore o historia",
      "author": "Nombre o Discord Tag"
    }
    ```
  - `image`: Archivo binario de imagen (procesado por Multer e `imageService`). Obligatorio.
- `type` se envía como **nombre** (`common`, `rare`, `epic`, `legendary`, `mythic`) y el backend lo convierte a número (0-4).
- `name` debe ser único.
- **Respuesta (201 Created):** Objeto `Card` creado con `imageUrl` y `imagePublicId`.
- **Errores:**
  - `400 { "error": "image is required" }`: falta la imagen.
  - `400`: `data` no es JSON válido o no es un objeto; `type` no es uno de los cinco nombres; error de validación de Mongoose (p. ej. falta un campo obligatorio).
  - `409 { "error": "A card with that name already exists" }`: `name` duplicado.
- Si la creación falla después de guardar la imagen, el backend la borra del proveedor para no dejar ficheros huérfanos.

---

## 6. Eliminar Carta
- **Ruta:** `DELETE /card/:id`
- **Efecto (en una única transacción de MongoDB, `cardService.deleteCard`):**
  1. Quita la carta de **todos los inventarios** (`User.cards`).
  2. **Cancela** las ofertas activas de esa carta: `status = 'CANCELLED'`, `active = false`, `cancelledAt`. La copia retenida en escrow **se pierde**: no vuelve al vendedor porque la carta deja de existir.
  3. Registra un `CARD_REMOVED` en el Ledger por cada usuario afectado (poseedores y vendedores de ofertas canceladas). Ver [Mercado y Transacciones](../Modelos%20y%20Contratos/Mercado%20y%20Transacciones.md).
  4. Borra la carta.
- Tras confirmar la transacción se borra la imagen del proveedor de hosting (Cloudinary / Supabase / Local). Si ese borrado falla se registra en el log y la respuesta sigue siendo `200`.
- **Respuesta (200):**
  ```json
  {
    "deleted": true,
    "cardId": "642dbd17fc0fd3e62bde6659",
    "usersAffected": 3,
    "copiesRemoved": 5,
    "offersCancelled": 1
  }
  ```
- **Errores:** `404 { "error": "Card not found" }` si no existe o el `id` no es válido. Cualquier otro fallo → `500` y no se aplica ningún cambio (rollback).
- Requiere que MongoDB sea un **replica set** (como el mercado).
